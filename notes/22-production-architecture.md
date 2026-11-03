# 22. System Design Architecture: Putting Everything Together

## 1. Concept and Fundamentals

A complete system design connects requirements, clients, edge/network, application tiers, data stores, asynchronous processing, observability, security, deployment, scaling, and recovery into one coherent architecture.

There is no universal best architecture. There is a design that best fits requirements, team capability, business risk, growth, and operational capacity.

## 2. Real-World Analogy

A complete city includes roads, districts, utilities, emergency services, traffic control, building codes, warehouses, and recovery plans. Scaling one road without hospitals, electricity, or zoning does not create a functioning city.

## 3. Technical Architecture

Example production e-commerce/healthcare-ready architecture:

```text
Users
 -> Route 53
 -> CloudFront/S3 UI assets
 -> WAF
 -> ALB
 -> ECS/Fargate API autoscaling across 2+ AZs
 -> ElastiCache Redis
 -> RDS/Aurora primary and replicas
 -> S3 documents/backups

API mutations/events
 -> SQS/SNS/EventBridge
 -> worker service/queue consumers
 -> notification/search/analytics systems

Operations
 -> IAM least privilege
 -> Secrets Manager
 -> VPC private subnets/Security Groups
 -> CloudWatch metrics/logs/alarms
 -> OpenTelemetry traces
 -> Jenkins/Terraform/ECR deployments
```

### Workflow Example

1. User loads static assets from S3/CloudFront.
2. API request reaches WAF/ALB and a healthy ECS task.
3. The task authenticates and validates input.
4. Redis serves non-authoritative reads.
5. A transaction writes to RDS and publishes an event.
6. Workers process notifications or document tasks asynchronously.
7. Telemetry records request IDs, latency, errors, and dependency behavior.
8. Autoscaling and backup/failover mechanisms respond to capacity/failure.

## 4. Practical Use Cases

- E-commerce checkout, catalog, payments, fulfillment, and analytics.
- Banking accounts, ledgers, transfers, fraud checks, audit/archive.
- Healthcare appointments, records, document ingestion, notifications.
- SaaS tenant authorization, configuration, usage, billing, reporting.
- Streaming catalogs, playback metadata, upload/processing/CDN delivery.

## 5. DevOps and AWS Implementation

### Network

Use public/private subnets across multiple AZs. ALB/NLB sit at public or internal ingress boundaries. Compute and databases do not need public IPs. Route traffic through security groups, NACLs, endpoints, TLS, and private DNS.

### Compute

EC2/ECS/EKS/Lambda are choices based on runtime/control needs. ECS Fargate reduces host operations. EC2 remains suitable for detailed OS/control needs. Lambda is useful for event-driven bursty work with cost and runtime constraints.

### Data

RDS/Aurora for relational transactions. DynamoDB for known high-scale key/document patterns. Redis for cache/session/rate/work coordination with cache fallback. S3 for objects and lifecycle. Read replicas/partitioning/sharding only at demonstrated need.

### Delivery and Operations

Jenkins builds/tests/scans packages and images. ECR stores immutable images. Terraform declares infrastructure. ECS/Auto Scaling replaces instances/tasks. CloudWatch records metrics/logs; tracing connects distributed operations. Alarms trigger human/actionable response with runbooks.

### Security

Use OIDC/IAM roles, least privilege, KMS, secrets management, TLS, WAF, audit logs, private networks, and security-focused deployment scans. Encrypt backups and restrict restore/deletion operations.

## 6. Practical Production Scenario

A hospital appointment platform uses React/CloudFront/S3 and .NET APIs on ECS behind ALB. Auth is issued through an identity provider. The API uses Redis for reference data but treats RDS MySQL as source of truth. Appointment writes use database transactions and an idempotency key. SQS processes SMS/email reminders. PHI uploads go directly to pre-signed S3 URLs and are scanned asynchronously.

### Failure Matrix

| Failure | Detection | Response |
|---|---|---|
| API task crash | ECS/ALB health | replacement and traffic drain |
| Cache outage | hit/command alarms | bounded database fallback/load shedding |
| Database failover | connection/lag metrics | reconnect/retry and promote standby |
| Queue backlog | visible/dead-letter depth | autoscale workers and protect downstream |
| Region outage | route/health telemetry | declared DR process or failover architecture |
| Bad deployment | error/latency alarms | rollback to known image/config |

