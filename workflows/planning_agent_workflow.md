# Planning Agent Workflow

This document details the Holiday Planner Agent workflow and its decision-making process.

## Agent Overview

```mermaid
graph TB
    Agent["Holiday Planner Agent<br/>AssistantAgent"]
    
    Agent --> Props["Properties<br/>- name: Holiday_Planner<br/>- description: Planning expert<br/>- model_client: OpenAI GPT-4o<br/>- system_message: Defined prompt"]
    
    Agent --> Methods["Methods<br/>- on_messages<br/>- create_message<br/>- reset<br/>- process"]
    
    style Agent fill:#87CEEB
    style Props fill:#FFFACD
    style Methods fill:#F0E68C
```

## Planning Process Flow

```mermaid
graph TD
    Start["User Request Received"] --> Parse["Parse Request String"]
    
    Parse --> Extract["Extract Information<br/>- Destination(s)<br/>- Duration<br/>- Budget<br/>- Interests<br/>- Special Needs"]
    
    Extract --> Validate["Validate Constraints<br/>- Duration > 0?<br/>- Destination known?<br/>- Budget reasonable?"]
    
    Validate --> Check{Valid?}
    Check -->|No| Error["Flag for User Clarification"]
    Check -->|Yes| Analyze["Analyze Trip Requirements"]
    
    Analyze --> Strategy["Develop Strategy<br/>- Primary interests<br/>- Time distribution<br/>- Geographic logistics<br/>- Activity sequencing"]
    
    Strategy --> Create["Create Skeleton Itinerary<br/>Generic Day-by-Day Plan"]
    
    Create --> Days["Break into Daily Activities"]
    Days --> D1["Day 1:<br/>Arrival & Acclimatization"]
    Days --> D2["Day 2-N:<br/>Main Activities"]
    Days --> DN["Day N:<br/>Departure"]
    
    D1 --> Details["Add Activity Details<br/>- Time slots<br/>- Locations<br/>- Categories"]
    D2 --> Details
    DN --> Details
    
    Details --> Review["Review Plan Logic<br/>- Geographic coherence<br/>- Time feasibility<br/>- Interest alignment"]
    
    Review --> Output["Output Skeleton Plan<br/>Ready for Research"]
    
    Error --> Clarify["Request Clarification"]
    Output --> End["Pass to Researcher Agent"]
    
    style Start fill:#FFE4B5
    style Parse fill:#FFE4B5
    style Extract fill:#FFE4B5
    style Strategy fill:#87CEEB
    style Create fill:#87CEEB
    style Details fill:#87CEEB
    style Review fill:#DDA0DD
    style Output fill:#98FB98
    style End fill:#FFB6C6
```

## Decision Tree: Activity Selection

```mermaid
graph TD
    Start["Activity Needed"] --> Type{Interest Type?}
    
    Type -->|Cultural| Cultural["Museums<br/>Temples<br/>Historical Sites<br/>Local Markets"]
    Type -->|Adventure| Adventure["Hiking<br/>Water Sports<br/>Extreme Activities<br/>Nature Exploration"]
    Type -->|Food| Food["Restaurant Tours<br/>Local Cuisine<br/>Cooking Classes<br/>Food Markets"]
    Type -->|Relaxation| Relax["Spas<br/>Beaches<br/>Parks<br/>Wellness Centers"]
    Type -->|Entertainment| Entertain["Shows<br/>Nightlife<br/>Theme Parks<br/>Attractions"]
    Type -->|Other| Other["Shopping<br/>Day Trips<br/>Photography<br/>Custom"]
    
    Cultural --> TimeCheck{Time Available?}
    Adventure --> TimeCheck
    Food --> TimeCheck
    Relax --> TimeCheck
    Entertain --> TimeCheck
    Other --> TimeCheck
    
    TimeCheck -->|<2 hours| Quick["Quick Activity<br/>1-2 hours"]
    TimeCheck -->|2-4 hours| Medium["Half-Day<br/>2-4 hours"]
    TimeCheck -->|>4 hours| Full["Full-Day<br/>4+ hours"]
    
    Quick --> Add["Add to Itinerary"]
    Medium --> Add
    Full --> Add
    
    Add --> Next{More Activities?}
    Next -->|Yes| Start
    Next -->|No| Done["Plan Complete"]
    
    style Start fill:#FFE4B5
    style Type fill:#FFE4B5
    style Cultural fill:#87CEEB
    style Adventure fill:#87CEEB
    style Food fill:#87CEEB
    style Relax fill:#87CEEB
    style Entertain fill:#87CEEB
    style Other fill:#87CEEB
    style Quick fill:#DDA0DD
    style Medium fill:#DDA0DD
    style Full fill:#DDA0DD
    style Add fill:#98FB98
    style Done fill:#FFB6C6
```

## Geographic Coherence Check

```mermaid
graph TB
    Day1["Day 1 Location"] --> Geo["Geographic Analysis"]
    Day2["Day 2 Location"] --> Geo
    DayN["Day N Location"] --> Geo
    
    Geo --> Check["Check Distances<br/>- Between locations<br/>- Travel time<br/>- Transportation mode"]
    
    Check --> Dist{Distance Reasonable?}
    
    Dist -->|Too Far| Optimize["Optimize Route<br/>Reorder Days<br/>Add intermediate stops"]
    Dist -->|Acceptable| Approve["Approve Geographic Plan"]
    
    Optimize --> Recheck["Re-check Distances"]
    Recheck --> Dist
    
    Approve --> Output["Add to Final Plan"]
    
    style Day1 fill:#FFE4B5
    style Day2 fill:#FFE4B5
    style DayN fill:#FFE4B5
    style Geo fill:#87CEEB
    style Check fill:#87CEEB
    style Optimize fill:#DDA0DD
    style Approve fill:#98FB98
    style Output fill:#F0E68C
```

