# Day 4: Function Calling & Tool Use Architecture

## Function Calling Flow

### Complete Function Calling Process

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant LLM
    participant ToolRegistry
    participant Tool
    participant ExternalAPI

    User->>Agent: "What's the weather in Tokyo?"
    Agent->>LLM: Analyze query + tool schemas
    LLM->>Agent: function_call: get_weather(location="Tokyo")
    Agent->>ToolRegistry: Lookup "get_weather"
    ToolRegistry->>Agent: Tool function reference
    Agent->>Tool: Execute(location="Tokyo")
    Tool->>ExternalAPI: HTTP GET /weather?city=Tokyo
    ExternalAPI->>Tool: {temp: 22, condition: "Sunny"}
    Tool->>Agent: Result
    Agent->>LLM: Format with context
    LLM->>Agent: "It's 22°C and sunny in Tokyo"
    Agent->>User: Final response

    Note over LLM,Tool: LLM decides when<br/>to use tools
```

## Tool Registry Architecture

```mermaid
graph TB
    subgraph "Tool Registry System"
        TR[Tool Registry]

        TR --> T1[Web Search Tool]
        TR --> T2[Calculator Tool]
        TR --> T3[File System Tool]
        TR --> T4[Database Tool]
        TR --> T5[API Tool]

        T1 --> T1A[Schema:<br/>name, description,<br/>parameters]
        T1 --> T1B[Function:<br/>Executable code]
        T1 --> T1C[Validation:<br/>Input validation]

        T2 --> T2A[Schema]
        T2 --> T2B[Function]
        T2 --> T2C[Validation]
    end

    LLM[LLM] -->|Get available tools| TR
    TR -->|Return schemas| LLM
    LLM -->|Select tool| TR
    TR -->|Execute| Result[Tool Result]

    style TR fill:#4a90e2
    style T1 fill:#f5a623
    style T2 fill:#7ed321
    style T3 fill:#bd10e0
```

## Tool Schema Structure

```mermaid
classDiagram
    class Tool {
        +String name
        +String description
        +ParameterSchema parameters
        +Function function
        +execute(args) Result
        +validate(args) Boolean
    }

    class ParameterSchema {
        +String type
        +Map~String,Property~ properties
        +List~String~ required
        +validate() Boolean
    }

    class Property {
        +String type
        +String description
        +List~String~ enum
        +Any default
    }

    class ToolRegistry {
        +Map~String,Tool~ tools
        +register(tool) void
        +get(name) Tool
        +list() List~Tool~
        +execute(name, args) Result
    }

    Tool --> ParameterSchema
    ParameterSchema --> Property
    ToolRegistry --> Tool
```

## Multi-Tool Execution Flow

```mermaid
graph TD
    A[User Query:<br/>"Research Python and<br/>calculate market size"] --> B[LLM Analysis]
    B --> C{Plan Required Tools}

    C --> D1[Tool 1: Web Search<br/>search Python market]
    C --> D2[Tool 2: Extract Data<br/>parse results]
    C --> D3[Tool 3: Calculate<br/>compute statistics]

    D1 --> E1[Result 1:<br/>Market data]
    D2 --> E2[Result 2:<br/>Structured data]
    D3 --> E3[Result 3:<br/>Calculations]

    E1 --> F[Combine Results]
    E2 --> F
    E3 --> F

    F --> G[LLM Synthesis]
    G --> H[Final Response]

    style B fill:#4a90e2
    style C fill:#f5a623
    style F fill:#7ed321
    style G fill:#bd10e0
```

## Tool Execution States

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Validating: Tool Selected
    Validating --> Executing: Valid Parameters
    Validating --> Error: Invalid Parameters
    Executing --> Success: Execution OK
    Executing --> Retrying: Transient Error
    Executing --> Error: Fatal Error
    Retrying --> Executing: Retry Attempt
    Retrying --> Error: Max Retries
    Success --> [*]
    Error --> [*]

    note right of Validating
        Check parameter types,
        required fields,
        constraints
    end note

    note right of Retrying
        Exponential backoff,
        max 3 attempts
    end note
```

## Tool Communication Patterns

### Synchronous Tool Execution

```mermaid
sequenceDiagram
    participant Agent
    participant Tool
    participant API

    Agent->>Tool: execute(params)
    activate Tool
    Tool->>API: Request
    activate API
    API-->>Tool: Response
    deactivate API
    Tool-->>Agent: Result
    deactivate Tool

    Note over Agent,Tool: Blocking call<br/>Agent waits for result
```

### Asynchronous Tool Execution

