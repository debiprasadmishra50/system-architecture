# Caching: LRU and LFU

## Table of Contents
1. [What is Caching?](#what-is-caching)
2. [LRU Cache (Least Recently Used)](#lru-cache-least-recently-used)
3. [LFU Cache (Least Frequently Used)](#lfu-cache-least-frequently-used)
4. [Comparison: LRU vs LFU](#comparison-lru-vs-lfu)
5. [Cache Invalidation Strategies](#cache-invalidation-strategies)
6. [Architect's Perspective](#architects-perspective)

### What is Caching?

- **Definition**: Storing frequently accessed data in fast-access memory to reduce latency and improve performance
- **Purpose**: Reduce database/disk I/O operations, decrease response times, improve system throughput
- **Trade-off**: Uses additional memory to gain speed
- **Key Principle**: Temporal and spatial locality - recently used data is likely to be used again soon
- **Levels**: CPU cache, memory cache (Redis, Memcached), disk cache, CDN cache

### Cache Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT REQUEST                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │   L1: Browser Cache    │ (Fastest, Smallest)
        │   (HTML, CSS, JS)      │
        └────────────┬───────────┘
                     │ Cache Miss
                     ▼
        ┌────────────────────────┐
        │   L2: CDN Cache        │
        │   (Static Content)     │
        └────────────┬───────────┘
                     │ Cache Miss
                     ▼
        ┌────────────────────────┐
        │   L3: Application      │
        │   Cache (Redis)        │
        │   (Session, Data)      │
        └────────────┬───────────┘
                     │ Cache Miss
                     ▼
        ┌────────────────────────┐
        │   L4: Database         │ (Slowest, Largest)
        │   (Persistent Data)    │
        └────────────────────────┘
```

### Cache Operations

```
┌──────────────────────────────────────────────────────┐
│              CACHE OPERATIONS FLOW                   │
└──────────────────────────────────────────────────────┘

1. READ REQUEST
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │ Request Data (Key)
          ▼
   ┌─────────────────────┐
   │  Cache Lookup       │
   │  (Check if exists)  │
   └──┬─────────────┬────┘
      │             │
   HIT│             │MISS
      ▼             ▼
   ┌──────┐    ┌──────────────┐
   │Return│    │Fetch from DB │
   │Data  │    │Store in Cache│
   └──────┘    └──────┬───────┘
                      │
                      ▼
                  ┌──────┐
                  │Return│
                  │Data  │
                  └──────┘

2. WRITE REQUEST
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │ Write Data
          ▼
   ┌──────────────────────┐
   │ Update Database      │
   │ Invalidate Cache     │
   │ (Remove old entry)   │
   └──────────────────────┘
```

---

## LRU Cache (Least Recently Used)

### Definition and Concept

- **Definition**: Cache eviction policy that removes the least recently used item when cache is full
- **Principle**: If data hasn't been accessed recently, it's less likely to be needed soon
- **Tracking**: Maintains access order of items
- **Eviction**: When capacity is reached, removes the item that was accessed longest time ago

### How LRU Works

```
┌─────────────────────────────────────────────────────┐
│           LRU CACHE EXAMPLE (Capacity: 3)           │
└─────────────────────────────────────────────────────┘

Initial State: Empty
Cache: [ ]

Step 1: Access A
Cache: [A]  (A is most recent)

Step 2: Access B
Cache: [B, A]  (B is most recent, A is older)

Step 3: Access C
Cache: [C, B, A]  (C is most recent)

Step 4: Access D (Cache Full!)
Cache: [D, C, B]  (A is evicted - least recently used)

Step 5: Access B (B already in cache)
Cache: [B, D, C]  (B moves to front - most recent)

Step 6: Access E (Cache Full!)
Cache: [E, B, D]  (C is evicted - least recently used)
```

### Implementation Details

- **Data Structure**: Doubly-linked list + HashMap
  - HashMap: O(1) lookup
  - Doubly-linked list: O(1) insertion/deletion
- **Operations**:
  - Get: O(1) - lookup in map, move to front
  - Put: O(1) - add to map, add to front of list
  - Evict: O(1) - remove from back of list

### Advantages of LRU

- **Simple Logic**: Easy to understand and implement
- **Good Performance**: Works well for temporal locality patterns
- **Predictable**: Evicts based on clear, deterministic rule
- **Low Overhead**: Minimal computational cost
- **Effective for**: Web caching, page replacement, session storage

### Disadvantages of LRU

- **Ignores Frequency**: Doesn't consider how often items are accessed
- **Scan Attacks**: Sequential access pattern can evict frequently used items
  - Example: Accessing A, B, C, D, E in sequence evicts A even if A is accessed 1000 times
- **Recency Bias**: Recent one-time accesses can evict frequently used old items
- **Memory Overhead**: Requires maintaining linked list structure
- **Not Optimal**: May not minimize cache misses in all scenarios

### Use Cases for LRU

- **Web Page Caching**: Recently visited pages likely to be revisited
- **CPU Cache**: Temporal locality in program execution
- **Database Query Cache**: Recent queries likely to be repeated
- **Browser Cache**: Recent website visits likely to be revisited
- **CDN Cache**: Recent content requests likely to be repeated

---

## LFU Cache (Least Frequently Used)

### Definition and Concept
- **Definition**: Cache eviction policy that removes the least frequently used item when cache is full
- **Principle**: Items accessed more often are more valuable and should be kept
- **Tracking**: Maintains frequency count for each item
- **Eviction**: When capacity is reached, removes the item with lowest access frequency

### How LFU Works

```
┌─────────────────────────────────────────────────────┐
│           LFU CACHE EXAMPLE (Capacity: 3)           │
└─────────────────────────────────────────────────────┘

Initial State: Empty
Cache: [ ]

Step 1: Access A
Cache: [A(freq:1)]

Step 2: Access B
Cache: [A(freq:1), B(freq:1)]

Step 3: Access C
Cache: [A(freq:1), B(freq:1), C(freq:1)]

Step 4: Access A (A accessed again)
Cache: [A(freq:2), B(freq:1), C(freq:1)]

Step 5: Access A (A accessed again)
Cache: [A(freq:3), B(freq:1), C(freq:1)]

Step 6: Access D (Cache Full!)
Cache: [A(freq:3), B(freq:1), D(freq:1)]  (C evicted - freq:1, least frequent)

Step 7: Access B (B accessed again)
Cache: [A(freq:3), B(freq:2), D(freq:1)]

Step 8: Access E (Cache Full!)
Cache: [A(freq:3), B(freq:2), E(freq:1)]  (D evicted - freq:1, least frequent)
```

### Implementation Details

- **Data Structure**: HashMap + Min-Heap + Frequency Map
  - HashMap: O(1) lookup
  - Frequency Map: Track access count for each item
  - Min-Heap: O(log n) to find minimum frequency
- **Operations**:
  - Get: O(1) - lookup, increment frequency
  - Put: O(log n) - add to heap, update frequency
  - Evict: O(log n) - remove from heap

### Advantages of LFU

- **Frequency-Aware**: Considers how often items are accessed
- **Optimal for Patterns**: Better for workloads with clear access patterns
- **Reduces Misses**: Keeps frequently accessed items longer
- **Resistant to Scans**: Sequential access doesn't evict frequently used items
- **Better Hit Rate**: Generally achieves higher cache hit rates than LRU

### Disadvantages of LFU

- **Complexity**: More complex to implement than LRU
- **Higher Overhead**: Requires maintaining frequency counters and heap
- **Slower Operations**: O(log n) operations vs O(1) for LRU
- **Cold Start Problem**: New items have low frequency, may be evicted quickly
- **Frequency Decay**: Doesn't account for time - old frequent items stay forever
- **Memory Overhead**: Requires additional data structures

### Use Cases for LFU

- **Database Query Cache**: Frequently executed queries should stay cached
- **API Response Cache**: Popular endpoints should be cached longer
- **Image/Video Cache**: Popular content accessed frequently
- **Machine Learning Models**: Frequently used models should stay in memory
- **Recommendation Systems**: Popular items should be cached

---

## Comparison: LRU vs LFU

| Aspect | LRU | LFU |
|--------|-----|-----|
| **Eviction Rule** | Least recently used | Least frequently used |
| **Time Complexity (Get)** | O(1) | O(1) |
| **Time Complexity (Put)** | O(1) | O(log n) |
| **Space Complexity** | O(n) | O(n) |
| **Implementation** | Linked List + HashMap | HashMap + Heap + Frequency Map |
| **Temporal Locality** | Excellent | Good |
| **Frequency Awareness** | No | Yes |
| **Scan Attack Resistant** | No | Yes |
| **Cold Start Problem** | No | Yes |
| **Overhead** | Low | High |
| **Hit Rate** | Good | Better |
| **Best For** | Recent access patterns | Frequency-based patterns |

---

## Cache Invalidation Strategies

### Time-Based (TTL - Time To Live)
- **How it works**: Cache expires after fixed time period
- **Example**: Cache data for 5 minutes, then refresh
- **Advantages**: Simple, automatic cleanup
- **Disadvantages**: May serve stale data, may refresh unnecessarily

### Event-Based Invalidation
- **How it works**: Cache invalidated when data changes
- **Example**: When user updates profile, invalidate user cache
- **Advantages**: Always fresh data
- **Disadvantages**: Complex to implement, requires event tracking

### LRU/LFU Eviction
- **How it works**: Remove items based on usage patterns
- **Advantages**: Automatic, memory-efficient
- **Disadvantages**: May lose useful data

### Write-Through
- **How it works**: Write to cache and database simultaneously
- **Advantages**: Cache always consistent with database
- **Disadvantages**: Slower writes, double write operations

### Write-Behind (Write-Back)
- **How it works**: Write to cache first, database later
- **Advantages**: Fast writes, improved performance
- **Disadvantages**: Risk of data loss if cache fails

---

## Architect's Perspective

- **Choose LRU When**:
  - Access patterns are temporal (recent = likely to be used again)
  - Simple implementation is preferred
  - Performance overhead must be minimal
  - Examples: Web caching, browser cache, page replacement

- **Choose LFU When**:
  - Access patterns are frequency-based (popular = likely to be used again)
  - Hit rate optimization is critical
  - Computational overhead is acceptable
  - Examples: Database query cache, API response cache, recommendation systems

- **Hybrid Approaches**:
  - **LRU with TTL**: Combine recency with time-based expiration
  - **LFU with Decay**: Reduce frequency over time to handle changing patterns
  - **Adaptive Caching**: Switch between LRU and LFU based on workload

- **Cache Levels**:
  - Use multiple cache levels for optimal performance
  - Browser cache → CDN → Application cache → Database
  - Each level serves different purposes

- **Monitoring**:
  - Track cache hit rate (hits / total requests)
  - Monitor eviction rate
  - Measure cache memory usage
  - Analyze access patterns to optimize strategy

- **Common Pitfalls**:
  - Cache stampede: Multiple requests for expired item
  - Cache invalidation: "There are only two hard things in Computer Science: cache invalidation and naming things"
  - Memory leaks: Unbounded cache growth
  - Stale data: Serving outdated information

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
