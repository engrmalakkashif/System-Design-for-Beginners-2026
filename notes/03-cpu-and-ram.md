# 3. CPU and RAM Limitations

## 1. Concept and Fundamentals

CPU is processing capacity for instructions. RAM is fast working memory for active processes, buffers, caches, application state, and data structures. Disk is persistent storage, typically slower than RAM. Network bandwidth is the rate data can move between services.

A bottleneck is the resource preventing a system from doing more work. Exhausting CPU, RAM, disk I/O, file descriptors, database connections, or network bandwidth can slow or stop a service.

### Resource Symptoms

| Resource | Typical Symptoms | Common Causes |
|---|---|---|
| CPU | High latency, throttling | Encryption, serialization, loops, insufficient workers |
| RAM | OOM kills, swap, GC pauses | Memory leaks, large payloads, caches |
| Disk | Queues and slow writes | Logs, database growth, low IOPS |
| Network | Timeouts and retransmission | Bandwidth limits, connection churn |

### Why It Matters

Each service instance has finite capacity. Scaling copies instances, but the system also needs database capacity, connection limits, and dependency budgets.

## 2. Real-World Analogy

CPU is the number of chefs cooking, RAM is the counter space and ingredients currently prepared, and disk is the warehouse. Too few chefs creates CPU queues; too little counter space forces chefs to repeatedly fetch ingredients; full warehouse shelves slow operations.

## 3. Technical Architecture

```text
Client
 -> ALB
 -> Application worker pool
      CPU: parsing, encryption, business logic
      RAM: request state, heap, buffers, cache
      Disk: logs, temporary files, databases
      Network: inbound/outbound calls
```

Linux tools expose pressure through `/proc`, `top`, `htop`, `vmstat`, `iostat`, `free`, `ss`, and `sar`/CloudWatch.

## 4. Practical Use Cases

- Image-processing service benefits from CPU/worker tuning.
- Redis benefits from sufficient RAM, because evictions increase latency and misses.
- Video transcoding needs CPU, memory, disk throughput, and duration limits.
- API middleware may bottleneck on CPU due to TLS and JSON serialization.

## 5. DevOps and AWS Implementation

EC2 instance families provide different CPU/RAM ratios. Use CloudWatch metrics for CPU utilization, memory requires an agent or application metric, and EBS `VolumeReadOps`/`VolumeWriteOps`. Containers need CPU/memory requests and limits in ECS/EKS.

For Lambda, memory allocation also changes available CPU proportionally. For Fargate, CPU and memory must be paired within supported configurations. For RDS, instance memory affects buffer pools, query cache behavior, and connection headroom.

Set alarms on trends and saturation, not only average CPU. Use load tests to establish breaking points and define autoscaling policies with cooldowns.

## 6. Practical Production Scenario

A .NET API begins timing out during payroll processing. CloudWatch shows moderate CPU but high RAM and frequent garbage collection. New instances may reduce request queue briefly, but memory profiling discovers response objects retained after PDF generation.

Troubleshooting:

```bash
top -o %MEM
free -h
vmstat 1
pidstat 1
journalctl -u your-service --since '-30 min'
```

Fix the leak, cap worker concurrency, stream large reports, increase memory, and add a memory limit so the orchestrator restarts a runaway task safely.

## 7. Hands-On Lab

Create a CPU load container and observe it:

```bash
docker run -d --name cpu-load alpine:3.20 sh -c \
  'while true; do :; done'

docker stats cpu-load
```

Stop it:

```bash
docker stop cpu-load && docker rm cpu-load
```

Then run a local API load test with a tool such as `hey`:

```bash
hey -z 30s -c 20 http://localhost:8080/health
```

Record p50/p95 latency, CPU, memory, request throughput, and error rate. Increase concurrency only after baselines are recorded.

## 8. Advantages, Disadvantages and Best Practices

- CPU limits should include request and burst behavior.
- Memory limits prevent one process from harming neighbors, but poor limits cause OOM termination.
- Measure p95/p99 latency and tail errors.
- Pool connections and threads; unbounded concurrency moves bottlenecks elsewhere.
- Stream large files instead of loading them entirely into memory.

Do not autoscale only on average CPU when traffic bursts or memory limits are the actual constraint.

## 9. Interview Preparation

**Q1: What is CPU?** The compute capacity for executing instructions.

**Q2: What is a memory leak?** Memory remains allocated after it is no longer needed.

**Q3: CPU high and RAM low?** Tune CPU-heavy code or scale out compute.

**Q4: How do threads affect CPU/RAM?** They improve parallelism to a limit, but increase memory and synchronization overhead.

**Q5: How would you troubleshoot latency?** Establish metrics, locate the bottleneck, profile the process, load test, change one variable, and remeasure.

**Scenario:** Autoscaling adds instances but latency remains high. Check database connections, downstream dependencies, and instance quotas.

**Scenario:** Containers restart every few minutes. Inspect OOMKilled status, memory limits, heap settings, and retention leaks.

**60-second answer:** “CPU, memory, disk, and network are finite resources. A production system identifies saturation, protects process limits, scales the correct layer, and measures percentiles under realistic load.”

## 10. Revision Notes

- CPU: instruction execution.
- RAM: active working data.
- Disk/network: data durability and transfer.
- High average usage is less important than saturation and tail latency.
- Load, profile, change one factor, then compare.

**Challenge:** Give an API container 256 MB RAM and deliberately produce a memory-intensive request. Observe OOM behavior and then redesign it with streaming and limits.
