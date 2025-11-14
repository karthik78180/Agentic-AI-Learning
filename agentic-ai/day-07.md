# Day 7: Week 1 Project - CLI Assistant Agent

## Overview
Today you'll build a comprehensive CLI assistant that combines all the concepts from Week 1: LLM APIs, prompt engineering, function calling, and agent architectures. This is your Week 1 capstone project.

## Project: Advanced CLI Assistant

### Features
- Multi-turn conversations with context
- Web search and research capabilities
- File system operations
- Code execution and analysis
- Task management
- Note-taking and organization
- Multiple agent modes (ReAct, Plan-Execute)

### Requirements
- Clean, modular code architecture
- Comprehensive error handling
- Persistent storage
- Extensible tool system
- User-friendly interface
- Cost tracking

## Implementation

### Project Structure

```
cli_assistant/
├── main.py                 # Entry point
├── config/
│   ├── __init__.py
│   └── settings.py         # Configuration
├── agents/
│   ├── __init__.py
│   ├── base_agent.py       # Base agent class
│   ├── react_agent.py      # ReAct implementation
│   └── plan_agent.py       # Plan-Execute implementation
├── tools/
│   ├── __init__.py
│   ├── registry.py         # Tool registry
│   ├── search_tools.py     # Web search
│   ├── file_tools.py       # File operations
│   ├── code_tools.py       # Code execution
│   └── task_tools.py       # Task management
├── memory/
│   ├── __init__.py
│   ├── conversation.py     # Conversation memory
│   └── storage.py          # Persistent storage
├── ui/
│   ├── __init__.py
│   └── cli.py              # CLI interface
├── utils/
│   ├── __init__.py
│   ├── cost_tracker.py     # Cost tracking
│   └── logger.py           # Logging
└── tests/
    ├── test_agents.py
    ├── test_tools.py
    └── test_integration.py
```

### Core Implementation

