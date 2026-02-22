# Durability / Disaster Recovery

## Table of Contents
1. [What is Durability](#what-is-durability)
2. [How is Durability Achieved](#how-is-durability-achieved)
3. [The Durability Spectrum](#the-durability-spectrum)
   - [Weak Durability](#weak-durability)
   - [Moderate Durability](#moderate-durability)
   - [Strong Durability](#strong-durability)
   - [Maximum Durability](#maximum-durability)
4. [Different Levels of Durability](#different-levels-of-durability)
   - [In-Memory Only: No Durability](#in-memory-only-no-durability)
   - [Single Disk: Weak Durability](#single-disk-weak-durability)
   - [Sync Replication: Strong Durability](#sync-replication-strong-durability)
   - [Async Replication: Moderate-Strong Durability](#async-replication-moderate-strong-durability)
   - [Multi-Region: Maximum Durability](#multi-region-maximum-durability)
5. [Write Ahead Logs (WAL)](#write-ahead-logs-wal)
6. [FSync and Durability](#fsync-and-durability)
7. [Metrics: RPO and RTO](#metrics-rpo-and-rto)
   - [RPO (Recovery Point Objective)](#rpo-recovery-point-objective)
   - [RTO (Recovery Time Objective)](#rto-recovery-time-objective)
   - [RPO vs RTO Trade-off](#rpo-vs-rto-trade-off)
8. [Disaster Recovery Options](#disaster-recovery-options)
9. [Checkpointing](#checkpointing)
   - [Checkpointing Process](#checkpointing-process)
   - [Checkpointing Trade-offs](#checkpointing-trade-offs)
   - [Checkpoint Strategies](#checkpoint-strategies)
10. [When to Use Strong vs Relaxed Durability](#when-to-use-strong-vs-relaxed-durability)
   - [Decision Matrix](#decision-matrix)
   - [Use Cases for Strong Durability](#use-cases-for-strong-durability)
   - [Use Cases for Relaxed Durability](#use-cases-for-relaxed-durability)
11. [Real-World Examples](#real-world-examples)
    - [Example 1: Banking System with Maximum Durability](#example-1-banking-system-with-maximum-durability)
    - [Example 2: Real-Time Analytics Dashboard with Relaxed Durability](#example-2-real-time-analytics-dashboard-with-relaxed-durability)

---

## What is Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 5.55.25 PM.png' width='500' />

- **Definition**: Guarantee that committed data persists and survives system failures
- **Core Promise**: Once data is acknowledged as written, it will not be lost
- **Scope**: Protects against various failure scenarios (crashes, power loss, hardware failures)
- **Trade-off**: Durability guarantees come at the cost of latency and throughput
- **Critical for**: Financial systems, healthcare records, legal documents, any data that cannot be recreated

---

## How is Durability Achieved

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 5.54.38 PM.png' width='500' />

- **Persistent Storage**: Write data to disk/SSD instead of keeping only in memory
- **Replication**: Maintain multiple copies across different physical locations
- **Synchronous Writes**: Ensure data is written before acknowledging to client
- **Checksums & Verification**: Detect and prevent data corruption
- **Point-in-Time Recovery**: Maintain backups and transaction logs for recovery
- **Redundancy**: Multiple independent systems to survive single/multiple failures

---

## The Durability Spectrum

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 5.58.42 PM.png' width='700' />

### Weak Durability
- **Survives**: Server restart
- **Fails on**: Hardware failure, power loss
- **Example**: Single server with local disk
- **Use Case**: Development, non-critical caches

### Moderate Durability
- **Survives**: Server failure, single disk failure
- **Fails on**: Regional disaster, multiple simultaneous failures
- **Example**: RAID-configured single server
- **Use Case**: Small production systems, internal tools

### Strong Durability
- **Survives**: Regional disaster, multiple server failures
- **Fails on**: Catastrophic multi-region failure
- **Example**: Synchronous replication across 3+ availability zones
- **Use Case**: Critical production systems, financial data

### Maximum Durability
- **Survives**: Multi-region failures, extended outages
- **Fails on**: Extremely rare scenarios (multiple regions destroyed simultaneously)
- **Example**: Async replication across continents with point-in-time recovery
- **Use Case**: Mission-critical systems, regulatory compliance

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 5.59.25 PM.png' width='700' />

---

## Different Levels of Durability

### In-Memory Only: No Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.03.56 PM.png' height=250 />

**What it survives**: Nothing - data lost on any failure

**Example**:
```
Client → Redis (no persistence) → Data lost on restart
```

**Benefits**:
- Extreme performance (sub-millisecond latency)
- Simplicity - no disk I/O overhead
- Ideal for temporary data

**Trade-offs**:
- Complete data loss on failure
- Cannot recover any committed data
- Unsuitable for any critical data

**Use Case**: Session caches, real-time counters, temporary computations

---

### Single Disk: Weak Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.04.00 PM.png' height=250 />

**What it survives**: Server restart, process crash

**Example**:
```
Client → Server → Local Disk → Data persists across restarts
         ↓
    Disk failure = Data loss
```

**Benefits**:
- Simple architecture
- Reasonable performance (few milliseconds latency)
- Data survives process crashes

**Trade-offs**:
- Single disk failure = total data loss
- No redundancy
- Limited recovery options

**Use Case**: Development databases, non-critical staging systems

---

### Sync Replication: Strong Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.04.14 PM.png' height=250 />

**What it survives**: Single server failure, single disk failure, regional issues

**Example**:
```
Client → Primary Server → Disk 1
         ↓ (wait for ack)
         Replica 1 → Disk 2
         ↓ (wait for ack)
         Replica 2 → Disk 3
         ↓
    All acknowledge → Return success to client
```

**Benefits**:
- Data survives single/multiple server failures
- Strong consistency guarantees
- Point-in-time recovery possible
- Meets regulatory requirements

**Trade-offs**:
- Higher latency (10-50ms) - must wait for replicas
- Reduced throughput - synchronous writes
- Network dependency - slow replicas block writes
- Operational complexity

**Use Case**: Banking systems, financial transactions, healthcare records

---

### Async Replication: Moderate-Strong Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.04.18 PM.png' height=250 />

**What it survives**: Single server failure (with small data loss window), regional issues

**Example**:
```
Client → Primary Server → Disk 1 → Return success immediately
         ↓ (async, in background)
         Replica 1 → Disk 2
         ↓ (async, in background)
         Replica 2 → Disk 3
```

**Benefits**:
- Low latency (1-5ms) - return immediately
- High throughput - don't wait for replicas
- Good availability - survives most failures
- Scales better than sync replication

**Trade-offs**:
- Data loss possible if primary fails before replication
- Eventual consistency - replicas lag behind
- RPO measured in seconds/minutes
- Requires monitoring for replication lag

**Use Case**: E-commerce orders, user profiles, analytics data

---

### Multi-Region: Maximum Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.04.57 PM.png' height=250 />

**What it survives**: Regional disaster, multiple simultaneous failures, extended outages

**Example**:
```
Client → Region A (Primary)
         ├─ Server 1 → Disk 1
         ├─ Server 2 → Disk 2
         └─ Server 3 → Disk 3
         ↓ (async replication)
         Region B (Standby)
         ├─ Server 4 → Disk 4
         ├─ Server 5 → Disk 5
         └─ Server 6 → Disk 6
         ↓ (async replication)
         Region C (Backup)
         └─ Archive storage
```

**Benefits**:
- Survives entire region failure
- Highest availability (99.99%+)
- Disaster recovery built-in
- Meets strictest compliance requirements

**Trade-offs**:
- Highest latency (50-200ms) - cross-region replication
- Significant operational complexity
- Expensive - multiple regions, multiple copies
- Consistency challenges - eventual consistency only
- Data sovereignty/compliance issues

**Use Case**: Global financial systems, critical infrastructure, mission-critical SaaS

---

## Write Ahead Logs (WAL)

### What is WAL

- **Definition**: Technique where changes are written to a log file before being applied to main data structure
- **Order**: Log write → Acknowledge → Apply to data structure
- **Immutable**: Log entries are never modified, only appended
- **Recovery**: Log can be replayed to reconstruct state after failure

### What Problem Does It Solve

**Without WAL**:
```
Client request → Update in-memory data → Write to disk
                                         ↓ (crash here)
                                    Data lost!
```

**With WAL**:
```
Client request → Write to log (durable) → Update in-memory → Acknowledge
                                          ↓ (crash here)
                                    Replay log to recover
```

**Problems Solved**:
- Prevents data loss from crashes during writes
- Enables recovery without full data reconstruction
- Allows efficient batch writes to main storage
- Provides audit trail of all changes

### How WAL Works

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.10.25 PM.png' width=700 />

1. **Write Phase**: Append change to log file on disk
2. **Acknowledge Phase**: Return success to client (data is durable)
3. **Apply Phase**: Update in-memory data structure
4. **Checkpoint Phase**: Periodically flush data to main storage, truncate log

**Example Timeline**:
```
T1: Client sends "SET key=value"
T2: Write "SET key=value" to WAL on disk ✓
T3: Return success to client
T4: Update in-memory cache
T5: (Crash happens - no problem, log has the change)
T6: On recovery, replay WAL entries
T7: Reconstruct exact state before crash
```

**Benefits**:
- Durability without waiting for slow disk writes
- Efficient batch processing
- Complete recovery capability
- Minimal performance impact

---

## FSync and Durability


### What is FSync

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.27.53 PM.png' width=800 />

- **Definition**: System call that forces OS to write buffered data to physical disk
- **Without FSync**: OS buffers writes in memory, may lose data on power loss
- **With FSync**: Guarantees data reaches disk before returning

### How FSync Helps Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.29.07 PM.png' width=700 />

**Without FSync**:
```
Application → OS Buffer (in RAM) → Physical Disk
              ↓ (power loss)
              Data lost!
```

**With FSync**:
```
Application → OS Buffer → FSync call → Physical Disk ✓
                                       ↓ (power loss)
                                       Data safe!
```

### FSync Trade-offs

**Benefits**:
- Guarantees durability against power loss
- Prevents data corruption from unexpected shutdowns
- Essential for critical data

**Trade-offs**:
- **Latency**: FSync is slow (5-10ms per call)
- **Throughput**: Blocks until disk write completes
- **Scalability**: Becomes bottleneck under high load
- **Wear**: Increases SSD wear (limited write cycles)

### FSync Strategies

**Every Write** (Maximum Durability):
```
FOR each client request:
  Write to WAL
  FSync()  ← Slow but safest
  Acknowledge to client
```
- Latency: 10-50ms per write
- Use Case: Banking, financial transactions

**Every N Writes** (Balanced):
```
counter = 0
FOR each client request:
  Write to WAL
  counter++
  IF counter % 100 == 0:
    FSync()
  Acknowledge to client
```
- Latency: <1ms per write
- Data loss window: ~100 writes
- Use Case: Most production systems

**Every Second** (Relaxed):
```
Background thread:
  EVERY 1 second:
    FSync()

FOR each client request:
  Write to WAL (in memory buffer)
  Acknowledge to client
```
- Latency: <1ms per write
- Data loss window: ~1 second of writes
- Use Case: Analytics, caches, non-critical data

---

## Metrics: RPO and RTO

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.30.46 PM.png' width=700 />

### RPO (Recovery Point Objective)

**Definition**: Maximum acceptable amount of data loss measured in time

**Example**:
- RPO = 1 hour means you can lose up to 1 hour of data
- If system fails at 3:00 PM, you recover to 2:00 PM state
- Data from 2:00-3:00 PM is lost

**Calculation**:
```
RPO = Time between last backup and failure
```

**RPO by Durability Level**:
| Level | RPO |
|-------|-----|
| In-Memory | All data |
| Single Disk | All data |
| Sync Replication | 0 (no data loss) |
| Async Replication | Seconds to minutes |
| Multi-Region | Minutes to hours |

### RTO (Recovery Time Objective)

**Definition**: Maximum acceptable time to restore system to operational state

**Example**:
- RTO = 15 minutes means system must be back online within 15 minutes
- Includes detection time + failover time + startup time

**Calculation**:
```
RTO = Detection time + Failover time + Startup time
```

**RTO by Durability Level**:
| Level | RTO |
|-------|-----|
| In-Memory | Minutes (restart) |
| Single Disk | Minutes (restart + recovery) |
| Sync Replication | Seconds (automatic failover) |
| Async Replication | Seconds (automatic failover) |
| Multi-Region | Seconds (DNS failover) |

### RPO vs RTO Trade-off

```
High RPO (more data loss acceptable)
    ↓
Easier to achieve, cheaper
    ↓
Async replication, less frequent backups

Low RPO (less data loss acceptable)
    ↓
Harder to achieve, expensive
    ↓
Sync replication, frequent backups, multiple copies
```

---

## Disaster Recovery Options
<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-16 at 1.54.37 PM.png' width=800 />


## Checkpointing

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.34.00 PM.png' width=700 />

### What is Checkpointing

- **Definition**: Periodic snapshot of system state saved to durable storage
- **Purpose**: Reduce recovery time by avoiding full log replay
- **Frequency**: Every N operations or every T seconds

### How Checkpointing Works

**Without Checkpointing**:
```
Crash at T=1000
↓
Replay entire WAL from T=0 to T=1000
↓
Recovery time: 1000 operations
```

**With Checkpointing**:
```
Checkpoint at T=500 (save full state)
Crash at T=1000
↓
Load checkpoint from T=500
Replay WAL from T=500 to T=1000
↓
Recovery time: 500 operations
```

### Checkpointing Process

1. **Pause Writes**: Stop accepting new writes temporarily
2. **Flush Data**: Write all in-memory data to disk
3. **Mark Checkpoint**: Record checkpoint timestamp
4. **Resume Writes**: Accept new writes again
5. **Truncate Log**: Delete old WAL entries before checkpoint

### Checkpointing Trade-offs

**Benefits**:
- Faster recovery (don't replay entire log)
- Reduced disk space (can delete old logs)
- Predictable recovery time

**Trade-offs**:
- Pause during checkpoint (brief unavailability)
- Disk I/O spike during checkpoint
- Complexity in implementation
- Storage overhead (full state copies)

### Checkpoint Strategies

**Time-Based**:
```
Every 5 minutes:
  Create checkpoint
```
- Predictable recovery time
- May create unnecessary checkpoints

**Operation-Based**:
```
Every 100,000 operations:
  Create checkpoint
```
- Adapts to load
- May create checkpoints at bad times

**Hybrid**:
```
Every 5 minutes OR every 100,000 operations (whichever comes first)
```
- Balanced approach
- Most production systems use this

---

## When to Use Strong vs Relaxed Durability

### Decision Matrix

| Factor | Strong Durability | Relaxed Durability |
|--------|-------------------|-------------------|
| **Data Criticality** | Cannot be recreated | Can be recreated/regenerated |
| **Latency Tolerance** | Can accept 10-50ms | Need <5ms |
| **Cost Tolerance** | High cost acceptable | Cost-sensitive |
| **Failure Impact** | Financial/legal loss | Minor inconvenience |
| **Compliance** | Regulatory requirements | No strict requirements |
| **Data Volume** | Small to medium | Large volume |
| **Write Frequency** | Low to medium | High frequency |

### Use Cases for Strong Durability

**Financial Systems**:
- Bank transactions
- Payment processing
- Accounting records
- Investment portfolios

**Healthcare**:
- Patient medical records
- Prescription history
- Lab results
- Billing information

**Legal/Compliance**:
- Audit logs
- Regulatory records
- Contracts
- Compliance documentation

**E-Commerce**:
- Order data
- Customer information
- Inventory (critical)
- Payment records

### Use Cases for Relaxed Durability

**Analytics & Reporting**:
- Aggregated metrics
- Dashboard data
- Historical trends
- Non-real-time reports

**Caching**:
- Session data
- Temporary computations
- Cache layers
- Ephemeral state

**Real-Time Systems**:
- Live metrics
- Streaming data
- Real-time counters
- Temporary buffers

**Social Media**:
- Feed data
- Comments (can be regenerated)
- Likes/reactions
- Temporary notifications

---

## Real-World Examples

### Example 1: Banking System with Maximum Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.35.50 PM.png' height=350 />

**Scenario**: Online banking platform handling millions of transactions daily

**Architecture**:
```
Client Request (Transfer $1000)
    ↓
Primary Database (Region A, AZ 1)
    ├─ Write to WAL
    ├─ FSync on every commit
    ├─ Update in-memory state
    └─ Acknowledge to client
    ↓ (Sync replication)
Replica 1 (Region A, AZ 2)
    ├─ Write to WAL
    ├─ FSync on every commit
    └─ Acknowledge to primary
    ↓ (Sync replication)
Replica 2 (Region A, AZ 3)
    ├─ Write to WAL
    ├─ FSync on every commit
    └─ Acknowledge to primary
    ↓ (Async replication)
Backup Region (Region B)
    └─ Async copy for disaster recovery
```

**Durability Guarantees**:
- **RPO**: 0 seconds (no data loss)
- **RTO**: 5 seconds (automatic failover)
- **Survives**: Single/multiple server failures, entire AZ failure
- **Recovery**: 7-year point-in-time recovery capability

**Implementation Details**:
- Sync replication across 3+ availability zones
- FSync on every commit (no batching)
- Write-ahead logs with checksums
- Continuous replication monitoring
- Automated failover with health checks
- Daily backups to separate region
- Monthly full backups to cold storage

**Trade-offs**:
- **Latency**: +10-15ms per transaction (wait for replicas)
- **Throughput**: Limited by slowest replica
- **Cost**: 3x storage, 3x compute, cross-region bandwidth
- **Complexity**: Distributed consensus, failover logic, monitoring

**Benefits**:
- Zero data loss guarantee
- Regulatory compliance (PCI-DSS, SOX)
- Customer trust and confidence
- Legal protection

---

### Example 2: Real-Time Analytics Dashboard with Relaxed Durability

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.35.56 PM.png' height=350 />

**Scenario**: Live analytics dashboard showing user activity, metrics, trends

**Architecture**:
```
User Events (1M events/sec)
    ↓
Event Stream (Kafka)
    ↓
Redis Cluster (In-memory aggregates)
    ├─ FSync every 1 second
    ├─ Sub-millisecond latency
    └─ Real-time metrics
    ↓ (Async batch)
Data Warehouse (Parquet files)
    ├─ Batch writes every 5 minutes
    ├─ Durable long-term storage
    └─ Historical analysis
    ↓
Dashboard
    └─ Display real-time + historical data
```

**Durability Guarantees**:
- **RPO**: 1 second (Redis) + 5 minutes (warehouse)
- **RTO**: <1 second (Redis failover)
- **Survives**: Single Redis node failure
- **Data Loss**: Acceptable (can regenerate from events)

**Implementation Details**:
- Redis with fsync every 1 second
- In-memory aggregates (counters, percentiles)
- Async batch writes to warehouse
- Event stream replay capability
- No sync replication (too slow)
- Simple failover (restart from last checkpoint)

**Trade-offs**:
- **Data Loss**: Up to 1 second of metrics lost
- **Consistency**: Eventual consistency (lag between Redis and warehouse)
- **Complexity**: Simpler than banking system
- **Cost**: Minimal (single Redis cluster + warehouse)

**Benefits**:
- **Latency**: Sub-millisecond response times
- **Throughput**: Handles 1M+ events/second
- **Cost**: 10x cheaper than maximum durability
- **Simplicity**: Easier to operate and scale
- **Acceptable Loss**: Metrics can be regenerated from event stream

---

## Summary

<img src='../../Resources/17-system-architecture-basics/durability/Screenshot 2026-02-12 at 6.37.35 PM.png' width=500 />

| Aspect | Strong Durability | Relaxed Durability |
|--------|-------------------|-------------------|
| **Replication** | Sync across 3+ AZs | Async or in-memory |
| **FSync** | Every commit | Every second or batch |
| **RPO** | 0 seconds | Seconds to minutes |
| **RTO** | Seconds | Seconds to minutes |
| **Latency** | 10-50ms | <5ms |
| **Cost** | High | Low |
| **Use Case** | Financial, Healthcare | Analytics, Caching |
| **Data Loss** | Unacceptable | Acceptable |
| **Complexity** | High | Low |

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
