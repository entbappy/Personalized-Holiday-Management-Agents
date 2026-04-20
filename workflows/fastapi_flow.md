# FastAPI Request Flow

This document details the HTTP request/response flow through the FastAPI application.

## Request Handling Pipeline

```mermaid
graph TB
    Client["HTTP Client<br/>Browser/API/Mobile"] -->|POST /plan| Request["HTTP Request"]
    
    Request --> FastAPI["FastAPI Router<br/>app.py"]
    
    FastAPI --> CORS["CORS Middleware<br/>Check origin"]
    
    CORS --> Timeout["Timeout Middleware<br/>Set timeout"]
    
    Timeout --> Route["Route Matching<br/>/plan endpoint"]
    
    Route --> Handler["Route Handler<br/>plan()"]
    
    Handler --> Depend["Dependency<br/>Injection"]
    
    Depend --> Parse["Parse Request Body"]
    
    Parse --> Validate["Pydantic Validation<br/>PlanRequest Model"]
    
    Validate --> CheckValid{Valid?}
    
    CheckValid -->|No| Error422["422 Error<br/>Unprocessable Entity"]
    CheckValid -->|Yes| Process["Process Request"]
    
    Error422 --> Response422["JSON Error Response"]
    
    Process --> Team["Invoke Agent Team<br/>RoundRobinGroupChat"]
    
    Team --> Result["Collect Results"]
    
    Result --> Format["Format Response<br/>JSON Structure"]
    
    Format --> Success200["200 Success<br/>OK"]
    
    Success200 --> Response200["JSON Response"]
    
    Response200 --> Return["Return to Client"]
    Response422 --> Return
    
    Return --> Client
    
    style Client fill:#E0FFE0
    style Request fill:#FFE4B5
    style FastAPI fill:#87CEEB
    style CORS fill:#87CEEB
    style Timeout fill:#87CEEB
    style Route fill:#87CEEB
    style Handler fill:#87CEEB
    style Depend fill:#87CEEB
    style Parse fill:#87CEEB
    style Validate fill:#DDA0DD
    style CheckValid fill:#DDA0DD
    style Error422 fill:#FFB6C6
    style Process fill:#87CEEB
    style Team fill:#DDA0DD
    style Result fill:#DDA0DD
    style Format fill:#DDA0DD
    style Success200 fill:#98FB98
    style Response200 fill:#98FB98
    style Return fill:#F0E68C
```

## Endpoint Details

### GET / Endpoint

```mermaid
graph TD
    Client["Browser Request<br/>GET /"] --> Handler["Route Handler<br/>index()"]
    
    Handler --> Template["Load Template<br/>templates/index.html"]
    
    Template --> Render["Render with Jinja2<br/>pass request context"]
    
    Render --> Response["HTMLResponse"]
    
    Response --> Browser["Render HTML<br/>in Browser"]
    
    Browser --> JS["JavaScript<br/>Event Listeners"]
    
    JS --> Form["HTML Form<br/>with Input Fields"]
    
    Form --> UI["User Interface<br/>Ready for Input"]
    
    style Client fill:#E0FFE0
    style Handler fill:#87CEEB
    style Template fill:#87CEEB
    style Render fill:#87CEEB
    style Response fill:#98FB98
    style Browser fill:#F0E68C
    style JS fill:#F0E68C
    style Form fill:#FFFACD
    style UI fill:#FFFACD
```

### POST /plan Endpoint

