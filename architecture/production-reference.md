# Reference Architecture: Production Agentic AI System

## Overview

This document outlines a production-ready architecture for deploying agentic AI systems at scale. Based on real-world deployments handling 1M+ requests/month.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway                           │
│                  (Authentication, Rate Limiting)             │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   Agent Orchestrator                         │
│              (Routing, Load Balancing)                       │
└─────┬──────────┬──────────┬──────────┬──────────────────────┘
      │          │          │          │
┌─────▼───┐ ┌───▼────┐ ┌───▼────┐ ┌───▼────┐
│Agent A  │ │Agent B │ │Agent C │ │Agent D │
│Runtime  │ │Runtime │ │Runtime │ │Runtime │
└─────┬───┘ └───┬────┘ └───┬────┘ └───┬────┘
      │         │          │          │
      └─────────┴──────────┴──────────┘
                     │
      ┌──────────────┼──────────────┐
      │              │              │
┌─────▼─────┐  ┌────▼────┐  ┌──────▼──────┐
│  Memory   │  │ Gateway │  │Observability│
│  (STM/LTM)│  │ (Tools) │  │ (Traces)    │
└───────────┘  └─────────┘  └─────────────┘
```

## Components

### 1. API Gateway

**Purpose:** Entry point for all requests

**Responsibilities:**
- Authentication (API keys, OAuth)
- Rate limiting
- Request validation
- SSL termination

**Technology Options:**
- AWS API Gateway
- Kong
- Nginx

**Configuration:**
```yaml
rate_limits:
  per_user: 100 requests/minute
  per_ip: 1000 requests/minute
  
authentication:
  type: bearer_token
  validation: jwt
  
timeout: 30 seconds
```

### 2. Agent Orchestrator

**Purpose:** Routes requests to appropriate agents

**Responsibilities:**
- Request classification
- Load balancing
- Circuit breaking
- Retry logic

**Implementation:**
```python
class AgentOrchestrator:
    def route_request(self, request):
        # Classify request type
        agent_type = self.classifier.classify(request)
        
        # Get healthy agent instance
        agent = self.load_balancer.get_agent(agent_type)
        
        # Execute with retry
        return self.execute_with_retry(agent, request)
```

### 3. Agent Runtime

**Purpose:** Executes agent logic

**Options:**

**A. AWS Bedrock AgentCore**
```yaml
runtime:
  type: agentcore
  memory: enabled
  gateway: enabled
  observability: enabled
```

**B. Self-Hosted (ECS/EKS)**
```yaml
runtime:
  type: container
  image: agent:latest
  replicas: 3
  resources:
    cpu: 2
    memory: 4Gi
```

### 4. Memory Layer

**Purpose:** Persistent agent memory

**Types:**

**Short-Term Memory (STM):**
- Session-based
- Redis/ElastiCache
- TTL: 1 hour

**Long-Term Memory (LTM):**
- Cross-session
- DynamoDB/PostgreSQL
- Indexed for fast retrieval

**Schema:**
```sql
CREATE TABLE agent_memory (
    user_id VARCHAR(255),
    session_id VARCHAR(255),
    memory_type VARCHAR(50), -- 'fact', 'preference', 'summary'
    content TEXT,
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    INDEX idx_user_session (user_id, session_id)
);
```

### 5. Gateway (Tools)

**Purpose:** Connect agents to external systems

**Integration Patterns:**

**A. REST APIs**
```python
@tool
def get_customer_data(customer_id: str):
    response = requests.get(
        f"{CRM_API}/customers/{customer_id}",
        headers={"Authorization": f"Bearer {token}"}
    )
    return response.json()
```

**B. Database Queries**
```python
@tool
def query_orders(customer_id: str):
    return db.execute(
        "SELECT * FROM orders WHERE customer_id = ?",
        [customer_id]
    )
```

**C. MCP Tools**
```yaml
tools:
  - name: slack
    type: mcp
    endpoint: slack-mcp-server
  - name: jira
    type: mcp
    endpoint: jira-mcp-server
```

### 6. Observability

**Purpose:** Monitor and debug agents

**Components:**

**Tracing:**
- OpenTelemetry
- AWS X-Ray
- Datadog APM

**Logging:**
- CloudWatch Logs
- ELK Stack
- Splunk

**Metrics:**
- Prometheus
- CloudWatch Metrics
- Grafana

**Key Metrics:**
```yaml
metrics:
  - agent_requests_total
  - agent_latency_seconds
  - agent_errors_total
  - agent_token_usage
  - agent_cost_dollars
```

## Data Flow

### Request Flow

```
1. User Request
   ↓
2. API Gateway (auth, rate limit)
   ↓
3. Orchestrator (route to agent)
   ↓
4. Agent Runtime
   ├─ Load memory (STM/LTM)
   ├─ Execute agent logic
   ├─ Call tools via Gateway
   ├─ Generate response
   └─ Save memory
   ↓
5. Response to user
   ↓
6. Log to Observability
```

### Example: Customer Support Query

```
User: "What's my order status?"
  ↓
API Gateway: Validates token, checks rate limit
  ↓
Orchestrator: Routes to "customer_support_agent"
  ↓
Agent Runtime:
  1. Loads user memory (name, preferences)
  2. Calls get_customer_data(user_id)
  3. Calls query_orders(customer_id)
  4. Generates response
  5. Saves interaction to memory
  ↓
Response: "Your order #12345 shipped yesterday, arriving tomorrow"
  ↓
Observability: Logs trace, metrics, cost
```

## Scaling Patterns

### Horizontal Scaling

**Agent Instances:**
```yaml
autoscaling:
  min_instances: 2
  max_instances: 20
  target_cpu: 70%
  target_requests: 100/instance
