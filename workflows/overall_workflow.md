# Overall System Workflow

This document shows the complete end-to-end workflow of the Holiday Management Agent system.

## Main System Flow

```mermaid
graph TD
    Start([User Initiates Request]) --> Web[Web Browser]
    Start --> API[API Call]
    
    Web --> Input1["HTML Form Input<br/>(Destination, Duration, Interests)"]
    API --> Input2["JSON POST Request<br/>/plan endpoint"]
    
    Input1 --> Submit["Submit Request"]
    Input2 --> Submit
    
    Submit --> FastAPI["FastAPI Router<br/>app.py"]
    FastAPI --> Validate["Validate Input<br/>Pydantic Model"]
    
    Validate --> ValidCheck{Valid?}
    ValidCheck -->|No| Error["Return 422 Error"]
    Error --> Response1["Error Response"]
    
    ValidCheck -->|Yes| TextMsg["Create TextMessage<br/>Object"]
    TextMsg --> Team["Pass to Holiday Team<br/>RoundRobinGroupChat"]
    
    Team --> Planner["Holiday Planner Agent<br/>Strategy Layer"]
    Planner --> Planning["Analyze Constraints<br/>Create Skeleton Plan"]
    Planning --> DraftPlan["Draft Itinerary Output"]
    
    DraftPlan --> Researcher["Holiday Researcher Agent<br/>Data Layer"]
    Researcher --> Research["Verify Information<br/>Find Addresses, Prices, Hours"]
    Research --> ResearchOut["Research Output<br/>Verified Facts"]
    
    ResearchOut --> Agg["Aggregate Messages"]
    Agg --> Format["Format Response<br/>JSON"]
    
    Format --> Success["Return 200 OK"]
    Success --> Response2["JSON Response<br/>with Messages"]
    
    Response1 --> Display["Display to User"]
    Response2 --> Display
    
    Display --> Render["Render Markdown<br/>Itinerary"]
    Render --> End([User Gets Itinerary])
    
    style Start fill:#90EE90
    style End fill:#FFB6C6
    style Planner fill:#87CEEB
    style Researcher fill:#87CEEB
    style Error fill:#FF6B6B
    style Success fill:#98FB98
```

## Processing Pipeline

```mermaid
graph LR
    A["Input"] --> B["Validation"] --> C["Planner"] --> D["Researcher"] --> E["Aggregation"] --> F["Output"]
    
    A -->|User Request| A
    B -->|Type Check| B
    C -->|Create Plan| C
    D -->|Verify Facts| D
    E -->|Combine Results| E
    F -->|JSON Response| F
    
    style A fill:#FFE4B5
    style B fill:#FFE4B5
    style C fill:#87CEEB
    style D fill:#87CEEB
    style E fill:#DDA0DD
    style F fill:#F0E68C
```

## State Management Flow

```mermaid
graph TD
    Init["Initialize State Object<br/>Request + Metadata"] --> State["State Contains:<br/>- content: str<br/>- draft_plan: list<br/>- research_data: dict<br/>- final_output: str"]
    
    State --> Pass1["Pass to Planner"]
    Pass1 --> Update1["Planner Updates<br/>draft_plan"]
    
    Update1 --> Pass2["Pass Updated State<br/>to Researcher"]
    Pass2 --> Update2["Researcher Updates<br/>research_data"]
    
    Update2 --> Final["Final State Ready"]
    Final --> Output["Generate Output"]
    Output --> Return["Return to User"]
    
    style Init fill:#E0FFE0
    style State fill:#FFFACD
    style Pass1 fill:#87CEEB
    style Update1 fill:#87CEEB
    style Pass2 fill:#87CEEB
    style Update2 fill:#87CEEB
    style Final fill:#98FB98
    style Output fill:#DDA0DD
    style Return fill:#FFB6C6
```

## Team Orchestration

```mermaid
graph TB
    Team["Holiday Team<br/>RoundRobinGroupChat"]
    
    Team --> Agent1["Holiday_Planner<br/>Agent Instance"]
    Team --> Agent2["Holidaya_Researcher<br/>Agent Instance"]
    
    Agent1 --> Model1["OpenAI GPT-4o<br/>Client"]
    Agent2 --> Model2["OpenAI GPT-4o<br/>Client"]
    
    Model1 --> API["OpenAI API"]
    Model2 --> API
    
    Agent1 -.->|Messages| Agent2
    Agent2 -.->|Messages| Agent1
    
    Team --> Terminator["Termination Condition<br/>Check"]
    Terminator --> Complete{Complete?}
    Complete -->|No| Team
    Complete -->|Yes| Return["Return Results"]
    
    style Team fill:#DDA0DD
    style Agent1 fill:#87CEEB
    style Agent2 fill:#87CEEB
    style Model1 fill:#F0E68C
    style Model2 fill:#F0E68C
    style API fill:#FFE4B5
    style Terminator fill:#FFB6C6
    style Return fill:#98FB98
```