```mermaid
sequenceDiagram
    participant Agent
    participant Tool1
    participant Tool2
    participant Tool3

    par Parallel Execution
        Agent->>Tool1: async execute()
    and
        Agent->>Tool2: async execute()
    and
        Agent->>Tool3: async execute()
    end

    Tool1-->>Agent: Result 1
    Tool2-->>Agent: Result 2
    Tool3-->>Agent: Result 3

    Agent->>Agent: Combine results

    Note over Agent,Tool3: Non-blocking<br/>Parallel execution
```

## Error Handling Architecture

```mermaid
graph TB
    A[Tool Execution] --> B{Try Execute}

    B -->|Success| C[Return Result]
    B -->|Error| D{Error Type?}

    D -->|Validation| E[Return Validation Error]
    D -->|Network| F{Retry?}
    D -->|Permission| G[Return Permission Error]
    D -->|Timeout| H{Retry?}
    D -->|Unknown| I[Return Generic Error]

    F -->|Yes| J[Exponential Backoff]
    F -->|No| K[Return Network Error]
    H -->|Yes| J
    H -->|No| L[Return Timeout Error]

    J --> M{Attempts < Max?}
    M -->|Yes| B
    M -->|No| N[Return Max Retries Error]

    style B fill:#4a90e2
    style D fill:#f5a623
    style F fill:#ffcc00
    style H fill:#ffcc00
```

## Tool Library Organization

```mermaid
graph TB
    subgraph "Tool Library"
        direction TB

        CAT1[Search Tools]
        CAT2[Data Tools]
        CAT3[File Tools]
        CAT4[Communication Tools]
        CAT5[Computation Tools]

        CAT1 --> T1A[Web Search]
        CAT1 --> T1B[Knowledge Base Search]
        CAT1 --> T1C[Vector Search]

        CAT2 --> T2A[Database Query]
        CAT2 --> T2B[API Call]
        CAT2 --> T2C[Data Transform]

        CAT3 --> T3A[Read File]
        CAT3 --> T3B[Write File]
        CAT3 --> T3C[List Directory]

        CAT4 --> T4A[Send Email]
        CAT4 --> T4B[Post Slack]
        CAT4 --> T4C[Create Ticket]

        CAT5 --> T5A[Calculator]
        CAT5 --> T5B[Data Analysis]
        CAT5 --> T5C[Code Execution]
    end

    style CAT1 fill:#ff6b6b
    style CAT2 fill:#4ecdc4
    style CAT3 fill:#45b7d1
    style CAT4 fill:#f9ca24
    style CAT5 fill:#6c5ce7
```

## Tool Chain Example

```mermaid
graph LR
    A[User: Analyze<br/>competitor pricing] --> B[Search Tool]
    B --> C[Scrape Tool]
    C --> D[Parse Tool]
    D --> E[Analyze Tool]
    E --> F[Visualize Tool]
    F --> G[Report Tool]
    G --> H[Final Report]

    B -.->|Data| C
    C -.->|HTML| D
    D -.->|Structured| E
    E -.->|Stats| F
    F -.->|Charts| G

    style A fill:#e1f5ff
    style H fill:#e1ffe1
```

## Function Calling Standards

```mermaid
graph TB
    subgraph "OpenAI Function Calling"
        O1[Tool Schema<br/>JSON Schema]
        O2[Function Call<br/>Structured Output]
        O3[Tool Response<br/>String/JSON]
    end

    subgraph "Anthropic Tool Use"
        A1[Tool Definition<br/>Input Schema]
        A2[Tool Use Block<br/>ID + Input]
        A3[Tool Result<br/>ID + Content]
    end

    subgraph "Agent Implementation"
        I1[Unified Tool Interface]
        I2[Adapter Pattern]
        I3[Execution Engine]
    end

    O1 --> I1
    O2 --> I2
    O3 --> I3

    A1 --> I1
    A2 --> I2
    A3 --> I3

    style I1 fill:#4a90e2
    style I2 fill:#f5a623
    style I3 fill:#7ed321
```

## Related Topics

- **Day 1**: Agent fundamentals (why tools matter)
- **Day 2**: LLM APIs (how LLMs call functions)
- **Day 5**: Building first agent (practical implementation)
- **Day 6**: Agent patterns (tool orchestration)
- **Day 13**: Error handling (robust tool execution)

## Best Practices

```mermaid
mindmap
    root((Tool Design))
        Single Responsibility
            One clear purpose
            Well-defined scope
            Minimal side effects
        Clear Interface
            Descriptive names
            Good documentation
            Type hints
        Robust Execution
            Input validation
            Error handling
            Timeout management
        Idempotency
            Safe to retry
            No duplicate effects
            Deterministic
        Observability
            Logging
            Metrics
            Tracing
```

---

**Related Days**: [Day 1](./day-01-agent-fundamentals.md) | [Day 5](./day-05-first-agent.md) | [Day 6](./day-06-agent-patterns.md)
