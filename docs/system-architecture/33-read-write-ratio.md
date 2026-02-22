# Read/Write Ratio

## Table of Contents

1. [What is Read/Write Ratio](#what-is-readwrite-ratio)
2. [Why is it Important](#why-is-it-important)
3. [Read vs Write Ratio](#read-vs-write-ratio)
   - [Read Heavy (Read >> Write)](#read-heavy-read--write)
   - [Balanced (Read ≈ Write)](#balanced-read--write)
   - [Write Heavy (Write >> Read)](#write-heavy-write--read)
4. [How Databases Store Data on Disk](#how-databases-store-data-on-disk)
   - [B-Trees](#b-trees)
   - [LSM Trees (Log-Structured Merge Trees)](#lsm-trees-log-structured-merge-trees)
   - [Comparison Table](#comparison-table)
5. [Caching Strategy: Read vs Write Heavy Systems](#caching-strategy-read-vs-write-heavy-systems)
   - [Read Heavy Systems Optimization Strategies](#read-heavy-systems-optimization-strategies)
     - [Aggressive Caching](#aggressive-caching)
     - [Read Replicas](#read-replicas)
     - [Eventual Consistency](#eventual-consistency)
     - [Denormalization](#denormalization)
     - [Fanout on Write](#fanout-on-write)
   - [Write Heavy Systems Optimization Strategies](#write-heavy-systems-optimization-strategies)
     - [Choose LSM Tree Database Engine](#choose-lsm-tree-database-engine)
     - [Batching and Buffering](#batching-and-buffering)
     - [Stay Normalized](#stay-normalized)
     - [Selective Indexing](#selective-indexing)
     - [Fan-out on Read](#fan-out-on-read)
6. [The Write-Heavy Caching Problem](#the-write-heavy-caching-problem)
   - [The Challenge](#the-challenge)
   - [Solutions](#solutions)
     - [1. Write-Through Cache](#1-write-through-cache)
     - [2. Write-Behind Cache](#2-write-behind-cache)
     - [3. Cache Bypass for Writes](#3-cache-bypass-for-writes)
     - [4. Event-Driven Cache Updates](#4-event-driven-cache-updates)
7. [Measuring Read-Write Ratio](#measuring-read-write-ratio)
   - [Metrics Collection](#metrics-collection)
   - [Tools and Methods](#tools-and-methods)
     - [1. Database Monitoring](#1-database-monitoring)
     - [2. Application Instrumentation](#2-application-instrumentation)
     - [3. Load Testing](#3-load-testing)
   - [Interactive Ratio Calculator](#interactive-ratio-calculator)
8. [Summary](#summary)

---

## What is Read/Write Ratio

The read/write ratio is the proportion of read operations to write operations in a system over a given time period.

**Formula**: `Read/Write Ratio = Total Read Operations / Total Write Operations`

**Example**: A ratio of 100:1 means for every 1 write operation, there are 100 read operations.

### Key Metrics

- **Read Operations**: SELECT queries, GET requests, cache lookups
- **Write Operations**: INSERT, UPDATE, DELETE queries, POST/PUT requests
- **Measurement Period**: Typically measured per second, minute, or hour depending on system scale

---

## Why is it Important

### Impact on System Design

- **Database Selection**: Different databases optimize for different ratios (B-trees for reads, LSM trees for writes)
- **Caching Strategy**: Read-heavy systems benefit from aggressive caching; write-heavy systems need different approaches
- **Infrastructure Scaling**: Determines whether to scale read replicas or optimize write throughput
- **Cost Optimization**: Influences choice of storage engines and replication strategies

### Business Implications

- **User Experience**: Read-heavy systems can serve cached data faster
- **Data Consistency**: Write-heavy systems face eventual consistency challenges
- **Resource Allocation**: Guides investment in caching layers vs. write optimization

---

## Read vs Write Ratio

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 12.36.20 AM.png' width=800 />

### Read Heavy (Read >> Write)

**Characteristics**: Ratio > 10:1

- Most operations are reads (e.g., 99% reads, 1% writes)
- Examples: Social media feeds, news sites, analytics dashboards, search engines
- Data changes infrequently but accessed frequently

**Optimization Focus**:
- Aggressive caching (Redis, Memcached)
- Read replicas for horizontal scaling
- Denormalization for faster queries
- Eventual consistency acceptable

**Example Systems**:
- Twitter feed (billions of reads, fewer writes)
- YouTube video metadata (millions of views, few uploads)
- Google Search (trillions of reads, periodic index updates)

### Balanced (Read ≈ Write)

**Characteristics**: Ratio 1:1 to 10:1

- Moderate mix of reads and writes
- Examples: E-commerce platforms, banking systems, collaborative tools
- Data consistency and performance both critical

**Optimization Focus**:
- Moderate caching with cache invalidation
- Normalized schema for consistency
- Selective indexing
- Strong consistency requirements

**Example Systems**:
- E-commerce checkout (reads product info, writes orders)
- Banking transactions (reads account balance, writes transactions)
- Collaborative documents (reads content, writes edits)

### Write Heavy (Write >> Read)

**Characteristics**: Ratio < 1:1

- Most operations are writes (e.g., 70% writes, 30% reads)
- Examples: Time-series databases, IoT sensors, logging systems, event streams
- Data written frequently, read less often

**Optimization Focus**:
- LSM tree databases for sequential writes
- Batching and buffering writes
- Selective indexing (avoid over-indexing)
- Normalized schema to reduce write complexity

**Example Systems**:
- InfluxDB for metrics (millions of writes per second)
- Kafka for event streaming (high write throughput)
- CloudWatch logs (continuous log ingestion)
- IoT sensor data (constant stream of measurements)

---

## How Databases Store Data on Disk

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 12.54.05 AM.png' width=700 />

### B-Trees

**Structure**: Balanced tree where each node can have multiple children

```
        [30, 70]
       /    |    \
    [10]  [40]  [80]
```

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 10.21.52 AM.png' width=700 />

**Characteristics**:
- Self-balancing tree structure
- All leaf nodes at same depth
- Each node stores multiple keys and values
- Optimized for disk I/O (reads entire node at once)

**Write Process**:
1. Find correct leaf node
2. Insert key-value pair
3. If node overflows, split node
4. Propagate changes up the tree

**Read Process**:
1. Start at root
2. Binary search within node
3. Follow pointer to child node
4. Repeat until leaf found

**Pros and Cons**:

| Aspect | Pros | Cons |
|--------|------|------|
| **Read Performance** | Excellent - O(log n) with few disk seeks | - |
| **Write Performance** | Moderate - requires tree rebalancing | Slower than LSM for sequential writes |
| **Space Efficiency** | Good - compact storage | More overhead than LSM |
| **Cache Friendly** | Excellent - locality of reference | - |
| **Disk I/O** | Optimized for random access | Multiple seeks per operation |
| **Compaction** | Minimal overhead | - |

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 10.22.25 AM.png' width=700 />

**Used By**:
- PostgreSQL (default)
- MySQL InnoDB
- SQLite
- Most traditional SQL databases

---

### LSM Trees (Log-Structured Merge Trees)

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 10.25.16 AM.png' width=700 />

**Structure**: Multiple levels of sorted tables, with writes going to in-memory buffer first

```
Write Buffer (MemTable)
        ↓
    Level 0 (SSTables)
        ↓
    Level 1 (SSTables)
        ↓
    Level 2 (SSTables)
```

**Characteristics**:
- Writes go to in-memory buffer (MemTable) first
- When buffer fills, flushed to disk as SSTable (Sorted String Table)
- Multiple levels of SSTables, periodically merged (compaction)
- Optimized for sequential writes

**Write Process**:
1. Write to in-memory MemTable (fast)
2. When MemTable full, flush to Level 0 SSTable
3. Background compaction merges SSTables
4. Data moves down levels

**Read Process**:
1. Check MemTable first
2. Check Level 0 SSTables
3. Check Level 1, 2, etc. (binary search)
4. May need to check multiple levels

**Pros and Cons**:

| Aspect | Pros | Cons |
|--------|------|------|
| **Write Performance** | Excellent - sequential writes | - |
| **Read Performance** | Good but slower than B-tree | May check multiple levels |
| **Throughput** | Very high write throughput | - |
| **Compaction** | Efficient background process | CPU/IO overhead during compaction |
| **Space Efficiency** | Good with compression | Temporary space during compaction |
| **Disk I/O** | Sequential writes (fast) | Random reads (slower) |

**Used By**:
- ScyllaDB
- RocksDB
- LevelDB
- Cassandra
- HBase
- DynamoDB (configurable)
- InfluxDB

---

### Comparison Table

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 12.38.28 AM.png' width=700 />

| Feature | B-Tree | LSM Tree |
|---------|--------|----------|
| **Write Speed** | Moderate | Very Fast |
| **Read Speed** | Very Fast | Fast |
| **Best For** | Read-heavy workloads | Write-heavy workloads |
| **Disk I/O Pattern** | Random access | Sequential writes |
| **Space Overhead** | Lower | Higher (during compaction) |
| **Compaction** | Minimal | Continuous background |
| **Latency Consistency** | Predictable | Variable (compaction spikes) |
| **Memory Usage** | Lower | Higher (MemTable) |

---

## Caching Strategy: Read vs Write Heavy Systems

### Read Heavy Systems Optimization Strategies

<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 12.40.28 AM.png' width=450 />

**Strategies**
<img src='../../Resources/17-system-architecture-basics/read-write-ratio/Screenshot 2026-02-13 at 12.50.47 AM.png' width=700 />

1. [Aggresive Caching](#aggressive-caching)
2. [Read Replicas](#read-replicas)
3. [Eventual Consistency](#eventual-consistency)
4. [Denormalization](#denormalization)
5. [Fanout on Write](#fanout-on-write)

---

#### Aggressive Caching

**Strategy**: Cache frequently accessed data in memory

```
User Request
    ↓
Cache Layer (Redis/Memcached)
    ↓ (miss)
Database
    ↓
Update Cache
    ↓
Return to User
```

**Implementation**:
- Cache-aside pattern: Check cache first, fetch from DB on miss
- TTL (Time-To-Live) for automatic expiration
- Cache warming for predictable access patterns

**Tools**: Redis, Memcached, Varnish

**Example**: E-commerce product catalog cached for 1 hour

---

#### Read Replicas

**Strategy**: Distribute read load across multiple database replicas

```
Write Master
    ↓
Replication
    ↓
Read Replica 1, Read Replica 2, Read Replica 3
```

**Benefits**:
- Horizontal scaling of read capacity
- Geographic distribution for lower latency
- Failover capability

**Trade-offs**:
- Replication lag (eventual consistency)
- Increased storage costs
- Complexity in managing replicas

**Example**: MySQL with 3 read replicas for 100:1 read/write ratio

---

#### Eventual Consistency

**Strategy**: Accept temporary inconsistency for better performance

**Model**:
- Writes go to master immediately
- Reads may see stale data from replicas
- Data eventually becomes consistent

**When Acceptable**:
- Social media feeds (slight delay acceptable)
- Analytics dashboards (hourly updates fine)
- Caching layers (TTL-based consistency)

**When Not Acceptable**:
- Financial transactions
- Inventory management
- User authentication

---

#### Denormalization

**Strategy**: Store redundant data to avoid expensive joins

**Before (Normalized)**:
```
Users Table: id, name, email
Orders Table: id, user_id, amount
```

**After (Denormalized)**:
```
Orders Table: id, user_id, user_name, user_email, amount
```

**Benefits**:
- Fewer joins = faster queries
- Reduced database load
- Better cache hit rates

**Costs**:
- Data redundancy
- Complex updates (update multiple places)
- Increased storage

---

#### Fanout on Write

**Strategy**: Pre-compute and distribute data on write

```
User publishes post
    ↓
Fanout Service
    ↓
Write to followers' feed caches
    ↓
Followers see post immediately
```

**Use Case**: Social media feeds

**Benefits**:
- Instant read performance
- No computation on read

**Costs**:
- Write latency increases
- Storage for each follower's feed
- Expensive for users with millions of followers

---

### Write Heavy Systems Optimization Strategies

1. [Choose LSM Tree Database Engine](#choose-lsm-tree-database-engine)
2. [Batching and Buffering](#batching-and-buffering)
3. [Stay Normalized](#stay-normalized)
4. [Selective Indexing](#selective-indexing)
5. [Fan-out on Read](#fan-out-on-read)

---


#### Choose LSM Tree Database Engine

**Strategy**: Use databases optimized for sequential writes

**Databases**:
- RocksDB: Embedded key-value store
- Cassandra: Distributed LSM database
- HBase: Hadoop-based LSM database
- InfluxDB: Time-series with LSM

**Benefits**:
- High write throughput
- Efficient disk usage
- Built-in compaction

---

#### Batching and Buffering

**Strategy**: Accumulate writes and process in batches

```
Individual Writes
    ↓
Buffer (in-memory)
    ↓
Batch Write (every 100ms or 1000 items)
    ↓
Database
```

**Benefits**:
- Reduced I/O operations
- Better throughput
- Lower latency per operation

**Example**: Batch 1000 log entries before writing to disk

---

#### Stay Normalized

**Strategy**: Keep schema normalized to simplify writes

**Benefits**:
- Single write location per entity
- Easier consistency maintenance
- Simpler update logic

**Trade-off**:
- Reads require joins (acceptable if reads are less frequent)

---

#### Selective Indexing

**Strategy**: Index only frequently queried columns

**Approach**:
- Avoid indexing write-heavy columns
- Index only read-heavy columns
- Use composite indexes strategically

**Benefits**:
- Reduced write overhead (fewer indexes to update)
- Faster writes
- Lower storage

**Example**: Index user_id and created_at, but not status (frequently updated)

---

#### Fan-out on Read

**Strategy**: Compute aggregations on read, not write

```
Raw Data (normalized)
    ↓
User Query
    ↓
Compute Aggregation
    ↓
Return Result
```

**Benefits**:
- Fast writes (no pre-computation)
- Flexible read patterns
- Lower write complexity

**Costs**:
- Read latency higher
- Repeated computation
- More CPU on read path

**Example**: Calculate user statistics on-demand instead of updating counters on every action

---

## The Write-Heavy Caching Problem

### The Challenge

In write-heavy systems, traditional caching becomes problematic:

```
Problem Scenario:
Write → Cache Invalidation → Stale Data Risk
```

**Issues**:
1. **Cache Invalidation Complexity**: Every write requires cache updates
2. **Stale Data**: Invalidation lag causes inconsistency
3. **Write Amplification**: Each write hits both cache and database
4. **Thundering Herd**: Cache miss after invalidation causes spike

### Solutions

#### 1. Write-Through Cache

```
Write Request
    ↓
Update Cache
    ↓
Update Database
    ↓
Acknowledge
```

**Pros**: Consistency guaranteed
**Cons**: Write latency increases

#### 2. Write-Behind Cache

```
Write Request
    ↓
Update Cache
    ↓
Acknowledge (async DB write)
    ↓
Background: Update Database
```

**Pros**: Fast writes
**Cons**: Data loss risk if cache fails

#### 3. Cache Bypass for Writes

```
Write Request
    ↓
Update Database Only
    ↓
Invalidate Cache
    ↓
Next Read: Fetch from DB
```

**Pros**: Simplicity, consistency
**Cons**: Cache miss on next read

#### 4. Event-Driven Cache Updates

```
Write to Database
    ↓
Emit Event
    ↓
Cache Service Listens
    ↓
Update Cache
```

**Pros**: Decoupled, scalable
**Cons**: Eventual consistency

---

## Measuring Read-Write Ratio

### Metrics Collection

**Application Level**:
```
Total Reads = SELECT + GET + CACHE_HIT + CACHE_MISS
Total Writes = INSERT + UPDATE + DELETE + POST + PUT
Ratio = Total Reads / Total Writes
```

**Database Level**:
- Query logs
- Performance schema (MySQL)
- pg_stat_statements (PostgreSQL)
- CloudWatch metrics (AWS)

### Tools and Methods

#### 1. Database Monitoring

**MySQL**:
```sql
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Com_select';
SHOW STATUS LIKE 'Com_insert';
```

**PostgreSQL**:
```sql
SELECT query, calls FROM pg_stat_statements 
ORDER BY calls DESC;
```

#### 2. Application Instrumentation

**Logging**:
```
[2026-02-13 10:30:45] READ: user_profile (cache_hit)
[2026-02-13 10:30:46] WRITE: order_created
[2026-02-13 10:30:47] READ: product_list (cache_miss)
```

**Metrics**:
- Prometheus counters for read/write operations
- Grafana dashboards for visualization
- CloudWatch custom metrics

#### 3. Load Testing

**Tools**:
- Apache JMeter
- Locust
- k6

**Approach**:
- Simulate realistic user behavior
- Measure read/write distribution
- Identify bottlenecks

### Interactive Ratio Calculator

**How to Use**:

1. **Measure Operations**: Count reads and writes over 1 hour
2. **Calculate Ratio**: `Reads / Writes`
3. **Determine Category**:
   - Ratio > 10 → Read-Heavy
   - Ratio 1-10 → Balanced
   - Ratio < 1 → Write-Heavy

**Example Calculation**:

| Metric | Value |
|--------|-------|
| Total Reads (1 hour) | 360,000 |
| Total Writes (1 hour) | 3,600 |
| **Read/Write Ratio** | **100:1** |
| **Category** | **Read-Heavy** |

**Recommended Actions**:
- Implement aggressive caching (Redis)
- Deploy read replicas
- Consider denormalization
- Use CDN for static content

---

## Summary

| Aspect | Read-Heavy | Balanced | Write-Heavy |
|--------|-----------|----------|------------|
| **Ratio** | > 10:1 | 1-10:1 | < 1:1 |
| **Database** | B-tree (PostgreSQL) | B-tree (MySQL) | LSM (RocksDB) |
| **Caching** | Aggressive | Moderate | Selective |
| **Replication** | Read replicas | Master-slave | Write optimization |
| **Consistency** | Eventual OK | Strong preferred | Strong required |
| **Example** | Twitter feed | E-commerce | Time-series DB |

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
