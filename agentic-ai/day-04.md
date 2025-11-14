# Day 4: Function Calling & Tool Use

## Overview
Function calling (tool use) is what transforms an LLM from a text generator into an agent that can take actions. Today you'll learn how agents interact with external tools, APIs, and functions to accomplish real-world tasks.

## Learning Objectives
- Understand function calling mechanics
- Implement tools with OpenAI and Anthropic APIs
- Create reusable tool libraries
- Handle tool execution and error recovery
- Build agents that chain multiple tool calls
- Understand when and how to use tools

## Theoretical Concepts

### What is Function Calling?

Function calling allows LLMs to:
1. **Recognize** when a tool is needed
2. **Structure** the call with correct parameters
3. **Return** results in a parseable format

The flow:
```
User: "What's the weather in Tokyo?"
    ↓
Agent: Recognizes need for weather tool
    ↓
Agent: Returns function call: get_weather(location="Tokyo")
    ↓
System: Executes function, gets result
    ↓
Agent: Formats result for user
```

### Function Calling vs. Prompting
| Approach | Function Calling | Prompt-Based |
|----------|------------------|--------------|
| Reliability | High - structured output | Variable |
| Parsing | Native JSON | Manual parsing |
| Validation | Schema-enforced | Manual validation |
| Error Handling | Built-in | Custom logic |

### Tool Design Principles
1. **Single Responsibility**: Each tool does one thing well
2. **Clear Parameters**: Well-defined inputs with types
3. **Robust Error Handling**: Graceful failures with helpful messages
4. **Idempotent**: Safe to retry
5. **Well-Documented**: Clear descriptions for the LLM

## Hands-On Exercises

### Exercise 1: Basic Function Calling (OpenAI)

```python
# basic_function_calling.py
import json
from openai import OpenAI

client = OpenAI()

# Define tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name, e.g., Tokyo, London"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit"
                    }
                },
                "required": ["location"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Perform mathematical calculations",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Math expression to evaluate, e.g., '2 + 2'"
                    }
                },
                "required": ["expression"]
            }
        }
    }
]

# Implement actual functions
def get_weather(location, unit="celsius"):
    """Simulated weather API"""
    # In production, call real weather API
    weather_data = {
        "Tokyo": {"temp": 22, "condition": "Sunny"},
        "London": {"temp": 15, "condition": "Cloudy"},
        "New York": {"temp": 18, "condition": "Rainy"}
    }

    data = weather_data.get(location, {"temp": 20, "condition": "Unknown"})

    if unit == "fahrenheit":
        data["temp"] = data["temp"] * 9/5 + 32

    return json.dumps(data)

def calculate(expression):
    """Safe calculator"""
    try:
        # Security: eval is dangerous! Use ast.literal_eval or a proper parser
        # This is simplified for demonstration
        result = eval(expression, {"__builtins__": {}}, {})
        return json.dumps({"result": result})
    except Exception as e:
        return json.dumps({"error": str(e)})

# Function dispatcher
available_functions = {
    "get_weather": get_weather,
    "calculate": calculate
}

# Agent loop
def run_agent(user_message):
    """Run agent with function calling"""
    messages = [{"role": "user", "content": user_message}]

    # First API call
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=messages,
        tools=tools,
        tool_choice="auto"
    )

    response_message = response.choices[0].message
    messages.append(response_message)

    # Check if function calling is needed
    if response_message.tool_calls:
        for tool_call in response_message.tool_calls:
            function_name = tool_call.function.name
            function_args = json.loads(tool_call.function.arguments)

            print(f"Calling: {function_name}({function_args})")

            # Execute function
            function_response = available_functions[function_name](**function_args)

            # Add function response to messages
            messages.append({
                "tool_call_id": tool_call.id,
                "role": "tool",
                "name": function_name,
                "content": function_response
            })

        # Second API call with function results
        second_response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=messages
        )

        return second_response.choices[0].message.content

    return response_message.content

# Test
print(run_agent("What's the weather in Tokyo?"))
print(run_agent("Calculate 15 * 7 + 23"))
```

