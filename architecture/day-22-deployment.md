# Day 22: Production Deployment Architecture

## Deployment Architecture Overview

```mermaid
graph TB
    subgraph "User Layer"
        WEB[Web Clients]
        MOBILE[Mobile Apps]
        API_CLIENTS[API Clients]
    end

    subgraph "API Gateway Layer"
        LB[Load Balancer]
        GW[API Gateway<br/>Auth, Rate Limiting]
    end

    subgraph "Application Layer"
        APP1[Agent Service 1]
        APP2[Agent Service 2]
        APP3[Agent Service 3]
    end

    subgraph "Infrastructure Layer"
        CACHE[(Redis Cache)]
        DB[(PostgreSQL)]
        VECTOR[(Vector DB)]
        QUEUE[(Message Queue)]
    end

    subgraph "External Services"
        LLM[LLM APIs<br/>OpenAI/Anthropic]
        TOOLS[External Tools<br/>APIs]
    end

    WEB --> LB
    MOBILE --> LB
    API_CLIENTS --> LB

    LB --> GW
    GW --> APP1
    GW --> APP2
    GW --> APP3

    APP1 --> CACHE
    APP1 --> DB
    APP1 --> VECTOR
    APP1 --> QUEUE

    APP2 --> CACHE
    APP2 --> DB
    APP2 --> VECTOR
    APP2 --> QUEUE

    APP3 --> CACHE
    APP3 --> DB
    APP3 --> VECTOR
    APP3 --> QUEUE

    APP1 --> LLM
    APP2 --> LLM
    APP3 --> LLM

    APP1 --> TOOLS
    APP2 --> TOOLS
    APP3 --> TOOLS

    style GW fill:#4a90e2
    style APP1 fill:#4ecdc4
    style APP2 fill:#4ecdc4
    style APP3 fill:#4ecdc4
```

## FastAPI Service Architecture

```mermaid
graph LR
    subgraph "FastAPI Application"
        direction TB

        ROUTER[API Routes<br/>/query, /health]
        MIDDLEWARE[Middleware<br/>Auth, CORS, Logging]
        AGENT[Agent Core<br/>Business Logic]
        MODELS[Pydantic Models<br/>Request/Response]

        ROUTER --> MIDDLEWARE
        MIDDLEWARE --> AGENT
        ROUTER --> MODELS
    end

    ROUTER --> DEPENDS[Dependencies<br/>DB, Cache, Config]

    CLIENT[HTTP Client] --> ROUTER
    AGENT --> EXTERNAL[External Services]

    style ROUTER fill:#4ecdc4
    style AGENT fill:#f9ca24
```

**FastAPI Endpoint Structure:**

```mermaid
sequenceDiagram
    participant Client
    participant APIGateway
    participant Middleware
    participant Route
    participant Agent
    participant Database

    Client->>APIGateway: POST /query
    APIGateway->>Middleware: Validate JWT
    Middleware->>Middleware: Rate limit check
    Middleware->>Route: Forward request
    Route->>Agent: Process query
    Agent->>Database: Fetch context
    Database->>Agent: Context data
    Agent->>Agent: Generate response
    Agent->>Route: Response
    Route->>Client: JSON response
```

## Docker Deployment

```mermaid
graph TB
    subgraph "Docker Architecture"
        direction LR

        subgraph "Container 1"
            APP1[Agent App<br/>Python + FastAPI]
        end

        subgraph "Container 2"
            APP2[Agent App<br/>(Replica)]
        end

        subgraph "Container 3"
            DB[PostgreSQL<br/>Database]
        end

        subgraph "Container 4"
            REDIS[Redis<br/>Cache]
        end

        subgraph "Container 5"
            VECTOR[Qdrant<br/>Vector DB]
        end
    end

    NETWORK[Docker Network] -.-> APP1
    NETWORK -.-> APP2
    NETWORK -.-> DB
    NETWORK -.-> REDIS
    NETWORK -.-> VECTOR

    VOLUMES[(Docker Volumes)] -.-> DB
    VOLUMES -.-> VECTOR

    style APP1 fill:#4ecdc4
    style APP2 fill:#4ecdc4
    style NETWORK fill:#f9ca24
```

**Docker Compose Structure:**

```mermaid
classDiagram
    class DockerCompose {
        version: "3.8"
        services
        networks
        volumes
    }

    class Service {
        build: string
        image: string
        ports: array
        environment: object
        depends_on: array
        volumes: array
    }

    DockerCompose --> Service : defines
```

## Kubernetes Deployment

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        INGRESS[Ingress Controller<br/>nginx/traefik]

        subgraph "Namespace: production"
            SVC[Service<br/>ClusterIP]

            subgraph "Deployment"
                POD1[Pod 1<br/>Agent Container]
                POD2[Pod 2<br/>Agent Container]
                POD3[Pod 3<br/>Agent Container]
            end

            CM[ConfigMap<br/>Environment Vars]
            SECRET[Secret<br/>API Keys]

            PVC[Persistent Volume]
        end

        INGRESS --> SVC
        SVC --> POD1
        SVC --> POD2
        SVC --> POD3

        POD1 --> CM
        POD1 --> SECRET
        POD2 --> CM
        POD2 --> SECRET
        POD3 --> CM
        POD3 --> SECRET

        POD1 --> PVC
    end

    style INGRESS fill:#4a90e2
    style SVC fill:#4ecdc4
    style POD1 fill:#45b7d1
    style POD2 fill:#45b7d1
    style POD3 fill:#45b7d1
