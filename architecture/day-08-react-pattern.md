# Day 8: ReAct Pattern Architecture

## ReAct Core Loop

```mermaid
graph TD
    START([Start with Goal]) --> THOUGHT1[💭 Thought:<br/>Analyze situation]
    THOUGHT1 --> ACTION1[🔧 Action:<br/>Choose & execute tool]
    ACTION1 --> OBS1[👁️ Observation:<br/>Receive result]

    OBS1 --> THOUGHT2[💭 Thought:<br/>Interpret result]
    THOUGHT2 --> DECIDE{🎯 Goal<br/>Achieved?}

    DECIDE -->|Not Yet| ACTION2[🔧 Action:<br/>Next step]
    ACTION2 --> OBS2[👁️ Observation]
    OBS2 --> THOUGHT3[💭 Thought]
    THOUGHT3 --> DECIDE

    DECIDE -->|Yes| FINAL[✅ Final Answer]

    style THOUGHT1 fill:#fff4e1
    style THOUGHT2 fill:#fff4e1
    style THOUGHT3 fill:#fff4e1
    style ACTION1 fill:#e1f5ff
    style ACTION2 fill:#e1f5ff
    style OBS1 fill:#f0e1ff
    style OBS2 fill:#f0e1ff
    style FINAL fill:#e1ffe1
```

## Complete ReAct Flow with LLM

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant LLM
    participant ToolRegistry
    participant Tool

    User->>Agent: "How many years ago was Python created?"

    Agent->>LLM: System Prompt + Query
    Note over LLM: Thought: I need current<br/>year and Python creation year

    LLM->>Agent: Thought + Action: get_current_date()
    Agent->>ToolRegistry: get_current_date
    ToolRegistry->>Tool: execute()
    Tool->>Agent: {"year": 2024}

    Agent->>LLM: Observation: {"year": 2024}
    Note over LLM: Thought: Need Python<br/>creation year

    LLM->>Agent: Thought + Action: search("Python created")
    Agent->>ToolRegistry: search
    ToolRegistry->>Tool: execute("Python created")
    Tool->>Agent: "Python created in 1991"

    Agent->>LLM: Observation: "Python created in 1991"
    Note over LLM: Thought: Can calculate<br/>2024 - 1991

    LLM->>Agent: Thought + Action: calculate("2024-1991")
    Agent->>ToolRegistry: calculate
    ToolRegistry->>Tool: execute("2024-1991")
    Tool->>Agent: 33

    Agent->>LLM: Observation: 33
    Note over LLM: Thought: Have answer

    LLM->>Agent: Final Answer: "Python was created 33 years ago"
    Agent->>User: "Python was created 33 years ago"
```

## ReAct State Machine

```mermaid
stateDiagram-v2
    [*] --> Thinking
    Thinking --> Acting: Select Tool
    Acting --> Observing: Execute Tool
    Observing --> Thinking: Process Result

    Thinking --> Answering: Goal Achieved
    Answering --> [*]

    Thinking --> Error: Invalid Action
    Acting --> Error: Tool Failure
    Observing --> Error: Bad Result

    Error --> Thinking: Retry
    Error --> [*]: Max Retries

    note right of Thinking
        LLM analyzes situation
        and decides next action
    end note

    note right of Acting
        Execute selected tool
        with parameters
    end note

    note right of Observing
        Receive and validate
        tool result
    end note
```

## ReAct Implementation Architecture

```mermaid
graph TB
    subgraph "ReAct Agent Components"
        direction TB

        PROMPT[Prompt Template<br/>ReAct Format]
        PARSER[Response Parser<br/>Extract Actions]
        EXECUTOR[Tool Executor]
        MEMORY[History Buffer]

        PROMPT --> LLM[LLM]
        LLM --> PARSER
        PARSER --> DECISION{Action or<br/>Answer?}

        DECISION -->|Action| EXECUTOR
        DECISION -->|Answer| FINAL[Final Response]

        EXECUTOR --> RESULT[Tool Result]
        RESULT --> MEMORY
        MEMORY --> PROMPT

        EXECUTOR --> TOOLS[(Tool<br/>Registry)]
    end

    style PROMPT fill:#fff4e1
    style EXECUTOR fill:#e1f5ff
    style RESULT fill:#f0e1ff
    style MEMORY fill:#f5e1ff
