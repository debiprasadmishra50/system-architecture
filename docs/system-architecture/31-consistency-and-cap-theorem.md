# Consistency and CAP Theorem

## Table of Contents

1. [Consistency: What It Is](#consistency-what-it-is)
2. [The Consistency Spectrum](#the-consistency-spectrum)
   - [Strong Consistency](#strong-consistency)
   - [Eventual Consistency](#eventual-consistency)
3. [Consistency Models: Applications and Industries](#consistency-models-applications-and-industries)
4. [Benefits and Tradeoffs](#benefits-and-tradeoffs)
5. [Network Partition: The Problem](#network-partition-the-problem)
   - [What Is a Network Partition?](#what-is-a-network-partition)
   - [The Choice at Hand: CP vs CA](#the-choice-at-hand-cp-vs-ca)
6. [CAP Theorem and System Choices](#cap-theorem-and-system-choices)
   - [CAP Theorem Definition](#cap-theorem-definition)
   - [Why Partition Tolerance Is Non-Negotiable](#why-partition-tolerance-is-non-negotiable)
7. [Architecture Reshaping: CA vs CP Systems](#architecture-reshaping-ca-vs-cp-systems)
   - [Database Selection](#database-selection)
   - [Caching Strategy](#caching-strategy)
   - [Replication Flow](#replication-flow)
   - [Conflict Resolution](#conflict-resolution)
8. [Handling Concurrent User Data Updates in Eventual Consistency](#handling-concurrent-user-data-updates-in-eventual-consistency)
   - [Scenario: Two Users Updating Same Record](#scenario-two-users-updating-same-record)
   - [Best Practices for Concurrent Updates](#best-practices-for-concurrent-updates)
9. [Summary](#summary)

---

## Consistency: What It Is

**Definition**: Consistency ensures that all nodes in a distributed system see the same data at the same time. It guarantees that once data is written, all subsequent reads return that written value.

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 12.55.04 PM.png' width='700' />

- **Single Source of Truth**: All replicas maintain identical state
- **Synchronization**: Updates propagate to all nodes before acknowledging success
- **Predictability**: Clients always see the most recent data
- **Trade-off**: Requires coordination overhead, reducing availability and performance

---

## The Consistency Spectrum

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 12.56.01 PM.png' width='700' />

### Strong Consistency

**Definition**: All reads return the most recent write; all nodes are synchronized before acknowledging updates.

- **Characteristics**:
  - Immediate visibility of updates across all nodes
  - Requires blocking writes until all replicas acknowledge
  - Highest coordination overhead
  - Simplest mental model for developers

- **Implementation**:
  - Synchronous replication
  - Two-phase commit (2PC)
  - Consensus algorithms (Raft, Paxos)

- **Example**: Banking transactions, inventory management

### Eventual Consistency

**Definition**: Nodes may temporarily diverge, but will converge to the same state given sufficient time without new updates.

- **Characteristics**:
  - Asynchronous replication
  - Writes acknowledged immediately
  - Temporary inconsistency window
  - Higher availability and performance

- **Implementation**:
  - Asynchronous replication
  - Conflict-free replicated data types (CRDTs)
  - Last-write-wins (LWW)
  - Vector clocks

- **Example**: Social media feeds, DNS caches, distributed caches

---

## Consistency Models: Applications and Industries

| Consistency Model | Industries | Use Cases | Characteristics |
|---|---|---|---|
| **Strong Consistency** | Finance, Healthcare, E-commerce | Bank transfers, Medical records, Order processing | Immediate sync, blocking writes, high latency |
| **Eventual Consistency** | Social Media, Content Delivery, Analytics | User feeds, Cache updates, Log aggregation | Async replication, low latency, temporary divergence |
| **Causal Consistency** | Messaging, Collaboration | Chat systems, Document editing | Respects causality, moderate latency |
| **Read-Your-Writes** | User-facing apps | Profile updates, Settings changes | User sees own writes immediately, others see eventually |
| **Monotonic Reads** | Distributed caches | Session data, User preferences | No rollback of reads, eventual convergence |

---

## Benefits and Tradeoffs

| Aspect | Strong Consistency | Eventual Consistency |
|---|---|---|
| **Data Accuracy** | ✅ Always correct | ⚠️ Temporarily stale |
| **Availability** | ❌ Lower (requires coordination) | ✅ Higher (no blocking) |
| **Latency** | ❌ Higher (sync replication) | ✅ Lower (async replication) |
| **Partition Tolerance** | ❌ Fails during partitions | ✅ Continues operating |
| **Complexity** | ✅ Simpler logic | ❌ Complex conflict resolution |
| **Scalability** | ❌ Limited (coordination bottleneck) | ✅ Better (independent nodes) |
| **User Experience** | ✅ Predictable | ⚠️ May see stale data |
| **Cost** | ❌ Higher (more coordination) | ✅ Lower (less overhead) |

---

## Network Partition: The Problem

### What Is a Network Partition?

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.02.15 PM.png' width='700' />

A network partition occurs when nodes in a distributed system cannot communicate with each other due to network failure, creating isolated groups that cannot exchange messages.

```
Before Partition:
┌─────────────┐         ┌─────────────┐
│   Node A    │◄───────►│   Node B    │
└─────────────┘         └─────────────┘
   (Replica 1)            (Replica 2)

During Partition:
┌─────────────┐   ✗✗✗   ┌─────────────┐
│   Node A    │         │   Node B    │
└─────────────┘         └─────────────┘
   (Isolated)             (Isolated)
```

### The Choice at Hand: CP vs CA

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.03.35 PM.png' width='700' />

When a network partition occurs, systems must choose:

- **Consistency \((C\))**: All nodes see the same data
- **Availability (A)**: System remains operational
- **Partition Tolerance (P)**: System continues despite network failures

**The Dilemma**: You cannot have all three. During a partition, you must choose:

| Choice | Behavior | Trade-off |
|---|---|---|
| **CP (Consistency + Partition Tolerance)** | Block writes, maintain consistency | Reduced availability, timeouts |
| **CA (Consistency + Availability)** | Assume no partitions, sync all nodes | Cannot tolerate network failures |
| **AP (Availability + Partition Tolerance)** | Accept inconsistency, keep operating | Temporary data divergence |

---

## CAP Theorem and System Choices

### CAP Theorem Definition

**Brewer's Theorem**: In a distributed system, you can guarantee at most two of three properties: Consistency, Availability, and Partition Tolerance.

```
        ┌─────────────────────────────────┐
        │      Distributed System         │
        └─────────────────────────────────┘
                    /    |    \
                   /     |     \
                  C      A      P
              (Consistency) (Availability) (Partition Tolerance)
              
    Pick 2:
    ├─ CA: No partition tolerance (single datacenter)
    ├─ CP: Sacrifices availability (blocks during partition)
    └─ AP: Sacrifices consistency (eventual consistency)
```

### Why Partition Tolerance Is Non-Negotiable

In modern distributed systems, **P (Partition Tolerance) is mandatory** because:

- Network failures are inevitable
- Distributed systems span multiple nodes/datacenters
- You cannot prevent partitions, only handle them

**Therefore, the real choice is between C and A**:

    CP Systems: Prioritize consistency over availability

    OR

    AP Systems: Prioritize availability over consistency

---

## Architecture Reshaping: CA vs CP Systems

### Database Selection

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.06.38 PM.png' width='500' />

#### CP Systems (Consistency-Preferred)

- **Examples**: PostgreSQL (with synchronous replication), MongoDB (with strong consistency), HBase
- **Approach**: 
  - Synchronous replication to all replicas
  - Consensus-based leader election
  - Blocking writes until quorum acknowledges
- **When to use**: Financial transactions, inventory, critical data

#### AP Systems (Availability-Preferred)

- **Examples**: Cassandra, DynamoDB, Riak
- **Approach**:
  - Asynchronous replication
  - Quorum reads/writes (eventual consistency)
  - Conflict resolution mechanisms
- **When to use**: Social feeds, caches, analytics, user sessions

---

### Caching Strategy

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.08.39 PM.png' width='500' />

#### CP Systems

```
Client Request
    ↓
┌─────────────────────────────────┐
│  Check Primary Cache            │
│  (Must be consistent)           │
└─────────────────────────────────┘
    ↓
  Hit? → Return (Consistent)
    ↓
  Miss? → Query Database (Sync)
    ↓
  Update Cache (Synchronously)
```

- **Cache Invalidation**: Immediate, synchronous
- **TTL**: Short or event-driven
- **Consistency**: Cache always reflects database state
- **Trade-off**: Higher latency, more cache misses

#### AP Systems

```
Client Request
    ↓
┌─────────────────────────────────┐
│  Check Local Cache              │
│  (May be stale)                 │
└─────────────────────────────────┘
    ↓
  Hit? → Return (May be stale)
    ↓
  Miss? → Query Database (Async)
    ↓
  Update Cache (Asynchronously)
```

- **Cache Invalidation**: Eventual, asynchronous
- **TTL**: Longer, time-based expiration
- **Consistency**: Cache may lag behind database
- **Trade-off**: Lower latency, better availability

---

### Replication Flow

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.08.52 PM.png' width='500' />

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.10.22 PM.png' width='500' />

#### CP Systems (Synchronous Replication)

```
Write Request
    ↓
┌──────────────────────────────────┐
│  Primary Node                    │
│  ├─ Write to local storage       │
│  └─ Send to all replicas         │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  Wait for ACK from all replicas  │
│  (Blocking)                      │
└──────────────────────────────────┘
    ↓
  All ACK? → Commit & Return Success
    ↓
  Timeout? → Rollback & Return Error
```

- **Latency**: High (waits for slowest replica)
- **Availability**: Lower (fails if any replica is down)
- **Consistency**: Guaranteed

#### AP Systems (Asynchronous Replication)

```
Write Request
    ↓
┌──────────────────────────────────┐
│  Primary Node                    │
│  ├─ Write to local storage       │
│  └─ Return Success Immediately   │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  Replicate to other nodes        │
│  (Background, asynchronous)      │
└──────────────────────────────────┘
    ↓
  Replicas eventually converge
```

- **Latency**: Low (returns immediately)
- **Availability**: Higher (continues despite replica failures)
- **Consistency**: Eventual (temporary divergence)

---

### Conflict Resolution

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.10.56 PM.png' width='500' />

When using eventual consistency, concurrent updates to the same data can create conflicts. Multiple strategies exist:

#### Last-Write-Wins (LWW)

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.11.55 PM.png' width='500' />

**Approach**: Keep the write with the latest timestamp, discard others.

```
Timeline:
T1: Node A writes value="Alice" (timestamp: 1000)
T2: Node B writes value="Bob"   (timestamp: 2000)
    ↓
Conflict detected at merge
    ↓
LWW: Keep "Bob" (timestamp 2000 > 1000)
```

- **Pros**: Simple, deterministic, no coordination needed
- **Cons**: Data loss (earlier writes discarded), timestamp dependency
- **Use Cases**: User profiles, cache updates, non-critical data
- **Example**: DynamoDB, Cassandra (default)

#### Vector Clocks

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.12.43 PM.png' width='500' />

**Approach**: Track causal relationships between writes using logical clocks.

```
Initial state: [A:0, B:0]

Node A writes: [A:1, B:0]
Node B writes: [A:0, B:1]

Concurrent writes detected (neither happened before the other)
→ Conflict requires application-level resolution
```

- **Pros**: Detects concurrent writes, preserves causality
- **Cons**: Overhead (clock vector per object), complex implementation
- **Use Cases**: Distributed databases, version control systems
- **Example**: Riak, Dynamo

#### Conflict-Free Replicated Data Types (CRDTs)

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.14.00 PM.png' width='500' />

**Approach**: Use mathematically designed data structures that automatically resolve conflicts.

```
Node A: Add "Alice" to set
  Set = {Alice}

Node B: Add "Bob" to set
  Set = {Bob}

Merge: Union of both sets
  Set = {Alice, Bob}
  (No conflict, automatically resolved)
```

- **Pros**: Automatic conflict resolution, no data loss, strong eventual consistency
- **Cons**: Limited data types, memory overhead
- **Use Cases**: Collaborative editing, real-time applications
- **Example**: Yjs, Automerge, Riak

#### Application-Level Resolution

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.14.37 PM.png' width='500' />

**Approach**: Let the application decide how to resolve conflicts.

```
Conflict detected:
  Version A: {name: "Alice", age: 25}
  Version B: {name: "Bob", age: 30}

Application logic:
  if (timestamp_A > timestamp_B) {
    use Version A
  } else if (user_preference == "manual") {
    show both versions to user
  } else {
    merge fields intelligently
  }
```

- **Pros**: Maximum flexibility, domain-specific logic
- **Cons**: Complex, requires careful implementation
- **Use Cases**: Complex business logic, user-facing conflicts
- **Example**: Git (manual merge), custom applications

---

## Handling Concurrent User Data Updates in Eventual Consistency

### Scenario: Two Users Updating Same Record

```
User A (Node 1)          User B (Node 2)
    │                         │
    ├─ Read balance: $100     │
    │                         ├─ Read balance: $100
    │                         │
    ├─ Withdraw $30           │
    │  balance = $70          │
    │                         ├─ Withdraw $20
    │                         │  balance = $80
    │                         │
    ├─ Write $70 (T1)         │
    │                         ├─ Write $80 (T2)
    │                         │
    └─────────────────────────┘
         Conflict!
    
    Which value is correct?
    - LWW: $80 (T2 > T1)
    - Vector Clocks: Both are concurrent
    - CRDT: Depends on operation type
    - Application: Custom logic
```

### Best Practices for Concurrent Updates

<img src='../../Resources/17-system-architecture-basics/consistency/Screenshot 2026-02-12 at 1.15.03 PM.png' width='500' />

- **Use Idempotent Operations**: Ensure retries don't cause issues
- **Version Data**: Include version numbers or timestamps
- **Atomic Operations**: Use compare-and-swap or conditional writes
- **Immutable Events**: Store events instead of mutable state (Event Sourcing)
- **Conflict Monitoring**: Alert on conflicts, log for analysis
- **User Communication**: Inform users of potential inconsistencies

---

## Summary

| Aspect | Strong Consistency | Eventual Consistency |
|---|---|---|
| **Guarantee** | All nodes synchronized | Nodes converge over time |
| **Partition Behavior** | CP (blocks writes) | AP (continues operating) |
| **Latency** | Higher | Lower |
| **Availability** | Lower | Higher |
| **Conflict Resolution** | N/A (no conflicts) | LWW, Vector Clocks, CRDTs, Application-level |
| **Best For** | Finance, Healthcare | Social Media, Caching, Analytics |

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
