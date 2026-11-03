# 10. Five Questions Before Designing a System

## 1. Concept and Fundamentals

Before drawing components, obtain answers that determine architecture:

1. **What is the user’s core action?**
2. **How much traffic and data?**
3. **How quickly and correctly must reads/writes respond?**
4. **Which failures and recovery objectives matter?**
5. **Which security, cost, consistency, and compliance constraints apply?**

A useful sixth question is “Which data is the source of truth?”

### Why These Questions Matter

They turn vague requests into capacity, latency, data, failure, and trust-boundary decisions. The answers prevent generic architecture diagrams.

### Without them

Architects assume scale, choose databases before access patterns, ignore failure behavior, and add unused components.

## 2. Real-World Analogy

Before building a house, ask who lives there, peak occupancy, desired materials, earthquake/fire risks, budget, and local codes.

## 3. Technical Architecture

```text
Requirements discovery
 -> Estimate traffic/data growth
 -> Define consistency/latency/availability
 -> Define threat/security/compliance boundaries
 -> Define failure/recovery targets
 -> Select components and trade-offs
 -> Validate through load/recovery/security tests
```

For a URL shortener, estimates might be 100 write RPS/1,000 read RPS, 100 million links/year, 10-year retention, 99.9% availability, and sub-100 ms redirect reads.

## 4. Practical Use Cases

- E-commerce checkout: peak orders, inventory consistency, payment idempotency, fraud latency.
- Telehealth: clinician availability, privacy, audit records, appointment conflict rules.
- Streaming: concurrent viewers, video formats, global delivery, write frequency.
- Internal SaaS: authentication, tenant isolation, tenant data volume, reporting.

## 5. DevOps and AWS Implementation

Model AWS capacity based on estimated traffic and payload size. Ask whether CloudFront, Route 53, ALB, ECS, Lambda, S3, RDS/DynamoDB, Redis, SQS, and KMS fit measurable requirements.

Make assumptions visible:

- Peak requests per second.
- Request/response sizes.
- Read/write ratio.
- Number of users/tenants.
- Growth over 12-36 months.
- Multi-AZ versus multi-region.
- RPO/RTO and compliance.

## 6. Practical Production Scenario

Product: “Build a file scan and approval service.” Questions reveal:

1. Core action: upload, scan, approve/reject, download.
2. Traffic: 200 uploads/minute peak, 50 MB average files.
3. Latency: upload should accept quickly; scanning can be asynchronous.
4. Availability: 99.9%; RPO 5 min; RTO 30 min.
5. Security: malware scan, encryption, tenant isolation, audit log, limited download.

Architecture becomes S3 presigned upload, malware-processing queue/workers, metadata database, immutable audit records, CloudWatch alarms, and separate read/download authorization.

## 7. Hands-On Lab

Use a requirement worksheet for a parking reservation system:

```text
1. Core actions: search, reserve, extend, cancel, pay.
2. Peak traffic: 2,000 searches/sec; 100 reservations/sec.
3. Requirements: p95 < 500 ms; atomic reservation; no double booking.
4. Failures: database AZ outage; payment timeout.
5. Security: authentication; PCI scope; audit; cost budget.
6. Source of truth: reservation database and ledger integration status.
```

Then sketch one architecture and identify two components that are not needed by the assumptions.

## 8. Advantages, Disadvantages and Best Practices

- Improves answer quality in interviews and reviews.
- Makes cost and operational trade-offs explicit.
- Avoids premature technology selection.
- State assumptions instead of pretending data exists.
- Revisit estimates after load tests.

Do not invent precise metrics from no evidence; label them as requirements or assumptions and explain alternatives.

## 9. Interview Preparation

**Q1: What should the first interview question be?** Clarify the main user workflow and scale.

**Q2: Why ask about traffic?** It determines capacity, replicas, cache, queues, and database load.

**Q3: Why ask about consistency?** It determines transaction and replica/partition behavior.

**Q4: Why ask about availability?** It determines redundancy, failover, cost, and operational complexity.

**Q5: Why ask about source of truth?** It identifies authority and prevents conflicting data mutations.

**Scenario:** Design food delivery. Clarify live location frequency, order accuracy, payment retries, restaurant availability, and peak use.

**Scenario:** Design file storage. Clarify file size, read/write ratio, sharing model, deletion, encryption, region, and synchronization semantics.

**60-second answer:** “Before choosing technology, clarify the core workflow, scale, latency/consistency, failure/recovery objectives, and security/cost constraints. This turns a system design into decisions based on evidence.”

## 10. Revision Notes

1. Core action.
2. Scale and data.
3. Latency/throughput/consistency.
4. Failure/recovery.
5. Security/cost/compliance.
6. Source of truth.

**Challenge:** Apply these six questions to a ride-sharing app and state three assumptions explicitly.