```

## ReAct Prompt Structure

```mermaid
graph LR
    subgraph "ReAct Prompt Components"
        SYS[System Instructions:<br/>You are an agent<br/>with access to tools]

        FORMAT[Format Template:<br/>Thought: ...<br/>Action: ...<br/>Observation: ...]

        TOOLS[Available Tools:<br/>- search<br/>- calculate<br/>- get_date]

        HISTORY[Conversation History:<br/>Previous Thought-Action-Obs]

        QUERY[Current Query]
    end

    SYS --> COMBINED[Combined Prompt]
    FORMAT --> COMBINED
    TOOLS --> COMBINED
    HISTORY --> COMBINED
    QUERY --> COMBINED

    COMBINED --> LLM[LLM]

    style COMBINED fill:#4a90e2
```

## Loop Detection & Management

```mermaid
graph TD
    ITER[Iteration Counter] --> CHECK1{Iterations > Max?}
    CHECK1 -->|Yes| STOP1[Stop: Max iterations]
    CHECK1 -->|No| EXECUTE[Execute Step]

    EXECUTE --> HISTORY[Add to History]
    HISTORY --> CHECK2{Same action<br/>repeated 3x?}

    CHECK2 -->|Yes| STOP2[Stop: Loop detected]
    CHECK2 -->|No| CHECK3{Same observation<br/>3x in a row?}

    CHECK3 -->|Yes| STOP3[Stop: Stuck]
    CHECK3 -->|No| CONTINUE[Continue to next iteration]

    CONTINUE --> ITER

    style CHECK1 fill:#ff6b6b
    style CHECK2 fill:#ff6b6b
    style CHECK3 fill:#ff6b6b
    style STOP1 fill:#ff4444
    style STOP2 fill:#ff4444
    style STOP3 fill:#ff4444
```

## ReAct with Multiple Tools

```mermaid
graph TB
    START[Query: Analyze competitor<br/>and create report] --> T1[💭 Need competitor data]

    T1 --> A1[🔧 web_search<br/>competitor info]
    A1 --> O1[👁️ Found website<br/>and news]

    O1 --> T2[💭 Need pricing data]
    T2 --> A2[🔧 scrape_website<br/>pricing page]
    A2 --> O2[👁️ Got pricing table]

    O2 --> T3[💭 Need to analyze]
    T3 --> A3[🔧 analyze_data<br/>compare prices]
    A3 --> O3[👁️ Analysis complete]

    O3 --> T4[💭 Need to save]
    T4 --> A4[🔧 create_document<br/>write report]
    A4 --> O4[👁️ Document created]

    O4 --> T5[💭 Goal achieved]
    T5 --> ANSWER[✅ Report ready at<br/>path/to/report.pdf]

    style T1 fill:#fff4e1
    style T2 fill:#fff4e1
    style T3 fill:#fff4e1
    style T4 fill:#fff4e1
    style T5 fill:#fff4e1
```

## Error Recovery in ReAct

```mermaid
sequenceDiagram
    participant Agent
    participant LLM
    participant Tool

    Agent->>LLM: Thought + Action
    LLM->>Tool: Execute action
    Tool-->>LLM: Error: API timeout

    LLM->>Agent: Observation: Tool failed
    Agent->>LLM: Process error

    Note over LLM: Thought: Previous approach<br/>failed, try alternative

    LLM->>Tool: Different action
    Tool-->>LLM: Success
    LLM->>Agent: Continue with result

    Note over Agent,Tool: ReAct naturally handles<br/>errors through reasoning
