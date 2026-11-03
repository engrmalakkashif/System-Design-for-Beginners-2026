# 21. Database Sharding

## 1. Concept and Fundamentals

Sharding horizontally partitions database data across multiple independent database nodes. Each shard owns a subset identified by a shard key. It is a scaling strategy for high write volume, data size, or isolation that exceeds one database node.

Sharding is not simply replication. Replicas copy the same data; shards split ownership of the dataset.

## 2. Real-World Analogy

A school stores student records by first letter of surname across cabinets. Each cabinet is faster and smaller; finding a name requires knowing the cabinet. Cross-cabinet searches and moves become more complex.

## 3. Technical Architecture

```text
Router/proxy/application
 -> shard key hash/range lookup
 -> shard 0
 -> shard 1
 -> shard 2

Metadata: key -> shard mapping
```

Common sharding schemes:

- Hash-based: even distribution, difficult range scans/resizing.
- Range-based: ranges can be efficient but create hot ranges.
- Directory-based: explicit mapping enables flexibility but adds lookup dependency.
- Tenant-based: strong isolation but can create uneven tenants.

## 4. Practical Use Cases

- Large messaging/chat history keyed by conversation/user.
- IoT metrics by device/time with retention policies.
- Multi-tenant SaaS keyed by tenant only after isolation and fairness requirements.
- Regional transaction systems where data locality is required.

## 5. DevOps and AWS Implementation

AWS managed options include Amazon Aurora capacity scaling and DynamoDB partitioning before manual sharding. DynamoDB partition key design and repartitioning behavior usually come before custom application shards.

If custom sharding is needed, use a proxy/router, connection pool per shard, shard mapping, migration tooling, observability by shard, automated backups, and a rebalancing procedure. Athena, Redshift, or an analytics store can handle cross-shard reporting via exports instead of complex distributed queries.

Sharding metadata and routing secrets/security require high availability and auditing.

## 6. Practical Production Scenario

A messaging platform shards conversations by `conversation_id`. One API sends writes to the correct shard and queries direct conversation history from it. Cross-user inbox queries use an indexed event feed or materialized read store rather than querying every shard.

A hot celebrity conversation can cause a hot shard. Mitigate through range/salt based keys only when query semantics permit; otherwise isolate hot data and use caching/queues.

Resharding requires dual reads/writes, routing migration, backfill validation, and cutover. Keep transactions/joins within shard boundaries where possible.

## 7. Hands-On Lab

Build a simple deterministic sharder in a script for learning, not production:

```python
import hashlib

shards = ["shard-a", "shard-b", "shard-c"]

def shard_for(account_id):
    digest = hashlib.sha256(account_id.encode()).digest()
    return shards[int.from_bytes(digest[:4], "big") % len(shards)]

print(shard_for("account-1001"))
```

Send 100 IDs through it and count distribution. Test random ID spread and a list of sequential IDs. Observe that a sequential range is more likely to benefit range queries but may hot spot a workload.

## 8. Advantages, Disadvantages and Best Practices

- Increases write/data capacity beyond a single database.
- Can isolate tenant/regional data and operational recovery.
- Adds routing, cross-shard query, consistency, backup, migration, and operational complexity.
- Avoid sharding until vertical scaling, replicas, indexes, caching, archival, and workload isolation are insufficient.
- Choose a stable key that appears in real queries and has balanced cardinality.

Avoid a shard key that is tenant-default, monotonically increasing timestamp, Boolean, or a small enum with hot distributions.

## 9. Interview Preparation

**Q1: Sharding vs partitioning?** Sharding distributes independent dataset partitions across servers; partitioning may be logical inside or across storage.

**Q2: What is a shard key?** The value used to determine a data owner/shard.

**Q3: Why is shard key choice critical?** It determines distribution, query routing, and hotspot risk.

**Q4: What is cross-shard join?** A query that combines data owned by multiple shards, usually costly.

**Q5: How to reshard?** Use routing metadata, dual write/read, backfill, validation, cutover, and rollback.

**Scenario:** One shard receives 80% traffic. Inspect key skew and hot tenants; rebalance hot keys, cache reads, or redesign routing without breaking lookup semantics.

**Scenario:** Customer requests global invoice search. Query a materialized cross-shard search/report index; keep the database authoritative for mutations.

**60-second answer:** “Sharding splits a dataset across independent nodes by a routing key. It scales storage and writes but makes joins, transactions, resharding, recovery, and hotspots substantially harder, so it should be a measured last step.”

## 10. Revision Notes

- Shards split data ownership; replicas duplicate data.
- Choose a stable, high-cardinality, even shard key.
- Hash/range/directory are schemes.
- Rebalancing, metadata, and cross-shard reads are major risks.
- Prefer simpler capacity/data strategies first.

**Challenge:** Shard an e-commerce system and list which transactions stay inside a shard and which require a cross-service workflow/index.
