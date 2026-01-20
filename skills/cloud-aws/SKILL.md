---
name: cloud-aws
description: AWS (Amazon Web Services) expert. Covers Lambda, S3, DynamoDB, IAM, CloudWatch, API Gateway, and all AWS services. Use for any AWS question.
model: opus
---

# AWS Skill

Apply these AWS-specific patterns and practices when working with Amazon Web Services.

## Core Services

### Compute

| Service | Use Case | Considerations |
|---------|----------|----------------|
| Lambda | Event-driven, short tasks (<15 min) | Cold starts, 10GB memory max |
| ECS/Fargate | Containers, long-running processes | No server management with Fargate |
| EKS | Kubernetes workloads | Complex, higher operational overhead |
| EC2 | Full control, specialized hardware | You manage patching and scaling |

### API & Networking

| Service | Use Case | Considerations |
|---------|----------|----------------|
| API Gateway | REST/HTTP/WebSocket APIs | Request validation, throttling built-in |
| ALB | Load balancing, path routing | WebSocket support, cheaper than API GW |
| CloudFront | CDN, edge caching | SSL termination, Lambda@Edge |
| Route 53 | DNS, health checks | Weighted/geolocation routing |
| VPC | Network isolation | Plan CIDR blocks carefully |

### Data Storage

| Service | Use Case | Considerations |
|---------|----------|----------------|
| DynamoDB | Key-value, <10ms latency | Design access patterns first |
| RDS/Aurora | Relational, complex queries | Aurora Serverless for variable load |
| ElastiCache | Caching, sessions | Redis vs Memcached |
| S3 | Objects, static assets | Storage classes for cost optimization |

### Messaging & Events

| Service | Use Case | Considerations |
|---------|----------|----------------|
| SQS | Queue-based decoupling | Standard vs FIFO queues |
| SNS | Pub/sub, fan-out | Message filtering |
| EventBridge | Event routing, scheduling | Cross-account, SaaS integration |
| Kinesis | Real-time streaming | Shard management |

### Security

| Service | Use Case |
|---------|----------|
| IAM | Identity, roles, policies |
| Cognito | User auth, OAuth/OIDC |
| Secrets Manager | Credential storage, rotation |
| KMS | Encryption key management |
| WAF | Web application firewall |

### Observability

| Service | Use Case |
|---------|----------|
| CloudWatch | Logs, metrics, alarms |
| X-Ray | Distributed tracing |
| CloudTrail | API audit logging |

## Lambda Patterns

### Best Practices
- Initialize SDK clients outside handler (reuse connections)
- Use environment variables for configuration
- Keep handlers thin, business logic in separate modules
- Set appropriate memory (CPU scales with memory)
- Use provisioned concurrency for latency-sensitive workloads

### Event Sources
- API Gateway (sync) - REST/HTTP APIs
- SQS (async) - Queue processing with batching
- EventBridge (async) - Event-driven workflows
- S3 (async) - Object notifications
- DynamoDB Streams (async) - Change data capture
- Kinesis (async) - Real-time streaming

### Cold Start Mitigation
- Minimize deployment package size
- Use ARM64 architecture (Graviton2)
- Lazy load dependencies
- Consider provisioned concurrency for critical paths

## DynamoDB Design

### Single Table Design

```
PK                  | SK                  | Attributes
--------------------|---------------------|------------------
USER#123            | PROFILE             | name, email...
USER#123            | ORDER#2024-001      | total, status...
ORDER#2024-001      | ORDER#2024-001      | userId, items...
ORDER#2024-001      | ITEM#SKU-123        | quantity, price...
```

### Access Pattern Mapping

| Pattern | Key Design |
|---------|------------|
| Get user profile | PK=USER#id, SK=PROFILE |
| List user orders | PK=USER#id, SK begins_with ORDER# |
| Get order with items | PK=ORDER#id |
| Query by status | GSI: GSI1PK=STATUS#pending |

### GSI Strategies
- Overload GSIs for multiple access patterns
- Use sparse indexes (only items with GSI keys are indexed)
- Consider GSI projections (KEYS_ONLY, INCLUDE, ALL)

### Capacity Planning
- On-demand: Variable/unpredictable traffic
- Provisioned: Predictable traffic, cost savings
- Auto-scaling: Provisioned with flexibility

## IAM Best Practices

### Least Privilege
- Start with minimal permissions, add as needed
- Use resource-level permissions (not *)
- Separate roles per function/service
- Use conditions to restrict access

### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:Query"],
      "Resource": "arn:aws:dynamodb:REGION:ACCOUNT:table/TABLE_NAME",
      "Condition": {
        "ForAllValues:StringEquals": {
          "dynamodb:LeadingKeys": ["USER#${aws:userId}"]
        }
      }
    }
  ]
}
```

### Service Roles
- Lambda execution role: logs, X-Ray, specific service access
- ECS task role: application permissions
- ECS execution role: pull images, write logs

## Serverless Architecture

### API Pattern

```
CloudFront → API Gateway → Lambda → DynamoDB
                  ↓
              Cognito (auth)
```

### Event-Driven Pattern

```
Source → EventBridge → Lambda → Target
              ↓
         SQS (DLQ)
```

### Async Processing

```
API Gateway → Lambda → SQS → Lambda → DynamoDB
                         ↓
                   DLQ (failures)
```

## Cost Optimization

### Compute
- Right-size Lambda memory (test with Power Tuning)
- Use Graviton2 (ARM64) for better price/performance
- Spot instances for fault-tolerant workloads
- Reserved capacity for predictable baselines

### Storage
- S3 lifecycle policies (transition to cheaper tiers)
- DynamoDB on-demand vs provisioned analysis
- Delete unused EBS snapshots and volumes

### Networking
- VPC endpoints to avoid NAT Gateway costs
- CloudFront for reduced data transfer
- Evaluate cross-region data transfer

## Security Checklist

- [ ] Secrets in Secrets Manager (not environment variables)
- [ ] Encryption at rest enabled (S3, DynamoDB, RDS)
- [ ] Encryption in transit (TLS everywhere)
- [ ] VPC for sensitive workloads
- [ ] Security groups with minimal ingress
- [ ] CloudTrail enabled for audit logging
- [ ] AWS Config for compliance monitoring
- [ ] GuardDuty for threat detection

## Reliability Patterns

### Multi-AZ Deployment
- RDS Multi-AZ for database failover
- ALB across multiple AZs
- Lambda automatically multi-AZ

### Disaster Recovery
- Cross-region replication (S3, DynamoDB Global Tables)
- Backup retention policies
- Documented recovery procedures

### Circuit Breaker
- Implement timeouts on all external calls
- Use Step Functions for complex workflows with retry
- DLQs for failed async processing

## Observability Strategy

### Logging
- Structured JSON logs
- Correlation IDs across services
- Log levels: ERROR, WARN, INFO, DEBUG
- CloudWatch Log Insights for querying

### Metrics
- Business metrics (orders, signups)
- Technical metrics (latency, errors, throttles)
- Custom CloudWatch metrics for application-specific data

### Alarms
- Error rate thresholds
- Latency percentiles (p99, p95)
- Throttling detection
- DLQ message count

### Tracing
- X-Ray for distributed tracing
- Trace sampling for high-traffic services
- Service maps for dependency visualization
