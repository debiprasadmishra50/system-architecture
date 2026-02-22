# Rate Limiting

## Table of Contents

1. [What is Rate Limiting](#what-is-rate-limiting)
2. [Problems It Solves](#problems-it-solves)
   - [Denial of Service (DoS)](#denial-of-service-dos)
   - [Distributed Denial of Service (DDoS)](#distributed-denial-of-service-ddos)
3. [Types of Rate Limiting](#types-of-rate-limiting)
   - [User-Based Rate Limiting](#user-based-rate-limiting)
   - [Server-Based Rate Limiting](#server-based-rate-limiting)
4. [Rate Limiting Algorithms](#rate-limiting-algorithms)
   - [Common Algorithms](#common-algorithms)
   - [1. Fixed Window (Counter-Based)](#1-fixed-window-counter-based)
   - [2. Sliding Window](#2-sliding-window)
   - [3. Token Bucket](#3-token-bucket)
   - [4. Leaky Bucket](#4-leaky-bucket)
5. [Comparison Table](#comparison-table)
6. [Implementation Considerations](#implementation-considerations)

---

## What is Rate Limiting
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.33.22 AM.png' width=500 />

Rate limiting is a technique that controls the number of requests a client can make to a server within a specified time window. It enforces a maximum threshold on request frequency, ensuring fair resource distribution and system stability.

**Key Characteristics:**
- Defines maximum requests per time unit (e.g., 100 requests/minute)
- Rejects or delays excess requests
- Applies at various layers (API gateway, service, database)
- Transparent to legitimate users within limits

---

## Problems It Solves

### Denial of Service (DoS)
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.35.09 AM.png' width=500 />

A single attacker floods the server with requests, exhausting resources and making the service unavailable to legitimate users.

**Impact:**
- Server CPU/memory exhaustion
- Network bandwidth saturation
- Service downtime

**Rate Limiting Mitigation:**
- Caps requests per IP address
- Detects abnormal traffic patterns
- Blocks or throttles suspicious sources

### Distributed Denial of Service (DDoS)
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.35.35 AM.png' width=500 />

Multiple compromised machines (botnet) simultaneously attack the server, making source-based blocking ineffective.

**Impact:**
- Coordinated attack from many IPs
- Harder to distinguish from legitimate traffic
- Requires sophisticated detection

**Rate Limiting Mitigation:**
- Implements per-endpoint limits
- Uses behavioral analysis
- Combines with WAF (Web Application Firewall)
- Leverages CDN/DDoS protection services

---

## Types of Rate Limiting

### User-Based Rate Limiting
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.36.28 AM.png' width=350 />

Limits requests per authenticated user or API key.

**Use Cases:**
- SaaS platforms with tiered plans
- API services with user quotas
- Preventing individual user abuse

**Example:**
```
User A: 1000 requests/day
User B: 5000 requests/day (premium tier)
```

**Advantages:**
- Fair resource allocation
- Supports different user tiers
- Tracks usage per user

**Disadvantages:**
- Requires authentication
- Doesn't protect against unauthenticated attacks
- More storage overhead

### Server-Based Rate Limiting
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.37.57 AM.png' width=350 />

Limits requests per IP address, endpoint, or globally.

**Use Cases:**
- Protecting against anonymous attacks
- Preventing resource exhaustion
- Enforcing global API limits

**Example:**
```
Per IP: 100 requests/minute
Per Endpoint: 1000 requests/minute
Global: 10,000 requests/minute
```

**Advantages:**
- Works without authentication
- Simple to implement
- Protects against DDoS

**Disadvantages:**
- Affects all users behind same IP (proxy/NAT)
- Less granular control
- May block legitimate traffic

---

## Rate Limiting Algorithms

### Common Algorithms

| Algorithm | Description | Best For |
|-----------|-------------|----------|
| **Token Bucket** | Tokens refill at fixed rate; each request consumes tokens | Bursty traffic, flexible rate limiting |
| **Leaky Bucket** | Requests leak out at constant rate from queue | Smooth traffic flow, preventing bursts |
| **Sliding Window** | Tracks requests in rolling time window | Precise rate limiting, simple implementation |
| **Fixed Window** | Resets counter at fixed intervals | Simple, low overhead |

---

### 1. Fixed Window (Counter-Based)

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.39.42 AM.png' width=600 />

Divides time into fixed intervals and counts requests in each window.

**How It Works:**
```
Window 1: [0s - 60s]   → 100 requests allowed
Window 2: [60s - 120s] → 100 requests allowed
```

**Implementation:**
```
Timestamp: 0s
Request 1: counter = 1
Request 2: counter = 2
...
Request 100: counter = 100
Request 101: REJECTED (limit exceeded)

At 60s: counter resets to 0
```

**Diagram:**
```
Time →
|----60s----|----60s----|----60s----|
| ✓✓✓✓✓✓✓✓✓ | ✓✓✓✓✓✓✓✓✓ | ✓✓✓✓✓✓✓✓✓ |
| 100 reqs  | 100 reqs  | 100 reqs  |
```

**Pros:**
- Simple to implement
- Low memory overhead
- Fast computation

**Cons:**
- Burst at window boundaries (allows 200 requests in 2 seconds if timed right)
- Uneven distribution
- Not smooth rate limiting

---

### 2. Sliding Window
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.41.09 AM.png' width=600 />

Maintains a rolling time window, smoothing out boundary issues.

**How It Works:**
```
Current time: 45s
Window: [45s - 105s] (last 60 seconds)
Count requests in this window
```

**Implementation:**
```
Requests: [10s, 15s, 20s, 45s, 50s, 55s, 60s]
Current time: 65s
Window: [5s - 65s]
Valid requests: [10s, 15s, 20s, 45s, 50s, 55s, 60s] = 7 requests
```

**Diagram:**
```
Time →
Requests: ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓
          |←---- 60s window ----|
                              ↑ current time
```

**Pros:**
- Eliminates boundary burst problem
- Smooth rate limiting
- More accurate

**Cons:**
- Higher memory usage (stores timestamps)
- More complex implementation
- Slower lookups

---

### 3. Token Bucket
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.41.27 AM.png' width=350 />

Tokens accumulate at a fixed rate; each request consumes one token.

**How It Works:**
```
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second

Time 0s: 100 tokens
Request: -1 token → 99 tokens
Request: -1 token → 98 tokens
...
Time 1s: +10 tokens (refill) → 108 tokens (capped at 100)
```

**Implementation:**
```
tokens = min(capacity, tokens + (now - last_refill) * rate)
if tokens >= 1:
    tokens -= 1
    allow request
else:
    reject request
last_refill = now
```

**Diagram:**
```
Tokens
  100 |●●●●●●●●●●
      |●●●●●●●●●●
   50 |●●●●●●●●●●
      |●●●●●●●●●●
    0 |___________|___________|
      0s         10s         20s
      ↑ Requests consume tokens
      ↑ Tokens refill at constant rate
```

**Pros:**
- Allows controlled bursts
- Smooth rate limiting
- Flexible (can adjust rate/capacity)
- Handles variable traffic well

**Cons:**
- Requires timer/clock
- More complex than fixed window
- Moderate memory usage

---

### 4. Leaky Bucket
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.42.25 AM.png' width=500 />

Requests enter a queue; they leak out at a fixed rate.

**How It Works:**
```
Bucket capacity: 100 requests
Leak rate: 10 requests/second

Request arrives: added to queue (if space available)
Leak: 10 requests/second exit the bucket
```

**Implementation:**
```
queue = []
leak_rate = 10 requests/second

on_request():
    if len(queue) < capacity:
        queue.append(request)
    else:
        reject request

on_leak_timer():
    for i in range(leak_rate):
        if queue not empty:
            process(queue.pop())
```

**Diagram:**
```
Incoming Requests
      ↓ ↓ ↓ ↓ ↓ ↓ ↓
    [●●●●●●●●●●]  ← Queue (capacity: 100)
      ↓ ↓ ↓ ↓ ↓ ↓ ↓
    Leak (10/sec)
      ↓
   Processed Requests
```

**Pros:**
- Smooth, predictable output rate
- Prevents burst traffic
- Fair queuing
- Good for traffic shaping

**Cons:**
- Requests may wait in queue
- Higher latency
- Requires queue management
- More memory overhead

---

## Comparison Table

| Algorithm | Burst Allowed | Memory | Complexity | Boundary Issue | Use Case |
|-----------|---------------|--------|-----------|-----------------|----------|
| Fixed Window | High | Low | Low | Yes | Simple APIs |
| Sliding Window | No | High | Medium | No | Accurate limiting |
| Token Bucket | Controlled | Medium | Medium | No | APIs with bursts |
| Leaky Bucket | No | High | High | No | Traffic shaping |

---

## Implementation Considerations

**Storage Backend:**
- In-memory: Fast, single-server only
- Redis: Distributed, persistent
- Database: Persistent, slower

**Key Metrics:**
- Requests per second (RPS)
- Requests per minute/hour/day
- Concurrent connections
- Burst capacity

**Response Strategies:**
- Reject with 429 (Too Many Requests)
- Queue and delay
- Degrade service quality
- Redirect to backup

**Monitoring:**
- Track rejection rate
- Monitor latency impact
- Alert on anomalies
- Log rate limit violations
