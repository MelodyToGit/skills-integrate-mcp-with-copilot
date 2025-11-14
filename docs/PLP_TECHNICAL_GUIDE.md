# PLP Technical Implementation Guide
## Developer Reference - Team 3

**Version:** 1.0 | **Date:** November 13, 2025 | **ADO:** 16467101

---

## 🎯 Purpose

This guide provides developers with practical, code-level guidance for implementing the Product Listing Page. It includes code examples, patterns, and best practices based on the analysis in [PLP_IMPLEMENTATION_ANALYSIS.md](./PLP_IMPLEMENTATION_ANALYSIS.md).

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                      Client Browser                      │
│  ┌──────────────────────────────────────────────────┐  │
│  │  React/Vue Components                             │  │
│  │  - ProductGrid, ProductCard, Filters, Sort       │  │
│  └────────────────┬─────────────────────────────────┘  │
└───────────────────┼──────────────────────────────────────┘
                    │ HTTP/REST
┌───────────────────┼──────────────────────────────────────┐
│                   ▼                                       │
│  ┌──────────────────────────────────────────────────┐   │
│  │  FastAPI Application Server                       │   │
│  │  - Routes, Controllers, Business Logic           │   │
│  └────────┬───────────────────┬──────────────────────┘  │
│           │                   │                          │
│  ┌────────▼─────┐    ┌───────▼────────┐                │
│  │  PostgreSQL  │    │  Redis Cache   │                 │
│  │  (Products)  │    │  (API Cache)   │                 │
│  └──────────────┘    └────────────────┘                 │
└─────────────────────────────────────────────────────────┘
```

---

## 📦 Data Models

### Database Schema (PostgreSQL)

```sql
-- Products Table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    product_id VARCHAR(50) UNIQUE NOT NULL,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(100),
    brand VARCHAR(100),
    regular_price DECIMAL(10, 2) NOT NULL,
    sale_price DECIMAL(10, 2),
    currency VARCHAR(3) DEFAULT 'USD',
    inventory_quantity INTEGER DEFAULT 0,
    low_stock_threshold INTEGER DEFAULT 10,
    is_available BOOLEAN DEFAULT TRUE,
    average_rating DECIMAL(3, 2),
    review_count INTEGER DEFAULT 0,
    tags TEXT[],
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes for performance
    INDEX idx_products_category (category),
    INDEX idx_products_brand (brand),
    INDEX idx_products_price (regular_price, sale_price),
    INDEX idx_products_created_at (created_at DESC),
    INDEX idx_products_available (is_available),
    
    -- Full-text search
    FULLTEXT INDEX idx_products_search (name, description)
);

-- Product Images Table
CREATE TABLE product_images (
    id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(id) ON DELETE CASCADE,
    url VARCHAR(500) NOT NULL,
    alt_text VARCHAR(255),
    image_type VARCHAR(20) DEFAULT 'secondary',
    sort_order INTEGER DEFAULT 0,
    
    INDEX idx_images_product (product_id)
);

-- Product Attributes Table
CREATE TABLE product_attributes (
    id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(id) ON DELETE CASCADE,
    attribute_name VARCHAR(50) NOT NULL,
    attribute_value VARCHAR(100) NOT NULL,
    
    INDEX idx_attributes_product (product_id),
    INDEX idx_attributes_name (attribute_name)
);
```

### Python Data Models (Pydantic)

```python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from datetime import datetime
from decimal import Decimal

class PriceInfo(BaseModel):
    regular: Decimal = Field(gt=0, description="Regular price")
    sale: Optional[Decimal] = Field(None, ge=0, description="Sale price")
    currency: str = Field(default="USD", max_length=3)
    
    @validator('sale')
    def sale_less_than_regular(cls, v, values):
        if v is not None and 'regular' in values and v >= values['regular']:
            raise ValueError('Sale price must be less than regular price')
        return v

class InventoryInfo(BaseModel):
    available: bool = True
    quantity: int = Field(ge=0)
    low_stock_threshold: int = Field(default=10, ge=0)
    
    @property
    def is_low_stock(self) -> bool:
        return self.available and 0 < self.quantity <= self.low_stock_threshold