```

**Database:**
- Read replicas for memory queries
- Caching layer (Redis)
- Connection pooling

### Vertical Scaling

**When to Use:**
- Complex agent logic
- Large context windows
- Memory-intensive operations

**Configuration:**
```yaml
resources:
  cpu: 4 cores
  memory: 16Gi
  gpu: optional (for local models)
```

## Security

### Authentication & Authorization

**API Level:**
- API keys for service-to-service
- OAuth 2.0 for user access
- JWT tokens with expiration

**Agent Level:**
- IAM roles for AWS resources
- Service accounts for Kubernetes
- Secrets management (AWS Secrets Manager)

### Data Protection

**In Transit:**
- TLS 1.3 for all connections
- Certificate pinning

**At Rest:**
- Encrypted databases (KMS)
- Encrypted S3 buckets
- Encrypted memory stores

**PII Handling:**
- Detect and mask PII
- Audit logs for data access
- Data retention policies

### Network Security

**VPC Configuration:**
```yaml
vpc:
  private_subnets:
    - agent_runtime
    - databases
  public_subnets:
    - api_gateway
    - load_balancer
  
  security_groups:
    - agent_sg: allow from orchestrator only
    - db_sg: allow from agents only
```

## Cost Optimization

### Model Selection

**By Use Case:**
- Simple queries: Claude Haiku ($0.25/1M tokens)
- Complex reasoning: Claude Sonnet ($3/1M tokens)
- Specialized: Fine-tuned models

### Caching

**Response Caching:**
```python
@cache(ttl=3600)
def handle_faq(question):
    return agent(question)
```

**Prompt Caching:**
- Cache system prompts
- Cache knowledge base context
- Save 90% on repeated context

### Resource Management

**Right-Sizing:**
- Monitor actual usage
- Scale down during off-hours
- Use spot instances for non-critical

**Cost Monitoring:**
```yaml
alerts:
  - cost_per_request > $0.10
  - daily_cost > $1000
  - token_usage_spike > 2x average
```

## Disaster Recovery

### Backup Strategy

**Memory:**
- Daily snapshots
- Point-in-time recovery
- Cross-region replication

**Configuration:**
- Version control (Git)
- Infrastructure as Code (Terraform)
- Automated backups

### Failover

**Multi-Region:**
```yaml
regions:
  primary: us-west-2
  secondary: us-east-1
  
failover:
  automatic: true
  rpo: 1 hour
  rto: 15 minutes
```

**Circuit Breaker:**
```python
@circuit_breaker(
    failure_threshold=5,
    timeout=60,
    fallback=cached_response
)
def call_agent(request):
    return agent.invoke(request)
```

## Deployment Pipeline

### CI/CD

```yaml
stages:
  - test:
      - unit_tests
      - integration_tests
      - load_tests
  
  - deploy_staging:
      - deploy_agents
      - smoke_tests
      - performance_tests
  
  - deploy_production:
      - blue_green_deployment
      - canary_release (10% traffic)
      - full_rollout
```

### Testing Strategy

**Unit Tests:**
- Agent logic
- Tool integrations
- Memory operations

**Integration Tests:**
- End-to-end flows
- Multi-agent scenarios
- Error handling

**Load Tests:**
- 1000 requests/second
- Sustained load (1 hour)
- Spike tests

## Monitoring & Alerts

### Key Dashboards

**Operational:**
- Requests per second
- Latency (p50, p95, p99)
- Error rate
- Agent availability

**Business:**
- Cost per request
- User satisfaction
- Task completion rate
- ROI metrics

**Technical:**
- Token usage
- Model latency
- Tool call success rate
- Memory hit rate

### Alert Rules

```yaml
alerts:
  critical:
    - error_rate > 5%
    - latency_p99 > 10s
    - agent_down
  
  warning:
    - error_rate > 1%
    - latency_p95 > 5s
    - cost_spike > 50%
```

## Best Practices

### Do's ✅

- Start simple, add complexity as needed
- Monitor everything from day one
- Implement circuit breakers
- Cache aggressively
- Use managed services when possible
- Test failure scenarios
- Document agent behavior

### Don'ts ❌

- Don't skip observability
- Don't ignore cost monitoring
- Don't deploy without testing
- Don't hard-code credentials
- Don't skip security reviews
- Don't over-engineer initially

## Technology Stack Recommendations

### For Startups

**Managed Services:**
- AWS Bedrock AgentCore (runtime)
- API Gateway (entry point)
- DynamoDB (memory)
- CloudWatch (observability)

**Why:** Fast to deploy, minimal ops overhead

### For Scale-Ups

**Hybrid:**
- AgentCore or self-hosted (ECS)
- Kong (API gateway)
- PostgreSQL + Redis (memory)
- Datadog (observability)

**Why:** More control, cost optimization

### For Enterprise

**Self-Hosted:**
- Kubernetes (EKS)
- Custom orchestration
- PostgreSQL cluster
- Enterprise observability stack

**Why:** Maximum control, compliance, multi-cloud

## Migration Path

### Phase 1: Prototype (Weeks 1-2)
- Single agent
- No memory
- Basic observability

### Phase 2: MVP (Weeks 3-6)
- Multi-agent
- Session memory
- Full observability

### Phase 3: Production (Weeks 7-12)
- Auto-scaling
- Long-term memory
- Advanced features

### Phase 4: Scale (Month 4+)
- Multi-region
- Advanced caching
- Cost optimization

---

**Author:** Nida Beig  
**Last Updated:** January 2026  
**Based on:** 10+ production deployments
