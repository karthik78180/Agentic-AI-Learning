# Day 5: Building Your First Simple Agent

## Overview
Today you'll build your first complete agent from scratch, combining everything you've learned: LLM APIs, prompt engineering, and function calling. This is a milestone day where concepts become reality.

## Learning Objectives
- Build an end-to-end agent system
- Implement the agent execution loop
- Handle conversation state and memory
- Create a user interface (CLI)
- Debug and test agent behavior
- Deploy a working agent

## Project: CLI Research Assistant

We'll build a research assistant agent that can:
- Search the web for information
- Summarize findings
- Save notes to files
- Answer follow-up questions with context

### Architecture

```
User Input
    ↓
CLI Interface
    ↓
Agent Controller (orchestration)
    ↓
LLM (reasoning) ←→ Tools (actions)
    ↓
State Manager (memory)
    ↓
Output Formatter
    ↓
User
```

## Implementation

### Step 1: Project Setup

```python
# project_structure.py
"""
research_assistant/
├── agent.py          # Main agent logic
├── tools.py          # Tool implementations
├── memory.py         # State management
├── cli.py            # User interface
├── config.py         # Configuration
└── requirements.txt  # Dependencies
"""
```

### Step 2: Configuration

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    # API Keys
    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

    # Model Settings
    MODEL = "gpt-4-turbo-preview"
    TEMPERATURE = 0.7
    MAX_TOKENS = 2000

    # Agent Settings
    MAX_ITERATIONS = 10
    MAX_HISTORY = 20

    # File Settings
    NOTES_DIR = "./research_notes"

    @classmethod
    def validate(cls):
        """Validate configuration"""
        if not cls.OPENAI_API_KEY:
            raise ValueError("OPENAI_API_KEY not set")

        os.makedirs(cls.NOTES_DIR, exist_ok=True)
```

### Step 3: Tool Implementation

```python
# tools.py
import json
import requests
from datetime import datetime
from pathlib import Path
from config import Config

class ResearchTools:
    """Tools for the research assistant"""

    @staticmethod
    def web_search(query: str, num_results: int = 5) -> dict:
        """
        Search the web for information

        Args:
            query: Search query
            num_results: Number of results to return

        Returns:
            Dictionary with search results
        """
        # Simulated search (in production, use real API like SerpAPI, Bing, etc.)
        print(f"🔍 Searching for: {query}")

        # Mock results
        results = [
            {
                "title": f"Result {i+1} for '{query}'",
                "snippet": f"This is snippet {i+1} containing information about {query}...",
                "url": f"https://example.com/result{i+1}"
            }
            for i in range(num_results)
        ]

        return {
            "query": query,
            "num_results": len(results),
            "results": results
        }

    @staticmethod
    def save_note(title: str, content: str) -> dict:
        """
        Save a research note to file

        Args:
            title: Note title
            content: Note content

        Returns:
            Success status and file path
        """
        try:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filename = f"{timestamp}_{title.replace(' ', '_')}.md"
            filepath = Path(Config.NOTES_DIR) / filename

            with open(filepath, 'w') as f:
                f.write(f"# {title}\n\n")
                f.write(f"*Created: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}*\n\n")
                f.write(content)

            print(f"💾 Saved note: {filepath}")

            return {
                "success": True,
                "filepath": str(filepath),
                "message": f"Note saved successfully"
            }
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }

    @staticmethod
    def list_notes() -> dict:
        """
        List all saved research notes

        Returns:
            List of notes with metadata
        """
        notes_dir = Path(Config.NOTES_DIR)
        notes = []

        for filepath in notes_dir.glob("*.md"):
            notes.append({
                "filename": filepath.name,
                "created": datetime.fromtimestamp(filepath.stat().st_mtime).strftime('%Y-%m-%d %H:%M'),
                "size": filepath.stat().st_size
            })

        return {
            "count": len(notes),
            "notes": sorted(notes, key=lambda x: x['created'], reverse=True)
        }

    @staticmethod
    def read_note(filename: str) -> dict:
        """
        Read a saved note

        Args:
            filename: Name of the note file

        Returns:
            Note content
        """
        try:
            filepath = Path(Config.NOTES_DIR) / filename
            with open(filepath, 'r') as f:
                content = f.read()

            return {
                "success": True,
                "filename": filename,
                "content": content
            }
        except FileNotFoundError:
            return {
                "success": False,
                "error": f"Note not found: {filename}"
            }

    @staticmethod
    def get_tool_schemas():
        """Get OpenAI function schemas for all tools"""
        return [
            {
                "type": "function",
                "function": {
                    "name": "web_search",
                    "description": "Search the web for information on a topic",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "query": {
                                "type": "string",
                                "description": "The search query"
                            },
                            "num_results": {
                                "type": "integer",
                                "description": "Number of results to return (default: 5)"
                            }
                        },
                        "required": ["query"]
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "save_note",
                    "description": "Save research notes to a file",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "title": {
                                "type": "string",
                                "description": "Title of the note"
                            },
                            "content": {
                                "type": "string",
                                "description": "Content to save"
                            }
                        },
                        "required": ["title", "content"]
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "list_notes",
                    "description": "List all saved research notes",
                    "parameters": {
                        "type": "object",
                        "properties": {}
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "read_note",
                    "description": "Read the content of a saved note",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "filename": {
                                "type": "string",
                                "description": "Name of the note file"
                            }
                        },
                        "required": ["filename"]
                    }
                }
            }
        ]

    @staticmethod
    def execute(function_name: str, **kwargs):
        """Execute a tool by name"""
        tools_map = {
            "web_search": ResearchTools.web_search,
            "save_note": ResearchTools.save_note,
            "list_notes": ResearchTools.list_notes,
            "read_note": ResearchTools.read_note
        }

        if function_name not in tools_map:
            raise ValueError(f"Unknown tool: {function_name}")

        return tools_map[function_name](**kwargs)