### Exercise 2: Function Calling with Anthropic Claude

```python
# anthropic_tools.py
import json
from anthropic import Anthropic

client = Anthropic()

# Define tools
tools = [
    {
        "name": "search_knowledge_base",
        "description": "Search a knowledge base for information on a topic",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "The search query"
                },
                "category": {
                    "type": "string",
                    "enum": ["tech", "science", "business", "general"],
                    "description": "Category to search within"
                }
            },
            "required": ["query"]
        }
    }
]

def search_knowledge_base(query, category="general"):
    """Simulated knowledge base search"""
    results = {
        "query": query,
        "category": category,
        "results": [
            {"title": f"Article about {query}", "relevance": 0.95},
            {"title": f"Guide to {query}", "relevance": 0.87}
        ]
    }
    return json.dumps(results)

# Agent execution
def run_claude_agent(user_message):
    """Run Claude agent with tools"""
    messages = [{"role": "user", "content": user_message}]

    response = client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )

    print(f"Stop reason: {response.stop_reason}")

    if response.stop_reason == "tool_use":
        # Extract tool use
        tool_use_block = next(block for block in response.content if block.type == "tool_use")

        tool_name = tool_use_block.name
        tool_input = tool_use_block.input

        print(f"Using tool: {tool_name}")
        print(f"Input: {tool_input}")

        # Execute tool
        if tool_name == "search_knowledge_base":
            result = search_knowledge_base(**tool_input)

        # Continue conversation with tool result
        messages.append({"role": "assistant", "content": response.content})
        messages.append({
            "role": "user",
            "content": [{
                "type": "tool_result",
                "tool_use_id": tool_use_block.id,
                "content": result
            }]
        })

        # Get final response
        final_response = client.messages.create(
            model="claude-3-sonnet-20240229",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        return final_response.content[0].text

    return response.content[0].text

# Test
print(run_claude_agent("Search for information about quantum computing"))
```

### Exercise 3: Building a Tool Library

```python
# tool_library.py
import json
import requests
from typing import Dict, Any, Callable
from dataclasses import dataclass

@dataclass
class Tool:
    """Represents a tool that an agent can use"""
    name: str
    description: str
    parameters: Dict[str, Any]
    function: Callable

class ToolLibrary:
    """Manages a collection of tools for agents"""

    def __init__(self):
        self.tools = {}

    def register(self, name: str, description: str, parameters: Dict[str, Any]):
        """Decorator to register a tool"""
        def decorator(func: Callable):
            self.tools[name] = Tool(
                name=name,
                description=description,
                parameters=parameters,
                function=func
            )
            return func
        return decorator

    def get_tool_schemas(self):
        """Get OpenAI-compatible tool schemas"""
        return [
            {
                "type": "function",
                "function": {
                    "name": tool.name,
                    "description": tool.description,
                    "parameters": tool.parameters
                }
            }
            for tool in self.tools.values()
        ]

    def execute(self, name: str, **kwargs):
        """Execute a tool by name"""
        if name not in self.tools:
            raise ValueError(f"Tool {name} not found")

        try:
            result = self.tools[name].function(**kwargs)
            return {"success": True, "result": result}
        except Exception as e:
            return {"success": False, "error": str(e)}

# Create library and register tools
library = ToolLibrary()

@library.register(
    name="web_search",
    description="Search the web for information",
    parameters={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Search query"},
            "num_results": {"type": "integer", "description": "Number of results", "default": 5}
        },
        "required": ["query"]
    }
)
def web_search(query: str, num_results: int = 5):
    """Simulated web search"""
    return {
        "query": query,
        "results": [f"Result {i+1} for '{query}'" for i in range(num_results)]
    }

@library.register(
    name="file_read",
    description="Read contents of a file",
    parameters={
        "type": "object",
        "properties": {
            "file_path": {"type": "string", "description": "Path to file"}
        },
        "required": ["file_path"]
    }
)
def file_read(file_path: str):
    """Read file contents"""
    try:
        with open(file_path, 'r') as f:
            return f.read()
    except FileNotFoundError:
        return f"File not found: {file_path}"

@library.register(
    name="send_email",
    description="Send an email",
    parameters={
        "type": "object",
        "properties": {
            "to": {"type": "string", "description": "Recipient email"},
            "subject": {"type": "string", "description": "Email subject"},
            "body": {"type": "string", "description": "Email body"}
        },
        "required": ["to", "subject", "body"]
    }
)
def send_email(to: str, subject: str, body: str):
    """Send email (simulated)"""
    return f"Email sent to {to}: {subject}"

# Usage
print("Available tools:")
for name in library.tools:
    print(f"- {name}: {library.tools[name].description}")

print("\nTool schemas for LLM:")
print(json.dumps(library.get_tool_schemas(), indent=2))

print("\nExecuting tool:")
result = library.execute("web_search", query="AI agents", num_results=3)
print(result)
```

