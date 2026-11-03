# 19. Caching with Redis

## 1. Concept and Fundamentals

Redis is an in-memory data structure store used for caching, sessions, counters, rate limits, distributed locks, pub/sub, streams, queues, and feature state. It is fast because data is held in memory, but memory capacity and persistence configuration determine durability.

Redis can be cache-only or contain durable application state. Teams must identify the authoritative copy before choosing Redis.

## 2. Real-World Analogy

Redis is a fast workbench holding frequently used tools and temporary counters. A filing warehouse is the durable database. Workbench data may disappear, be evicted, or become stale.

## 3. Technical Architecture

```text
App replicas
 -> Redis ElastiCache/managed cluster
 -> cache policy/keyspace TTL
 -> database fallback/source of truth
```

Redis structures include strings, hashes, lists, sets, sorted sets, streams, and bitmap/HyperLogLog types. Use structures based on operations, not only storing serialized JSON blobs.

## 4. Practical Use Cases

- Cache API/product/catalog responses.
- Token/session lookup with TTL.
- Distributed rate limiter.
- Leaderboard with sorted sets.
- Job queue/stream for lightweight asynchronous work.

## 5. DevOps and AWS Implementation

Amazon ElastiCache supports Redis OSS clusters, replicas, Multi-AZ, encryption, parameter control, monitoring, and automatic failover depending on configuration. Place it in private subnets and restrict security groups to application clients.

Set memory limits and eviction policy deliberately. Monitor CPU, engine CPU, memory usage, swaps, evictions, cache misses, connection count, replication lag, network, and command latency. Use cluster mode only when data/throughput requires sharding and access patterns work around it.

A cache is not a database for critical state unless Redis durability and HA requirements are explicitly designed, tested, and acceptable.

## 6. Practical Production Scenario

A recommendation API caches `recommendations:user:<id>` with a 15-minute TTL and cache stampede protection. Redis outage does not break profile/authentication. The API applies load shedding and short timeout, then reads a fallback store at lower concurrency.

A per-user rate limiter uses a Redis atomic increment and expiration. Unsuccessful writes fail closed for security-sensitive endpoints and degrade to database/provider control only if policy permits.

## 7. Hands-On Lab

```bash
docker run -d --name redis-lab -p 6379:6379 redis:7-alpine

docker exec -it redis-lab redis-cli SET session:abc '{"user":42}' EX 300
docker exec -it redis-lab redis-cli GET session:abc

docker exec -it redis-lab redis-cli INCR rate:user42
docker exec -it redis-lab redis-cli EXPIRE rate:user42 60

# Sorted-set leaderboard
docker exec -it redis-lab redis-cli ZADD leaderboard 100 alice 80 bob
docker exec -it redis-lab redis-cli ZREVRANGE leaderboard 0 9 WITHSCORES
```

Validate TTL and eviction behavior, then clear it:

```bash
docker rm -f redis-lab
```

## 8. Advantages, Disadvantages and Best Practices

- Very low latency and rich data types.
- Supports atomic counters and sorted structures.
- Memory cost and eviction risk.
- Cluster sharding complicates cross-key operations.
- Set key names, TTL, size limits, serialization, and metrics.
- Protect against hot keys and large key values.

Avoid using Redis `KEYS *` in production. Use `SCAN`, bounded command timeouts, connection pools, and redaction for sensitive values.

## 9. Interview Preparation

**Q1: Redis cache vs database?** Redis is memory-focused and can be cache or operational state depending on design.

**Q2: Redis persistence?** Persistence helps recovery but differs from database transactional semantics.

**Q3: Cache stampede?** Coalescing/lock/jitter/early refresh reduces simultaneous misses.

**Q4: Redis eviction policy?** Defines which keys are removed under memory pressure.

**Q5: Redis distributed lock caveats?** Lock semantics need fencing/token validation for correctness across failures.

**Scenario:** Redis node fails and sessions expire. Use HA/failover, short tokens, external session refresh, or graceful reauthentication depending on security requirements.

**Scenario:** One Redis key is very large. Split data, compress only if useful, bounded reads, and monitor command latency/memory.

**60-second answer:** “Redis provides low-latency in-memory structures for cache, sessions, counters, queues, and coordination. It must have explicit TTL, eviction, failover, security, and authority boundaries.”

## 10. Revision Notes

- Redis is memory-first and fast.
- Choose data structures intentionally.
- Use TTL and bounded keys.
- Cache miss/failure/stampede behavior matters.
- Managed Multi-AZ does not make Redis storage authoritative automatically.

**Challenge:** Implement rate limiting, cache-aside, and leaderboard features in one Redis lab and explain each key/TTL/atomicity choice.
