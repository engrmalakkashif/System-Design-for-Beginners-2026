# 7. Replication and Backups

## 1. Concept and Fundamentals

Replication copies data or systems to multiple locations to improve availability and/or read capacity. Backups create restorable copies for recovery from deletion, corruption, ransomware, accidental updates, and major outages.

They are related but not interchangeable:

- Replication keeps near-real-time copies but can replicate corruption.
- Backups preserve earlier states for point-in-time recovery.

### Why They Matter

One disk, host, AZ, or destructive command can destroy access to critical data.

### Without Them

A failed disk or accidental `DELETE FROM users` can become permanent loss.

## 2. Real-World Analogy

Replication is having duplicate cash registers that update each other immediately. Backups are the vault’s daily archived transaction ledger, usable to recover history even if a recent mistake reaches the registers.

## 3. Technical Architecture

```text
Primary -> synchronous/asynchronous replica
        -> cross-AZ or cross-region copy

Backup service -> snapshots -> object storage / vault
                -> point-in-time recovery logs

Recovery -> restore candidate -> validate -> promote
```

## 4. Practical Use Cases

- Multi-AZ database failover for availability.
- Cross-region replicas for disaster recovery or reduced read latency.
- S3 versioning and backup vault for object recovery.
- Read replicas for reporting workloads.

## 5. DevOps and AWS Implementation

RDS supports automated backups and snapshots; Multi-AZ is availability-oriented, not a performance read replica. EC2 snapshots/EBS, S3 versioning, AWS Backup, and DynamoDB PITR add other recovery options.

Define RPO (maximum tolerated data loss) and RTO (maximum tolerated recovery time). Cross-region replication, immutable backups, encryption, access policies, and testing must match those targets.

Monitor replica lag, failed backup jobs, replication events, storage capacity, and restore success.

## 6. Practical Production Scenario

An RDS MySQL database uses Multi-AZ and nightly backups with PITR. A deployment migration accidentally modifies records. An engineer restores an isolated database snapshot to a previous point, compares data, and performs a controlled data repair. The primary remains available, but application traffic is switched only after validation.

Backup access should be tightly limited and logs should be protected against deletion.

## 7. Hands-On Lab

Back up and restore a local PostgreSQL database:

```bash
docker run -d --name pg-backup -e POSTGRES_PASSWORD=demo postgres:16

docker exec pg-backup pg_dump -U postgres postgres > backup.sql

docker exec pg-backup dropdb -U postgres postgres 2>/dev/null || true
docker exec -i pg-backup psql -U postgres < backup.sql
docker exec pg-backup psql -U postgres -c '\l'
```

For point-in-time recovery, use the cloud provider or WAL archiving rather than relying on this basic physical SQL dump example.

## 8. Advantages, Disadvantages and Best Practices

- Replication improves failover/read scale.
- Backups permit recovery from logical and regional disasters.
- Test restore procedures, not only backup creation.
- Protect backup credentials and add retention/deletion controls.
- Measure restoration through real drills.

Do not assume snapshots are instant, consistent, or testable without validation.

## 9. Interview Preparation

**Q1: Replication vs backup?** Replication is live/copy availability; backup is recoverable historical state.

**Q2: RPO vs RTO?** RPO is acceptable data loss; RTO is acceptable recovery time.

**Q3: Multi-AZ vs read replica?** Multi-AZ is standby for availability/failover; read replicas serve read traffic and may be promoted or replicated.

**Q4: How often do backups run?** Based on RPO, data criticality, storage cost, and recovery tests.

**Q5: What cannot replication protect against?** Replicating accidental deletion, corruption, application bugs, or most ransomware unless protected copies exist.

**Scenario:** A table was dropped. Stop writes, identify the last valid restore point, restore to a separate environment, validate, and migrate only the required data.

**Scenario:** Region outage. Promote replicas, update DNS or routing, validate consistency/security, and communicate recovery progress.

**60-second answer:** “Replication increases availability and read scale, while backups preserve independent recoverable states. A complete design defines RPO/RTO and regularly proves restoration.”

## 10. Revision Notes

- RPO: data loss tolerance.
- RTO: recovery time tolerance.
- Replication protects some availability failures; backups protect history and destruction.
- Restore drills are required.

**Challenge:** For a database, define RPO=5 minutes, RTO=30 minutes, and choose/logically justify AWS components plus restore tests.
