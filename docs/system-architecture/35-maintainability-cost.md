# Maintainability and Cost

## Table of Contents

1. [Maintainability](#maintainability)
   - [Failure Mode and Mitigation](#failure-mode-and-mitigation)
   - [Monitoring](#monitoring)
   - [Testing](#testing)
   - [Deployment](#deployment)
2. [Cost](#cost)
   - [Cost Formula](#cost-formula)
   - [Resource Cost Breakdown](#resource-cost-breakdown)
   - [Cost Optimization Strategies](#cost-optimization-strategies)
3. [Real-World Impact and Minimization](#real-world-impact-and-minimization)
   - [Case Study 1: E-Commerce Platform](#case-study-1-e-commerce-platform)
   - [Case Study 2: SaaS Application](#case-study-2-saas-application)
4. [Minimization Strategies by Level](#minimization-strategies-by-level)
5. [Cost Monitoring Dashboard](#cost-monitoring-dashboard)
6. [Key Takeaways](#key-takeaways)


---

## Maintainability
<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.00.02 PM.png' width=800 />

Maintainability is the ability to keep a system running efficiently, detect issues early, and deploy changes safely. It encompasses four critical pillars: failure mode analysis, monitoring, testing, and deployment strategies.

<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 1.59.33 PM.png' width=500 />

<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.00.39 PM.png' width=800 />

### Failure Mode and Mitigation

**Definition**: Failure Mode and Effects Analysis (FMEA) identifies potential failure points and implements preventive measures.

#### Common Failure Modes

| Failure Mode | Impact | Mitigation Strategy |
|---|---|---|
| **Database Connection Pool Exhaustion** | Service becomes unresponsive | Connection pooling, circuit breakers, connection timeouts |
| **Memory Leak** | Gradual performance degradation, eventual crash | Memory profiling, automated heap dumps, restart policies |
| **Cascading Failures** | One service failure triggers others | Bulkhead pattern, timeout policies, fallback mechanisms |
| **Data Corruption** | Data integrity loss, business impact | Checksums, transaction logs, backup verification |
| **Network Partition** | Service isolation, inconsistent state | Consensus algorithms, split-brain detection |
| **Resource Exhaustion** | CPU/Disk/Memory saturation | Resource limits, autoscaling, quota management |
| **Configuration Drift** | Inconsistent behavior across instances | Infrastructure as Code, configuration management |

#### Mitigation Strategies

- **Redundancy**: Duplicate critical components (active-active or active-passive)
- **Circuit Breakers**: Prevent cascading failures by stopping requests to failing services
- **Timeouts**: Prevent indefinite waiting; fail fast
- **Retry Logic**: Exponential backoff with jitter for transient failures
- **Graceful Degradation**: Reduce functionality rather than complete failure
- **Health Checks**: Continuous monitoring of component status
- **Bulkhead Pattern**: Isolate failures to specific sections
- **Backup and Recovery**: Regular backups with tested recovery procedures

#### Real-World Example: E-Commerce Platform

```
User Request
    ↓
[API Gateway] → [Circuit Breaker] → [Service A]
    ↓                                    ↓
[Load Balancer]                    [Timeout: 5s]
    ↓                                    ↓
[Service B] ← [Fallback Cache] ← [Service A Fails]
    ↓
[Response with Cached Data]
```

---

### Monitoring

**Definition**: Continuous observation of system behavior to detect anomalies and performance degradation.

#### Monitoring Pillars

1. **Metrics**: Quantitative measurements
   - Request latency (p50, p95, p99)
   - Error rates (4xx, 5xx)
   - Throughput (requests/sec)
   - Resource utilization (CPU, memory, disk, network)
   - Business metrics (conversion rate, revenue)

2. **Logs**: Detailed event records
   - Application logs (INFO, WARN, ERROR, DEBUG)
   - Access logs (request/response details)
   - Audit logs (security events)
   - Structured logging (JSON format for easy parsing)

3. **Traces**: Request flow across services
   - Distributed tracing (OpenTelemetry, Jaeger)
   - Identify bottlenecks and latency sources
   - Service dependency mapping

4. **Alerts**: Proactive notifications
   - Threshold-based (CPU > 80%)
   - Anomaly detection (unusual patterns)
   - Composite alerts (multiple conditions)
   - Alert fatigue prevention (tuning thresholds)

#### Monitoring Stack Example

```
┌─────────────────────────────────────────────────────┐
│              Application Services                   │
├─────────────────────────────────────────────────────┤
│  Metrics (Prometheus) | Logs (ELK) | Traces (Jaeger)│
├─────────────────────────────────────────────────────┤
│         Aggregation & Correlation Layer             │
├─────────────────────────────────────────────────────┤
│  Alerting (AlertManager) | Dashboards (Grafana)     │
├─────────────────────────────────────────────────────┤
│         On-Call & Incident Response                 │
└─────────────────────────────────────────────────────┘
```

#### Key Metrics to Monitor

- **Latency**: Response time (p50, p95, p99)
- **Error Rate**: Percentage of failed requests
- **Saturation**: Resource utilization levels
- **Traffic**: Request volume and patterns
- **Dependency Health**: Status of external services
- **Cost Metrics**: Resource consumption per request

#### Cost Impact of Monitoring

- **Overhead**: 5-15% additional resource consumption
- **Storage**: Log retention (1-30 days typical)
- **Alerting**: Reduces MTTR (Mean Time To Recovery) by 50-80%

---

### Testing

**Definition**: Systematic validation of system behavior to ensure correctness and reliability.

#### Testing Pyramid

```
        ┌─────────────────┐
        │   E2E Tests     │  (5-10%)
        │  (Slow, Flaky)  │
        ├─────────────────┤
        │ Integration     │  (20-30%)
        │ Tests           │
        ├─────────────────┤
        │ Unit Tests      │  (60-70%)
        │ (Fast, Reliable)│
        └─────────────────┘
```

#### Testing Types and Strategies

| Test Type | Scope | Speed | Cost | Coverage |
|---|---|---|---|---|
| **Unit Tests** | Single function/method | <100ms | Low | High |
| **Integration Tests** | Multiple components | 100ms-1s | Medium | Medium |
| **Contract Tests** | API contracts | 100ms-500ms | Low | Medium |
| **E2E Tests** | Full user workflows | 1s-10s | High | Low |
| **Load Tests** | Performance under load | 1m-1h | High | Specific |
| **Chaos Tests** | Failure scenarios | 5m-30m | High | Specific |

#### Testing Best Practices

- **Test Coverage**: Aim for 80%+ code coverage
- **Automated Testing**: CI/CD pipeline integration
- **Test Data Management**: Realistic, isolated test data
- **Flaky Test Detection**: Identify and fix unreliable tests
- **Performance Testing**: Baseline and regression detection
- **Security Testing**: SAST, DAST, dependency scanning
- **Chaos Engineering**: Intentional failure injection

#### Cost Impact of Testing

- **Development Time**: 30-50% of engineering effort
- **Infrastructure**: Test environments, CI/CD runners
- **Defect Prevention**: Reduces production bugs by 70-90%
- **MTTR Reduction**: Faster issue identification and resolution

---

### Deployment

**Definition**: Process of releasing code changes to production safely and reliably.

#### Deployment Strategies

| Strategy | Risk | Speed | Rollback | Monitoring |
|---|---|---|---|---|
| **Blue-Green** | Low | Fast | Instant | Full traffic switch |
| **Canary** | Very Low | Slow | Gradual | Subset monitoring |
| **Rolling** | Low | Medium | Gradual | Progressive |
| **Feature Flags** | Very Low | Fast | Instant | Targeted |
| **Shadow** | Very Low | Slow | N/A | Parallel comparison |

#### Blue-Green Deployment

```
┌─────────────────────────────────────────────┐
│           Load Balancer                     │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │  Blue (v1.0)    │  │  Green (v1.1)   │   │
│  │  [Active]       │  │  [Standby]      │   │
│  │  100% Traffic   │  │  0% Traffic     │   │
│  └─────────────────┘  └─────────────────┘   │
│                                             │
│  After validation: Switch to Green          │
│  Rollback: Switch back to Blue              │
└─────────────────────────────────────────────┘
```

#### Canary Deployment

```
┌─────────────────────────────────────────────┐
│           Load Balancer                     │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │  Stable (v1.0)  │  │  Canary (v1.1)  │   │
│  │  95% Traffic    │  │  5% Traffic     │   │
│  └─────────────────┘  └─────────────────┘   │
│                                             │
│  Monitor metrics → Gradually increase       │
│  canary traffic if healthy                  │
└─────────────────────────────────────────────┘
```

#### Deployment Best Practices

- **Automated Deployments**: Reduce human error
- **Deployment Frequency**: Daily or multiple times per day
- **Lead Time**: Minimize time from commit to production
- **Change Failure Rate**: Track percentage of failed deployments
- **MTTR**: Measure recovery time from failures
- **Rollback Capability**: Instant rollback mechanisms
- **Deployment Windows**: Off-peak hours for high-risk changes
- **Smoke Tests**: Quick validation post-deployment

#### Cost Impact of Deployment

- **Downtime**: Blue-green eliminates downtime (cost: infrastructure duplication)
- **Rollback Speed**: Reduces incident impact by 60-80%
- **Deployment Frequency**: Enables faster feature delivery
- **Infrastructure**: Additional resources for parallel environments

---

## Cost

### Cost Formula

```
Total Cost = Engineering Cost + Maintenance Cost + Resource Cost

Resource Cost = Hardware Cost + Software License Cost + Data Transfer Cost

Resource Cost = Hardware + Requests Made + Bytes Transferred + Bytes Stored
```


<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.01.28 PM.png' width=800 />
<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.02.20 PM.png' width=800 />

#### Cost Breakdown

```
┌─────────────────────────────────────────────────────────┐
│                   Total System Cost                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐  ┌──────────────────────────┐     │
│  │ Engineering Cost │  │  Maintenance Cost        │     │
│  │                  │  │                          │     │
│  │ • Development    │  │ • Monitoring             │     │
│  │ • Design         │  │ • Incident Response      │     │
│  │ • Testing        │  │ • Bug Fixes              │     │
│  │ • Documentation  │  │ • Performance Tuning     │     │
│  └──────────────────┘  └──────────────────────────┘     │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │         Resource Cost                            │   │
│  │                                                  │   │
│  │  ┌──────────────┐  ┌──────────────────────────┐  │   │
│  │  │ Hardware     │  │ Data Operations          │  │   │
│  │  │              │  │                          │  │   │
│  │  │ • Compute    │  │ • Requests Made          │  │   │
│  │  │ • Storage    │  │ • Bytes Transferred      │  │   │
│  │  │ • Network    │  │ • Bytes Stored           │  │   │
│  │  │ • Licenses   │  │ • API Calls              │  │   │
│  │  └──────────────┘  └──────────────────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### Resource Cost Breakdown

<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.03.41 PM.png' width=800 />

<img src='../../Resources/17-system-architecture-basics/maintainability/Screenshot 2026-02-27 at 2.05.39 PM.png' width=800 />

#### 1. Hardware Cost

| Component | Cost Driver | Optimization |
|---|---|---|
| **Compute (CPU/Memory)** | Instance type, uptime hours | Right-sizing, autoscaling, spot instances |
| **Storage** | GB stored, IOPS, throughput | Compression, tiering, lifecycle policies |
| **Network** | Data transfer, bandwidth | CDN, caching, compression |
| **Load Balancers** | Per LB, per processed GB | Consolidation, efficient routing |

**Example**: AWS EC2 Instance
- t3.medium: $0.0416/hour (on-demand) vs $0.0125/hour (spot) = 70% savings
- Annual cost: $365 (on-demand) vs $110 (spot)

#### 2. Requests Made

**Definition**: API calls, database queries, external service invocations.

| Service | Cost Model | Optimization |
|---|---|---|
| **DynamoDB** | Per request unit | Batch operations, caching, on-demand vs provisioned |
| **Lambda** | Per invocation + duration | Connection pooling, batch processing |
| **API Gateway** | Per million requests | Caching, request filtering, rate limiting |
| **Elasticsearch** | Per query | Query optimization, aggregation caching |

**Example**: DynamoDB
- Read: $0.25 per million requests (on-demand)
- 1 billion requests/month = $250/month
- With caching (90% hit rate): $25/month = 90% savings

#### 3. Bytes Transferred

**Definition**: Data egress from cloud provider, inter-region transfers, CDN usage.

| Transfer Type | Cost | Optimization |
|---|---|---|
| **Egress to Internet** | $0.09/GB (AWS) | CDN, compression, local caching |
| **Inter-Region** | $0.02/GB (AWS) | Single region, edge locations |
| **Ingress** | Free | N/A |
| **CDN** | $0.085/GB (CloudFront) | Cache hit ratio, compression |

**Example**: Video Streaming Service
- 1 PB/month egress without CDN: $90,000/month
- With CDN (80% cache hit): $18,000/month = 80% savings

#### 4. Bytes Stored

**Definition**: Data at rest in databases, object storage, backups.

| Storage Type | Cost | Optimization |
|---|---|---|
| **Hot Storage** | $0.023/GB/month (S3) | Tiering, lifecycle policies |
| **Warm Storage** | $0.0125/GB/month (S3-IA) | 30-day minimum retention |
| **Cold Storage** | $0.004/GB/month (Glacier) | Archive, 90-day minimum |
| **Database** | $0.10-1.00/GB/month | Compression, partitioning, cleanup |

**Example**: Data Lake
- 100 TB hot storage: $2,300/month
- Move 80% to cold after 30 days: $460/month = 80% savings

---

### Cost Optimization Strategies

#### By Level

##### Engineering Level
- **Code Efficiency**: Reduce algorithmic complexity (O(n²) → O(n log n))
- **Caching**: Reduce redundant computations and database queries
- **Batch Processing**: Combine multiple operations into single requests
- **Async Processing**: Defer non-critical work to off-peak hours
- **Resource Pooling**: Connection pooling, thread pools

**Impact**: 30-50% reduction in resource consumption

##### Maintenance Level
- **Monitoring Optimization**: Reduce log volume, sample metrics
- **Alert Tuning**: Reduce false positives, prevent alert fatigue
- **Incident Prevention**: Proactive monitoring reduces MTTR
- **Automation**: Reduce manual operational overhead
- **Documentation**: Reduce onboarding time for new team members

**Impact**: 20-40% reduction in operational costs

##### Resource Level
- **Right-Sizing**: Match instance types to actual workload
- **Autoscaling**: Scale down during off-peak hours
- **Reserved Instances**: 30-70% discount for committed capacity
- **Spot Instances**: 70-90% discount for interruptible workloads
- **Data Tiering**: Move cold data to cheaper storage
- **Compression**: Reduce storage and transfer costs
- **Deduplication**: Eliminate redundant data

**Impact**: 40-70% reduction in infrastructure costs

---

## Real-World Impact and Minimization

### Case Study 1: E-Commerce Platform

**Scenario**: 10 million requests/day, 500 GB data stored

#### Cost Breakdown (Monthly)

| Component | Cost | Optimization | New Cost | Savings |
|---|---|---|---|---|
| **Compute** | $5,000 | Autoscaling + spot | $1,500 | 70% |
| **Database** | $3,000 | Caching + read replicas | $900 | 70% |
| **Storage** | $1,000 | Tiering + compression | $300 | 70% |
| **Data Transfer** | $2,000 | CDN + compression | $400 | 80% |
| **Monitoring** | $500 | Log sampling | $250 | 50% |
| **Total** | **$11,500** | | **$3,350** | **71%** |

#### Implementation Timeline

1. **Week 1-2**: Implement caching layer (Redis)
2. **Week 3-4**: Set up autoscaling policies
3. **Week 5-6**: Configure CDN and data tiering
4. **Week 7-8**: Optimize monitoring and logging

---

### Case Study 2: SaaS Application

**Scenario**: 100,000 active users, 1 million API calls/day

#### Failure Mode Mitigation Impact

| Failure Mode | Downtime Cost | Mitigation Cost | ROI |
|---|---|---|---|
| **Database Failure** | $50,000/hour | $5,000 (replication) | 10x in first year |
| **Service Crash** | $30,000/hour | $2,000 (monitoring) | 15x in first year |
| **Data Corruption** | $100,000 | $3,000 (backups) | 33x in first year |

#### Testing Impact

- **Without Testing**: 5 production bugs/month × $10,000 = $50,000/month
- **With Testing**: 0.5 production bugs/month × $10,000 = $5,000/month
- **Testing Cost**: $15,000/month
- **Net Savings**: $30,000/month

---

### Minimization Strategies by Level

#### Engineering Level

1. **Algorithm Optimization**
   - Profile code to identify bottlenecks
   - Replace O(n²) with O(n log n) algorithms
   - Impact: 50-80% reduction in CPU usage

2. **Caching Strategy**
   - Multi-level caching (L1: in-memory, L2: Redis, L3: CDN)
   - Cache hit ratio target: 90%+
   - Impact: 70-90% reduction in database load

3. **Batch Processing**
   - Combine 100 individual requests into 1 batch request
   - Impact: 99% reduction in API calls

4. **Connection Pooling**
   - Reuse database connections instead of creating new ones
   - Impact: 50-70% reduction in connection overhead

#### Maintenance Level

1. **Proactive Monitoring**
   - Detect issues before they impact users
   - Reduce MTTR from hours to minutes
   - Impact: 60-80% reduction in incident costs

2. **Automated Incident Response**
   - Auto-scaling on high load
   - Automatic failover on component failure
   - Impact: 50-70% reduction in manual intervention

3. **Efficient Logging**
   - Sample logs instead of logging everything
   - Structured logging for easy analysis
   - Impact: 60-80% reduction in log storage

4. **Alert Optimization**
   - Reduce false positives through better thresholds
   - Composite alerts to reduce noise
   - Impact: 50-70% reduction in alert fatigue

#### Resource Level

1. **Compute Optimization**
   - Right-size instances to actual workload
   - Use autoscaling for variable load
   - Use spot instances for non-critical workloads
   - Impact: 60-80% reduction in compute costs

2. **Storage Optimization**
   - Compress data (2-10x reduction)
   - Tier data by access pattern (hot/warm/cold)
   - Delete old data according to retention policy
   - Impact: 70-90% reduction in storage costs

3. **Network Optimization**
   - Use CDN for static content (80% cache hit)
   - Compress responses (gzip, brotli)
   - Batch requests to reduce overhead
   - Impact: 70-90% reduction in data transfer costs

4. **Database Optimization**
   - Index frequently queried columns
   - Denormalize for read-heavy workloads
   - Use read replicas for scaling reads
   - Impact: 50-80% reduction in database costs

---

### Cost Monitoring Dashboard

```
┌─────────────────────────────────────────────────────────┐
│              Cost Monitoring Dashboard                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Daily Cost: $450  (↓ 15% vs last week)                 │
│  Monthly Projection: $13,500  (↓ 20% vs last month)     │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Cost by Component                               │    │
│  │ Compute:      45% ($6,075)                      │    │
│  │ Storage:      20% ($2,700)                      │    │
│  │ Data Transfer: 25% ($3,375)                     │    │
│  │ Services:     10% ($1,350)                      │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Cost per Request                                │    │
│  │ Current: $0.0045  (↓ 10% vs last month)         │    │
│  │ Target:  $0.0035  (22% reduction needed)        │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Top Cost Drivers                                │    │
│  │ 1. Database queries (35%)                       │    │
│  │ 2. Data egress (25%)                            │    │
│  │ 3. Compute instances (25%)                      │    │
│  │ 4. Storage (15%)                                │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### Key Takeaways

1. **Maintainability and Cost are Interconnected**
   - Good monitoring prevents expensive outages
   - Comprehensive testing reduces production bugs
   - Safe deployment strategies minimize rollback costs

2. **Optimization Requires Multi-Level Approach**
   - Engineering: Code efficiency and caching
   - Maintenance: Proactive monitoring and automation
   - Resources: Right-sizing and tiering

3. **Measure and Monitor**
   - Track cost per request, per user, per feature
   - Set cost budgets and alerts
   - Regular cost reviews and optimization cycles

4. **Trade-offs Matter**
   - Higher upfront investment in monitoring/testing reduces operational costs
   - Redundancy increases infrastructure costs but reduces downtime costs
   - Balance between cost and reliability based on business requirements