### Exercise 4: Multi-Step Tool Chaining

```python
# tool_chaining_agent.py
from openai import OpenAI
import json

client = OpenAI()

class ChainAgent:
    """Agent that can chain multiple tool calls"""

    def __init__(self, tools, functions):
        self.tools = tools
        self.functions = functions
        self.max_iterations = 10

    def run(self, user_message):
        """Run agent with tool chaining"""
        messages = [{"role": "user", "content": user_message}]
        iteration = 0

        while iteration < self.max_iterations:
            iteration += 1
            print(f"\n--- Iteration {iteration} ---")

            response = client.chat.completions.create(
                model="gpt-4-turbo-preview",
                messages=messages,
                tools=self.tools,
                tool_choice="auto"
            )

            response_message = response.choices[0].message
            messages.append(response_message)

            # Check if done
            if not response_message.tool_calls:
                return response_message.content

            # Execute all tool calls
            for tool_call in response_message.tool_calls:
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)

                print(f"Calling: {function_name}({function_args})")

                function_response = self.functions[function_name](**function_args)

                messages.append({
                    "tool_call_id": tool_call.id,
                    "role": "tool",
                    "name": function_name,
                    "content": json.dumps(function_response)
                })

        return "Max iterations reached"

# Define multi-step tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_user_data",
            "description": "Get user information from database",
            "parameters": {
                "type": "object",
                "properties": {
                    "user_id": {"type": "string"}
                },
                "required": ["user_id"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_order_history",
            "description": "Get order history for a user",
            "parameters": {
                "type": "object",
                "properties": {
                    "user_id": {"type": "string"}
                },
                "required": ["user_id"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "calculate_discount",
            "description": "Calculate discount based on order history",
            "parameters": {
                "type": "object",
                "properties": {
                    "total_orders": {"type": "integer"},
                    "total_spent": {"type": "number"}
                },
                "required": ["total_orders", "total_spent"]
            }
        }
    }
]

# Implement functions
def get_user_data(user_id):
    return {"user_id": user_id, "name": "John Doe", "tier": "gold"}

def get_order_history(user_id):
    return {"user_id": user_id, "total_orders": 15, "total_spent": 1500.00}

def calculate_discount(total_orders, total_spent):
    if total_orders > 10 and total_spent > 1000:
        return {"discount_percent": 15, "reason": "VIP customer"}
    elif total_orders > 5:
        return {"discount_percent": 10, "reason": "Loyal customer"}
    return {"discount_percent": 5, "reason": "Standard"}

functions = {
    "get_user_data": get_user_data,
    "get_order_history": get_order_history,
    "calculate_discount": calculate_discount
}

# Run agent
agent = ChainAgent(tools, functions)
result = agent.run("What discount should we offer user 'user_123'?")
print(f"\nFinal Answer:\n{result}")
```

### Exercise 5: Error Handling in Tool Execution

