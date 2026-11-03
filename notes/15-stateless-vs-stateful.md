# 15. Stateless vs Stateful Architecture

## 1. Concept and Fundamentals

A stateless service needs no local memory of prior client interactions to handle a request. Each request includes or retrieves everything required. A stateful service stores session/context/connection state locally, making one instance responsible or requiring synchronization.

### Why This Matters

Stateless replicas are easy to replace, add, and distribute. Stateful components require placement, replication, persistence, failover, and backup planning.

### Without External State Design

Sticky sessions hide architecture problems but fail after instance replacement, restart, or deployment.

## 2. Real-World Analogy

A stateless cashier uses a customer’s receipt and central account database. A stateful cashier keeps a customer’s open tab locally; moving customers to another cashier requires transferring the tab.

## 3. Technical Architecture

```text
Stateless API -> ALB -> any instance -> Redis/RDS/S3

Stateful service -> owned shard/partition -> local memory/disk
                 -> replication/snapshot -> failover process
```

Session or workflow state belongs in Redis, a database, signed tokens, or another explicitly managed store. JWT/OIDC session design must consider revocation, expiration, key rotation, and token size.

## 4. Practical Use Cases

- Stateless REST API instances scale behind ALB.
- Stateful WebSocket server tracks active connections.
- Stateful streaming/auction worker owns transient in-memory sessions but checkpoints data externally.
- Stateful databases use replication and consistency semantics.

## 5. DevOps and AWS Implementation

Use EC2/ECS/Fargate tasks for stateless API replicas. Store sessions in Redis, DynamoDB, or database-backed session systems based on requirements. Prefer DynamoDB signed session claims or OIDC provider sessions for suitable flows.

Avoid ALB sticky sessions unless protocol/latency requirements justify local affinity. For WebSockets, account for long-lived connections during deployment and autoscaling. StatefulSets and PersistentVolumes can manage local state in Kubernetes but remain operationally complex.

RDS, ElastiCache, Kafka, and DynamoDB are stateful managed services that Amazon operates, while application tasks remain mostly stateless.

## 6. Practical Production Scenario

A React user logs into a .NET API. The application does not store sessions in process memory. Authentication claims come from a signed OIDC token, and server-side revocation/refresh state uses Redis or DynamoDB with TTL. Any API instance may serve the request.

When one instance terminates, sessions continue. Sticky sessions do not act as a fallback. Security state includes short-lived access tokens, secure refresh storage, key rotation, and logout/revocation behavior.

## 7. Hands-On Lab

Run an Nginx or application service in two containers. Have it return the container hostname. Test request distribution. Next, add Redis for shared state, store a counter there, and stop one app instance. Redis-backed state must survive replacement.

```bash
docker exec redis-cache redis-cli INCR visits
```

Observe a shared counter as evidence that application state no longer lives on one replica.

## 8. Advantages, Disadvantages and Best Practices

### Stateless Benefits

- Easy scaling and replacement.
- Better rolling deploy behavior.
- No affinity requirement.

### Stateful Benefits

- Lower network round trips for some workflows.
- Protocols or domains with genuinely persistent context.

### Risks

- Stateful data loss/recovery.
- Hot shards and uneven affinity.
- Difficulty deploying/draining connections.

Externalize durable state and design transient state with expiration and recovery.

## 9. Interview Preparation

**Q1: Stateless architecture?** A request needs no instance-local memory from past requests.

**Q2: Stateful architecture?** Instance behavior depends on locally stored durable or session state.

**Q3: Why is stateless easier?** Any instance can handle a request and instances can be replaced without migrating state.

**Q4: How is login state managed?** Signed tokens, external sessions, cookies, refresh records, or identity provider sessions.

**Q5: What is sticky session?** Load balancer affinity sending a client to the same instance; not a substitute for shared state.

**Scenario:** Logged-out users remain authenticated. Verify logout token/session invalidation, refresh token rotation, cache revocation, and cookie scope.

**Scenario:** WebSocket traffic drops during deployment. Implement connection draining, checkpointing, reconnect logic, and state migration before large reductions.

**60-second answer:** “Stateless services depend on tokens, requests, or external stores, making replacement easy. Stateful services keep context locally and require replication, placement, and failover because instance failure can affect in-progress work.”

## 10. Revision Notes

- Stateless = replaceable instances, external/shared state.
- Stateful = local context or data, specialized lifecycle.
- Sticky sessions hide but do not solve state loss.
- Externalize tokens/session data with controlled lifetime/security.

**Challenge:** Design session handling for a video-meeting app that has WebSocket connections, authentication, and temporary meeting state.
