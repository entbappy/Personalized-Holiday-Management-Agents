# Data Flow Diagram

This document shows how data flows through the entire Holiday Management Agent system.

## Complete Data Flow

```mermaid
graph TD
    User["User"] -->|Input Request| Input["Raw User Input<br/>e.g., '7-day trip to Japan<br/>for anime and food'"]
    
    Input -->|Type Check| Validate["PlanRequest Model<br/>Pydantic Validation"]
    
    Validate -->|Valid| TextMsg["TextMessage Object<br/>content: request<br/>source: 'User'"]
    
    TextMsg -->|Passed to| Team["Holiday Team<br/>RoundRobinGroupChat"]
    
    Team -->|Distribution| Planner["Planning Agent<br/>Process & LLM Call"]
    
    Planner -->|Output| DraftPlan["Draft Plan Data<br/>List of activities<br/>by day"]
    
    DraftPlan -->|Passed to| Researcher["Research Agent<br/>Process & LLM Call"]
    
    Researcher -->|Output| ResearchData["Research Data<br/>Dict of verified facts<br/>addresses, prices, hours"]
    
    DraftPlan -.->|Feed| Context["Context Window<br/>LLM Memory"]
    ResearchData -.->|Feed| Context
    
    Context -->|Generate| Final["Final Itinerary<br/>Markdown format<br/>Day-by-day plan"]
    
    Final -->|Aggregate| Messages["Message Objects<br/>source + content"]
    
    Messages -->|Format| Response["JSON Response<br/>{messages: [...]}")
    
    Response -->|Send| Client["Client/Browser"]
    
    Client -->|Render| Display["Displayed Itinerary<br/>Formatted text"]
    
    Display -->|User Action| Export["Export/Share<br/>Save as file"]
    
    style User fill:#E0FFE0
    style Input fill:#FFE4B5
    style Validate fill:#87CEEB
    style TextMsg fill:#87CEEB
    style Team fill:#DDA0DD
    style Planner fill:#87CEEB
    style DraftPlan fill:#FFFACD
    style Researcher fill:#87CEEB
    style ResearchData fill:#FFFACD
    style Context fill:#F0E68C
    style Final fill:#98FB98
    style Messages fill:#F0E68C
    style Response fill:#F0E68C
    style Client fill:#E0FFE0
    style Display fill:#FFFACD
    style Export fill:#FFFACD
```

## State Object Transformation

```mermaid
graph TB
    Start["Initial State Object"] --> State1["State V0:<br/>
    - content: request string<br/>
    - draft_plan: None<br/>
    - research_data: None<br/>
    - final_output: None"]
    
    State1 -->|Planner Agent| State2["State V1:<br/>
    - content: (unchanged)<br/>
    - draft_plan: UPDATED<br/>
    - research_data: None<br/>
    - final_output: None"]
    
    State2 -->|Researcher Agent| State3["State V2:<br/>
    - content: (unchanged)<br/>
    - draft_plan: (unchanged)<br/>
    - research_data: UPDATED<br/>
    - final_output: None"]
    
    State3 -->|Output Generator| State4["State V3:<br/>
    - content: (unchanged)<br/>
    - draft_plan: (unchanged)<br/>
    - research_data: (unchanged)<br/>
    - final_output: GENERATED"]
    
    State4 -->|Return to User| Final["Complete State<br/>All fields populated"]
    
    style Start fill:#FFE4B5
    style State1 fill:#FFFACD
    style State2 fill:#87CEEB
    style State3 fill:#87CEEB
    style State4 fill:#98FB98
    style Final fill:#F0E68C
```

## LLM Context Window Data Flow

