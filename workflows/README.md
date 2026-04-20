# Workflow Diagrams

This folder contains detailed Mermaid flowchart diagrams showing the workflow and data flow through the Holiday Management Agent system.

## Quick Navigation

### System Workflows

1. **[Overall Workflow](overall_workflow.md)** - The complete end-to-end system flow
   - Main processing pipeline
   - State management
   - Team orchestration
   - Error handling
   - User journey

2. **[Planning Agent Workflow](planning_agent_workflow.md)** - How the Planner agent works
   - Agent initialization
   - Activity selection logic
   - Geographic coherence checking
   - Time allocation algorithm
   - Output formatting

3. **[Research Agent Workflow](research_agent_workflow.md)** - How the Researcher agent works
   - Research process pipeline
   - Address verification
   - Hours of operation checking
   - Price verification
   - Availability assessment
   - Hallucination prevention

4. **[FastAPI Flow](fastapi_flow.md)** - Web API request/response flow
   - Request handling pipeline
   - Endpoint details (GET / and POST /plan)
   - Request/Response models
   - Error handling
   - Middleware stack
   - Async processing

5. **[Data Flow](data_flow.md)** - Data transformations through the system
   - Complete data journey
   - State object transformations
   - LLM context window
   - Request/Response transformation
   - Memory flow
   - Parallel processing (future)

## Diagram Types

### Flowcharts
Show sequential processes and decision points using standard flowchart symbols.

Example symbols:
- **Rounded box**: Start/End
- **Rectangle**: Process
- **Diamond**: Decision
- **Parallelogram**: Input/Output

### State Diagrams
Show how data structures change as they flow through the system.

### Timeline Diagrams
Show execution timing and concurrency patterns.

### Sequence Diagrams
Show interactions between components over time.

## Color Coding

In the Mermaid diagrams, colors indicate different types of steps:

- 🟢 **Green (#98FB98)**: Success states, completed operations
- 🔵 **Blue (#87CEEB)**: Main processing steps
- 🟡 **Yellow (#FFFACD)**: Data structures, state
- 🟣 **Purple (#DDA0DD)**: Decision points, transformations
- 🟠 **Orange (#FFE4B5)**: Input/Output operations
- 🔴 **Red (#FFB6C6)**: Error states, warnings
- 🟦 **Gray-Blue (#F0E68C)**: API calls, external services

## How to Use These Diagrams

### For Understanding the System
Start with [Overall Workflow](overall_workflow.md) for a high-level view, then dive into specific component workflows.

### For Debugging
Use [Data Flow](data_flow.md) to trace how data flows through the system and identify where issues might occur.

### For Development
Use the specific agent workflows when implementing changes to agent behavior.

### For Documentation
Reference these diagrams in your project documentation to explain how the system works.

## Viewing Diagrams

### In GitHub
Mermaid diagrams render automatically in README.md files and markdown documents.

### In VS Code
Install the "Markdown Preview Mermaid Support" extension for better preview.

### Online Tools
- [Mermaid Live Editor](https://mermaid.live) - Paste diagram markdown to edit
- [Kroki.io](https://kroki.io) - Convert diagrams to PNG/SVG

## Creating Your Own Diagrams

### Basic Mermaid Syntax

```markdown
graph TD
    A["Start"] --> B["Process"]
    B --> C{Decision}
    C -->|Yes| D["End"]
    C -->|No| E["Error"]
```

### Common Diagram Types

1. **Flowchart**: `graph TD` (top-down) or `graph LR` (left-right)
2. **Sequence Diagram**: `sequenceDiagram`
3. **State Diagram**: `stateDiagram-v2`
4. **Timeline**: `timeline`
5. **Class Diagram**: `classDiagram`
6. **Entity Relationship**: `erDiagram`
7. **Gantt Chart**: `gantt`

### Resources
- [Mermaid Documentation](https://mermaid.js.org/)
- [Diagram Examples](https://mermaid.js.org/ecosystem/integrations.html)

## Workflow Components Reference

### Planning Agent
- **Input**: User request with constraints
- **Process**: Analyzes trip requirements, creates skeleton itinerary
- **Output**: Day-by-day activity list (generic)
- **Role**: Strategy and structure

### Research Agent
- **Input**: Skeleton itinerary from planner
- **Process**: Verifies all information, finds specific details
- **Output**: Verified facts dictionary with addresses, prices, hours
- **Role**: Data validation and fact-checking

### FastAPI Server
- **Input**: HTTP requests from clients
- **Process**: Routes requests, validates, orchestrates agents
- **Output**: HTTP responses with formatted itineraries
- **Role**: Web interface and API

### Chroma Database
- **Input**: Vector embeddings from processing
- **Process**: Stores and retrieves similar content
- **Output**: Context for enhanced agent responses
- **Role**: Persistent memory and similarity search

## Integration Points

The diagrams show how different components integrate:

1. **Request Flow**: Client → FastAPI → Team → Agents → OpenAI API
2. **Data Flow**: Request → Validation → Processing → Aggregation → Response
3. **State Flow**: Initial State → Agent Updates → Final State
4. **Error Flow**: Detection → Logging → Classification → Response

## Performance Considerations

Key metrics shown in diagrams:

- **Planning Time**: ~30-90 seconds (varies by complexity)
- **Research Time**: ~60-150 seconds (data verification)
- **Total Time**: ~2-3 minutes per itinerary
- **Memory Usage**: ~500MB for LLM context

## Future Enhancements

Diagrams include placeholders for planned features:

- Parallel research processing
- Multiple agent types
- Tool integration
- PDF export workflow
- Real-time streaming

## Troubleshooting Diagram Display

### Diagrams Not Rendering
- Ensure Markdown preview extension is installed
- Check Mermaid syntax in the document
- Try online editor at mermaid.live

### Unclear Layouts
- Use `graph TD` for top-down (most common)
- Use `graph LR` for left-right (data flow)
- Add explicit styling with `style` statements

### Too Many Connections
- Break into sub-diagrams
- Use subgraph to group related items
- Reference other diagrams

## Contributing Workflow Diagrams

To add or modify diagrams:

1. Edit the relevant markdown file
2. Update Mermaid syntax
3. Test in [Mermaid Live Editor](https://mermaid.live)
4. Commit and create pull request
5. Reference diagrams in main documentation

## Related Documentation

- [Architecture Guide](../docs/ARCHITECTURE.md)
- [API Documentation](../docs/API.md)
- [Setup Guide](../docs/SETUP.md)
- [Main README](../README.md)

---

Last updated: April 2026

For questions about diagrams, open an issue or see [CONTRIBUTING.md](../docs/CONTRIBUTING.md)