```

## ReAct Execution Trace

```mermaid
graph TB
    subgraph "Trace Structure"
        direction TB

        TRACE[Execution Trace]

        TRACE --> STEP1[Step 1]
        TRACE --> STEP2[Step 2]
        TRACE --> STEP3[Step 3]

        STEP1 --> S1T[Thought 1]
        STEP1 --> S1A[Action 1]
        STEP1 --> S1O[Observation 1]

        STEP2 --> S2T[Thought 2]
        STEP2 --> S2A[Action 2]
        STEP2 --> S2O[Observation 2]

        STEP3 --> S3T[Thought 3]
        STEP3 --> S3A[Action 3]
        STEP3 --> S3O[Observation 3]
    end

    TRACE --> METADATA[Metadata:<br/>- Total steps<br/>- Tools used<br/>- Token count<br/>- Duration]

    style TRACE fill:#4a90e2
    style METADATA fill:#f5a623
```

## ReAct vs Other Patterns

```mermaid
graph LR
    subgraph "Simple Chain"
        C1[Input] --> C2[LLM] --> C3[Tool] --> C4[Output]
    end

    subgraph "ReAct"
        R1[Input] --> R2[LLM Thought]
        R2 --> R3[Tool]
        R3 --> R4[Observation]
        R4 --> R2
        R2 --> R5[Output]
    end

    style C2 fill:#ff6b6b
    style R2 fill:#4ecdc4
    style R4 fill:#7ed321
```

## ReAct Performance Optimization

```mermaid
graph TD
    OPT[Optimization Strategies]

    OPT --> O1[Cache Tool Results<br/>Same inputs → cached output]
    OPT --> O2[Parallel Tool Calls<br/>Independent actions together]
    OPT --> O3[Early Stopping<br/>Detect answer sooner]
    OPT --> O4[Compress History<br/>Summarize old steps]
    OPT --> O5[Smart Tool Selection<br/>Hint relevant tools]

    O1 --> PERF[Improved<br/>Performance]
    O2 --> PERF
    O3 --> PERF
    O4 --> PERF
    O5 --> PERF

    style PERF fill:#7ed321
```

## Advanced ReAct: Self-Correction

```mermaid
sequenceDiagram
    participant Agent
    participant LLM
    participant Validator

    Agent->>LLM: Generate answer
    LLM->>Validator: Check answer
    Validator->>LLM: Issues found

    Note over LLM: Thought: My answer has<br/>issues, need to fix

    LLM->>Agent: Action: revise_answer
    Agent->>LLM: Observation: revised version
    LLM->>Validator: Check revised answer
    Validator->>LLM: Valid!
    LLM->>Agent: Final Answer
```

## Related Topics

- **Day 1**: Agent fundamentals (basic loop)
- **Day 4**: Function calling (tool execution)
- **Day 6**: Agent patterns (ReAct overview)
- **Day 9**: Memory systems (history management)
- **Day 10**: Planning (alternative pattern)
- **Day 13**: Error handling (robustness)

## ReAct Debugging

```mermaid
graph TB
    DEBUG[Debugging ReAct]

    DEBUG --> D1[Log Each Step<br/>Thought/Action/Obs]
    DEBUG --> D2[Visualize Trace<br/>Step-by-step flow]
    DEBUG --> D3[Check Loop Detection<br/>Repeated patterns]
    DEBUG --> D4[Validate Tool Calls<br/>Parameters correct?]
    DEBUG --> D5[Monitor Token Usage<br/>History size]

    D1 --> FIX[Identify<br/>Issues]
    D2 --> FIX
    D3 --> FIX
    D4 --> FIX
    D5 --> FIX

    style DEBUG fill:#4a90e2
    style FIX fill:#7ed321
```

---

**Related Days**: [Day 6](./day-06-agent-patterns.md) | [Day 9](./day-09-memory-systems.md) | [Day 10](./day-10-planning.md)