```mermaid
graph TB
    Messages["Message History"] --> Context["LLM Context Window<br/>GPT-4o<br/>~8k tokens available"]
    
    Context --> System["System Prompt<br/>Role definition<br/>~200 tokens"]
    
    Context --> ConversationHistory["Conversation History<br/>User & Assistant msgs<br/>~3000 tokens"]
    
    Context --> CurrentInput["Current Input<br/>User request<br/>~100 tokens"]
    
    Context --> ContextData["Provided Context<br/>Plan from planner<br/>Research from researcher<br/>~3000 tokens"]
    
    System --> Processing["LLM Processing"]
    ConversationHistory --> Processing
    CurrentInput --> Processing
    ContextData --> Processing
    
    Processing --> TokenCount{"Within<br/>Limit?"}
    
    TokenCount -->|No| Summarize["Summarize/Truncate<br/>Older messages"]
    TokenCount -->|Yes| Generate["Generate Output<br/>Next token prediction"]
    
    Summarize --> Generate
    
    Generate --> Output["LLM Output<br/>Response tokens<br/>~500-2000 tokens"]
    
    Output --> Return["Return to User"]
    
    style Messages fill:#FFE4B5
    style Context fill:#87CEEB
    style System fill:#FFFACD
    style ConversationHistory fill:#FFFACD
    style CurrentInput fill:#FFFACD
    style ContextData fill:#FFFACD
    style Processing fill:#DDA0DD
    style TokenCount fill:#DDA0DD
    style Summarize fill:#FFB6C6
    style Generate fill:#98FB98
    style Output fill:#98FB98
    style Return fill:#F0E68C
```

## Agent Processing Pipeline

```mermaid
graph TD
    Input["Input Data"] --> Agent["Agent Instance<br/>AssistantAgent"]
    
    Agent --> Parse["1. Parse Input<br/>Extract meaning<br/>Understand requirements"]
    
    Parse --> LLMCall["2. Call LLM<br/>Send prompt + context<br/>Get response"]
    
    LLMCall --> OpenAI["OpenAI GPT-4o<br/>API Call<br/>Rate limited"]
    
    OpenAI --> Response["LLM Response<br/>Text output"]
    
    Response --> PostProcess["3. Post-Process<br/>Clean text<br/>Validate format"]
    
    PostProcess --> Format{Format<br/>Correct?}
    
    Format -->|No| Reformat["Reformat/Parse<br/>Extract structured data"]
    Format -->|Yes| Pass
    
    Reformat --> Pass["4. Pass Output<br/>To next component<br/>Or return to team"]
    
    Pass --> Output["Processed Output"]
    
    style Input fill:#FFE4B5
    style Agent fill:#87CEEB
    style Parse fill:#87CEEB
    style LLMCall fill:#87CEEB
    style OpenAI fill:#FFE4B5
    style Response fill:#FFFACD
    style PostProcess fill:#DDA0DD
    style Format fill:#DDA0DD
    style Reformat fill:#DDA0DD
    style Pass fill:#98FB98
    style Output fill:#F0E68C
```

## Request/Response Data Transformation

```mermaid
graph TB
    Client["Client<br/>Browser/API"]
    
    Client -->|HTTP POST| Request["HTTP Request<br/>JSON Body<br/>Content-Type: application/json<br/>
    {
      'content': 'string',
      'source': 'string'
    }"]
    
    Request -->|Parse| JsonParse["JSON Parser"]
    
    JsonParse -->|Extract| Fields["Extract Fields:<br/>- content<br/>- source"]
    
    Fields -->|Map to| PydanticModel["Pydantic PlanRequest<br/>Type: str<br/>Type: str<br/>Validate constraints"]
    
    PydanticModel -->|Validated| AppData["Application Data<br/>Python objects<br/>Type-safe"]
    
    AppData -->|Process| Team["Team Processing<br/>Agent execution<br/>LLM calls"]
    
    Team -->|Generate| Results["Results List<br/>Multiple Message objects<br/>source + content pairs"]
    
    Results -->|Structure| ResponseModel["Pydantic PlanResponse<br/>messages: List[Message]<br/>Validate schema"]
    
    ResponseModel -->|Serialize| JsonSerialize["JSON Serializer<br/>Pydantic .json()"]
    
    JsonSerialize -->|Create| JsonResponse["JSON Response<br/>Content-Type: application/json<br/>
    {
      'messages': [
        {
          'source': 'Agent',
          'content': 'text'
        }
      ]
    }"]
    
    JsonResponse -->|HTTP| Response["HTTP Response<br/>Status: 200<br/>Headers<br/>Body"]
    
    Response -->|Send| Client
    
    style Client fill:#E0FFE0
    style Request fill:#FFE4B5
    style JsonParse fill:#87CEEB
    style Fields fill:#87CEEB
    style PydanticModel fill:#87CEEB
    style AppData fill:#FFFACD
    style Team fill:#DDA0DD
    style Results fill:#FFFACD
    style ResponseModel fill:#87CEEB
    style JsonSerialize fill:#87CEEB
    style JsonResponse fill:#FFE4B5
    style Response fill:#98FB98
```

