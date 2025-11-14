# Day 19: AutoGen & CrewAI Frameworks

## Overview
Explore AutoGen and CrewAI - frameworks designed for specific multi-agent use cases.

## Learning Objectives
- Build AutoGen conversational agents
- Create CrewAI teams and tasks
- Implement group chat patterns
- Design role-based agent systems
- Compare with LangChain/LangGraph

## AutoGen Framework

### Key Features
- Multi-agent conversations
- Code execution capabilities
- Human proxy agents
- Group chat orchestration

### Example

```python
from autogen import AssistantAgent, UserProxyAgent, GroupChat, GroupChatManager

# Create agents
coder = AssistantAgent(
    name="Coder",
    llm_config={"model": "gpt-4"}
)

reviewer = AssistantAgent(
    name="Reviewer",
    system_message="Review code for quality"
)

user = UserProxyAgent(
    name="User",
    human_input_mode="NEVER"
)

# Group chat
group_chat = GroupChat(
    agents=[coder, reviewer, user],
    messages=[],
    max_round=10
)

manager = GroupChatManager(groupchat=group_chat)

user.initiate_chat(
    manager,
    message="Create a Python function to calculate fibonacci"
)
```

## CrewAI Framework

### Key Features
- Role-based agents
- Sequential and hierarchical tasks
- Built-in tools
- Simple API

### Example

```python
from crewai import Agent, Task, Crew, Process

# Define agents
researcher = Agent(
    role="Senior Researcher",
    goal="Research cutting-edge AI",
    tools=[search_tool],
    verbose=True
)

writer = Agent(
    role="Tech Writer",
    goal="Create engaging articles",
    tools=[],
    verbose=True
)

# Define tasks
research_task = Task(
    description="Research latest AI trends",
    agent=researcher,
    expected_output="Research report"
)

write_task = Task(
    description="Write article based on research",
    agent=writer,
    expected_output="Article"
)

# Create crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)

result = crew.kickoff()
```

## When to Use Each

| Use Case | Framework |
|----------|-----------|
| Research & Code | AutoGen |
| Business Workflows | CrewAI |
| Custom Logic | LangGraph |
| Quick Prototypes | LangChain |

## Resources
- [AutoGen Examples](https://github.com/microsoft/autogen/tree/main/notebook)
- [CrewAI Examples](https://github.com/joaomdmoura/crewAI-examples)

## Daily Challenge
Build a software development team using AutoGen with: architect, developer, tester, and reviewer agents.

---

**Progress**: 19/30 days completed

[← Previous: Day 18](./day-18.md) | [Back to Overview](../README.md) | [Next: Day 20 →](./day-20.md)
