# Scalability

## Table of Contents
1. [Why Scalability Matters](#why-scalability-matters)
2. [Dimensions of Scale](#dimensions-of-scale)
   - [User Scale](#user-scale)
   - [Data Scale](#data-scale)
   - [Request Scale](#request-scale)
   - [Growth Rate](#growth-rate)
3. [How Scale Changes the Architecture](#how-scale-changes-the-architecture)
   - [Database](#database)
   - [Caching](#caching)
   - [Load Balancing](#load-balancing)
   - [Data Storage](#data-storage)
   - [Async Processing](#async-processing)
4. [Fundamental Scaling Patterns](#fundamental-scaling-patterns)
   - [Vertical Scaling (Scale Up)](#vertical-scaling-scale-up)
   - [Horizontal Scaling (Scale Out)](#horizontal-scaling-scale-out)
5. [When to Choose What](#when-to-choose-what)
    - [Choose Vertical Scaling When](#choose-vertical-scaling-when)
    - [Choose Horizontal Scaling When](#choose-horizontal-scaling-when)
    - [Hybrid Approach](#hybrid-approach)
6. [Scaling Journey Example](#scaling-journey-example)


---

## Why Scalability Matters

<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.25.34 AM.png' width='700' />

- **Business Growth**: Systems must handle increasing users, data, and traffic without degradation
- **Cost Efficiency**: Proper scaling prevents over-provisioning and reduces operational costs
- **User Experience**: Maintains performance and reliability as demand grows
- **Competitive Advantage**: Ability to scale quickly enables rapid market expansion
- **System Reliability**: Distributed scaling improves fault tolerance and availability
- **Future-Proofing**: Anticipating scale prevents costly architectural rewrites

---

## Dimensions of Scale

<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.32.32 AM.png' width='700' />

### User Scale
- **Definition**: Number of concurrent and total users accessing the system
- **Impact**: Increases request volume, connection management complexity, and session state
- **Challenges**: Connection pooling, session management, authentication bottlenecks
- **Example**: 100 users → 1M users requires different architecture

### Data Scale
- **Definition**: Volume of data stored and processed by the system
- **Impact**: Storage capacity, query performance, backup/recovery complexity
- **Challenges**: Database indexing, partitioning, memory constraints
- **Example**: 1GB → 1TB dataset requires sharding and distributed storage

### Request Scale
- **Definition**: Number of requests per second (RPS) or throughput
- **Impact**: Processing capacity, I/O bandwidth, network utilization
- **Challenges**: Request queuing, rate limiting, load distribution
- **Example**: 100 RPS → 100K RPS requires load balancing and async processing

### Growth Rate
- **Definition**: Speed at which scale increases over time
- **Impact**: Time available for architectural changes and optimization
- **Challenges**: Rapid growth leaves little time for refactoring
- **Example**: Doubling users every month vs. every year requires different strategies

---

## Things to keep in mind while scaling

<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.34.46 AM.png' width='700' />

---

## How Scale Changes the Architecture

<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.40.54 AM.png' width='700' />
<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.43.43 AM.png' width='350' />

### Database
- **Single Database**: Suitable for small scale; becomes bottleneck at scale
- **Read Replicas**: Distribute read traffic across multiple database instances
- **Sharding**: Partition data horizontally across multiple databases by key (user ID, region, etc.)
- **NoSQL**: Horizontal scalability for unstructured data and high throughput
- **Connection Pooling**: Manage limited database connections efficiently

```
Small Scale:          Medium Scale:         Large Scale:
┌─────────┐           ┌─────────┐          ┌──────────┐
│ Database│           │ Primary │          │ Shard 1  │
└─────────┘           └────┬────┘          └──────────┘
                           │               ┌──────────┐
                      ┌────┴────┐          │ Shard 2  │
                      │ Replica │          └──────────┘
                      └─────────┘          ┌──────────┐
                                           │ Shard 3  │
                                           └──────────┘
```

### Caching
- **In-Memory Cache**: Redis/Memcached for frequently accessed data
- **Cache Layers**: Application-level, database-level, CDN-level caching
- **Cache Invalidation**: Strategies to keep cache fresh (TTL, event-based)
- **Distributed Cache**: Shared cache across multiple servers
- **Cache Stampede Prevention**: Prevent thundering herd when cache expires

```
Request Flow with Caching:
┌────────┐
│ Client │
└───┬────┘
    │
    ▼
┌──────────────┐
│ CDN Cache    │ ◄─── Geo-distributed
└──────┬───────┘
       │ (miss)
       ▼
┌──────────────┐
│ App Cache    │ ◄─── Redis/Memcached
└──────┬───────┘
       │ (miss)
       ▼
┌──────────────┐
│ Database     │
└──────────────┘
```

### Load Balancing
- **Distribute Traffic**: Spread requests across multiple servers
- **Health Checks**: Route away from unhealthy instances
- **Session Affinity**: Sticky sessions for stateful applications
- **Algorithms**: Round-robin, least connections, weighted, IP hash
- **Geographic Distribution**: Route users to nearest data center

```
┌─────────────────────────────────────┐
│      Load Balancer                  │
│  (Round-robin, Least Conn, etc.)    │
└────────────┬────────────────────────┘
             │
    ┌────────┼────────┬────────┐
    ▼        ▼        ▼        ▼
┌────────┐┌────────┐┌────────┐┌────────┐
│Server 1││Server 2││Server 3││Server 4│
└────────┘└────────┘└────────┘└────────┘
```

### Data Storage
- **Blob Storage**: S3, GCS for large files and media
- **Data Warehouse**: Separate analytics from operational databases
- **Message Queues**: Decouple producers and consumers
- **Time-Series DB**: InfluxDB, Prometheus for metrics and logs
- **Search Engines**: Elasticsearch for full-text search at scale

### Async Processing
- **Job Queues**: Offload long-running tasks (email, image processing)
- **Event-Driven**: Decouple services via events instead of synchronous calls
- **Background Workers**: Process tasks asynchronously with retry logic
- **Message Brokers**: RabbitMQ, Kafka for reliable message delivery
- **Rate Limiting**: Prevent queue overflow and resource exhaustion

```
Sync vs Async:

Sync (Blocking):
Client ──► Server ──► Task ──► Response
         (waits)

Async (Non-blocking):
Client ──► Server ──► Queue ──► Worker ──► Callback
         (returns immediately)
```

---

## Fundamental Scaling Patterns

<img src='../../Resources/17-system-architecture-basics/scalability/Screenshot 2026-02-12 at 11.44.44 AM.png' width='700' />

### Vertical Scaling (Scale Up)

**Definition**: Increase capacity of existing servers by adding more CPU, RAM, or storage

**Characteristics**:
- Single machine becomes more powerful
- Simpler to implement initially
- No code changes required
- Limited by hardware constraints

**Pros**:
- Simple to implement
- No distributed system complexity
- Lower operational overhead
- Easier debugging and monitoring

**Cons**:
- Hardware limits (can't scale infinitely)
- Single point of failure
- Downtime during upgrades
- Expensive at large scale
- Doesn't improve fault tolerance

**Example**:
```
Before:  Server (8 CPU, 16GB RAM)
After:   Server (32 CPU, 128GB RAM)
```

### Horizontal Scaling (Scale Out)

**Definition**: Increase capacity by adding more servers/instances to distribute load

**Characteristics**:
- Multiple machines work together
- Requires load balancing
- Distributed system complexity
- Theoretically unlimited scalability

**Pros**:
- Unlimited scalability
- Better fault tolerance (one server failure doesn't crash system)
- No downtime for upgrades (rolling deployments)
- Cost-effective at large scale
- Improved availability and redundancy

**Cons**:
- Increased complexity (distributed systems challenges)
- Data consistency issues
- Network latency between servers
- Operational overhead (monitoring, deployment)
- Debugging becomes harder

**Example**:
```
Before:  1 Server (8 CPU, 16GB RAM)
After:   4 Servers (2 CPU, 4GB RAM each)
```

---

## When to Choose What

### Choose Vertical Scaling When:
- **Early Stage**: Small user base, simple architecture
- **Predictable Growth**: Gradual, manageable increase in load
- **Stateful Applications**: Hard to distribute (legacy systems)
- **Cost Constraints**: Limited budget for infrastructure
- **Simplicity Priority**: Team lacks distributed systems expertise
- **Latency Sensitive**: Single machine reduces network overhead

### Choose Horizontal Scaling When:
- **Rapid Growth**: Exponential user or traffic increase
- **High Availability Required**: Cannot afford downtime
- **Geographic Distribution**: Users spread across regions
- **Fault Tolerance Critical**: System must survive server failures
- **Cost Optimization**: Cheaper commodity hardware vs. expensive servers
- **Microservices Architecture**: Already distributed system design

### Hybrid Approach (Recommended):
- **Combine Both**: Use vertical scaling for initial growth, then horizontal
- **Optimal Balance**: Scale vertically until cost/complexity inflection point
- **Staged Growth**: 
  1. Start with single powerful server (vertical)
  2. Add read replicas and caching (vertical + simple horizontal)
  3. Shard database and add load balancers (full horizontal)
  4. Microservices and distributed architecture (advanced horizontal)

**Decision Matrix**:

| Factor | Vertical | Horizontal |
|--------|----------|-----------|
| Scalability Limit | Hardware ceiling | Theoretically unlimited |
| Complexity | Low | High |
| Cost at Scale | Expensive | Economical |
| Availability | Single point of failure | Fault tolerant |
| Deployment | Requires downtime | Zero-downtime possible |
| Data Consistency | Easier | Challenging |
| Team Expertise | Low requirement | High requirement |

---

## Scaling Journey Example

```
Stage 1: Single Server (Vertical)
┌──────────────────────┐
│ Web + App + Database │
└──────────────────────┘

Stage 2: Separate Database (Vertical + Simple Horizontal)
┌──────────────┐  ┌──────────────┐
│ Web + App    │  │ Database     │
└──────────────┘  └──────────────┘

Stage 3: Load Balancing (Horizontal)
        ┌──────────────┐
        │ Load Balancer│
        └──────┬───────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│App 1   │ │App 2   │ │App 3   │
└────────┘ └────────┘ └────────┘
               │
        ┌──────┴──────┐
        ▼             ▼
    ┌────────┐   ┌────────┐
    │Primary │   │Replica │
    │Database│   │Database│
    └────────┘   └────────┘

Stage 4: Sharding (Advanced Horizontal)
        ┌──────────────┐
        │ Load Balancer│
        └──────┬───────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│App 1   │ │App 2   │ │App 3   │
└────────┘ └────────┘ └────────┘
    │          │          │
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│Shard 1 │ │Shard 2 │ │Shard 3 │
└────────┘ └────────┘ └────────┘
```
