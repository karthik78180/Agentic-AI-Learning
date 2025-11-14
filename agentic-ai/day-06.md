# Day 6: Agent Architectures & Patterns

## Overview
Today you'll learn different architectural patterns for building agents, from simple reactive agents to complex hierarchical systems. Understanding these patterns helps you choose the right architecture for your use case.

## Learning Objectives
- Understand different agent architectures
- Implement ReAct (Reasoning and Acting) pattern
- Learn Plan-and-Execute pattern
- Build reflection-based agents
- Compare architecture trade-offs
- Design scalable agent systems

## Theoretical Concepts

### Agent Architecture Types

#### 1. Simple Reflex Agent
- **How it works**: Direct stimulus → response mapping
- **Best for**: Simple, well-defined tasks
- **Limitations**: No planning, no memory

```
User Input → LLM → Direct Response
```

#### 2. Model-Based Agent
- **How it works**: Maintains internal state/model of the world
- **Best for**: Tasks requiring context awareness
- **Features**: Memory, state tracking

```
User Input → LLM + Memory → Response
      ↑           ↓
      └─── Update State ───┘
```

#### 3. Goal-Based Agent (ReAct)
- **How it works**: Reasons about goals, plans actions
- **Best for**: Complex, multi-step tasks
- **Features**: Planning, tool use, reasoning

```
Goal → Think → Act → Observe → Think → ...
```

#### 4. Utility-Based Agent
- **How it works**: Evaluates multiple options, chooses best
- **Best for**: Tasks with trade-offs, optimization
- **Features**: Scoring, comparison, decision-making

```
Options → Evaluate Each → Score → Select Best → Execute
```

#### 5. Hierarchical Agent
- **How it works**: Master agent coordinates sub-agents
- **Best for**: Complex systems with specialized components
- **Features**: Delegation, specialization, coordination

```
Master Agent
    ├── Research Agent
    ├── Writing Agent
    └── Critic Agent
```

### ReAct Pattern (Reasoning and Acting)

The most popular pattern for modern agents:

```
Thought: What do I need to do?
Action: tool_name(parameters)
Observation: [result of action]
Thought: What does this mean?
Action: another_tool(parameters)
Observation: [result]
Thought: I can now answer
Answer: [final response]
```

**Benefits**:
- Transparent reasoning
- Debuggable decision-making
- Natural error recovery
- Step-by-step progress

## Hands-On Exercises

### Exercise 1: ReAct Agent Implementation

