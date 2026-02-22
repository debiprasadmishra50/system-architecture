# Hashing & Consistent Hashing

## Table of Contents
1. [Hashing and Server Selection](#hashing-and-server-selection)
2. [Consistent Hashing to the Rescue](#consistent-hashing-to-the-rescue)
    1. [Problem Statement](#problem-statement)
    2. [Horizontal Partitioning (Sharding)](#horizontal-partitioning-sharding)
    3. [Design Algorithms](#design-algorithms)
    4. [Virtual Nodes Solution](#virtual-nodes-solution)
    5. [Real-World Applications](#real-world-applications)
3. [Summary](#summary)

## Hashing and Server Selection

### Hashing Fundamentals

**What is Hashing?**
- One-way function converting input to fixed-size hash value
- Deterministic: same input always produces same output
- Fast computation
- Avalanche effect: small input change produces completely different hash

**Common Hash Algorithms**:
- **MD5**: 128-bit hash, cryptographically broken, fast but insecure
- **SHA (SHA-1, SHA-256)**: Secure Hash Algorithm, SHA-256 produces 256-bit hash, widely used
- **Argon2**: Memory-hard password hashing, resistant to GPU/ASIC attacks, modern standard
- **Bcrypt**: Adaptive password hashing with salt, slower by design, good for passwords

**Cracking Tools**:
- **Hashcat**: GPU-accelerated hash cracking, supports 300+ algorithms
- **John the Ripper**: CPU-based password cracker, dictionary and brute-force attacks

### Server Selection Strategies by Load Balancers

| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.24.25 PM.png) | → | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.26.42 PM.png) |
| :---: | :---: | :---: |

**Random Strategy**
```
hash(request_id) % number_of_servers = server_index
```
- **Problem**: No consistency - same user's requests go to different servers
- **Impact**: Cache misses, session loss, longer request execution time
- **Example**: User A's request goes to Server 1 (cache hit), next request goes to Server 2 (cache miss, 10x slower)

**Round Robin Strategy**
```
request_counter % number_of_servers = server_index
```
- **Problem**: Doesn't account for server capacity or current load
- **Impact**: Uneven distribution, some servers overloaded while others idle
- **Cache Issue**: Requests distributed evenly but no locality, cache ineffective

**Modulo Strategy (Simple Hashing)**
| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.30.14 PM.png) |  ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.36.48 PM.png) | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.44.13 PM.png) |
| :---: | :---: | :---: |

```
hash(user_id) % number_of_servers = server_index
```
- **Advantage**: Same user always routes to same server
- **Benefit**: Cache locality, session persistence, faster execution
- **Problem**: When servers added/removed, hash % changes, all mappings break
- **Example**: 
  - With 3 servers: user_id=5 → hash(5) % 3 = 2 (Server 2)
  - Add 1 server (4 total): hash(5) % 4 = 1 (Server 1) - CACHE INVALIDATED
  - 75% of users remapped to different servers

**Horizontal Scaling Problem**
```
- When adding new servers, modulo value changes
- Requires recomputing hash for every user to find new server
- Massive cache invalidation and data redistribution
- Expensive operation: O(n) where n = number of users/keys
```

---

## Consistent Hashing to the Rescue

### Problem Statement
- Simple hashing breaks when cluster size changes
- Need: Minimal remapping when servers added/removed
- Goal: Only affected keys should move to new servers

### Horizontal Partitioning (Sharding)


| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.32.52 PM.png) | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.37.44 PM.png) | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.38.53 PM.png) |
| :---: | :---: | :---: |

**Definition**: Splitting data across multiple servers based on key ranges or hash values

**Key Goals of Sharding**:
1. **Even Distribution**: Data spread uniformly across shards
2. **Add Shards**: New servers added without massive remapping
3. **Delete/Failed Shards**: Handle server failures gracefully

### Design Algorithms

**Brute Force / Simple Hashing Approach**

```
server_index = hash(key) % number_of_servers
```
- **Pros**: Simple, fast
- **Cons**: Breaks on cluster changes, massive remapping required

**Number Line Approach (Consistent Hashing)**

**Concept**: Arrange servers on a circular number line (hash ring)
- Hash space: 0 to 2^32 - 1 (or 2^64)
- Servers placed at hash positions on ring
- Key assigned to next server clockwise on ring

**Algorithm**:

