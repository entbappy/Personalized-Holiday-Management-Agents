# System Architecture

## Overview

The Personalized Holiday Management Agent follows a **Sequential Multi-Agent Pattern** where specialized AI agents work together in a coordinated pipeline to transform user travel requests into verified, day-by-day itineraries.

## Architecture Diagram

```
┌─────────────────┐
│   User Input    │
│  (Constraints)  │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│   State Initialization              │
│  - request: str                     │
│  - draft_plan: list                 │
│  - research_data: dict              │
│  - final_output: str                │
└────────┬────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│  Holiday Planner Agent (Strategy Layer)      │
│  - Analyzes user constraints                 │
│  - Creates skeleton itinerary                │
│  - Focuses on geography & logistics          │
│  - Output: Generic activities list           │
└────────┬─────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│  Holiday Researcher Agent (Data Layer)       │
│  - Takes skeleton itinerary                  │
│  - Performs targeted verification            │
│  - Finds addresses, prices, hours            │
│  - Output: Structured facts dictionary       │
└────────┬─────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│   Final Itinerary (Markdown)            │
│   - Day-by-day activities                │
│   - Verified information                 │
│   - Ready for export & use               │
└──────────────────────────────────────────┘
```

## Component Architecture

```
Personalized-Holiday-Management-Agent/
│
├── FastAPI Application (app.py)
│   └── REST API endpoints
│       └── /plan - POST endpoint for planning
│
├── holiday_management/ (Main Package)
│   │
│   ├── agents/ (AI Logic)
│   │   ├── planner.py - Strategy agent
│   │   └── researcher.py - Data validation agent
│   │
│   ├── teams/ (Orchestration)
│   │   └── holiday_team.py - Multi-agent coordination
│   │
│   ├── models/ (LLM Integration)
│   │   └── gpt_model.py - OpenAI model client
│   │
│   ├── config/ (Settings)
│   │   └── settings.py - Configuration
│   │
│   └── utils/ (Helpers)
│       └── utils.py - Utility functions
│
├── static/ (Frontend Assets)
│   └── styles.css - CSS styling
│
├── templates/ (HTML)
│   └── index.html - Web interface
│
└── db/ (Vector Database)
    └── chroma.sqlite3 - Chroma vector store
```

## Agent Specifications

### 1. Holiday Planner Agent

**Role**: Strategic Planning and Itinerary Structure

**Responsibilities**:
- Analyze user travel constraints (budget, duration, interests)
- Create a skeleton itinerary with day-by-day structure
- Focus on geography and logistics (minimize travel time)
- Suggest generic activities (e.g., "Visit temple district")
- Ensure activities are geographically coherent

**Input**:
- User request string with travel constraints

**Output**:
- List of planned activities organized by day
- High-level itinerary structure

**System Message**:
```
You are a Holiday planner agent. Your task is to help users plan their trips 
by providing information about destinations, itineraries, and travel tips.
```

### 2. Holiday Researcher Agent

**Role**: Data Validation and Fact-Checking

**Responsibilities**:
- Verify all planned activities from the Planner
- Research specific attraction details (addresses, hours, prices)
- Check current availability and seasons
- Validate transportation options
- Ensure information accuracy

**Input**:
- Skeleton itinerary from Planner agent

**Output**:
- Structured dictionary with verified facts
- Real addresses, prices, and operating hours
- Alternative suggestions if needed

**System Message**:
```
You are a Holiday researcher agent. Your task is to help users research about 
their holiday destinations by providing information about attractions, 
local culture, and travel tips.
```

## Communication Flow

### 1. Request Phase
```
User → FastAPI /plan endpoint → PlanRequest object
```

### 2. State Management
```
State object holds:
├── content: Original user request
├── draft_plan: Planner's output
├── research_data: Researcher's output
└── final_output: Generated itinerary
```

### 3. Agent Coordination
```
FastAPI
  ↓
TextMessage wrapper
  ↓
Team (RoundRobinGroupChat)
  ├─ Planner Agent → processes
  └─ Researcher Agent → processes
  ↓
Result messages aggregated
  ↓
Response returned to user
```

## Technology Stack Deep Dive

### AutoGen Framework
- **Purpose**: Multi-agent orchestration and LLM conversation management
- **Key Class**: `RoundRobinGroupChat` - coordinates agent interactions
- **Features**:
  - Automatic message routing
  - Conversation history management
  - Termination conditions

### FastAPI
- **Purpose**: Web API and routing
- **Key Features**:
  - Async request handling
  - Automatic OpenAPI documentation
  - Jinja2 template rendering
  - Static file serving

