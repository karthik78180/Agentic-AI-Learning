# Day 6: Agent Architecture Patterns

## Agent Pattern Comparison

```mermaid
graph TB
    subgraph "Agent Patterns"
        A[User Request]

        A --> P1[Simple Reflex]
        A --> P2[Model-Based]
        A --> P3[Goal-Based ReAct]
        A --> P4[Utility-Based]
        A --> P5[Hierarchical]

        P1 --> R1[Direct Response]
        P2 --> R2[Stateful Response]
        P3 --> R3[Planned Actions]
        P4 --> R4[Optimized Choice]
        P5 --> R5[Delegated Tasks]
    end

    style P1 fill:#ff6b6b
    style P2 fill:#4ecdc4
    style P3 fill:#45b7d1
    style P4 fill:#f9ca24
    style P5 fill:#6c5ce7
```

## 1. Simple Reflex Agent

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant LLM

    User->>Agent: Query
    Agent->>LLM: Process
    LLM->>Agent: Response
    Agent->>User: Output

    Note over Agent: No memory<br/>No planning<br/>Direct mapping
```

**Architecture:**

```mermaid
graph LR
    INPUT[Input] --> RULES[Condition-Action<br/>Rules]
    RULES --> LLM[LLM]
    LLM --> OUTPUT[Output]

    style RULES fill:#ff6b6b
```

## 2. Model-Based Agent

```mermaid
graph TB
    subgraph "Model-Based Agent"
        INPUT[User Input]
        STATE[Internal State<br/>Model]
        MEMORY[Memory]
        LLM[LLM Reasoning]
        OUTPUT[Action/Response]

        INPUT --> STATE
        MEMORY <--> STATE
        STATE --> LLM
        LLM --> OUTPUT
        OUTPUT --> UPDATE[Update State]
        UPDATE --> STATE
    end

    style STATE fill:#4ecdc4
    style MEMORY fill:#7ed321
```

**State Flow:**

```mermaid
stateDiagram-v2
    [*] --> Initialize
    Initialize --> ReadState: Load Memory
    ReadState --> Process: User Input
    Process --> UpdateState: LLM Response
    UpdateState --> SaveState: Persist
    SaveState --> Output
    Output --> ReadState: Next Input
    Output --> [*]: End Session
```

## 3. ReAct (Reasoning + Acting) Pattern

```mermaid
graph TD
    START[Goal] --> THINK1[Thought:<br/>What do I need?]
    THINK1 --> ACT1[Action:<br/>Use Tool]
    ACT1 --> OBS1[Observation:<br/>Tool Result]

    OBS1 --> THINK2[Thought:<br/>What does this mean?]
    THINK2 --> DECIDE{Goal<br/>Achieved?}

    DECIDE -->|No| ACT2[Action:<br/>Next Tool]
    ACT2 --> OBS2[Observation]
    OBS2 --> THINK2

    DECIDE -->|Yes| ANSWER[Final Answer]

    style THINK1 fill:#fff4e1
    style THINK2 fill:#fff4e1
    style ACT1 fill:#e1f5ff
    style ACT2 fill:#e1f5ff
    style OBS1 fill:#f0e1ff
    style OBS2 fill:#f0e1ff
```

**Detailed ReAct Loop:**

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant LLM
    participant Tools

    User->>Agent: Complex Query
    loop Until Goal Achieved
        Agent->>LLM: What should I do?
        LLM->>Agent: Thought + Action
        Agent->>Tools: Execute Action
        Tools->>Agent: Observation
        Agent->>LLM: Process Observation
        LLM->>Agent: Analysis
    end
    Agent->>User: Final Answer
```

## 4. Plan-and-Execute Pattern