class ProductImage(BaseModel):
    url: str
    alt: Optional[str] = None
    type: str = Field(default="secondary", pattern="^(primary|secondary|thumbnail)$")

class RatingInfo(BaseModel):
    average: Decimal = Field(ge=0, le=5)
    count: int = Field(ge=0)

class Product(BaseModel):
    product_id: str = Field(..., max_length=50)
    sku: str = Field(..., max_length=50)
    name: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = None
    category: Optional[str] = None
    brand: Optional[str] = None
    price: PriceInfo
    inventory: InventoryInfo
    images: List[ProductImage] = []
    attributes: dict = {}
    ratings: Optional[RatingInfo] = None
    tags: List[str] = []
    created_at: datetime
    updated_at: datetime
    
    class Config:
        json_encoders = {
            Decimal: lambda v: float(v),
            datetime: lambda v: v.isoformat()
        }

class ProductListItem(BaseModel):
    """Lightweight product model for list display"""
    product_id: str
    name: str
    price: PriceInfo
    primary_image: Optional[str] = None
    rating: Optional[RatingInfo] = None
    in_stock: bool
    badge: Optional[str] = None  # "Best Seller", "New", "Sale"

class PaginationInfo(BaseModel):
    total: int = Field(ge=0)
    page: int = Field(ge=1)
    limit: int = Field(ge=1, le=100)
    pages: int = Field(ge=0)

class FilterOptions(BaseModel):
    categories: List[str]
    brands: List[str]
    price_range: dict = {"min": 0, "max": 0}

class ProductListResponse(BaseModel):
    products: List[ProductListItem]
    pagination: PaginationInfo
    filters: FilterOptions
```

---

## 🛣️ API Implementation

### FastAPI Routes

```python
from fastapi import FastAPI, Query, HTTPException, Depends
from typing import Optional, List
from sqlalchemy.orm import Session
from . import models, schemas, crud

app = FastAPI(title="E-Commerce API", version="1.0")

@app.get("/api/v1/products", response_model=schemas.ProductListResponse)
async def list_products(
    page: int = Query(1, ge=1, description="Page number"),
    limit: int = Query(24, ge=1, le=100, description="Items per page"),
    category: Optional[str] = Query(None, description="Filter by category"),
    brand: Optional[str] = Query(None, description="Filter by brand"),
    min_price: Optional[float] = Query(None, ge=0, description="Minimum price"),
    max_price: Optional[float] = Query(None, ge=0, description="Maximum price"),
    sort: str = Query("newest", regex="^(price_asc|price_desc|rating|newest|popular)$"),
    search: Optional[str] = Query(None, min_length=1, description="Search query"),
    in_stock: Optional[bool] = Query(None, description="Filter by availability"),
    tags: Optional[List[str]] = Query(None, description="Filter by tags"),
    db: Session = Depends(get_db)
):
    """
    Retrieve paginated list of products with filtering and sorting.
    
    - **page**: Page number (starts at 1)
    - **limit**: Number of items per page (max 100)
    - **category**: Filter by product category
    - **brand**: Filter by brand name
    - **min_price**, **max_price**: Price range filter
    - **sort**: Sort order (price_asc, price_desc, rating, newest, popular)
    - **search**: Full-text search query
    - **in_stock**: Show only available products
    - **tags**: Filter by product tags
    """
    
    try:
        # Build query filters
        filters = {
            "category": category,
            "brand": brand,
            "min_price": min_price,
            "max_price": max_price,
            "search": search,
            "in_stock": in_stock,
            "tags": tags
        }
        
        # Get products from database
        products, total = await crud.get_products(
            db=db,
            page=page,
            limit=limit,
            filters=filters,
            sort=sort
        )
        
        # Get filter options for UI
        filter_options = await crud.get_filter_options(db)
        
        # Build response
        return schemas.ProductListResponse(
            products=products,
            pagination=schemas.PaginationInfo(
                total=total,
                page=page,
                limit=limit,
                pages=(total + limit - 1) // limit
            ),
            filters=filter_options
        )
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/api/v1/products/{product_id}", response_model=schemas.Product)
async def get_product(product_id: str, db: Session = Depends(get_db)):
    """Get detailed information for a single product."""
    
    product = await crud.get_product_by_id(db, product_id)
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    return product