1. Hash each server to position on ring: hash(server_id) → position

| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.47.17 PM.png) | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.47.28 PM.png) | ![alt](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.48.42 PM.png) |
| :---: | :---: | :---: | 

2. Hash each key to position on ring: hash(key) → position

| ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.46.39 PM.png) | ![alt](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.49.17 PM.png) |
| :---: | :---: |

3. Find next server clockwise from key position
4. That server stores the key
5. When a server is added or removed, only keys between the two nearest servers need to be remapped

<img src="../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.50.22 PM.png" width="500" />


**Example**:
```
Ring positions: 0 -------- 100 -------- 200 -------- 300 -------- 360
Servers:        S1(30)     S2(120)      S3(250)
Key K1(45):     → S2 (next clockwise)
Key K2(150):    → S3 (next clockwise)
Key K3(280):    → S1 (wraps around)
```

**Advantages**:
- When server added: only keys between new server and previous server remapped
- When server removed: keys redistribute to next servers
- Minimal disruption: O(k/n) keys remapped where k=total keys, n=number of servers

**Limitations**:
- **Uneven Distribution**: Servers placed randomly may create hot spots
- **Server Assignment**: Some servers get more keys than others
- **Load Imbalance**: One server may have 10x more data than another

### Virtual Nodes Solution

**Problem**: Random server placement causes uneven distribution

**Solution**: Each physical server represented by multiple virtual nodes on ring

<img src="../../Resources/17-system-architecture-basics/Screenshot 2026-02-08 at 11.52.33 PM.png" width="500" />

```
Physical Server S1 → Virtual Nodes: S1_v1, S1_v2, S1_v3, ... S1_v150
Physical Server S2 → Virtual Nodes: S2_v1, S2_v2, S2_v3, ... S2_v150
Physical Server S3 → Virtual Nodes: S3_v1, S3_v2, S3_v3, ... S3_v150
```

**Benefits**:
- More virtual nodes = better distribution
- When server fails, load distributed across all remaining servers
- When server added, load taken from all existing servers proportionally
- Typical: 150-200 virtual nodes per physical server

**Example**:
```
Without Virtual Nodes:
S1(30) → 40% keys, S2(120) → 20% keys, S3(250) → 40% keys (uneven)

With Virtual Nodes (5 each):
S1_v1(30), S1_v2(45), S1_v3(60), S1_v4(75), S1_v5(90)
S2_v1(120), S2_v2(140), S2_v3(160), S2_v4(180), S2_v5(200)
S3_v1(250), S3_v2(270), S3_v3(290), S3_v4(310), S3_v5(330)
→ Each server gets ~33% keys (even distribution)
```

### Real-World Applications

**Amazon DynamoDB**
- Uses consistent hashing with virtual nodes
- Partitions data across multiple servers
- Handles millions of requests per second
- Automatic failover and replication

**Apache Cassandra**
- Distributed NoSQL database
- Consistent hashing for data distribution
- Virtual nodes for even load balancing
- Peer-to-peer architecture, no single point of failure

**Content Delivery Networks (CDN)**
- Distributes content across edge servers globally
- Consistent hashing determines which edge server caches content
- User requests routed to nearest server
- Minimizes latency for end users

**Load Balancers**
- Distribute incoming requests across backend servers
- Consistent hashing ensures session persistence
- User requests always go to same server (if available)
- Reduces cache misses and improves performance
- Example: Nginx, HAProxy use consistent hashing algorithms

**Use Case Example - E-commerce Platform**:
```
User Session Persistence:
- User logs in, routed to Server A (via consistent hash)
- Shopping cart cached on Server A
- Next request: consistent hash routes to Server A again
- Cart data available immediately (cache hit)
- Without consistent hashing: cart data lost, user re-authenticates (poor UX)
```

---

## Summary

**Architecture Evolution**:
1. **Single Server**: Simple but limited scale
2. **Multiple Servers + Random/Round Robin**: Load distributed but cache ineffective
3. **Multiple Servers + Simple Hashing**: Cache works but breaks on scaling
4. **Consistent Hashing**: Scales efficiently with minimal remapping
5. **Consistent Hashing + Virtual Nodes**: Even distribution at scale

**Key Takeaway**: Consistent hashing is fundamental to modern distributed systems, enabling horizontal scaling without massive data redistribution or performance degradation.

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
