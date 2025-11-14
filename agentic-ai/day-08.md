# Day 8: ReAct Pattern - Reasoning and Acting

## Overview
The ReAct (Reasoning and Acting) pattern is one of the most important and widely-used agent architectures. Today you'll master this pattern through deep implementation and real-world examples.

## Learning Objectives
- Understand ReAct pattern in depth
- Implement production-grade ReAct agents
- Handle complex multi-step reasoning
- Debug ReAct loops effectively
- Optimize ReAct performance
- Compare ReAct with other patterns

## Theoretical Concepts

### What is ReAct?

ReAct combines:
- **Reasoning**: Thinking through problems step-by-step
- **Acting**: Taking actions using tools

The key insight: **Interleaving reasoning and acting** produces better results than doing them separately.

### The ReAct Loop

```
Question/Goal
    ↓
Thought: What do I need to know/do?
    ↓
Action: [tool_name](parameters)
    ↓
Observation: [tool result]
    ↓
Thought: What does this tell me?
    ↓
Action: [next tool] OR Final Answer
    ↓
...repeat until solved...
    ↓
Thought: I can now answer the question
    ↓
Final Answer: [solution]
```

### Benefits of ReAct

1. **Transparency**: See the reasoning process
2. **Debuggability**: Identify where things went wrong
3. **Reliability**: Step-by-step validation
4. **Adaptability**: Can change course based on observations
5. **Error Recovery**: Natural retry and correction

### Challenges

1. **Token Usage**: Verbose, uses more tokens
2. **Latency**: Multiple LLM calls
3. **Loop Control**: Can get stuck in loops
4. **Prompt Drift**: Long contexts can confuse the model

## Hands-On Exercises

### Exercise 1: Production ReAct Agent