```mermaid
graph TD
    Client["Client Request<br/>POST /plan"] --> Body["Request Body<br/>{
      'content': 'string',
      'source': 'string'
    }"]
    
    Body --> Handler["Handler Function<br/>async def plan()"]
    
    Handler --> ModelValidate["Pydantic Model<br/>Validation<br/>PlanRequest"]
    
    ModelValidate --> Valid{Schema Valid?}
    
    Valid -->|No| MissingField["Check Errors:<br/>- Missing field?<br/>- Wrong type?<br/>- Invalid format?"]
    
    MissingField --> Return422["Return 422<br/>Validation Error"]
    
    Valid -->|Yes| Extract["Extract Fields"]
    
    Extract --> Content["content: str"]
    Extract --> Source["source: str"]
    
    Content --> TextMsg["Create TextMessage<br/>Object"]
    Source --> TextMsg
    
    TextMsg --> Try["Try-Catch Block"]
    
    Try --> Team["Call team.run()<br/>RoundRobinGroupChat"]
    
    Team --> Execute["Team Executes:<br/>1. Planner Agent<br/>2. Researcher Agent"]
    
    Execute --> Messages["Collect Messages"]
    
    Messages --> Aggregate["Aggregate Results<br/>List of Message objects"]
    
    Aggregate --> Format["Format Response:<br/>{ 'messages': [...] }"]
    
    Format --> Success["200 OK"]
    
    Try -.->|Exception| Catch["Catch Block"]
    Catch --> Exception["Log Exception"]
    Exception --> Return500["Return 500<br/>Internal Server Error"]
    
    Success --> Response["HTTP Response"]
    Return422 --> Response
    Return500 --> Response
    
    Response --> Client
    
    style Client fill:#E0FFE0
    style Body fill:#FFE4B5
    style Handler fill:#87CEEB
    style ModelValidate fill:#87CEEB
    style Valid fill:#DDA0DD
    style MissingField fill:#FFB6C6
    style Return422 fill:#FFB6C6
    style Extract fill:#87CEEB
    style Content fill:#FFFACD
    style Source fill:#FFFACD
    style TextMsg fill:#87CEEB
    style Try fill:#87CEEB
    style Team fill:#DDA0DD
    style Execute fill:#DDA0DD
    style Messages fill:#DDA0DD
    style Aggregate fill:#DDA0DD
    style Format fill:#F0E68C
    style Success fill:#98FB98
    style Catch fill:#FFB6C6
    style Exception fill:#FFB6C6
    style Return500 fill:#FFB6C6
    style Response fill:#F0E68C
```

## Request/Response Models

### PlanRequest Model

```mermaid
graph TB
    HTTPBody["HTTP Request Body<br/>JSON"] --> Model["PlanRequest<br/>Pydantic Model"]
    
    Model --> Field1["content: str<br/>required<br/>description: 'The travel request'"]
    
    Model --> Field2["source: str<br/>optional<br/>default: 'User'<br/>description: 'Source identifier'"]
    
    Field1 --> Validate1["Type Check:<br/>must be string"]
    Field2 --> Validate2["Type Check:<br/>must be string"]
    
    Validate1 --> Valid{Both Valid?}
    Validate2 --> Valid
    
    Valid -->|No| Error["Validation Error<br/>422"]
    Valid -->|Yes| Instance["PlanRequest<br/>Instance"]
    
    Error --> Return["Return Error"]
    Instance --> Use["Use in Handler"]
    
    style HTTPBody fill:#FFE4B5
    style Model fill:#87CEEB
    style Field1 fill:#FFFACD
    style Field2 fill:#FFFACD
    style Validate1 fill:#DDA0DD
    style Validate2 fill:#DDA0DD
    style Valid fill:#DDA0DD
    style Error fill:#FFB6C6
    style Return fill:#FFB6C6
    style Instance fill:#98FB98
    style Use fill:#98FB98
```

### PlanResponse Model

```mermaid
graph TB
    Data["Agent Results"] --> Format["Format Response"]
    
    Format --> Response["PlanResponse<br/>Pydantic Model"]
    
    Response --> Field["messages: List<br/>Array of Message objects"]
    
    Field --> Message["Message Object<br/>- source: str<br/>- content: str"]
    
    Message --> Ex1["Example:<br/>source: 'Holiday_Planner'<br/>content: 'Day 1: ...']
    Message --> Ex2["Example:<br/>source: 'Holidaya_Researcher'<br/>content: 'Verified: ...'"]
    
    Ex1 --> JSON["Serialize to JSON"]
    Ex2 --> JSON
    
    JSON --> Response["HTTP Response<br/>200 OK<br/>Content-Type: application/json"]
    
    Response --> Client["Send to Client"]
    
    style Data fill:#FFE4B5
    style Format fill:#87CEEB
    style Response fill:#87CEEB
    style Field fill:#FFFACD
    style Message fill:#DDA0DD
    style Ex1 fill:#FFFACD
    style Ex2 fill:#FFFACD
    style JSON fill:#F0E68C
    style Client fill:#E0FFE0
```

