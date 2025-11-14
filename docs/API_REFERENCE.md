# API Reference

This document provides detailed information about the Mergington High School Activities API endpoints.

## Base URL

- **Development**: `http://localhost:8000`
- **Codespaces**: `https://{codespace-name}-8000.app.github.dev`

## API Endpoints

### 1. Root Endpoint

**GET /** 

Redirects to the main application interface.

**Response:**
- **Status Code**: `307 Temporary Redirect`
- **Location**: `/static/index.html`

**Example:**
```bash
curl -L http://localhost:8000/
```

---

### 2. Get All Activities

**GET /activities**

Retrieves all available extracurricular activities with their details and current participants.

**Request:**
```http
GET /activities HTTP/1.1
Host: localhost:8000
```

**Response:**
```json
{
  "Chess Club": {
    "description": "Learn strategies and compete in chess tournaments",
    "schedule": "Fridays, 3:30 PM - 5:00 PM",
    "max_participants": 12,
    "participants": [
      "michael@mergington.edu",
      "daniel@mergington.edu"
    ]
  },
  "Programming Class": {
    "description": "Learn programming fundamentals and build software projects",
    "schedule": "Tuesdays and Thursdays, 3:30 PM - 4:30 PM",
    "max_participants": 20,
    "participants": [
      "emma@mergington.edu",
      "sophia@mergington.edu"
    ]
  }
}
```

**Response Fields:**
- **Activity Name** (object key): The name of the activity
  - `description` (string): What the activity is about
  - `schedule` (string): When the activity meets
  - `max_participants` (integer): Maximum number of students allowed
  - `participants` (array): List of student email addresses currently signed up

**Status Codes:**
- `200 OK`: Successfully retrieved activities

**Example:**
```bash
curl http://localhost:8000/activities
```

**Example (Python):**
```python
import requests

response = requests.get("http://localhost:8000/activities")
activities = response.json()

for name, details in activities.items():
    print(f"{name}: {len(details['participants'])}/{details['max_participants']} participants")
```

**Example (JavaScript):**
```javascript
fetch('http://localhost:8000/activities')
  .then(response => response.json())
  .then(activities => {
    console.log(activities);
  });
```

---

### 3. Sign Up for Activity

**POST /activities/{activity_name}/signup**

Allows a student to sign up for an extracurricular activity.

**Path Parameters:**
- `activity_name` (string, required): The name of the activity (URL-encoded)

**Query Parameters:**
- `email` (string, required): Student's email address

**Request:**
```http
POST /activities/Chess%20Club/signup?email=john@mergington.edu HTTP/1.1
Host: localhost:8000
```

**Success Response:**
```json
{
  "message": "Signed up john@mergington.edu for Chess Club"
}
```

**Status Codes:**
- `200 OK`: Successfully signed up
- `400 Bad Request`: Student already signed up
- `404 Not Found`: Activity does not exist

**Error Response (Already Signed Up):**
```json
{
  "detail": "Student is already signed up"
}
```

**Error Response (Activity Not Found):**
```json
{
  "detail": "Activity not found"
}
```

**Example:**
```bash
curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=jane@mergington.edu"
```

**Example (Python):**
```python
import requests

response = requests.post(
    "http://localhost:8000/activities/Chess Club/signup",
    params={"email": "jane@mergington.edu"}
)

if response.status_code == 200:
    print(response.json()["message"])
else:
    print(f"Error: {response.json()['detail']}")
```

**Example (JavaScript):**
```javascript
const activityName = 'Chess Club';
const email = 'jane@mergington.edu';

fetch(`http://localhost:8000/activities/${encodeURIComponent(activityName)}/signup?email=${email}`, {
  method: 'POST'
})
  .then(response => response.json())
  .then(data => console.log(data.message))
  .catch(error => console.error('Error:', error));
```

---

### 4. Unregister from Activity

**DELETE /activities/{activity_name}/unregister**

Allows a student to unregister from an activity they previously signed up for.

**Path Parameters:**
- `activity_name` (string, required): The name of the activity (URL-encoded)

**Query Parameters:**
- `email` (string, required): Student's email address

**Request:**
```http
DELETE /activities/Chess%20Club/unregister?email=john@mergington.edu HTTP/1.1
Host: localhost:8000
```

**Success Response:**
```json
{
  "message": "Unregistered john@mergington.edu from Chess Club"
}
```

**Status Codes:**
- `200 OK`: Successfully unregistered
- `400 Bad Request`: Student is not signed up for this activity
- `404 Not Found`: Activity does not exist

**Error Response (Not Signed Up):**
```json
{
  "detail": "Student is not signed up for this activity"
}
```

**Example:**
```bash
curl -X DELETE "http://localhost:8000/activities/Chess%20Club/unregister?email=jane@mergington.edu"
```

**Example (Python):**
```python
import requests

response = requests.delete(
    "http://localhost:8000/activities/Chess Club/unregister",
    params={"email": "jane@mergington.edu"}
)

if response.status_code == 200:
    print(response.json()["message"])
```

---

## Data Models

### Activity Object

```json
{
  "description": "string",
  "schedule": "string",
  "max_participants": "integer",
  "participants": ["string"]
}
```

**Fields:**
- `description`: Description of what the activity involves
- `schedule`: When and where the activity meets
- `max_participants`: Maximum capacity (currently not enforced)
- `participants`: Array of student email addresses

### Available Activities

The system currently includes these activities:

1. **Chess Club** - Strategy and tournaments (12 max)
2. **Programming Class** - Learn to code (20 max)
3. **Gym Class** - Physical education (30 max)
4. **Soccer Team** - School soccer team (22 max)
5. **Basketball Team** - School basketball team (15 max)
6. **Art Club** - Painting and drawing (15 max)
7. **Drama Club** - Acting and performances (20 max)
8. **Math Club** - Problem solving and competitions (10 max)
9. **Debate Team** - Public speaking and argumentation (12 max)

## Interactive API Documentation

FastAPI provides built-in interactive documentation:

### Swagger UI
Navigate to `/docs` for an interactive API explorer:
- **URL**: http://localhost:8000/docs
- **Features**: Try endpoints directly from the browser
- **Auto-generated** from code annotations

### ReDoc
Alternative documentation at `/redoc`:
- **URL**: http://localhost:8000/redoc
- **Features**: Clean, searchable documentation
- **Better for reading** and reference

## Authentication

**Current Status**: No authentication required (demo application)

**Future Considerations:**
- Student authentication (email verification)
- Admin authentication for activity management
- API keys for external integrations
- OAuth 2.0 for single sign-on

## Rate Limiting

**Current Status**: No rate limiting

**Production Recommendations:**
- Implement rate limiting (e.g., 100 requests/minute)
- Use tools like `slowapi` or `fastapi-limiter`
- Add IP-based throttling

## Error Handling

### Standard Error Format

All errors follow FastAPI's HTTPException format:

```json
{
  "detail": "Error message here"
}
```

### Common HTTP Status Codes

- `200 OK`: Successful operation
- `307 Temporary Redirect`: Root endpoint redirect
- `400 Bad Request`: Invalid request (already signed up, not registered, etc.)
- `404 Not Found`: Resource not found (activity doesn't exist)
- `422 Unprocessable Entity`: Invalid parameters
- `500 Internal Server Error`: Server error

### Example Error Handling (Python)

```python
import requests

try:
    response = requests.post(
        "http://localhost:8000/activities/Nonexistent/signup",
        params={"email": "test@mergington.edu"}
    )
    response.raise_for_status()
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 404:
        print("Activity not found")
    elif e.response.status_code == 400:
        print("Already signed up or other validation error")
```

## Data Persistence

**Storage**: JSON file (`src/activities.json`)

**Behavior:**
- Changes are immediately written to disk
- Data persists across server restarts
- No transaction support (not suitable for high concurrency)
- File-based locking not implemented

**Example activities.json structure:**
```json
{
  "Activity Name": {
    "description": "Activity description",
    "schedule": "Meeting schedule",
    "max_participants": 20,
    "participants": ["email1@example.com"]
  }
}
```

## Usage Examples

### Complete Workflow (Python)

```python
import requests

BASE_URL = "http://localhost:8000"

# 1. Get all activities
activities = requests.get(f"{BASE_URL}/activities").json()
print(f"Found {len(activities)} activities")

# 2. Find an activity with space
for name, details in activities.items():
    spaces = details['max_participants'] - len(details['participants'])
    if spaces > 0:
        print(f"{name} has {spaces} spaces available")
        chosen_activity = name
        break

# 3. Sign up for the activity
response = requests.post(
    f"{BASE_URL}/activities/{chosen_activity}/signup",
    params={"email": "newstudent@mergington.edu"}
)
print(response.json()["message"])

# 4. Verify signup
activities = requests.get(f"{BASE_URL}/activities").json()
participants = activities[chosen_activity]["participants"]
print(f"Current participants: {len(participants)}")
```

### Complete Workflow (cURL)

```bash
# Get all activities
curl http://localhost:8000/activities

# Sign up for Chess Club
curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=student@mergington.edu"

# Verify signup
curl http://localhost:8000/activities

# Unregister from Chess Club
curl -X DELETE "http://localhost:8000/activities/Chess%20Club/unregister?email=student@mergington.edu"
```

## Testing with HTTPie

[HTTPie](https://httpie.io/) provides a more user-friendly CLI:

```bash
# Get activities
http GET localhost:8000/activities

# Sign up
http POST localhost:8000/activities/"Chess Club"/signup email==student@mergington.edu

# Unregister
http DELETE localhost:8000/activities/"Chess Club"/unregister email==student@mergington.edu
```

## CORS Configuration

**Current Status**: No CORS configuration (same-origin only)

**To Enable CORS** (if needed):

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Or specific domains
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Performance Considerations

**Current Implementation:**
- File I/O on every request (read and write)
- No caching
- Synchronous operations
- Suitable for demo/learning purposes

**Production Improvements:**
- Use a proper database (PostgreSQL, MongoDB)
- Implement caching (Redis)
- Add connection pooling
- Use async database drivers
- Implement request queuing

## API Versioning

**Current**: No versioning (v1 implicit)

**Recommended for Production:**
```python
# Version in URL
app = FastAPI(title="School Activities API", version="1.0.0")

# Or use router prefixes
api_v1_router = APIRouter(prefix="/v1")
api_v2_router = APIRouter(prefix="/v2")
```

## Future Enhancements

Potential API additions:
- **GET /activities/{name}**: Get single activity details
- **POST /activities**: Create new activity (admin only)
- **PUT /activities/{name}**: Update activity details
- **DELETE /activities/{name}**: Remove activity
- **GET /students/{email}**: Get student's activities
- **POST /activities/{name}/waitlist**: Join waitlist when full
- **GET /activities/search**: Search activities by criteria

---

**For more information:**
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