```python
# config/settings.py
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

class Settings:
    # API Configuration
    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")

    # Model Configuration
    DEFAULT_MODEL = "gpt-4-turbo-preview"
    TEMPERATURE = 0.7
    MAX_TOKENS = 2000

    # Agent Configuration
    MAX_ITERATIONS = 15
    MAX_HISTORY = 50
    DEFAULT_MODE = "react"  # or "plan"

    # Storage Configuration
    BASE_DIR = Path.home() / ".cli_assistant"
    DATA_DIR = BASE_DIR / "data"
    NOTES_DIR = BASE_DIR / "notes"
    TASKS_DIR = BASE_DIR / "tasks"
    LOGS_DIR = BASE_DIR / "logs"

    # Cost Tracking
    TRACK_COSTS = True
    COST_WARNING_THRESHOLD = 5.0  # dollars

    @classmethod
    def setup(cls):
        """Initialize directories and validate config"""
        for dir_path in [cls.BASE_DIR, cls.DATA_DIR, cls.NOTES_DIR,
                        cls.TASKS_DIR, cls.LOGS_DIR]:
            dir_path.mkdir(parents=True, exist_ok=True)

        if not cls.OPENAI_API_KEY:
            raise ValueError("OPENAI_API_KEY not set")

# tools/registry.py
from typing import Dict, Callable, Any
import json

class ToolRegistry:
    """Central registry for all agent tools"""

    def __init__(self):
        self._tools = {}

    def register(self, name: str, description: str,
                parameters: Dict, function: Callable):
        """Register a new tool"""
        self._tools[name] = {
            "name": name,
            "description": description,
            "parameters": parameters,
            "function": function
        }

    def get_schemas(self) -> list:
        """Get OpenAI-compatible schemas"""
        return [
            {
                "type": "function",
                "function": {
                    "name": tool["name"],
                    "description": tool["description"],
                    "parameters": tool["parameters"]
                }
            }
            for tool in self._tools.values()
        ]

    def execute(self, name: str, **kwargs) -> Any:
        """Execute a tool"""
        if name not in self._tools:
            raise ValueError(f"Tool not found: {name}")

        return self._tools[name]["function"](**kwargs)

    def list_tools(self) -> Dict:
        """List all available tools"""
        return {
            name: {
                "description": tool["description"],
                "parameters": list(tool["parameters"]["properties"].keys())
            }
            for name, tool in self._tools.items()
        }

# tools/search_tools.py
import requests
from datetime import datetime

def register_search_tools(registry: ToolRegistry):
    """Register search-related tools"""

    def web_search(query: str, num_results: int = 5) -> dict:
        """Search the web for information"""
        # In production: integrate with SerpAPI, Bing, etc.
        return {
            "query": query,
            "results": [
                {
                    "title": f"Result {i+1}",
                    "snippet": f"Information about {query}...",
                    "url": f"https://example.com/{i+1}"
                }
                for i in range(num_results)
            ],
            "timestamp": datetime.now().isoformat()
        }

    def get_news(topic: str, days: int = 7) -> dict:
        """Get recent news on a topic"""
        return {
            "topic": topic,
            "articles": [
                {
                    "title": f"News about {topic}",
                    "source": "News Source",
                    "date": datetime.now().isoformat()
                }
            ]
        }

    registry.register(
        "web_search",
        "Search the web for information on any topic",
        {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"},
                "num_results": {"type": "integer", "description": "Number of results (default: 5)"}
            },
            "required": ["query"]
        },
        web_search
    )

    registry.register(
        "get_news",
        "Get recent news articles on a topic",
        {
            "type": "object",
            "properties": {
                "topic": {"type": "string", "description": "News topic"},
                "days": {"type": "integer", "description": "Days to look back (default: 7)"}
            },
            "required": ["topic"]
        },
        get_news
    )

# tools/file_tools.py
from pathlib import Path
import json

def register_file_tools(registry: ToolRegistry):
    """Register file operation tools"""

    def save_note(title: str, content: str, tags: list = None) -> dict:
        """Save a note to file"""
        from config.settings import Settings

        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{timestamp}_{title.replace(' ', '_')}.md"
        filepath = Settings.NOTES_DIR / filename

        metadata = {
            "title": title,
            "created": datetime.now().isoformat(),
            "tags": tags or []
        }

        with open(filepath, 'w') as f:
            f.write(f"# {title}\n\n")
            f.write(f"*Created: {metadata['created']}*\n\n")
            if tags:
                f.write(f"*Tags: {', '.join(tags)}*\n\n")
            f.write(content)

        # Save metadata
        meta_file = filepath.with_suffix('.json')
        with open(meta_file, 'w') as f:
            json.dump(metadata, f, indent=2)

        return {
            "success": True,
            "filepath": str(filepath),
            "filename": filename
        }

    def list_notes(tag: str = None) -> dict:
        """List all saved notes"""
        from config.settings import Settings

        notes = []
        for md_file in Settings.NOTES_DIR.glob("*.md"):
            meta_file = md_file.with_suffix('.json')
            if meta_file.exists():
                with open(meta_file) as f:
                    metadata = json.load(f)

                # Filter by tag if specified
                if tag and tag not in metadata.get('tags', []):
                    continue

                notes.append({
                    "filename": md_file.name,
                    "title": metadata["title"],
                    "created": metadata["created"],
                    "tags": metadata.get("tags", [])
                })

        return {
            "count": len(notes),
            "notes": sorted(notes, key=lambda x: x['created'], reverse=True)
        }

    def read_file(filepath: str) -> dict:
        """Read file contents"""
        try:
            path = Path(filepath)
            with open(path, 'r') as f:
                content = f.read()

            return {
                "success": True,
                "filepath": str(path),
                "content": content,
                "size": path.stat().st_size
            }
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }

    registry.register(
        "save_note",
        "Save a note with title, content, and optional tags",
        {
            "type": "object",
            "properties": {
                "title": {"type": "string", "description": "Note title"},
                "content": {"type": "string", "description": "Note content"},
                "tags": {"type": "array", "items": {"type": "string"}, "description": "Tags for organization"}
            },
            "required": ["title", "content"]
        },
        save_note
    )

    registry.register(
        "list_notes",
        "List all saved notes, optionally filtered by tag",
        {
            "type": "object",
            "properties": {
                "tag": {"type": "string", "description": "Filter by tag"}
            }
        },
        list_notes
    )

    registry.register(
        "read_file",
        "Read the contents of a file",
        {
            "type": "object",
            "properties": {
                "filepath": {"type": "string", "description": "Path to file"}
            },
            "required": ["filepath"]
        },
        read_file
    )

# tools/task_tools.py
import uuid
import json
from datetime import datetime
from pathlib import Path

class TaskManager:
    """Manage tasks and todos"""

    def __init__(self):
        from config.settings import Settings
        self.tasks_file = Settings.DATA_DIR / "tasks.json"
        self.tasks = self._load_tasks()

    def _load_tasks(self):
        if self.tasks_file.exists():
            with open(self.tasks_file) as f:
                return json.load(f)
        return []

    def _save_tasks(self):
        with open(self.tasks_file, 'w') as f:
            json.dump(self.tasks, f, indent=2)

    def add_task(self, description: str, priority: str = "medium") -> dict:
        task = {
            "id": str(uuid.uuid4())[:8],
            "description": description,
            "priority": priority,
            "status": "pending",
            "created": datetime.now().isoformat(),
            "completed": None
        }
        self.tasks.append(task)
        self._save_tasks()
        return task

    def list_tasks(self, status: str = None) -> list:
        if status:
            return [t for t in self.tasks if t["status"] == status]
        return self.tasks

    def complete_task(self, task_id: str) -> dict:
        for task in self.tasks:
            if task["id"] == task_id:
                task["status"] = "completed"
                task["completed"] = datetime.now().isoformat()
                self._save_tasks()
                return task
        raise ValueError(f"Task not found: {task_id}")

def register_task_tools(registry: ToolRegistry):
    """Register task management tools"""
    manager = TaskManager()

    registry.register(
        "add_task",
        "Add a new task to the todo list",
        {
            "type": "object",
            "properties": {
                "description": {"type": "string", "description": "Task description"},
                "priority": {"type": "string", "enum": ["low", "medium", "high"], "description": "Priority level"}
            },
            "required": ["description"]
        },
        manager.add_task
    )

    registry.register(
        "list_tasks",
        "List tasks, optionally filtered by status",
        {
            "type": "object",
            "properties": {
                "status": {"type": "string", "enum": ["pending", "completed"], "description": "Filter by status"}
            }
        },
        manager.list_tasks
    )

    registry.register(
        "complete_task",
        "Mark a task as completed",
        {
            "type": "object",
            "properties": {
                "task_id": {"type": "string", "description": "Task ID to complete"}
            },
            "required": ["task_id"]
        },
        manager.complete_task
    )

# agents/base_agent.py
from abc import ABC, abstractmethod
from typing import Dict, Any

class BaseAgent(ABC):
    """Base class for all agents"""

    def __init__(self, tools_registry: ToolRegistry):
        self.tools = tools_registry

    @abstractmethod
    def process(self, user_input: str) -> str:
        """Process user input and return response"""
        pass

    @abstractmethod
    def reset(self):
        """Reset agent state"""
        pass

# utils/cost_tracker.py
class CostTracker:
    """Track API usage and costs"""

    def __init__(self):
        self.total_cost = 0.0
        self.total_tokens = 0
        self.calls = []

    def track_call(self, model: str, input_tokens: int, output_tokens: int):
        """Track an API call"""
        # Pricing (update with current rates)
        prices = {
            "gpt-4-turbo-preview": {"input": 0.01, "output": 0.03},
            "gpt-3.5-turbo": {"input": 0.0005, "output": 0.0015}
        }

        if model in prices:
            cost = (input_tokens / 1000 * prices[model]["input"] +
                   output_tokens / 1000 * prices[model]["output"])

            self.total_cost += cost
            self.total_tokens += input_tokens + output_tokens

            self.calls.append({
                "model": model,
                "input_tokens": input_tokens,
                "output_tokens": output_tokens,
                "cost": cost
            })

    def get_summary(self) -> dict:
        """Get cost summary"""
        return {
            "total_cost": round(self.total_cost, 4),
            "total_tokens": self.total_tokens,
            "total_calls": len(self.calls),
            "average_cost_per_call": round(self.total_cost / len(self.calls), 4) if self.calls else 0
        }

# main.py
from config.settings import Settings
from tools.registry import ToolRegistry
from tools.search_tools import register_search_tools
from tools.file_tools import register_file_tools
from tools.task_tools import register_task_tools
from ui.cli import CLI

def main():
    """Main entry point"""
    # Setup
    Settings.setup()

    # Initialize tool registry
    registry = ToolRegistry()
    register_search_tools(registry)
    register_file_tools(registry)
    register_task_tools(registry)

    # Start CLI
    cli = CLI(registry)
    cli.run()

if __name__ == "__main__":
    main()
```

