# API Documentation

## Overview

The Holiday Management Agent exposes a RESTful API built with FastAPI that allows users to request travel planning and receive intelligent, verified itineraries.

## Base URL

```
http://localhost:8000
```

## Endpoints

### 1. Get Home Page

Returns the web interface HTML.

**Endpoint**:
```
GET /
```

**Response**:
- **Content-Type**: `text/html`
- **Status Code**: 200

**Example**:
```bash
curl http://localhost:8000/
```

---

### 2. Plan Holiday

Create a personalized holiday itinerary based on user requirements.

**Endpoint**:
```
POST /plan
```

**Request Body**:

```json
{
  "content": "string (required)",
  "source": "string (optional, default: 'User')"
}
```

**Parameters**:

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `content` | string | Yes | The travel request with constraints | "7-day trip to Japan focusing on anime and food" |
| `source` | string | No | Source identifier for tracking | "User", "API", "Mobile" |

**Response**:

**Success (200)**:
```json
{
  "messages": [
    {
      "source": "Holiday_Planner",
      "content": "..."
    },
    {
      "source": "Holidaya_Researcher",
      "content": "..."
    }
  ]
}
```

**Error (500)**:
```json
{
  "detail": "Error message describing what went wrong"
}
```

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `messages` | array | Array of agent responses |
| `messages[].source` | string | Name of the agent (Holiday_Planner, Holidaya_Researcher) |
| `messages[].content` | string | The agent's response/output |

---

## Examples

### Example 1: Basic Trip Planning

**Request**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Plan a 5-day trip to Paris for couples interested in art and fine dining",
    "source": "User"
  }'
```

**Response**:
```json
{
  "messages": [
    {
      "source": "Holiday_Planner",
      "content": "# 5-Day Paris Romantic Getaway\n\nDay 1: Arrival & Louvre...\nDay 2: Musée d'Orsay & Latin Quarter...\nDay 3: Versailles Palace...\nDay 4: Montmartre & Art Galleries...\nDay 5: Fine Dining Experiences...\n"
    },
    {
      "source": "Holidaya_Researcher",
      "content": "## Verified Information\n\n### Day 1\n- Louvre Museum: 9 Rue de Rivoli, Hours: 9am-6pm, Entry: €17\n- Restaurant: Le Jules Verne - Michelin Star - Reservation Required\n\n### Day 2\n- Musée d'Orsay: 1 Rue de la Légion d'Honneur, Hours: 9:30am-6pm, Entry: €16\n...\n"
    }
  ]
}
```

### Example 2: Budget Travel

**Request**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "3-day budget backpacking trip to Southeast Asia, max $50 per day, interested in nature and local culture",
    "source": "Traveler"
  }'
```

**Response**:
```json
{
  "messages": [
    {
      "source": "Holiday_Planner",
      "content": "# 3-Day Southeast Asia Budget Adventure\n\nDay 1: Bangkok arrival & street food...\nDay 2: Temple visits & local markets...\nDay 3: Natural attractions...\n"
    },
    {
      "source": "Holidaya_Researcher",
      "content": "## Budget Breakdown & Verified Information\n\nDay 1:\n- Hostel: Bangkok Backpackers - $12/night\n- Street Food: $3-5 per meal\n- Temple Visit: Free entry\n- Transport: BTS Skytrain - $0.75 per trip\n\nDay 2:\n- Khao Yai National Park: Entry $3\n- Guided tour: $8-10\n...\n"
    }
  ]
}
```

### Example 3: Family Vacation

**Request**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "10-day family trip to Florida, kids aged 5 and 10, interested in theme parks and beaches, budget friendly",
    "source": "Family"
  }'