@app.get("/api/v1/categories")
async def get_categories(db: Session = Depends(get_db)):
    """Get all available product categories."""
    
    categories = await crud.get_categories(db)
    return {"categories": categories}
```

### CRUD Operations

```python
# crud.py
from sqlalchemy.orm import Session
from sqlalchemy import or_, and_, func
from typing import List, Tuple, Optional
from . import models, schemas

async def get_products(
    db: Session,
    page: int,
    limit: int,
    filters: dict,
    sort: str
) -> Tuple[List[schemas.ProductListItem], int]:
    """Retrieve filtered and sorted products with pagination."""
    
    # Start with base query
    query = db.query(models.Product)
    
    # Apply filters
    if filters.get("category"):
        query = query.filter(models.Product.category == filters["category"])
    
    if filters.get("brand"):
        query = query.filter(models.Product.brand == filters["brand"])
    
    if filters.get("min_price"):
        query = query.filter(models.Product.regular_price >= filters["min_price"])
    
    if filters.get("max_price"):
        query = query.filter(models.Product.regular_price <= filters["max_price"])
    
    if filters.get("in_stock"):
        query = query.filter(models.Product.is_available == True)
        query = query.filter(models.Product.inventory_quantity > 0)
    
    if filters.get("search"):
        search_term = f"%{filters['search']}%"
        query = query.filter(
            or_(
                models.Product.name.ilike(search_term),
                models.Product.description.ilike(search_term)
            )
        )
    
    if filters.get("tags"):
        query = query.filter(models.Product.tags.overlap(filters["tags"]))
    
    # Apply sorting
    if sort == "price_asc":
        query = query.order_by(models.Product.regular_price.asc())
    elif sort == "price_desc":
        query = query.order_by(models.Product.regular_price.desc())
    elif sort == "rating":
        query = query.order_by(models.Product.average_rating.desc())
    elif sort == "popular":
        query = query.order_by(models.Product.review_count.desc())
    else:  # newest
        query = query.order_by(models.Product.created_at.desc())
    
    # Get total count
    total = query.count()
    
    # Apply pagination
    offset = (page - 1) * limit
    products = query.offset(offset).limit(limit).all()
    
    # Convert to list items
    product_items = [
        schemas.ProductListItem(
            product_id=p.product_id,
            name=p.name,
            price=schemas.PriceInfo(
                regular=p.regular_price,
                sale=p.sale_price,
                currency=p.currency
            ),
            primary_image=get_primary_image(p),
            rating=schemas.RatingInfo(
                average=p.average_rating,
                count=p.review_count
            ) if p.average_rating else None,
            in_stock=p.is_available and p.inventory_quantity > 0,
            badge=get_product_badge(p)
        )
        for p in products
    ]
    
    return product_items, total

def get_primary_image(product: models.Product) -> Optional[str]:
    """Get primary image URL for a product."""
    for image in product.images:
        if image.image_type == "primary":
            return image.url
    return product.images[0].url if product.images else None

def get_product_badge(product: models.Product) -> Optional[str]:
    """Determine badge to display on product card."""
    if "bestseller" in product.tags:
        return "Best Seller"
    elif "new-arrival" in product.tags:
        return "New"
    elif product.sale_price:
        discount = ((product.regular_price - product.sale_price) / product.regular_price) * 100
        if discount >= 20:
            return f"{int(discount)}% OFF"
    return None

async def get_filter_options(db: Session) -> schemas.FilterOptions:
    """Get available filter options for UI."""
    
    categories = db.query(models.Product.category).distinct().all()
    brands = db.query(models.Product.brand).distinct().all()
    
    price_stats = db.query(
        func.min(models.Product.regular_price),
        func.max(models.Product.regular_price)
    ).first()
    
    return schemas.FilterOptions(
        categories=[c[0] for c in categories if c[0]],
        brands=[b[0] for b in brands if b[0]],
        price_range={
            "min": float(price_stats[0]) if price_stats[0] else 0,
            "max": float(price_stats[1]) if price_stats[1] else 0
        }
    )