## Testing Strategy

### Unit Tests
```python
# tests/test_tools.py
import pytest
from tools.registry import ToolRegistry

def test_tool_registration():
    registry = ToolRegistry()

    def test_func(param: str):
        return f"Result: {param}"

    registry.register(
        "test_tool",
        "A test tool",
        {"type": "object", "properties": {"param": {"type": "string"}}},
        test_func
    )

    assert "test_tool" in registry._tools
    result = registry.execute("test_tool", param="hello")
    assert result == "Result: hello"

# tests/test_integration.py
def test_full_workflow():
    """Test complete agent workflow"""
    # Setup
    # Create agent
    # Execute task
    # Verify results
    pass
```

## Deployment Checklist

- [ ] All environment variables set
- [ ] Dependencies installed
- [ ] Tests passing
- [ ] Error handling comprehensive
- [ ] Logging configured
- [ ] Cost tracking enabled
- [ ] Documentation complete
- [ ] User guide written

## Enhancement Ideas

1. **Voice Interface**: Add speech-to-text and text-to-speech
2. **Web UI**: Create a web dashboard
3. **Plugins**: Support for custom plugins
4. **Integrations**: Connect to Slack, Discord, etc.
5. **Analytics**: Usage analytics and insights
6. **Collaboration**: Multi-user support
7. **Scheduling**: Automated task execution
8. **Mobile App**: Mobile companion app

