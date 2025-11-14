# Day 23: Monitoring & Observability

## Overview
Monitor and debug production agents with comprehensive observability systems.

## Learning Objectives
- Implement logging for agents
- Set up metrics collection
- Create tracing for agent workflows
- Build monitoring dashboards
- Set up alerting
- Debug production issues

## Observability Stack

### Logging
```python
import logging
import json

class AgentLogger:
    def __init__(self, name):
        self.logger = logging.getLogger(name)

    def log_interaction(self, user_input, agent_output, metadata):
        self.logger.info(json.dumps({
            "event": "agent_interaction",
            "input": user_input,
            "output": agent_output,
            "tokens": metadata.get("tokens"),
            "cost": metadata.get("cost"),
            "duration_ms": metadata.get("duration")
        }))
```

### Metrics
```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
agent_requests = Counter('agent_requests_total', 'Total requests')
agent_latency = Histogram('agent_latency_seconds', 'Request latency')
agent_cost = Counter('agent_cost_dollars', 'Total cost')
agent_tokens = Counter('agent_tokens_total', 'Total tokens')

# Use in agent
@agent_latency.time()
def process_query(query):
    agent_requests.inc()
    result = agent.process(query)
    agent_cost.inc(result["cost"])
    agent_tokens.inc(result["tokens"])
    return result
```

### Tracing
```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

tracer = trace.get_tracer(__name__)

def process_with_tracing(query):
    with tracer.start_as_current_span("agent_process"):
        with tracer.start_as_current_span("tool_call"):
            result = call_tool()
        return result
```

## Monitoring Dashboard

Key metrics to track:
- Request rate (requests/minute)
- Success rate (%)
- Average latency (ms)
- Token usage (tokens/request)
- Cost ($/request)
- Error rate (%)

## Alerting Rules

```yaml
# alerts.yml
alerts:
  - name: high_error_rate
    condition: error_rate > 5%
    action: notify_team

  - name: high_cost
    condition: hourly_cost > $10
    action: notify_admin

  - name: slow_response
    condition: p95_latency > 5s
    action: investigate
```

## Resources
- [OpenTelemetry Python](https://opentelemetry.io/docs/instrumentation/python/)
- [Prometheus Guide](https://prometheus.io/docs/introduction/overview/)
- [LangSmith](https://www.langchain.com/langsmith)

## Daily Challenge
Add comprehensive monitoring to your agent with logs, metrics, and tracing.

---

**Progress**: 23/30 days completed

[← Previous: Day 22](./day-22.md) | [Back to Overview](../README.md) | [Next: Day 24 →](./day-24.md)