```python
# production_react_agent.py
import json
import re
from openai import OpenAI
from typing import List, Dict, Optional, Tuple
from dataclasses import dataclass
from enum import Enum

client = OpenAI()

class ReActState(Enum):
    THINKING = "thinking"
    ACTING = "acting"
    OBSERVING = "observing"
    FINISHED = "finished"
    ERROR = "error"

@dataclass
class ReActStep:
    """Represents a single ReAct step"""
    iteration: int
    thought: str
    action: Optional[str] = None
    action_input: Optional[Dict] = None
    observation: Optional[str] = None
    state: ReActState = ReActState.THINKING

class ProductionReActAgent:
    """
    Production-grade ReAct agent with:
    - Robust error handling
    - Loop detection
    - State management
    - Detailed logging
    """

    def __init__(self, tools: Dict, max_iterations: int = 10,
                 verbose: bool = True):
        self.tools = tools
        self.max_iterations = max_iterations
        self.verbose = verbose
        self.history: List[ReActStep] = []

    def _create_system_prompt(self) -> str:
        """Create system prompt for ReAct"""
        tools_desc = "\n".join([
            f"- {name}: {info['description']}\n  Parameters: {info['parameters']}"
            for name, info in self.tools.items()
        ])

        return f"""You are a helpful agent that solves problems using available tools.

IMPORTANT: Follow this exact format for each step:

Thought: [Your reasoning about what to do next]
Action: [tool_name]
Action Input: {{"param1": "value1", "param2": "value2"}}
Observation: [This will be filled in by the system]

After you receive an observation, think about what it means and decide next action.

When you have enough information to answer the question:
Thought: I now have enough information to answer the question
Final Answer: [Your detailed answer]

Available Tools:
{tools_desc}

Guidelines:
1. Always think before acting
2. Use tools when you need information
3. Validate observations before proceeding
4. If a tool fails, try a different approach
5. Provide detailed final answers with reasoning

Let's begin!
"""

    def _parse_react_step(self, text: str) -> Tuple[Optional[str], Optional[str], Optional[str]]:
        """
        Parse ReAct step from LLM output

        Returns: (thought, action, action_input)
        """
        thought = None
        action = None
        action_input = None

        # Extract Thought
        thought_match = re.search(r'Thought:\s*(.+?)(?=\n|Action:|Final Answer:|$)',
                                 text, re.IGNORECASE | re.DOTALL)
        if thought_match:
            thought = thought_match.group(1).strip()

        # Extract Action
        action_match = re.search(r'Action:\s*(\w+)', text, re.IGNORECASE)
        if action_match:
            action = action_match.group(1).strip()

        # Extract Action Input
        input_match = re.search(r'Action Input:\s*(\{.+?\})',
                               text, re.IGNORECASE | re.DOTALL)
        if input_match:
            try:
                action_input = json.loads(input_match.group(1))
            except json.JSONDecodeError:
                # Try to extract as plain text
                action_input = input_match.group(1).strip()

        return thought, action, action_input

    def _check_final_answer(self, text: str) -> Optional[str]:
        """Check if text contains final answer"""
        final_match = re.search(r'Final Answer:\s*(.+)',
                               text, re.IGNORECASE | re.DOTALL)
        if final_match:
            return final_match.group(1).strip()
        return None

    def _detect_loop(self) -> bool:
        """Detect if agent is stuck in a loop"""
        if len(self.history) < 3:
            return False

        # Check if last 3 actions are identical
        recent_actions = [step.action for step in self.history[-3:]]
        if len(set(recent_actions)) == 1 and recent_actions[0] is not None:
            return True

        return False

    def _execute_tool(self, tool_name: str, tool_input: Dict) -> str:
        """Execute a tool with error handling"""
        if tool_name not in self.tools:
            return f"Error: Tool '{tool_name}' not found. Available tools: {list(self.tools.keys())}"

        try:
            result = self.tools[tool_name]['function'](**tool_input)
            return json.dumps(result) if isinstance(result, dict) else str(result)
        except TypeError as e:
            return f"Error: Invalid parameters for {tool_name}. {str(e)}"
        except Exception as e:
            return f"Error executing {tool_name}: {str(e)}"

    def run(self, question: str) -> str:
        """
        Run the ReAct agent

        Args:
            question: The question/goal to solve

        Returns:
            Final answer
        """
        self.history = []
        messages = [
            {"role": "system", "content": self._create_system_prompt()},
            {"role": "user", "content": question}
        ]

        if self.verbose:
            print(f"\n{'='*70}")
            print(f"Question: {question}")
            print(f"{'='*70}\n")

        for iteration in range(1, self.max_iterations + 1):
            if self.verbose:
                print(f"\n--- Iteration {iteration} ---\n")

            # Check for loop
            if self._detect_loop():
                if self.verbose:
                    print("⚠️  Loop detected! Breaking out...")
                return "I seem to be stuck in a loop. Please rephrase the question or provide more context."

            # Get LLM response
            try:
                response = client.chat.completions.create(
                    model="gpt-4-turbo-preview",
                    messages=messages,
                    temperature=0.3,
                    max_tokens=1000
                )

                llm_output = response.choices[0].message.content

                if self.verbose:
                    print(llm_output)

                # Check for final answer
                final_answer = self._check_final_answer(llm_output)
                if final_answer:
                    step = ReActStep(
                        iteration=iteration,
                        thought="Final answer reached",
                        state=ReActState.FINISHED
                    )
                    self.history.append(step)

                    if self.verbose:
                        print(f"\n{'='*70}")
                        print(f"✓ Final Answer: {final_answer}")
                        print(f"{'='*70}")

                    return final_answer

                # Parse ReAct step
                thought, action, action_input = self._parse_react_step(llm_output)

                step = ReActStep(
                    iteration=iteration,
                    thought=thought,
                    action=action,
                    action_input=action_input,
                    state=ReActState.THINKING
                )

                # Execute action if present
                if action and action_input:
                    step.state = ReActState.ACTING

                    if self.verbose:
                        print(f"\n🔧 Executing: {action}({action_input})")

                    observation = self._execute_tool(action, action_input)
                    step.observation = observation
                    step.state = ReActState.OBSERVING

                    if self.verbose:
                        print(f"📊 Observation: {observation}\n")

                    # Add observation to messages
                    messages.append({"role": "assistant", "content": llm_output})
                    messages.append({"role": "user", "content": f"Observation: {observation}"})
                else:
                    # No valid action, prompt for one
                    messages.append({"role": "assistant", "content": llm_output})
                    messages.append({
                        "role": "user",
                        "content": "Please specify an Action and Action Input, or provide a Final Answer."
                    })

                self.history.append(step)

            except Exception as e:
                if self.verbose:
                    print(f"❌ Error: {e}")

                step = ReActStep(
                    iteration=iteration,
                    thought=f"Error occurred: {e}",
                    state=ReActState.ERROR
                )
                self.history.append(step)

                return f"An error occurred: {e}"

        return "Maximum iterations reached. Unable to find a complete answer."

    def get_trace(self) -> List[Dict]:
        """Get execution trace for debugging"""
        return [
            {
                "iteration": step.iteration,
                "thought": step.thought,
                "action": step.action,
                "action_input": step.action_input,
                "observation": step.observation,
                "state": step.state.value
            }
            for step in self.history
        ]

# Example tools
def search_database(query: str) -> Dict:
    """Simulated database search"""
    database = {
        "Python": {"type": "language", "year": 1991, "creator": "Guido van Rossum"},
        "React": {"type": "library", "year": 2013, "creator": "Facebook"},
        "Docker": {"type": "tool", "year": 2013, "creator": "Solomon Hykes"}
    }

    for key, value in database.items():
        if key.lower() in query.lower():
            return value

    return {"error": "Not found in database"}

def calculate(expression: str) -> float:
    """Safe calculator"""
    try:
        # Use a safe eval alternative in production
        result = eval(expression, {"__builtins__": {}}, {})
        return {"result": result}
    except Exception as e:
        return {"error": str(e)}

def get_current_date() -> Dict:
    """Get current date"""
    from datetime import datetime
    return {
        "date": datetime.now().strftime("%Y-%m-%d"),
        "year": datetime.now().year
    }

# Setup and run
tools = {
    "search_database": {
        "description": "Search the knowledge database for information",
        "parameters": {"query": "string - search query"},
        "function": search_database
    },
    "calculate": {
        "description": "Calculate a mathematical expression",
        "parameters": {"expression": "string - math expression"},
        "function": calculate
    },
    "get_current_date": {
        "description": "Get the current date and year",
        "parameters": {},
        "function": get_current_date
    }
}

# Test cases
agent = ProductionReActAgent(tools, max_iterations=8, verbose=True)

# Test 1: Simple query
print("\n" + "="*70)
print("TEST 1: Simple Database Query")
print("="*70)
result1 = agent.run("What year was Python created?")

# Test 2: Multi-step reasoning
print("\n" + "="*70)
print("TEST 2: Multi-Step Reasoning")
print("="*70)
result2 = agent.run("How many years ago was React created?")

# Test 3: Complex reasoning
print("\n" + "="*70)
print("TEST 3: Complex Reasoning")
print("="*70)
result3 = agent.run("If Python was created in year X and React in year Y, what is (Y - X) * 2?")

# Get execution trace
print("\n" + "="*70)
print("EXECUTION TRACE for last question:")
print("="*70)
import json
print(json.dumps(agent.get_trace(), indent=2))
```

## See the continuation...

This day continues with more advanced topics...

---

**Progress**: 8/30 days completed

[← Previous: Day 7](./day-07.md) | [Back to Overview](../README.md) | [Next: Day 9 →](./day-09.md)