## Data Flow: Planning Phase

```mermaid
graph LR
    Input["User<br/>Request"]
    
    Input -->|Extract| Analyze["Analyze<br/>Destination<br/>Duration<br/>Interests<br/>Budget"]
    
    Analyze -->|Create| Skeleton["Skeleton<br/>Itinerary<br/>Day structure<br/>Activity list"]
    
    Skeleton -->|Format| Output["Draft Plan<br/>Markdown or<br/>JSON format"]
    
    Output -->|Pass| Next["To Research<br/>Agent"]
    
    style Input fill:#FFE4B5
    style Analyze fill:#87CEEB
    style Skeleton fill:#DDA0DD
    style Output fill:#FFFACD
    style Next fill:#FFB6C6
```

## Data Flow: Research Phase

```mermaid
graph LR
    Input["Draft Plan<br/>from Planner"]
    
    Input -->|Parse| Activities["Extract<br/>Activities<br/>Locations<br/>Names"]
    
    Activities -->|Research| Verify["Verify Info<br/>Address<br/>Hours<br/>Prices<br/>Availability"]
    
    Verify -->|Aggregate| Facts["Aggregated<br/>Facts<br/>Dictionary<br/>All verified data"]
    
    Facts -->|Format| Output["Research<br/>Output<br/>JSON or<br/>Markdown"]
    
    Output -->|Return| Team["Back to Team<br/>for aggregation"]
    
    style Input fill:#FFE4B5
    style Activities fill:#87CEEB
    style Verify fill:#87CEEB
    style Facts fill:#DDA0DD
    style Output fill:#FFFACD
    style Team fill:#FFB6C6
```

## Database Data Flow

```mermaid
graph TB
    Agents["Agent Execution"] --> VectorData["Vector Data<br/>Embeddings<br/>Similarity vectors"]
    
    VectorData -->|Store| ChromaDB["Chroma Vector DB<br/>Persistent storage<br/>SQLite backend"]
    
    ChromaDB -->|Query| Retrieval["Similarity Search<br/>Find related content<br/>Retrieve context"]
    
    Retrieval -->|Enhance| Context["Enhanced Context<br/>Feed back to agents"]
    
    Context -->|Improve| Agents
    
    Agents -->|Direct| SQLite["Optional:<br/>SQLite database<br/>Metadata storage"]
    
    SQLite -->|Persist| Cache["Cached responses<br/>Previously searched<br/>attractions"]
    
    Cache -->|Reuse| Agents
    
    style Agents fill:#87CEEB
    style VectorData fill:#FFFACD
    style ChromaDB fill:#DDA0DD
    style Retrieval fill:#DDA0DD
    style Context fill:#FFFACD
    style SQLite fill:#DDA0DD
    style Cache fill:#FFFACD
```

## Error Data Flow

