# Day 10: Planning and Task Decomposition

## Overview
Planning is what enables agents to tackle complex, multi-step tasks. Today you'll learn how agents break down goals into executable plans and adapt those plans based on results.

## Learning Objectives
- Understand planning strategies for agents
- Implement task decomposition algorithms
- Build plan-and-execute systems
- Handle plan failures and replanning
- Optimize planning for efficiency
- Compare planning approaches

## Theoretical Concepts

### What is Planning?

Planning transforms a high-level goal into a sequence of actionable steps:

```
Goal: "Create a weather dashboard"
    ↓
Plan:
1. Research weather APIs
2. Design UI mockup
3. Set up project structure
4. Implement API integration
5. Build frontend components
6. Add error handling
7. Test and deploy
```

### Planning Approaches

#### 1. Forward Planning
- Start from current state
- Apply actions to reach goal
- Breadth-first or depth-first search

#### 2. Backward Planning (Goal Regression)
- Start from goal
- Work backwards to current state
- Find prerequisite steps

#### 3. Hierarchical Task Network (HTN)
- Break complex tasks into subtasks
- Recursively decompose until primitive actions
- Natural for complex domains

#### 4. Reactive Planning
- No upfront plan
- React to observations
- Suitable for uncertain environments

### Task Decomposition Strategies

1. **Sequential**: Tasks must be done in order
2. **Parallel**: Tasks can be done simultaneously
3. **Conditional**: Tasks depend on conditions
4. **Iterative**: Tasks repeat until condition met

## Hands-On Exercises

### Exercise 1: Hierarchical Task Decomposition

```python
# task_decomposition.py
from openai import OpenAI
from typing import List, Dict
import json

client = OpenAI()

class TaskDecomposer:
    """Decompose complex tasks into subtasks"""

    def decompose(self, goal: str, max_depth: int = 3) -> Dict:
        """
        Decompose a goal into hierarchical tasks

        Returns:
            Task tree with subtasks
        """
        prompt = f"""Break down this goal into a hierarchical task structure:

Goal: {goal}

Return a JSON structure with:
- task: main task description
- subtasks: array of subtasks (each can have their own subtasks)
- estimated_time: estimated time in minutes
- dependencies: array of task IDs this depends on
- type: "sequential", "parallel", or "conditional"

Example format:
{{
  "task": "Main goal",
  "type": "sequential",
  "estimated_time": 120,
  "subtasks": [
    {{
      "id": "1",
      "task": "Subtask 1",
      "type": "sequential",
      "estimated_time": 30,
      "dependencies": [],
      "subtasks": [
        {{"id": "1.1", "task": "Sub-subtask", "type": "atomic", "estimated_time": 15, "dependencies": []}}
      ]
    }},
    {{
      "id": "2",
      "task": "Subtask 2",
      "type": "atomic",
      "estimated_time": 45,
      "dependencies": ["1"]
    }}
  ]
}}

Return only valid JSON.
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0.3
        )

        return json.loads(response.choices[0].message.content)

    def flatten_tasks(self, task_tree: Dict, parent_id: str = "") -> List[Dict]:
        """Flatten hierarchical tasks into a list"""
        tasks = []

        def traverse(node, prefix=""):
            task_id = node.get("id", prefix)
            tasks.append({
                "id": task_id,
                "task": node["task"],
                "type": node.get("type", "sequential"),
                "estimated_time": node.get("estimated_time", 0),
                "dependencies": node.get("dependencies", []),
                "parent": prefix.rsplit(".", 1)[0] if "." in prefix else None
            })

            for i, subtask in enumerate(node.get("subtasks", [])):
                sub_id = f"{task_id}.{i+1}" if task_id else str(i+1)
                subtask["id"] = sub_id
                traverse(subtask, sub_id)

        traverse(task_tree)
        return tasks

# Usage
decomposer = TaskDecomposer()

goal = "Build a full-stack task management application with authentication"
task_tree = decomposer.decompose(goal)

print("Hierarchical Task Structure:")
print(json.dumps(task_tree, indent=2))

print("\n\nFlattened Task List:")
flat_tasks = decomposer.flatten_tasks(task_tree)
for task in flat_tasks:
    indent = "  " * task["id"].count(".")
    print(f"{indent}{task['id']}: {task['task']} ({task['estimated_time']}min)")
```

