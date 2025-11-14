# Day 9: Memory Systems for Agents

## Overview
Memory is what separates intelligent agents from simple chatbots. Today you'll learn to implement different memory systems that allow agents to learn, remember context, and improve over time.

## Learning Objectives
- Understand different types of agent memory
- Implement short-term (conversation) memory
- Build long-term (persistent) memory systems
- Create semantic memory with embeddings
- Implement memory retrieval strategies
- Manage memory costs and performance

## Theoretical Concepts

### Types of Memory

#### 1. Short-Term Memory
- **Purpose**: Maintain conversation context
- **Duration**: Single session
- **Implementation**: Message history buffer
- **Size**: Limited by context window

#### 2. Long-Term Memory
- **Purpose**: Persist information across sessions
- **Duration**: Permanent
- **Implementation**: Database, files
- **Size**: Unlimited (disk-based)

#### 3. Semantic Memory
- **Purpose**: Store and retrieve relevant knowledge
- **Duration**: Permanent
- **Implementation**: Vector embeddings + similarity search
- **Size**: Large, indexed

#### 4. Episodic Memory
- **Purpose**: Remember specific past interactions
- **Duration**: Permanent
- **Implementation**: Structured event logs
- **Size**: Grows over time

### Memory Architecture

```
User Input
    ↓
Short-Term Memory (recent context)
    ↓
Semantic Memory (relevant knowledge) ← Vector Search
    ↓
Long-Term Memory (facts, preferences)
    ↓
Agent Processing
    ↓
Update All Memory Types
```

## Hands-On Exercises

### Exercise 1: Conversation Memory

```python
# conversation_memory.py
from collections import deque
from typing import List, Dict, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    """Represents a single message"""
    role: str  # user, assistant, system, tool
    content: str
    timestamp: datetime
    metadata: Optional[Dict] = None

class ConversationMemory:
    """Manages conversation history with smart truncation"""

    def __init__(self, max_messages: int = 20, max_tokens: int = 4000):
        self.max_messages = max_messages
        self.max_tokens = max_tokens
        self.messages: deque = deque(maxlen=max_messages)
        self.system_prompt: Optional[str] = None

    def set_system_prompt(self, prompt: str):
        """Set the system prompt"""
        self.system_prompt = prompt

    def add_message(self, role: str, content: str, metadata: Dict = None):
        """Add a message to memory"""
        message = Message(
            role=role,
            content=content,
            timestamp=datetime.now(),
            metadata=metadata or {}
        )
        self.messages.append(message)

    def get_messages(self, include_system: bool = True) -> List[Dict]:
        """Get messages in OpenAI format"""
        result = []

        if include_system and self.system_prompt:
            result.append({
                "role": "system",
                "content": self.system_prompt
            })

        for msg in self.messages:
            result.append({
                "role": msg.role,
                "content": msg.content
            })

        return result

    def get_recent_messages(self, n: int = 5) -> List[Message]:
        """Get n most recent messages"""
        return list(self.messages)[-n:]

    def search_messages(self, keyword: str) -> List[Message]:
        """Search messages by keyword"""
        return [
            msg for msg in self.messages
            if keyword.lower() in msg.content.lower()
        ]

    def summarize_conversation(self) -> str:
        """Get a summary of the conversation"""
        user_messages = [m for m in self.messages if m.role == "user"]
        assistant_messages = [m for m in self.messages if m.role == "assistant"]

        return f"""Conversation Summary:
- Total messages: {len(self.messages)}
- User messages: {len(user_messages)}
- Assistant responses: {len(assistant_messages)}
- Duration: {self.messages[0].timestamp if self.messages else 'N/A'} to {self.messages[-1].timestamp if self.messages else 'N/A'}
"""

    def clear(self):
        """Clear all messages"""
        self.messages.clear()

    def export_to_json(self) -> List[Dict]:
        """Export conversation to JSON"""
        return [
            {
                "role": msg.role,
                "content": msg.content,
                "timestamp": msg.timestamp.isoformat(),
                "metadata": msg.metadata
            }
            for msg in self.messages
        ]

# Usage
memory = ConversationMemory(max_messages=10)
memory.set_system_prompt("You are a helpful assistant")
memory.add_message("user", "Hello!")
memory.add_message("assistant", "Hi! How can I help?")
memory.add_message("user", "Tell me about Python")
memory.add_message("assistant", "Python is a programming language...")

print(memory.summarize_conversation())
print("\nRecent messages:")
for msg in memory.get_recent_messages(3):
    print(f"{msg.role}: {msg.content}")
```

### Exercise 2: Long-Term Memory with SQLite

