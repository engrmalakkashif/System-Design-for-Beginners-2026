# 20. Database Replication

## 1. Concept and Fundamentals

Database replication copies write-ahead/transaction/change data from a primary/source to one or more replicas. It can improve availability, read scale, disaster recovery, and geographic access.

Replication modes include:

- Synchronous: source waits for replica acknowledgment.
- Asynchronous: source does not wait for every replica.
- Semi-synchronous: waits for at least one acknowledgement depending on configuration.

## 2. Real-World Analogy

The original legal book is copied to branch libraries. A synchronously maintained copy is updated during the transaction, while a copy updated overnight may be slightly behind.

## 3. Technical Architecture

```text
Application writes -> primary
Primary -> replication stream -> replicas
Reads -> primary or replicas based on consistency needs

Promotion workflow:
replica lag/health check -> choose promotion candidate -> DNS/endpoint switch
```

Replica types include read replicas, standby Multi-AZ nodes, and cross-region replicas. Each serves a different primary goal.

## 4. Practical Use Cases

- Reporting reads go to replicas to protect transaction workload.
- Multi-AZ standby reduces primary failure impact.
- Cross-region replica supports regional disaster recovery and read locality.
- Analytics replicas isolate expensive queries.

## 5. DevOps and AWS Implementation

RDS Multi-AZ provides synchronous-ish availability within the database engine/model and automatic primary failover. A read replica is asynchronous and designed for read scaling or promotion, with engine-specific constraints. RDS Proxy can reduce connection storms, but it does not solve replica lag or wrong queries.

Monitor write I/O, network replication throughput, replica lag, promotion process, backup/PITR, DNS/endpoint behavior, and read-after-write correctness.

For Aurora, reader endpoints distribute reads across replicas; a custom endpoint or application routing may isolate analytical traffic.

## 6. Practical Production Scenario

An API has 100 read RPS, 20 write RPS. It routes non-critical reads to a RDS read replica but routes profile writes and immediately following reads to the primary. Every user mutation returns a status version; read-after-write endpoints include a consistency strategy.

Failover promotes a standby. The API must tolerate connection retry, a short primary endpoint transition, and applications reconnecting rather than caching stale connection state.

## 7. Hands-On Lab

On PostgreSQL, configure physical or logical replication only in an environment where you understand disk/network requirements. For a simpler AWS-free lab, start primary and replica containers and demonstrate backup/restore, but a replica requires streaming replication configuration.

High-level steps:

1. Configure primary WAL/replication access.
2. Copy baseline with `pg_basebackup`.
3. Configure replica recovery.
4. Start replica in read-only mode.
5. Insert a row on primary.
6. Read it from replica, account for asynchronous lag.
7. Stop primary and document promotion procedure.

Avoid a manual lab where promotion is practiced without DNS/endpoint automation in production.

## 8. Advantages, Disadvantages and Best Practices

- Improves availability and read locality/scale.
- Read replicas can isolate reporting.
- Asynchronous replicas can lag.
- Replication can cross regional cost and privacy boundaries.
- Prevent unsafe random read routing for transactional reads.

Test failover, connection behavior, DNS/endpoint updates, application retries, and replica replacement.

## 9. Interview Preparation

**Q1: Synchronous vs asynchronous replication?** Sync waits for acknowledgement; async does not wait on the source path.

**Q2: Multi-AZ vs read replica?** Multi-AZ is availability; read replica adds read capacity and/or cross-region strategy.

**Q3: Why replica lag?** Network, source load, replay work, snapshots, network partitions, or replica storage can delay changes.

**Q4: What consistency guarantee do replicas offer?** It varies; write-from-primary/read-from-replicas is often eventually consistent.

**Q5: Can replication protect logical corruption?** No; corrupted changes can replicate and backups/PITR are needed.

**Scenario:** An updated profile is still old after redirect. Route reads for the authenticated user to primary after writes or use a version/replica lag token.

**Scenario:** Failover causes duplicate writes or client errors. Add idempotency, bounded retries, reconnect handling, transaction status APIs, and health checks.

**60-second answer:** “Replication copies database changes to additional nodes. It supports availability and read scaling, but asynchronous replicas may lag and require consistency-aware routing and tested failover.”

## 10. Revision Notes

- Primary is write authority.
- Synchronous favors consistency/availability trade-offs.
- Replica lag is normal in async systems.
- Multi-AZ ≠ read replica.
- Test promotion and application reconnect behavior.

**Challenge:** Design primary/read replica routing for a social profile API and define which routes require primary consistency.
