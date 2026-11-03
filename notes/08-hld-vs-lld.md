# 8. HLD vs LLD

## 1. Concept and Fundamentals

High-Level Design (HLD) describes major components, relationships, data flow, scaling, availability, and technology choices. Low-Level Design (LLD) describes implementation details such as APIs, classes, database schemas, state transitions, algorithms, error handling, and configuration.

### HLD Example

Client → ALB → API service → Redis → RDS; async jobs → SQS → workers → S3.

### LLD Example

`POST /orders` validates fields, checks inventory, creates an order transaction, publishes an event, and returns an idempotency key.

## 2. Real-World Analogy

HLD is a city map: districts, roads, and utilities. LLD is the engineering plan for one building: rooms, wiring, pipes, doors, and dimensions.

## 3. Technical Architecture

HLD decisions:

- Trust boundaries.
- Component ownership.
- Synchronous and asynchronous flows.
- Database and cache boundaries.
- Failure mode and deployment topology.

LLD decisions:

- Endpoint contracts.
- Tables, indexes, constraints.
- DTO and domain model.
- Locking and transaction boundaries.
- Idempotency keys and retries.

## 4. Practical Use Cases

- Design review for a new payment platform.
- Architecture review before cloud migration.
- API implementation by an individual engineering squad.

## 5. DevOps and AWS Implementation

HLD includes AWS regions/AZs, network tiers, managed services, CI/CD boundaries, monitoring, and disaster recovery. LLD includes ECS task definitions, ALB listener rules, Terraform resources, IAM policies, health paths, environment variables, and database migration scripts.

Both levels should include security, observability, operational runbooks, and rollback strategy.

## 6. Practical Production Scenario

HLD: a document-processing service receives uploads through API Gateway/S3, queues jobs in SQS, runs workers in ECS, and stores metadata in RDS.

LLD: define upload object key, presigned URL lifetime, event schema, idempotency key, retry count, dead-letter queue, status states, and worker transaction behavior.

## 7. Hands-On Lab

Write both artifacts for a URL-shortener:

1. HLD diagram and data flow.
2. API contracts:

```text
POST /links {longUrl} -> {shortCode, expiresAt}
GET /{shortCode} -> 302 redirect
```

3. Table: `links(short_code pk, long_url, created_at, expires_at, owner_id)`.
4. Cache key: `link:<shortCode>`.
5. Failure behavior: duplicate link, cache miss, expired code, database unavailable.

Validate each contract with tests before infrastructure implementation.

## 8. Advantages, Disadvantages and Best Practices

- HLD improves alignment and risk discovery before implementation.
- LLD prevents ambiguous behavior and hidden dependencies.
- Keep diagrams close to deployment and update ADRs for decisions.
- Do not let low-level detail slow requirement discovery.
- Do not specify technologies in HLD without a technical reason.

## 9. Interview Preparation

**Q1: Difference between HLD and LLD?** HLD maps system components; LLD specifies implementation behavior.

**Q2: What belongs in HLD?** Scaling, data flow, boundaries, availability, dependencies, security, and decisions.

**Q3: What belongs in LLD?** APIs, schemas, classes/functions, constraints, state transitions, and algorithms.

**Q4: Why both?** One communicates architecture; the other makes implementation and testing precise.

**Q5: What is an ADR?** Architecture Decision Record documenting a decision, alternatives, context, and consequences.

**Scenario:** Interviewer asks to design Twitter. Start with HLD requirements and component flow, then choose one area, such as timeline fan-out, for an LLD deep dive.

**Scenario:** Microservice team misses deadlines. Examine whether service boundaries and contracts were defined at HLD/LLD before coding.

**60-second answer:** “HLD answers what major parts the system has and how they interact; LLD answers exactly how a component implements behavior, state, APIs, and storage.”

## 10. Revision Notes

- HLD: system context and component interaction.
- LLD: detailed implementation contracts.
- Requirements precede either.
- Record decisions and alternatives.

**Challenge:** Produce an HLD and one LLD API contract for an appointment-booking service.