```

**Response**:
```json
{
  "messages": [
    {
      "source": "Holiday_Planner",
      "content": "# 10-Day Florida Family Adventure\n\nDay 1-3: Orlando - Theme Parks\nDay 4-5: Beach time\nDay 6-7: Wildlife experiences\n...\n"
    },
    {
      "source": "Holidaya_Researcher",
      "content": "## Family-Friendly Verified Information\n\n### Theme Parks\n- Magic Kingdom: $109-159 per person (kids discounted)\n- Hours: 9am-11pm\n- Kid-friendly rides: 25+ attractions\n...\n"
    }
  ]
}
```

---

## Error Handling

### HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful, itinerary generated |
| 422 | Unprocessable Entity | Invalid request body or missing required fields |
| 500 | Internal Server Error | Server error during processing |

### Error Response Examples

**Missing Required Field**:
```json
{
  "detail": [
    {
      "loc": ["body", "content"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

**Server Error**:
```json
{
  "detail": "An error occurred while generating the itinerary. Please try again."
}
```

---

## Authentication & Rate Limiting

### Current Implementation
- No authentication required (public API)
- No built-in rate limiting

### Production Recommendations
- Implement API key authentication
- Add rate limiting middleware:
  ```python
  from slowapi import Limiter
  from slowapi.util import get_remote_address
  
  limiter = Limiter(key_func=get_remote_address)
  app.state.limiter = limiter
  
  @app.post("/plan")
  @limiter.limit("5/minute")
  async def plan(req: PlanRequest):
      ...
  ```

---

## Request/Response Models

### PlanRequest

```python
class PlanRequest(BaseModel):
    content: str = Field(..., description="The travel request with constraints")
    source: str = Field("User", description="Source identifier for tracking")
```

**Schema**:
```json
{
  "title": "PlanRequest",
  "type": "object",
  "properties": {
    "content": {
      "title": "Content",
      "type": "string",
      "description": "The travel request with constraints"
    },
    "source": {
      "title": "Source",
      "type": "string",
      "description": "Source identifier for tracking",
      "default": "User"
    }
  },
  "required": ["content"]
}
```

### PlanResponse

```python
class Message(BaseModel):
    source: str
    content: str

class PlanResponse(BaseModel):
    messages: List[Message]
```

**Schema**:
```json
{
  "title": "PlanResponse",
  "type": "object",
  "properties": {
    "messages": {
      "title": "Messages",
      "type": "array",
      "items": {
        "title": "Message",
        "type": "object",
        "properties": {
          "source": {"type": "string"},
          "content": {"type": "string"}
        },
        "required": ["source", "content"]
      }
    }
  },
  "required": ["messages"]
}
```

---

## Integration Examples

### Python Client

```python
import requests
import json

def plan_holiday(content: str, source: str = "User"):
    url = "http://localhost:8000/plan"
    
    payload = {
        "content": content,
        "source": source
    }
    
    headers = {
        "Content-Type": "application/json"
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        return result
    else:
        raise Exception(f"Error: {response.status_code} - {response.text}")

# Usage
itinerary = plan_holiday("7-day trip to Japan for anime and food")
for message in itinerary["messages"]:
    print(f"\n{message['source']}:\n{message['content']}")
```

### JavaScript/Node.js Client

```javascript
async function planHoliday(content, source = "User") {
  const url = "http://localhost:8000/plan";
  
  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      content: content,
      source: source
    })
  });
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return await response.json();
}

// Usage
planHoliday("7-day trip to Japan for anime and food")
  .then(result => {
    result.messages.forEach(msg => {
      console.log(`\n${msg.source}:\n${msg.content}`);
    });
  })
  .catch(error => console.error("Error:", error));
```

### cURL Examples

**Basic Request**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{"content": "5-day trip to Tokyo"}'
```

**With Source**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "5-day trip to Tokyo",
    "source": "MobileApp"
  }'
```

**With Pretty JSON Output**:
```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{"content": "5-day trip to Tokyo"}' | jq .
```

---

## WebSocket API (Future)

For real-time streaming of agent responses:

```javascript
// Coming in v2.0
const ws = new WebSocket("ws://localhost:8000/ws");

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(`${message.source}: ${message.content}`);
};

ws.send(JSON.stringify({
  "content": "5-day trip to Tokyo"
}));
```

---

## GraphQL API (Future)

For flexible querying of planning data:

```graphql
# Coming in v2.0
query {
  planHoliday(input: {
    content: "5-day trip to Tokyo"
    source: "User"
  }) {
    messages {
      source
      content
    }
  }
}
```

---

## API Testing

### Using Postman

1. Set method to `POST`
2. Enter URL: `http://localhost:8000/plan`
3. Go to Headers tab, set: `Content-Type: application/json`
4. Go to Body tab (raw, JSON):
   ```json
   {
     "content": "5-day trip to Tokyo"
   }
   ```
5. Click Send

### Using Swagger UI

1. Start the application: `uvicorn app:app --reload`
2. Open browser: `http://localhost:8000/docs`
3. Find `/plan` endpoint
4. Click "Try it out"
5. Enter request body and click "Execute"

### Using ReDoc

1. Open browser: `http://localhost:8000/redoc`
2. View interactive API documentation

---

## Performance Considerations

### Response Times

| Request Type | Avg Time | Notes |
|--------------|----------|-------|
| Short trip (3 days) | 1-2 min | Simple planning |
| Medium trip (7 days) | 2-3 min | Standard itinerary |
| Complex trip (14+ days) | 3-5 min | Multiple destinations |

### Optimization Tips

1. Keep requests concise but detailed
2. Specify clear constraints (budget, interests)
3. Use the API asynchronously in production
4. Cache results for identical requests
5. Implement pagination for multiple requests

---

## Changelog

### v1.0.0 (Current)
- Initial API release
- `/` - HTML interface endpoint
- `/plan` - Holiday planning endpoint
- Pydantic model validation

### Planned Features
- Batch planning endpoint
- Saved itineraries
- User preferences storage
- PDF export
- Email delivery

---

For more information, see:
- [Architecture Guide](ARCHITECTURE.md) - System design
- [Setup Guide](SETUP.md) - Installation and configuration
- [OpenAPI Documentation](http://localhost:8000/docs) - Interactive API docs
