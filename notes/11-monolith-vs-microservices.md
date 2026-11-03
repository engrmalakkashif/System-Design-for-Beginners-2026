# 11. Monolith vs Microservices

## 1. Concept and Fundamentals

A monolith deploys one application unit containing many business capabilities. A modular monolith keeps boundaries inside the code but shares a deployment unit. Microservices deploy capabilities as independently operating services.

### What Problem Each Solves

Monoliths reduce deployment, transaction, debugging, and network complexity. Microservices enable independent scaling/deployment and team ownership, but introduce distributed-system operations.

### Without Architecture Intent

A “monolith” becomes a spaghetti application; “microservices” become distributed modules with shared databases, synchronous chains, and no ownership.

## 2. Real-World Analogy

A monolith is a restaurant under one kitchen and management system. Microservices are separate restaurants: specialist operations can scale independently, but coordination, menus, accounting, and delivery become more complex.

## 3. Technical Architecture

### Modular Monolith

```text
Single API deployment
 -> Modules: User, Catalog, Orders, Payments
 -> shared database with controlled module boundaries
```

### Microservices

```text
API Gateway/ALB
 -> Identity Service
 -> Catalog Service
 -> Order Service
 -> Payment Service

Each service -> own database/store
Events -> queue/topic for collaboration
```

Communication choices: synchronous HTTP/gRPC for immediate results, asynchronous events for eventual workflows, and message queues for buffering.

## 4. Practical Use Cases

- Monolith: early-stage SaaS, small team, low operational maturity, simple transactional flows.
- Microservices: large organizations, separately changing domains, independent scaling, strong platform capability.
- Modular monolith: common bridge while discovering stable service boundaries.

## 5. DevOps and AWS Implementation

A monolith can run on EC2/ECS, Jenkins CI, ALB, Auto Scaling, and RDS. Microservices add service discovery, API gateway, container orchestration, distributed logs/traces, secrets, queues/topics, deployment strategies, and service-level ownership.

AWS options include ECS/EKS for containers, SQS/SNS/EventBridge for messaging, API Gateway/App Mesh/Cloud Map for APIs/service routing, and CloudWatch/X-Ray/OpenTelemetry for observability.

Start microservices only after teams, release boundaries, data ownership, and operational tooling support them.

## 6. Practical Production Scenario

An order system begins as a modular monolith. As checkout grows independently, checkout is extracted to an ECS service with its own database and publishes events for fulfillment. Inventory validation remains local initially and uses an API/event contract with timeouts and compensation.

Failure handling includes retries with bounded budgets, circuit breakers, dead-letter queues, correlation IDs, and idempotent event processing.

## 7. Hands-On Lab

Use Docker Compose to run a two-service design:

```yaml
services:
  catalog:
    build: ./catalog
    expose: [8080]
  order:
    build: ./order
    expose: [8080]
    environment:
      CATALOG_URL: http://catalog:8080
```

Add Docker health checks. Stop catalog and verify order returns a controlled timeout/fallback. Restore catalog and verify service recovery. This demonstrates distributed dependency behavior without cloud cost.

## 8. Advantages, Disadvantages and Best Practices

### Monolith Benefits

- Simple deployments and transactions.
- Easier local debugging.
- Lower network/operational cost.

### Monolith Limitations

- Independent component scaling is difficult.
- One deployable unit can increase blast radius.
- Long-term boundaries can decay.

### Microservices Benefits

- Independent ownership/deployments/scaling.
- Technology choices may differ per service.

### Microservices Limitations

- Network failures and partial consistency.
- Distributed tracing and data ownership complexity.
- More CI/CD, security, cost, and operational overhead.

## 9. Interview Preparation

**Q1: Monolith vs microservices?** Deployment granularity and ownership complexity differ.

**Q2: When choose a monolith?** Small team/domain, early stage, simple operations, predictable traffic.

**Q3: When choose microservices?** Stable domain boundaries, independent scaling/deployment needs, sufficient platform maturity.

**Q4: What is a modular monolith?** A single deployable unit with strict internal domain boundaries.

**Q5: Are shared databases allowed?** Microservices should avoid shared ownership; otherwise services share coupling and migrations.

**Scenario:** A monolith deployment fails and all features are unavailable. Introduce modular boundaries, safer deploys, autoscaling, and extract only proven bottleneck/ownership domains.

**Scenario:** Twenty microservices depend on one synchronous call chain. Replace fragile chains with asynchronous workflows and enforced timeouts/idempotency.

**60-second answer:** “A monolith is simpler and better for early systems; microservices add independent scaling and ownership at the cost of distributed operations. A modular monolith is often the best first step.”

## 10. Revision Notes

- Start with boundaries, not service count.
- Microservices require automation and observability.
- Shared database often defeats service independence.
- Use asynchronous messaging only where eventual consistency is acceptable.

**Challenge:** Convert a simple bookstore monolith into three service boundaries and explain what data/event ownership belongs to each.
