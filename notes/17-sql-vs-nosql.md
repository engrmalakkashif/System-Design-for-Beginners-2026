# 17. SQL vs NoSQL Databases

## 1. Concept and Fundamentals

SQL databases typically use structured schemas, relational joins, transactions, and SQL query language. NoSQL covers document, key-value, wide-column, graph, and time-series models optimized around different access patterns and scale characteristics.

### Why Compare

The choice depends on relationships, transaction requirements, query flexibility, scale, consistency, operational maturity, and cost. “NoSQL is faster” is not a rule.

### Without a Choice Framework

Teams either use a relational database for every workload or choose a document store that cannot express required invariants, transactions, or queries.

## 2. Real-World Analogy

SQL is a library with strict catalog cards and cross-references. NoSQL is a warehouse with purpose-built bins: one for packages by tracking number, one for documents by category, and one for sensor timelines. Each is efficient for its intended retrieval pattern.

## 3. Technical Architecture

```text
SQL transaction workload
 -> primary + replicas
 -> tables, constraints, indexes
 -> joins/transactions

NoSQL access workload
 -> partition key + sort key/key
 -> known item/query patterns
 -> sparse fields, denormalization
 -> replica/failover controls
```

SQL and NoSQL are families, not a single binary choice. Managed systems also offer serverless throughput, cross-region replica, TTL, streams/events, and analytics integration.

## 4. Practical Use Cases

- SQL: accounting ledger, ordering, inventory, user/role relationships, complex reports.
- Key-value/document: profiles, sessions, product catalog, feature configurations.
- Wide-column: massive time-series or high-write workloads.
- Graph: fraud/identity relationship traversal.
- Search: full-text and faceted retrieval, usually backed by an authoritative store.

## 5. DevOps and AWS Implementation

AWS relational choices include RDS for MySQL/PostgreSQL/SQL Server and Aurora for managed scaling/replication. DynamoDB is a managed key-value/document service; ElastiCache is a cache, not a durable authority.

DynamoDB requires the partition key/access pattern, hot partition, item size, throughput, GSI, TTL, streams, consistency, and backup plan before implementation. RDS requires capacity, connection pooling, indexes, backups, multi-AZ, replicas, and query monitoring.

Choose managed search/warehouse systems for specialized queries rather than forcing an operational database to become an analytics engine.

## 6. Practical Production Scenario

A SaaS ordering platform uses PostgreSQL for accounts, orders, payments, and inventory uniqueness. DynamoDB stores high-volume per-tenant session notifications where item key design supports list-by-user queries. An analytics warehouse copies completed events for reporting.

The team documents which database owns each fact and does not duplicate payment truth across systems.

## 7. Hands-On Lab

Compare a PostgreSQL join with a DynamoDB/local Dynamo alternative. For SQL locally:

```sql
CREATE TABLE users (id int PRIMARY KEY, name text);
CREATE TABLE orders (id int PRIMARY KEY, user_id int REFERENCES users(id));

SELECT u.name, o.id
FROM users u JOIN orders o ON o.user_id = u.id
WHERE u.id = 1;
```

Then state equivalent DynamoDB data needs:

```text
PK=USER#1, SK=PROFILE
PK=USER#1, SK=ORDER#timestamp#orderId
```

The SQL model supports flexible joins. DynamoDB-style layout supports repeated queries only for planned partition/sort key access.

## 8. Advantages, Disadvantages and Best Practices

### SQL

- Flexible query and joins.
- Constraints and transactions.
- Vertical/replica capacity may require tuning and connection limits.

### NoSQL

- Purpose-built scale and predictable key access.
- Flexible documents or specialized models.
- Transactions/joins/query flexibility vary.
- Denormalized data creates update complexity.

Select based on workload and invariants, not market popularity.

## 9. Interview Preparation

**Q1: SQL vs NoSQL difference?** Schema/query model, relationships, transactions, and scaling patterns differ.

**Q2: What is BASE?** Generally available systems favor eventual consistency over strict ACID behavior.

**Q3: Is SQL inconsistent?** No; consistency depends on isolation/transaction/replication configuration.

**Q4: Can NoSQL transactions exist?** Yes; support varies by database and access pattern.

**Q5: Why add SQL and NoSQL together?** Specialized stores serve different access patterns, each with authoritative ownership boundaries.

**Scenario:** DynamoDB hot partition occurs. Identify skewed partition key, traffic concentration, and change key/query design or use offloading/queueing.

**Scenario:** Relational database joins become slow. Inspect indexes/query plans, cache/read replicas, denormalize a read model, or use a specialized analytics index without changing authority.

**60-second answer:** “SQL is strong for relational transactions and flexible queries; NoSQL is strong for known key/document/time-series/graph access patterns. Choice should follow invariants, consistency, queries, scale, and operations.”

## 10. Revision Notes

- SQL: relations, constraints, joins, ACID.
- DynamoDB: partition/sort key, known query patterns.
- Document/key-value/graph/time-series have different models.
- “NoSQL” does not mean no consistency or transactions.
- Multiple stores need clear source-of-truth boundaries.

**Challenge:** Compare PostgreSQL, DynamoDB, Redis, and OpenSearch for a product catalog, order, search, and session workload.
