# Availability: SLI, SLO, SLA & Error Budgets

## Table of Contents
1. [Availability Overview](#availability-overview)
2. [The Famous Nines & Calculation](#the-famous-nines--calculation)
   - 2.1 [Availability Tiers](#availability-tiers)
   - 2.2 [Calculation Example](#calculation-example)
3. [SLI, SLO, SLA Definitions](#sli-slo-sla-definitions)
   - 3.1 [Service Level Indicator (SLI)](#service-level-indicator-sli)
   - 3.2 [Service Level Objective (SLO)](#service-level-objective-slo)
   - 3.3 [Service Level Agreement (SLA)](#service-level-agreement-sla)
   - 3.4 [Relationship Diagram](#relationship-diagram)
4. [How to Measure Availability](#how-to-measure-availability)
   - 4.1 [Measurement Methods](#measurement-methods)
   - 4.2 [Measurement Example](#measurement-example)
5. [Hierarchy of SLI, SLO, SLA](#hierarchy-of-sli-slo-sla)
   - 5.1 [How Companies Structure the Hierarchy](#how-companies-structure-the-hierarchy)
   - 5.2 [Company Implementation Example](#company-implementation-example)
6. [Error Budget](#error-budget)
   - 6.1 [Definition](#definition)
   - 6.2 [Error Budget Calculation](#error-budget-calculation)
   - 6.3 [Error Budget Consumption Tracking](#error-budget-consumption-tracking)
7. [Error Budget Calculation & MAANG Usage](#error-budget-calculation--maang-usage)
   - 7.1 [How MAANG Companies Use Error Budgets](#how-maang-companies-use-error-budgets)
   - 7.2 [Error Budget Policies](#error-budget-policies)
8. [Availability Impact on Design Decisions](#availability-impact-on-design-decisions)
   - 8.1 [Database Architecture Comparison: 99.9% vs 99.99%](#database-architecture-comparison-999-vs-9999)
   - 8.2 [Design Decision Comparison Table](#design-decision-comparison-table)
9. [Real-World Comparison: Instagram vs URL Shortener](#real-world-comparison-instagram-vs-url-shortener)
   - 9.1 [Instagram: 99.99% Availability Design](#instagram-9999-availability-design)
   - 9.2 [URL Shortener: 99.9% Availability Design](#url-shortener-999-availability-design)
   - 9.3 [Comparison Summary](#comparison-summary)
10. [Critical Tradeoffs for Higher Availability](#critical-tradeoffs-for-higher-availability)
    - 10.1 [Tradeoff 1: Cost vs Availability](#tradeoff-1-cost-vs-availability)
    - 10.2 [Tradeoff 2: Consistency vs Availability](#tradeoff-2-consistency-vs-availability)
    - 10.3 [Tradeoff 3: Complexity vs Availability](#tradeoff-3-complexity-vs-availability)
    - 10.4 [Tradeoff 4: Latency vs Availability](#tradeoff-4-latency-vs-availability)
    - 10.5 [Tradeoff 5: Feature Velocity vs Availability](#tradeoff-5-feature-velocity-vs-availability)
    - 10.6 [Tradeoff 6: Operational Complexity vs Availability](#tradeoff-6-operational-complexity-vs-availability)
    - 10.7 [Decision Matrix: When to Choose Each Availability Level](#decision-matrix-when-to-choose-each-availability-level)
    - 10.8 [Key Principles](#key-principles)

---

## Availability Overview

**Availability** measures the percentage of time a system is operational and accessible to users.

- **Formula**: `Availability = (Total Time - Downtime) / Total Time × 100%`
- **Importance**: Directly impacts user experience, revenue, and trust
- **Measurement**: Typically expressed as a percentage (99%, 99.9%, 99.99%, etc.)
- **Context**: Different systems require different availability levels based on business criticality

---

## The Famous Nines & Calculation

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.01.39 PM.png' width='500' />

The "Nines" represent availability percentages and their corresponding maximum acceptable downtime.

### Availability Tiers

| Availability | Downtime/Year | Downtime/Month | Downtime/Week | Downtime/Day |
|---|---|---|---|---|
| **99%** (Two Nines) | 3.65 days | 7.2 hours | 1.68 hours | 14.4 minutes |
| **99.9%** (Three Nines) | 8.76 hours | 43.2 minutes | 10.08 minutes | 86.4 seconds |
| **99.99%** (Four Nines) | 52.6 minutes | 4.32 minutes | 1.008 minutes | 8.64 seconds |
| **99.999%** (Five Nines) | 5.26 minutes | 25.9 seconds | 6.05 seconds | 0.864 seconds |
| **99.9999%** (Six Nines) | 31.5 seconds | 2.59 seconds | 0.605 seconds | 0.0864 seconds |

### Calculation Example

**For 99.9% availability over a year:**
- Total minutes in year = 365 × 24 × 60 = 525,600 minutes
- Allowed downtime = 525,600 × (1 - 0.999) = 525.6 minutes ≈ **8.76 hours/year**

**For 99.99% availability over a month:**
- Total minutes in month = 30 × 24 × 60 = 43,200 minutes
- Allowed downtime = 43,200 × (1 - 0.9999) = 4.32 minutes ≈ **4 minutes 19 seconds/month**

---

## SLI, SLO, SLA Definitions

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.02.04 PM.png' width='600' />

### Service Level Indicator (SLI)

**Definition**: A quantitative measurement of a specific aspect of service performance.

    It's like "What are we measuring?"

- **Characteristics**:
  - Measurable metric
  - Reflects user experience
  - Based on actual system behavior
  - Examples: request latency, error rate, throughput

- **Common SLIs**:
  - Request latency (p50, p95, p99)
  - Error rate (% of failed requests)
  - Availability (% of time service is up)
  - Throughput (requests per second)
  - Data freshness (staleness of cached data)

### Service Level Objective (SLO)

    It's like "What are we aiming for?"

**Definition**: A target value or range for an SLI that the team commits to achieving.

- **Characteristics**:
  - Internal commitment
  - Drives engineering decisions
  - Defines acceptable performance
  - Stricter than SLA

- **Examples**:
  - "99.95% availability"
  - "p99 latency < 100ms"
  - "Error rate < 0.1%"

### Service Level Agreement (SLA)

    It's like "What are we promising to the customer?"

**Definition**: A contractual commitment to customers about service performance with penalties for non-compliance.

- **Characteristics**:
  - Legal/contractual obligation
  - Includes penalties (credits, refunds)
  - Typically less strict than SLO
  - Publicly communicated

- **Examples**:
  - "99.9% availability with 10% service credit if breached"
  - "p99 latency < 200ms"

### Relationship Diagram

```
SLI (Measurement)
    ↓
SLO (Internal Target) ← Stricter
    ↓
SLA (Customer Promise) ← Less Strict
```

---

## How to Measure Availability

### Measurement Methods

**1. Uptime Monitoring**
- Continuous health checks from multiple locations
- Synthetic monitoring (simulated user requests)
- Real user monitoring (RUM)

**2. Error Rate Tracking**
- Count failed requests vs total requests
- Track HTTP 5xx errors
- Monitor timeout errors

**3. Latency Monitoring**
- Measure response times at percentiles (p50, p95, p99)
- Track SLA violations

**4. Dependency Health**
- Monitor database availability
- Track external service health
- Monitor infrastructure components

### Measurement Example

```
Total Requests: 1,000,000
Failed Requests: 100
Successful Requests: 999,900

Availability = (999,900 / 1,000,000) × 100 = 99.99%
Error Rate = (100 / 1,000,000) × 100 = 0.01%
```

---

## Hierarchy of SLI, SLO, SLA

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.04.03 PM.png' width='500' />


### How Companies Structure the Hierarchy

```
┌─────────────────────────────────────────┐
│  SLA (Customer Commitment)              │
│  99.9% availability                     │
│  (Contractual, with penalties)          │
└─────────────────────────────────────────┘
                    ↑
┌─────────────────────────────────────────┐
│  SLO (Internal Target)                  │
│  99.95% availability                    │
│  (Buffer between SLA and reality)       │
└─────────────────────────────────────────┘
                    ↑
┌─────────────────────────────────────────┐
│  SLI (Actual Measurement)               │
│  Real-time monitoring data              │
│  (What we actually achieve)             │
└─────────────────────────────────────────┘
```

### Company Implementation Example

**Google Cloud Platform:**
- **SLA**: 99.95% availability (monthly uptime percentage)
- **SLO**: 99.99% availability (internal target)
- **SLI**: Measured via synthetic monitoring + RUM

**AWS:**
- **SLA**: 99.99% availability for EC2
- **SLO**: 99.999% availability (internal)
- **SLI**: CloudWatch metrics + health checks

**Netflix:**
- **SLA**: 99.99% availability
- **SLO**: 99.999% availability
- **SLI**: Hystrix circuit breakers + real-time monitoring

---

## Error Budget

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.04.50 PM.png' width='500' />

### Definition

**Error Budget** is the maximum amount of downtime or errors allowed while still meeting the SLO.

- **Formula**: `Error Budget = (1 - SLO) × Total Time`
- **Purpose**: Allows controlled risk-taking for faster feature deployment
- **Allocation**: Shared between planned maintenance and unexpected failures

### Error Budget Calculation

**Example: 99.9% SLO for a month**

```
Total minutes in month = 30 × 24 × 60 = 43,200 minutes
SLO = 99.9%
Error Budget = (1 - 0.999) × 43,200 = 43.2 minutes

Allocation:
├─ Planned Maintenance: 20 minutes
├─ Unexpected Failures: 23.2 minutes
└─ Buffer: 0 minutes
```

**Example: 99.99% SLO for a month**

```
Total minutes in month = 43,200 minutes
SLO = 99.99%
Error Budget = (1 - 0.9999) × 43,200 = 4.32 minutes

Allocation:
├─ Planned Maintenance: 2 minutes
├─ Unexpected Failures: 2 minutes
└─ Buffer: 0.32 minutes
```

### Error Budget Consumption Tracking

```
Month Start: 43.2 minutes available
├─ Week 1: Deploy new feature (5 min downtime) → 38.2 min remaining
├─ Week 2: Database migration (8 min downtime) → 30.2 min remaining
├─ Week 3: Incident (2 min downtime) → 28.2 min remaining
└─ Week 4: Safe to deploy? Only 28.2 min left → Risky
```

---

## Error Budget Calculation & MAANG Usage

### How MAANG Companies Use Error Budgets

**Meta (Facebook)**
- **SLO**: 99.99% availability
- **Error Budget**: ~4.3 minutes/month
- **Usage**: 
  - Allocates 2 minutes for planned deployments
  - Reserves 2 minutes for unexpected failures
  - Deployment velocity: ~1000+ deploys/day (uses error budget aggressively)

**Apple**
- **SLO**: 99.999% availability (iCloud)
- **Error Budget**: ~26 seconds/month
- **Usage**:
  - Very conservative deployment strategy
  - Extensive testing before production
  - Minimal error budget consumption

**Google**
- **SLO**: 99.99% availability (Gmail)
- **Error Budget**: ~4.3 minutes/month
- **Usage**:
  - Canary deployments (1% traffic first)
  - Automated rollback on SLO breach
  - Real-time error budget monitoring

**Amazon (AWS)**
- **SLO**: 99.99% availability (S3)
- **Error Budget**: ~4.3 minutes/month
- **Usage**:
  - Multi-region deployment strategy
  - Gradual rollout across regions
  - Error budget per region

**Netflix**
- **SLO**: 99.99% availability
- **Error Budget**: ~4.3 minutes/month
- **Usage**:
  - Chaos engineering to test resilience
  - Rapid deployment with circuit breakers
  - Error budget as deployment gate

### Error Budget Policies

**Aggressive (Meta, Netflix)**
- Spend error budget to deploy faster
- Accept higher risk for feature velocity
- Suitable for: Social media, streaming

**Conservative (Apple)**
- Preserve error budget for emergencies
- Slow, careful deployments
- Suitable for: Critical infrastructure, payments

**Balanced (Google, Amazon)**
- Spend error budget strategically
- Automated safeguards (canary, rollback)
- Suitable for: Cloud services, email

---

## Availability Impact on Design Decisions

### Database Architecture Comparison: 99.9% vs 99.99%

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.11.17 PM.png' width='700' />

---

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.12.43 PM.png' width='700' />

#### 99.9% Availability (8.76 hours downtime/year)

**Database Design:**
- Single primary database with read replicas
- Synchronous replication to 1-2 replicas
- RTO (Recovery Time Objective): 5-10 minutes
- RPO (Recovery Point Objective): < 1 minute

**Architecture:**
```
┌──────────────────┐
│  Primary DB      │
│  (Master)        │
└────────┬─────────┘
         │ Sync Replication
    ┌────┴────┐
    │         │
┌───▼──┐  ┌───▼──┐
│ Read │  │ Read │
│ Rep1 │  │ Rep2 │
└──────┘  └──────┘
```

**Characteristics:**
- Single region or dual region
- Failover time: 5-10 minutes
- Cost: Moderate
- Complexity: Medium

#### 99.99% Availability (52.6 minutes downtime/year)

**Database Design:**
- Multi-region active-active setup
- Asynchronous replication across regions
- RTO: < 1 minute
- RPO: < 10 seconds

**Architecture:**
```
┌──────────────────┐
│  Region 1        │
│  Primary DB      │
└────────┬─────────┘
         │ Async Replication
    ┌────┴────────────────┐
    │                     │
┌───▼──────────┐  ┌───────▼──┐
│  Region 2    │  │ Region 3 │
│  Primary DB  │  │ Primary  │
└──────────────┘  └──────────┘
```

**Characteristics:**
- Multi-region active-active
- Automatic failover (< 1 minute)
- Cost: 3-4x higher
- Complexity: High

### Design Decision Comparison Table

| Aspect | 99.9% | 99.99% |
|---|---|---|
| **Regions** | 1-2 | 3+ |
| **Replication** | Sync | Async |
| **Failover Time** | 5-10 min | < 1 min |
| **Data Loss Risk** | Low | Very Low |
| **Cost** | 1x | 3-4x |
| **Complexity** | Medium | High |
| **Deployment** | Weekly | Daily |
| **Testing** | Standard | Chaos Engineering |

---

## Real-World Comparison: Instagram vs URL Shortener

### Instagram: 99.99% Availability Design

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.16.22 PM.png' width='600' />

**Requirements:**
- 2+ billion users
- Real-time feed updates
- Photo/video uploads
- Critical business service

**Design Decisions:**

| Component | Decision | Rationale |
|---|---|---|
| **Database** | Multi-region active-active (3 regions) | Minimize failover time, handle regional outages |
| **Replication** | Async with eventual consistency | Accept slight delays for availability |
| **Cache** | Multi-layer (Redis, Memcached) | Reduce DB load, serve stale data if needed |
| **Load Balancer** | Global load balancing | Route around failed regions |
| **Deployment** | Canary (1% → 10% → 100%) | Catch issues before full rollout |
| **Monitoring** | Real-time SLI tracking | Immediate incident detection |
| **Failover** | Automatic (< 30 seconds) | No manual intervention needed |
| **Data Consistency** | Eventual consistency | Accept temporary inconsistencies |
| **Infrastructure** | Dedicated hardware | Guaranteed capacity |
| **Cost** | ~$500M+ annually | Justified by revenue impact |

**Architecture:**
```
┌──────────────────────────────────┐
│  Global Load Balancer (Anycast)  │
└────────────┬─────────────────────┘
             │
    ┌────────┼────────┐
    │        │        │
┌───▼──┐ ┌───▼───┐┌───▼──┐
│ US   │ │ EU   │ │ APAC │
│ DC   │ │ DC   │ │ DC   │
└───┬──┘ └──┬───┘ └──┬───┘
    │       │       │
    └───┬───┴───┬───┘
        │       │
    ┌───▼──┐ ┌─▼────┐
    │ Cache│ │ Cache│
    └───┬──┘ └─┬────┘
        │      │
    ┌───▼──┐ ┌─▼────┐
    │ DB   │ │ DB   │
    │ Shard│ │ Shard│
    └──────┘ └──────┘
```

### URL Shortener: 99.9% Availability Design

<img src='../../Resources/17-system-architecture-basics/availability//Screenshot 2026-02-11 at 6.17.03 PM.png' width='600' />

**Requirements:**
- High throughput (millions of requests/day)
- Simple read-heavy workload
- Acceptable downtime: 8.76 hours/year
- Cost-sensitive

**Design Decisions:**

| Component | Decision | Rationale |
|---|---|---|
| **Database** | Single primary + 2 read replicas (1 region) | Sufficient for availability target |
| **Replication** | Synchronous | Ensure data consistency |
| **Cache** | Single Redis cluster | Cost-effective, acceptable latency |
| **Load Balancer** | Regional load balancing | Simpler, cheaper than global |
| **Deployment** | Rolling deployment (10% at a time) | Acceptable risk for this SLO |
| **Monitoring** | Hourly SLI checks | Less frequent monitoring |
| **Failover** | Manual or 5-10 min automatic | Acceptable for non-critical service |
| **Data Consistency** | Strong consistency | Important for URL mappings |
| **Infrastructure** | Shared cloud resources | Cost optimization |
| **Cost** | ~$50K annually | Proportional to traffic |

**Architecture:**
```
┌────────────────────────────┐
│  Regional Load Balancer    │
└────────────┬───────────────┘
             │
    ┌────────┼────────┐
    │        │        │
┌───▼──┐ ┌──▼───┐ ┌──▼───┐
│ App  │ │ App  │ │ App  │
│ Srv1 │ │ Srv2 │ │ Srv3 │
└───┬──┘ └──┬───┘ └──┬───┘
    │       │       │
    └───┬───┴───┬───┘
        │       │
    ┌───▼──┐ ┌─▼────┐
    │ Cache│ │ Cache│
    └───┬──┘ └─┬────┘
        │      │
    ┌───▼──────▼──┐
    │ Primary DB  │
    └───┬─────────┘
        │ Sync Replication
    ┌───┴──────┬──────┐
    │          │      │
┌───▼──┐  ┌───▼──┐ ┌─▼────┐
│ Read │  │ Read │ │ Backup│
│ Rep1 │  │ Rep2 │ │ (Cold)│
└──────┘  └──────┘ └───────┘
```

### Comparison Summary

<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.19.07 PM.png' width='700' />

| Aspect | Instagram (99.99%) | URL Shortener (99.9%) |
|---|---|---|
| **Regions** | 3+ | 1 |
| **Replication** | Async | Sync |
| **Failover** | < 30 sec | 5-10 min |
| **Deployment Frequency** | 10+ per day | 1-2 per week |
| **Error Budget/Month** | 4.3 min | 43.2 min |
| **Infrastructure Cost** | $500M+ | $50K |
| **Complexity** | Very High | Medium |
| **Data Consistency** | Eventual | Strong |
| **Monitoring** | Real-time | Hourly |
| **Incident Response** | Automatic | Manual |

---

## Critical Tradeoffs for Higher Availability


<img src='../../Resources/17-system-architecture-basics/availability/Screenshot 2026-02-11 at 6.19.37 PM.png' width='600' />

### Tradeoff 1: Cost vs Availability

**99.9% Availability:**
- Single region
- Shared infrastructure
- Cost: $1x

**99.99% Availability:**
- Multi-region (3+)
- Dedicated infrastructure
- Cost: $3-4x

**Tradeoff**: Each additional nine costs exponentially more.

```
Cost Curve:
│
│     ╱╱╱╱
│   ╱╱
│ ╱╱
└─────────────────
  99% 99.9% 99.99% 99.999%
```

### Tradeoff 2: Consistency vs Availability

**Strong Consistency (99.9%):**
- Synchronous replication
- All replicas always in sync
- Slower writes
- No data loss

**Eventual Consistency (99.99%):**
- Asynchronous replication
- Temporary inconsistencies
- Faster writes
- Possible data loss in failures

**Example:**
```
Strong Consistency:
Write → Primary → Replicas (wait) → Ack to client
Latency: 100ms

Eventual Consistency:
Write → Primary → Ack to client → Replicas (async)
Latency: 10ms
```

### Tradeoff 3: Complexity vs Availability

**Simple Architecture (99.9%):**
- Single database
- Basic monitoring
- Easy to understand
- Easier to debug

**Complex Architecture (99.99%):**
- Multi-region, multi-database
- Advanced monitoring (distributed tracing)
- Hard to understand
- Difficult to debug

**Operational Overhead:**
- 99.9%: 2-3 engineers
- 99.99%: 10-15 engineers

### Tradeoff 4: Latency vs Availability

**Low Latency (99.9%):**
- Single region
- Synchronous operations
- p99 latency: 50ms

**High Availability (99.99%):**
- Multi-region
- Asynchronous operations
- p99 latency: 200ms

**Example:**
```
Single Region (99.9%):
User → US DC → Response (50ms)

Multi-Region (99.99%):
User → Nearest DC → Async to other DCs → Response (200ms)
```

### Tradeoff 5: Feature Velocity vs Availability

**Fast Deployment (99.9%):**
- Deploy multiple times per day
- Spend error budget on features
- Higher risk of incidents

**Slow Deployment (99.99%):**
- Deploy once per week
- Preserve error budget
- Lower risk of incidents

**Example:**
```
99.9% SLO (43.2 min/month error budget):
├─ Deploy 20 times/month
├─ Each deploy: 2 min downtime
└─ Total: 40 min (within budget)

99.99% SLO (4.3 min/month error budget):
├─ Deploy 2 times/month
├─ Each deploy: 2 min downtime
└─ Total: 4 min (at limit)
```

### Tradeoff 6: Operational Complexity vs Availability

**Simple Operations (99.9%):**
- Manual failover
- Basic alerting
- Runbooks for common issues

**Automated Operations (99.99%):**
- Automatic failover
- Advanced alerting (ML-based)
- Self-healing systems

**Example:**
```
99.9% Incident Response:
1. Alert fires (5 min delay)
2. Engineer pages (2 min)
3. Engineer investigates (5 min)
4. Manual failover (5 min)
Total: 17 minutes

99.99% Incident Response:
1. Automatic detection (10 sec)
2. Automatic failover (20 sec)
3. Alert to engineer (30 sec)
Total: 1 minute
```

### Decision Matrix: When to Choose Each Availability Level

| Availability | Use Case | Example |
|---|---|---|
| **99%** | Non-critical services | Internal tools, batch jobs |
| **99.9%** | Standard services | URL shortener, blog platform |
| **99.95%** | Important services | E-commerce, SaaS platforms |
| **99.99%** | Critical services | Instagram, Gmail, AWS |
| **99.999%** | Mission-critical | Banking, healthcare, 911 |

### Key Principles

1. **Don't over-engineer**: Choose availability based on business impact, not technical capability
2. **Error budget is a tool**: Use it to balance velocity and stability
3. **Measure what matters**: SLI should reflect user experience
4. **Automate everything**: Manual processes don't scale to high availability
5. **Plan for failure**: Assume components will fail and design accordingly

---

## Summary

- **Availability** is measured in "nines" (99%, 99.9%, 99.99%, etc.)
- **SLI** measures actual performance, **SLO** sets internal targets, **SLA** makes customer promises
- **Error budget** allows controlled risk-taking for faster deployment
- **Higher availability** requires exponential increases in cost and complexity
- **Design decisions** must balance availability, consistency, latency, and cost
- **MAANG companies** use error budgets strategically based on business priorities
- **Real-world systems** choose availability levels based on business criticality, not technical perfection

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