## Error Handling Flow

```mermaid
graph TD
    Request["Request Processing"] --> Point1["Point 1: CORS"]
    Point1 --> CORSError{CORS<br/>Failed?}
    
    CORSError -->|Yes| Error1["403 Forbidden"]
    CORSError -->|No| Point2["Point 2: Routing"]
    
    Point2 --> RouteError{Route<br/>Found?}
    RouteError -->|No| Error2["404 Not Found"]
    RouteError -->|Yes| Point3["Point 3: Validation"]
    
    Point3 --> ValidError{Request<br/>Valid?}
    ValidError -->|No| Error3["422 Unprocessable Entity"]
    ValidError -->|Yes| Point4["Point 4: Execution"]
    
    Point4 --> ExecError{Execution<br/>Error?}
    ExecError -->|No| Point5["Success"]
    ExecError -->|Yes| CatchError["Try-Catch Handler"]
    
    CatchError --> ErrorType{Error Type?}
    
    ErrorType -->|Validation| Error3
    ErrorType -->|API Error| Error4["500 Internal Server Error"]
    ErrorType -->|Other| Error4
    
    Point5 --> Response["200 OK"]
    
    Error1 --> Client["Response to Client"]
    Error2 --> Client
    Error3 --> Client
    Error4 --> Client
    Response --> Client
    
    style Request fill:#FFE4B5
    style Point1 fill:#87CEEB
    style Point2 fill:#87CEEB
    style Point3 fill:#87CEEB
    style Point4 fill:#87CEEB
    style Point5 fill:#98FB98
    style CORSError fill:#DDA0DD
    style RouteError fill:#DDA0DD
    style ValidError fill:#DDA0DD
    style ExecError fill:#DDA0DD
    style CatchError fill:#87CEEB
    style ErrorType fill:#87CEEB
    style Error1 fill:#FFB6C6
    style Error2 fill:#FFB6C6
    style Error3 fill:#FFB6C6
    style Error4 fill:#FFB6C6
    style Client fill:#E0FFE0
```

## Async Request Handling

```mermaid
graph TB
    Client1["Client 1<br/>Request"] --> Queue["Request Queue"]
    Client2["Client 2<br/>Request"] --> Queue
    Client3["Client 3<br/>Request"] --> Queue
    
    Queue --> EventLoop["Async Event Loop<br/>asyncio"]
    
    EventLoop --> Handle1["Handle Request 1<br/>await team.run()"]
    EventLoop --> Handle2["Handle Request 2<br/>await team.run()"]
    EventLoop --> Handle3["Handle Request 3<br/>await team.run()"]
    
    Handle1 --> Team1["Async Team Execution"]
    Handle2 --> Team2["Async Team Execution"]
    Handle3 --> Team3["Async Team Execution"]
    
    Team1 --> Result1["Results 1"]
    Team2 --> Result2["Results 2"]
    Team3 --> Result3["Results 3"]
    
    Result1 --> Response1["Response 1"]
    Result2 --> Response2["Response 2"]
    Result3 --> Response3["Response 3"]
    
    Response1 --> Client1Resp["Back to Client 1"]
    Response2 --> Client2Resp["Back to Client 2"]
    Response3 --> Client3Resp["Back to Client 3"]
    
    style Client1 fill:#E0FFE0
    style Client2 fill:#E0FFE0
    style Client3 fill:#E0FFE0
    style Queue fill:#FFE4B5
    style EventLoop fill:#87CEEB
    style Handle1 fill:#87CEEB
    style Handle2 fill:#87CEEB
    style Handle3 fill:#87CEEB
    style Team1 fill:#DDA0DD
    style Team2 fill:#DDA0DD
    style Team3 fill:#DDA0DD
    style Result1 fill:#98FB98
    style Result2 fill:#98FB98
    style Result3 fill:#98FB98
    style Response1 fill:#F0E68C
    style Response2 fill:#F0E68C
    style Response3 fill:#F0E68C
    style Client1Resp fill:#E0FFE0
    style Client2Resp fill:#E0FFE0
    style Client3Resp fill:#E0FFE0
```

