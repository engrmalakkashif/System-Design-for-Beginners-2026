# 18. Database Indexes

## 1. Concept and Fundamentals

A database index is an auxiliary data structure that maps key values to row locations, enabling lookup, filtering, sorting, and range operations without scanning all rows.

Indexes speed reads but consume storage and add write overhead because inserts/updates/deletes maintain index pages. “Index everything” is a mistake.

## 2. Real-World Analogy

A book index maps topics to page numbers. It avoids reading every page, but it adds extra pages and requires updating whenever content changes.

## 3. Technical Architecture

```text
Table -> data pages
Index -> ordered key -> row pointer
Query planner -> choose table/index scan
```

Common index types:

- B-tree: equality/range/order.
- Composite: multiple columns in order.
- Unique: enforce uniqueness and lookup.
- Hash: equality in supported engines.
- Partial/filtered: rows matching a predicate.
- Full-text/search: text content.
- Covering: includes columns needed by a query.

## 4. Practical Use Cases

- Account lookup by email and status.
- Orders by customer and creation time.
- Inventory reservation by SKU and warehouse.
- Audit records by actor and timestamp range.

## 5. DevOps and AWS Implementation

Use RDS Performance Insights, CloudWatch database metrics, engine slow-query logs, and `EXPLAIN`/`EXPLAIN ANALYZE`. On PostgreSQL use `pg_stat_statements`; on MySQL use slow query log/Performance Schema.

Composite index column order must follow equality-first/range/order patterns. A single index may serve many queries but be too wide, too unique, or cause write amplification.

Create large indexes with engine-supported safe methods and test lock behavior, storage, backfill duration, and rollback.

## 6. Practical Production Scenario

An appointment API runs:

```sql
SELECT * FROM appointments
WHERE patient_id = $1
  AND status = 'confirmed'
ORDER BY appointment_at DESC
LIMIT 20;
```

The likely index is `(patient_id, status, appointment_at DESC)`. If status has low selectivity and appointment_at is the primary filter, use the actual query distribution and plan rather than assumptions.

A missing index creates sequential scans. An unused wide index adds insert/update cost. Before adding, capture frequency, read/write ratio, table size, and data distribution.

## 7. Hands-On Lab

```sql
CREATE TABLE events (
  id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  tenant_id bigint NOT NULL,
  status text NOT NULL,
  created_at timestamptz NOT NULL
);

INSERT INTO events(tenant_id,status,created_at)
SELECT (random()*1000)::int, 'open', now()-random()*interval '365 days'
FROM generate_series(1, 500000);

EXPLAIN ANALYZE
SELECT * FROM events
WHERE tenant_id = 42 AND status = 'open'
ORDER BY created_at DESC LIMIT 20;

CREATE INDEX idx_events_tenant_status_created
ON events(tenant_id, status, created_at DESC);

EXPLAIN ANALYZE
SELECT * FROM events
WHERE tenant_id = 42 AND status = 'open'
ORDER BY created_at DESC LIMIT 20;
```

Compare planning/scanning/execution time and rows returned/scanned.

## 8. Advantages, Disadvantages and Best Practices

- Speed selective reads/ranges/sorts.
- Enforce unique constraints.
- Reduce network payload by covering queries when appropriate.
- Add write/storage and maintenance cost.
- Keep statistics current.
- Remove genuinely unused indexes.

Avoid adding indexes solely because a query is slow: check data volume, query shape, network, locks, joins, and result count first.

## 9. Interview Preparation

**Q1: What is an index?** Auxiliary lookup structure pointing to table rows.

**Q2: Why do indexes improve reads?** They reduce scanned keys/pages and provide ordering.

**Q3: Why can indexes hurt writes?** Every modifying operation updates every affected index.

**Q4: What is composite index order?** Left-prefix and query predicate/order determine utility.

**Q5: What is covering index?** An index contains every value needed by a query, avoiding table access.

**Scenario:** Query uses an index but remains slow. Inspect result count, network payload, lock waits, disk I/O, join cardinality, and coverability.

**Scenario:** Add index during deployment blocks writes. Use engine-safe concurrent methods, low-traffic windows, storage monitoring, and rollback plans.

**60-second answer:** “Indexes make reads faster through ordered lookup structures, but consume storage and slow writes. Index design must follow actual predicates, joins, sort/order patterns, cardinality, and query plans.”

## 10. Revision Notes

- B-tree default for equality/range/order.
- Composite index order matters.
- Unique index enforces uniqueness.
- Use `EXPLAIN`/Performance Insights before adding.
- Remove unused/wide indexes.

**Challenge:** Design indexes for 5 different appointment queries and explain why two queries cannot share a single effective composite index.