## Error Handling Path

```mermaid
graph TD
    Request["Incoming Request"] --> Try["Try Execute"]
    
    Try --> Catch{Exception?}
    
    Catch -->|No Exception| Success["Process Successfully"]
    Catch -->|Validation Error| VError["Pydantic ValidationError"]
    Catch -->|API Error| APIError["OpenAI API Error"]
    Catch -->|Other Error| OError["Generic Exception"]
    
    VError --> Handle1["Extract Error Details"]
    APIError --> Handle2["Log API Error"]
    OError --> Handle3["Log Exception"]
    
    Handle1 --> Status["422 Unprocessable Entity"]
    Handle2 --> Status2["500 Internal Server Error"]
    Handle3 --> Status2
    
    Status --> Response["HTTP Error Response"]
    Status2 --> Response
    
    Success --> OK["200 OK"]
    OK --> Response2["HTTP Success Response"]
    
    Response --> Client["Send to Client"]
    Response2 --> Client
    
    style Try fill:#FFE4B5
    style Catch fill:#FFE4B5
    style VError fill:#FF6B6B
    style APIError fill:#FF6B6B
    style OError fill:#FF6B6B
    style Success fill:#98FB98
    style OK fill:#98FB98
    style Client fill:#F0E68C
```

## Concurrent vs Sequential Processing

### Current: Sequential Processing

```mermaid
timeline
    title Execution Timeline (Sequential)
    
    section Planning Phase
    T0 : Input Received
    T0-T30 : Planner Agent Processing
    T30 : Plan Complete
    
    section Research Phase
    T30 : Start Research
    T30-T90 : Researcher Agent Processing
    T90 : Research Complete
    
    section Output
    T90 : Format & Return
    T100 : User Receives Result
```

### Future: Parallel Processing

```mermaid
timeline
    title Execution Timeline (Parallel - Future)
    
    section Day-by-Day Research
    T0 : Input & Plan Complete
    T0-T30 : Day 1 Research
    T0-T30 : Day 2 Research
    T0-T30 : Day 3 Research
    T30 : All Research Complete
    
    section Output
    T30 : Format & Return
    T40 : User Receives Result
```

## User Journey

```mermaid
journey
    title Holiday Agent User Journey
    section Discovery
      Discover Holiday Agent: 5: User
      Read Documentation: 5: User
      Understand Features: 5: User
    section Onboarding
      Create Account: 4: User
      Get API Key: 3: User
      Setup Environment: 4: User
    section Usage
      Open Web Interface: 5: User
      Enter Trip Details: 5: User
      Submit Request: 5: User
      Wait for Processing: 3: User
      Review Itinerary: 5: User
    section Satisfaction
      Export/Share: 5: User
      Plan Trip: 5: User
      Recommend: 5: User
```

## Integration Points

```mermaid
graph TB
    subgraph "External Services"
        OpenAI["OpenAI API<br/>GPT-4o"]
        Future["Future APIs<br/>Booking.com, Skyscanner"]
    end
    
    subgraph "Core System"
        FastAPI["FastAPI<br/>Web Server"]
        Agents["Multi-Agent<br/>System"]
        DB["Chroma<br/>Vector DB"]
    end
    
    subgraph "Client"
        Web["Web Browser"]
        Mobile["Mobile App"]
        CLI["CLI Tools"]
    end
    
    Web --> FastAPI
    Mobile --> FastAPI
    CLI --> FastAPI
    
    FastAPI --> Agents
    Agents --> OpenAI
    Agents --> Future
    
    Agents --> DB
    DB --> Agents
    
    style OpenAI fill:#FFE4B5
    style Future fill:#FFE4B5
    style FastAPI fill:#87CEEB
    style Agents fill:#DDA0DD
    style DB fill:#F0E68C
    style Web fill:#E0FFE0
    style Mobile fill:#E0FFE0
    style CLI fill:#E0FFE0
```

---

For more details, see:
- [Planning Agent Workflow](planning_agent_workflow.md)
- [Research Agent Workflow](research_agent_workflow.md)
- [FastAPI Flow](fastapi_flow.md)
- [Data Flow](data_flow.md)
