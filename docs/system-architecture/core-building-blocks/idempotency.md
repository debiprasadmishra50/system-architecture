# Idempotency

## Table of Contents
1. [What is Idempotency](#what-is-idempotency)
2. [Problems It Solves](#problems-it-solves)
3. [Why It's Required](#why-its-required)
4. [How It Ensures Consistency](#how-it-ensures-consistency)
5. [Real-World Examples](#real-world-examples)
   - [PayPal Payment Processing](#paypal-payment-processing)
   - [Stripe Charge Creation](#stripe-charge-creation)
6. [Implementation Strategies](#implementation-strategies)
7. [Diagram: Idempotency Flow](#diagram-idempotency-flow)

---

## What is Idempotency
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.25.15 PM.png' width=600 />

- **Definition**: A property where performing an operation multiple times produces the same result as performing it once
- **Core Principle**: Repeated identical requests yield identical responses without side effects
- **Scope**: Applies to API calls, database operations, and distributed system transactions
- **Key Characteristic**: Safe to retry without causing unintended duplications or state changes

---

## Problems It Solves

- **Duplicate Requests**: Network failures cause clients to retry; idempotency prevents duplicate processing
- **Inconsistent State**: Multiple executions of the same operation could corrupt data; idempotency maintains consistency
- **Lost Updates**: In distributed systems, retries might apply changes multiple times; idempotency prevents this
- **Race Conditions**: Concurrent identical requests could cause conflicts; idempotency ensures single execution effect
- **User Experience**: Clients can safely retry without worrying about unintended side effects

---

## Why It's Required

- **Network Unreliability**: Networks fail; clients don't know if requests succeeded, so they retry
- **Distributed Systems**: Multiple services communicating asynchronously need guaranteed single execution semantics
- **Financial Transactions**: Payment systems cannot afford duplicate charges or transfers
- **Data Integrity**: Systems must maintain consistency even with retries and failures
- **User Trust**: Critical operations (payments, transfers) require certainty of single execution
- **Resilience**: Enables safe retry mechanisms without manual intervention

---

## How It Ensures Consistency
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.27.30 PM.png' width=600 />

### 1. **Idempotency Keys**
- Client generates unique identifier (UUID) for each request
- Server stores key with operation result
- Duplicate requests with same key return cached result instead of re-executing
- Prevents duplicate state changes

### 2. **Deduplication**
- Server maintains idempotency key → result mapping
- Checks if key exists before processing
- Returns previous result if key found
- Eliminates duplicate side effects

### 3. **Atomic Operations**
- Combine key check and operation in single atomic transaction
- Ensures either both succeed or both fail
- Prevents race conditions between check and execution

### 4. **State Verification**
- Verify operation result matches expected state
- Detect if operation was partially completed
- Rollback or complete as needed

### 5. **Timeout Handling**
- Set expiration on idempotency keys (e.g., 24 hours)
- Prevent indefinite storage of old keys
- Balance between consistency and resource usage

---

## Real-World Examples

### PayPal Payment Processing
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.30.43 PM.png' width=700 />

**Scenario**: User initiates $100 payment transfer

```
Request 1: POST /api/transfer
Headers: Idempotency-Key: "txn-12345-abc"
Body: { amount: 100, recipient: "john@example.com" }

Response: { status: "success", transaction_id: "txn-001", amount: 100 }
```

**Network Timeout** → Client retries with same Idempotency-Key

```
Request 2: POST /api/transfer
Headers: Idempotency-Key: "txn-12345-abc"  (same key)
Body: { amount: 100, recipient: "john@example.com" }

Response: { status: "success", transaction_id: "txn-001", amount: 100 }
(Same result returned from cache - no duplicate charge)
```

**Without Idempotency**: User charged $200 (duplicate debit)
**With Idempotency**: User charged $100 (single execution)

---

### Stripe Charge Creation

**Scenario**: Create $50 charge for subscription

```
Request 1: POST /v1/charges
Headers: Idempotency-Key: "charge-stripe-789"
Body: { amount: 5000, currency: "usd", customer: "cus_123" }

Response: { id: "ch_1234567", amount: 5000, status: "succeeded" }
```

**Client Timeout** → Retry with same Idempotency-Key

```
Request 2: POST /v1/charges
Headers: Idempotency-Key: "charge-stripe-789"  (same key)
Body: { amount: 5000, currency: "usd", customer: "cus_123" }

Response: { id: "ch_1234567", amount: 5000, status: "succeeded" }
(Cached response - no new charge created)
```

**Without Idempotency**: Two charges of $50 each
**With Idempotency**: Single charge of $50

---

## Implementation Strategies

### Strategy 1: Database-Backed Deduplication
```
1. Client sends request with Idempotency-Key header
2. Server checks idempotency_keys table for key
3. If found: return cached result
4. If not found: execute operation, store result with key
5. Return result
```

### Strategy 2: Cache-Based Deduplication
```
1. Check Redis/Memcached for Idempotency-Key
2. If hit: return cached result
3. If miss: execute operation, cache result with TTL
4. Return result
```

### Strategy 3: Event Sourcing
```
1. Store all operations as immutable events
2. Assign unique event ID to each request
3. Check if event ID already processed
4. If yes: replay result from event log
5. If no: create new event and process
```

### Key Considerations
- **Storage**: Idempotency keys require persistent storage (database or cache)
- **TTL**: Set expiration to balance consistency and storage costs
- **Scope**: Define which operations are idempotent (reads always are; writes need explicit handling)
- **Monitoring**: Track idempotency key hits to identify retry patterns

---

## Diagram: Idempotency Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT REQUEST                              │
│  POST /api/payment                                              │
│  Idempotency-Key: "key-12345"                                   │
│  Body: { amount: 100 }                                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │  Server Receives Request       │
        └────────────┬───────────────────┘
                     │
                     ▼
        ┌────────────────────────────────┐
        │  Check Idempotency Key Store   │
        │  (Database/Cache)              │
        └────────┬───────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
    ┌────────┐        ┌──────────┐
    │ KEY    │        │ KEY NOT  │
    │ FOUND  │        │ FOUND    │
    └───┬────┘        └────┬─────┘
        │                  │
        ▼                  ▼
    ┌────────────┐    ┌──────────────────┐
    │ Return     │    │ Execute          │
    │ Cached     │    │ Operation        │
    │ Result     │    │ (Process Payment)│
    └────────────┘    └────┬─────────────┘
        │                  │
        │                  ▼
        │             ┌──────────────────┐
        │             │ Store Result     │
        │             │ with Key in      │
        │             │ Idempotency Store│
        │             └────┬─────────────┘
        │                  │
        └──────────┬───────┘
                   │
                   ▼
        ┌────────────────────────────────┐
        │  Return Response to Client     │
        │  { status: "success", ... }    │
        └────────────────────────────────┘
```

---

## Summary

| Aspect | Details |
|--------|---------|
| **Purpose** | Ensure single execution semantics for repeated requests |
| **Key Mechanism** | Idempotency keys + deduplication |
| **Storage** | Database or cache with TTL |
| **Scope** | Critical for payments, transfers, and state-changing operations |
| **Benefit** | Safe retries without duplicate side effects |
| **Cost** | Additional storage and lookup overhead |