```mermaid
graph TB
    Process["Processing"] --> Error{Error<br/>Occurs?}
    
    Error -->|No| Success["Continue Processing"]
    Error -->|Yes| Catch["Exception Caught"]
    
    Catch --> Log["Log Error Data:<br/>- Type<br/>- Message<br/>- Stack trace<br/>- Context"]
    
    Log --> Classify["Classify Error:<br/>- Input validation?<br/>- API error?<br/>- Network error?<br/>- Other?"]
    
    Classify --> Handle["Handle Error:<br/>- Retry?<br/>- Fallback?<br/>- Propagate?"]
    
    Handle --> Response["Error Response<br/>- Status code<br/>- Error message<br/>- Details"]
    
    Response -->|Send| Client["Return to Client"]
    
    Success -->|Continue| Client
    
    style Process fill:#FFE4B5
    style Error fill:#87CEEB
    style Success fill:#98FB98
    style Catch fill:#FFB6C6
    style Log fill:#FFB6C6
    style Classify fill:#DDA0DD
    style Handle fill:#DDA0DD
    style Response fill:#FFB6C6
    style Client fill:#E0FFE0
```

## Memory Flow During Execution

```mermaid
graph TB
    Start["Agent Starts"] --> Memory1["Memory State 1:<br/>- System prompt loaded<br/>- Empty history<br/>- Initial context"]
    
    Memory1 -->|First Input| Message1["Message 1:<br/>User request<br/>received"]
    
    Message1 -->|Store| Memory2["Memory State 2:<br/>- System prompt<br/>- Message 1 stored<br/>- Processing context"]
    
    Memory2 -->|Generate| Output1["Output 1:<br/>Agent response<br/>generated"]
    
    Output1 -->|Store| Memory3["Memory State 3:<br/>- System prompt<br/>- Message 1+2 stored<br/>- Response context"]
    
    Memory3 -->|Next Input| Message2["Message 2:<br/>Feedback/next step<br/>received"]
    
    Message2 -->|Store| Memory4["Memory State 4:<br/>- System prompt<br/>- Messages 1-3 stored<br/>- Continuation context"]
    
    Memory4 -->|Check| Limit{Token<br/>Limit?}
    
    Limit -->|Exceeded| Truncate["Truncate old<br/>messages<br/>Keep context"]
    Limit -->|OK| Continue["Continue"]
    
    Truncate --> Memory5["Memory State 5:<br/>- System prompt<br/>- Recent messages<br/>- Summary of old"]
    
    Memory5 --> Continue
    
    Continue --> Output2["Final Output:<br/>Complete<br/>response"]
    
    style Start fill:#FFE4B5
    style Memory1 fill:#FFFACD
    style Message1 fill:#FFE4B5
    style Memory2 fill:#FFFACD
    style Output1 fill:#98FB98
    style Memory3 fill:#FFFACD
    style Message2 fill:#FFE4B5
    style Memory4 fill:#FFFACD
    style Limit fill:#DDA0DD
    style Truncate fill:#FFB6C6
    style Memory5 fill:#FFFACD
    style Continue fill:#98FB98
    style Output2 fill:#98FB98
```

## Parallel Data Processing (Future)

```mermaid
graph TB
    Plan["Draft Plan<br/>Multiple Days"]
    
    Plan -->|Day 1| Research1["Research Day 1"]
    Plan -->|Day 2| Research2["Research Day 2"]
    Plan -->|Day 3| Research3["Research Day 3"]
    
    Research1 -->|Facts 1| Combine["Combine Results"]
    Research2 -->|Facts 2| Combine
    Research3 -->|Facts 3| Combine
    
    Combine -->|Aggregate| Output["Final Research<br/>Data"]
    
    style Plan fill:#FFE4B5
    style Research1 fill:#87CEEB
    style Research2 fill:#87CEEB
    style Research3 fill:#87CEEB
    style Combine fill:#DDA0DD
    style Output fill:#98FB98
```

---

For more details, see:
- [Overall Workflow](overall_workflow.md)
- [Planning Agent Workflow](planning_agent_workflow.md)
- [Research Agent Workflow](research_agent_workflow.md)
- [FastAPI Flow](fastapi_flow.md)
