# System Architecture: Diskless Kafka

## Table of Contents
1. [Introduction to Kafka](#introduction-to-kafka)
   - [Key Characteristics](#key-characteristics)
2. [Kafka's Commit Log Storage](#kafkas-commit-log-storage)
   - [What is a Commit Log?](#what-is-a-commit-log)
   - [How Kafka Stores the Commit Log](#how-kafka-stores-the-commit-log)
     - [1. Partition-Based Storage](#1-partition-based-storage)
     - [2. Log Segments](#2-log-segments)
     - [3. Disk I/O Optimization](#3-disk-io-optimization)
     - [4. Replication](#4-replication)
   - [Example: Message Flow to Disk](#example-message-flow-to-disk)
3. [Trade-offs of Traditional Kafka at Scale](#trade-offs-of-traditional-kafka-at-scale)
   - [1. Storage Costs](#1-storage-costs)
   - [2. Disk I/O Bottleneck](#2-disk-io-bottleneck)
   - [3. Operational Complexity](#3-operational-complexity)
   - [4. Retention vs. Cost Trade-off](#4-retention-vs-cost-trade-off)
   - [5. Replication Overhead](#5-replication-overhead)
   - [6. Scaling Limitations](#6-scaling-limitations)
4. [Diskless Kafka](#diskless-kafka)
   - [What is Diskless Kafka?](#what-is-diskless-kafka)
   - [How Diskless Kafka Works](#how-diskless-kafka-works)
     - [1. In-Memory Storage](#1-in-memory-storage)
     - [2. Replication-Based Durability](#2-replication-based-durability)
     - [3. Architecture](#3-architecture)
     - [4. Message Lifecycle](#4-message-lifecycle)
     - [5. Retention Mechanisms](#5-retention-mechanisms)
5. [Benefits of Diskless Kafka](#benefits-of-diskless-kafka)
   - [1. Dramatically Lower Costs](#1-dramatically-lower-costs)
   - [2. Extreme Performance](#2-extreme-performance)
   - [3. Simplified Operations](#3-simplified-operations)
   - [4. Better Cloud-Native Fit](#4-better-cloud-native-fit)
   - [5. Reduced Network Bandwidth](#5-reduced-network-bandwidth)
   - [6. Faster Failure Recovery](#6-faster-failure-recovery)
6. [Trade-offs of Diskless Kafka](#trade-offs-of-diskless-kafka)
   - [1. Limited Retention](#1-limited-retention)
   - [2. Memory Constraints](#2-memory-constraints)
   - [3. Data Loss Risk](#3-data-loss-risk)
   - [4. No Replay Capability](#4-no-replay-capability)
   - [5. Replication Overhead](#5-replication-overhead-1)
   - [6. Not Suitable for All Use Cases](#6-not-suitable-for-all-use-cases)
   - [7. Operational Complexity](#7-operational-complexity)
7. [When to Use Each Approach](#when-to-use-each-approach)
   - [Use Traditional Kafka When:](#use-traditional-kafka-when)
   - [Use Diskless Kafka When:](#use-diskless-kafka-when)
   - [Hybrid Approach:](#hybrid-approach)
8. [Conclusion](#conclusion)

---

## Introduction to Kafka

Apache Kafka is a distributed event streaming platform designed for high-throughput, low-latency data pipelines. It acts as a publish-subscribe messaging system where producers send messages to topics, and consumers read from those topics. Kafka's architecture is built around the concept of an **immutable commit log**, which is the foundation of its reliability and performance characteristics.

### Key Characteristics
- **Distributed**: Runs across multiple brokers for fault tolerance
- **Durable**: Messages are persisted to disk
- **Scalable**: Handles millions of messages per second
- **Ordered**: Maintains message order within partitions
- **Replayable**: Consumers can replay messages from any point in time

---

## Kafka's Commit Log Storage

### What is a Commit Log?

A commit log (also called an event log or write-ahead log) is an append-only data structure that records every event or transaction in the order it occurred. In Kafka, this is the fundamental mechanism for storing messages.

### How Kafka Stores the Commit Log

<img src='../../Resources/real-world-arch-resources/03-diskless-kafka/Screenshot 2026-02-22 at 2.02.59 PM.png' width=700 />

#### 1. **Partition-Based Storage**
- Each topic is divided into partitions
- Each partition is an ordered, immutable sequence of messages
- Messages are assigned sequential offsets (0, 1, 2, ...)
- Each partition is stored as a separate log file on disk

#### 2. **Log Segments**
Kafka doesn't store all messages in a single file. Instead, it breaks the log into segments:

```
Topic: user-events, Partition: 0
├── 00000000000000000000.log (messages 0-999)
├── 00000000000001000000.log (messages 1000-1999)
├── 00000000000002000000.log (messages 2000-2999)
└── 00000000000003000000.log (messages 3000+, active segment)
```

Each segment contains:
- **`.log` file**: Actual message data
- **`.index` file**: Offset index for fast lookups
- **`.timeindex` file**: Timestamp-based index

#### 3. **Disk I/O Optimization**
- **Sequential writes**: Messages are appended sequentially, which is extremely fast on disk
- **Memory-mapped files**: Kafka uses OS-level memory mapping for efficient I/O
- **Batch writes**: Multiple messages are written together
- **Compression**: Messages can be compressed (gzip, snappy, lz4, zstd)

#### 4. **Replication**
- Each partition has multiple replicas across different brokers
- One replica is the **leader** (handles reads/writes)
- Others are **followers** (replicate data)
- Replication ensures durability and fault tolerance

### Example: Message Flow to Disk

```
Producer sends message
    ↓
Message arrives at broker
    ↓
Written to memory buffer
    ↓
Flushed to disk (fsync)
    ↓
Replicated to follower brokers
    ↓
Acknowledged to producer
    ↓
Consumer can read from any replica
```

---

## Trade-offs of Traditional Kafka at Scale

<img src='../../Resources/real-world-arch-resources/03-diskless-kafka/Screenshot 2026-02-22 at 2.03.24 PM.png' width=750 />

### 1. **Storage Costs**

**Problem**: As data volume grows, disk storage becomes expensive.

- A single broker handling 1 million messages/second with 7-day retention requires:
  - ~600 GB per day of raw data
  - ~4.2 TB for 7-day retention
  - Multiply by replication factor (typically 3x) = **12.6 TB per broker**

**Impact**:
- High infrastructure costs for large-scale systems
- Expensive SSD storage for performance
- Data center space constraints
- Backup and disaster recovery overhead

### 2. **Disk I/O Bottleneck**

**Problem**: Disk I/O becomes a limiting factor at extreme scale.

- Sequential writes are fast, but still slower than memory operations
- Disk seeks for index lookups can cause latency spikes
- Multiple replicas writing simultaneously can saturate disk bandwidth
- Compaction (cleanup of old segments) requires additional I/O

**Impact**:
- Latency increases during peak traffic
- Throughput plateaus despite adding more brokers
- Compaction pauses can cause consumer lag

### 3. **Operational Complexity**

**Problem**: Managing disk-based systems at scale is complex.

- Disk failures require data recovery and rebalancing
- Capacity planning becomes critical (running out of disk space is catastrophic)
- Monitoring disk health, I/O patterns, and performance
- Backup and restore procedures are time-consuming
- Scaling requires provisioning new hardware with sufficient storage

**Impact**:
- Higher operational overhead
- Increased risk of data loss during failures
- Longer recovery times
- More complex disaster recovery procedures

### 4. **Retention vs. Cost Trade-off**

**Problem**: Longer retention periods multiply storage costs.

| Retention Period | Storage per Broker (1M msg/sec) | Cost Impact |
|---|---|---|
| 1 day | 1.8 TB | Baseline |
| 7 days | 12.6 TB | 7x |
| 30 days | 54 TB | 30x |
| 90 days | 162 TB | 90x |

**Impact**:
- Organizations must choose between retention and cost
- Cannot afford long-term data archival in Kafka
- Requires separate cold storage solutions for compliance

### 5. **Replication Overhead**

**Problem**: Replication multiplies storage and network costs.

- 3x replication = 3x storage cost
- 3x network bandwidth for replication traffic
- Replication lag can cause consistency issues
- Rebalancing after broker failures is slow

**Impact**:
- Cannot reduce replication factor without risking data loss
- Network bandwidth becomes expensive at scale
- Rebalancing causes temporary performance degradation

### 6. **Scaling Limitations**

**Problem**: Adding brokers requires provisioning new disks.

- Each new broker needs sufficient disk capacity
- Rebalancing data across brokers is slow and resource-intensive
- Disk-based systems don't scale as elastically as cloud-native systems
- Difficult to handle traffic spikes without pre-provisioning

**Impact**:
- Slower horizontal scaling
- Requires capacity planning months in advance
- Cannot quickly respond to unexpected traffic growth

---

## Diskless Kafka

### What is Diskless Kafka?

Diskless Kafka is an architectural variant where Kafka brokers **do not persist messages to disk**. Instead, messages are stored in **memory** and replicated across brokers. This approach trades durability for performance and cost efficiency.

### How Diskless Kafka Works
<img src='../../Resources/real-world-arch-resources/03-diskless-kafka/Screenshot 2026-02-22 at 2.02.23 PM.png' width=800 />

#### 1. **In-Memory Storage**
- Messages are stored in RAM instead of disk
- No disk I/O operations
- Extremely fast read/write operations
- Limited by available memory

#### 2. **Replication-Based Durability**
- Instead of disk persistence, durability comes from **replication**
- Messages are replicated to multiple brokers in memory
- If a broker fails, data is still available on replicas
- Requires sufficient replication factor (typically 3+)

#### 3. **Architecture**

```
Producer
    ↓
Broker 1 (Leader)
├── Message in memory
├── Replicate to Broker 2
├── Replicate to Broker 3
└── Acknowledge to producer
    ↓
Consumer reads from any replica
```

#### 4. **Message Lifecycle**

```
Message arrives
    ↓
Stored in memory buffer
    ↓
Replicated to follower brokers (in-memory)
    ↓
Acknowledged to producer
    ↓
Consumer reads from memory
    ↓
Message expires (TTL or offset retention)
    ↓
Memory freed
```

#### 5. **Retention Mechanisms**

Instead of time-based retention on disk, diskless Kafka uses:

- **Memory-based TTL**: Messages expire after a configurable time
- **Offset-based retention**: Keep only the last N messages
- **Size-based retention**: Evict oldest messages when memory limit reached
- **Consumer lag tracking**: Ensure replicas have time to consume before expiration

---

## Benefits of Diskless Kafka

### 1. **Dramatically Lower Costs**

**Advantage**: Eliminate expensive disk storage.

- No SSD/HDD costs
- No storage infrastructure
- Reduced data center footprint
- Lower power consumption (memory uses less power than spinning disks)

**Example**:
- Traditional Kafka: 12.6 TB storage per broker = $5,000-10,000/month
- Diskless Kafka: 256 GB RAM per broker = $500-1,000/month
- **Savings: 80-90% on storage costs**

### 2. **Extreme Performance**

**Advantage**: Memory-based operations are orders of magnitude faster.

- **Latency**: Sub-millisecond p99 latency (vs. 10-50ms with disk)
- **Throughput**: No disk I/O bottleneck
- **Consistency**: No replication lag due to disk sync delays
- **Predictability**: No garbage collection pauses from disk operations

**Benchmark**:
```
Traditional Kafka:
- Throughput: 500K msg/sec
- p99 latency: 25ms
- p99.9 latency: 100ms

Diskless Kafka:
- Throughput: 2M+ msg/sec
- p99 latency: 0.5ms
- p99.9 latency: 2ms
```

### 3. **Simplified Operations**

**Advantage**: No disk management complexity.

- No disk failure handling
- No capacity planning for storage
- No compaction operations
- No backup/restore procedures
- Faster broker startup (no disk recovery)
- Easier horizontal scaling

**Impact**:
- Reduced operational overhead
- Fewer failure modes
- Faster incident response
- Easier to automate

### 4. **Better Cloud-Native Fit**

**Advantage**: Aligns with cloud-native architecture.

- Stateless brokers (state is in memory, replicated)
- Easy to containerize and orchestrate
- Works well with Kubernetes
- Supports auto-scaling based on memory usage
- Ephemeral storage is acceptable

### 5. **Reduced Network Bandwidth**

**Advantage**: No disk I/O means less network traffic.

- Replication is the only network overhead
- No background compaction traffic
- No recovery traffic after failures
- More predictable network usage

### 6. **Faster Failure Recovery**

**Advantage**: No disk recovery needed.

- Broker restart is instant (no fsync recovery)
- Rebalancing is faster (no disk reads)
- Failover is immediate
- No data loss if replication factor is sufficient

---

## Trade-offs of Diskless Kafka

### 1. **Limited Retention**

**Trade-off**: Cannot store data long-term in Kafka.

- Retention is limited by available memory
- Typical retention: hours to days (not weeks/months)
- Cannot use Kafka as a data archive
- Requires external storage for long-term retention

**Impact**:
- Not suitable for compliance scenarios requiring long-term data retention
- Requires separate data lake/warehouse for archival
- Consumers must keep up with producers or lose data

**Example**:
```
Traditional Kafka: 30-day retention
Diskless Kafka: 4-hour retention (with 256GB RAM at 1M msg/sec)
```

### 2. **Memory Constraints**

**Trade-off**: Broker capacity is limited by RAM.

- Each broker can only hold as much data as its RAM
- Scaling requires adding more brokers (not just more storage)
- Memory is more expensive than disk
- Memory failures are catastrophic (no disk backup)

**Impact**:
- Cannot handle sudden traffic spikes (memory fills up)
- Requires careful capacity planning
- Vertical scaling is limited (can't add unlimited RAM to one machine)
- Higher per-broker cost than disk-based systems

### 3. **Data Loss Risk**

**Trade-off**: Replication is the only durability mechanism.

- If all replicas fail simultaneously, data is lost
- Network partitions can cause data loss
- No persistent backup on disk
- Requires very reliable replication (3+ replicas minimum)

**Scenarios**:
- Broker failure before replication completes → data loss
- Network partition isolating all replicas → data loss
- Cascading failures → data loss
- Software bugs affecting all replicas → data loss

**Impact**:
- Not suitable for mission-critical data
- Requires robust monitoring and alerting
- Higher operational risk
- May violate compliance requirements

### 4. **No Replay Capability**

**Trade-off**: Cannot replay old messages.

- Once messages expire from memory, they're gone
- Consumers cannot catch up if they fall behind
- No ability to reprocess historical data
- Requires external event store for replay scenarios

**Impact**:
- Cannot debug issues by replaying events
- Cannot onboard new consumers to historical data
- Requires separate event sourcing system
- Limits use cases (e.g., CQRS, event sourcing)

### 5. **Replication Overhead**

**Trade-off**: Replication is mandatory and expensive.

- Must replicate to multiple brokers (3+ minimum)
- Replication traffic consumes network bandwidth
- Replication latency affects producer acknowledgment
- Cannot reduce replication factor without risking data loss

**Impact**:
- Network bandwidth becomes a bottleneck
- Cannot use single-replica setup
- Replication lag can cause consistency issues
- Higher operational complexity

### 6. **Not Suitable for All Use Cases**

**Trade-off**: Limited applicability.

| Use Case | Traditional Kafka | Diskless Kafka |
|---|---|---|
| Real-time analytics | ✓ | ✓ |
| Event streaming | ✓ | ✓ |
| Log aggregation | ✓ | ✗ (need long retention) |
| Data archival | ✗ | ✗ |
| Event sourcing | ✓ | ✗ (need replay) |
| Compliance logging | ✗ | ✗ |
| Audit trails | ✗ | ✗ |
| Message queue | ✓ | ✓ |

### 7. **Operational Complexity**

**Trade-off**: Different operational model.

- Requires different monitoring (memory instead of disk)
- Different failure modes to handle
- Replication must be perfectly configured
- Consumer lag becomes critical (data expires)
- Requires careful TTL tuning

**Impact**:
- Requires specialized expertise
- Different runbooks and procedures
- More frequent operational decisions needed
- Higher risk of misconfiguration

---

## When to Use Each Approach

### Use Traditional Kafka When:

1. **Long-term retention needed**
   - Compliance requirements (audit logs, financial records)
   - Data archival and historical analysis
   - Event sourcing and replay scenarios

2. **Cost is secondary to reliability**
   - Mission-critical systems
   - Financial transactions
   - Healthcare data

3. **Data volume is moderate**
   - Can afford disk storage
   - Retention periods are reasonable
   - Compliance requirements are strict

4. **Operational simplicity is important**
   - Smaller teams
   - Limited DevOps resources
   - Prefer proven, battle-tested systems

### Use Diskless Kafka When:

1. **Real-time performance is critical**
   - Sub-millisecond latency required
   - Extremely high throughput needed
   - Predictable latency is essential

2. **Cost is a primary concern**
   - Startup/scale-up phase
   - High-volume, low-value data
   - Cloud-native environments with cost constraints

3. **Data retention is short**
   - Real-time analytics (hours/days)
   - Live event streaming
   - Temporary message queuing

4. **Cloud-native architecture**
   - Kubernetes deployments
   - Auto-scaling requirements
   - Ephemeral infrastructure

5. **Operational simplicity is important**
   - No disk management needed
   - Faster scaling
   - Simpler failure recovery

### Hybrid Approach:

Many organizations use **both**:

```
Real-time Pipeline:
Producer → Diskless Kafka (fast, cheap) → Consumer

Long-term Storage:
Consumer → Data Lake (S3, HDFS, etc.)

Replay/Audit:
Data Lake → Batch Processing
```

This combines the benefits of both approaches:
- Fast real-time processing with diskless Kafka
- Long-term retention in cost-effective storage
- Ability to replay data from the data lake
- Compliance with retention requirements

---

## Conclusion

The choice between traditional Kafka and diskless Kafka depends on your specific requirements:

- **Traditional Kafka** is the safe, proven choice for most production systems. It offers durability, long-term retention, and replay capabilities at the cost of higher storage expenses and operational complexity.

- **Diskless Kafka** is an optimization for specific scenarios where performance and cost matter more than durability and retention. It's ideal for real-time analytics, event streaming, and cloud-native systems.

In practice, most large-scale systems use a **hybrid approach**: diskless Kafka for real-time processing and traditional Kafka (or other storage) for long-term retention and compliance. This balances performance, cost, and reliability.

The key is understanding your data's value, retention requirements, and performance needs, then choosing the architecture that best serves those requirements.
