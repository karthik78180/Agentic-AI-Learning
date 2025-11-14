# Day 16: Agent Communication Patterns

## Overview
Effective communication is crucial for multi-agent systems. Master the patterns and protocols that enable agents to collaborate efficiently.

## Learning Objectives
- Implement message-based communication
- Design communication protocols
- Build event-driven agent systems
- Handle asynchronous messaging
- Create shared state mechanisms
- Optimize communication efficiency

## Communication Patterns

### 1. Request-Response
Simple synchronous communication.

### 2. Publish-Subscribe
Event-driven, decoupled communication.

### 3. Message Queue
Asynchronous, buffered communication.

### 4. Shared Memory
Agents read/write to common data store.

## Implementation Example

```python
# agent_communication.py
from queue import Queue
from typing import Dict, Any
import threading

class MessageBus:
    """Central message bus for agent communication"""

    def __init__(self):
        self.queues: Dict[str, Queue] = {}
        self.subscribers: Dict[str, list] = {}

    def register(self, agent_id: str):
        self.queues[agent_id] = Queue()

    def send(self, to: str, message: Dict[Any, Any]):
        if to in self.queues:
            self.queues[to].put(message)

    def receive(self, agent_id: str, timeout=1):
        try:
            return self.queues[agent_id].get(timeout=timeout)
        except:
            return None

    def subscribe(self, agent_id: str, event_type: str):
        if event_type not in self.subscribers:
            self.subscribers[event_type] = []
        self.subscribers[event_type].append(agent_id)

    def publish(self, event_type: str, data: Dict[Any, Any]):
        if event_type in self.subscribers:
            for agent_id in self.subscribers[event_type]:
                self.send(agent_id, {
                    "type": "event",
                    "event_type": event_type,
                    "data": data
                })
```

## Resources
- [Message Patterns](https://www.enterpriseintegrationpatterns.com/)
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)

## Daily Challenge
Build a task distribution system where a coordinator agent distributes work to worker agents via message queue.

---

**Progress**: 16/30 days completed

[← Previous: Day 15](./day-15.md) | [Back to Overview](../README.md) | [Next: Day 17 →](./day-17.md)