### Pydantic
- **Purpose**: Data validation and type checking
- **Usage**: 
  - `PlanRequest` model validation
  - Agent response parsing
  - Type safety throughout application

### OpenAI GPT-4o
- **Purpose**: Advanced reasoning and natural language understanding
- **Used for**: 
  - Planning complex itineraries
  - Understanding user constraints
  - Generating structured responses

## Data Flow Example

```
User Input: "7-day trip to Japan for anime and food"
           │
           ▼
PlanRequest(content="7-day trip to Japan...")
           │
           ▼
TextMessage wrapper
           │
           ▼
Holiday Planner Agent
  System: "You are a Holiday planner agent..."
  User: "7-day trip to Japan for anime and food"
           │
           ▼
Draft Plan Output:
  Day 1: Tokyo - Shibuya district, anime shops
  Day 2: Tokyo - Tsukiji market, food tours
  Day 3: Kyoto - Temple district
  ... etc
           │
           ▼
Holiday Researcher Agent
  System: "You are a Holiday researcher agent..."
  Input: Draft plan from above
           │
           ▼
Research Output:
  {
    "day_1": {
      "shibuya_district": "Address: XYZ, Hours: 10am-10pm",
      "anime_shops": ["Shop A - Price: ¥5000", "Shop B - Price: ¥3000"]
    },
    ...
  }
           │
           ▼
Final Markdown Itinerary
```

## Scalability Considerations

### Current Architecture
- Sequential processing (Planner → Researcher)
- Single threaded agent workflow
- Suitable for moderate load

### Future Enhancements

1. **Parallel Research**
   - Research multiple days simultaneously
   - Reduces latency from O(n) to O(1)

2. **Tool Integration**
   - Direct API connections (Booking.com, Skyscanner)
   - Real-time pricing and availability

3. **Caching Layer**
   - Redis for frequently searched destinations
   - Vector similarity search for similar trips

4. **Database Optimization**
   - Separate read replicas
   - Indexed queries for common searches

## Error Handling

```
Request Processing:
  ├─ Input validation (Pydantic)
  ├─ Agent execution (try-catch)
  ├─ Message formatting
  └─ Response aggregation
     └─ Exception → HTTPException 500
```

## Security Considerations

1. **API Key Management**
   - Environment variables via `python-dotenv`
   - Never commit `.env` file

2. **Input Validation**
   - Pydantic models prevent injection attacks
   - Type checking on all inputs

3. **Rate Limiting**
   - OpenAI API rate limits enforced
   - Implement request throttling for production

4. **CORS**
   - Configure for trusted domains only
   - Implement in production deployment

## Performance Metrics

| Operation | Time | Notes |
|-----------|------|-------|
| Planning | ~30-60s | Depends on itinerary complexity |
| Research | ~60-120s | Multiple verification requests |
| Total | ~2-3 min | Per itinerary generation |
| Memory | ~500MB | LLM context + agent state |

## Integration Points

### External APIs
- **OpenAI**: GPT-4o model calls
- **Potential**: Booking.com, Skyscanner, Google Maps

### Database
- **Chroma**: Vector storage for embeddings
- **SQLite**: Lightweight persistence

### Frontend
- **Jinja2**: Template rendering
- **HTML/CSS**: Web interface

## Configuration Management

```
Settings hierarchy:
  1. Environment variables (.env)
  2. Config files (config/settings.py)
  3. Default values (in code)
  4. Runtime overrides
```

## Monitoring & Logging

Current setup:
- Console output for agent conversations
- FastAPI request logging
- Exception tracking via HTTPException

Production recommendations:
- Structured logging (JSON format)
- Centralized log aggregation (ELK, Datadog)
- Performance monitoring (APM)
- Error tracking (Sentry)

## Deployment Architecture

```
Production Deployment:
  ┌─────────────────┐
  │  Load Balancer  │
  └────────┬────────┘
           │
  ┌────────┴────────┐
  │                 │
  ▼                 ▼
┌──────────────┐ ┌──────────────┐
│ Uvicorn 1    │ │ Uvicorn 2    │
│ (Instance 1) │ │ (Instance 2) │
└──────┬───────┘ └───────┬──────┘
       │                 │
       │   ┌─────────────┘
       │   │
       └───┤
           │
           ▼
    ┌─────────────────┐
    │  Database       │
    │ (Chroma/SQLite) │
    └─────────────────┘
```

---

For more detailed information, see:
- [Setup Guide](SETUP.md) - Installation and configuration
- [API Documentation](API.md) - REST API endpoints
- [Workflow Diagrams](../workflows/) - Detailed Mermaid diagrams