### Capacity Flow

CloudFront absorbs static traffic. ALB and ECS handle dynamic traffic. Redis absorbs repetitive reads. RDS handles authoritative transactions; replicas and partitioned stores serve specialized read workloads. Queues absorb variable work and prevent synchronous overload.

## 7. Hands-On Lab: Small Production-Style Stack

Run:

```bash
mkdir -p system-design-lab && cd system-design-lab

cat > compose.yaml <<'EOF'
services:
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
  api:
    image: nginx:alpine
    ports: ["8080:80"]
    depends_on: [redis]
EOF

docker compose up -d
docker compose ps
curl -i http://localhost:8080/

docker compose down
```

This minimal lab does not demonstrate the full architecture; expand it with two API instances, Nginx upstream, PostgreSQL, SQS-style local queue, health endpoints, metrics, and a deployment/rollback script.

Validate:

- Both API replicas receive traffic.
- Redis cache failure triggers fallback behavior.
- Database unavailable response is bounded and retry-safe.
- One API task removal does not interrupt user state.
- Bad image rollback reaches known version.

Cleanup and cost:

```bash
docker compose down --volumes
```

Local lab avoids AWS charges. If using AWS, shut down EC2/ECS, delete test DB/clusters/load balancers, empty/remove S3 objects, remove DNS records, and review the billing account.

## 8. Advantages, Disadvantages and Best Practices

### Design Principles

- Start from requirements and peak assumptions.
- Separate external, application, and data trust boundaries.
- Make state ownership explicit.
- Use health, scaling, timeout, retry, and circuit policies.
- Prefer managed services where operational value exceeds control trade-off.
- Avoid technology without a measured requirement.
- Add resilience incrementally and test it.

### Security

- No public database.
- Short-lived credentials/IAM roles.
- Secrets out of source/image.
- Encryption in transit/at rest.
- Audit mutations and privileged operations.

### Operations

- Dashboards have latency, traffic, errors, saturation, and downstream metrics.
- Alerts include runbooks and owner.
- Deployments are observable and reversible.
- Capacity and backup/recovery tests occur before incidents.

## 9. Interview Preparation

**Q1: How do you begin a design interview?** Clarify functional/NFRs, scale, data, consistency, security, and failure targets.

**Q2: Why use a load balancer?** Stable entry point, health routing, TLS, and horizontal scaling.

**Q3: Why use cache and database?** Cache optimizes reads and database remains authoritative.

**Q4: How achieve availability?** Redundant components in AZs, health-based routing, replacement, and graceful failure handling.

**Q5: How evolve architecture?** Start simple/modular, measure bottlenecks, and add services/queues/replicas/partitions only when justified.

**Scenario:** Flash-sale payment system. Address demand, inventory uniqueness, payment idempotency, rate limits, queueing, database contention, fraud, retries, and recovery.

**Scenario:** A distributed system has outages from cascading retry failures. Add dependency budgets, timeouts, jitter, circuit breakers, queue buffering, load shedding, and per-service capacity.

**Scenario:** Deployment to new region fails consistency checks. Define write authority, replica lag, DNS, identity, encryption, and legal/data residency behavior before cutover.

**60-second answer:** “A production architecture connects a measurable user workflow to a resilient network, stateless compute, managed data stores, asynchronous processing, security, observability, deployment automation, and tested recovery. Components are added for measured requirements, not fashion.”

## 10. Revision Notes

### Architecture Checklist

- Requirements and assumptions.
- Read/write estimates.
- Functional/NFR targets.
- Client/DNS/edge/network path.
- Stateless/scalable compute.
- Cache/database authority.
- Replication/backup/sharding decisions.
- Queue/asynchronous boundaries.
- IAM/security/encryption.
- Metrics/logs/traces/alarms.
- CI/CD, rollback, recovery drills.

### Final Practical Challenge

Design and diagram a hospital telehealth appointment system including:

- React frontend and .NET API.
- Nginx/ALB and Docker/ECS.
- MySQL/RDS/Aurora.
- Redis cache.
- S3 document storage.
- SQS notification worker.
- CloudWatch/OpenTelemetry.
- Multi-AZ and backup/RPO/RTO.
- Security, audit, and data retention.
- One bad deployment and one database failure runbook.