```

## Auto-Scaling Architecture

```mermaid
graph LR
    subgraph "Horizontal Pod Autoscaler"
        METRICS[Metrics Server<br/>CPU, Memory, Custom]

        METRICS --> HPA[HPA Controller]

        HPA --> DECIDE{Scaling<br/>Needed?}

        DECIDE -->|Scale Up| INCREASE[Add Pods]
        DECIDE -->|Scale Down| DECREASE[Remove Pods]
        DECIDE -->|Maintain| KEEP[Keep Current]

        INCREASE --> DEPLOY[Deployment]
        DECREASE --> DEPLOY
        KEEP --> DEPLOY
    end

    DEPLOY --> PODS[Pod Replicas:<br/>2-10 pods]

    style HPA fill:#4a90e2
    style PODS fill:#7ed321
```

## Cloud Deployment Options

```mermaid
graph TB
    subgraph "AWS Deployment"
        ECS[ECS/Fargate<br/>Container Service]
        RDS[RDS<br/>Managed DB]
        ELASTICACHE[ElastiCache<br/>Redis]
        S3[S3<br/>Storage]
        ALB[Application LB]
    end

    subgraph "GCP Deployment"
        GKE[GKE<br/>Kubernetes]
        CLOUDSQL[Cloud SQL<br/>Managed DB]
        MEMSTORE[Memorystore<br/>Redis]
        GCS[Cloud Storage<br/>Files]
        GCLB[Cloud LB]
    end

    subgraph "Azure Deployment"
        AKS[AKS<br/>Kubernetes]
        AZURESQL[Azure SQL<br/>Managed DB]
        AZUREREDIS[Azure Cache<br/>Redis]
        BLOB[Blob Storage<br/>Files]
        APPGW[App Gateway<br/>LB]
    end

    style ECS fill:#ff9900
    style GKE fill:#4285f4
    style AKS fill:#0089d6
```

## Serverless Architecture

```mermaid
graph TB
    CLIENT[API Client] --> APIGW[API Gateway]

    APIGW --> LAMBDA1[Lambda Function 1<br/>Query Handler]
    APIGW --> LAMBDA2[Lambda Function 2<br/>Admin Handler]

    LAMBDA1 --> DYNAMODB[(DynamoDB)]
    LAMBDA1 --> S3[(S3 Bucket)]
    LAMBDA1 --> LLM[OpenAI API]

    LAMBDA2 --> DYNAMODB
    LAMBDA2 --> S3

    LAMBDA1 -.->|Async| SQS[SQS Queue]
    SQS --> LAMBDA3[Lambda Function 3<br/>Background Worker]

    style APIGW fill:#4a90e2
    style LAMBDA1 fill:#ff9900
    style LAMBDA2 fill:#ff9900
    style LAMBDA3 fill:#ff9900
```

## CI/CD Pipeline

```mermaid
graph LR
    CODE[Code Push<br/>GitHub] --> CI{CI Pipeline}

    CI --> TEST[Run Tests<br/>pytest, integration]
    TEST --> BUILD[Build Docker<br/>Image]
    BUILD --> SCAN[Security Scan<br/>Vulnerability check]
    SCAN --> PUSH[Push to Registry<br/>Docker Hub/ECR]

    PUSH --> CD{CD Pipeline}

    CD --> STAGING[Deploy to<br/>Staging]
    STAGING --> E2E[E2E Tests]
    E2E --> APPROVE{Manual<br/>Approval?}

    APPROVE -->|Yes| PROD[Deploy to<br/>Production]
    APPROVE -->|No| ROLLBACK[Rollback]

    PROD --> MONITOR[Monitor<br/>Metrics]

    style CI fill:#4a90e2
    style CD fill:#7ed321
    style APPROVE fill:#f9ca24
```

## Blue-Green Deployment

```mermaid
graph TB
    LB[Load Balancer]

    subgraph "Blue Environment (Live)"
        BLUE1[Instance 1<br/>v1.0]
        BLUE2[Instance 2<br/>v1.0]
        BLUE3[Instance 3<br/>v1.0]
    end

    subgraph "Green Environment (New)"
        GREEN1[Instance 1<br/>v1.1]
        GREEN2[Instance 2<br/>v1.1]
        GREEN3[Instance 3<br/>v1.1]
    end

    LB -->|100% Traffic| BLUE1
    LB -->|100% Traffic| BLUE2
    LB -->|100% Traffic| BLUE3

    LB -.->|0% Traffic| GREEN1
    LB -.->|0% Traffic| GREEN2
    LB -.->|0% Traffic| GREEN3

    SWITCH[Switch Traffic] -->|Cutover| LB

    style BLUE1 fill:#4ecdc4
    style BLUE2 fill:#4ecdc4
    style BLUE3 fill:#4ecdc4
    style GREEN1 fill:#7ed321
    style GREEN2 fill:#7ed321
    style GREEN3 fill:#7ed321
