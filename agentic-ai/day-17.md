# Day 17: Agent Orchestration Frameworks

## Overview
Learn to use production frameworks that simplify multi-agent system development. Focus on LangGraph, AutoGen, and CrewAI.

## Learning Objectives
- Master LangGraph for stateful agents
- Build AutoGen multi-agent conversations
- Create CrewAI teams
- Compare framework capabilities
- Choose the right framework for your use case

## Framework Overview

### LangGraph
- Graph-based agent workflows
- Stateful execution
- Complex control flow
- Great for custom logic

### AutoGen
- Conversational agents
- Group chat patterns
- Code execution
- Research-focused

### CrewAI
- Role-based agents
- Sequential/hierarchical tasks
- Simple API
- Production-ready

## LangGraph Example

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    next_action: str

def researcher(state):
    # Research logic
    return {"messages": state["messages"] + ["Research complete"]}

def writer(state):
    # Writing logic
    return {"messages": state["messages"] + ["Draft complete"]}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("research", researcher)
workflow.add_node("write", writer)
workflow.add_edge("research", "write")
workflow.add_edge("write", END)

app = workflow.compile()
```

## AutoGen Example

```python
from autogen import AssistantAgent, UserProxyAgent

assistant = AssistantAgent("assistant")
user_proxy = UserProxyAgent("user")

user_proxy.initiate_chat(
    assistant,
    message="Analyze this code and suggest improvements"
)
```

## CrewAI Example

```python
from crewai import Agent, Task, Crew

researcher = Agent(
    role="Researcher",
    goal="Find accurate information",
    backstory="Expert researcher"
)

writer = Agent(
    role="Writer",
    goal="Create engaging content",
    backstory="Skilled writer"
)

task = Task(
    description="Research and write about AI",
    agent=researcher
)

crew = Crew(
    agents=[researcher, writer],
    tasks=[task]
)

result = crew.kickoff()
```

## Framework Comparison

| Feature | LangGraph | AutoGen | CrewAI |
|---------|-----------|---------|--------|
| Learning Curve | Medium | Low | Low |
| Flexibility | High | Medium | Low |
| Production Ready | Yes | Yes | Yes |
| Best For | Custom workflows | Research/Code | Business tasks |

## Resources
- [LangGraph Docs](https://langchain-ai.github.io/langgraph/)
- [AutoGen Docs](https://microsoft.github.io/autogen/)
- [CrewAI Docs](https://docs.crewai.com/)

## Daily Challenge
Build the same multi-agent system using all three frameworks and compare.

---

**Progress**: 17/30 days completed

[← Previous: Day 16](./day-16.md) | [Back to Overview](../README.md) | [Next: Day 18 →](./day-18.md)
