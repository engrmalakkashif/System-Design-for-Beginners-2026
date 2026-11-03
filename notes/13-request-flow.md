# 13. How a Request Travels Through a System

## 1. Concept and Fundamentals

A request travels through networking, security, routing, application logic, data stores, and observability layers. Understanding the path helps locate latency and failures.

### Core Flow

DNS → edge/CDN → load balancer → service → cache/database/dependencies → response.

### Why It Matters

Each hop introduces latency, limits, retries, and failure modes. The full tail latency is the sum of dependencies plus queueing and processing time.

## 2. Real-World Analogy

Sending a package travels from the sender through postal routing, sorting centers, local delivery, and return tracking. One delayed sorting center delays the whole delivery even if the driver is fast.

## 3. Technical Architecture

```text
Browser
 -> DNS/Route 53
 -> CDN/CloudFront
 -> WAF
 -> ALB/API Gateway
 -> application pod/task
 -> Auth/IAM policy
 -> cache
 -> RDS/NoSQL
 -> queue/downstream service
 <- response/error
```

The application should preserve a correlation ID, enforce timeouts, validate authorization, avoid trusting client identifiers, and emit logs and metrics.

## 4. Practical Use Cases

- Checkout request writes an order, checks inventory, calls payment, and publishes event.
- Appointment request checks authentication, availability, conflicts, and notifications.
- Video playback request resolves CDN token/URL and fetches media from S3-backed distribution.

## 5. DevOps and AWS Implementation

Trace the request with ALB access logs, application logs, distributed tracing, CloudWatch metrics, Route 53 health checks, and downstream dependency IDs. API Gateway and ALB can enforce TLS/limits while services enforce authorization and business rules.

Set DNS TTL based on failover needs. Security groups control network reachability. IAM roles govern AWS service access. WAF rules protect public endpoints. CloudFront can serve static content without reaching origin.

Every outbound call needs:

- Connect/read timeout.
- Bounded retry count.
- Backoff/jitter.
- Circuit breaker or fallback when appropriate.
- Correlation and trace IDs.

## 6. Practical Production Scenario

A React app calls `POST /appointments`. CloudFront serves the UI, while `/api` routes to ALB. ALB TLS-terminates and selects a .NET task. The task authenticates through IAM-backed identity, checks Redis availability, opens a bounded MySQL transaction, writes audit and appointment data, then enqueues reminder processing.

If SQS throttles, the API does not falsely report failure for the appointment. It records retryable notification status and returns appointment success. If MySQL fails, the API returns a retryable error and uses an idempotency key to avoid duplicate bookings.

## 7. Hands-On Lab

Inspect one request locally with curl and logs:

```bash
curl -v http://localhost:8080/api/health
```

Use DNS resolution and network testing:

```bash
getent hosts localhost
ss -ltnp | grep 8080
```

Add a log line containing request ID, route, status, duration, and downstream duration. Then make a database call slow and verify whether p95 latency rises and timeouts prevent request accumulation.

## 8. Advantages, Disadvantages and Best Practices

- Follow one successful request and one failure request end-to-end.
- Define timeout budgets for every hop.
- Keep retries bounded and idempotent.
- Avoid exposing internal stack traces.
- Record trace IDs in all logs.

Do not retry non-idempotent operations without an idempotency mechanism.

## 9. Interview Preparation

**Q1: What are the first network hops?** Client DNS, edge/CDN/WAF, and load balancer/API gateway.

**Q2: What is request latency made of?** Client/network, proxy, queueing, processing, and dependency time.

**Q3: What is correlation ID?** Identifier propagated across logs and services for one operation.

**Q4: Why are downstream timeouts important?** They prevent threads/connections from accumulating indefinitely.

**Q5: What makes retries safe?** Idempotency, bounded retry count, backoff/jitter, and handling transient failures.

**Scenario:** Homepage loads slowly. Separate edge cache, origin, app, database, and third-party timings before scaling infrastructure.

**Scenario:** Duplicate payments occur after retry. Add request/event idempotency keys and transaction/uniqueness constraints.

**60-second answer:** “A request flows through DNS, edge security, load routing, application logic, and dependencies before returning. Timeouts, tracing, validation, authorization, and bounded retries make that journey observable and recoverable.”

## 10. Revision Notes

- Follow DNS → edge → load balancer → app → dependency → response.
- Every dependency needs timeout/retry policy.
- Preserve request and trace IDs.
- Validate input and authorize before mutations.
- Measure each hop and tail latency.

**Challenge:** Draw the request flow for one checkout operation and mark timeout, consistency, and failure behavior on each hop.