```python
# react_agent.py
import json
import re
from openai import OpenAI
from typing import List, Dict, Tuple

client = OpenAI()

class ReActAgent:
    """Agent using ReAct (Reasoning + Acting) pattern"""

    def __init__(self, tools: Dict, verbose: bool = True):
        self.tools = tools
        self.verbose = verbose
        self.max_iterations = 10

    def _create_prompt(self, question: str, history: List[str]) -> str:
        """Create ReAct-style prompt"""
        tools_desc = "\n".join([
            f"- {name}: {info['description']}"
            for name, info in self.tools.items()
        ])

        prompt = f"""Answer the following question using this format:

Thought: [Your reasoning about what to do next]
Action: [tool_name: parameters]
Observation: [Result will be provided]

Continue this process until you can provide a final answer:

Thought: I now know the final answer
Final Answer: [Your answer to the original question]

Available tools:
{tools_desc}

Question: {question}

"""
        if history:
            prompt += "\n" + "\n".join(history) + "\n"

        return prompt

    def _parse_action(self, text: str) -> Tuple[str, str]:
        """Parse action from agent response"""
        # Look for Action: tool_name: parameters
        action_pattern = r"Action:\s*(\w+):\s*(.+?)(?=\n|$)"
        match = re.search(action_pattern, text, re.IGNORECASE)

        if match:
            tool_name = match.group(1).strip()
            parameters = match.group(2).strip()
            return tool_name, parameters

        return None, None

    def run(self, question: str) -> str:
        """Run the ReAct agent"""
        history = []
        iteration = 0

        while iteration < self.max_iterations:
            iteration += 1

            if self.verbose:
                print(f"\n{'='*60}")
                print(f"Iteration {iteration}")
                print(f"{'='*60}")

            # Get agent's reasoning and action
            prompt = self._create_prompt(question, history)

            response = client.chat.completions.create(
                model="gpt-4-turbo-preview",
                messages=[{"role": "user", "content": prompt}],
                temperature=0.3,
                max_tokens=500
            )

            agent_response = response.choices[0].message.content

            if self.verbose:
                print(f"\n{agent_response}")

            # Check if we have final answer
            if "Final Answer:" in agent_response:
                # Extract and return final answer
                final_answer = agent_response.split("Final Answer:")[-1].strip()
                return final_answer

            # Parse and execute action
            tool_name, parameters = self._parse_action(agent_response)

            if tool_name and tool_name in self.tools:
                # Execute tool
                try:
                    result = self.tools[tool_name]["function"](parameters)
                    observation = f"Observation: {result}"
                except Exception as e:
                    observation = f"Observation: Error - {str(e)}"

                if self.verbose:
                    print(f"\n{observation}")

                # Add to history
                history.append(agent_response)
                history.append(observation)
            else:
                # No valid action found
                observation = "Observation: Invalid action. Please use one of the available tools."
                history.append(agent_response)
                history.append(observation)

        return "Maximum iterations reached without finding an answer."

# Define tools
def search_web(query: str) -> str:
    """Simulated web search"""
    results = {
        "Python": "Python is a high-level programming language known for its simplicity and readability.",
        "AI": "Artificial Intelligence is the simulation of human intelligence by machines.",
        "quantum computing": "Quantum computing uses quantum phenomena like superposition and entanglement."
    }

    for key, value in results.items():
        if key.lower() in query.lower():
            return value

    return f"No results found for '{query}'"

def calculate(expression: str) -> str:
    """Safe calculator"""
    try:
        # WARNING: eval is dangerous! Use a proper math parser in production
        result = eval(expression, {"__builtins__": {}}, {})
        return str(result)
    except Exception as e:
        return f"Error: {str(e)}"

# Setup tools
tools = {
    "search": {
        "description": "Search for information on a topic",
        "function": search_web
    },
    "calculate": {
        "description": "Calculate a mathematical expression",
        "function": calculate
    }
}

# Run agent
agent = ReActAgent(tools, verbose=True)
result = agent.run("What is Python and what is 15 multiplied by 23?")
print(f"\n\nFinal Result: {result}")
```

### Exercise 2: Plan-and-Execute Pattern