```

## Secrets Management

```mermaid
graph TB
    APP[Application] --> SECRET_MGR{Secret<br/>Management}

    SECRET_MGR --> VAULT[HashiCorp Vault]
    SECRET_MGR --> AWS_SM[AWS Secrets Manager]
    SECRET_MGR --> GCP_SM[GCP Secret Manager]
    SECRET_MGR --> AZURE_KV[Azure Key Vault]

    VAULT --> API_KEYS[API Keys]
    VAULT --> DB_CREDS[DB Credentials]
    VAULT --> CERTS[Certificates]

    APP --> ENV[Environment Variables]
    ENV --> VAULT

    style SECRET_MGR fill:#ff6b6b
    style VAULT fill:#4a90e2
```

## Health Checks & Monitoring

```mermaid
graph LR
    subgraph "Health Check System"
        HC[Health Check Endpoint<br/>/health]

        HC --> CHECK1[Database<br/>Connection]
        HC --> CHECK2[Redis<br/>Connection]
        HC --> CHECK3[LLM API<br/>Reachable]
        HC --> CHECK4[Disk Space<br/>Available]

        CHECK1 --> STATUS{All OK?}
        CHECK2 --> STATUS
        CHECK3 --> STATUS
        CHECK4 --> STATUS

        STATUS -->|Yes| HEALTHY[200 OK]
        STATUS -->|No| UNHEALTHY[503 Unavailable]
    end

    LB[Load Balancer] -.->|Poll| HC
    K8S[Kubernetes] -.->|Liveness Probe| HC

    style HC fill:#4a90e2
    style HEALTHY fill:#7ed321
    style UNHEALTHY fill:#ff6b6b
```

## Monitoring Stack

```mermaid
graph TB
    subgraph "Application"
        APP[Agent Service]
    end

    subgraph "Metrics Collection"
        PROM[Prometheus<br/>Metrics DB]
        GRAF[Grafana<br/>Dashboards]
    end

    subgraph "Logging"
        FLUENTD[Fluentd/Filebeat<br/>Log Collector]
        ELK[Elasticsearch<br/>Log Storage]
        KIBANA[Kibana<br/>Log Visualization]
    end

    subgraph "Tracing"
        JAEGER[Jaeger<br/>Distributed Tracing]
    end

    subgraph "Alerting"
        ALERT[Alertmanager]
        PAGER[PagerDuty/Slack]
    end

    APP --> PROM
    APP --> FLUENTD
    APP --> JAEGER

    PROM --> GRAF
    PROM --> ALERT

    FLUENTD --> ELK
    ELK --> KIBANA

    ALERT --> PAGER

    style PROM fill:#e6522c
    style GRAF fill:#f46800
    style ELK fill:#005571
    style JAEGER fill:#60d0e4
```

## Disaster Recovery

```mermaid
graph TB
    PROD[Production<br/>Primary Region] --> BACKUP{Backup<br/>Strategy}

    BACKUP --> DB_BACKUP[Database<br/>Continuous Backup]
    BACKUP --> STATE_BACKUP[State<br/>Snapshots]
    BACKUP --> CONFIG_BACKUP[Config<br/>Version Control]

    DB_BACKUP --> DR_REGION[DR Region<br/>Standby]
    STATE_BACKUP --> DR_REGION
    CONFIG_BACKUP --> DR_REGION

    PROD -.->|Replication| DR_REGION

    FAILOVER[Failover<br/>Trigger] -.-> DR_REGION
    DR_REGION -.->|Promote| ACTIVE[Active<br/>Primary]

    style PROD fill:#4ecdc4
    style DR_REGION fill:#f9ca24
    style ACTIVE fill:#7ed321
```

## Related Topics

- **Day 1**: Agent fundamentals (what we're deploying)
- **Day 23**: Monitoring & Observability (detailed monitoring)
- **Day 24**: Cost Optimization (cost-effective deployment)
- **Day 25**: Security (deployment security)

## Deployment Checklist

```mermaid
mindmap
    root((Production<br/>Deployment))
        Infrastructure
            Provision resources
            Configure networking
            Set up databases
            Deploy storage
        Security
            Set up secrets
            Configure firewalls
            Enable HTTPS
            Audit access
        Application
            Build containers
            Run migrations
            Deploy services
            Configure routing
        Monitoring
            Set up metrics
            Configure alerts
            Enable logging
            Deploy dashboards
        Testing
            Health checks
            Load testing
            Security scan
            E2E validation
```

---

**Related Days**: [Day 23](./day-23-monitoring.md) | [Day 24](./day-24-optimization.md) | [Day 25](./day-25-security.md)