```mermaid
graph TB
    GOAL[User Goal] --> PLANNER[Planning Phase]

    PLANNER --> PLAN[Detailed Plan:<br/>Step 1, 2, 3...]

    PLAN --> EX1[Execute Step 1]
    EX1 --> CHK1{Success?}
    CHK1 -->|Yes| EX2[Execute Step 2]
    CHK1 -->|No| REPLAN1[Replan]
    REPLAN1 --> EX1

    EX2 --> CHK2{Success?}
    CHK2 -->|Yes| EX3[Execute Step 3]
    CHK2 -->|No| REPLAN2[Replan]
    REPLAN2 --> EX2

    EX3 --> RESULT[Final Result]

    style PLANNER fill:#f5a623
    style PLAN fill:#fff4e1
    style REPLAN1 fill:#ff6b6b
    style REPLAN2 fill:#ff6b6b
```

**Plan Structure:**

```mermaid
classDiagram
    class Plan {
        +String goal
        +List~Step~ steps
        +execute() Result
        +replan(failed_step) Plan
    }

    class Step {
        +int id
        +String description
        +String tool
        +Map parameters
        +List~int~ dependencies
        +execute() Result
    }

    class PlanExecutor {
        +Plan plan
        +execute_step(step) Result
        +handle_failure(step) void
        +check_dependencies(step) boolean
    }

    Plan --> Step
    PlanExecutor --> Plan
```

## 5. Reflection Pattern

```mermaid
graph LR
    TASK[Task] --> GEN1[Generate<br/>Solution v1]
    GEN1 --> EVAL1[Self-Evaluate]
    EVAL1 --> SCORE1{Score > 8?}

    SCORE1 -->|No| CRITIQUE[Identify<br/>Weaknesses]
    CRITIQUE --> GEN2[Generate<br/>Solution v2]
    GEN2 --> EVAL2[Self-Evaluate]
    EVAL2 --> SCORE2{Score > 8?}

    SCORE2 -->|No| CRITIQUE
    SCORE2 -->|Yes| DONE[Final Solution]
    SCORE1 -->|Yes| DONE

    style EVAL1 fill:#f0e1ff
    style EVAL2 fill:#f0e1ff
    style CRITIQUE fill:#ff6b6b
```

## 6. Hierarchical Agent Pattern

```mermaid
graph TB
    USER[User] --> SUPERVISOR[Supervisor Agent<br/>Orchestrator]

    SUPERVISOR --> AGENT1[Research Agent]
    SUPERVISOR --> AGENT2[Analysis Agent]
    SUPERVISOR --> AGENT3[Writing Agent]
    SUPERVISOR --> AGENT4[Review Agent]

    AGENT1 --> T1[Web Search]
    AGENT1 --> T2[Doc Retrieval]

    AGENT2 --> T3[Data Analysis]
    AGENT2 --> T4[Statistics]

    AGENT3 --> T5[Content Gen]
    AGENT3 --> T6[Formatting]

    AGENT4 --> T7[Quality Check]
    AGENT4 --> T8[Fact Verify]

    AGENT1 -.->|Results| SUPERVISOR
    AGENT2 -.->|Results| SUPERVISOR
    AGENT3 -.->|Results| SUPERVISOR
    AGENT4 -.->|Results| SUPERVISOR

    SUPERVISOR --> USER

    style SUPERVISOR fill:#6c5ce7
    style AGENT1 fill:#4ecdc4
    style AGENT2 fill:#45b7d1
    style AGENT3 fill:#f9ca24
    style AGENT4 fill:#ff6b6b
```

## Pattern Selection Matrix

```mermaid
graph TD
    Q1{Task<br/>Complexity?}
    Q1 -->|Simple| REFLEX[Simple Reflex<br/>Pattern]
    Q1 -->|Medium| Q2{Need<br/>Context?}
    Q1 -->|Complex| Q3{Single or<br/>Multi-Agent?}

    Q2 -->|Yes| MODEL[Model-Based<br/>Pattern]
    Q2 -->|No| REFLEX

    Q3 -->|Single| Q4{Need<br/>Planning?}
    Q3 -->|Multi| HIER[Hierarchical<br/>Pattern]

    Q4 -->|Yes| PLAN[Plan-Execute<br/>Pattern]
    Q4 -->|No| REACT[ReAct<br/>Pattern]

    style REFLEX fill:#ff6b6b
    style MODEL fill:#4ecdc4
    style REACT fill:#45b7d1
    style PLAN fill:#f9ca24
    style HIER fill:#6c5ce7
```