```python
# long_term_memory.py
import sqlite3
import json
from datetime import datetime
from typing import List, Dict, Optional

class LongTermMemory:
    """Persistent memory using SQLite"""

    def __init__(self, db_path: str = "agent_memory.db"):
        self.db_path = db_path
        self.conn = sqlite3.connect(db_path)
        self._create_tables()

    def _create_tables(self):
        """Create database tables"""
        cursor = self.conn.cursor()

        # Facts table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS facts (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                category TEXT NOT NULL,
                key TEXT NOT NULL,
                value TEXT NOT NULL,
                confidence REAL DEFAULT 1.0,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                UNIQUE(category, key)
            )
        """)

        # Experiences table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS experiences (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                event_type TEXT NOT NULL,
                description TEXT NOT NULL,
                outcome TEXT,
                metadata TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)

        # User preferences table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS preferences (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id TEXT NOT NULL,
                preference_key TEXT NOT NULL,
                preference_value TEXT NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                UNIQUE(user_id, preference_key)
            )
        """)

        self.conn.commit()

    def store_fact(self, category: str, key: str, value: str,
                   confidence: float = 1.0):
        """Store a fact"""
        cursor = self.conn.cursor()
        cursor.execute("""
            INSERT OR REPLACE INTO facts (category, key, value, confidence, updated_at)
            VALUES (?, ?, ?, ?, ?)
        """, (category, key, value, confidence, datetime.now()))
        self.conn.commit()

    def get_fact(self, category: str, key: str) -> Optional[Dict]:
        """Retrieve a fact"""
        cursor = self.conn.cursor()
        cursor.execute("""
            SELECT value, confidence, created_at, updated_at
            FROM facts
            WHERE category = ? AND key = ?
        """, (category, key))

        row = cursor.fetchone()
        if row:
            return {
                "value": row[0],
                "confidence": row[1],
                "created_at": row[2],
                "updated_at": row[3]
            }
        return None

    def get_facts_by_category(self, category: str) -> List[Dict]:
        """Get all facts in a category"""
        cursor = self.conn.cursor()
        cursor.execute("""
            SELECT key, value, confidence
            FROM facts
            WHERE category = ?
            ORDER BY updated_at DESC
        """, (category,))

        return [
            {"key": row[0], "value": row[1], "confidence": row[2]}
            for row in cursor.fetchall()
        ]

    def store_experience(self, event_type: str, description: str,
                        outcome: str = None, metadata: Dict = None):
        """Store an experience/event"""
        cursor = self.conn.cursor()
        cursor.execute("""
            INSERT INTO experiences (event_type, description, outcome, metadata)
            VALUES (?, ?, ?, ?)
        """, (event_type, description, outcome, json.dumps(metadata or {})))
        self.conn.commit()

    def get_experiences(self, event_type: str = None, limit: int = 10) -> List[Dict]:
        """Retrieve past experiences"""
        cursor = self.conn.cursor()

        if event_type:
            cursor.execute("""
                SELECT event_type, description, outcome, metadata, created_at
                FROM experiences
                WHERE event_type = ?
                ORDER BY created_at DESC
                LIMIT ?
            """, (event_type, limit))
        else:
            cursor.execute("""
                SELECT event_type, description, outcome, metadata, created_at
                FROM experiences
                ORDER BY created_at DESC
                LIMIT ?
            """, (limit,))

        return [
            {
                "event_type": row[0],
                "description": row[1],
                "outcome": row[2],
                "metadata": json.loads(row[3]),
                "created_at": row[4]
            }
            for row in cursor.fetchall()
        ]

    def store_preference(self, user_id: str, key: str, value: str):
        """Store user preference"""
        cursor = self.conn.cursor()
        cursor.execute("""
            INSERT OR REPLACE INTO preferences (user_id, preference_key, preference_value)
            VALUES (?, ?, ?)
        """, (user_id, key, value))
        self.conn.commit()

    def get_preference(self, user_id: str, key: str) -> Optional[str]:
        """Get user preference"""
        cursor = self.conn.cursor()
        cursor.execute("""
            SELECT preference_value
            FROM preferences
            WHERE user_id = ? AND preference_key = ?
        """, (user_id, key))

        row = cursor.fetchone()
        return row[0] if row else None

    def get_all_preferences(self, user_id: str) -> Dict:
        """Get all preferences for a user"""
        cursor = self.conn.cursor()
        cursor.execute("""
            SELECT preference_key, preference_value
            FROM preferences
            WHERE user_id = ?
        """, (user_id,))

        return {row[0]: row[1] for row in cursor.fetchall()}

    def close(self):
        """Close database connection"""
        self.conn.close()

# Usage
ltm = LongTermMemory("my_agent_memory.db")

# Store facts
ltm.store_fact("user_info", "name", "Alice")
ltm.store_fact("user_info", "language", "Python")
ltm.store_fact("tech_stack", "framework", "FastAPI")

# Retrieve facts
name = ltm.get_fact("user_info", "name")
print(f"User name: {name}")

# Store experiences
ltm.store_experience(
    "task_completion",
    "Created a REST API",
    "success",
    {"duration_minutes": 45, "lines_of_code": 250}
)

# Get experiences
experiences = ltm.get_experiences("task_completion")
print(f"\nPast experiences: {experiences}")

# Store preferences
ltm.store_preference("user_001", "code_style", "pep8")
ltm.store_preference("user_001", "theme", "dark")

prefs = ltm.get_all_preferences("user_001")
print(f"\nUser preferences: {prefs}")

ltm.close()
```

### Exercise 3: Semantic Memory with Embeddings