### Exercise 2: Plan-Execute-Replan Agent

```python
# plan_execute_replan.py
from typing import List, Dict, Optional
from enum import Enum
import json

class TaskStatus(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    FAILED = "failed"
    BLOCKED = "blocked"

class Task:
    def __init__(self, id: str, description: str, dependencies: List[str] = None):
        self.id = id
        self.description = description
        self.dependencies = dependencies or []
        self.status = TaskStatus.PENDING
        self.result: Optional[str] = None
        self.error: Optional[str] = None

class PlanExecuteReplanAgent:
    """Agent that plans, executes, and replans when needed"""

    def __init__(self, tools: Dict):
        self.tools = tools
        self.tasks: List[Task] = []
        self.completed_tasks: List[str] = []

    def create_plan(self, goal: str) -> List[Task]:
        """Create initial plan"""
        prompt = f"""Create a step-by-step plan to achieve this goal:

Goal: {goal}

Available tools: {list(self.tools.keys())}

Return a JSON array of tasks:
[
  {{"id": "1", "description": "...", "tool": "tool_name", "dependencies": []}},
  {{"id": "2", "description": "...", "tool": "tool_name", "dependencies": ["1"]}}
]
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0.3
        )

        plan_data = json.loads(response.choices[0].message.content)
        tasks = [
            Task(t["id"], t["description"], t.get("dependencies", []))
            for t in plan_data.get("tasks", plan_data.get("plan", []))
        ]

        return tasks

    def can_execute(self, task: Task) -> bool:
        """Check if task dependencies are met"""
        return all(dep in self.completed_tasks for dep in task.dependencies)

    def execute_task(self, task: Task) -> bool:
        """Execute a single task"""
        print(f"\n🔨 Executing: {task.id} - {task.description}")

        try:
            task.status = TaskStatus.IN_PROGRESS

            # Simulated execution
            # In real implementation, this would use tools
            result = f"Completed: {task.description}"

            task.result = result
            task.status = TaskStatus.COMPLETED
            self.completed_tasks.append(task.id)

            print(f"   ✓ Success: {result}")
            return True

        except Exception as e:
            task.error = str(e)
            task.status = TaskStatus.FAILED
            print(f"   ✗ Failed: {e}")
            return False

    def replan(self, failed_task: Task, goal: str) -> List[Task]:
        """Create new plan after failure"""
        print(f"\n🔄 Replanning due to failure of task {failed_task.id}...")

        prompt = f"""A task failed. Create an alternative plan:

Original Goal: {goal}

Failed Task: {failed_task.description}
Error: {failed_task.error}

Completed Tasks: {self.completed_tasks}

Create an alternative plan to achieve the goal, avoiding the failure.
Return JSON array of new tasks.
"""

        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0.5
        )

        plan_data = json.loads(response.choices[0].message.content)
        new_tasks = [
            Task(t["id"], t["description"], t.get("dependencies", []))
            for t in plan_data.get("tasks", [])
        ]

        return new_tasks

    def execute_plan(self, goal: str, max_replans: int = 2) -> bool:
        """Execute plan with replanning on failure"""
        print(f"🎯 Goal: {goal}\n")

        # Create initial plan
        self.tasks = self.create_plan(goal)
        print(f"📋 Initial plan: {len(self.tasks)} tasks")

        replans = 0

        while self.tasks and replans <= max_replans:
            # Find next executable task
            next_task = None
            for task in self.tasks:
                if task.status == TaskStatus.PENDING and self.can_execute(task):
                    next_task = task
                    break

            if not next_task:
                # No executable tasks, check if all complete
                if all(t.status == TaskStatus.COMPLETED for t in self.tasks):
                    print(f"\n✅ Goal achieved! Completed {len(self.completed_tasks)} tasks")
                    return True

                # Check for blocked tasks
                pending = [t for t in self.tasks if t.status == TaskStatus.PENDING]
                if pending:
                    print(f"\n⚠️  Tasks blocked, cannot proceed")
                    return False

            # Execute task
            success = self.execute_task(next_task)

            if not success:
                # Replan
                if replans < max_replans:
                    self.tasks = self.replan(next_task, goal)
                    replans += 1
                else:
                    print(f"\n❌ Max replans reached, giving up")
                    return False

        return False

# Example usage
tools = {
    "search": lambda q: f"Results for {q}",
    "code": lambda spec: f"Code for {spec}",
    "test": lambda code: "Tests passed"
}

agent = PlanExecuteReplanAgent(tools)
agent.execute_plan("Create a Python web scraper for news articles")
```