## Hybrid Pattern: ReAct + Planning

```mermaid
sequenceDiagram
    participant User
    participant Planner
    participant Executor
    participant Tools

    User->>Planner: Complex Goal
    Planner->>Planner: Create High-Level Plan
    Planner->>Executor: Execute Step 1

    loop ReAct for Each Step
        Executor->>Executor: Thought
        Executor->>Tools: Action
        Tools->>Executor: Observation
        Executor->>Executor: Evaluate
    end

    Executor->>Planner: Step 1 Complete
    Planner->>Executor: Execute Step 2
    note over Executor,Tools: Repeat ReAct loop
    Executor->>Planner: Step 2 Complete
    Planner->>User: Final Result
```

## Agent Pattern Characteristics

```mermaid
graph LR
    subgraph "Pattern Comparison"
        direction TB

        P1[Simple Reflex]
        P2[Model-Based]
        P3[ReAct]
        P4[Plan-Execute]
        P5[Hierarchical]

        P1 -.->|Speed: ⚡⚡⚡| S1
        P1 -.->|Cost: $| C1
        P1 -.->|Reliability: ⭐⭐| R1

        P2 -.->|Speed: ⚡⚡| S2
        P2 -.->|Cost: $$| C2
        P2 -.->|Reliability: ⭐⭐⭐| R2

        P3 -.->|Speed: ⚡| S3
        P3 -.->|Cost: $$$| C3
        P3 -.->|Reliability: ⭐⭐⭐⭐| R3

        P4 -.->|Speed: ⚡| S4
        P4 -.->|Cost: $$$$| C4
        P4 -.->|Reliability: ⭐⭐⭐⭐⭐| R4

        P5 -.->|Speed: ⚡| S5
        P5 -.->|Cost: $$$$$| C5
        P5 -.->|Reliability: ⭐⭐⭐⭐⭐| R5
    end
```

## State Management Across Patterns

```mermaid
graph TB
    subgraph "State in Different Patterns"
        SR[Simple Reflex:<br/>Stateless]
        MB[Model-Based:<br/>Session State]
        RE[ReAct:<br/>Step State]
        PE[Plan-Execute:<br/>Plan State]
        HI[Hierarchical:<br/>Distributed State]
    end

    subgraph "State Storage"
        NONE[No Storage]
        MEM[In-Memory]
        DISK[Disk/DB]
        DIST[Distributed]
    end

    SR --> NONE
    MB --> MEM
    RE --> MEM
    PE --> DISK
    HI --> DIST

    style SR fill:#ff6b6b
    style MB fill:#4ecdc4
    style RE fill:#45b7d1
    style PE fill:#f9ca24
    style HI fill:#6c5ce7
```

## Related Topics

- **Day 1**: Agent fundamentals
- **Day 4**: Tool use (used in all patterns)
- **Day 8**: ReAct pattern deep dive
- **Day 9**: Memory systems (for Model-Based)
- **Day 10**: Planning (for Plan-Execute)
- **Day 15**: Multi-agent systems (for Hierarchical)

## Pattern Evolution

```mermaid
timeline
    title Agent Pattern Evolution
    2020 : Simple prompt-based
         : Direct LLM calls
    2021 : Chain-of-Thought
         : Few-shot learning
    2022 : ReAct Pattern
         : Tool use emerges
    2023 : Plan-and-Execute
         : Multi-agent systems
    2024 : Self-improving agents
         : Autonomous operation
```

---

**Related Days**: [Day 1](./day-01-agent-fundamentals.md) | [Day 8](./day-08-react-pattern.md) | [Day 10](./day-10-planning.md)
