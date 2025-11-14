# Day 24: Cost Optimization & Efficiency

## Overview
Learn strategies to reduce costs and improve efficiency of AI agents in production.

## Learning Objectives
- Optimize token usage
- Implement caching strategies
- Choose cost-effective models
- Batch operations efficiently
- Reduce unnecessary API calls
- Monitor and control costs

## Cost Optimization Strategies

### 1. Token Optimization

```python
class TokenOptimizer:
    def compress_history(self, messages, max_tokens=2000):
        """Compress conversation history"""
        # Keep system prompt + recent messages
        system = messages[0]
        recent = messages[-5:]

        # Summarize middle messages if needed
        if len(messages) > 7:
            middle = messages[1:-5]
            summary = self.summarize(middle)
            return [system, summary] + recent

        return messages

    def summarize(self, messages):
        """Create summary of messages"""
        # Use cheaper model for summarization
        # ...
        pass
```

### 2. Caching

```python
from functools import lru_cache
import hashlib

class ResponseCache:
    def __init__(self):
        self.cache = {}

    def get_cache_key(self, query):
        return hashlib.md5(query.encode()).hexdigest()

    def get(self, query):
        key = self.get_cache_key(query)
        return self.cache.get(key)

    def set(self, query, response):
        key = self.get_cache_key(query)
        self.cache[key] = response

# Usage
cache = ResponseCache()

def process_with_cache(query):
    cached = cache.get(query)
    if cached:
        return cached

    response = expensive_llm_call(query)
    cache.set(query, response)
    return response
```

### 3. Model Selection

```python
class SmartRouter:
    """Route to appropriate model based on complexity"""

    def analyze_complexity(self, query):
        # Simple heuristics
        if len(query.split()) < 10:
            return "simple"
        elif any(word in query for word in ["analyze", "complex", "detailed"]):
            return "complex"
        return "medium"

    def route(self, query):
        complexity = self.analyze_complexity(query)

        models = {
            "simple": "gpt-3.5-turbo",  # Cheaper
            "medium": "gpt-4-turbo",
            "complex": "gpt-4"
        }

        return models[complexity]
```

### 4. Batching

```python
async def batch_process(queries, batch_size=10):
    """Process queries in batches"""
    results = []

    for i in range(0, len(queries), batch_size):
        batch = queries[i:i+batch_size]
        batch_results = await asyncio.gather(*[
            process_query(q) for q in batch
        ])
        results.extend(batch_results)

    return results
```

## Cost Monitoring

```python
class CostTracker:
    def __init__(self):
        self.total_cost = 0
        self.cost_by_model = {}

    def track(self, model, tokens_in, tokens_out):
        cost = self.calculate_cost(model, tokens_in, tokens_out)
        self.total_cost += cost
        self.cost_by_model[model] = self.cost_by_model.get(model, 0) + cost

        # Alert if threshold exceeded
        if self.total_cost > 100:  # $100 threshold
            self.send_alert()

        return cost
```

## Best Practices

1. **Use Cheaper Models**: GPT-3.5 for simple tasks
2. **Cache Aggressively**: Cache repeated queries
3. **Compress Context**: Summarize old messages
4. **Batch Requests**: Reduce API call overhead
5. **Set Budgets**: Hard limits on spending
6. **Monitor Continuously**: Track costs in real-time

## Cost Comparison

| Model | Input ($/1K) | Output ($/1K) | Use Case |
|-------|-------------|---------------|----------|
| GPT-3.5 | $0.0005 | $0.0015 | Simple tasks |
| GPT-4 Turbo | $0.01 | $0.03 | Complex reasoning |
| GPT-4 | $0.03 | $0.06 | Critical tasks |
| Claude Haiku | $0.00025 | $0.00125 | High volume |
| Claude Sonnet | $0.003 | $0.015 | Balanced |

## Resources
- [OpenAI Pricing](https://openai.com/pricing)
- [Token Optimization Guide](https://platform.openai.com/docs/guides/optimizing)

## Daily Challenge
Optimize your agent to reduce costs by 50% while maintaining quality.

---

**Progress**: 24/30 days completed

[← Previous: Day 23](./day-23.md) | [Back to Overview](../README.md) | [Next: Day 25 →](./day-25.md)