```python
# plan_execute_agent.py
from openai import OpenAI
from typing import List, Dict
import json

client = OpenAI()

class PlanExecuteAgent:
    """Agent that plans first, then executes"""

    def __init__(self, tools: Dict):
        self.tools = tools

    def create_plan(self, goal: str) -> List[Dict]:
        """Create a step-by-step plan"""
        tools_desc = "\n".join([
            f"- {name}: {info['description']}"
            for name, info in self.tools.items()
        ])

        prompt = f"""Create a detailed step-by-step plan to accomplish this goal:

Goal: {goal}

Available tools:
{tools_desc}

Return your plan as a JSON array of steps, where each step has:
- step_number: integer
- description: what to do
- tool: which tool to use (or "none")
- parameters: parameters for the tool (or null)
- depends_on: array of step numbers this depends on (or empty array)

Example format:
[
  {{"step_number": 1, "description": "Search for X", "tool": "search", "parameters": "X", "depends_on": []}},
  {{"step_number": 2, "description": "Calculate Y", "tool": "calculate", "parameters": "Y", "depends_on": [1]}}
]

Return ONLY the JSON array, no other text.
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0.3
        )

        # Parse the plan
        try:
            result = json.loads(response.choices[0].message.content)
            # Handle both direct array and object with 'plan' key
            if isinstance(result, dict) and 'plan' in result:
                return result['plan']
            elif isinstance(result, dict) and 'steps' in result:
                return result['steps']
            return result
        except:
            return []

    def execute_plan(self, plan: List[Dict]) -> Dict:
        """Execute the plan step by step"""
        results = {}

        print("\n📋 Plan created:")
        for step in plan:
            print(f"  Step {step['step_number']}: {step['description']}")

        print("\n🚀 Executing plan...\n")

        for step in plan:
            step_num = step['step_number']
            print(f"Step {step_num}: {step['description']}")

            # Check dependencies
            for dep in step.get('depends_on', []):
                if dep not in results:
                    print(f"  ⚠️  Waiting for step {dep}...")
                    results[step_num] = {"error": f"Dependency step {dep} not completed"}
                    continue

            # Execute tool if specified
            if step['tool'] != "none" and step['tool'] in self.tools:
                try:
                    result = self.tools[step['tool']]['function'](step['parameters'])
                    results[step_num] = {"success": True, "result": result}
                    print(f"  ✓ Result: {result}")
                except Exception as e:
                    results[step_num] = {"success": False, "error": str(e)}
                    print(f"  ✗ Error: {e}")
            else:
                results[step_num] = {"success": True, "result": "No tool needed"}
                print(f"  ✓ Completed")

            print()

        return results

    def run(self, goal: str) -> str:
        """Plan and execute to achieve goal"""
        # Planning phase
        print(f"🎯 Goal: {goal}\n")
        print("🤔 Creating plan...")

        plan = self.create_plan(goal)

        if not plan:
            return "Failed to create a valid plan"

        # Execution phase
        results = self.execute_plan(plan)

        # Summarize
        summary_prompt = f"""Summarize the results of executing this plan:

Goal: {goal}

Plan: {json.dumps(plan, indent=2)}

Results: {json.dumps(results, indent=2)}

Provide a brief summary of what was accomplished.
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": summary_prompt}],
            temperature=0.5
        )

        return response.choices[0].message.content

# Define tools (same as before)
def search_web(query: str) -> str:
    return f"Search results for: {query}"

def calculate(expression: str) -> str:
    try:
        result = eval(expression, {"__builtins__": {}}, {})
        return str(result)
    except Exception as e:
        return f"Error: {e}"

def save_to_file(content: str) -> str:
    return f"Saved: {content[:50]}..."

tools = {
    "search": {"description": "Search for information", "function": search_web},
    "calculate": {"description": "Calculate math", "function": calculate},
    "save": {"description": "Save content to file", "function": save_to_file}
}

# Run
agent = PlanExecuteAgent(tools)
result = agent.run("Research Python programming, calculate how many years since it was created (2024 - 1991), and save a summary")
print(f"\n📊 Final Summary:\n{result}")
```

### Exercise 3: Self-Reflection Agent

```python
# reflection_agent.py
from openai import OpenAI
from typing import Dict, List

client = OpenAI()

class ReflectionAgent:
    """Agent that reflects on and improves its outputs"""

    def __init__(self, max_reflections: int = 3):
        self.max_reflections = max_reflections

    def generate(self, task: str) -> str:
        """Generate initial output"""
        prompt = f"""Complete this task:

{task}

Provide your best answer.
"""
        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7
        )

        return response.choices[0].message.content

    def reflect(self, task: str, output: str) -> Dict:
        """Reflect on the output"""
        prompt = f"""Review this output for the given task:

Task: {task}

Output:
{output}

Provide:
1. Quality score (1-10)
2. Specific issues or weaknesses
3. Suggestions for improvement

Format as JSON:
{{
  "score": 8,
  "issues": ["issue 1", "issue 2"],
  "suggestions": ["suggestion 1", "suggestion 2"]
}}
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0.3
        )

        import json
        return json.loads(response.choices[0].message.content)

    def improve(self, task: str, output: str, reflection: Dict) -> str:
        """Improve output based on reflection"""
        prompt = f"""Improve this output based on the reflection:

Task: {task}

Current Output:
{output}

Reflection:
- Score: {reflection['score']}/10
- Issues: {', '.join(reflection['issues'])}
- Suggestions: {', '.join(reflection['suggestions'])}

Provide an improved version addressing these points.
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7
        )

        return response.choices[0].message.content

    def run(self, task: str) -> str:
        """Generate with reflection loop"""
        print(f"📝 Task: {task}\n")

        # Initial generation
        output = self.generate(task)
        print(f"Initial output:\n{output}\n")

        # Reflection loop
        for i in range(self.max_reflections):
            print(f"{'='*60}")
            print(f"Reflection {i+1}/{self.max_reflections}")
            print(f"{'='*60}\n")

            # Reflect
            reflection = self.reflect(task, output)
            print(f"Score: {reflection['score']}/10")
            print(f"Issues: {reflection['issues']}")
            print(f"Suggestions: {reflection['suggestions']}\n")

            # Check if good enough
            if reflection['score'] >= 9:
                print("✓ Output quality sufficient!")
                break

            # Improve
            output = self.improve(task, output, reflection)
            print(f"Improved output:\n{output}\n")

        return output

# Test
agent = ReflectionAgent(max_reflections=2)
task = "Write a Python function to find prime numbers up to n. Include error handling and documentation."
final_output = agent.run(task)
```

