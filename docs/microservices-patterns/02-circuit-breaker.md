# Event Driven Systems: Circuit Breaker

## Table of Contents

1. [Overview](#overview)
2. [Faults and Their Protection Strategies](#faults-and-their-protection-strategies)
   - [Cascading Failures → Overload Protection](#cascading-failures--overload-protection)
   - [Transient Faults → Retry Pattern](#transient-faults--retry-pattern-as-resiliency-strategy)
   - [Persistent Faults → Failing Quickly](#persistent-faults--failing-quickly-for-high-concurrent-systems)
3. [State Transition Diagram](#state-transition-diagram)
4. [Configuration Parameters](#configuration-parameters)
5. [Implementation Patterns](#implementation-patterns)
6. [Benefits](#benefits)
7. [Drawbacks and Considerations](#drawbacks-and-considerations)
8. [Real-World Examples](#real-world-examples)
9. [Integration with Event-Driven Systems](#integration-with-event-driven-systems)
10. [Popular Implementations](#popular-implementations)
11. [Best Practices](#best-practices)
12. [Comparison with Other Patterns](#comparison-with-other-patterns)
13. [Conclusion](#conclusion)

---

## Overview

The **Circuit Breaker** pattern is a critical design pattern used in distributed systems and microservices to _**prevent cascading failures**_. It acts as a protective mechanism that stops requests to failing services, allowing them time to recover while preventing resource exhaustion in the calling service.

The pattern is inspired by electrical circuit breakers, which automatically interrupt the flow of electricity when a fault is detected.

---

## Faults and Their Protection Strategies

### Cascading Failures → Overload Protection

**Definition:** Failure in one service causes failures in dependent services, propagating through the system.

**Real-World Example: Netflix Outage (2012)**
- AWS region experiences network latency
- Video streaming service becomes slow
- API Gateway exhausts thread pool waiting for responses
- User authentication service becomes unavailable
- Entire Netflix platform fails

**Solution:** Circuit Breaker Pattern
- Detect failures early and stop sending requests
- Return fast-fail response immediately
- Allow failing service time to recover

---

### Transient Faults → Retry Pattern as Resiliency Strategy

**Definition:** Temporary failures that resolve themselves within seconds (network hiccup, brief service restart, resource contention).

**Real-World Example: AWS Network Blip**
- Payment gateway experiences 2-second network latency spike
- First request times out
- Service automatically retries after 100ms
- Second request succeeds
- Transaction completes without user noticing

**Solution:** Exponential Backoff with Jitter
- Retry failed requests with increasing delays
- Add randomness to prevent thundering herd
- Set maximum retry attempts (3-5)
- Only retry on transient errors (timeouts, 5xx)

---

### Persistent Faults → Failing Quickly for High Concurrent Systems

**Definition:** Long-lasting failures that won't resolve on their own (service crash, database corruption, deployment bug).

**Real-World Example: Kubernetes Pod Crash Loop**
- New deployment has memory leak bug
- Pod crashes after 5 minutes
- Kubernetes restarts pod
- Pod crashes again after 5 minutes
- Requests keep failing repeatedly
- System becomes overloaded with retry attempts

**Solution:** Fail Quickly Strategy
- Detect persistent failures quickly
- Stop retrying immediately
- Return error to caller
- Preserve system resources
- Allow other requests to proceed

**Configuration:**
- Timeout: 2-5 seconds (not 30s)
- Failure threshold: 2-5 consecutive failures
- Circuit open duration: 30 seconds
- Bulkhead: Separate thread pools per service

---

## State Transition Diagram
![Circuit Breaker](../../Resources/02-circuit-breaker/Screenshot%202026-02-06%20at%201.39.10 PM.png)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  CLOSED                                                     │
│  ├─ Requests pass through                                   │
│  ├─ Monitor success/failure rate                            │
│  └─ Transition to OPEN when failures exceed threshold       │
│         │                                                   │
│         │ (Failure threshold exceeded)                      │
│         ↓                                                   │
│  OPEN                                                       │
│  ├─ Requests fail immediately (fast-fail)                   │
│  ├─ No calls to downstream service                          │
│  └─ Transition to HALF-OPEN after timeout                   │
│         │                                                   │
│         │ (Timeout elapsed)                                 │
│         ↓                                                   │
│  HALF-OPEN                                                  │
│  ├─ Limited test requests allowed                           │
│  ├─ Monitor test request results                            │
│  └─ Transition based on test results                        │
│         │                                                   │
│         ├─ (Test requests succeed) → CLOSED                 │
│         │                                                   │
│         └─ (Test requests fail) → OPEN                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

The Circuit Breaker pattern introduces a state machine with three states:
### 1. **CLOSED State** (Normal Operation)
- Requests flow normally to the downstream service.
- The circuit breaker monitors the success/failure rate.
- If failures exceed a threshold, the circuit opens.
**Characteristics:**
- All requests pass through to the service
- Failures are counted
- Success rate is monitored
- Transition to OPEN when failure threshold is exceeded
### 2. **OPEN State** (Failure Detected)
- Requests are immediately rejected without calling the downstream service.
- A fast-fail response is returned to the caller.
- The downstream service is given time to recover.
- After a timeout period, the circuit transitions to HALF-OPEN.
**Characteristics:**
- Requests fail immediately (fast-fail)
- No calls to the downstream service
- Prevents resource exhaustion
- Allows the failing service to recover
- Transition to HALF-OPEN after timeout
### 3. **HALF-OPEN State** (Testing Recovery)
- A limited number of test requests are allowed through.
- If these requests succeed, the circuit closes (service recovered).
- If these requests fail, the circuit opens again (service still failing).
**Characteristics:**
- Limited requests allowed (e.g., 1-5 requests)
- Tests if the service has recovered
- Transition to CLOSED if successful
- Transition to OPEN if failed
---

## Configuration Parameters
### Key Parameters
1. **Failure Threshold**
   - Number or percentage of failures that trigger the OPEN state
   - Example: 5 consecutive failures or 50% failure rate in last 10 requests
2. **Success Threshold**
   - Number of successful requests in HALF-OPEN state to close the circuit
   - Example: 2 consecutive successful requests
3. **Timeout Duration**
   - Time to wait in OPEN state before transitioning to HALF-OPEN
   - Example: 30 seconds, 1 minute, 5 minutes
4. **Window Size**
   - Number of requests to consider for calculating failure rate
   - Example: Last 10 requests, last 100 requests
5. **Failure Types**
   - Which exceptions/errors trigger the circuit breaker
   - Example: Timeout, Connection Refused, HTTP 5xx errors

### Example Configuration
```yaml
CircuitBreaker:
  failureThreshold: 5              # Open after 5 consecutive failures
  successThreshold: 2              # Close after 2 successful requests in HALF-OPEN
  timeout: 30000                   # Wait 30 seconds before trying again (milliseconds)
  windowSize: 10                   # Consider last 10 requests
  failurePercentageThreshold: 50   # Open if 50% of requests fail
  slowCallDurationThreshold: 2000  # Consider calls > 2 seconds as slow
  slowCallRateThreshold: 50        # Open if 50% of calls are slow
```

## Implementation Patterns
### Pattern 1: Count-Based Threshold
Opens the circuit after N consecutive failures.
```
Requests: [Success, Success, Failure, Failure, Failure, Failure, Failure]
                                                                    ↑
                                                    Circuit Opens (5 failures)
```
### Pattern 2: Percentage-Based Threshold
Opens the circuit when failure percentage exceeds threshold in a window.
```
Window: [Success, Failure, Success, Failure, Failure, Success, Failure, Failure]
        Failure Rate: 50% (4 failures out of 8 requests)
        If threshold is 50%, circuit opens
```
### Pattern 3: Slow Call Detection
Opens the circuit when slow calls exceed threshold.
```
Requests: [100ms, 150ms, 2500ms, 2800ms, 2600ms, 3000ms]
                          ↑
                    Slow calls detected (> 2000ms threshold)
                    If 50% are slow, circuit opens
```
---

## Benefits

1. **Prevents Cascading Failures**
   - Stops propagation of failures through the system
   - Protects upstream services from being overwhelmed

2. **Fast Failure**
   - Returns errors immediately instead of waiting for timeouts
   - Improves user experience with quick feedback

3. **Resource Protection**
   - Prevents exhaustion of threads, connections, and memory
   - Allows resources to be used for healthy services

4. **Automatic Recovery**
   - Gives failing services time to recover
   - Automatically retries when service is healthy

5. **Observability**
   - Provides visibility into service health
   - Enables monitoring and alerting on circuit state changes

6. **Graceful Degradation**
   - Can return cached responses or default values
   - Maintains partial functionality during outages

---

## Drawbacks and Considerations

1. **Complexity**
   - Adds complexity to the codebase
   - Requires careful configuration tuning

2. **Configuration Tuning**
   - Threshold values must be carefully chosen
   - Too aggressive: False positives (circuit opens unnecessarily)
   - Too lenient: Doesn't prevent cascading failures

3. **Fallback Strategy**
   - Must decide what to do when circuit is open
   - Options: Return cached data, default value, error message

4. **Monitoring Required**
   - Need to monitor circuit state changes
   - Need to alert on repeated circuit openings

5. **Testing Complexity**
   - Difficult to test all state transitions
   - Requires simulating failures

---

## Real-World Examples

### Example 1: E-Commerce Order Service

```
Order Service → Payment Service (Circuit Breaker)

Scenario:
1. Payment Service becomes slow (network issue)
2. Circuit Breaker detects high latency
3. Circuit opens after 5 slow requests
4. Order Service immediately returns error
5. User sees "Payment service temporarily unavailable"
6. After 30 seconds, circuit tries again (HALF-OPEN)
7. Payment Service has recovered
8. Circuit closes, normal operation resumes
```

### Example 2: Microservices Chain

```
API Gateway → Service A → Service B → Service C

Scenario:
1. Service C fails
2. Service B's circuit breaker to Service C opens
3. Service B returns cached data or error
4. Service A continues to function
5. API Gateway returns partial response
6. System degrades gracefully instead of cascading failure
```

---

## Integration with Event-Driven Systems

In event-driven architectures, the Circuit Breaker pattern works alongside other patterns:

### With Saga Pattern
- Circuit breaker protects saga steps
- If a step fails repeatedly, circuit opens
- Saga can handle the failure and trigger compensating transactions

### With Retry Pattern
- Retry attempts before circuit opens
- Circuit breaker prevents infinite retries
- Exponential backoff + circuit breaker = robust resilience

### With Bulkhead Pattern
- Bulkhead isolates resources
- Circuit breaker prevents resource exhaustion
- Together they provide defense in depth

### With Event Notification
- Circuit breaker status can be published as events
- Other services can react to circuit state changes
- Enables coordinated failure handling

---

## Popular Implementations

### Java/Spring
- **Resilience4j** - Lightweight, functional library
- **Hystrix** - Netflix's circuit breaker (legacy)
- **Spring Cloud Circuit Breaker** - Spring's abstraction

### Node.js
- **opossum** - Lightweight circuit breaker
- **node-circuit-breaker** - Simple implementation

### Python
- **pybreaker** - Simple circuit breaker
- **tenacity** - Retry library with circuit breaker support

### Go
- **gobreaker** - Simple circuit breaker
- **grpc-go** - Built-in circuit breaker support

---

## Best Practices

1. **Set Appropriate Thresholds**
   - Start conservative, adjust based on monitoring
   - Consider service SLA and acceptable failure rate

2. **Implement Fallback Strategies**
   - Return cached data when possible
   - Provide meaningful error messages
   - Consider default values for non-critical operations

3. **Monitor Circuit State**
   - Log state transitions
   - Alert on repeated circuit openings
   - Track metrics: open count, duration, recovery time

4. **Combine with Other Patterns**
   - Use with retry pattern for transient failures
   - Use with bulkhead for resource isolation
   - Use with timeout for bounded waiting

5. **Test Thoroughly**
   - Test all state transitions
   - Simulate failures and recovery
   - Test fallback behavior

6. **Document Configuration**
   - Document why thresholds were chosen
   - Include rationale for timeout values
   - Make it easy to adjust in production

---

## Comparison with Other Patterns

| Pattern | Purpose | When to Use |
|---------|---------|------------|
| **Circuit Breaker** | Prevent cascading failures | When calling external services |
| **Retry** | Handle transient failures | For temporary issues |
| **Timeout** | Bound waiting time | Always use with external calls |
| **Bulkhead** | Isolate resources | Prevent resource exhaustion |
| **Fallback** | Provide alternative response | When graceful degradation is needed |

---

## Conclusion

The Circuit Breaker pattern is essential for building resilient microservices and event-driven systems. By preventing cascading failures and enabling automatic recovery, it helps maintain system stability and user experience even when individual services fail.

Key takeaways:
- **Three states:** CLOSED (normal), OPEN (failing), HALF-OPEN (testing)
- **Fast failure:** Prevents resource exhaustion and timeouts
- **Automatic recovery:** Gives services time to recover
- **Requires monitoring:** Track state changes and adjust thresholds
- **Combine with other patterns:** Use with retry, timeout, bulkhead, and fallback

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
