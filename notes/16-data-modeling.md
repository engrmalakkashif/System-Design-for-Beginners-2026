# 16. Data Modeling and Access Patterns

## 1. Concept and Fundamentals

Data modeling defines entities, relationships, keys, constraints, lifecycle, ownership, and consistency boundaries. Access patterns define which queries, writes, ranges, ordering, and latency requirements the model must support.

### Why It Matters

A normalized relational model may fit transactions but be poor for a high-volume single-partition pattern. A denormalized NoSQL model may support reads but complicate updates.

### Without It

Teams add indexes after deployment, duplicate mutable facts, hit database hotspots, and discover that the API is querying a model optimized for different use cases.

## 2. Real-World Analogy

A filing cabinet can organize documents by cabinet/drawer/folder. It is efficient for known lookup patterns, but reopening a customer timeline by searching every folder is poor. A model should reflect how records are retrieved, not just how they relate conceptually.

## 3. Technical Architecture

Modeling process:

1. List entities and invariants.
2. List write operations and frequency.
3. List read queries, filters, sorting, and pagination.
4. Define consistency and access ownership.
5. Choose keys/relationships/partitions.
6. Define retention, deletion, backup, and audit.

Example relational model:

```text
User(id PK)
Account(id PK, user_id FK)
Order(id PK, account_id FK, status, created_at)
OrderItem(order_id FK, sku, quantity, unit_price)

Unique/constraints:
- Account.owner exists
- quantity > 0
- order status is a controlled transition
```

## 4. Practical Use Cases

- E-commerce: orders, payment status, inventory reservations, product snapshots.
- Banking: accounts, ledger entries, balance derivation, reversal/audit records.
- Healthcare: patient, encounter, authorization, clinical record, immutable audit.
- SaaS: tenant, user, role, subscription, usage event.

## 5. DevOps and AWS Implementation

Use RDS/PostgreSQL/MySQL for relationships and ACID transactions. Use DynamoDB for known key/access patterns and high scale. Use S3 for objects and Athena/warehouse tools for analytics. Use KMS/IAM/VPC controls for protected data.

For migrations, use versioned schema scripts, expand-contract changes, rollback checkpoints, and online index changes where supported. Never couple schema deployment to unsafe application assumptions.

Document partition keys, sort keys, GSI projections, TTL usage, data retention, and tenant isolation in architecture records.

## 6. Practical Production Scenario

An event-booking platform stores users, seats, reservations, payments, and audit events. The invariant “a seat cannot be reserved twice” is enforced by a unique `(event_id, seat_id)` constraint. Reservation state transitions are transactional. Payment callback retries use an idempotency key, and expired reservations are released by scheduled jobs.

Search reads may use an Elasticsearch/OpenSearch index, while the relational database remains authoritative.

## 7. Hands-On Lab

Create a PostgreSQL order schema and enforce invariants:

```sql
CREATE TABLE orders (
  id uuid PRIMARY KEY,
  status text NOT NULL CHECK (status IN ('pending','paid','cancelled')),
  account_id uuid NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
  order_id uuid REFERENCES orders(id),
  sku text NOT NULL,
  quantity integer NOT NULL CHECK (quantity > 0),
  PRIMARY KEY(order_id, sku)
);
```

Test invalid quantity and duplicate SKU. Then add a query and index based on the actual read pattern:

```sql
SELECT id, status, created_at
FROM orders
WHERE account_id = $1
ORDER BY created_at DESC LIMIT 20;
```

## 8. Advantages, Disadvantages and Best Practices

- Base design on measured access patterns.
- Use constraints for invariants; application checks are not sufficient under concurrency.
- Prefer immutable event/audit records for history.
- Keep sensitive and analytical data separate based on access needs.
- Use pagination based on stable keys/cursors.

Avoid designing normalized tables first and indexes later without query workload evidence. Also avoid giant JSON blobs as an escape hatch for every model.

## 9. Interview Preparation

**Q1: What is access-pattern design?** Designing keys, structure, indexes, and partitions for known reads/writes.

**Q2: Normalization?** Removing redundant data to preserve consistency.

**Q3: Denormalization?** Duplicating data to optimize reads while accepting update complexity.

**Q4: What is the source of truth?** The store that owns a fact and is authoritative during conflicts.

**Q5: Why use immutable events?** They preserve history and reduce rewrite of prior facts.

**Scenario:** A messaging app has billions of messages. Model partition by conversation/user and sort by time; archive old partitions and support pagination.

**Scenario:** A finance balance is derived incorrectly. Store ledger entries, derive balance using a transaction-safe process, and preserve audit/reversal events.

**60-second answer:** “Data modeling defines entities and invariants; access-pattern analysis defines how they are queried and changed. The resulting keys, indexes, partitions, and consistency boundaries make the database support real workloads.”

## 10. Revision Notes

- Entities + relationships + constraints.
- Writes and reads before implementation.
- Source of truth and duplication policy.
- Partition/tenant/key design is part of modeling.
- Test migrations and concurrency.

**Challenge:** Model a ride-sharing system with 10 read/write patterns and identify two relationships you would avoid querying under every ride request.