### Exercise 3: Constraint-Based Planning

```python
# constraint_planning.py
from typing import List, Dict, Set
from dataclasses import dataclass

@dataclass
class Constraint:
    """Represents a planning constraint"""
    type: str  # "before", "after", "parallel", "mutex"
    task1: str
    task2: str

class ConstraintPlanner:
    """Plan with constraints"""

    def __init__(self):
        self.tasks: Dict[str, Dict] = {}
        self.constraints: List[Constraint] = []

    def add_task(self, id: str, description: str, duration: int):
        """Add a task"""
        self.tasks[id] = {
            "id": id,
            "description": description,
            "duration": duration
        }

    def add_constraint(self, type: str, task1: str, task2: str):
        """Add a constraint"""
        self.constraints.append(Constraint(type, task1, task2))

    def topological_sort(self) -> List[str]:
        """Sort tasks respecting constraints"""
        # Build dependency graph
        deps: Dict[str, Set[str]] = {tid: set() for tid in self.tasks}

        for constraint in self.constraints:
            if constraint.type == "before":
                # task1 must be before task2
                deps[constraint.task2].add(constraint.task1)

        # Kahn's algorithm
        in_degree = {tid: len(deps[tid]) for tid in self.tasks}
        queue = [tid for tid, degree in in_degree.items() if degree == 0]
        result = []

        while queue:
            task = queue.pop(0)
            result.append(task)

            # Remove this task from dependencies
            for tid in self.tasks:
                if task in deps[tid]:
                    deps[tid].remove(task)
                    in_degree[tid] -= 1
                    if in_degree[tid] == 0:
                        queue.append(tid)

        if len(result) != len(self.tasks):
            raise ValueError("Circular dependency detected")

        return result

    def get_execution_plan(self) -> List[Dict]:
        """Get optimized execution plan"""
        order = self.topological_sort()

        plan = []
        for tid in order:
            task = self.tasks[tid]
            plan.append({
                "order": len(plan) + 1,
                "id": task["id"],
                "description": task["description"],
                "duration": task["duration"]
            })

        return plan

# Usage
planner = ConstraintPlanner()

planner.add_task("A", "Design database schema", 60)
planner.add_task("B", "Set up development environment", 30)
planner.add_task("C", "Implement API endpoints", 120)
planner.add_task("D", "Write tests", 90)
planner.add_task("E", "Deploy to staging", 45)

planner.add_constraint("before", "A", "C")  # Design before implementation
planner.add_constraint("before", "B", "C")  # Environment before implementation
planner.add_constraint("before", "C", "D")  # Implement before testing
planner.add_constraint("before", "D", "E")  # Test before deployment

plan = planner.get_execution_plan()

print("Optimized Execution Plan:")
total_time = 0
for step in plan:
    print(f"{step['order']}. {step['description']} ({step['duration']} min)")
    total_time += step['duration']

print(f"\nTotal estimated time: {total_time} minutes ({total_time/60:.1f} hours)")
```

## Key Takeaways

1. Planning transforms goals into actionable steps
2. Hierarchical decomposition handles complex tasks
3. Dependencies and constraints guide execution order
4. Replanning enables adaptation to failures
5. Good planning improves efficiency and success rate

## Resources

- [Classical Planning Problems](https://en.wikipedia.org/wiki/Automated_planning_and_scheduling)
- [HTN Planning](https://en.wikipedia.org/wiki/Hierarchical_task_network)
- [LangChain Plan-and-Execute](https://python.langchain.com/docs/use_cases/more/agents/plan_and_execute)

## Daily Challenge

Build a meal planning agent that:
1. Takes dietary preferences and constraints
2. Creates a weekly meal plan
3. Generates shopping list
4. Handles ingredient substitutions
5. Optimizes for nutrition and budget

## Tomorrow's Preview

Day 11: RAG - Retrieval Augmented Generation - Learn how to give agents access to large knowledge bases through retrieval.

---

**Progress**: 10/30 days completed | 1/3 Complete!

[← Previous: Day 9](./day-09.md) | [Back to Overview](../README.md) | [Next: Day 11 →](./day-11.md)
