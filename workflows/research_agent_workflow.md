# Research Agent Workflow

This document details the Holiday Researcher Agent workflow and its data validation process.

## Agent Overview

```mermaid
graph TB
    Agent["Holiday Researcher Agent<br/>AssistantAgent"]
    
    Agent --> Props["Properties<br/>- name: Holidaya_Researcher<br/>- description: Research expert<br/>- model_client: OpenAI GPT-4o<br/>- system_message: Defined prompt"]
    
    Agent --> Methods["Methods<br/>- verify_information<br/>- research_location<br/>- validate_prices<br/>- check_hours<br/>- aggregate_facts"]
    
    style Agent fill:#87CEEB
    style Props fill:#FFFACD
    style Methods fill:#F0E68C
```

## Research Process Flow

```mermaid
graph TD
    Start["Receive Skeleton Plan<br/>from Planner"] --> Parse["Parse Activities<br/>Extract Details"]
    
    Parse --> Loop["For Each Day"]
    
    Loop --> Activity["For Each Activity"]
    
    Activity --> Identify["Identify Attraction<br/>Name & Location"]
    
    Identify --> Research["Research Activity"]
    
    Research --> Verify1["Verify Address<br/>Check location accuracy"]
    Research --> Verify2["Verify Operating Hours<br/>Check if open"]
    Research --> Verify3["Verify Pricing<br/>Check current costs"]
    Research --> Verify4["Verify Availability<br/>Check if accessible"]
    
    Verify1 --> Combine["Combine Research Data"]
    Verify2 --> Combine
    Verify3 --> Combine
    Verify4 --> Combine
    
    Combine --> Store["Store Verified Facts<br/>in Dictionary"]
    
    Store --> HasNext{More Activities?}
    
    HasNext -->|Yes| Activity
    HasNext -->|No| DayCheck{More Days?}
    
    DayCheck -->|Yes| Loop
    DayCheck -->|No| Aggregate["Aggregate All Research<br/>Create comprehensive dict"]
    
    Aggregate --> Output["Output Research Data"]
    Output --> End["Return to Team<br/>for Output Generation"]
    
    style Start fill:#FFE4B5
    style Parse fill:#FFE4B5
    style Loop fill:#87CEEB
    style Activity fill:#87CEEB
    style Identify fill:#87CEEB
    style Research fill:#87CEEB
    style Verify1 fill:#DDA0DD
    style Verify2 fill:#DDA0DD
    style Verify3 fill:#DDA0DD
    style Verify4 fill:#DDA0DD
    style Combine fill:#DDA0DD
    style Store fill:#98FB98
    style Aggregate fill:#98FB98
    style Output fill:#FFB6C6
    style End fill:#FFB6C6
```

## Verification Pipeline

```mermaid
graph LR
    Input["Activity Name"] --> Step1["Address<br/>Verification"]
    Step1 --> Step2["Hours<br/>Verification"]
    Step2 --> Step3["Price<br/>Verification"]
    Step3 --> Step4["Availability<br/>Check"]
    Step4 --> Output["Verified<br/>Data"]
    
    style Input fill:#FFE4B5
    style Step1 fill:#87CEEB
    style Step2 fill:#87CEEB
    style Step3 fill:#87CEEB
    style Step4 fill:#87CEEB
    style Output fill:#98FB98
```

## Address Verification Logic

```mermaid
graph TD
    Start["Attraction Name<br/>+ Location"] --> Search["Search for Address"]
    
    Search --> Found{Found?}
    
    Found -->|No| Variations["Try Name Variations<br/>Common spellings<br/>Local names<br/>English translations"]
    
    Variations --> Found2{Found?}
    Found2 -->|No| Fallback["Mark as<br/>Requires Verification"]
    Found2 -->|Yes| Validate
    
    Found -->|Yes| Validate["Validate Address"]
    
    Validate --> Check{Seems<br/>Accurate?}
    
    Check -->|No| Research["More Research"]
    Research --> Validate
    
    Check -->|Yes| Store["Store Address<br/>with Source"]
    
    Fallback --> Store
    
    Store --> Output["Address: 123 Main St<br/>City: Tokyo<br/>Verified: Yes/Partial"]
    
    style Start fill:#FFE4B5
    style Search fill:#87CEEB
    style Found fill:#87CEEB
    style Variations fill:#DDA0DD
    style Validate fill:#87CEEB
    style Check fill:#87CEEB
    style Store fill:#98FB98
    style Output fill:#F0E68C
```