```

---

## ⚛️ Frontend Implementation

### React Component Example

```javascript
// ProductListingPage.jsx
import React, { useState, useEffect } from 'react';
import ProductGrid from './components/ProductGrid';
import ProductFilters from './components/ProductFilters';
import ProductSort from './components/ProductSort';
import Pagination from './components/Pagination';
import { fetchProducts } from './services/productService';

const ProductListingPage = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState({
    category: null,
    brand: null,
    minPrice: null,
    maxPrice: null,
    inStock: false,
    tags: []
  });
  const [sort, setSort] = useState('newest');
  const [pagination, setPagination] = useState({
    page: 1,
    limit: 24,
    total: 0,
    pages: 0
  });
  const [filterOptions, setFilterOptions] = useState({
    categories: [],
    brands: [],
    priceRange: { min: 0, max: 0 }
  });

  useEffect(() => {
    loadProducts();
  }, [filters, sort, pagination.page]);

  const loadProducts = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetchProducts({
        page: pagination.page,
        limit: pagination.limit,
        ...filters,
        sort
      });
      
      setProducts(response.products);
      setPagination(response.pagination);
      setFilterOptions(response.filters);
    } catch (err) {
      setError('Failed to load products. Please try again.');
      console.error('Error loading products:', err);
    } finally {
      setLoading(false);
    }
  };

  const handleFilterChange = (newFilters) => {
    setFilters(newFilters);
    setPagination({ ...pagination, page: 1 }); // Reset to first page
  };

  const handleSortChange = (newSort) => {
    setSort(newSort);
    setPagination({ ...pagination, page: 1 }); // Reset to first page
  };

  const handlePageChange = (newPage) => {
    setPagination({ ...pagination, page: newPage });
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  return (
    <div className="product-listing-page">
      <div className="plp-header">
        <h1>Products</h1>
        <ProductSort currentSort={sort} onSortChange={handleSortChange} />
      </div>
      
      <div className="plp-container">
        <aside className="plp-sidebar">
          <ProductFilters
            filters={filters}
            filterOptions={filterOptions}
            onFilterChange={handleFilterChange}
          />
        </aside>
        
        <main className="plp-main">
          {loading && <LoadingSpinner />}
          
          {error && <ErrorMessage message={error} />}
          
          {!loading && !error && products.length === 0 && (
            <EmptyState message="No products found matching your criteria." />
          )}
          
          {!loading && !error && products.length > 0 && (
            <>
              <ProductGrid products={products} />
              
              <Pagination
                current={pagination.page}
                total={pagination.pages}
                onPageChange={handlePageChange}
              />
            </>
          )}
        </main>
      </div>
    </div>
  );
};

export default ProductListingPage;
```

### Product Card Component

```javascript
// ProductCard.jsx
import React from 'react';
import { useCart } from '../hooks/useCart';
import { formatPrice } from '../utils/formatters';

const ProductCard = ({ product }) => {
  const { addToCart, isAdding } = useCart();
  
  const handleAddToCart = async (e) => {
    e.preventDefault();
    await addToCart(product.product_id, 1);
  };
  
  const displayPrice = product.price.sale || product.price.regular;
  const hasDiscount = product.price.sale && product.price.sale < product.price.regular;
  
  return (
    <article className="product-card">
      <a href={`/products/${product.product_id}`} className="product-card-link">
        <div className="product-image-container">
          {product.badge && (
            <span className="product-badge">{product.badge}</span>
          )}
          <img
            src={product.primary_image || '/images/placeholder.jpg'}
            alt={product.name}
            loading="lazy"
            className="product-image"
          />
        </div>
        
        <div className="product-info">
          <h3 className="product-name">{product.name}</h3>
          
          {product.rating && (
            <div className="product-rating">
              <span className="rating-stars" aria-label={`${product.rating.average} out of 5 stars`}>
                {'★'.repeat(Math.round(product.rating.average))}
                {'☆'.repeat(5 - Math.round(product.rating.average))}
              </span>
              <span className="rating-count">({product.rating.count})</span>
            </div>
          )}
          
          <div className="product-price">
            <span className="current-price">
              {formatPrice(displayPrice, product.price.currency)}
            </span>
            {hasDiscount && (
              <span className="original-price">
                {formatPrice(product.price.regular, product.price.currency)}
              </span>
            )}
          </div>
          
          {!product.in_stock && (
            <p className="out-of-stock">Out of Stock</p>
          )}
        </div>
      </a>
      
      <button
        className="add-to-cart-btn"
        onClick={handleAddToCart}
        disabled={!product.in_stock || isAdding}
        aria-label={`Add ${product.name} to cart`}
      >
        {isAdding ? 'Adding...' : 'Add to Cart'}
      </button>
    </article>
  );
};