## Middleware Stack

```mermaid
graph TB
    Request["Incoming Request"] --> CORS["CORS Middleware<br/>Check origin<br/>Add headers"]
    
    CORS --> Timeout["Timeout Middleware<br/>Set request timeout<br/>Prevent hanging"]
    
    Timeout --> Exception["Exception Middleware<br/>Catch errors<br/>Generate responses"]
    
    Exception --> Logging["Logging Middleware<br/>Log request details<br/>Log response time"]
    
    Logging --> Route["Route Processing"]
    
    Route --> Handler["Handler Function"]
    
    Handler --> Process["Process Request"]
    
    Process --> Response["Generate Response"]
    
    Response --> Return["Return through<br/>Middleware Stack"]
    
    Return --> Client["Sent to Client"]
    
    style Request fill:#FFE4B5
    style CORS fill:#87CEEB
    style Timeout fill:#87CEEB
    style Exception fill:#87CEEB
    style Logging fill:#87CEEB
    style Route fill:#DDA0DD
    style Handler fill:#DDA0DD
    style Process fill:#DDA0DD
    style Response fill:#DDA0DD
    style Return fill:#F0E68C
    style Client fill:#E0FFE0
```

## Static Files & Templates

```mermaid
graph TB
    Request["GET /"] --> Handler["index() Handler"]
    
    Handler --> StaticFiles["Static Files<br/>Mount /static"]
    Handler --> Templates["Template Directory<br/>templates/"]
    
    Templates --> Load["Load index.html"]
    
    Load --> Render["Jinja2 Render<br/>Pass context:
    {'request': Request}"]
    
    Render --> HTML["Rendered HTML<br/>Complete page"]
    
    HTML --> Browser["Browser Receives"]
    
    Browser --> Parse["Parse HTML/CSS/JS"]
    
    Parse --> Display["Display Page"]
    
    Display --> Assets["Load Assets<br/>static/styles.css"]
    
    Assets --> Ready["Ready for Input"]
    
    style Request fill:#FFE4B5
    style Handler fill:#87CEEB
    style StaticFiles fill:#87CEEB
    style Templates fill:#87CEEB
    style Load fill:#DDA0DD
    style Render fill:#DDA0DD
    style HTML fill:#98FB98
    style Browser fill:#F0E68C
    style Parse fill:#F0E68C
    style Display fill:#FFFACD
    style Assets fill:#FFFACD
    style Ready fill:#FFFACD
```

## Performance Metrics

```mermaid
graph TB
    Start["Request Start"] --> T0["T=0ms"]
    T0 --> RouteT["T=1-2ms"]
    RouteT -->|Routing| T1["T=2-3ms"]
    
    T1 -->|Validation| T2["T=3-5ms"]
    T2 -->|Processing| T3["T=5-150000ms<br/>(Agent Execution)"]
    T3 -->|Formatting| T4["T=150000-150100ms"]
    T4 -->|Sending| T5["T=150100-150200ms"]
    
    T5 --> Total["Total: ~2-3 minutes"]
    
    Total --> Breakdown["Breakdown:<br/>- Routing: 2ms<br/>- Validation: 2ms<br/>- Agent Work: 150000ms<br/>- Response: 100ms"]
    
    style Start fill:#FFE4B5
    style T0 fill:#FFE4B5
    style RouteT fill:#87CEEB
    style T1 fill:#87CEEB
    style T2 fill:#87CEEB
    style T3 fill:#DDA0DD
    style T4 fill:#DDA0DD
    style T5 fill:#F0E68C
    style Total fill:#98FB98
    style Breakdown fill:#FFFACD
```

---

For related workflows, see:
- [Overall Workflow](overall_workflow.md)
- [Data Flow](data_flow.md)
- [API Documentation](../docs/API.md)