### Exercise 4: Comparing Architectures

```python
# architecture_comparison.py
import time
from typing import Dict, Callable

class ArchitectureBenchmark:
    """Compare different agent architectures"""

    def __init__(self):
        self.metrics = {}

    def benchmark(self, name: str, agent_func: Callable, task: str) -> Dict:
        """Benchmark an architecture"""
        print(f"\n{'='*60}")
        print(f"Testing: {name}")
        print(f"{'='*60}\n")

        start_time = time.time()
        iterations = 0
        tools_used = []

        # Run agent (simplified - track metrics)
        result = agent_func(task)

        end_time = time.time()

        metrics = {
            "name": name,
            "time": end_time - start_time,
            "result_length": len(result),
            "success": len(result) > 0
        }

        self.metrics[name] = metrics
        return metrics

    def compare(self):
        """Compare all benchmarked architectures"""
        print(f"\n{'='*60}")
        print("Architecture Comparison")
        print(f"{'='*60}\n")

        for name, metrics in self.metrics.items():
            print(f"{name}:")
            print(f"  Time: {metrics['time']:.2f}s")
            print(f"  Success: {metrics['success']}")
            print(f"  Output length: {metrics['result_length']} chars")
            print()

# Usage
benchmark = ArchitectureBenchmark()

# Define simple agents for comparison
def simple_agent(task):
    return f"Simple response to: {task}"

def react_agent_wrapper(task):
    # Use ReAct agent from earlier
    return f"ReAct response to: {task}"

def plan_execute_wrapper(task):
    # Use Plan-Execute agent
    return f"Plan-Execute response to: {task}"

task = "Research AI and calculate ROI"

benchmark.benchmark("Simple Reflex", simple_agent, task)
benchmark.benchmark("ReAct Pattern", react_agent_wrapper, task)
benchmark.benchmark("Plan-Execute", plan_execute_wrapper, task)

benchmark.compare()
```

## Architecture Selection Guide

| Use Case | Recommended Architecture | Why |
|----------|------------------------|-----|
| Simple Q&A | Simple Reflex | Fast, cheap, sufficient |
| Research tasks | ReAct | Transparent, debuggable |
| Complex workflows | Plan-Execute | Structured, reliable |
| Quality-critical | Reflection | Self-improving, high quality |
| Multi-domain | Hierarchical | Specialized, scalable |

## Key Takeaways

1. Different architectures suit different use cases
2. ReAct is the most versatile general-purpose pattern
3. Plan-Execute is best for complex, multi-step workflows
4. Reflection improves quality at the cost of speed/cost
5. Architecture choice impacts performance, cost, and reliability
6. Hybrid approaches often work best

## Resources

- **Papers**:
  - "ReAct: Synergizing Reasoning and Acting in Language Models" (Yao et al., 2022)
  - "Chain-of-Thought Prompting" (Wei et al., 2022)
  - "Reflexion: Language Agents with Verbal Reinforcement Learning" (Shinn et al., 2023)
- **Frameworks**:
  - [LangGraph](https://github.com/langchain-ai/langgraph) - Graph-based agents
  - [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) - Autonomous agents

## Daily Challenge

**Build a Hybrid Agent**

Combine multiple patterns:
1. Use Plan-Execute for overall structure
2. Use ReAct for individual steps
3. Add Reflection for quality control

Test with: "Create a comprehensive report on renewable energy, including research, statistics, and recommendations"

## Reflection Questions

1. When would you choose ReAct over Plan-Execute?
2. How does architecture affect debugging difficulty?
3. What are the cost implications of each pattern?
4. How would you test different architectures objectively?

## Tomorrow's Preview

Day 7: Week 1 Project - CLI Assistant Agent - Build a complete, production-ready CLI assistant using the patterns learned this week.

---

**Progress**: 6/30 days completed

[← Previous: Day 5](./day-05.md) | [Back to Overview](../README.md) | [Next: Day 7 →](./day-07.md)