export default ProductCard;
```

---

## 🎨 Styling Guide

### CSS Structure

```css
/* plp.css */

/* Grid Layout */
.product-listing-page {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
}

.plp-container {
  display: grid;
  grid-template-columns: 250px 1fr;
  gap: 30px;
  margin-top: 20px;
}

/* Product Grid */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

/* Product Card */
.product-card {
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.product-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.product-image-container {
  position: relative;
  padding-top: 100%; /* 1:1 aspect ratio */
  overflow: hidden;
  background: #f5f5f5;
}

.product-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.product-badge {
  position: absolute;
  top: 10px;
  left: 10px;
  background: #ff6b6b;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: bold;
  z-index: 1;
}

.product-info {
  padding: 15px;
}

.product-name {
  font-size: 16px;
  margin-bottom: 8px;
  color: #333;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.product-rating {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 8px;
  font-size: 14px;
}

.rating-stars {
  color: #ffa726;
}

.rating-count {
  color: #666;
}

.product-price {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}

.current-price {
  font-size: 20px;
  font-weight: bold;
  color: #d32f2f;
}

.original-price {
  font-size: 16px;
  color: #999;
  text-decoration: line-through;
}

.add-to-cart-btn {
  width: 100%;
  padding: 12px;
  background: #1976d2;
  color: white;
  border: none;
  border-radius: 0 0 8px 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  transition: background 0.2s;
}

.add-to-cart-btn:hover:not(:disabled) {
  background: #1565c0;
}

.add-to-cart-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* Responsive Design */
@media (max-width: 768px) {
  .plp-container {
    grid-template-columns: 1fr;
  }
  
  .plp-sidebar {
    order: 2;
  }
  
  .product-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 15px;
  }
}
```

---

## 🔧 Utility Functions

```javascript
// utils/formatters.js

export const formatPrice = (price, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency,
  }).format(price);
};

export const formatNumber = (num) => {
  if (num >= 1000000) {
    return (num / 1000000).toFixed(1) + 'M';
  }
  if (num >= 1000) {
    return (num / 1000).toFixed(1) + 'K';
  }
  return num.toString();
};

export const debounce = (func, wait) => {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
};

// utils/validators.js

export const validateFilters = (filters) => {
  const validated = { ...filters };
  
  if (validated.minPrice < 0) validated.minPrice = 0;
  if (validated.maxPrice < 0) validated.maxPrice = 0;
  if (validated.minPrice > validated.maxPrice) {
    [validated.minPrice, validated.maxPrice] = [validated.maxPrice, validated.minPrice];
  }
  
  return validated;
};
```

---

## 🚀 Performance Optimizations

### 1. Image Optimization

```javascript
// Image component with lazy loading
const OptimizedImage = ({ src, alt, ...props }) => {
  const [imageSrc, setImageSrc] = useState('/images/placeholder.jpg');
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [src]);

  return <img ref={imgRef} src={imageSrc} alt={alt} {...props} />;
};
```

### 2. API Caching

```python
from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache

@app.on_event("startup")
async def startup():
    redis = aioredis.from_url("redis://localhost")
    FastAPICache.init(RedisBackend(redis), prefix="fastapi-cache")

