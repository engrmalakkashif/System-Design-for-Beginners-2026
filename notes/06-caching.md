# 6. Caching

## 1. Concept and Fundamentals

A cache stores a copy of data near the place it is needed, trading memory and freshness for latency and backend capacity. Cache hits avoid recomputation or database access; misses fall through to the authoritative source.

### Fundamental Properties

- **Hit ratio:** percentage of cache reads that find data.
- **TTL:** maximum acceptable age.
- **Eviction:** removal due to capacity or policy.
- **Invalidation:** ensuring stale values stop being served.
- **Stampede:** many cache misses overload the source simultaneously.

### Why Use It

Caching reduces latency, repeated expensive work, database load, and cross-region dependency traffic.

### What Happens Without It

Repeated reads reach databases and remote services, increasing cost and tail latency. Heavy reads can cause database overload.

## 2. Real-World Analogy

A desk drawer holds frequently used tools. The toolbox is the database. Tools taken from the drawer are fast, but the drawer copy must be replaced when the original tool changes.

## 3. Technical Architecture

```text
Request -> App Cache -> Redis/CDN -> Authoritative database
             |              |
             +-- hit: return response
             +-- miss: load then cache
```

Common patterns: cache-aside, read-through, write-through, write-back, and negative caching.

## 4. Practical Use Cases

- CDN caches images/video metadata.
- API caches product catalog data with short TTL.
- Redis caches session lookup or feature flags.
- DNS caches domain resolution.

## 5. DevOps and AWS Implementation

CloudFront caches static and selected dynamic responses at the edge. ElastiCache Redis caches application data. For an EC2-backed API, cache the warmest read values and isolate cache failures from essential business paths.

Cache deployment should include memory alarms, evictions, hit ratio, connection counts, network throughput, TTL policy, and Redis endpoint failover behavior. Never treat cache values as authoritative for inventory, balances, or security decisions.

## 6. Practical Production Scenario

A React healthcare dashboard reads patient-care reference data repeatedly. The API uses cache-aside:

1. Check Redis.
2. On miss, query RDS.
3. Write the result with TTL and fallback value when appropriate.
4. If Redis fails, proceed directly to RDS with reduced request limits.
5. When administrators update data, publish an invalidation event.

Avoid caching personal data without the access model, encryption, retention, and logging required by policy.

## 7. Hands-On Lab

Run Redis with Docker:

```bash
docker run -d --name redis-cache -p 6379:6379 redis:7-alpine

docker exec -it redis-cache redis-cli SET product:42 '{"name":"Laptop"}' EX 60
docker exec -it redis-cache redis-cli GET product:42
```

Observe TTL:

```bash
docker exec -it redis-cache redis-cli TTL product:42
```

Cache-aside pseudocode:

```text
value = cache.get(key)
if missing:
  value = database.query(key)
  cache.set(key, value, ttl)
return value
```

## 8. Advantages, Disadvantages and Best Practices

- Improves latency and reduces backend load.
- Use explicit TTLs and bounded sizes.
- Add single-flight or request coalescing to prevent stampedes.
- Handle cache failure gracefully with fallbacks or load shedding.
- Use versioned keys for deployment/schema changes.

Avoid caching sensitive data unless required controls exist; cache invalidation can become a correctness bug.

## 9. Interview Preparation

**Q1: Cache-aside?** Application checks cache, loads from database on miss, and stores the result.

**Q2: Write-through?** Write cache and backing store in the same operation path.

**Q3: Cache stampede?** Concurrent misses overwhelm source storage.

**Q4: How stale can cached data be?** Based on domain and TTL/SLO requirements.

**Q5: How do caches improve availability?** They reduce dependency traffic, though stale fallback can hide freshness issues.

**Scenario:** Redis eviction rate increases. Inspect key size, TTL distribution, memory policy, hit ratio, and hot objects.

**Scenario:** Users receive old prices after updates. Add versioned keys, event invalidation, or shorter TTLs and correct cache-update ordering.

**60-second answer:** “Caching duplicates frequently accessed data closer to computation to improve latency and reduce backend load, with careful TTL, invalidation, eviction, failure, and security policies.”

## 10. Revision Notes

- Cache != database.
- Cache hit/miss; TTL; eviction; invalidation.
- Cache-aside is common but needs stampede protection.
- Failure should not erase correctness or security requirements.

**Challenge:** Implement cache-aside for an API; then simulate cache failure, cache outage, and database update freshness.
