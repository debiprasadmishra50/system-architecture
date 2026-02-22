# Partitioning and Sharding

## Table of Contents
1. [Why Partitioning and Sharding is Required](#why-partitioning-and-sharding-is-required)
2. [Problems Solved](#problems-solved)
3. [Partitioning vs Sharding](#partitioning-vs-sharding)
4. [Sharding in Relational Databases](#sharding-in-relational-databases)
5. [Sharding in Non-Relational Databases](#sharding-in-non-relational-databases)
6. [Sharding Strategies](#sharding-strategies)
7. [Hashing for Uniform Distribution](#hashing-for-uniform-distribution)
8. [Data Hotspots and Shard Failures](#data-hotspots-and-shard-failures)
9. [Architect's Perspective](#architects-perspective)

---

## Why Partitioning and Sharding is Required

### Scalability Limitations
- **Single Database Bottleneck**: A single database server has finite CPU, memory, and disk capacity
- **Read/Write Throughput Limits**: Database can only handle a certain number of queries per second
- **Storage Constraints**: Disk space is limited; cannot store unlimited data on one machine
- **Network Bandwidth**: Network I/O becomes a bottleneck with massive data volumes

### Real-World Scenarios
- **Large-Scale Applications**: Social media platforms (billions of users), e-commerce (millions of products)
- **High-Frequency Operations**: Trading systems, real-time analytics, IoT data ingestion
- **Geographic Distribution**: Serving users across multiple regions with low latency
- **Data Growth**: Applications that grow exponentially over time

---

## Problems Solved

### 1. **Scalability**
- Distribute data across multiple servers to handle larger datasets
- Parallelize queries across shards for faster execution
- Scale horizontally instead of hitting vertical scaling limits

### 2. **Performance**
- Reduce query latency by querying smaller datasets
- Distribute load across multiple machines
- Improve throughput by parallel processing

### 3. **Availability**
- Isolate failures to specific shards
- Replicate each shard independently
- Continue serving requests even if one shard fails

### 4. **Cost Efficiency**
- Use commodity hardware instead of expensive single server
- Distribute costs across multiple machines
- Better resource utilization

---

## Partitioning vs Sharding

| Aspect | Partitioning | Sharding |
|--------|--------------|----------|
| **Scope** | Dividing data within single database | Dividing data across multiple databases/servers |
| **Location** | Same physical server | Different physical servers |
| **Transparency** | Often transparent to application | Application aware of shard logic |
| **Complexity** | Lower complexity | Higher complexity |
| **Use Case** | Performance optimization | Horizontal scalability |
| **Example** | Table partitioned by date in PostgreSQL | User data split across 10 database servers |

---

## Sharding in Relational Databases

### Challenges with Relational Databases

```
┌────────────────────────────────────────────────────────┐
│         RELATIONAL DATABASE SHARDING CHALLENGES        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Shard 1    │  │   Shard 2    │  │   Shard 3    │  │
│  │ (Users 1-M)  │  │ (Users M-N)  │  │ (Users N-Z)  │  │
│  │              │  │              │  │              │  │
│  │ ✓ Queries    │  │ ✓ Queries    │  │ ✓ Queries    │  │
│  │ ✗ Joins      │  │ ✗ Joins      │  │ ✗ Joins      │  │
│  │ ✗ Aggregate  │  │ ✗ Aggregate  │  │ ✗ Aggregate  │  │
│  │ ✗ Unique     │  │ ✗ Unique     │  │ ✗ Unique     │  │
│  │   Constraints│  │   Constraints│  │   Constraints│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### Key Challenges
- **Cross-Shard Joins**: Joining data from multiple shards is expensive
- **Distributed Transactions**: ACID transactions across shards are complex
- **Aggregations**: SUM, COUNT, AVG across shards require post-processing
- **Unique Constraints**: Enforcing uniqueness across shards is difficult
- **Foreign Keys**: Referential integrity across shards is problematic

### Sharding Strategies for Relational Databases

#### 1. **Range-Based Sharding**
```
Shard 1: User IDs 1-1000
Shard 2: User IDs 1001-2000
Shard 3: User IDs 2001-3000

Query: SELECT * FROM users WHERE user_id = 1500
→ Route to Shard 2
```

**Pros**: Simple to implement, easy to understand
**Cons**: Uneven distribution, hotspots possible

#### 2. **Directory-Based Sharding**
```
┌────────────────────────────────────┐
│    Shard Directory/Lookup Table    │
├────────────────────────────────────┤
│ user_id  │  shard_id               │
├──────────┼─────────────────────────┤
│ 1        │  shard_1                │
│ 2        │  shard_3                │
│ 3        │  shard_2                │
│ ...      │  ...                    │
└────────────────────────────────────┘
```

**Pros**: Flexible, can rebalance without rehashing
**Cons**: Lookup overhead, directory becomes bottleneck

#### 3. **Hash-Based Sharding**
```
shard_id = hash(user_id) % num_shards

user_id = 42
hash(42) = 8734
8734 % 3 = 1
→ Route to Shard 1
```

**Pros**: Uniform distribution, no hotspots
**Cons**: Difficult to rebalance when adding shards

---

## Sharding in Non-Relational Databases

### Advantages
- **Built-in Sharding**: Most NoSQL databases have native sharding support
- **No Join Issues**: Document/key-value stores don't require joins
- **Flexible Schema**: Easier to denormalize and avoid cross-shard queries
- **Horizontal Scaling**: Designed for distributed architectures

### Sharding Strategies for NoSQL

#### 1. **MongoDB Sharding**
```
Shard Key: user_id

┌──────────────────────────────────────────────┐
│              Shard Cluster                   │
├──────────────────────────────────────────────┤
│                                              │
│  ┌─────────────────┐  ┌─────────────────┐    │
│  │ Shard 1         │  │ Shard 2         │    │
│  │ user_id: 1-500  │  │ user_id: 501-1K │    │
│  │ {_id, name,     │  │ {_id, name,     │    │
│  │  email, ...}    │  │  email, ...}    │    │
│  └─────────────────┘  └─────────────────┘    │
│                                              │
│  ┌─────────────────┐  ┌─────────────────┐    │
│  │ Shard 3         │  │ Shard 4         │    │
│  │ user_id: 1K-2K  │  │ user_id: 2K-3K  │    │
│  │ {_id, name,     │  │ {_id, name,     │    │
│  │  email, ...}    │  │  email, ...}    │    │
│  └─────────────────┘  └─────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

#### 2. **Cassandra Sharding (Token-Based)**
```
Token Ring: 0 to 2^127-1

Node 1: Tokens 0-100
Node 2: Tokens 101-200
Node 3: Tokens 201-300

Data placement: hash(partition_key) → token → node
```

#### 3. **Redis Cluster Sharding**
```
Hash Slot: 0 to 16383

slot = CRC16(key) % 16384

┌─────────────────────────────────┐
│ Master 1: Slots 0-5460          │
│ Master 2: Slots 5461-10922      │
│ Master 3: Slots 10923-16383     │
└─────────────────────────────────┘
```

---

## Sharding Strategies

### 1. **Range-Based Sharding**
```
Shard Assignment: shard_id = range_lookup(key)

Example: User IDs
Shard 1: 1-1000
Shard 2: 1001-2000
Shard 3: 2001-3000
```

**Best For**: Time-series data, sequential IDs
**Pros**: Simple, easy to understand
**Cons**: Uneven distribution, hotspots

### 2. **Hash-Based Sharding**
```
Shard Assignment: shard_id = hash(key) % num_shards

Example: User ID 42
hash(42) % 3 = 1 → Shard 1
```

**Best For**: Uniform distribution, general purpose
**Pros**: Even distribution, no hotspots
**Cons**: Difficult to rebalance

### 3. **Consistent Hashing**
```
Hash Ring: 0 to 2^32-1

┌─────────────────────────────────────┐
│         Consistent Hash Ring        │
│                                     │
│    Node A (hash: 100)               │
│    ↓                                │
│    ●─────────────────────────●      │
│   /                           \     │
│  ●                             ●    │
│ /                               \   │
│●                                 ●  │
│ \                               /   │
│  ●                             ●    │
│   \                           /     │
│    ●─────────────────────────●      │
│    ↑           ↑           ↑        │
│  Node C      Node B      Node A     │
│  (hash:      (hash:      (hash:     │
│   300)       200)        100)       │
│                                     │
└─────────────────────────────────────┘

Key placement: hash(key) → nearest node clockwise
```

**Best For**: Distributed systems, dynamic node addition/removal
**Pros**: Minimal rebalancing on node changes
**Cons**: More complex implementation

### 4. **Directory-Based Sharding**
```
Lookup Table:
┌────────┬──────────┐
│ Key    │ Shard ID │
├────────┼──────────┤
│ user_1 │ shard_2  │
│ user_2 │ shard_1  │
│ user_3 │ shard_3  │
└────────┴──────────┘

Query: SELECT shard_id FROM directory WHERE key = 'user_1'
→ Shard 2
```

**Best For**: Flexible rebalancing, complex logic
**Pros**: Flexible, can rebalance easily
**Cons**: Lookup overhead, directory bottleneck

### 5. **Geographic Sharding**
```
┌───────────────────────────────────────┐
│         Geographic Sharding           │
├───────────────────────────────────────┤
│                                       │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ US Shard     │  │ EU Shard     │   │
│  │ (Low latency)│  │ (Low latency)│   │
│  └──────────────┘  └──────────────┘   │
│                                       │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ APAC Shard   │  │ LATAM Shard  │   │
│  │ (Low latency)│  │ (Low latency)│   │
│  └──────────────┘  └──────────────┘   │
│                                       │
└───────────────────────────────────────┘
```

**Best For**: Global applications, compliance requirements
**Pros**: Low latency, data residency compliance
**Cons**: Complex replication, uneven load

## **Which Strategy is Best?**

| Strategy | Best For | Trade-offs |
|----------|----------|-----------|
| **Range-Based** | Time-series, sequential data | Simple but uneven distribution |
| **Hash-Based** | General purpose, uniform distribution | Hard to rebalance |
| **Consistent Hashing** | Dynamic clusters, node changes | Complex but minimal rebalancing |
| **Directory-Based** | Flexible logic, easy rebalancing | Lookup overhead |
| **Geographic** | Global apps, compliance | Complex, uneven load |

**Recommendation**: Start with **Hash-Based** for simplicity. Use **Consistent Hashing** if you need dynamic scaling. Use **Directory-Based** if you need flexibility.

---

## Hashing for Uniform Distribution

### Why Hashing Matters

```
┌─────────────────────────────────────────────────────┐
│    WITHOUT HASHING (Range-Based)                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Shard 1: IDs 1-1000      (1000 records)            │
│  Shard 2: IDs 1001-2000   (1000 records)            │
│  Shard 3: IDs 2001-3000   (1000 records)            │
│                                                     │
│  Problem: New users (IDs 3001+) all go to Shard 3   │
│  Result: Shard 3 becomes hotspot!                   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│    WITH HASHING (Hash-Based)                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  hash(1) % 3 = 1 → Shard 1                          │
│  hash(2) % 3 = 2 → Shard 2                          │
│  hash(3) % 3 = 0 → Shard 3                          │
│  hash(4) % 3 = 1 → Shard 1                          │
│  hash(5) % 3 = 2 → Shard 2                          │
│  hash(3001) % 3 = 0 → Shard 3                       │
│  hash(3002) % 3 = 1 → Shard 1                       │
│                                                     │
│  Result: Even distribution across all shards!       │
└─────────────────────────────────────────────────────┘
```

### Hash Function Properties

- **Deterministic**: Same input always produces same output
- **Uniform Distribution**: Hash values spread evenly across range
- **Fast Computation**: Quick to calculate
- **Avalanche Effect**: Small input change produces large output change

### Common Hash Functions

```
1. Modulo Hashing
   shard_id = hash(key) % num_shards
   
   Example: hash("user_42") % 3 = 1 → Shard 1

2. Consistent Hashing
   shard_id = find_node(hash(key)) on hash ring
   
   Example: hash("user_42") = 8734 → Node at position 8734

3. Jump Hash
   shard_id = jump_hash(key, num_shards)
   
   Minimizes rebalancing when shards change
```

### Reducing Data Hotspots

#### Problem: Hotspot Identification
```
┌───────────────────────────────────────────────┐
│         HOTSPOT SCENARIO                      │
├───────────────────────────────────────────────┤
│                                               │
│  Shard 1: 100 GB, 1000 RPS                    │
│  Shard 2: 100 GB, 1000 RPS                    │
│  Shard 3: 100 GB, 5000 RPS  ← HOTSPOT!        │
│                                               │
│  Cause: Popular user (celebrity) in Shard 3   │
│  Impact: Shard 3 CPU/Memory maxed out         │
│                                               │
└───────────────────────────────────────────────┘
```

#### Solution 1: Shard Splitting
```
Before:
Shard 3: hash(key) % 3 = 2

After:
Shard 3a: hash(key) % 6 = 2
Shard 3b: hash(key) % 6 = 5

Redistribute data from Shard 3 to Shard 3a and 3b
```

#### Solution 2: Replica Shards
```
┌───────────────────────────────────────┐
│         REPLICA SHARDS FOR HOTSPOTS   │
├───────────────────────────────────────┤
│                                       │
│  Primary Shard 3                      │
│  ├─ Replica 3a (Read-only)            │
│  ├─ Replica 3b (Read-only)            │
│  └─ Replica 3c (Read-only)            │
│                                       │
│  Distribute reads across replicas     │
│  Writes still go to primary           │
│                                       │
└───────────────────────────────────────┘
```

#### Solution 3: Micro-Sharding
```
Instead of: hash(key) % 3
Use: hash(key) % 300 (100 micro-shards per shard)

Allows fine-grained rebalancing
Move micro-shards between physical shards
```

---

## Data Hotspots and Shard Failures

### Types of Hotspots

#### 1. **Write Hotspots**
```
Scenario: Real-time event tracking for trending topic

Before: hash(event) % 3
Shard 1: 100 writes/sec
Shard 2: 100 writes/sec
Shard 3: 10,000 writes/sec ← HOTSPOT!

Cause: All events for trending topic hash to Shard 3
```

**Solutions**:
- Add random suffix to key: `hash(event + random()) % 3`
- Use time-based bucketing: `hash(event + timestamp_bucket) % 3`
- Implement write-through cache with batching

#### 2. **Read Hotspots**
```
Scenario: Popular user profile (celebrity)

Shard 2 contains celebrity profile
Shard 2 receives 50,000 reads/sec
Shard 1: 1,000 reads/sec
Shard 3: 1,000 reads/sec
```

**Solutions**:
- Replicate hot data across multiple shards
- Use caching layer (Redis, Memcached)
- Implement read replicas for hot shard

#### 3. **Storage Hotspots**
```
Scenario: Uneven data distribution

Shard 1: 50 GB
Shard 2: 50 GB
Shard 3: 500 GB ← HOTSPOT!

Cause: Poor shard key choice or data skew
```

**Solutions**:
- Re-shard with better key
- Use consistent hashing for better distribution
- Implement directory-based sharding for flexibility

### Handling Shard Failures

#### Failure Scenario
```
┌────────────────────────────────────────────┐
│         SHARD FAILURE SCENARIO             │
├────────────────────────────────────────────┤
│                                            │
│  Shard 1: ONLINE  ✓                        │
│  Shard 2: OFFLINE ✗ (Network partition)    │
│  Shard 3: ONLINE  ✓                        │
│                                            │
│  Impact: 33% of data unavailable           │
│          Requests to Shard 2 fail          │
│                                            │
└────────────────────────────────────────────┘
```

#### Solution 1: Replication
```
┌──────────────────────────────────────────┐
│         SHARD REPLICATION                │
├──────────────────────────────────────────┤
│                                          │
│  Primary Shard 2: OFFLINE ✗              │
│  Replica Shard 2a: ONLINE ✓              │
│  Replica Shard 2b: ONLINE ✓              │
│                                          │
│  Failover: Route requests to Replica 2a  │
│  Result: No data loss, minimal downtime  │
│                                          │
└──────────────────────────────────────────┘
```

#### Solution 2: Consistent Hashing with Replicas
```
Hash Ring with Replication Factor = 3

Key "user_42" hashes to position 8734
Replicas placed at:
- Node A (position 8734)
- Node B (next node clockwise)
- Node C (next node after B)

If Node A fails, read from Node B or C
```

#### Solution 3: Automatic Failover
```
┌──────────────────────────────────────────────┐
│         AUTOMATIC FAILOVER PROCESS           │
├──────────────────────────────────────────────┤
│                                              │
│  1. Health Check: Shard 2 not responding     │
│  2. Detection: Marked as OFFLINE             │
│  3. Failover: Promote Replica 2a to Primary  │
│  4. Rebalance: Redistribute load             │
│  5. Recovery: Restore Shard 2, sync data     │
│                                              │
└──────────────────────────────────────────────┘
```

### Monitoring and Prevention

- **Metrics to Track**:
  - Shard size (GB)
  - Queries per second per shard
  - Latency per shard
  - Replication lag
  - Disk usage percentage

- **Alerting Thresholds**:
  - Shard size > 80% of capacity
  - Latency > 2x average
  - Replication lag > 5 seconds
  - Disk usage > 90%

---

## Architect's Perspective

### When to Shard
- **Data Size**: > 100 GB and growing
- **Query Load**: > 10,000 QPS
- **Write Load**: > 5,000 writes/sec
- **Latency Requirements**: < 100ms response time needed

### Sharding Checklist
- [ ] Identify shard key (immutable, high cardinality)
- [ ] Choose sharding strategy (hash-based recommended)
- [ ] Plan for replication (minimum 2 replicas)
- [ ] Implement monitoring and alerting
- [ ] Design failover mechanism
- [ ] Plan for resharding strategy
- [ ] Test failure scenarios
- [ ] Document shard topology

### Common Pitfalls
- **Poor Shard Key**: Leads to hotspots and uneven distribution
- **No Replication**: Single shard failure causes data loss
- **Ignoring Hotspots**: Causes performance degradation
- **Difficult Resharding**: Plan resharding strategy upfront
- **Cross-Shard Queries**: Expensive and slow, avoid if possible

### Best Practices
- **Immutable Shard Key**: Never change shard key after assignment
- **High Cardinality**: Choose key with many unique values
- **Uniform Distribution**: Verify even distribution across shards
- **Replication**: Always replicate shards for fault tolerance
- **Monitoring**: Track shard health continuously
- **Gradual Rollout**: Start with few shards, scale gradually
- **Denormalization**: Avoid cross-shard joins through denormalization

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
