# Day 22: Production Deployment Strategies

## Overview
Learn to deploy AI agents to production with proper infrastructure, scaling, and reliability.

## Learning Objectives
- Deploy agents to cloud platforms
- Implement API endpoints for agents
- Set up containerization with Docker
- Configure auto-scaling
- Implement load balancing
- Manage secrets and configuration

## Deployment Architectures

### 1. API Service
```
User → API Gateway → Agent Service → LLM API
                         ↓
                    Vector DB / Cache
```

### 2. Serverless
```
User → AWS Lambda / Cloud Function → Agent
```

### 3. Microservices
```
Multiple Agent Services ← Load Balancer ← Users
```

## FastAPI Agent Service

```python
# app.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from agent import MyAgent

app = FastAPI()
agent = MyAgent()

class Query(BaseModel):
    text: str
    user_id: str

class Response(BaseModel):
    response: str
    cost: float
    tokens: int

@app.post("/query", response_model=Response)
async def query_agent(query: Query):
    try:
        result = agent.process(query.text)
        return Response(
            response=result["output"],
            cost=result["cost"],
            tokens=result["tokens"]
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health():
    return {"status": "healthy"}
```

## Docker Deployment

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  agent:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./data:/app/data
```

## Best Practices

1. **Environment Variables**: Never hardcode secrets
2. **Health Checks**: Implement liveness/readiness probes
3. **Graceful Shutdown**: Handle SIGTERM properly
4. **Resource Limits**: Set memory/CPU limits
5. **Logging**: Structured JSON logging
6. **Metrics**: Expose Prometheus metrics

## Resources
- [FastAPI Deployment](https://fastapi.tiangolo.com/deployment/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [AWS Lambda for AI](https://aws.amazon.com/lambda/)

## Daily Challenge
Deploy your research assistant as a REST API with Docker and implement health checks.

---

**Progress**: 22/30 days completed

[← Previous: Day 21](./day-21.md) | [Back to Overview](../README.md) | [Next: Day 23 →](./day-23.md)