## Hours of Operation Verification

```mermaid
graph TD
    Start["Attraction Name"] --> Search["Search Operating Hours"]
    
    Search --> Current["Get Current Info"]
    
    Current --> Check["Check:
    - Open today?
    - Opening time
    - Closing time
    - Holidays/Closures
    - Seasonal changes"]
    
    Check --> Seasonal{Seasonal<br/>Hours?}
    
    Seasonal -->|Yes| Month["Check Current Month<br/>Hours"]
    Seasonal -->|No| Standard["Use Standard Hours"]
    
    Month --> Verify["Verify Against<br/>User's Trip Dates"]
    Standard --> Verify
    
    Verify --> Open{Open During<br/>Visit?}
    
    Open -->|No| Alert["Flag Warning<br/>Closed on travel date"]
    Open -->|Yes| Store["Store Hours"]
    
    Alert --> Store
    
    Store --> Output["Hours: 9am-5pm<br/>Closed: Mondays<br/>Status: Open"]
    
    style Start fill:#FFE4B5
    style Search fill:#87CEEB
    style Current fill:#87CEEB
    style Check fill:#87CEEB
    style Seasonal fill:#87CEEB
    style Verify fill:#DDA0DD
    style Open fill:#DDA0DD
    style Alert fill:#FFB6C6
    style Store fill:#98FB98
    style Output fill:#F0E68C
```

## Price Verification Logic

```mermaid
graph TD
    Start["Attraction Name"] --> Search["Search Entry Fee/Price"]
    
    Search --> Type{Price Type?}
    
    Type -->|Free| Free["Mark as Free<br/>No admission fee"]
    Type -->|Paid| Paid["Get Price Info"]
    Type -->|Variable| Variable["Get Price Range"]
    
    Paid --> Details["Get Details:
    - Adult price
    - Child price
    - Senior price
    - Combo deals
    - Advance discounts"]
    
    Variable --> Details
    
    Details --> Currency["Note Currency<br/>Convert if needed"]
    
    Currency --> Source["Check Source<br/>Official website?
    Recent update?"]
    
    Source --> Confidence{High<br/>Confidence?}
    
    Confidence -->|Yes| Approve["Approve Price"]
    Confidence -->|No| Flag["Flag for User<br/>Verification"]
    
    Free --> Store["Store as Free"]
    Approve --> Store["Store Prices"]
    Flag --> Store
    
    Store --> Output["Price: $20 USD<br/>Child: $10<br/>Source: Official<br/>Updated: 2024"]
    
    style Start fill:#FFE4B5
    style Search fill:#87CEEB
    style Type fill:#87CEEB
    style Free fill:#98FB98
    style Paid fill:#87CEEB
    style Variable fill:#87CEEB
    style Details fill:#DDA0DD
    style Currency fill:#DDA0DD
    style Source fill:#DDA0DD
    style Confidence fill:#87CEEB
    style Approve fill:#98FB98
    style Flag fill:#FFB6C6
    style Store fill:#98FB98
    style Output fill:#F0E68C
```

## Availability & Accessibility Check

```mermaid
graph TD
    Start["Attraction"] --> Check1["Accessibility<br/>Wheelchair access?
    Age restrictions?
    Health requirements?"]
    
    Check1 --> Check2["Advance Booking<br/>Required?"]
    
    Check2 --> Check3["Capacity<br/>Likely to be full?"]
    
    Check3 --> Check4["Ticket Types<br/>Available options"]
    
    Check4 --> Summary["Accessibility Summary"]
    
    Summary --> Store["Store in<br/>Recommendations"]
    
    Store --> Output["Accessibility:<br/>- Wheelchair: Yes<br/>- Advance booking: No<br/>- Best time: Off-peak<br/>- Ticket types: 3 options"]
    
    style Start fill:#FFE4B5
    style Check1 fill:#87CEEB
    style Check2 fill:#87CEEB
    style Check3 fill:#87CEEB
    style Check4 fill:#87CEEB
    style Summary fill:#DDA0DD
    style Store fill:#98FB98
    style Output fill:#F0E68C
```

## Data Quality Assessment

