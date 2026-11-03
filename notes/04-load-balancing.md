# 4. Load Balancing

## 1. Concept and Fundamentals

A load balancer distributes requests across multiple backend targets. It improves capacity, availability, health-based routing, TLS termination, and traffic control.

It must not send traffic to an unhealthy target. It can also enforce connection limits, authenticate edges, and route by host/path.

### Why It Is Needed

One server has a maximum throughput and is a single point of failure. A load balancer exposes a stable endpoint while replacing or adding backend instances.

### Without a Load Balancer

Clients may connect directly to one server, deployments require downtime, failures remain manual, and horizontal scaling is difficult.

## 2. Real-World Analogy

A reception desk directs visitors among several available service counters. It keeps a list of open counters, avoids closed counters, and routes specialized requests to specialized desks.

## 3. Technical Architecture

```text
Client
 -> DNS
 -> ALB / NLB
 -> health-aware target group
       -> instance A
       -> instance B
       -> instance C
```

A load balancer may operate at transport (L4) or application (L7) layers. L7 understands HTTP, headers, cookies, and paths. L4 understands IP/TCP/UDP and often offers high throughput with less request inspection.

## 4. Practical Use Cases

- Web tier distributes UI/API traffic across Auto Scaling instances.
- Microservices expose separate listener rules/path routes.
- Gaming or finance workloads use NLB for low-latency TCP traffic.
- Blue/green deployments shift traffic between target groups.

## 5. DevOps and AWS Implementation

Use an internet-facing or internal ALB. Place instances in private subnets, attach a target group, configure listener ports, and assign health checks. Security groups permit traffic only from the ALB on the application port, not directly from the internet.

Choose NLB when TCP/UDP behavior, static IP, or low-latency transport routing is the priority. Choose ALB for HTTP host/path routing, headers, redirects, WebSockets, or Lambda targets.

Deployment options:

- Rolling deployments replace targets gradually.
- Blue/green moves traffic to a new target group.
- Canary sends a small percentage first.

## 6. Practical Production Scenario

Three EC2 instances run a React API backend across Availability Zones. The ALB performs HTTPS TLS termination and forwards `/api/*` to the API target group. Health checks call `/health/ready` every 15 seconds.

An instance hangs. ALB stops routing after consecutive failures. Auto Scaling launches a replacement. The ALB begins routing only after readiness checks pass. The deployment drain policy allows existing connections to finish.

## 7. Hands-On Lab

Run two web containers with Docker and Nginx as a local L7 load balancer:

```bash
docker run -d --name app1 --network appnet nginx:alpine
printf 'server { listen 8080; location / { proxy_pass http://app1:80; } }' \
  > nginx.conf
```

Production-style test should be done with real multiple containers and stable identifiers. Nginx configuration:

```nginx
upstream app_servers {
  server app1:80;
  server app2:80;
}
server {
  listen 8080;
  location / {
    proxy_pass http://app_servers;
  }
}
```

Validate distribution through response headers, container instance IDs, or logs.

## 8. Advantages, Disadvantages and Best Practices

- Distributes capacity and isolates unhealthy targets.
- Supports secure TLS and deployment strategies.
- Adds latency, cost, and configuration dependencies.
- Avoid sticky sessions unless explicitly required.
- Configure access logs, connection limits, timeouts, health checks, and TLS policy.

Do not use a health check that returns 200 while dependencies are unavailable unless liveness and readiness are intentionally separate.

## 9. Interview Preparation

**Q1: What does a load balancer solve?** Capacity distribution, availability, and replacement-aware routing.

**Q2: L4 vs L7?** L4 routes network connections; L7 routes HTTP application traffic.

**Q3: What is a health check?** A repeatable probe showing whether a target should receive traffic.

**Q4: ALB vs NLB?** ALB for HTTP L7 features; NLB for high-performance TCP/UDP and static IP needs.

**Q5: What is sticky session routing?** Sending a client to the same backend; usually state is better stored externally.

**Scenario:** One target receives all traffic. Inspect listener rules, target registration, connection reuse, autoscaling group membership, and load algorithm behavior.

**Scenario:** Deployment returns 5xx during rollout. Verify readiness, graceful draining, connection timeouts, and database migration compatibility.

**60-second answer:** “A load balancer distributes requests to healthy targets, enables scaling and replacement, and provides a stable secure entry point with health-aware traffic control.”

## 10. Revision Notes

- Stable frontend; changing backend targets.
- L4 = connection; L7 = HTTP semantics.
- Readiness should test the service, liveness only whether the process is alive.
- Health checks and scaling policies must work together.

**Challenge:** Run four application containers behind Nginx, stop one, and confirm traffic continues on the remaining containers.