@app.get("/api/v1/products")
@cache(expire=300)  # Cache for 5 minutes
async def list_products(...):
    # Implementation
    pass
```

### 3. Database Query Optimization

```python
# Use eager loading to prevent N+1 queries
from sqlalchemy.orm import joinedload

query = db.query(models.Product).options(
    joinedload(models.Product.images),
    joinedload(models.Product.attributes)
)
```

---

## ✅ Testing Examples

### Backend Unit Test

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_list_products_success():
    response = client.get("/api/v1/products?page=1&limit=24")
    assert response.status_code == 200
    data = response.json()
    assert "products" in data
    assert "pagination" in data
    assert "filters" in data
    assert len(data["products"]) <= 24

def test_list_products_with_filters():
    response = client.get(
        "/api/v1/products",
        params={
            "category": "Electronics",
            "min_price": 50,
            "max_price": 200,
            "sort": "price_asc"
        }
    )
    assert response.status_code == 200
    products = response.json()["products"]
    for product in products:
        assert product["price"]["regular"] >= 50
        assert product["price"]["regular"] <= 200

def test_get_product_not_found():
    response = client.get("/api/v1/products/INVALID-ID")
    assert response.status_code == 404
```

### Frontend Component Test

```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import ProductCard from './ProductCard';

describe('ProductCard', () => {
  const mockProduct = {
    product_id: 'PROD-123',
    name: 'Test Product',
    price: { regular: 99.99, sale: null, currency: 'USD' },
    primary_image: '/test-image.jpg',
    in_stock: true,
    rating: { average: 4.5, count: 100 }
  };

  it('renders product information correctly', () => {
    render(<ProductCard product={mockProduct} />);
    
    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toBeInTheDocument();
    expect(screen.getByAltText('Test Product')).toBeInTheDocument();
  });

  it('shows sale price when available', () => {
    const productWithSale = {
      ...mockProduct,
      price: { regular: 99.99, sale: 79.99, currency: 'USD' }
    };
    
    render(<ProductCard product={productWithSale} />);
    
    expect(screen.getByText('$79.99')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toHaveClass('original-price');
  });

  it('disables add to cart when out of stock', () => {
    const outOfStockProduct = { ...mockProduct, in_stock: false };
    
    render(<ProductCard product={outOfStockProduct} />);
    
    const button = screen.getByRole('button', { name: /add to cart/i });
    expect(button).toBeDisabled();
  });
});
```

---

## 📝 Checklist for Implementation

### Backend Tasks
- [ ] Create database schema and migrations
- [ ] Implement product models (SQLAlchemy/Pydantic)
- [ ] Create API endpoints with proper validation
- [ ] Add filtering, sorting, pagination logic
- [ ] Implement caching layer (Redis)
- [ ] Add full-text search (if needed)
- [ ] Write unit tests for all endpoints
- [ ] Add API documentation (OpenAPI/Swagger)
- [ ] Optimize database queries
- [ ] Set up monitoring and logging

### Frontend Tasks
- [ ] Set up project structure and routing
- [ ] Create reusable components (Card, Grid, Filters)
- [ ] Implement state management
- [ ] Add API service layer
- [ ] Build responsive layouts
- [ ] Implement lazy loading for images
- [ ] Add error handling and loading states
- [ ] Write component tests
- [ ] Optimize bundle size
- [ ] Add accessibility features (ARIA labels, keyboard nav)

### DevOps Tasks
- [ ] Set up CI/CD pipeline
- [ ] Configure staging and production environments
- [ ] Set up CDN for static assets
- [ ] Configure Redis cache
- [ ] Set up monitoring (APM, logs)
- [ ] Configure database backups
- [ ] Set up SSL certificates
- [ ] Performance monitoring setup

---

## 🔗 Additional Resources

- [Full Analysis Document](./PLP_IMPLEMENTATION_ANALYSIS.md)
- [Quick Reference](./PLP_ANALYSIS_SUMMARY.md)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)
- [PostgreSQL Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)

---

*Last updated: November 13, 2025 by Team 3*
