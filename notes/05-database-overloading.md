# 5. Database Overloading

## 1. Concept and Fundamentals

Database overloading means the database cannot serve the required read/write work, connections, or storage operations within acceptable latency. It may show as CPU saturation, connection exhaustion, lock contention, slow queries, disk I/O pressure, or replication lag.

### Why It Happens

- Direct queries from every request.
- N+1 query patterns.
- Missing or inefficient indexes.
- Unbounded connection pools.
- Large scans or reports during peak traffic.
- Writes from queues, batch jobs, and users compete.

### What Happens Without Protection

The database becomes the shared bottleneck, latency multiplies through retries, connection queues grow, application pods fail, and recovery can take longer than the outage.

## 2. Real-World Analogy

A library has one checkout desk. Ten thousand customers asking for entire aisle scans overwhelms it, even though the building has many shelves. Separating searches, adding catalogs/indexes, appointment systems, and self-service kiosks reduces desk work.

## 3. Technical Architecture

```text
API
 -> cache hot reads
 -> rate limits
 -> read replica for reporting
 -> primary database for writes
 -> queue for expensive asynchronous work

Database telemetry: CPU, connections, locks, I/O, slow queries
```

## 4. Practical Use Cases

- E-commerce product details are cached; checkout writes remain consistent.
- Banking reporting queries are sent to replicas, respecting read-after-write needs.
- Healthcare analytics jobs are isolated from clinical transactions.
- SaaS tenant-heavy queries use indexes and tenant-aware limits.

## 5. DevOps and AWS Implementation

Use RDS Performance Insights to inspect database load. Amazon CloudWatch provides CPU, connections, storage, IOPS, and replica lag. Use RDS Proxy for connection pooling when serverless or bursty applications create many connections.

Use ElastiCache for cacheable reads, read replicas for suitable read traffic, SQS/EventBridge for asynchronous work, and CloudWatch alarms for capacity thresholds. Keep database maintenance and migrations outside peak hours.

Use PgBouncer/MySQL ProxySQL when architecture-specific connection pooling is needed.

## 6. Practical Production Scenario

A product API performs one query for the product and N additional queries for reviews. During a sale, database connections reach 100% and latency rises.

Resolution:

1. Batch review queries to eliminate N+1 behavior.
2. Add indexes for actual filter/sort patterns.
3. Cache product metadata with a bounded TTL.
4. Move analytics jobs to replicas or a separate warehouse.
5. Limit pool size per application replica.
6. Add query timeouts and retry budgets.

Never solve capacity solely with retries; retries amplify load.

## 7. Hands-on Lab

Create PostgreSQL locally:

```bash
docker run -d --name pg -e POSTGRES_PASSWORD=demo -p 5432:5432 postgres:16
psql 'postgres://postgres:demo@localhost:5432/postgres'
```

Create a table and query execution plan:

```sql
CREATE TABLE orders(id serial PRIMARY KEY, customer_id integer, status text);
INSERT INTO orders(customer_id, status)
SELECT (random()*1000)::int, 'new'
FROM generate_series(1, 100000);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 42;

CREATE INDEX idx_orders_customer ON orders(customer_id);
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 42;
```

A query may initially perform a sequential scan and much better after indexing.

## 8. Advantages, Disadvantages and Best Practices

- Define connection limits and query timeouts.
- Monitor slow-query logs and database wait events.
- Batch writes and reads.
- Cache carefully; do not cache regulated or rapidly changing data blindly.
- Separate transaction and analytical workloads.
- Use a retry policy with jitter and only retry transient operations.

Avoid manually scaling database every time application inefficiency is present.

## 9. Interview Preparation

**Q1: Signs of database overload?** High CPU, connections, I/O wait, locks, slow queries, queueing, replication lag.

**Q2: Why connection pooling?** Reuses connections, limits concurrent database sessions, and absorbs bursty client traffic.

**Q3: N+1 problem?** One query plus repeated queries for each returned item instead of one or two batched queries.

**Q4: Cache or read replica first?** Cache for repeatable reads; replica when database-compatible read scaling is needed and staleness/read-after-write are handled.

**Q5: How do migrations affect production?** Large locking migrations can block traffic, so test lock behavior and apply online/expand-contract changes.

**Scenario:** Writes overload a primary. Profile writes, batch where possible, move bulk jobs and analytics, add a durable queue, partition data, or shard only after measured need.

**Scenario:** Database is healthy but application sees timeouts. Inspect connection pool exhaustion, DNS, network, retries, and downstream lock waits.

**60-second answer:** “Database overload should be diagnosed by workload and resource signals, then reduced at the application and data layers through indexes, batching, pooling, caching, replicas, queues, and isolation.”

## 10. Revision Notes

- Diagnose query, connection, lock, CPU, I/O, and network pressure.
- N+1 and full scans are common application causes.
- Pooling, caching, replicas, and asynchronous work reduce load differently.
- Retries need budgets and jitter.

**Challenge:** Run a baseline SQL workload before and after adding an appropriate index; measure latency and execution plans.
