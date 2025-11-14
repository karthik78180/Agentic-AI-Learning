# Day 13: Error Handling & Reliability

## Overview
Production agents must handle errors gracefully. Learn strategies to build robust, reliable agents that fail safely and recover automatically.

## Learning Objectives
- Implement comprehensive error handling
- Build retry mechanisms with exponential backoff
- Create circuit breakers for failing services
- Add fallback strategies
- Implement graceful degradation
- Monitor and log errors effectively

## Error Handling Strategies

### 1. Retry with Exponential Backoff
```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    delay = base_delay * (2 ** attempt)
                    print(f"Retry {attempt + 1}/{max_retries} after {delay}s")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
def call_api():
    # API call that might fail
    pass
```

### 2. Circuit Breaker Pattern
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failures = 0
        self.last_failure_time = None
        self.state = "closed"  # closed, open, half-open

    def call(self, func, *args, **kwargs):
        if self.state == "open":
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "half-open"
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        self.failures = 0
        self.state = "closed"

    def on_failure(self):
        self.failures += 1
        self.last_failure_time = time.time()
        if self.failures >= self.failure_threshold:
            self.state = "open"
```

### 3. Fallback Strategies
```python
class RobustAgent:
    def process(self, query):
        try:
            return self.primary_method(query)
        except PrimaryServiceError:
            logger.warning("Primary service failed, using fallback")
            return self.fallback_method(query)
        except Exception as e:
            logger.error(f"All methods failed: {e}")
            return self.safe_default_response()
```

## Key Takeaways
1. Always validate inputs and outputs
2. Use retries for transient failures
3. Implement circuit breakers for persistent failures
4. Provide fallbacks for critical functionality
5. Log errors for debugging and monitoring
6. Never expose sensitive error details to users

## Resources
- [Python Error Handling Best Practices](https://docs.python.org/3/tutorial/errors.html)
- [Circuit Breaker Pattern](https://martinfowler.com/bliki/CircuitBreaker.html)

## Daily Challenge
Add comprehensive error handling to your Day 5 research assistant agent.

---

**Progress**: 13/30 days completed

[← Previous: Day 12](./day-12.md) | [Back to Overview](../README.md) | [Next: Day 14 →](./day-14.md)