## Time Allocation Logic

```mermaid
graph TD
    Budget["Total Time Available<br/>e.g., 7 days"] --> Allocation["Allocate Time Blocks"]
    
    Allocation --> Travel["Calculate Travel Days<br/>- Arrival day<br/>- Departure day<br/>- Inter-city travel"]
    
    Travel --> RestDays["Add Rest/Buffer Days<br/>- Relaxation<br/>- Flexibility<br/>- Recovery"]
    
    RestDays --> Active["Remaining = Active Days"]
    
    Active --> Interest["Distribute by Interest<br/>e.g., 40% Culture<br/>30% Food<br/>20% Adventure<br/>10% Leisure"]
    
    Interest --> Schedule["Create Daily Schedule<br/>Morning: Cultural<br/>Afternoon: Food<br/>Evening: Entertainment"]
    
    Schedule --> Review["Review Balance<br/>- Not too packed?<br/>- Enough variety?<br/>- Realistic pace?"]
    
    Review --> Approve{Good Plan?}
    
    Approve -->|No| Adjust["Adjust Allocation"]
    Adjust --> Interest
    
    Approve -->|Yes| Final["Finalize Schedule"]
    
    style Budget fill:#FFE4B5
    style Allocation fill:#87CEEB
    style Travel fill:#87CEEB
    style RestDays fill:#87CEEB
    style Active fill:#FFE4B5
    style Interest fill:#DDA0DD
    style Schedule fill:#DDA0DD
    style Review fill:#DDA0DD
    style Approve fill:#F0E68C
    style Adjust fill:#FFB6C6
    style Final fill:#98FB98
```

## Output Format

```mermaid
graph TB
    Plan["Skeleton Itinerary"] --> Format["Format for Output"]
    
    Format --> MD["Markdown Format:<br/>
    # 7-Day Japan Trip
    
    ## Day 1: Tokyo Arrival
    - Morning: Arrive at Narita
    - Afternoon: Hotel check-in
    - Evening: Shibuya walk
    
    ## Day 2: Tokyo Culture
    - Morning: Senso-ji Temple
    - Afternoon: Meiji Shrine
    - Evening: Shibuya dinner
    
    ..."]
    
    MD --> Struct["Structured Data:<br/>
    {
      'day_1': {
        'theme': 'Arrival',
        'activities': [...]
      },
      'day_2': {
        'theme': 'Culture',
        'activities': [...]
      }
    }"]
    
    MD --> Next["Send to Researcher<br/>for Verification"]
    Struct --> Next
    
    style Plan fill:#FFE4B5
    style Format fill:#87CEEB
    style MD fill:#FFFACD
    style Struct fill:#FFFACD
    style Next fill:#FFB6C6
```

## Performance Considerations

```mermaid
graph LR
    Input["Trip Duration"] --> Time["Planning Time"]
    
    Input -->|3 days| T1["~30-40 seconds"]
    Input -->|7 days| T2["~60-90 seconds"]
    Input -->|14 days| T3["~120-150 seconds"]
    Input -->|21+ days| T4["~180+ seconds"]
    
    Time --> Factor["Factors Affecting<br/>- LLM response time<br/>- Complexity analysis<br/>- Route optimization"]
    
    style Input fill:#FFE4B5
    style T1 fill:#98FB98
    style T2 fill:#98FB98
    style T3 fill:#FFE4B5
    style T4 fill:#FFB6C6
    style Factor fill:#F0E68C
```

## System Message & Behavior

```mermaid
graph TB
    System["System Message<br/>'You are a Holiday planner agent...
    Your task is to help users plan their trips
    by providing information about destinations,
    itineraries, and travel tips.'"]
    
    Behavior["Defines Agent Behavior:"]
    
    Behavior --> B1["✓ Focus on planning<br/>not booking"]
    Behavior --> B2["✓ Suggest activities<br/>by interest"]
    Behavior --> B3["✓ Consider geography<br/>and logistics"]
    Behavior --> B4["✓ Provide generic<br/>information"]
    Behavior --> B5["✓ Output structured<br/>itineraries"]
    
    B1 --> Constraint1["Cannot: Make assumptions<br/>about prices or hours"]
    B2 --> Constraint2["Cannot: Book flights<br/>or hotels"]
    B3 --> Constraint3["Cannot: Verify<br/>real-time data"]
    B4 --> Constraint4["Cannot: Guarantee<br/>availability"]
    B5 --> Constraint5["Cannot: Access<br/>external APIs"]
    
    style System fill:#FFE4B5
    style Behavior fill:#87CEEB
    style B1 fill:#FFFACD
    style B2 fill:#FFFACD
    style B3 fill:#FFFACD
    style B4 fill:#FFFACD
    style B5 fill:#FFFACD
    style Constraint1 fill:#FFB6C6
    style Constraint2 fill:#FFB6C6
    style Constraint3 fill:#FFB6C6
    style Constraint4 fill:#FFB6C6
    style Constraint5 fill:#FFB6C6
```

---

For related workflows, see:
- [Research Agent Workflow](research_agent_workflow.md)
- [Overall Workflow](overall_workflow.md)
- [Data Flow](data_flow.md)
