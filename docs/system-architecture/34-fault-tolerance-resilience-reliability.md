# Fault Tolerance, Resilience, and Reliability

## Table of Contents

1. [Fault Tolerance](#fault-tolerance)
   - [Definition](#definition)
   - [Fault Tolerance vs High Availability](#fault-tolerance-vs-high-availability)
   - [Relationship Between Fault Tolerance and High Availability](#relationship-between-fault-tolerance-and-high-availability)
   - [How Fault Tolerance is Achieved](#how-fault-tolerance-is-achieved)
   - [Benefits and Tradeoffs](#benefits-and-tradeoffs)
2. [Resilience](#resilience)
   - [Definition](#resilience-definition)
   - [Resilience vs Fault Tolerance](#resilience-vs-fault-tolerance)
   - [How Resilience is Achieved](#how-resilience-is-achieved)
     - [Graceful Degradation](#1-graceful-degradation)
     - [Circuit Breaker Pattern](#2-circuit-breaker-pattern)
     - [Adaptive Rate Limiting](#3-adaptive-rate-limiting)
     - [Bulkhead Pattern](#4-bulkhead-pattern)
   - [Resilience Patterns](#resilience-patterns)
3. [Reliability](#reliability)
   - [Definition](#reliability-definition)
   - [Reliability Formula](#reliability-formula)
   - [How Reliability is Achieved](#how-reliability-is-achieved)
     - [Availability](#availability)
     - [Correctness](#correctness)
     - [Timeliness](#timeliness)
     - [Monitoring and Observability](#monitoring-and-observability)
     - [Testing and Validation](#testing-and-validation)
4. [Chaos Engineering](#chaos-engineering)
   - [Definition](#chaos-engineering-definition)
   - [How Chaos Engineering is Achieved](#how-chaos-engineering-is-achieved)
   - [Chaos Monkey](#chaos-monkey)

---

## Fault Tolerance

### Definition

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.26.47 PM.png' width=500 />

**Fault Tolerance** is the ability of a system to continue operating correctly even when one or more of its components fail. It ensures that failures in individual components do not cause the entire system to fail.

- **Core Principle**: Anticipate failures and design systems to handle them gracefully
- **Scope**: Focuses on component-level failures (hardware, software, network)
- **Goal**: Maintain system functionality despite failures

| ![image](../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot%202026-02-13%20at%201.01.41 PM.png) | → | ![image](../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot%202026-02-13%20at%201.03.06 PM.png) |
| :---: | :---: | :---: |

### Fault Tolerance vs High Availability

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.00.29 PM.png' width=500 />

| Aspect | Fault Tolerance | High Availability |
|--------|-----------------|-------------------|
| **Focus** | System continues operating despite failures | System is accessible and operational most of the time |
| **Approach** | Redundancy and failover mechanisms | Minimizing downtime through quick recovery |
| **Failure Handling** | Absorbs failures without interruption | Recovers quickly from failures |
| **Downtime** | Zero or minimal downtime | Acceptable minimal downtime (99.9%, 99.99%) |
| **Cost** | Higher (requires redundancy) | Moderate to high |
| **Example** | RAID storage continues despite disk failure | Database cluster with automatic failover |

### Relationship Between Fault Tolerance and High Availability

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.04.04 PM.png' width=500 />

```
┌─────────────────────────────────────────────────────────┐
│                    System Reliability                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────┐    ┌──────────────────────┐   │
│  │  Fault Tolerance     │    │  High Availability   │   │
│  │  (Prevent Failure)   │    │  (Quick Recovery)    │   │
│  │                      │    │                      │   │
│  │  • Redundancy        │    │  • Failover          │   │
│  │  • Replication       │    │  • Load Balancing    │   │
│  │  • Error Correction  │    │  • Health Checks     │   │
│  └──────────────────────┘    └──────────────────────┘   │
│           ↓                            ↓                │
│    System continues despite      System recovers        │
│    component failures            quickly from failures  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Complementary Relationship**:
- **Fault Tolerance** prevents failures from affecting the system
- **High Availability** ensures quick recovery when failures occur
- Together they maximize system uptime and reliability

### How Fault Tolerance is Achieved

1. [Redundancy](#1-redundancy)
2. [Replication](#2-replication)
3. [Error Detection and Correction](#3-error-detection-and-correction)
4. [Failover Mechanisms](#4-failover-mechanisms)
5. [Graceful Degradation](#5-graceful-degradation)
6. [Isolation (Bulkhead Pattern)](#6-isolation-bulkhead-pattern)

#### 1. **Redundancy**
- Duplicate critical components (servers, databases, networks)
- If one component fails, others take over
- **Types**:
  - **Active-Active**: All components operate simultaneously
  - **Active-Passive**: Standby components activate on failure

#### 2. **Replication**
- Data is copied across multiple nodes
- Ensures data availability even if one node fails
- **Strategies**:
  - Synchronous replication (strong consistency, higher latency)
  - Asynchronous replication (eventual consistency, lower latency)

#### 3. **Error Detection and Correction**
- **Checksums**: Detect data corruption
- **Parity Bits**: Correct single-bit errors
- **Health Checks**: Monitor component status continuously
- **Heartbeat Monitoring**: Detect failed nodes

#### 4. **Failover Mechanisms**
- Automatic detection of failures
- Seamless switching to backup components
- **Failover Time**: Measured in seconds to milliseconds
- **Example**: Database replication with automatic promotion of replica to primary

#### 5. **Graceful Degradation**
- System continues with reduced functionality
- Non-critical features disabled to maintain core operations
- **Example**: E-commerce site shows cached product data if database is slow

#### 6. **Isolation (Bulkhead Pattern)**
- Partition system into independent sections
- Failure in one section doesn't cascade to others
- **Example**: Separate thread pools for different services

### Benefits and Tradeoffs

#### Benefits
- **Increased Uptime**: System remains operational during failures
- **Data Protection**: Replication prevents data loss
- **User Experience**: Transparent failure handling
- **Business Continuity**: Reduced revenue loss from downtime
- **Compliance**: Meets regulatory requirements for availability

#### Tradeoffs
- **Increased Complexity**: More components to manage and monitor
- **Higher Costs**: Redundancy requires additional hardware/infrastructure
- **Performance Overhead**: Replication and synchronization add latency
- **Operational Burden**: More complex deployment and maintenance
- **Consistency Challenges**: Replication can lead to eventual consistency issues

---

## Resilience
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.07.42 PM.png' width=500 />

### Resilience Definition

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.06.54 PM.png' width=600 />
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.07.22 PM.png' width=600 />

**Resilience** is the ability of a system to handle unexpected failures, adapt to changing conditions, and recover gracefully. It goes beyond fault tolerance by addressing surprises and unknown failure modes.

- **Core Principle**: Expect the unexpected and design systems to adapt
- **Scope**: Handles both known and unknown failure scenarios
- **Goal**: Maintain acceptable service levels despite adverse conditions

### Resilience vs Fault Tolerance

| Aspect | Fault Tolerance | Resilience |
|--------|-----------------|-----------|
| **Scope** | Known, anticipated failures | Known and unknown failures |
| **Approach** | Redundancy and failover | Adaptation and graceful degradation |
| **Failure Types** | Component failures | Cascading failures, resource exhaustion, unexpected behavior |
| **Recovery** | Automatic failover | Adaptive response and recovery |
| **Design** | Proactive (prevent failures) | Reactive (handle surprises) |
| **Example** | RAID disk failure | System handles sudden traffic spike |

### How Resilience is Achieved
1. [Graceful Degradation](#1-graceful-degradation)
2. [Circuit Breaker Pattern](#2-circuit-breaker-pattern)
3. [Adaptive Rate Limiting](#3-adaptive-rate-limiting)
4. [Bulkhead Pattern](#4-bulkhead-pattern)


#### 1. **Graceful Degradation**
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.08.01 PM.png' width=600 />

- System reduces functionality instead of failing completely
- Prioritizes critical features
- **Example**: 
  - Search service slow → Show cached results
  - Image service down → Display placeholder images
  - Recommendation engine unavailable → Show popular items

```
┌─────────────────────────────────────────┐
│         System Under Stress             │
├─────────────────────────────────────────┤
│                                         │
│  Normal Operation                       │
│  ✓ All features available               │
│  ✓ Full functionality                   │
│                                         │
│           ↓ (Stress/Failure)            │
│                                         │
│  Degraded Operation                     │
│  ✓ Core features available              │
│  ✗ Non-critical features disabled       │
│  ✓ Reduced performance acceptable       │
│                                         │
│           ↓ (Severe Stress)             │
│                                         │
│  Minimal Operation                      │
│  ✓ Critical features only               │
│  ✗ Most features disabled               │
│  ✓ System still responsive              │
│                                         │
└─────────────────────────────────────────┘
```

#### 2. **Circuit Breaker Pattern**
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.08.38 PM.png' width=600 />

- Prevents cascading failures by stopping requests to failing services
- **States**:
  - **Closed**: Normal operation, requests pass through
  - **Open**: Service failing, requests rejected immediately
  - **Half-Open**: Testing if service recovered, limited requests allowed

```
┌──────────────────────────────────────────────────────┐
│           Circuit Breaker State Machine              │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────┐                                         │
│  │ CLOSED  │ ← Normal operation                      │
│  │ (Allow) │   Requests pass through                 │
│  └────┬────┘                                         │
│       │ Failure threshold exceeded                   │
│       ↓                                              │
│  ┌─────────┐                                         │
│  │  OPEN   │ ← Failure detected                      │
│  │(Reject) │   Requests rejected immediately         │
│  └────┬────┘                                         │
│       │ Timeout elapsed                              │
│       ↓                                              │
│  ┌──────────────┐                                    │
│  │ HALF-OPEN    │ ← Testing recovery                 │
│  │(Test Limit)  │   Limited requests allowed         │
│  └────┬─────────┘                                    │
│       │ Success → CLOSED                             │
│       │ Failure → OPEN                               │
│       ↓                                              │
│  (Back to CLOSED or OPEN)                            │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Benefits**:
- Prevents cascading failures
- Allows failing service time to recover
- Provides fast failure feedback to clients
- Reduces resource waste on failing requests

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.09.01 PM.png' width=700 />

#### 3. **Adaptive Rate Limiting**
- Dynamically adjust request rates based on system load
- Prevents resource exhaustion
- **Strategies**:
  - **Token Bucket**: Refill tokens at fixed rate, request consumes tokens
  - **Leaky Bucket**: Fixed outflow rate, requests queued
  - **Sliding Window**: Track requests in time window
  - **Adaptive**: Adjust limits based on system metrics (CPU, memory, latency)

```
┌─────────────────────────────────────────┐
│    Adaptive Rate Limiting               │
├─────────────────────────────────────────┤
│                                         │
│  Monitor System Metrics                 │
│  • CPU usage                            │
│  • Memory usage                         │
│  • Response latency                     │
│  • Queue depth                          │
│           ↓                             │
│  Adjust Rate Limits                     │
│  • High load → Reduce limit             │
│  • Low load → Increase limit            │
│  • Critical threshold → Reject requests │
│           ↓                             │
│  Maintain Acceptable Performance        │
│  • Prevent overload                     │
│  • Fair resource allocation             │
│  • Graceful degradation                 │
│                                         │
└─────────────────────────────────────────┘
```

#### 4. **Bulkhead Pattern**
- Isolate failures to specific components
- Prevent failure propagation across system
- **Implementation**:
  - Separate thread pools for different services
  - Dedicated resources for critical operations
  - Partition system into independent sections

```
┌───────────────────────────────────────────────────────┐
│              System with Bulkheads                    │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐   │
│  │  Service A   │  │  Service B   │  │ Service C  │   │
│  │              │  │              │  │            │   │
│  │ Thread Pool  │  │ Thread Pool  │  │Thread Pool │   │
│  │   (10 thr)   │  │   (10 thr)   │  │ (10 thr)   │   │
│  └──────────────┘  └──────────────┘  └────────────┘   │
│         ↓                 ↓                  ↓        │
│    Isolated         Isolated           Isolated       │
│    Failure in A     Failure in B       Failure in C   │
│    doesn't affect   doesn't affect     doesn't affect │
│    B or C           A or C             A or B         │
│                                                       │
└───────────────────────────────────────────────────────┘
```

**Benefits**:
- Limits blast radius of failures
- Prevents resource starvation
- Enables independent scaling
- Improves system stability

### Resilience Patterns

| Pattern | Purpose | Implementation |
|---------|---------|-----------------|
| **Retry** | Handle transient failures | Exponential backoff, max retries |
| **Timeout** | Prevent hanging requests | Set max wait time |
| **Fallback** | Provide alternative response | Use cached data, default values |
| **Bulkhead** | Isolate failures | Separate resources, thread pools |
| **Circuit Breaker** | Stop cascading failures | State machine with thresholds |
| **Rate Limiting** | Prevent overload | Token bucket, adaptive limits |

---

## Reliability
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.10.33 PM.png' width=700 />

### Reliability Definition

**Reliability** is the probability that a system will perform its intended function correctly over a specified time period under specified conditions.

- **Formula**: `Reliability = Availability + Correctness + Timeliness`
- **Scope**: Encompasses availability, data correctness, and performance
- **Goal**: System operates correctly and consistently

### Reliability Formula

| ![image](../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot%202026-02-13%20at%201.11.30 PM.png) | ![image](../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot%202026-02-13%20at%201.11.39 PM.png) | ![image](../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot%202026-02-13%20at%201.11.53 PM.png) |
| :---: | :---: | :---: |

```
Reliability = Availability + Correctness + Timeliness

Where:
  • Availability: System is operational and accessible (uptime %)
  • Correctness: System produces accurate results (data integrity)
  • Timeliness: System responds within acceptable time (latency SLA)

Example:
  Availability = 99.9% (3 nines)
  Correctness = 99.99% (data accuracy)
  Timeliness = 99.5% (response time SLA)
  
  Reliability = 0.999 × 0.9999 × 0.995 = 0.9939 (99.39%)
```

### How Reliability is Achieved

#### 1. **Availability**
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.11.30 PM.png' width=350 />

- Minimize downtime through redundancy and failover
- **Techniques**:
  - Multi-region deployment
  - Load balancing
  - Health checks and automatic recovery
  - Graceful degradation

#### 2. **Correctness**
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.11.39 PM.png' width=350 />

- Ensure data integrity and accuracy
- **Techniques**:
  - Data validation and verification
  - Checksums and error detection
  - ACID transactions
  - Data replication with consistency checks
  - Comprehensive testing (unit, integration, end-to-end)

#### 3. **Timeliness**
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.11.53 PM.png' width=350 />

- Meet performance requirements and SLAs
- **Techniques**:
  - Caching strategies
  - Database optimization
  - Load balancing
  - Asynchronous processing
  - Resource provisioning

#### 4. **Monitoring and Observability**
- Continuous monitoring of system health
- **Metrics**:
  - Error rates
  - Response latency
  - Resource utilization
  - Data consistency checks
- **Tools**: Logging, metrics, distributed tracing, alerting

#### 5. **Testing and Validation**
- Comprehensive testing before production
- **Types**:
  - Unit tests (individual components)
  - Integration tests (component interactions)
  - End-to-end tests (full workflows)
  - Load tests (performance under stress)
  - Chaos tests (failure scenarios)

---

## Chaos Engineering
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.17.27 PM.png' width=600 />

### Chaos Engineering Definition

**Chaos Engineering** is the discipline of experimenting on a system to build confidence in its ability to withstand turbulent conditions in production.

- **Core Principle**: Proactively inject failures to discover weaknesses
- **Goal**: Improve system resilience before failures occur in production
- **Approach**: Controlled experiments in production-like environments

### How Chaos Engineering is Achieved

#### 1. **Hypothesis-Driven Experiments**
- Define expected system behavior
- Inject controlled failures
- Observe actual behavior
- Compare with hypothesis

```
┌─────────────────────────────────────────────────┐
│        Chaos Engineering Process                │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Define Hypothesis                           │
│     "System remains available if one DB fails"  │
│                                                 │
│  2. Establish Baseline                          │
│     Measure normal system behavior              │
│                                                 │
│  3. Inject Chaos                                │
│     Kill database instance                      │
│                                                 │
│  4. Observe Impact                              │
│     Monitor metrics, logs, user experience      │
│                                                 │
│  5. Analyze Results                             │
│     Did system behave as expected?              │
│                                                 │
│  6. Remediate Issues                            │
│     Fix discovered weaknesses                   │
│                                                 │
│  7. Repeat                                      │
│     Test more complex failure scenarios         │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### 2. **Failure Injection Techniques**
- **Network Failures**: Latency, packet loss, partition
- **Resource Failures**: CPU spike, memory exhaustion, disk full
- **Service Failures**: Kill processes, crash instances
- **Data Failures**: Corrupt data, introduce inconsistencies
- **Cascading Failures**: Multiple simultaneous failures

#### 3. **Scope and Scale**
- **Blast Radius**: Start small, gradually increase scope
- **Timing**: Run during low-traffic periods initially
- **Monitoring**: Continuous observation during experiments
- **Rollback**: Ability to stop experiment immediately

#### 4. **Automation**
- Automated chaos injection tools
- Scheduled experiments
- Integration with CI/CD pipelines
- Automated remediation

### Chaos Monkey
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.17.58 PM.png' width=600 />

**Chaos Monkey** is a tool developed by Netflix that randomly terminates instances in production to ensure system resilience.

#### Purpose
- Ensure system can handle instance failures
- Prevent dependency on specific instances
- Build confidence in failover mechanisms
- Discover hidden weaknesses

#### How It Works

```
┌──────────────────────────────────────────────────┐
│         Chaos Monkey Operation                   │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. Identify Target                              │
│     Select random instance from production       │
│                                                  │
│  2. Verify Safety                                │
│     Check if termination is safe                 │
│     • Not during critical operations             │
│     • Sufficient healthy instances remain        │
│                                                  │
│  3. Terminate Instance                           │
│     Kill the selected instance                   │
│                                                  │
│  4. Monitor Recovery                             │
│     Observe system behavior                      │
│     • Does traffic reroute?                      │
│     • Do new instances spawn?                    │
│     • Is service still available?                │
│                                                  │
│  5. Log Results                                  │
│     Record what happened                         │
│     • Recovery time                              │
│     • Any errors or issues                       │
│     • User impact                                │
│                                                  │
│  6. Analyze and Improve                          │
│     Fix discovered issues                        │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### Key Features
- **Random Selection**: Unpredictable instance termination
- **Scheduled Execution**: Runs during business hours (Netflix philosophy)
- **Safety Checks**: Prevents terminating too many instances
- **Monitoring Integration**: Tracks system behavior during failures
- **Reporting**: Documents findings and improvements

#### Benefits
- **Confidence**: Proves system handles failures
- **Discovery**: Finds weaknesses before they cause outages
- **Culture**: Builds resilience mindset in teams
- **Automation**: Reduces manual testing burden
- **Continuous Improvement**: Regular testing drives improvements

#### Evolution: Chaos Toolkit
Modern chaos engineering extends beyond Chaos Monkey:
- **Gremlin**: Commercial chaos engineering platform
- **Chaos Mesh**: Kubernetes-native chaos engineering
- **Litmus**: Open-source chaos engineering for Kubernetes
- **AWS FIS**: AWS Fault Injection Simulator

#### Best Practices
- Start with non-critical systems
- Gradually increase complexity of experiments
- Automate chaos injection
- Integrate with monitoring and alerting
- Document findings and improvements
- Build organizational culture around resilience
- Run experiments regularly (weekly, monthly)

---

## Summary

<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.57.03 PM.png' width=700 />

| Concept | Focus | Approach | Goal |
|---------|-------|----------|------|
| **Fault Tolerance** | Component failures | Redundancy, failover | Zero downtime |
| **Resilience** | Unexpected failures | Adaptation, degradation | Acceptable service |
| **Reliability** | Overall correctness | Availability + Correctness + Timeliness | Consistent operation |
| **Chaos Engineering** | Discovering weaknesses | Controlled failure injection | Improved resilience |

**Key Relationships**:
<img src='../../Resources/17-system-architecture-basics/fault-tolerance-resilience-reliability/Screenshot 2026-02-13 at 1.18.55 PM.png' width=500 />
- Fault Tolerance prevents failures
- Resilience handles surprises
- Reliability measures overall system quality
- Chaos Engineering validates resilience

--- 


<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
