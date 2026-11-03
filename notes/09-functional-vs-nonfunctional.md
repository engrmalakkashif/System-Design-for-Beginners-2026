# 9. Functional vs Non-Functional Requirements

## 1. Concept and Fundamentals

Functional requirements describe what the system does: APIs, workflows, outputs, rules, and data operations. Non-functional requirements (NFRs) describe how the system behaves: latency, availability, throughput, consistency, security, maintainability, recoverability, and cost.

Examples:

| Type | Example |
|---|---|
| Functional | A user can transfer money and receive an audit record |
| Non-functional | 99.9% of requests complete within 500 ms |
| Functional | Upload a medical document |
| Non-functional | Data is encrypted and accessible only to authorized roles |

### Why They Matter

Technology choices follow measurable requirements. “Fast” is not testable; “p95 < 300 ms at 1,000 RPS” is.

### Without them

Teams make assumptions, argue late, overbuild, or discover outages only after release.

## 2. Real-World Analogy

A car’s functional requirements are steering, braking, carrying passengers, and starting. NFRs are fuel efficiency, crash safety, noise, reliability, maintenance cost, and repairability.

## 3. Technical Architecture

For every requirement, record:

- Scope and workload.
- Target metric and percentile.
- Measurement location.
- Normal and peak conditions.
- Failure mode and recovery target.
- Privacy/security/control requirements.

Architecture maps requirements to services, for example:

```text
API latency target -> ALB/AWS metrics -> compute sizing -> cache/database query budgets
Availability target -> Multi-AZ -> failover -> backup/PITR -> recovery tests
Security target -> VPC/IAM/KMS/WAF -> audit logs -> key rotation
```

## 4. Practical Use Cases

- E-commerce: checkout correctness, availability, p99 latency, fraud control.
- Banking: consistency, auditability, data residency, RPO/RTO.
- Healthcare: authorization, encryption, availability, retention, deletion.
- Video streaming: start time, buffering, throughput, CDN availability.

## 5. DevOps and AWS Implementation

Measure SLI (actual indicator), publish SLO (internal target), and define SLA (external commitment). Use CloudWatch dashboards and alarms.

Examples:

- API availability: `successful requests / total requests`.
- Latency: CloudFront/ALB request p95/p99, not server CPU alone.
- Durability: RDS/S3 backup and failure test results.
- Security: findings, access anomalies, certificate/key expiry.

Alert on burn rate or user impact, and avoid noisy thresholds without context. Infrastructure as code and deployment gates help maintain repeatability.

## 6. Practical Production Scenario

A clinic API requires:

```text
Functional:
- Create/cancel/reschedule appointments
- Notify patient
- Maintain clinical audit record

Non-functional:
- API p95 < 400 ms
- 99.95% monthly availability
- RPO < 5 min; RTO < 30 min
- Encrypt records; role-based access
- Retain audits for compliance period
```

The implementation uses Multi-AZ compute/database, Redis only for non-sensitive non-authoritative data, CloudWatch telemetry, KMS encryption, IAM, and tested recovery.

## 7. Hands-On Lab

Create a requirement table for a small checkout API:

| Requirement | Metric | Load | Validation |
|---|---|---|---|
| Create order | p95 < 300 ms | 500 RPS | Load test |
| Availability | 99.9% | one AZ failure | failover drill |
| Data loss | RPO <= 1 min | primary failure | PITR restore |

Run a basic load test:

```bash
hey -z 60s -c 25 http://localhost:8080/api/health
```

Compare measured results to targets and record whether assumptions held.

## 8. Advantages, Disadvantages and Best Practices

- NFRs make testing and capacity planning objective.
- They reveal cost and availability trade-offs.
- Too many arbitrary targets waste time and money.
- Define regional scope and client-level percentiles.
- Link requirements to architecture components, alerts, and tests.

Do not claim 99.99% availability if dependencies, data backups, or operational recovery cannot support it.

## 9. Interview Preparation

**Q1: What is a functional requirement?** Behavior the system must provide.

**Q2: What is a non-functional requirement?** A quality attribute or operational constraint.

**Q3: What is SLI/SLO/SLA?** SLI is measurement, SLO is internal goal, SLA is contractual commitment.

**Q4: Why is p95 useful?** It describes tail latency better than a simple average.

**Q5: Can requirements conflict?** Yes; strong consistency, low latency, and cross-region availability may conflict and need trade-offs.

**Scenario:** The interviewer says the system must “handle millions.” Ask for peak RPS, read/write ratio, payload size, concurrency, and growth assumptions.

**Scenario:** Product says “always available.” Clarify target uptime, maintenance, regional scope, data freshness, and dependency failures.

**60-second answer:** “Functional requirements specify behavior; non-functional requirements specify measurable qualities such as latency, availability, security, consistency, and recovery.”

## 10. Revision Notes

- Function = what.
- NFR = quality/constraint.
- Use percentiles, peak traffic, and recovery targets.
- SLI → measurement; SLO → target; SLA → promise.

**Challenge:** Write 10 requirements for an online exam platform, including latency, availability, consistency, cheating detection, recovery, and audit needs.