## Key Takeaways - Week 1

1. **LLM Fundamentals**: Understanding how LLMs work is crucial
2. **Prompt Engineering**: Quality prompts = quality outputs
3. **Function Calling**: Tools enable real-world actions
4. **Architecture Matters**: Choose the right pattern for the task
5. **State Management**: Memory is essential for coherent agents
6. **Error Handling**: Robust error handling is non-negotiable
7. **User Experience**: Interface design impacts usability

## Resources

- [OpenAI Best Practices](https://platform.openai.com/docs/guides/production-best-practices)
- [LangChain Templates](https://github.com/langchain-ai/langchain/tree/master/templates)
- [Agent Examples](https://github.com/topics/ai-agents)

## Week 1 Reflection

1. What was the most challenging concept this week?
2. Which agent pattern do you prefer and why?
3. What real-world problem could you solve with an agent?
4. What do you want to learn more about?

## Next Week Preview

**Week 2: Core Agent Capabilities**
- Day 8: ReAct Pattern Deep Dive
- Day 9: Memory Systems
- Day 10: Planning and Task Decomposition
- Day 11-12: RAG and Vector Databases
- Day 13: Error Handling
- Day 14: Week 2 Project

---

**Progress**: 7/30 days completed | Week 1 Complete! 🎉

[← Previous: Day 6](./day-06.md) | [Back to Overview](../README.md) | [Next: Day 8 →](./day-08.md)
