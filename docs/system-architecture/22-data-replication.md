# Data Replication

## Table of Contents
1. [Overview](#overview)
2. [Problem Statement](#problem-statement)
3. [Core Concepts](#core-concepts)
4. [Replication Strategies](#replication-strategies)
5. [Resiliency Benefits](#resiliency-benefits)
6. [Availability Benefits](#availability-benefits)
7. [Latency Benefits](#latency-benefits)
8. [Replication Topologies](#replication-topologies)
9. [Consistency Models](#consistency-models)
10. [Trade-offs](#trade-offs)
11. [Real-World Example](#real-world-example)

---

## Overview

Data Replication is the process of creating and maintaining multiple copies of data across different nodes, databases, or geographic locations. It ensures data is available, resilient, and accessible with minimal latency.

---

## Problem Statement

### Without Replication
- **Single Point of Failure**: One database failure = complete data loss
- **Limited Availability**: Maintenance or crashes cause downtime
- **High Latency**: All requests must travel to single location
- **No Fault Tolerance**: No backup if primary node fails
- **Scalability Bottleneck**: Cannot distribute read load

### With Replication
- Multiple copies ensure data survives failures
- Requests served from nearest replica
- Read load distributed across replicas
- Automatic failover to healthy replicas
- Geographic distribution reduces latency

| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-10%20at%201.15.21 AM.png) | → | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-10%20at%201.15.31 AM.png) |
| :---: | :---: | :---: |

---

## Core Concepts

### Primary (Master)
- Single authoritative copy of data
- Receives all write operations
- Propagates changes to replicas
- Responsible for consistency

### Replica (Slave/Secondary)
- Copy of primary data
- Receives updates from primary
- Serves read operations
- Can become primary if needed

### Replication Lag
- Delay between write on primary and update on replica
- Measured in milliseconds to seconds
- Affects consistency guarantees

### Synchronization
- **Synchronous**: Primary waits for replica acknowledgment before confirming write
- **Asynchronous**: Primary confirms write immediately, updates replicas later

---

## Replication Strategies

### 1. Full Replication
- Complete copy of entire database on each node
- Highest redundancy and availability
- Highest storage cost
- Best for: Small to medium datasets

```
Primary DB (Complete)
    ↓
Replica 1 (Complete)
Replica 2 (Complete)
Replica 3 (Complete)
```

### 2. Partial Replication
- Only critical data replicated
- Reduced storage overhead
- Selective redundancy
- Best for: Large datasets with hot/cold data

```
Primary DB (All data)
    ↓
Replica 1 (Critical data only)
Replica 2 (Critical data only)
```

### 3. Sharded Replication
- Data partitioned across nodes
- Each shard replicated separately
- Combines sharding with replication
- Best for: Massive scale systems

```
Shard 1 Primary → Shard 1 Replica
Shard 2 Primary → Shard 2 Replica
Shard 3 Primary → Shard 3 Replica
```

---

## Resiliency Benefits

### Fault Tolerance
- **Node Failure**: Data survives if one node fails
- **Disk Failure**: Replicas on different disks prevent data loss
- **Corruption**: Multiple copies reduce risk of all copies corrupting

### Automatic Failover
```
Primary fails
    ↓
Health check detects failure
    ↓
Replica promoted to primary
    ↓
Requests redirected to new primary
    ↓
System continues operating
```

### Recovery
- Failed node can be rebuilt from replicas
- No data loss if at least one replica survives
- Faster recovery with multiple replicas

| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-10%20at%201.15.21 AM.png) | → | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-10%20at%201.15.31 AM.png) |
| :---: | :---: | :---: |

### Example Scenario
```
Time 0: Primary DB fails
Time 1: Replica 1 detects failure
Time 2: Replica 1 promoted to primary
Time 3: Application reconnects to new primary
Time 4: Old primary recovered and rejoins as replica
```

---

## Availability Benefits

### Continuous Operation
- System remains available during maintenance
- Primary can be updated without downtime
- Replicas serve requests while primary is down

### Read Scaling
- Distribute read operations across replicas
- Primary handles writes, replicas handle reads
- Increases total system throughput

### Geographic Distribution
- Replicas in different regions
- Local replicas serve local requests
- Reduces dependency on single location

### Load Distribution
```
Write Request → Primary
Read Request 1 → Replica 1
Read Request 2 → Replica 2
Read Request 3 → Replica 3
```

---

## Latency Benefits

### Geographic Proximity
- Users served from nearest replica
- Reduces network round-trip time
- Improves user experience

### Read Latency Reduction
- Replicas closer to users than primary
- Local reads faster than remote reads
- Typical improvement: 50-80% latency reduction

### Example Latency Improvement
```
Without Replication:
User in Tokyo → Primary in US → 150ms latency

With Replication:
User in Tokyo → Replica in Tokyo → 5ms latency
```

### Caching Benefits
- Replicas can be cached locally
- Reduces database load
- Further improves response times

---

## Replication Topologies

### Master-Slave (One-to-Many)
```
        Master
       /  |  \
      /   |   \
   Slave Slave Slave
```
- Single primary, multiple replicas
- Simple, common topology
- Primary is bottleneck for writes

### Master-Master (Multi-Master)
```
    Master 1 ←→ Master 2
      ↓           ↓
    Slave       Slave
```
- Multiple primaries accept writes
- Complex conflict resolution
- Higher availability for writes

### Chain Replication
```
Primary → Replica 1 → Replica 2 → Replica 3
```
- Linear chain of replicas
- Each replica replicates to next
- Reduces primary load

---

## Consistency Models

### Strong Consistency
- All replicas have same data immediately
- Requires synchronous replication
- Higher latency, guaranteed correctness

### Eventual Consistency
- Replicas eventually match primary
- Asynchronous replication
- Lower latency, temporary inconsistency

### Read-After-Write Consistency
- User sees their own writes immediately
- Other users may see stale data
- Balance between consistency and latency

---

## Trade-offs

| Aspect | Benefit | Cost |
|--------|---------|------|
| **Resiliency** | Survives failures | Storage overhead |
| **Availability** | Always accessible | Complexity |
| **Latency** | Faster reads | Replication lag |
| **Consistency** | Data accuracy | Write latency |
| **Scalability** | Distribute load | Coordination overhead |

### Synchronous vs Asynchronous

| Aspect | Synchronous | Asynchronous |
|--------|-------------|--------------|
| **Consistency** | Strong | Eventual |
| **Write Latency** | Higher | Lower |
| **Failure Risk** | Lower | Higher |
| **Complexity** | Lower | Higher |

---

## Real-World Example

### E-Commerce Platform: Order Database

**Scenario**: Online store with users in US, Europe, and Asia

**Without Replication**:
```
All users → Primary DB (US)
- US users: 10ms latency
- Europe users: 100ms latency
- Asia users: 200ms latency
- Single failure = complete outage
```

**With Replication**:
```
US users → Primary DB (US) + Replica (US)
Europe users → Replica (Europe)
Asia users → Replica (Asia)

Latencies:
- US users: 5ms (local replica)
- Europe users: 15ms (local replica)
- Asia users: 20ms (local replica)

Failure Handling:
- US Primary fails → US Replica promoted
- Europe Replica fails → Requests to US Primary
- System continues operating
```

**Benefits Realized**:
- **Resiliency**: Any single node failure doesn't stop system
- **Availability**: 99.99% uptime vs 99% without replication
- **Latency**: 95% reduction in average response time
- **Throughput**: 3x more read capacity with 3 replicas

---

## Implementation Considerations

### Replication Lag Management
- Monitor lag continuously
- Alert if lag exceeds threshold
- Adjust replication strategy if needed

### Conflict Resolution
- Last-write-wins strategy
- Application-level conflict resolution
- Vector clocks for ordering

### Monitoring
- Track replica health
- Monitor replication lag
- Alert on failures
- Log replication events

### Backup Strategy
- Replicas are not backups
- Maintain separate backup copies
- Test recovery procedures regularly

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
