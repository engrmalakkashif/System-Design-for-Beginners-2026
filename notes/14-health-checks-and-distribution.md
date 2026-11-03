# 14. Health Checks and Traffic Distribution

## 1. Concept and Fundamentals

Health checks determine whether targets should receive traffic. Traffic algorithms decide which healthy target receives a request.

Two checks should differ:

- **Liveness:** the process is running enough to serve basic workload.
- **Readiness:** the instance has dependencies/configuration and can serve requests safely.

### Why They Matter

Only a healthy target should receive traffic. Traffic distribution should account for capacity, not just current response count.

### Without Them

Load balancers route to broken tasks, deployments expose uninitialized instances, and healthy capacity remains underused.

## 2. Real-World Analogy

A taxi dispatcher keeps a list of drivers who are logged in and insured, not merely sitting by the roadside. Drivers with longer passenger queues receive fewer new assignments.

## 3. Technical Architecture

```text
Load balancer
 -> health checks every interval
 -> active healthy target pool
 -> distribution algorithm

Instance A: healthy, high capacity
Instance B: healthy, medium capacity
Instance C: unhealthy or draining
```

Common distribution algorithms include round robin, weighted round robin, least connections, random, hash/routing affinity, and latency-aware routing.

## 4. Practical Use Cases

- ALB sends HTTP traffic only to healthy application targets.
- NLB distributes connection-based traffic with flow hashing.
- Blue/green target groups use weighted traffic or instant switch.
- Stateful WebSocket workloads route and drain connections carefully.

## 5. DevOps and AWS Implementation

ALB target health states include initial, healthy, unhealthy, draining, unused, and other target registration states. Configure `/health/live` and `/health/ready`, protocol/port/path, interval, timeout, threshold, success codes, and deregistration delay.

Readiness should verify configuration and essential dependency access. For optional dependencies, do not mark every API instance unavailable if it can offer a bounded degraded response.

Use CloudWatch/ALB metrics: healthy/unhealthy host count, target response time, HTTP error codes, connection errors, and active connections. Deployment health checks should require readiness before increasing traffic.

## 6. Practical Production Scenario

An ECS deployment starts new tasks. The readiness endpoint verifies database and Redis connectivity and application configuration. ALB registers the task only after ECS health becomes ready. During shutdown, ECS deregisters and drains connections. Auto Scaling replaces a task when it stays unhealthy.

If Redis becomes unavailable but product reads are fallbackable, readiness can remain true while publishing a degraded-cache status. If database becomes unavailable, readiness should become false because writes cannot be served safely.

## 7. Hands-On Lab

Nginx health configuration concept:

```nginx
upstream app {
  server app1:8080 max_fails=3 fail_timeout=30s;
  server app2:8080 backup;
}

server {
  listen 80;
  location /health { return 200 "healthy\n"; }
  location / { proxy_pass http://app; }
}
```

Improve with separate readiness behavior that depends on actual service state. Stop one container, inspect traffic, then restart and observe it rejoin the pool.

## 8. Advantages, Disadvantages and Best Practices

- Use multiple Availability Zones.
- Separate liveness and readiness.
- Configure graceful shutdown/draining.
- Use dependency-aware readiness only for critical requirements.
- Avoid health probes that generate expensive database load.

Do not mark instance healthy when its startup migrations/configuration are incomplete or when 100% failures are hidden by a generic endpoint.

## 9. Interview Preparation

**Q1: Liveness vs readiness?** Liveness indicates process survival; readiness indicates safe traffic acceptance.

**Q2: Why separate them?** Restarting a process with an unavailable dependency is unnecessary; an unhealthy startup process should not receive traffic.

**Q3: Least connections?** Routes to a target with fewer active connections and is common for persistent traffic.

**Q4: Round robin?** Sequential distribution without considering current load.

**Q5: Sticky routing?** Routes one client to a consistent target, useful for certain stateful protocols but risky if state is not externalized.

**Scenario:** Healthy target percentage remains low. Inspect probe path, security groups, timeout interval, success codes, startup time, and dependency health.

**Scenario:** One target receives more traffic. Inspect algorithm, connection reuse, weights, unhealthy targets, and persistent HTTP connections.

**60-second answer:** “Health checks maintain a pool of targets capable of serving traffic. Distribution algorithms then route requests across the pool, while drain and startup policies prevent rollout traffic loss.”

## 10. Revision Notes

- Liveness: restart decision.
- Readiness: traffic decision.
- Separate optional vs critical dependencies.
- Use drain/deregistration delay.
- Monitor healthy host count and error rates.

**Challenge:** Design health endpoints for a payment API and decide whether a payment provider outage makes the API unready, degraded, or unaffected.