```

### Step 4: Memory Management

```python
# memory.py
from typing import List, Dict
from collections import deque
from config import Config

class ConversationMemory:
    """Manages conversation history and context"""

    def __init__(self, max_history: int = Config.MAX_HISTORY):
        self.messages: deque = deque(maxlen=max_history)
        self.system_prompt = """You are a helpful research assistant. You can:
1. Search the web for information
2. Save research notes to files
3. List and read saved notes
4. Answer questions based on research

Be thorough, accurate, and cite sources when possible.
When saving notes, organize information clearly with proper formatting."""

    def add_user_message(self, content: str):
        """Add a user message"""
        self.messages.append({
            "role": "user",
            "content": content
        })

    def add_assistant_message(self, message):
        """Add an assistant message (can include tool calls)"""
        self.messages.append(message)

    def add_tool_response(self, tool_call_id: str, function_name: str, content: str):
        """Add a tool execution result"""
        self.messages.append({
            "tool_call_id": tool_call_id,
            "role": "tool",
            "name": function_name,
            "content": content
        })

    def get_messages(self) -> List[Dict]:
        """Get all messages with system prompt"""
        return [
            {"role": "system", "content": self.system_prompt},
            *list(self.messages)
        ]

    def clear(self):
        """Clear conversation history"""
        self.messages.clear()

    def get_summary(self) -> str:
        """Get a summary of the conversation"""
        user_messages = [m for m in self.messages if m["role"] == "user"]
        return f"Conversation: {len(self.messages)} messages, {len(user_messages)} user queries"
```

### Step 5: Main Agent Logic

```python
# agent.py
import json
from openai import OpenAI
from tools import ResearchTools
from memory import ConversationMemory
from config import Config

class ResearchAgent:
    """Main research assistant agent"""

    def __init__(self):
        Config.validate()
        self.client = OpenAI(api_key=Config.OPENAI_API_KEY)
        self.tools = ResearchTools()
        self.memory = ConversationMemory()
        self.tool_schemas = ResearchTools.get_tool_schemas()

    def process(self, user_input: str) -> str:
        """
        Process user input and return response

        Args:
            user_input: User's message

        Returns:
            Agent's response
        """
        # Add user message to memory
        self.memory.add_user_message(user_input)

        # Agent loop
        iteration = 0
        while iteration < Config.MAX_ITERATIONS:
            iteration += 1

            # Get LLM response
            response = self.client.chat.completions.create(
                model=Config.MODEL,
                messages=self.memory.get_messages(),
                tools=self.tool_schemas,
                tool_choice="auto",
                temperature=Config.TEMPERATURE,
                max_tokens=Config.MAX_TOKENS
            )

            response_message = response.choices[0].message

            # Add assistant message to memory
            self.memory.add_assistant_message(response_message)

            # Check if we're done
            if not response_message.tool_calls:
                # No more tool calls, return the response
                return response_message.content

            # Execute tool calls
            for tool_call in response_message.tool_calls:
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)

                print(f"\n🔧 Using tool: {function_name}")
                print(f"   Arguments: {function_args}")

                try:
                    # Execute the tool
                    result = ResearchTools.execute(function_name, **function_args)
                    result_str = json.dumps(result)

                except Exception as e:
                    result_str = json.dumps({"error": str(e)})

                # Add tool result to memory
                self.memory.add_tool_response(
                    tool_call.id,
                    function_name,
                    result_str
                )

        return "⚠️ Maximum iterations reached. Please try rephrasing your request."

    def reset(self):
        """Reset the agent's memory"""
        self.memory.clear()
        print("🔄 Conversation reset")
```

### Step 6: CLI Interface

```python
# cli.py
import sys
from agent import ResearchAgent
from colorama import init, Fore, Style

# Initialize colorama for cross-platform colored output
init(autoreset=True)

