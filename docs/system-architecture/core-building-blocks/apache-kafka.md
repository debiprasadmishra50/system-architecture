# Apache Kafka

## Table of Contents

1. [What is Apache Kafka](#what-is-apache-kafka)
2. [Problems It Solves](#problems-it-solves)
3. [Comparison with Other Message Brokers](#comparison-with-other-message-brokers)
4. [Kafka vs P2P and PUBSUB](#kafka-vs-p2p-and-pubsub)
5. [Core Components](#core-components)
6. [Publish/Subscribe Architecture](#publishsubscribe-architecture)
7. [Kafka Cluster, Topics, Partitions & Offsets](#kafka-cluster-topics-partitions--offsets)
8. [Serialization and Deserialization](#serialization-and-deserialization)
9. [Rebalancing in Kafka](#rebalancing-in-kafka)
10. [Kafka Semantics](#kafka-semantics)
11. [Kafka Brokers](#kafka-brokers)
12. [Fault Tolerance, Reliability & Scalability](#fault-tolerance-reliability--scalability)
13. [When to Use and When to Avoid](#when-to-use-and-when-to-avoid)

---

## What is Apache Kafka

Apache Kafka is a distributed event streaming platform designed for high-throughput, low-latency data pipelines. It acts as a central hub for real-time data feeds, enabling applications to publish and subscribe to streams of records.

**Key Characteristics:**
- Distributed, fault-tolerant architecture
- Persistent message storage with configurable retention
- High throughput (millions of messages per second)
- Horizontal scalability
- Ordered message delivery within partitions
- Built-in replication and failover mechanisms

---

## Problems It Solves

| Problem | Solution |
|---------|----------|
| **Data Silos** | Centralizes data streams from multiple sources into a single platform |
| **Real-time Processing** | Enables low-latency event streaming for immediate data processing |
| **Decoupling** | Decouples producers from consumers; they don't need to know about each other |
| **Data Loss** | Persistent storage with replication ensures no message loss |
| **Scalability** | Handles millions of events per second across distributed clusters |
| **Ordering Guarantees** | Maintains message order within partitions for consistent processing |
| **Replay Capability** | Consumers can replay messages from any offset for reprocessing |

---

## Comparison with Other Message Brokers

| Feature | Kafka | IBM MQ | RabbitMQ |
|---------|-------|--------|----------|
| **Architecture** | Distributed, log-based | Centralized queue | Centralized broker |
| **Throughput** | Very High (millions/sec) | Medium | Medium-High |
| **Latency** | Low | Medium | Low-Medium |
| **Persistence** | Built-in, configurable | Built-in | Optional (plugins) |
| **Replication** | Native, automatic | Manual setup | Manual setup |
| **Scalability** | Horizontal | Vertical | Horizontal (limited) |
| **Message Ordering** | Per partition | Per queue | Per queue |
| **Retention** | Time/Size-based | Until consumed | Until consumed |
| **Use Case** | Event streaming, big data | Enterprise messaging | Task queues, microservices |
| **Learning Curve** | Moderate-High | High | Low |
| **Cost** | Open-source | Commercial | Open-source |

---

## Kafka vs P2P and PUBSUB

| Aspect | Kafka | P2P (Peer-to-Peer) | Traditional PUBSUB |
|--------|-------|-------------------|---------------------|
| **Architecture** | Broker-based distributed log | Direct peer connections | Broker-based (centralized) |
| **Persistence** | Messages stored durably | No persistence | Messages deleted after delivery |
| **Scalability** | Horizontal (add brokers) | Limited by peer count | Limited by broker capacity |
| **Message Ordering** | Guaranteed per partition | No ordering | No ordering guarantee |
| **Replay** | Full replay capability | Not possible | Not possible |
| **Fault Tolerance** | High (replication) | Low (peer dependent) | Medium (broker dependent) |
| **Latency** | Low | Very Low | Low |
| **Throughput** | Very High | Medium | Medium |
| **Decoupling** | Temporal & Spatial | Spatial only | Temporal & Spatial |
| **Best For** | Event streaming, data pipelines | File sharing, distributed systems | Real-time notifications |

---

## Core Components
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.29.20 PM.png'  width=700 />


### Producer

- **Role:** Publishes messages to Kafka topics
- **Responsibilities:**
  - Serializes data
  - Selects target partition (via key or round-robin)
  - Batches messages for efficiency
  - Handles retries and acknowledgments
- **Configuration:**
  - `acks`: Acknowledgment level (0, 1, all)
  - `batch.size`: Messages per batch
  - `linger.ms`: Wait time before sending batch
  - `compression.type`: Message compression (snappy, lz4, gzip)

### Queue (Topic Partitions - FIFO)

- **Role:** Ordered, immutable log of messages
- **Characteristics:**
  - FIFO ordering within a partition
  - Append-only structure
  - Configurable retention policy
  - Replicated across brokers
- **Structure:**
  ```
  Partition 0: [msg0] → [msg1] → [msg2] → [msg3]
  Partition 1: [msg0] → [msg1] → [msg2]
  Partition 2: [msg0] → [msg1] → [msg2] → [msg3] → [msg4]
  ```

### Consumer

- **Role:** Reads messages from Kafka topics
- **Responsibilities:**
  - Subscribes to topics/partitions
  - Tracks offset (position in partition)
  - Processes messages
  - Commits offsets for fault recovery
- **Configuration:**
  - `group.id`: Consumer group identifier
  - `auto.offset.reset`: Behavior when offset is lost (earliest, latest)
  - `max.poll.records`: Messages per poll
  - `session.timeout.ms`: Heartbeat timeout

---

## Publish/Subscribe Architecture


```
┌─────────────┐
│  Producer 1 │
└──────┬──────┘
       │
       ├─────────────────────────────────┐
       │                                 │
       ▼                                 ▼
┌──────────────────────────────────────────────┐
│         Kafka Cluster (Brokers)              │
│  ┌────────────────────────────────────────┐  │
│  │  Topic: user-events                    │  │
│  │  ├─ Partition 0: [msg] → [msg] → ...   │  │
│  │  ├─ Partition 1: [msg] → [msg] → ...   │  │
│  │  └─ Partition 2: [msg] → [msg] → ...   │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
       ▲                                 ▲
       │                                 │
       ├─────────────────────────────────┤
       │                                 │
┌──────┴──────┐                   ┌──────┴──────┐
│ Consumer 1  │                   │ Consumer 2  │
│ (Group A)   │                   │ (Group B)   │
└─────────────┘                   └─────────────┘
```

**Key Features:**
- **Decoupling:** Producers and consumers are independent
- **Multiple Subscribers:** Multiple consumer groups can read same topic
- **Retention:** Messages persist for configured duration
- **Replay:** Consumers can reset offset to replay messages

---

## Kafka Cluster, Topics, Partitions & Offsets
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.38.46 PM.png'  width=700 />

### Cluster

- Collection of brokers working together
- Minimum 3 brokers recommended for production
- Brokers coordinate via ZooKeeper (or KRaft in newer versions)
- Each broker stores partition replicas

### Topics
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.33.09 PM.png'  height=450 />

- Logical channel for messages
- Organized into partitions for parallelism
- Configurable retention (time or size-based)
- Example: `user-events`, `order-transactions`, `logs`

### Partitions
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.36.11 PM.png'  width=700 />

- Physical unit of parallelism
- Each partition is an ordered, immutable log
- Messages distributed across partitions via key hash
- Enables parallel processing by multiple consumers

```
Topic: orders
├─ Partition 0: [order-1] → [order-4] → [order-7]
├─ Partition 1: [order-2] → [order-5] → [order-8]
└─ Partition 2: [order-3] → [order-6] → [order-9]

(Messages with same key go to same partition)
```

### Offsets

- Unique identifier for each message within a partition
- Sequential integer starting from 0
- Consumer tracks current offset to resume from last processed message
- Enables exactly-once and at-least-once semantics

```
Partition 0:
Offset: 0    1    2    3    4    5
        ↓    ↓    ↓    ↓    ↓    ↓
        msg→msg→msg→msg→msg→msg
                        ↑
                   Consumer offset (3)
```

---
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.36.50 PM.png'  width=700 />

## Serialization and Deserialization

### Serialization (Producer)

Converts application objects to bytes for transmission:

- **JSON:** Human-readable, schema-less, larger payload
- **Avro:** Compact, schema-based, efficient
- **Protobuf:** Strongly-typed, compact, language-agnostic
- **String/Bytes:** Simple, no schema validation

**Example (Avro):**
```
User object → Avro Serializer → Binary bytes → Kafka
```

### Deserialization (Consumer)

Converts bytes back to application objects:

- **Schema Registry:** Centralized schema management (Confluent)
- **Schema Evolution:** Handle schema changes gracefully
- **Compatibility:** Backward/forward/full compatibility modes

**Example (Avro with Schema Registry):**
```
Binary bytes → Schema Registry lookup → Avro Deserializer → User object
```

---

## Rebalancing in Kafka

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.43.36 PM.png'  width=700 />

### What is Rebalancing?

Process of redistributing partitions among consumer group members when:
- Consumer joins/leaves group
- Consumer crashes
- New partitions added to topic

### Rebalancing Process

```
Initial State:
Consumer Group: [C1, C2]
Topic Partitions: [P0, P1, P2, P3]
Assignment: C1→[P0,P1], C2→[P2,P3]

↓ (C3 joins)

Rebalancing Triggered:
1. Stop consuming
2. Revoke current partitions
3. Rebalance coordinator assigns new partitions
4. Resume consuming

New State:
Assignment: C1→[P0,P2], C2→[P1], C3→[P3]
```

### Rebalancing Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| **Range** | Assign consecutive partitions | Simple, predictable |
| **Round-Robin** | Distribute evenly across consumers | Balanced load |
| **Sticky** | Minimize partition movement | Reduce stop-the-world time |
| **Cooperative** | Gradual reassignment | Zero downtime rebalancing |

### Impact

- **Stop-the-World:** All consumers pause during rebalancing
- **Duration:** Typically 1-30 seconds (configurable)
- **Mitigation:** Use sticky/cooperative assignors, tune timeouts

---

## Kafka Semantics
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.44.33 PM.png'  width=600 />
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.45.38 PM.png'  width=600 />

### At Most Once

**Definition:** Message processed 0 or 1 time; some messages may be lost.

**How it works:**
- Consumer commits offset BEFORE processing
- If consumer crashes after commit, message is lost
- No retries on failure

**Pros & Cons:**

| Pros | Cons |
|------|------|
| Highest throughput | Data loss possible |
| Lowest latency | Not suitable for critical data |
| Simplest implementation | Unreliable |

**Use Case:** Analytics, metrics, non-critical logs

---

### At Least Once

**Definition:** Message processed 1 or more times; no message loss but duplicates possible.

**How it works:**
- Consumer processes message BEFORE committing offset
- If consumer crashes before commit, message reprocessed
- Duplicates handled by idempotent processing

**Pros & Cons:**

| Pros | Cons |
|------|------|
| No data loss | Possible duplicates |
| Reliable | Requires idempotent logic |
| Suitable for most use cases | Slightly lower throughput |

**Use Case:** Financial transactions, order processing, critical events

---

### Exactly Once

**Definition:** Message processed exactly 1 time; no loss, no duplicates.

**How it works:**
- Transactional writes to Kafka and external system
- Atomic commit of offset and processing result
- Idempotent consumer logic
- Requires transactional support in sink system

**Pros & Cons:**

| Pros | Cons |
|------|------|
| No data loss | Highest latency |
| No duplicates | Complex implementation |
| Most reliable | Lowest throughput |
| | Requires transactional sink |

**Use Case:** Banking, payment systems, audit logs

---

## Kafka Brokers

### What is a Broker?

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.47.37 PM.png'  width=800 />

A Kafka broker is a server instance that:
- Stores partition replicas
- Handles producer/consumer requests
- Manages replication and failover
- Coordinates with cluster via controller

### Broker Architecture

```
┌─────────────────────────────────────┐
│         Kafka Broker                │
├─────────────────────────────────────┤
│  Partition Replicas:                │
│  ├─ Topic-A, Partition-0 (Leader)   │
│  ├─ Topic-A, Partition-1 (Replica)  │
│  ├─ Topic-B, Partition-0 (Replica)  │
│  └─ Topic-B, Partition-2 (Leader)   │
├─────────────────────────────────────┤
│  Request Handler (Producer/Consumer)│
│  Replication Manager                │
│  Log Manager                        │
│  Controller (if elected)            │
└─────────────────────────────────────┘
```

### Leader and Replicas

- **Leader:** Handles all reads/writes for partition
- **Replicas:** In-sync replicas (ISR) backup data
- **Failover:** If leader fails, ISR elected as new leader

### Benefits of Kafka Brokers

| Benefit | Description |
|---------|-------------|
| **Fault Tolerance** | Replication across brokers prevents data loss |
| **High Availability** | Automatic failover to replica brokers |
| **Load Distribution** | Partition leaders spread across brokers |
| **Scalability** | Add brokers to increase capacity |
| **Durability** | Persistent storage on disk |

### Tradeoffs of Kafka Brokers

| Tradeoff | Impact |
|----------|--------|
| **Operational Complexity** | Requires cluster management, monitoring |
| **Resource Overhead** | Replication consumes storage and network |
| **Latency** | Replication adds slight latency |
| **Cost** | Multiple brokers increase infrastructure cost |
| **Configuration** | Many tuning parameters to optimize |

---

## Fault Tolerance, Reliability & Scalability
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 4.48.41 PM.png'  height=500 />

### Fault Tolerance

**Replication Strategy:**
- Each partition replicated across N brokers (replication factor)
- In-Sync Replicas (ISR) maintain consistency
- Minimum ISR ensures durability before acknowledgment

```
Topic: orders (replication factor = 3)
├─ Partition 0:
│  ├─ Broker 1 (Leader)
│  ├─ Broker 2 (Replica)
│  └─ Broker 3 (Replica)
└─ Partition 1:
   ├─ Broker 2 (Leader)
   ├─ Broker 3 (Replica)
   └─ Broker 1 (Replica)
```

**Failure Scenarios:**
- **Broker Failure:** Replicas on other brokers take over
- **Network Partition:** ISR shrinks, availability reduced
- **Disk Failure:** Replica rebuilt from other brokers

### Reliability

**Durability Guarantees:**
- `acks=all`: Wait for all ISR replicas to acknowledge
- `min.insync.replicas`: Minimum replicas for write success
- Configurable retention ensures message persistence

**Consumer Reliability:**
- Offset commits track progress
- Rebalancing ensures no message loss
- Exactly-once semantics available with transactions

### Scalability

**Horizontal Scaling:**
- Add brokers to cluster
- Rebalance partitions across new brokers
- Increase throughput linearly with broker count

**Partition Scaling:**
- Increase partition count for parallelism
- Consumers scale with partition count
- Rebalancing distributes load

**Performance Metrics:**
- Throughput: Millions of messages/second
- Latency: Sub-second end-to-end
- Storage: Configurable retention (days/weeks/months)

---

## When to Use and When to Avoid

### When to Use Kafka ✓

- **Real-time Data Pipelines:** Stream processing, ETL
- **Event Sourcing:** Immutable event log for audit trails
- **Microservices Communication:** Decoupled async messaging
- **Log Aggregation:** Centralized log collection
- **Metrics & Monitoring:** High-volume metric streaming
- **Activity Tracking:** User behavior, clickstreams
- **Data Replication:** Sync data across systems
- **High-Throughput Systems:** Millions of events/second

### When NOT to Use Kafka ✗

- **Request-Response Patterns:** Use REST/gRPC instead
- **Low Latency Critical:** P2P or in-memory queues better
- **Simple Task Queues:** RabbitMQ or Redis more suitable
- **Small Scale:** Overhead not justified for low volume
- **Transactional Consistency:** Database transactions better
- **Complex Routing:** Message broker with routing logic
- **Synchronous Processing:** Direct service calls preferred
- **Small Team:** Operational complexity requires expertise

### Decision Matrix

| Requirement | Kafka | RabbitMQ | Redis | Direct Call |
|-------------|-------|----------|-------|-------------|
| High Throughput | ✓✓✓ | ✓✓ | ✓✓ | ✗ |
| Low Latency | ✓✓ | ✓✓ | ✓✓✓ | ✓✓✓ |
| Persistence | ✓✓✓ | ✓✓ | ✓ | ✗ |
| Replay | ✓✓✓ | ✗ | ✗ | ✗ |
| Ordering | ✓✓✓ | ✓✓ | ✓✓ | ✓✓✓ |
| Simplicity | ✗ | ✓✓ | ✓✓✓ | ✓✓✓ |
| Scalability | ✓✓✓ | ✓✓ | ✓✓ | ✓ |

---

## Summary

Apache Kafka is a distributed event streaming platform excelling at high-throughput, fault-tolerant data pipelines. Its log-based architecture, replication mechanisms, and semantic guarantees make it ideal for modern data-driven systems. However, operational complexity and resource overhead require careful evaluation against simpler alternatives for smaller-scale use cases.







