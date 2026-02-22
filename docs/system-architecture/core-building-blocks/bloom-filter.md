# Bloom Filter

## Table of Contents

1. [What is a Bloom Filter](#what-is-a-bloom-filter)
2. [Problem It Solves](#problem-it-solves)
3. [Maybe YES, Definite NO](#maybe-yes-definite-no)
4. [Extension of Hashtable](#extension-of-hashtable)
5. [Why It's Required](#why-its-required)
6. [Logic & Diagrams](#logic--diagrams)
7. [Key Operations](#key-operations)
8. [False Positives](#false-positives)
9. [Optimization Strategies](#optimization-strategies)
10. [Real-World Use Cases](#real-world-use-cases)
11. [Applications](#applications)

---

## What is a Bloom Filter

A **Bloom Filter** is a space-efficient, probabilistic data structure that tests whether an element is a member of a set.

- **Probabilistic**: Returns definite "NO" or "probably YES"
- **Space-efficient**: Uses a bit array instead of storing actual elements
- **Fast**: O(k) lookup time where k is the number of hash functions
- **Irreversible**: Cannot retrieve original elements from the filter

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.37.37 PM.png' width=700 />

---

## Problem It Solves

### Traditional Hashtable Problems

| Problem | Hashtable | Bloom Filter |
|---------|-----------|--------------|
| Memory Usage | High (stores full objects) | Very Low (bit array) |
| Lookup Speed | O(1) average | O(k) constant |
| False Negatives | No | No |
| False Positives | No | Yes |
| Scalability | Limited by memory | Highly scalable |

**Key Problem**: When you need to check membership in massive datasets without storing the entire dataset in memory.

---

## Maybe YES, Definite NO

Bloom Filters provide a unique guarantee:

```
Query Result:
├─ "Definitely NOT in set" → 100% accurate
└─ "Probably in set" → May have false positives
```

**Example**:
- Query: "Is user 'john@example.com' in our database?"
  - If Bloom Filter says **NO** → User definitely not in database
  - If Bloom Filter says **YES** → User might be in database (need to verify)

---

## Extension of Hashtable

### How Bloom Filter Extends Hashtable Concept

**Hashtable Approach**:
```
Input: "user123"
  ↓
Hash Function: hash("user123") = 5
  ↓
Lookup: hashtable[5] = "user123"
  ↓
Result: Found or Not Found
```

**Bloom Filter Approach**:
```
Input: "user123"
  ↓
Multiple Hash Functions:
  - hash1("user123") = 2
  - hash2("user123") = 7
  - hash3("user123") = 15
  ↓
Set bits at positions: [2, 7, 15]
  ↓
Lookup: Check if ALL positions are set
  ↓
Result: Definitely NO or Probably YES
```

### Library Problem Example

**Scenario**: Library with 1 million books. Check if a book exists before searching shelves.

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.37.59 PM.png' width=700 />

**Hashtable Approach**:
- Store all 1M book ISBNs in memory
- Memory: ~40MB (assuming 40 bytes per ISBN)
- Lookup: O(1) average, guaranteed accuracy

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.38.13 PM.png' width=600 />

**Bloom Filter Approach**:
- Use bit array of 10MB (80 million bits)
- Memory: ~10MB (4x smaller!)
- Lookup: O(k) with 3 hash functions
- Trade-off: 1-2% false positive rate

**Workflow**:
```
User asks: "Do we have ISBN 978-0-13-110362-7?"
  ↓
Bloom Filter check:
  - hash1(ISBN) = 1,234,567 → bit[1,234,567] = 1 ✓
  - hash2(ISBN) = 5,678,901 → bit[5,678,901] = 1 ✓
  - hash3(ISBN) = 8,901,234 → bit[8,901,234] = 1 ✓
  ↓
Result: "Probably YES, check shelf"
  ↓
Librarian searches shelf → Found or Not Found
```

---

## Why It's Required

### Use Cases Demanding Bloom Filters

1. **Massive Scale**: Billions of elements, limited memory
2. **Fast Rejection**: Quickly eliminate non-members before expensive operations
3. **Distributed Systems**: Reduce network calls for membership checks
4. **Real-time Processing**: Sub-millisecond lookup requirements
5. **Cost Optimization**: Minimize storage and bandwidth costs

### When NOT to Use

- Need 100% accuracy on positive results
- Need to retrieve original elements
- Dataset is small enough to fit in memory
- False positives are unacceptable

---

## Logic & Diagrams

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.44.29 PM.png' width=700 />

### Bloom Filter Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Bloom Filter                         │
├─────────────────────────────────────────────────────────┤
│  Bit Array (m bits):                                    │
│  [0][1][0][1][0][0][1][0][1][0][1][0][0][1][0][1]       │
│   0   1   2   3   4   5   6   7   8   9  10  11 12 13 14 15
│                                                         │
│  Hash Functions: k = 3                                  │
│  - hash1(x)                                             │
│  - hash2(x)                                             │
│  - hash3(x)                                             │
└─────────────────────────────────────────────────────────┘
```

### Insertion Process

```
Insert "apple":
  ↓
hash1("apple") = 3  → Set bit[3] = 1
hash2("apple") = 7  → Set bit[7] = 1
hash3("apple") = 13 → Set bit[13] = 1
  ↓
Bit Array: [0][1][0][1][0][0][1][0][1][0][1][0][0][1][0][1]
                    ↑           ↑               ↑
```

### Lookup Process

```
Lookup "apple":
  ↓
hash1("apple") = 3  → Check bit[3] = 1 ✓
hash2("apple") = 7  → Check bit[7] = 1 ✓
hash3("apple") = 13 → Check bit[13] = 1 ✓
  ↓
All bits set? YES → "Probably in set"
```

### False Positive Example

```
Lookup "banana" (not inserted):
  ↓
hash1("banana") = 3  → Check bit[3] = 1 ✓
hash2("banana") = 7  → Check bit[7] = 1 ✓
hash3("banana") = 15 → Check bit[15] = 0 ✗
  ↓
All bits set? NO → "Definitely NOT in set"

---

Lookup "orange" (not inserted):
  ↓
hash1("orange") = 3  → Check bit[3] = 1 ✓
hash2("orange") = 7  → Check bit[7] = 1 ✓
hash3("orange") = 13 → Check bit[13] = 1 ✓
  ↓
All bits set? YES → "Probably in set" (FALSE POSITIVE!)
```

---

## Key Operations

### 1. Insertion

**Algorithm**:
```
insert(element):
  for i = 1 to k:
    index = hash_i(element) % m
    bit_array[index] = 1
```

**Time Complexity**: O(k) where k = number of hash functions
**Space Complexity**: O(m) where m = bit array size

**Example**:
```
Insert "user123" with k=3, m=100:
  hash1("user123") % 100 = 23 → bit[23] = 1
  hash2("user123") % 100 = 67 → bit[67] = 1
  hash3("user123") % 100 = 89 → bit[89] = 1
```

### 2. Lookup

**Algorithm**:
```
lookup(element):
  for i = 1 to k:
    index = hash_i(element) % m
    if bit_array[index] == 0:
      return "DEFINITELY NOT"
  return "PROBABLY YES"
```

**Time Complexity**: O(k)
**Space Complexity**: O(1)

**Example**:
```
Lookup "user456" with k=3, m=100:
  hash1("user456") % 100 = 45 → bit[45] = 0
  ↓
  Return "DEFINITELY NOT" (no need to check further)
```

---

## False Positives

### What Are False Positives?

A **false positive** occurs when the filter says "probably YES" but the element was never inserted.

### False Positive Rate Formula

```
FP Rate ≈ (1 - e^(-kn/m))^k

Where:
  k = number of hash functions
  n = number of inserted elements
  m = size of bit array
```

### Example Calculation

```
Scenario: 1 million users, 10MB bit array (80M bits)
  n = 1,000,000
  m = 80,000,000
  k = 3 (optimal)

FP Rate ≈ (1 - e^(-3×1,000,000/80,000,000))^3
        ≈ (1 - e^(-0.0375))^3
        ≈ (0.0368)^3
        ≈ 0.0005 (0.05%)
```

### Reducing False Positives

| Strategy | Impact | Trade-off |
|----------|--------|-----------|
| Increase m (bit array) | Decreases FP rate | More memory |
| Increase k (hash functions) | Optimal at k=ln(2)×m/n | More CPU |
| Decrease n (elements) | Decreases FP rate | Limited capacity |

---

## Optimization Strategies

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.53.34 PM.png' width=700 />

### 1. Right Number of Hash Functions

**Optimal k Formula**:
```
k_optimal = (m / n) × ln(2)

Where:
  m = bit array size
  n = number of elements
```
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 12.54.11 PM.png' width=500 />

**Example**:
```
m = 80,000,000 bits
n = 1,000,000 elements

k_optimal = (80,000,000 / 1,000,000) × ln(2)
          = 80 × 0.693
          ≈ 5.5 → Use k = 5 or 6
```

**Impact**:
- Too few hash functions → High false positive rate
- Too many hash functions → Slower lookups, diminishing returns

### 2. Make Bit Array Bigger

**Effect on False Positive Rate**:
```
m = 10MB (80M bits):  FP ≈ 0.5%
m = 20MB (160M bits): FP ≈ 0.25%
m = 40MB (320M bits): FP ≈ 0.06%
```

**Trade-offs**:
- ✓ Significantly reduces false positives
- ✗ Increases memory usage
- ✗ May require distributed storage

### 3. Use Multiple Bloom Filters

**Scenario**: Different data types or time-based partitioning

```
Architecture:
┌─────────────────────────────────────────┐
│         Query: "Is user active?"        │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Bloom Filter 1 (Last 24 hours) │    │
│  │  Size: 10MB, k=5                │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Bloom Filter 2 (Last 7 days)   │    │
│  │  Size: 20MB, k=5                │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Bloom Filter 3 (Last 30 days)  │    │
│  │  Size: 40MB, k=5                │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘

Lookup Logic:
  if BF1.lookup(user) == YES:
    return "Active in last 24h"
  else if BF2.lookup(user) == YES:
    return "Active in last 7 days"
  else if BF3.lookup(user) == YES:
    return "Active in last 30 days"
  else:
    return "Inactive"
```

**Benefits**:
- Separate concerns (time-based, type-based)
- Easier to refresh/rotate filters
- Better false positive control per filter

---

## Real-World Use Cases

### 1. Cassandra/PostgreSQL Disk Check

**Problem**: Avoid expensive disk I/O for non-existent keys

```
Query: SELECT * FROM users WHERE id = 'user123'
  ↓
Check Bloom Filter (in memory)
  ├─ "DEFINITELY NOT" → Skip disk I/O, return empty
  └─ "PROBABLY YES" → Proceed with disk lookup
```

**Benefit**: Reduce disk I/O by 90%+ for non-existent keys

### 2. Chrome Malicious URL Check

**Problem**: Check billions of URLs against malicious list without downloading entire database

```
User visits: https://example.com
  ↓
Check local Bloom Filter (downloaded periodically)
  ├─ "DEFINITELY NOT" → Safe, proceed
  └─ "PROBABLY YES" → Query server for verification
```

**Benefit**: Instant local check, minimal bandwidth

### 3. Facebook: Avoid Caching Data

**Problem**: Cache only data that exists to avoid cache misses

```
Cache Lookup: Is user_profile_123 cached?
  ↓
Check Bloom Filter
  ├─ "DEFINITELY NOT" → Don't cache, fetch from DB
  └─ "PROBABLY YES" → Check cache, if miss fetch from DB
```

**Benefit**: Prevent cache pollution, reduce memory waste

### 4. Bitcoin Wallets: Sync Without Full Blockchain

**Problem**: Verify transactions without downloading entire blockchain

```
Wallet: "Do I have transactions in block 12345?"
  ↓
Check Bloom Filter (downloaded with block header)
  ├─ "DEFINITELY NOT" → Skip block
  └─ "PROBABLY YES" → Download and verify block
```

**Benefit**: 99% reduction in data transfer

### 5. Network Routers: DDoS Protection

**Problem**: Identify attack sources in real-time

```
Incoming packet from IP 192.168.1.100
  ↓
Check Bloom Filter of known attack IPs
  ├─ "DEFINITELY NOT" → Allow packet
  └─ "PROBABLY YES" → Rate limit or block
```

**Benefit**: Sub-microsecond filtering, minimal memory

### 6. Username Search

**Problem**: Check username availability without querying database

```
User types: "john_doe"
  ↓
Check Bloom Filter of taken usernames
  ├─ "DEFINITELY NOT" → "Available!" (instant feedback)
  └─ "PROBABLY YES" → Query DB for verification
```

**Benefit**: Instant feedback, reduced DB load

---

## Applications

### 1. URL Shortener

**Use Case**: Check if shortened URL already exists

```
Generate: short_url = "abc123"
  ↓
Check Bloom Filter
  ├─ "DEFINITELY NOT" → Use this URL
  └─ "PROBABLY YES" → Generate new URL
```

**Benefit**: Avoid collisions, instant availability check

### 2. Rate Limiting

**Use Case**: Track IPs/users that exceeded rate limit

```
Request from IP 192.168.1.50
  ↓
Check Bloom Filter of rate-limited IPs
  ├─ "DEFINITELY NOT" → Allow request
  └─ "PROBABLY YES" → Check counter, apply limit
```

**Benefit**: Fast rejection, minimal memory overhead

### 3. Cache System

**Use Case**: Bloom Filter as cache-aside pattern

```
Request for data_key_123
  ↓
Check Bloom Filter
  ├─ "DEFINITELY NOT" → Fetch from source
  └─ "PROBABLY YES" → Check cache, if miss fetch from source
```

**Benefit**: Prevent cache misses, reduce cache pollution

### 4. File Deduplication

**Use Case**: Identify duplicate files in backup system

```
New file: document.pdf (hash: abc123def456)
  ↓
Check Bloom Filter of backed-up files
  ├─ "DEFINITELY NOT" → Back up file
  └─ "PROBABLY YES" → Verify hash, skip if duplicate
```

**Benefit**: Reduce storage by 50%+, fast duplicate detection

---

## Summary

| Aspect | Details |
|--------|---------|
| **Best For** | Membership testing at massive scale |
| **Memory** | O(m) bits, typically 1-10 bits per element |
| **Lookup** | O(k) constant time |
| **False Positives** | Tunable, typically 0.01%-1% |
| **False Negatives** | Zero (guaranteed) |
| **Deletion** | Not supported (use Counting Bloom Filter) |
| **Scalability** | Excellent for billions of elements |

Bloom Filters are essential for building scalable systems where memory efficiency and fast membership testing are critical.