```python
# robust_tool_execution.py
import json
from typing import Any, Dict
from enum import Enum

class ToolStatus(Enum):
    SUCCESS = "success"
    ERROR = "error"
    RETRY = "retry"

class ToolResult:
    """Standardized tool result"""
    def __init__(self, status: ToolStatus, data: Any = None, error: str = None):
        self.status = status
        self.data = data
        self.error = error

    def to_json(self):
        return json.dumps({
            "status": self.status.value,
            "data": self.data,
            "error": self.error
        })

def execute_tool_safely(tool_func, **kwargs):
    """Execute tool with comprehensive error handling"""
    try:
        result = tool_func(**kwargs)
        return ToolResult(ToolStatus.SUCCESS, data=result)

    except ConnectionError as e:
        # Retryable error
        return ToolResult(ToolStatus.RETRY, error=f"Connection error: {e}")

    except ValueError as e:
        # Invalid input
        return ToolResult(ToolStatus.ERROR, error=f"Invalid input: {e}")

    except Exception as e:
        # Unexpected error
        return ToolResult(ToolStatus.ERROR, error=f"Unexpected error: {e}")

# Example tool with validation
def validated_tool(api_key: str, query: str):
    """Tool with input validation"""
    if not api_key:
        raise ValueError("API key is required")

    if len(query) < 3:
        raise ValueError("Query must be at least 3 characters")

    # Simulated API call
    if "error" in query.lower():
        raise ConnectionError("API temporarily unavailable")

    return {"query": query, "results": ["Result 1", "Result 2"]}

# Test error handling
test_cases = [
    {"api_key": "valid", "query": "test query"},  # Success
    {"api_key": "", "query": "test"},  # Validation error
    {"api_key": "valid", "query": "error test"},  # Connection error
]

for i, test in enumerate(test_cases, 1):
    print(f"\nTest {i}: {test}")
    result = execute_tool_safely(validated_tool, **test)
    print(f"Status: {result.status.value}")
    print(f"Result: {result.to_json()}")
```

## Key Takeaways
1. Function calling enables agents to take real-world actions
2. Tools should be single-purpose, well-documented, and robust
3. Tool schemas guide the LLM on when and how to use tools
4. Agents can chain multiple tools to accomplish complex tasks
5. Proper error handling is critical for reliable tool execution
6. Tool libraries enable code reuse and standardization

## Best Practices

### Tool Design:
- Clear, descriptive names
- Comprehensive parameter descriptions
- Proper type definitions
- Required vs optional parameters
- Error handling and validation

### Security:
- Validate all inputs
- Sanitize data before external calls
- Implement rate limiting
- Log tool usage
- Never execute arbitrary code (avoid eval)

### Performance:
- Cache frequently used data
- Implement timeouts
- Handle retries intelligently
- Monitor tool execution time

## Resources
- **Documentation**:
  - [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
  - [Anthropic Tool Use](https://docs.anthropic.com/claude/docs/tool-use)
- **Examples**:
  - [LangChain Tools](https://python.langchain.com/docs/modules/agents/tools/)
  - [Semantic Kernel Skills](https://learn.microsoft.com/en-us/semantic-kernel/ai-orchestration/plugins/)

## Daily Challenge

**Build a Personal Assistant Agent**

Create an agent with these tools:
1. `add_todo(task, priority)` - Add task to list
2. `list_todos(filter)` - List tasks
3. `mark_complete(task_id)` - Mark task done
4. `set_reminder(task_id, time)` - Set reminder
5. `get_weather(location)` - Check weather

The agent should:
- Handle conversational requests like "Add buying milk to my todo list"
- Chain tools: "What should I do today considering the weather?"
- Handle errors gracefully
- Maintain state across interactions

## Reflection Questions
1. When should an agent use a tool vs. generate a response directly?
2. How do you prevent agents from misusing tools?
3. What's the trade-off between many specialized tools vs. few general ones?
4. How do you test tool reliability?

## Tomorrow's Preview
Day 5: Building Your First Simple Agent - Put everything together to build a complete, working agent from scratch.

---

**Progress**: 4/30 days completed

[← Previous: Day 3](./day-03.md) | [Back to Overview](../README.md) | [Next: Day 5 →](./day-05.md)
