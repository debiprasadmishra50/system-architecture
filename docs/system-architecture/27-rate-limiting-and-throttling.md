# Rate Limiting and Throttling

## Table of Contents
1. [Rate Limiting](#rate-limiting)
2. [Throttling](#throttling)
3. [Rate Limiting vs Throttling](#rate-limiting-vs-throttling)
4. [When to Choose What](#when-to-choose-what)
5. [Implementation Examples](#implementation-examples)

---

## Rate Limiting

**Definition**: Controlling the number of requests a client can make to a server within a specified time window. Excess requests are rejected or queued.

### Key Characteristics
- **Hard Limit**: Strictly enforces maximum requests per time period
- **Rejection Strategy**: Denies requests exceeding the limit (returns 429 Too Many Requests)
- **Use Case**: Protecting APIs from abuse, preventing DDoS attacks, ensuring fair resource allocation
- **Behavior**: Client receives immediate feedback when limit is exceeded

### Common Algorithms

| Algorithm | Description | Best For |
|-----------|-------------|----------|
| **Token Bucket** | Tokens refill at fixed rate; each request consumes tokens | Bursty traffic, flexible rate limiting |
| **Leaky Bucket** | Requests leak out at constant rate from queue | Smooth traffic flow, preventing bursts |
| **Sliding Window** | Tracks requests in rolling time window | Precise rate limiting, simple implementation |
| **Fixed Window** | Resets counter at fixed intervals | Simple, low overhead |

### Token Bucket Diagram
```
┌─────────────────────────────────┐
│   Token Bucket (Capacity: 10)   │
├─────────────────────────────────┤
│ ● ● ● ● ● ● ● ● ● ●             │  Tokens available
├─────────────────────────────────┤
│ Refill Rate: 2 tokens/second    │
│ Request Cost: 1 token per req   │
└─────────────────────────────────┘
```

---

## Throttling

**Definition**: Slowing down or delaying requests to match system capacity. Requests are processed but at a controlled rate.

### Key Characteristics
- **Soft Limit**: Allows requests but processes them slower
- **Queuing Strategy**: Requests are queued and processed sequentially or in batches
- **Use Case**: Managing resource consumption, preventing system overload, backpressure handling
- **Behavior**: Client experiences delayed response but request eventually succeeds

### Common Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Queue-based** | Requests queued and processed in order | Fair processing, FIFO guarantee |
| **Adaptive** | Adjusts rate based on system load | Dynamic workloads, auto-scaling |
| **Priority-based** | High-priority requests processed first | Mixed workload types |
| **Backpressure** | Signals client to slow down | Preventing cascading failures |

### Leaky Bucket Diagram
```
┌──────────────────────────────┐
│   Request Queue              │
├──────────────────────────────┤
│ [Req1] [Req2] [Req3] [Req4]  │
├──────────────────────────────┤
│ Processing Rate: 1 req/sec   │
│ Output: ──→ ──→ ──→ ──→      │
└──────────────────────────────┘
```

---

## Rate Limiting vs Throttling

| Aspect | Rate Limiting | Throttling |
|--------|---------------|-----------|
| **Approach** | Reject excess requests | Delay/queue requests |
| **Response** | 429 Too Many Requests | 200 OK (delayed) |
| **Client Impact** | Immediate failure | Slower response time |
| **Resource Usage** | Prevents overload | Manages load gradually |
| **Data Loss** | Requests dropped | Requests preserved |
| **Use Case** | API protection, abuse prevention | Load management, backpressure |
| **Complexity** | Simpler to implement | More complex state management |

### Visual Comparison
```
Rate Limiting:
Request Stream: ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
Limit (10/sec): ▓▓▓▓▓▓▓▓▓▓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗
Result:         Rejected

Throttling:
Request Stream: ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
Rate (10/sec):  ▓▓▓▓▓▓▓▓▓▓ ▓▓▓▓▓▓▓▓▓▓
Result:         Queued & Processed
```

---

## When to Choose What

### Choose Rate Limiting When:
- **Protecting APIs** from malicious clients or DDoS attacks
- **Enforcing SLA** (Service Level Agreements) with strict quotas
- **Preventing abuse** of free tier services
- **Ensuring fair access** in multi-tenant systems
- **Cost control** is critical (e.g., third-party API calls)

### Choose Throttling When:
- **Managing internal load** within your system
- **Preventing cascading failures** in microservices
- **Handling traffic spikes** gracefully
- **Preserving data integrity** (no request loss acceptable)
- **Implementing backpressure** in event-driven systems
- **Gradual degradation** is preferred over rejection

### Hybrid Approach:
Combine both for optimal protection:
- **Rate Limiting** at API Gateway (external protection)
- **Throttling** at service level (internal load management)

---

## Implementation Examples

### Python: Token Bucket Rate Limiting

```python
import time
from threading import Lock

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        """
        capacity: max tokens in bucket
        refill_rate: tokens added per second
        """
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()
        self.lock = Lock()
    
    def allow_request(self, tokens_needed=1):
        with self.lock:
            now = time.time()
            elapsed = now - self.last_refill
            
            # Refill tokens
            self.tokens = min(
                self.capacity,
                self.tokens + elapsed * self.refill_rate
            )
            self.last_refill = now
            
            # Check if request allowed
            if self.tokens >= tokens_needed:
                self.tokens -= tokens_needed
                return True
            return False

# Usage
limiter = TokenBucket(capacity=10, refill_rate=2)

for i in range(15):
    if limiter.allow_request():
        print(f"Request {i+1}: Allowed")
    else:
        print(f"Request {i+1}: Rejected (rate limit exceeded)")
```

### Node.js: Leaky Bucket Throttling

```javascript
class LeakyBucket {
    constructor(capacity, leakRate) {
        /**
         * capacity: max queue size
         * leakRate: requests processed per second
         */
        this.capacity = capacity;
        this.queue = [];
        this.leakRate = leakRate;
        this.processing = false;
    }

    async addRequest(request) {
        if (this.queue.length >= this.capacity) {
            throw new Error('Queue full - throttling limit exceeded');
        }
        
        this.queue.push(request);
        
        if (!this.processing) {
            this.startProcessing();
        }
    }

    async startProcessing() {
        this.processing = true;
        const interval = 1000 / this.leakRate; // ms per request
        
        while (this.queue.length > 0) {
            const request = this.queue.shift();
            await this.processRequest(request);
            await new Promise(resolve => setTimeout(resolve, interval));
        }
        
        this.processing = false;
    }

    async processRequest(request) {
        console.log(`Processing: ${request}`);
        // Simulate processing
        return new Promise(resolve => setTimeout(resolve, 100));
    }
}

// Usage
const throttler = new LeakyBucket(capacity: 5, leakRate: 2);

for (let i = 1; i <= 10; i++) {
    throttler.addRequest(`Request ${i}`).catch(err => 
        console.error(`Request ${i}: ${err.message}`)
    );
}
```

### Python: Sliding Window Rate Limiting

```python
from collections import deque
import time

class SlidingWindowLimiter:
    def __init__(self, max_requests, window_seconds):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = deque()
    
    def is_allowed(self):
        now = time.time()
        
        # Remove old requests outside window
        while self.requests and self.requests[0] < now - self.window_seconds:
            self.requests.popleft()
        
        # Check limit
        if len(self.requests) < self.max_requests:
            self.requests.append(now)
            return True
        return False

# Usage: 5 requests per 10 seconds
limiter = SlidingWindowLimiter(max_requests=5, window_seconds=10)

for i in range(8):
    if limiter.is_allowed():
        print(f"Request {i+1}: Allowed")
    else:
        print(f"Request {i+1}: Rejected")
```

### Node.js: Adaptive Throttling

```javascript
class AdaptiveThrottler {
    constructor(initialRate = 10) {
        this.currentRate = initialRate; // requests/sec
        this.queue = [];
        this.systemLoad = 0;
        this.processing = false;
    }

    setSystemLoad(load) {
        // load: 0-1 (0=idle, 1=max capacity)
        this.systemLoad = load;
        
        // Adjust rate based on load
        if (load > 0.8) {
            this.currentRate = Math.max(1, this.currentRate * 0.8);
        } else if (load < 0.5) {
            this.currentRate = this.currentRate * 1.2;
        }
    }

    async addRequest(request) {
        this.queue.push(request);
        
        if (!this.processing) {
            this.startProcessing();
        }
    }

    async startProcessing() {
        this.processing = true;
        
        while (this.queue.length > 0) {
            const request = this.queue.shift();
            await this.processRequest(request);
            
            const interval = 1000 / this.currentRate;
            await new Promise(resolve => setTimeout(resolve, interval));
        }
        
        this.processing = false;
    }

    async processRequest(request) {
        console.log(`Processing at rate: ${this.currentRate.toFixed(2)} req/s`);
        return new Promise(resolve => setTimeout(resolve, 50));
    }
}

// Usage
const throttler = new AdaptiveThrottler(initialRate: 10);

// Simulate system load changes
throttler.setSystemLoad(0.3); // Light load - increase rate
throttler.setSystemLoad(0.9); // Heavy load - decrease rate
```

---

## Best Practices

- **Combine Strategies**: Use rate limiting at API gateway + throttling at service level
- **Monitor Metrics**: Track rejection/delay rates to tune limits
- **Graceful Degradation**: Provide clear error messages and retry guidance
- **Distributed Systems**: Use Redis/Memcached for shared state across instances
- **Client Awareness**: Expose rate limit headers (X-RateLimit-Remaining, X-RateLimit-Reset)
- **Testing**: Load test to determine optimal limits for your system

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
