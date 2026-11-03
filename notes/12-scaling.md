# 12. Vertical vs Horizontal Scaling

## 1. Concept and Fundamentals

Vertical scaling increases capacity of one instance: larger CPU, RAM, disk, or network. Horizontal scaling adds more instances and distributes work across them.

### Why They Exist

Traffic changes and failures require capacity changes. Vertical scaling is simple; horizontal scaling reduces per-instance dependency and supports capacity growth beyond a single machine.

### Without Scale Strategy

One server reaches a limit, becomes a SPOF, and growth requires risky downtime.

## 2. Real-World Analogy

Vertical scaling is buying a bigger truck. Horizontal scaling is buying a fleet of smaller trucks. A bigger truck has a limit; a fleet needs dispatch/routing.

## 3. Technical Architecture

```text
Vertical:
one instance
-> larger CPU/RAM
-> same endpoint

Horizontal:
ALB -> instance A
     -> instance B
     -> instance C
Auto Scaling adjusts count
```

Horizontally scalable components should be stateless or externalize state. Persistent state may scale through replicas, partitioning, or managed services.

## 4. Practical Use Cases

- Scale an API horizontally behind an ALB.
- Scale PostgreSQL/RDS vertically before introducing read replicas or sharding.
- Scale worker queues horizontally based on backlog.
- Scale Redis vertically first, then partition/cluster as needed.

## 5. DevOps and AWS Implementation

EC2 vertical scale requires changing instance type, potentially restart. RDS vertical scale may have limited downtime depending on engine and Multi-AZ behavior.

Horizontal AWS implementations:

- EC2 Auto Scaling Group across multiple AZs.
- ECS desired task count and CPU/memory target scaling.
- Lambda concurrency scaling managed by AWS.
- SQS queue depth scaling for workers.

Use warm-up periods, cooldowns, target utilization, predictive scaling when patterns repeat, and instance lifecycle events. Account for regional quotas.

## 6. Practical Production Scenario

An API reaches 70% CPU and p95 latency increases. First, load-test and profile. If compute is under load, add an Auto Scaling target policy from 70% average CPU. Scale to two AZs. If the database reaches 90% CPU due to N+1 queries, adding API instances is not the fix; optimize queries and index first.

For worker backlog, scale based on approximate queue age or number of visible messages.

## 7. Hands-On Lab

Use Docker to demonstrate horizontal scaling locally:

```bash
docker build -t demo-api ./api
docker run -d --name api1 -p 8081:8080 demo-api
docker run -d --name api2 -p 8082:8080 demo-api
```

Configure a local Nginx upstream, generate load, and stop one instance. Observe the remaining instance serving healthy traffic.

Docker resource limits demonstrate vertical capacity:

```bash
docker run -m 256m --cpus="0.50" demo-api
```

## 8. Advantages, Disadvantages and Best Practices

### Vertical

- Simple and no distributed routing.
- Bounded by maximum instance size.
- Single-node failure remains possible.

### Horizontal

- More fault isolation and growth capacity.
- Requires load balancing, statelessness, external state, and observability.
- More operational complexity.

Scale only measured bottlenecks; define minimum/maximum fleet size and test scale-down behavior.

## 9. Interview Preparation

**Q1: Vertical scaling?** Increase one machine’s resources.

**Q2: Horizontal scaling?** Add more instances.

**Q3: Which is simpler?** Vertical scaling, because no distributed routing is needed.

**Q4: What limits vertical scaling?** Hardware/cloud instance maximums and downtime.

**Q5: What makes horizontal scaling possible?** Stateless instances, stable load distribution, external state, and automated health checks.

**Scenario:** An API scales out but database saturation increases. Tune queries, cache, pool connections, and consider read replicas before more app replicas.

**Scenario:** Autoscaling rapidly adds and removes instances. Inspect metric lag, cooldowns, warm-up, sustained threshold, and scale-to-zero constraints.

**60-second answer:** “Vertical scaling makes one machine larger; horizontal scaling adds machines. Horizontal scaling generally increases availability and capacity, provided shared state and load-aware failover are designed correctly.”

## 10. Revision Notes

- Vertical: bigger instance.
- Horizontal: more instances.
- Autoscaling needs measured signals and warm-up.
- Workers can scale on queue depth.
- Databases may require vertical, replicas, partitioning, or sharding.

**Challenge:** Design scaling triggers for API workers and background jobs; explain the signals, targets, limits, and failure behavior.