```python
# semantic_memory.py
from openai import OpenAI
import numpy as np
from typing import List, Dict, Tuple
import json

client = OpenAI()

class SemanticMemory:
    """Memory system using embeddings for semantic search"""

    def __init__(self):
        self.memories: List[Dict] = []
        self.embeddings: List[np.ndarray] = []

    def _get_embedding(self, text: str) -> np.ndarray:
        """Get embedding for text"""
        response = client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return np.array(response.data[0].embedding)

    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        """Calculate cosine similarity"""
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

    def add_memory(self, content: str, metadata: Dict = None):
        """Add a memory"""
        embedding = self._get_embedding(content)

        memory = {
            "content": content,
            "metadata": metadata or {},
            "id": len(self.memories)
        }

        self.memories.append(memory)
        self.embeddings.append(embedding)

    def search(self, query: str, top_k: int = 5,
              threshold: float = 0.7) -> List[Tuple[Dict, float]]:
        """
        Search for relevant memories

        Args:
            query: Search query
            top_k: Number of results to return
            threshold: Minimum similarity score

        Returns:
            List of (memory, similarity_score) tuples
        """
        if not self.memories:
            return []

        query_embedding = self._get_embedding(query)

        # Calculate similarities
        similarities = [
            self._cosine_similarity(query_embedding, emb)
            for emb in self.embeddings
        ]

        # Get top-k results above threshold
        results = [
            (self.memories[i], similarities[i])
            for i in range(len(self.memories))
            if similarities[i] >= threshold
        ]

        # Sort by similarity (descending)
        results.sort(key=lambda x: x[1], reverse=True)

        return results[:top_k]

    def get_context(self, query: str, max_memories: int = 3) -> str:
        """Get relevant context for a query"""
        results = self.search(query, top_k=max_memories)

        if not results:
            return ""

        context = "Relevant information from memory:\n\n"
        for memory, score in results:
            context += f"- {memory['content']} (relevance: {score:.2f})\n"

        return context

    def save(self, filepath: str):
        """Save memories to file"""
        data = {
            "memories": self.memories,
            "embeddings": [emb.tolist() for emb in self.embeddings]
        }

        with open(filepath, 'w') as f:
            json.dump(data, f)

    def load(self, filepath: str):
        """Load memories from file"""
        with open(filepath, 'r') as f:
            data = json.load(f)

        self.memories = data["memories"]
        self.embeddings = [np.array(emb) for emb in data["embeddings"]]

# Usage
memory = SemanticMemory()

# Add knowledge
memory.add_memory("Python was created by Guido van Rossum in 1991")
memory.add_memory("FastAPI is a modern web framework for Python")
memory.add_memory("React is a JavaScript library for building UIs")
memory.add_memory("Docker helps containerize applications")
memory.add_memory("Kubernetes orchestrates Docker containers")

# Search
print("Search: 'Python web development'")
results = memory.search("Python web development", top_k=3)
for mem, score in results:
    print(f"- {mem['content']} (score: {score:.3f})")

print("\nContext for: 'How do I build web apps?'")
context = memory.get_context("How do I build web apps?")
print(context)
```

### Exercise 4: Integrated Memory System

```python
# integrated_memory.py
class IntegratedMemorySystem:
    """Combines all memory types"""

    def __init__(self):
        self.conversation = ConversationMemory()
        self.long_term = LongTermMemory()
        self.semantic = SemanticMemory()

    def process_interaction(self, user_input: str, agent_response: str):
        """Process an interaction across all memory types"""

        # Short-term memory
        self.conversation.add_message("user", user_input)
        self.conversation.add_message("assistant", agent_response)

        # Semantic memory (add important information)
        combined = f"Q: {user_input}\nA: {agent_response}"
        self.semantic.add_memory(combined)

        # Long-term memory (extract facts/experiences)
        # This would use NLP to extract entities and facts
        # Simplified here
        self.long_term.store_experience(
            "conversation",
            user_input,
            agent_response
        )

    def get_context_for_query(self, query: str) -> Dict:
        """Get all relevant context for a query"""
        return {
            "recent_conversation": self.conversation.get_recent_messages(5),
            "semantic_context": self.semantic.get_context(query),
            "relevant_experiences": self.long_term.get_experiences(limit=3)
        }

# Usage example with an agent
# This would be integrated into your agent's processing
```

## Key Takeaways

1. Memory is essential for context-aware agents
2. Different memory types serve different purposes
3. Semantic search enables intelligent retrieval
4. Balance memory size with cost and performance
5. Persistent storage enables learning over time

## Resources

- [LangChain Memory](https://python.langchain.com/docs/modules/memory/)
- [Vector Databases Comparison](https://www.pinecone.io/learn/vector-database/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)

## Daily Challenge

Build a personal knowledge assistant that:
1. Stores facts you tell it
2. Remembers conversation context
3. Can recall relevant information when asked
4. Learns your preferences over time

## Tomorrow's Preview

Day 10: Planning and Task Decomposition - Learn how agents break down complex goals into actionable steps.

---

**Progress**: 9/30 days completed

[← Previous: Day 8](./day-08.md) | [Back to Overview](../README.md) | [Next: Day 10 →](./day-10.md)
