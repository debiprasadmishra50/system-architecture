# HyperLogLog

## Table of Contents

1. [What is HyperLogLog](#what-is-hyperloglog)
2. [The Cardinality Problem](#the-cardinality-problem)
3. [Why HashMap Doesn't Scale](#why-hashmap-doesnt-scale)
4. [HyperLogLog Solution](#hyperloglog-solution)
5. [Implementation Approaches](#implementation-approaches)
   - [Approach 1: Direct Counting](#approach-1-direct-counting)
   - [Approach 2: M-Buckets with Harmonic Mean](#approach-2-m-buckets-with-harmonic-mean)
6. [Benefits and Trade-offs](#benefits-and-trade-offs)

---

## What is HyperLogLog

HyperLogLog is a probabilistic data structure that estimates the cardinality (number of unique elements) of a dataset using minimal memory. Instead of storing all unique elements, it uses a clever bit-pattern analysis to approximate the count with high accuracy.

**Key Characteristics:**
- Probabilistic algorithm (estimates, not exact counts)
- Constant memory usage: O(log log n)
- Extremely space-efficient
- Suitable for massive datasets

---

## The Cardinality Problem
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.15.30 PM.png' width=600 />

### Definition

Cardinality is the count of unique/distinct elements in a dataset.

### Real-World Example

| ![image](../../../Resources/19-core-building-blocks/Screenshot%202026-02-16%20at%201.16.03 PM.png) | ![image](../../../Resources/19-core-building-blocks/Screenshot%202026-02-16%20at%201.16.11 PM.png) |
|---|---|


**How many unique visitors read a website in a day?**

- Website receives millions of requests
- Same user may visit multiple times
- Need to count unique users, not total visits
- Traditional approach: store all user IDs in a set

### Why This Matters

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.15.37 PM.png' width=500 />

- E-commerce: unique customers per day
- Analytics: unique page views
- Social media: unique followers
- Databases: distinct value counts

---

## Why HashMap Doesn't Scale

### Direct Approach with HashMap

```
HashMap<UserId, Boolean>

Memory Usage = Number of Unique Users × Size per Entry
```

### Scaling Problem

| Unique Users | Memory Required |
|---|---|
| 1 Million | ~40 MB |
| 100 Million | ~4 GB |
| 1 Billion | ~40 GB |
| 1 Trillion | ~40 TB |

**Issues:**
- Linear memory growth with cardinality
- Impractical for massive datasets
- Expensive storage and network transmission
- Not suitable for real-time analytics at scale

---

## HyperLogLog Solution
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.17.42 PM.png' width=600 />

### Core Idea

Instead of storing actual elements, analyze the **bit patterns of their hashes** to estimate cardinality.

**Trade-off:** Accept ~2% error rate for massive memory savings

### Memory Efficiency

```
HyperLogLog Memory = O(log log n)

For 1 Trillion unique elements:
- HashMap: ~40 TB
- HyperLogLog: ~16 KB
```

---

## Implementation Approaches

### Approach 1: Direct Counting

#### Concept

Count the **maximum number of trailing zeros** in the binary representation of hash values.

```
hash(element) = 0b1010100...
                        ↑
                   trailing zeros
```

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.18.37 PM.png' width=650 />

#### Algorithm

1. Hash each element
2. Count trailing zeros in binary representation
3. Track the maximum trailing zeros seen
4. Estimate: `2^(max_trailing_zeros)`

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.19.20 PM.png' width=600 />

#### Example

```
Element: "user123"
hash("user123") = 0b11010100100 (binary)
trailing_zeros = 2
estimate = 2^2 = 4
```
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.20.30 PM.png' width=600 />

#### Problem: High Variance

```
Single max value is unreliable:

Dataset A: [hash1, hash2, hash3]
max_trailing_zeros = 5 → estimate = 32

Dataset B: [hash1, hash2, hash3, hash4]
max_trailing_zeros = 5 → estimate = 32 (same!)

Result: High variance, overestimation
```

---

### Approach 2: M-Buckets with Harmonic Mean
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.22.03 PM.png' width=600 />


#### Concept

Divide hash space into **m buckets** and compute average trailing zeros across all buckets.

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.22.24 PM.png' width=700 />

#### Algorithm

1. **Partition:** Use first k bits of hash to select bucket (m = 2^k buckets)
2. **Count:** For each bucket, track maximum trailing zeros in remaining bits
3. **Average:** Compute harmonic mean of all bucket values
4. **Estimate:** Apply formula with constant

#### Visualization




```
Hash Space Partitioned into M Buckets:

Bucket 0: [max_trailing_zeros = 3]
Bucket 1: [max_trailing_zeros = 5]
Bucket 2: [max_trailing_zeros = 2]
Bucket 3: [max_trailing_zeros = 4]
...
Bucket M: [max_trailing_zeros = 6]

Compute harmonic mean across all buckets
```

#### Formula

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.23.52 PM.png' width=500 />

**Basic Formula:**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.22.49 PM.png' width=500 />

```
Unique Entries = constant × m × 2^(average_trailing_zeros)
```

Where:
- `m` = number of buckets
- `constant` = 0.79 (empirically determined)
- `average_trailing_zeros` = harmonic mean of max trailing zeros per bucket
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.24.40 PM.png' width=700 />

#### Harmonic Mean

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.27.07 PM.png' width=500 />

```
Harmonic Mean = m / (1/x₁ + 1/x₂ + ... + 1/xₘ)

Where x₁, x₂, ..., xₘ are max trailing zeros per bucket
```

#### Improved Formula (with Harmonic Mean)
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.27.29 PM.png' width=700 />

```
Unique Entries = 0.79 × m² / Σ(2^(-max_trailing_zeros[i]))

Where:
- m = number of buckets
- max_trailing_zeros[i] = max trailing zeros in bucket i
```

#### Example Calculation

```
m = 16 buckets
Bucket trailing zeros: [3, 5, 2, 4, 6, 3, 4, 5, 2, 3, 4, 5, 6, 3, 4, 5]

Harmonic Mean = 16 / (1/3 + 1/5 + 1/2 + ... + 1/5)
              ≈ 3.8

Estimate = 0.79 × 16 × 2^3.8
         ≈ 0.79 × 16 × 13.93
         ≈ 176 unique elements
```

#### Advantages Over Approach 1

| Aspect | Approach 1 | Approach 2 |
|---|---|---|
| Variance | High | Low |
| Accuracy | Poor | ~2% error |
| Stability | Unreliable | Reliable |
| Overestimation | Significant | Minimal |

---

## Benefits and Trade-offs

### Benefits
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 1.29.03 PM.png' width=600 />

✅ **Extreme Memory Efficiency**
- O(log log n) space complexity
- 16 KB for 1 trillion elements
- Constant memory regardless of cardinality

✅ **Fast Operations**
- O(1) add operation
- O(1) query operation
- Suitable for real-time analytics

✅ **Mergeable**
- Combine multiple HyperLogLog instances
- Distributed cardinality estimation
- Parallel processing friendly

✅ **Proven Accuracy**
- ~2% standard error
- Acceptable for most use cases
- Empirically validated

### Trade-offs

❌ **Probabilistic (Not Exact)**
- Estimates, not precise counts
- Not suitable for exact cardinality requirements
- Error margin of ~2%

❌ **Complex Implementation**
- Requires understanding of bit operations
- Careful tuning of bucket count
- Harmonic mean computation

❌ **Limited Use Cases**
- Only for cardinality estimation
- Cannot retrieve actual elements
- Cannot support range queries

### When to Use HyperLogLog

| Scenario | Suitable? | Reason |
|---|---|---|
| Unique daily visitors | ✅ Yes | 2% error acceptable, massive scale |
| Exact user count | ❌ No | Requires precision |
| Distinct product IDs | ✅ Yes | Approximate count sufficient |
| Fraud detection | ❌ No | Needs exact data |
| Analytics dashboards | ✅ Yes | Approximate metrics acceptable |

---

## Real-World Applications

- **Redis:** `PFADD`, `PFCOUNT` commands
- **Elasticsearch:** Cardinality aggregations
- **Google Analytics:** Unique visitor estimation
- **Apache Spark:** Approximate distinct counts
- **Stream processing:** Real-time unique element tracking
