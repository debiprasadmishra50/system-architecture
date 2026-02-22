# Latency

## Table of Contents
1. [Definition and Fundamentals](#definition-and-fundamentals)
2. [Latency vs Availability](#latency-vs-availability)
3. [Why Latency Matters](#why-latency-matters)
   - [User Experience Impact](#user-experience-impact)
   - [Business Impact](#business-impact)
   - [Technical Impact](#technical-impact)
4. [Average Latency vs Percentiles](#average-latency-vs-percentiles)
   - [Why Average is Misleading](#why-average-is-misleading)
   - [Percentile Metrics](#percentile-metrics)
   - [Distribution Example](#distribution-example)
   - [Why Percentiles Matter](#why-percentiles-matter)
5. [Latency Benchmarks by Use Case](#latency-benchmarks-by-use-case)
   - [Real-time Systems](#real-time-systems)
   - [Web Applications](#web-applications)
   - [Mobile Applications](#mobile-applications)
   - [Backend Services](#backend-services)
   - [Critical Infrastructure](#critical-infrastructure)
6. [Impact on System Design](#impact-on-system-design)
   - [Database Choice](#database-choice)
   - [Geographic Distribution](#geographic-distribution)
   - [Caching Strategy](#caching-strategy)
   - [Architecture Style](#architecture-style)
     - [Monolith vs Microservices](#monolith-vs-microservices)
     - [Synchronous vs Asynchronous](#synchronous-vs-asynchronous)
   - [Infrastructure Cost](#infrastructure-cost)
7. [Trade-offs](#trade-offs)
   - [Latency vs Consistency](#latency-vs-consistency)
   - [Latency vs Cost](#latency-vs-cost)
   - [Latency vs Complexity](#latency-vs-complexity)
8. [Cold Caches and Cache Misses](#cold-caches-and-cache-misses)
   - [Problem: Cold Cache Impact](#problem-cold-cache-impact)
   - [Causes of Cold Caches](#causes-of-cold-caches)
   - [Strategies to Minimize Cold Cache Impact](#strategies-to-minimize-cold-cache-impact)
     - [1. Cache Warming](#1-cache-warming)
     - [2. Lazy Loading with Fallback](#2-lazy-loading-with-fallback)
     - [3. Tiered Caching](#3-tiered-caching)
     - [4. Cache Replication](#4-cache-replication)
     - [5. Predictive Loading](#5-predictive-loading)
   - [Cache Invalidation Strategies](#cache-invalidation-strategies)
     - [Challenge: Cache Invalidation](#challenge-cache-invalidation)
     - [1. Time-Based Expiration (TTL)](#1-time-based-expiration-ttl)
     - [2. Event-Based Invalidation](#2-event-based-invalidation)
     - [3. Versioned Keys](#3-versioned-keys)
     - [4. Write-Through Cache](#4-write-through-cache)
     - [5. Write-Behind Cache](#5-write-behind-cache)
     - [6. Cache-Aside Pattern](#6-cache-aside-pattern)
   - [Optimal Strategy for 50ms Latency Systems](#optimal-strategy-for-50ms-latency-systems)

---

## Definition and Fundamentals

**Latency** is the time delay between initiating a request and receiving a response.

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-08 at 11.19.53 PM.png' width='500' />

- **Measured in**: milliseconds (ms), microseconds (μs), or nanoseconds (ns)
- **Types**:
  - **Network Latency**: Time for data to travel across network
  - **Database Latency**: Time for query execution and response
  - **Application Latency**: Time for business logic processing
  - **End-to-End Latency**: Total time from user action to visible result
- **Components**:
  - Request transmission time
  - Processing time
  - Response transmission time
  - Queueing delays

```
User Request → Network → Server Processing → Database → Response → Network → User
    ↓            ↓            ↓                  ↓          ↓         ↓        ↓
  1ms          5ms          10ms               20ms       5ms       5ms     Total: 46ms
```

---

## Latency vs Availability

| Aspect | Latency | Availability |
|--------|---------|--------------|
| **Definition** | Time delay in response | System uptime percentage |
| **Measurement** | Milliseconds | Percentage (99.9%, 99.99%) |
| **Focus** | Performance | Reliability |
| **Impact** | User experience speed | Service accessibility |
| **Example** | 50ms response time | 99.99% uptime (52.6 min/year downtime) |
| **Trade-off** | Faster response vs resource cost | High availability vs complexity |

**Key Difference**: A system can be highly available (always up) but have high latency (slow responses). Conversely, a system can be fast but frequently unavailable.

---

## Why Latency Matters

### User Experience Impact
<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.37.47 PM.png' width='500' />

- **0-100ms**: Feels instant, seamless interaction
- **100-300ms**: Noticeable delay, acceptable for most tasks
- **300-1000ms**: Significant delay, user frustration begins
- **>1000ms**: Unacceptable, users abandon interaction

### Business Impact
<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-12 at 10.52.23 AM.png' width='500' />

- **Amazon**: Every 100ms delay = 1% sales loss
- **Google**: Every 100ms delay = 0.2% traffic loss
- **Bounce Rate**: Increases exponentially with latency
- **Conversion Rate**: Directly correlates with page load time

### Technical Impact
- Cascading failures in microservices
- Timeout errors and retries
- Resource exhaustion under load
- Poor user perception of system reliability

---

## Average Latency vs Percentiles

### Why Average is Misleading

Average latency hides outliers and tail latencies that significantly impact user experience.

```
Scenario: 100 requests
- 99 requests: 10ms
- 1 request: 1000ms
- Average: 19.9ms ✗ (misleading!)
- P99: 1000ms ✓ (reveals the problem)
```

### Percentile Metrics

| Percentile | Meaning | Use Case |
|-----------|---------|----------|
| **P50 (Median)** | 50% of requests faster | Baseline performance |
| **P90** | 90% of requests faster | Most users' experience |
| **P95** | 95% of requests faster | Good performance target |
| **P99** | 99% of requests faster | Tail latency, SLA target |
| **P99.9** | 99.9% of requests faster | Critical systems |
| **Max** | Worst-case latency | Capacity planning |

### Distribution Example

```
P50:   20ms  ████
P90:   50ms  ██████████
P95:   80ms  ████████████
P99:  200ms  ██████████████████
P99.9: 500ms ██████████████████████████
Max:  1000ms ████████████████████████████████
```

### Why Percentiles Matter
- **P99 captures tail latency**: 1% of users experience this delay
- **Cumulative impact**: In high-traffic systems, 1% = millions of users
- **SLA compliance**: Most SLAs define P99 or P99.9 targets
- **Capacity planning**: Max latency determines infrastructure needs

---

## Latency Benchmarks by Use Case

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.35.19 PM.png' width='800' />

### Real-time Systems
- **High-frequency trading**: <1ms (microseconds critical)
- **Online gaming**: 50-100ms (competitive play)
- **Video conferencing**: 100-150ms (acceptable for conversation)
- **Live streaming**: 1-3 seconds (acceptable delay)

### Web Applications
- **Search results**: <100ms
- **E-commerce product page**: <200ms
- **Social media feed**: <300ms
- **Content delivery**: <500ms

### Mobile Applications
- **App launch**: <2 seconds
- **Page navigation**: <1 second
- **API response**: <500ms
- **Database query**: <100ms

### Backend Services
- **Cache hit**: 1-5ms
- **Database query**: 10-50ms
- **API call (same region)**: 10-100ms
- **API call (cross-region)**: 100-500ms
- **Message queue**: 5-50ms

### Critical Infrastructure
- **DNS lookup**: 10-100ms
- **SSL/TLS handshake**: 50-200ms
- **TCP connection**: 10-50ms
- **HTTP request**: 50-200ms

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.38.28 PM.png' width='700' />

---

## Impact on System Design

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.38.56 PM.png' width='700' />

### Database Choice

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.43.24 PM.png' width='700' />

**Low Latency Requirements**:
- **In-memory databases**: Redis, Memcached (1-5ms)
- **NoSQL**: MongoDB, Cassandra (5-20ms)
- **Time-series DB**: InfluxDB, Prometheus (5-15ms)

**Acceptable Latency**:
- **Relational DB**: PostgreSQL, MySQL (10-50ms)
- **Document DB**: DynamoDB, Firestore (20-100ms)

**High Latency Tolerance**:
- **Data warehouses**: Snowflake, BigQuery (seconds to minutes)
- **Batch processing**: Hadoop, Spark (minutes to hours)

### Geographic Distribution

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.41.02 PM.png' width='700' />

```
Single Region (Low Latency)
┌─────────────────────┐
│   User (US-East)    │
│        ↓ 5ms        │
│   Server (US-East)  │
│        ↓ 10ms       │
│   Database (US-East)│
└─────────────────────┘
Total: ~15ms

Multi-Region (High Latency)
┌──────────────────────────────────────┐
│   User (US-East)                     │
│        ↓ 5ms                         │
│   Server (US-East)                   │
│        ↓ 100ms (cross-region)        │
│   Database (EU-West)                 │
│        ↓ 100ms (return)              │
└──────────────────────────────────────┘
Total: ~205ms
```

**Solutions**:
- **CDN**: Cache content globally (reduce network latency)
- **Read replicas**: Distribute read-heavy workloads
- **Edge computing**: Process data closer to users
- **Eventual consistency**: Accept stale data for speed

### Caching Strategy

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.42.37 PM.png' width='700' />

**Cache Hierarchy** (fastest to slowest):
1. **L1 Cache**: In-process memory (1-10μs)
2. **L2 Cache**: Local Redis (1-5ms)
3. **L3 Cache**: Distributed cache (5-20ms)
4. **Database**: Primary source (10-100ms)

**Cache Placement**:
- **Client-side**: Browser cache, local storage
- **CDN**: Edge cache for static content
- **Application**: In-memory cache (Redis, Memcached)
- **Database**: Query result cache, materialized views

### Architecture Style

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.45.18 PM.png' width='700' />

#### Monolith vs Microservices

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Latency** | Lower (in-process calls) | Higher (network calls) |
| **Consistency** | Immediate | Eventual |
| **Scalability** | Limited | High |
| **Complexity** | Lower | Higher |

**Monolith Latency**: Single request = 20ms
```
Request → Auth (2ms) → Business Logic (8ms) → DB (10ms) → Response
Total: 20ms
```

**Microservices Latency**: Same request = 80ms
```
Request → API Gateway (2ms) → Auth Service (10ms) → Business Service (15ms) 
→ DB Service (20ms) → Cache Service (5ms) → Response (5ms)
Total: 57ms (+ network overhead)
```

#### Synchronous vs Asynchronous

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.44.14 PM.png' width='700' />

**Synchronous** (Low latency, blocking):
```
Client → Server → Process → Response
         ↓
      Waits for completion
```
- Latency: 50-200ms
- User sees result immediately
- Blocks resources during processing

**Asynchronous** (High latency, non-blocking):
```
Client → Server → Queue → Response (immediate)
                    ↓
                 Process (background)
                    ↓
                 Notify user (webhook/polling)
```
- Initial response: 5-10ms
- Processing latency: 100ms-minutes
- Resources freed immediately
- Better for long-running tasks

### Infrastructure Cost

**Latency vs Cost Trade-off**:

```
Cost vs Latency Curve

Cost
  ↑
  │     ╱╱╱╱╱╱╱╱╱╱╱
  │   ╱╱╱╱╱╱╱╱╱╱
  │ ╱╱╱╱╱╱╱╱╱
  │╱╱╱╱╱╱╱
  └─────────────────→ Latency
    Optimal Zone
```

**Cost Drivers**:
- **Premium hardware**: SSD vs HDD (10x cost, 10x latency improvement)
- **Geographic distribution**: Multi-region (2-3x cost)
- **Caching layers**: Redis, CDN (20-30% cost increase, 50% latency reduction)
- **Redundancy**: HA setup (2x cost, 99.99% availability)
- **Auto-scaling**: Handle spikes (variable cost, maintain latency)

**Cost Optimization**:
- Use caching to reduce database load
- Implement tiered storage (hot/warm/cold data)
- Compress data in transit
- Use connection pooling
- Batch requests where possible

---

## Trade-offs

### Latency vs Consistency

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.47.07 PM.png' width='700' />

**Strong Consistency** (High latency):
```
Write → Replicate to all nodes → Acknowledge
        (wait for all replicas)
Latency: 100-500ms
```
- All reads see latest data
- Slower writes
- Example: Banking transactions

**Eventual Consistency** (Low latency):
```
Write → Acknowledge → Replicate asynchronously
Latency: 5-10ms
```
- Reads may see stale data
- Fast writes
- Example: Social media likes

**Decision Matrix**:
| Use Case | Consistency | Latency |
|----------|-------------|---------|
| Banking | Strong | Acceptable |
| E-commerce | Strong | Important |
| Social media | Eventual | Critical |
| Analytics | Eventual | Not critical |

### Latency vs Cost

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.47.33 PM.png' width='700' />

**Low Latency (High Cost)**:
- Premium hardware (SSD, high-memory servers)
- Multiple geographic regions
- Extensive caching layers
- Redundancy and failover
- Cost: $10,000+/month for 50ms P99

**High Latency (Low Cost)**:
- Standard hardware
- Single region
- Minimal caching
- Basic redundancy
- Cost: $1,000/month for 500ms P99

**Optimization Strategy**:
- Identify latency-critical paths
- Invest in those specific areas
- Accept higher latency elsewhere
- Use tiered SLAs for different features

### Latency vs Complexity

<img src='../../Resources/17-system-architecture-basics/latency/Screenshot 2026-02-11 at 7.49.03 PM.png' width='700' />

**Simple Architecture** (Higher latency):
```
Client → Monolith → Database
Latency: 50-100ms
Complexity: Low
```

**Complex Architecture** (Lower latency):
```
Client → CDN → API Gateway → Cache → Microservices → Database
Latency: 20-50ms
Complexity: High (monitoring, debugging, deployment)
```

**Complexity Costs**:
- More services to monitor
- Distributed tracing required
- Harder debugging
- More operational overhead
- Higher team expertise needed

**When to Accept Complexity**:
- Latency is critical business requirement
- High traffic justifies operational cost
- Team has expertise to manage it
- ROI from latency improvement is clear

---

## Cold Caches and Cache Misses

### Problem: Cold Cache Impact

**Cold Cache Scenario**:
```
Warm Cache:  50ms (cache hit)
Cold Cache: 500ms (cache miss + DB query)
Difference: 10x latency increase!
```

### Causes of Cold Caches

1. **Cache Eviction**: LRU/LFU removes old entries
2. **Cache Expiration**: TTL expires, data removed
3. **Server Restart**: In-memory cache lost
4. **Deployment**: New instances without cache
5. **Scaling**: New servers added without pre-warming
6. **Data Updates**: Cache invalidation clears entries

### Strategies to Minimize Cold Cache Impact

#### 1. Cache Warming

**Pre-load on Startup**:
```
Server Start → Load hot data into cache → Accept traffic
```
- Load frequently accessed data at startup
- Warm cache during off-peak hours
- Pre-populate before deployment

**Implementation**:
```
// Pseudo-code
function warmCache() {
  const hotData = db.query("SELECT * FROM products WHERE popular=true");
  hotData.forEach(item => cache.set(item.id, item));
}

server.on('startup', warmCache);
```

#### 2. Lazy Loading with Fallback

**Graceful Degradation**:
```
Request → Cache Hit (50ms) → Return
       → Cache Miss (500ms) → DB Query → Cache + Return
       → DB Timeout → Return stale data from backup cache
```

- Serve stale data on cache miss
- Update cache asynchronously
- Prevents cascading failures

#### 3. Tiered Caching

**Multi-layer Cache**:
```
L1: In-process cache (1-5ms)
  ↓ (miss)
L2: Distributed cache (5-20ms)
  ↓ (miss)
L3: Database (50-100ms)
```

- Reduces probability of complete miss
- Faster fallback on L1 miss
- Distributes load

#### 4. Cache Replication

**Replicate Across Instances**:
```
Instance 1 Cache ←→ Instance 2 Cache ←→ Instance 3 Cache
(Sync via pub-sub or gossip protocol)
```

- Survive single instance failure
- Distribute cache load
- Faster recovery

#### 5. Predictive Loading

**Load Before Needed**:
```
User views Product A → Predict views Product B, C, D
                    → Pre-load B, C, D into cache
```

- Anticipate user behavior
- Load related data proactively
- Reduce future cache misses

### Cache Invalidation Strategies

#### Challenge: Cache Invalidation

> "There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

**Problem**: Keep cache fresh without excessive invalidation

#### 1. Time-Based Expiration (TTL)

**Simple but Stale**:
```
Set cache with TTL = 5 minutes
After 5 min → Automatic expiration
Next request → Cache miss → DB query
```

**Pros**: Simple, no coordination needed
**Cons**: Data can be stale, cache misses after expiration

**Best for**: Non-critical data, acceptable staleness

#### 2. Event-Based Invalidation

**Invalidate on Change**:
```
Data Update → Publish event → Cache listener → Invalidate entry
```

**Pros**: Immediate consistency, no stale data
**Cons**: Complex, requires event infrastructure

**Implementation**:
```
// Pseudo-code
function updateProduct(id, data) {
  db.update('products', id, data);
  eventBus.publish('product.updated', { id, data });
}

cache.on('product.updated', (event) => {
  cache.delete(`product:${event.id}`);
});
```

#### 3. Versioned Keys

**Avoid Invalidation**:
```
Instead of: cache.set('product:123', data)
Use:        cache.set('product:123:v2', data)
```

- Never invalidate, just create new version
- Old version expires naturally
- No coordination needed

**Pros**: No invalidation overhead, simple
**Cons**: Memory overhead, version management

#### 4. Write-Through Cache

**Update Cache with Data**:
```
Write Request → Update Cache → Update Database
                (synchronous)
```

- Cache always has latest data
- Slower writes (wait for cache update)
- Consistent reads

#### 5. Write-Behind Cache

**Update Cache First**:
```
Write Request → Update Cache → Acknowledge
                    ↓
              Update Database (async)
```

- Fast writes (cache updated immediately)
- Risk of data loss if cache fails
- Eventual consistency

#### 6. Cache-Aside Pattern

**Lazy Loading**:
```
Read Request → Check Cache
            → Hit: Return
            → Miss: Query DB → Update Cache → Return
```

- Simple to implement
- Cache miss penalty
- Works well with TTL

### Optimal Strategy for 50ms Latency Systems

**Combination Approach**:

```
1. Cache Warming (startup)
   ↓
2. Tiered Caching (L1 + L2)
   ↓
3. Event-Based Invalidation (immediate consistency)
   ↓
4. Versioned Keys (fallback)
   ↓
5. Lazy Loading (miss handling)
```

**Implementation Example**:
```
// Pseudo-code for 50ms latency system

class LatencyOptimizedCache {
  constructor() {
    this.l1 = new InProcessCache();      // 1-5ms
    this.l2 = new RedisCache();          // 5-20ms
    this.db = new Database();            // 50-100ms
  }

  async get(key) {
    // L1 check
    let value = this.l1.get(key);
    if (value) return value;              // 1-5ms

    // L2 check
    value = await this.l2.get(key);
    if (value) {
      this.l1.set(key, value);            // Populate L1
      return value;                       // 5-20ms
    }

    // Database query
    value = await this.db.query(key);
    if (value) {
      this.l1.set(key, value);            // Populate both
      await this.l2.set(key, value);
      return value;                       // 50-100ms
    }

    return null;
  }

  async set(key, value) {
    // Update both caches
    this.l1.set(key, value);
    await this.l2.set(key, value);
    
    // Update database asynchronously
    this.db.update(key, value).catch(err => {
      logger.error('DB update failed', err);
      // Retry logic
    });

    // Publish invalidation event
    eventBus.publish('cache.updated', { key, value });
  }

  // Warm cache on startup
  async warmCache() {
    const hotKeys = await this.db.getHotData();
    for (const key of hotKeys) {
      const value = await this.db.query(key);
      this.l1.set(key, value);
      await this.l2.set(key, value);
    }
  }
}
```

**Latency Breakdown**:
- Cache hit (L1): 5ms
- Cache hit (L2): 20ms
- Cache miss: 100ms (acceptable for 50ms P99 system with 99% hit rate)

**Key Metrics to Monitor**:
- Cache hit rate (target: >95%)
- P99 latency (target: <50ms)
- Cache eviction rate
- Cold start latency
- Invalidation frequency

---

## Summary

**Latency is critical** for user experience and business metrics. Key takeaways:

- **Measure percentiles**, not averages (P99 > average)
- **Design for your use case**: Different systems need different latency targets
- **Cache strategically**: Warm caches, tiered approach, proper invalidation
- **Accept trade-offs**: Consistency, cost, and complexity vs latency
- **Monitor continuously**: Track P50, P90, P95, P99, and max latency
- **Optimize iteratively**: Identify bottlenecks, fix highest-impact issues first

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
