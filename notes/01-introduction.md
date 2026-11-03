# 1. Introduction to System Design

## 1. Concept and Fundamentals

System design is the process of defining components, interfaces, data flow, scaling strategies, reliability controls, security, and operational behavior for a software system. It answers not only “what will the system do?” but also “will it continue working when demand, failures, data volume, or dependencies change?”

**Core goals:** scalability, availability, reliability, performance, maintainability, security, cost efficiency, and observability.

### Why System Design Is Needed

A working application is not automatically a production-ready system. System design prevents problems such as a single server becoming a bottleneck, one database outage stopping the business, or hidden coupling making changes risky.

### What Happens Without It?

Teams commonly discover outages, slow deployments, data loss risk, unbounded infrastructure cost, difficult debugging, and architecture decisions that require large rewrites.

### Key Trade-offs

Every choice balances competing goals. For example:

| Choice | Benefit | Cost or Risk |
|---|---|---|
| More availability | Users experience fewer outages | More infrastructure and complexity |
| Strong consistency | Reads see latest committed data | Possible higher latency or reduced availability |
| Microservices | Independent deployment | Operational and distributed-system complexity |
| Caching | Faster reads | Staleness and invalidation complexity |

## 2. Real-World Analogy

A restaurant is a software system. The dining area is the web/API tier, kitchen staff are application workers, ingredients and recipes are data, reservations are queues, and backups are the second kitchen location. Moving to a larger kitchen is vertical scaling; opening more kitchen stations is horizontal scaling.

## 3. Technical Architecture

```text
Users
  -> DNS / Route 53
  -> CDN / CloudFront
  -> Load Balancer / ALB
  -> Application instances
  -> Cache (Redis)
  -> Database (RDS)
  -> Object storage / S3

             | metrics, logs, traces |
             v
CloudWatch / logging / tracing tools
```

### Basic Request Flow

1. DNS resolves the domain.
2. The CDN serves cached static content or forwards the request.
3. The load balancer routes the request to a healthy application instance.
4. The application validates input, authorizes the user, and applies business logic.
5. It reads cache/database storage and calls other services when needed.
6. The response returns to the client.
7. Logs, metrics, and traces record the operation.

## 4. Practical Use Cases

- **E-commerce:** Handle flash sales, payments, inventory, and product catalog traffic without overselling or losing orders.
- **Banking:** Provide strong security, auditability, consistency, and disaster recovery.
- **Healthcare:** Protect patient data while supporting high availability and careful access control.
- **Streaming:** Deliver large objects through a CDN and isolate playback metadata from media storage.

## 5. DevOps and AWS Implementation

- **Route 53:** DNS and health-aware routing.
- **CloudFront:** edge cache and TLS termination.
- **ALB:** HTTP routing, health checks, and application-level load balancing.
- **EC2 + Auto Scaling Group:** compute fleet that scales and replaces unhealthy instances.
- **ECS/EKS:** managed container orchestration choices.
- **RDS:** managed relational database with backups and optional Multi-AZ deployment.
- **ElastiCache:** managed Redis-compatible caching.
- **S3:** durable object storage.
- **CloudWatch:** metrics, logs, alarms, dashboards.
- **IAM/Security Groups/VPC:** identity, network boundaries, and least privilege.

Prefer managed services when they reduce undifferentiated operational work and meet security, latency, and control requirements. Prefer self-managed infrastructure only when the added control is worth its operational cost.

## 6. Practical Production Scenario

Imagine a React frontend and .NET API serving a healthcare portal with MySQL data. The static frontend is in S3 behind CloudFront. APIs run on EC2 behind an ALB in private subnets. The MySQL database uses RDS Multi-AZ, while Redis caches frequently read, non-sensitive lookup data. A CI/CD pipeline builds Docker images, runs tests, pushes them to a registry, and deploys through Jenkins or AWS-native tooling.

### Failure Handling

- An instance fails: ALB stops sending traffic after a failed health check; Auto Scaling replaces it.
- An Availability Zone fails: Multi-AZ database and multi-AZ application deployment maintain service.
- A dependency is slow: use timeouts, retries with backoff, circuit breakers, and rate limits.
- Deployments break: version images, automate rollback, and use readiness checks.

## 7. Hands-On Lab

Run a local two-tier application with Docker:

```bash
docker network create appnet

docker run -d --name redis --network appnet redis:7-alpine

docker run -d --name api \
  --network appnet -p 8080:8080 \
  -e REDIS_HOST=redis your-api-image:latest

curl http://localhost:8080/health
curl http://localhost:8080/api/items
```

### Validation

- API returns 200 from `/health`.
- Application metrics show requests and latency.
- Redis container is reachable from the API container.
- Stopping one API container does not represent a highly available setup until the service has multiple instances and a load balancer.

### Cleanup

```bash
docker rm -f api redis
docker network rm appnet
```

## 8. Advantages, Disadvantages and Best Practices

### Benefits

- Converts requirements into verifiable architecture decisions.
- Improves availability, operability, security, and cost control.
- Makes scaling and failure behavior visible before implementation.

### Common Mistakes

- Starting with technology instead of requirements.
- Adding every distributed-system component without an measured problem.
- Forgetting failure handling, deployment, monitoring, and security.
- Designing for “internet scale” when traffic is small.

### Security and Best Practices

Encrypt data in transit and at rest; use private subnets, security groups, IAM roles, secrets managers, least privilege, timeouts, and health-based deployments.

### When Not to Over-Engineer

A new internal tool with low traffic may appropriately start as a simple monolith with managed database and backups. Introduce complexity only when a requirement demonstrates the need.

## 9. Interview Preparation

**Q1: What is system design?**
It is the process of choosing components and policies for functionality, scale, reliability, security, data, and operations.

**Q2: What makes a system scalable?**
Its capacity can grow with demand through mechanisms such as stateless replicas, load balancing, partitioning, caching, and asynchronous processing.

**Q3: Availability vs reliability?**
Availability is readiness at a moment; reliability is consistently correct operation over time.

**Q4: Why do requirements come first?**
They determine consistency, latency, availability, storage, cost, and component choices.

**Q5: Why is cloud managed service useful?**
It removes infrastructure maintenance but can introduce cost and vendor constraints.

**Scenario:** Design a flash-sale service. Start with availability targets, read/write volume, inventory consistency, checkout flow, and peak capacity. Use caching carefully, reserve inventory atomically, and queue non-critical work.

**Scenario:** An application works locally but fails in AWS. Inspect DNS, security groups, listener rules, target health, route tables, IAM, logs, and database reachability.

**60-second answer:** “System design turns product requirements into a scalable, reliable architecture using clear component boundaries, data flow, failure controls, security, and operational feedback.”

## 10. Revision Notes

- System design balances requirements and trade-offs.
- Components: DNS, edge cache, load balancer, compute, cache, database, object storage, observability.
- Scale from measured bottlenecks, not assumptions.
- Design for partial failure and operational recovery.

**Key terms:** throughput, latency, availability, reliability, consistency, scalability, bottleneck, SLA, SLO, SLI.

**Practical challenge:** Draw an architecture for a university course-registration system and identify its five largest failure risks.