class CLI:
    """Command-line interface for the research assistant"""

    def __init__(self):
        self.agent = ResearchAgent()
        self.commands = {
            "/help": self.show_help,
            "/reset": self.reset,
            "/quit": self.quit,
            "/exit": self.quit
        }

    def show_banner(self):
        """Display welcome banner"""
        banner = f"""
{Fore.CYAN}╔══════════════════════════════════════╗
║  Research Assistant Agent v1.0       ║
║  Type /help for commands             ║
╚══════════════════════════════════════╝{Style.RESET_ALL}
        """
        print(banner)

    def show_help(self):
        """Display help message"""
        help_text = f"""
{Fore.YELLOW}Available Commands:{Style.RESET_ALL}
  /help   - Show this help message
  /reset  - Reset conversation history
  /quit   - Exit the application

{Fore.YELLOW}Capabilities:{Style.RESET_ALL}
  • Search the web for information
  • Save research notes to files
  • List and read saved notes
  • Answer questions with context

{Fore.YELLOW}Example queries:{Style.RESET_ALL}
  - "Search for information about quantum computing"
  - "Save that information as a note"
  - "What notes do I have?"
  - "Read my latest note"
        """
        print(help_text)

    def reset(self):
        """Reset the agent"""
        self.agent.reset()

    def quit(self):
        """Exit the application"""
        print(f"\n{Fore.CYAN}Goodbye! Happy researching! 👋{Style.RESET_ALL}")
        sys.exit(0)

    def run(self):
        """Main CLI loop"""
        self.show_banner()

        while True:
            try:
                # Get user input
                user_input = input(f"\n{Fore.GREEN}You: {Style.RESET_ALL}").strip()

                # Handle empty input
                if not user_input:
                    continue

                # Handle commands
                if user_input in self.commands:
                    self.commands[user_input]()
                    continue

                # Process with agent
                print(f"\n{Fore.BLUE}Agent: {Style.RESET_ALL}", end="", flush=True)
                response = self.agent.process(user_input)
                print(response)

            except KeyboardInterrupt:
                print(f"\n\n{Fore.YELLOW}Use /quit to exit{Style.RESET_ALL}")
            except Exception as e:
                print(f"\n{Fore.RED}Error: {e}{Style.RESET_ALL}")

if __name__ == "__main__":
    cli = CLI()
    cli.run()
```

### Step 7: Requirements

```text
# requirements.txt
openai>=1.0.0
python-dotenv>=1.0.0
colorama>=0.4.6
```

## Testing Your Agent

### Test Cases

1. **Basic Search**:
   ```
   You: Search for information about transformer architecture
   ```

2. **Save Notes**:
   ```
   You: Save that information as a note titled "Transformer Architecture Basics"
   ```

3. **List Notes**:
   ```
   You: What notes do I have?
   ```

4. **Multi-step Task**:
   ```
   You: Research quantum computing and save a summary
   ```

5. **Follow-up Questions**:
   ```
   You: What are the main applications?
   ```

## Debugging Tips

### Common Issues

1. **API Key Errors**:
   - Check `.env` file exists
   - Verify API key is valid
   - Ensure proper environment variable loading

2. **Tool Not Called**:
   - Check tool schema format
   - Verify function descriptions are clear
   - Lower temperature for more consistent behavior

3. **Infinite Loops**:
   - Check max_iterations setting
   - Verify tool responses are properly formatted
   - Ensure assistant messages are added to memory

4. **Memory Issues**:
   - Monitor message count
   - Clear history when needed
   - Implement token counting

## Enhancements

### Ideas to Extend Your Agent

1. **Better Search**: Integrate real search APIs (SerpAPI, Bing)
2. **Summarization**: Add dedicated summarization tool
3. **Citations**: Track and format sources properly
4. **Export**: Support PDF, Word formats
5. **Voice**: Add speech-to-text input
6. **Web UI**: Create a web interface
7. **Persistence**: Save conversations to database
8. **Analytics**: Track usage and costs

## Key Takeaways

1. Building an agent requires orchestrating multiple components
2. State management is crucial for multi-turn conversations
3. Good error handling makes agents robust
4. Tool design significantly impacts agent capabilities
5. User interface matters for usability
6. Testing with diverse scenarios reveals edge cases

## Resources

- [OpenAI Cookbook - Agents](https://cookbook.openai.com/)
- [LangChain Agent Templates](https://github.com/langchain-ai/langchain/tree/master/templates)
- [Building Production Agents](https://www.anthropic.com/index/building-effective-agents)

## Daily Challenge

**Enhance Your Agent**

Add one of these features:
1. **Summarize Command**: Summarize conversation history
2. **Export Tool**: Export notes to different formats
3. **Search History**: Track and replay past searches
4. **Cost Tracking**: Monitor API usage and costs
5. **Fact Checking**: Cross-reference information

## Reflection Questions

1. What was the most challenging part of building your agent?
2. How would you improve the agent's reliability?
3. What additional tools would make this agent more useful?
4. How would you deploy this agent for others to use?

## Tomorrow's Preview

Day 6: Agent Architectures & Patterns - Learn different architectural patterns for building more sophisticated agents.

---

**Progress**: 5/30 days completed

[← Previous: Day 4](./day-04.md) | [Back to Overview](../README.md) | [Next: Day 6 →](./day-06.md)
