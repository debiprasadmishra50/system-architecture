# Event Driven Systems: Saga Pattern

## Table of Contents

1. [Distributed Transactions](#distributed-transactions)
2. [What is Two-Phase Commit (2PC)?](#what-is-two-phase-commit-2pc)
3. [Benefits and Limitations of 2PC](#benefits-and-limitations-of-2pc)
4. [Google Spanner](#google-spanner)
5. [What are Sagas?](#what-are-sagas)
6. [Summary](#summary)

---

## Distributed Transactions

### Why It's Necessary

- In a monolithic application, transactions are straightforward—a single database ensures ACID properties (Atomicity, Consistency, Isolation, Durability). 

- However, in microservices architectures, each service typically maintains its own database. When a business operation spans multiple services, we face a critical challenge:

**The Problem:**
- A single business transaction may require updates across multiple databases
- Network failures, service crashes, or partial failures can leave the system in an inconsistent state
- Traditional database transactions cannot span multiple independent databases
- We need a mechanism to maintain data consistency across service boundaries

**Example Scenario:**
Consider an e-commerce order processing system:
1. Order Service creates an order
2. Payment Service processes payment
3. Inventory Service reserves stock
4. Shipping Service schedules delivery

If the payment fails after the order is created, we need to rollback the order. If inventory reservation fails, we need to refund the payment. Without distributed transaction support, the system could end up in an inconsistent state.

---

## What is Two-Phase Commit (2PC)?

Two-Phase Commit is a distributed algorithm that coordinates all processes participating in a distributed atomic transaction to ensure they either all commit or all abort (rollback).

### How 2PC Works

**Phase 1: Prepare/Voting Phase**
![Phase-1](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%207.50.43 PM.png)
1. The coordinator (transaction manager) sends a "prepare to commit" request to all participants
2. Each participant:
   - Executes the transaction up to the point of commit
   - Locks the required resources
   - Performs validation checks
   - Responds with "YES" (ready to commit) or "NO" (cannot commit)
3. The coordinator collects all votes

**Phase 2: Commit/Abort Phase**
![Phase-2](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%207.50.55 PM.png)
1. If all participants voted "YES":
   - Coordinator sends "commit" command to all participants
   - Each participant commits the transaction and releases locks
   - Participants acknowledge the commit
2. If any participant voted "NO":
   - Coordinator sends "abort" command to all participants
   - Each participant rolls back the transaction and releases locks
   - Participants acknowledge the rollback

### Example Flow

```
Coordinator                Participant A          Participant B
    |                           |                      |
    |----Prepare Request------->|                      |
    |                           |                      |
    |                    [Lock Resources]              |
    |                    [Validate]                    |
    |<----YES/NO Vote-----------|                      |
    |                                                  |
    |----Prepare Request------------------------------>|
    |                                          [Lock Resources]
    |                                          [Validate]
    |<----YES/NO Vote----------------------------------|
    |                                                  |
    |----Commit/Abort Command-->|                      |
    |                    [Commit/Rollback]             |
    |<----Acknowledgement-------|                      |
    |                                                  |
    |----Commit/Abort Command------------------------->|
    |                                          [Commit/Rollback]
    |<----Acknowledgement------------------------------|
```

---

## Benefits and Limitations of 2PC

### Benefits

1. **Strong Consistency**: Guarantees ACID properties across distributed systems
2. **Atomicity**: All-or-nothing semantics—either all services commit or all rollback
3. **Simplicity**: Conceptually straightforward to understand and implement
4. **Proven Technology**: Well-established protocol with mature implementations

### Limitations

1. **Blocking Protocol**: Resources remain locked during both phases, reducing concurrency
2. **Performance Overhead**: Multiple round-trips between coordinator and participants increase latency
3. **Availability Issues**: If the coordinator fails, participants remain blocked indefinitely
4. **Network Dependency**: Requires reliable network communication; network partitions can cause deadlocks
5. **Scalability Challenges**: Doesn't scale well with many participants or high-latency networks
6. **Not Suitable for Microservices**: Violates microservices principles of loose coupling and independent deployment
7. **Synchronous Nature**: Tightly couples services, making them dependent on each other's availability

### Why 2PC Fails in Modern Distributed Systems

- **CAP Theorem**: In the presence of network partitions, 2PC cannot guarantee both consistency and availability
- **Microservices Autonomy**: Services should be independently deployable and operable
- **Cloud-Native Environments**: Frequent failures and network issues make 2PC impractical

---

## Google Spanner

Google Spanner is a globally distributed, horizontally scalable relational database that provides strong consistency without the limitations of traditional 2PC.

![Google Spanner](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%207.53.58 PM.png)
### Key Features

1. **Global Distribution**: Data is replicated across multiple data centers worldwide
2. **Strong Consistency**: Provides ACID transactions across geographically distributed data
3. **Synchronization**: Uses TrueTime (atomic clocks) to maintain consistent timestamps across data centers
4. **Automatic Sharding**: Horizontally scales by automatically partitioning data
5. **High Availability**: Replication ensures data survives data center failures

### How Spanner Achieves Strong Consistency

**TrueTime API:**
- Uses atomic clocks and GPS receivers in Google data centers
- Provides timestamps with bounded uncertainty
- Enables causally consistent ordering of transactions across the globe

**Paxos Consensus:**
- Uses Paxos protocol for replication
- Ensures all replicas agree on transaction ordering
- Provides fault tolerance

### Advantages Over 2PC

1. **No Blocking**: Doesn't lock resources for extended periods
2. **Better Performance**: Optimized for distributed scenarios
3. **Automatic Failover**: Handles data center failures transparently
4. **Scalability**: Designed for global scale

### Limitations

1. **Cost**: Expensive to operate and maintain
2. **Complexity**: Requires specialized infrastructure (atomic clocks)
3. **Not Open Source**: Proprietary Google technology
4. **Overkill for Many Use Cases**: Most applications don't need global strong consistency

### When to Use Spanner

- Global applications requiring strong consistency
- Financial systems with strict consistency requirements
- Applications where data center failures are unacceptable
- Organizations with resources to operate and maintain it

---

## What are Sagas?

- Sagas are a pattern for managing distributed transactions in microservices architectures. 
- Instead of using 2PC's blocking approach, sagas `break a distributed transaction into a series of local transactions, each updating a single service's database.`

### Core Concept

A saga is a sequence of local transactions where:

    1. Each step is a local transaction in a single service
    2. Steps are coordinated through asynchronous messaging or orchestration
    3. If a step fails, compensating transactions are triggered to undo previous steps
    4. The system eventually reaches a consistent state

![Concept](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%208.09.01 PM.png)

### Two Implementation Patterns

#### 1. Choreography-Based Sagas
![Choreography](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%208.14.22 PM.png)
Services communicate through events. Each service listens for events and publishes new events based on local transaction results.

**Flow:**
```
Order Service                Payment Service              Inventory Service
    |                             |                            |
    |--Order Created Event------->|                            |
    |                             |                            |
    |                    [Process Payment]                     |
    |                             |                            |
    |<--Payment Processed Event---|                            |
    |                                                          |
    |--Reserve Stock Event------------------------------------>|
    |                                          [Reserve Stock]
    |<--Stock Reserved Event-----------------------------------|
    |
[Order Confirmed]
```

**Advantages:**
- Loose coupling between services
- No central coordinator needed
- Services are independent

**Disadvantages:**
- Complex to understand and debug
- Difficult to track saga progress
- Event chains can become convoluted
- Hard to implement compensating transactions

#### 2. Orchestration-Based Sagas
![Orchestration](../../Resources/03-saga-pattern/Screenshot%202026-02-06%20at%208.11.20 PM.png)
A central orchestrator (saga coordinator) directs each service on what to do. Services don't communicate directly; they respond to orchestrator commands.

**Flow:**
```
Saga Orchestrator
    |
    |--Create Order Command------->Order Service
    |                                    |
    |<--Order Created Response-----------|
    |
    |--Process Payment Command------->Payment Service
    |                                    |
    |<--Payment Processed Response-------|
    |
    |--Reserve Stock Command------->Inventory Service
    |                                    |
    |<--Stock Reserved Response----------|
    |
[Saga Completed]
```

**Advantages:**
- Centralized control and visibility
- Easier to understand and debug
- Simpler to implement compensating transactions
- Better error handling and recovery

**Disadvantages:**
- Central orchestrator becomes a potential bottleneck
- Tighter coupling to the orchestrator
- Orchestrator must handle all failure scenarios

### Comparison between Orchestration-Saga vs Choreography-Saga

| Aspect | Orchestration-Based | Choreography-Based |
|--------|--------------------|--------------------|
| **Architecture** | Centralized coordinator directs all steps | Distributed, services communicate via events |
| **Coupling** | Tighter coupling to orchestrator | Loose coupling between services |
| **Visibility** | Centralized view of saga progress | Distributed, harder to track |
| **Complexity** | Easier to understand and implement | Complex event chains |
| **Failure Handling** | Orchestrator manages all compensations | Services handle their own compensations |
| **Scalability** | Orchestrator can become a bottleneck | Better horizontal scalability |
| **Debugging** | Easier to debug and trace execution | Difficult to debug event chains |
| **Performance** | Synchronous request-response pattern | Asynchronous event-driven |
| **Latency** | Higher due to orchestrator coordination | Lower, parallel event processing |
| **Deployment** | Services tightly coupled to orchestrator | Services independently deployable |
| **Testing** | Easier to test with mocked orchestrator | Complex to test event interactions |
| **Monitoring** | Centralized monitoring of saga state | Distributed monitoring required |
| **Compensation Logic** | Centralized in orchestrator | Distributed across services |
| **Event Ordering** | Guaranteed by orchestrator | Must handle out-of-order events |
| **Use Case** | Complex workflows with many steps | Simple, independent service interactions |


### Compensating Transactions

When a saga step fails, compensating transactions are executed in reverse order to undo previous steps.

**Example:**
```
Step 1: Create Order ✓
Step 2: Process Payment ✓
Step 3: Reserve Stock ✗ (FAILS)

Compensation:
Step 2: Refund Payment
Step 1: Cancel Order
```

### Saga States and Transitions

```
[Start]
   |
   v
[Executing Step 1]
   |
   +--Success--> [Executing Step 2]
   |                   |
   |                   +--Success--> [Executing Step 3]
   |                   |                   |
   |                   |                   +--Success--> [Saga Completed]
   |                   |                   |
   |                   |                   +--Failure--> [Compensating]
   |                   |
   |                   +--Failure--> [Compensating]
   |
   +--Failure--> [Compensating]
                      |
                      v
                [Saga Failed]
```

### Saga vs 2PC Comparison

| Aspect | 2PC | Saga |
|--------|-----|------|
| **Consistency** | Strong | Eventual |
| **Blocking** | Yes (locks resources) | No (asynchronous) |
| **Latency** | High | Lower |
| **Scalability** | Poor | Good |
| **Complexity** | Simple | Complex |
| **Failure Handling** | Automatic rollback | Manual compensation |
| **Network Dependency** | High | Lower |
| **Microservices Friendly** | No | Yes |

### Similarity and Differences between Orchestrated Saga and 2PC

| Aspect | Orchestrated Saga | 2PC |
|--------|-------------------|-----|
| **Coordination Model** | Centralized orchestrator directs steps | Centralized coordinator manages phases |
| **Consistency Guarantee** | Eventual consistency | Strong consistency (ACID) |
| **Blocking Behavior** | Non-blocking, asynchronous | Blocking, synchronous |
| **Resource Locking** | No locks held during saga | Resources locked during both phases |
| **Latency** | Lower, parallel processing possible | Higher, sequential round-trips |
| **Scalability** | Better for microservices | Poor, doesn't scale well |
| **Network Resilience** | Tolerates network delays | Sensitive to network failures |
| **Failure Handling** | Compensating transactions | Automatic rollback |
| **Complexity** | More complex to implement | Simpler conceptually |
| **Visibility** | Centralized saga state tracking | Implicit transaction state |
| **Compensation Logic** | Explicit, must be coded | Implicit, automatic |
| **Idempotency Requirement** | Required for retries | Not required |
| **Partial Failure Handling** | Explicit compensation needed | Automatic rollback |
| **Service Coupling** | Tighter to orchestrator | Tighter to transaction manager |
| **Microservices Suitability** | Excellent | Poor |
| **Long-Running Transactions** | Suitable | Not suitable |
| **Data Consistency Window** | Temporary inconsistency possible | Always consistent |
| **Deployment Independence** | Services depend on orchestrator | Services depend on coordinator |
| **Monitoring Complexity** | Requires saga-specific monitoring | Standard transaction monitoring |
| **Use Case** | Modern microservices | Legacy monolithic systems |



### When to Use Sagas

1. **Microservices Architectures**: Distributed transactions across multiple services
2. **Eventual Consistency is Acceptable**: When strong consistency isn't required
3. **Long-Running Transactions**: Transactions that span minutes or hours
4. **High-Throughput Systems**: Where blocking would be problematic
5. **Autonomous Services**: When services need to be independently deployable

### Challenges with Sagas

1. **Eventual Consistency**: Data may be temporarily inconsistent
2. **Compensating Transactions**: Must be carefully designed and tested
3. **Idempotency**: Operations must be idempotent to handle retries
4. **Monitoring and Debugging**: Harder to track saga execution across services
5. **Partial Failures**: Handling edge cases where compensation itself fails

### Best Practices for Implementing Sagas

1. **Make Operations Idempotent**: Ensure operations can be safely retried
2. **Design Compensating Transactions**: Plan rollback logic upfront
3. **Use Unique Request IDs**: Track requests across service boundaries
4. **Implement Timeouts**: Prevent indefinite waiting for responses
5. **Log Everything**: Maintain detailed logs for debugging
6. **Use Dead Letter Queues**: Handle messages that can't be processed
7. **Monitor Saga Progress**: Track saga execution and failures
8. **Test Failure Scenarios**: Thoroughly test compensation logic

---

## Summary

- **Distributed Transactions** are necessary in microservices to maintain consistency across service boundaries
- **2PC** provides strong consistency but is impractical for modern distributed systems due to blocking and availability issues
- **Google Spanner** offers a proprietary solution with strong consistency at global scale
- **Sagas** are the recommended pattern for microservices, trading strong consistency for availability and scalability through eventual consistency and compensating transactions
- Choose between **choreography** (loose coupling) and **orchestration** (centralized control) based on your specific requirements

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