```mermaid
graph TB
    Data["Verified Information"] --> Quality["Quality Metrics"]
    
    Quality --> Source["Source Reliability<br/>Official: ★★★★★
    News: ★★★★☆
    Blog: ★★★☆☆"]
    
    Quality --> Recency["Recency<br/>0-3 months: ★★★★★
    3-6 months: ★★★★☆
    6+ months: ★★★☆☆"]
    
    Quality --> Coverage["Coverage<br/>Complete: ★★★★★
    Partial: ★★★☆☆
    Minimal: ★★☆☆☆"]
    
    Source --> Score["Calculate Quality Score"]
    Recency --> Score
    Coverage --> Score
    
    Score --> Level{Quality<br/>Level?}
    
    Level -->|High| High["Use as-is"]
    Level -->|Medium| Medium["Flag with confidence"]
    Level -->|Low| Low["Flag for user<br/>verification"]
    
    High --> Final["Add to Final Output<br/>Green checkmark"]
    Medium --> Final2["Add with caveat<br/>Yellow indicator"]
    Low --> Final3["Add with warning<br/>Red indicator"]
    
    Final --> Output["Final Verified Data"]
    Final2 --> Output
    Final3 --> Output
    
    style Data fill:#FFE4B5
    style Quality fill:#87CEEB
    style Source fill:#DDA0DD
    style Recency fill:#DDA0DD
    style Coverage fill:#DDA0DD
    style Score fill:#87CEEB
    style Level fill:#87CEEB
    style High fill:#98FB98
    style Medium fill:#F0E68C
    style Low fill:#FFB6C6
    style Output fill:#F0E68C
```

## Output Structure

```mermaid
graph TB
    Data["Raw Research Data"] --> Format["Structure Output"]
    
    Format --> JSON["JSON Format:<br/>
    {
      'day_1': {
        'shibuya_district': {
          'address': '...',
          'hours': '...',
          'admission': 'free'
        },
        'anime_cafe': {
          'address': '...',
          'hours': '10am-8pm',
          'price': '¥2000'
        }
      }
    }"]
    
    Format --> MD["Markdown Format:<br/>
    ## Day 1 Verified Info
    
    ### Shibuya District
    - Address: Shibuya, Tokyo
    - Hours: Open 24/7
    - Admission: Free
    
    ### Anime Cafe
    - Address: Meiji-dori
    - Hours: 10am-8pm (closed Mon)
    - Price: ¥2000 entry"]
    
    JSON --> Combine["Combine with Plan"]
    MD --> Combine
    
    Combine --> Final["Send to Output<br/>Generator"]
    
    style Data fill:#FFE4B5
    style Format fill:#87CEEB
    style JSON fill:#FFFACD
    style MD fill:#FFFACD
    style Combine fill:#DDA0DD
    style Final fill:#FFB6C6
```

## Performance Considerations

```mermaid
graph LR
    Complexity["Research Complexity"] --> Time["Processing Time"]
    
    Complexity -->|10 activities| T1["~60-90 seconds"]
    Complexity -->|20 activities| T2["~120-150 seconds"]
    Complexity -->|40+ activities| T3["~180+ seconds"]
    
    Time --> Factor["Factors:
    - Multiple web searches
    - API calls to services
    - Data aggregation
    - Quality assessment"]
    
    style Complexity fill:#FFE4B5
    style T1 fill:#98FB98
    style T2 fill:#F0E68C
    style T3 fill:#FFB6C6
    style Factor fill:#87CEEB
```

## Hallucination Prevention

```mermaid
graph TB
    Check["Hallucination Prevention"]
    
    Check --> Rule1["Never invent addresses"]
    Check --> Rule2["Never guess prices"]
    Check --> Rule3["Always note source"]
    Check --> Rule4["Flag uncertain info"]
    Check --> Rule5["Verify via multiple<br/>sources when possible"]
    
    Rule1 --> Enforce["Enforcement:
    - Search external data
    - Cross-reference sources
    - Flag unknowns
    - Don't assume"]
    
    Rule2 --> Enforce
    Rule3 --> Enforce
    Rule4 --> Enforce
    Rule5 --> Enforce
    
    Enforce --> Result["Result: Accurate,<br/>Trustworthy Itineraries"]
    
    style Check fill:#87CEEB
    style Rule1 fill:#DDA0DD
    style Rule2 fill:#DDA0DD
    style Rule3 fill:#DDA0DD
    style Rule4 fill:#DDA0DD
    style Rule5 fill:#DDA0DD
    style Enforce fill:#F0E68C
    style Result fill:#98FB98
```

---

For related workflows, see:
- [Planning Agent Workflow](planning_agent_workflow.md)
- [Overall Workflow](overall_workflow.md)
- [Data Flow](data_flow.md)
