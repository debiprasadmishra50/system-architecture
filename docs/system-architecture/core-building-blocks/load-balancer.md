# Load Balancer

## Table of Contents
1. [What is Load Balancing](#what-is-load-balancing)
2. [Problems It Solves](#problems-it-solves)
   - [Single Point of Failure](#1-single-point-of-failure)
   - [Uneven Load Distribution](#2-uneven-load-distribution)
   - [Scalability Limitations](#3-scalability-limitations)
   - [Performance Degradation](#4-performance-degradation)
   - [Resource Utilization](#5-resource-utilization)
3. [Types of Load Balancers](#types-of-load-balancers)
   - [Hardware Load Balancers](#1-hardware-load-balancers)
   - [Software Load Balancers](#2-software-load-balancers)
   - [Cloud Load Balancers](#3-cloud-load-balancers)
   - [Layer 4 (Transport Layer) Load Balancers](#4-layer-4-transport-layer-load-balancers)
   - [Layer 7 (Application Layer) Load Balancers](#5-layer-7-application-layer-load-balancers)
4. [Load Balancing Algorithms](#load-balancing-algorithms)
   - [Random](#1-random)
   - [Round Robin](#2-round-robin)
   - [Weighted Round Robin](#3-weighted-round-robin)
   - [Server Traffic (Least Connections / Least Load)](#4-server-traffic-least-connections--least-load)
   - [IP-Based (Source IP Hash)](#5-ip-based-source-ip-hash)
   - [Algorithm Comparison](#algorithm-comparison)
5. [Diagram: Load Balancer Architecture](#diagram-load-balancer-architecture)
6. [Diagram: Load Balancing Algorithms Comparison](#diagram-load-balancing-algorithms-comparison)
7. [Key Considerations](#key-considerations)

---

## What is Load Balancing

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 10.48.51 AM.png' width=600 />

- **Definition**: A technique that distributes incoming network traffic and requests across multiple servers to optimize resource utilization, maximize throughput, minimize response time, and avoid overload on any single server
- **Core Function**: Acts as an intermediary between clients and backend servers, receiving requests and forwarding them to appropriate servers
- **Placement**: Typically positioned between clients and application servers, or between application servers and databases
- **Transparency**: Clients communicate with load balancer as if it's the actual service; backend servers are hidden from clients

---

## Problems It Solves

### 1. **Single Point of Failure**
- Eliminates dependency on a single server
- If one server fails, traffic automatically routes to healthy servers
- Ensures service availability and high uptime

### 2. **Uneven Load Distribution**
- Prevents some servers from being overloaded while others remain idle
- Ensures all servers handle proportional amounts of traffic
- Improves overall system efficiency

### 3. **Scalability Limitations**
- Enables horizontal scaling by adding more servers
- Distributes load across multiple machines instead of upgrading a single server
- Cost-effective scaling approach

### 4. **Performance Degradation**
- Reduces response time by distributing requests
- Prevents bottlenecks at any single server
- Improves user experience through faster request processing

### 5. **Resource Utilization**
- Maximizes CPU, memory, and network bandwidth usage across servers
- Prevents resource wastage on underutilized servers
- Optimizes infrastructure investment

---

## Types of Load Balancers

### 1. **Hardware Load Balancers**
- **Description**: Physical devices that sit between clients and servers
- **Advantages**:
  - High performance and throughput
  - Can handle millions of requests per second
  - Dedicated hardware for load balancing
- **Disadvantages**:
  - Expensive upfront cost
  - Requires physical space and maintenance
  - Less flexible for dynamic scaling
- **Examples**: F5 BIG-IP, Citrix NetScaler, Radcom

### 2. **Software Load Balancers**
- **Description**: Applications running on standard servers that distribute traffic
- **Advantages**:
  - Cost-effective and flexible
  - Easy to scale and modify
  - Can run on commodity hardware
- **Disadvantages**:
  - Consumes server resources
  - May have lower throughput than hardware solutions
  - Requires careful configuration
- **Examples**: Nginx, HAProxy, Apache mod_proxy

### 3. **Cloud Load Balancers**
- **Description**: Managed load balancing services provided by cloud platforms
- **Advantages**:
  - Fully managed and highly available
  - Auto-scaling capabilities
  - Pay-as-you-go pricing model
  - Integrated with cloud infrastructure
- **Disadvantages**:
  - Vendor lock-in
  - Less control over configuration
  - Potential latency from cloud provider
- **Examples**: AWS ELB/ALB/NLB, Google Cloud Load Balancer, Azure Load Balancer

### 4. **Layer 4 (Transport Layer) Load Balancers**
- **Description**: Operates at TCP/UDP level, makes decisions based on IP protocol data
- **Advantages**:
  - Ultra-fast performance
  - Lower latency
  - Minimal processing overhead
- **Use Cases**: High-frequency trading, gaming, real-time applications
- **Limitations**: Cannot inspect application-level data

### 5. **Layer 7 (Application Layer) Load Balancers**
- **Description**: Operates at HTTP/HTTPS level, understands application protocols
- **Advantages**:
  - Intelligent routing based on URL, hostname, headers, cookies
  - Can route based on content type
  - Better for microservices architectures
- **Use Cases**: Web applications, API gateways, microservices
- **Limitations**: Higher latency due to deeper packet inspection

---

## Load Balancing Algorithms

Load balancers use various algorithms to decide which server receives each request. The choice depends on traffic patterns, server capabilities, and application requirements.

### 1. **Random**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.01.20 AM.png' width=500 />

```
Request 1 → Random Selection → Server A
Request 2 → Random Selection → Server C
Request 3 → Random Selection → Server B
Request 4 → Random Selection → Server A
```

- **How It Works**: Randomly selects a server from the available pool for each request
- **Advantages**:
  - Simple to implement
  - No state tracking required
  - Works well with uniform server capacity
- **Disadvantages**:
  - Uneven distribution possible
  - No consideration for server load or health
  - May send requests to overloaded servers
- **Best For**: Small deployments with identical servers and uniform traffic

---

### 2. **Round Robin**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.01.37 AM.png' width=500 />

```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A (cycles back)
Request 5 → Server B
```

- **How It Works**: Distributes requests sequentially across servers in a circular manner
- **Advantages**:
  - Fair distribution across all servers
  - Simple and predictable
  - No overhead for tracking server state
- **Disadvantages**:
  - Ignores server capacity differences
  - Doesn't account for current server load
  - May overload slower servers
- **Best For**: Servers with similar capacity and uniform request processing time

---

### 3. **Weighted Round Robin**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.01.49 AM.png' width=500 />

```
Server A (weight: 5) → 5 requests
Server B (weight: 3) → 3 requests
Server C (weight: 2) → 2 requests
(Then cycle repeats)
```

- **How It Works**: Assigns weights to servers; servers with higher weights receive more requests
- **Advantages**:
  - Accommodates servers with different capacities
  - More flexible than standard round robin
  - Predictable distribution
- **Disadvantages**:
  - Requires manual weight configuration
  - Weights must be adjusted if server capacity changes
  - Still doesn't adapt to real-time load
- **Best For**: Heterogeneous server environments with known capacity differences

---

### 4. **Server Traffic (Least Connections / Least Load)**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.02.08 AM.png' width=500 />

```
Current Connections:
Server A: 10 connections
Server B: 5 connections  ← Selected (least loaded)
Server C: 8 connections

New Request → Server B
```

- **How It Works**: Routes requests to the server with the fewest active connections or lowest current load
- **Advantages**:
  - Adapts to real-time server load
  - Prevents overloading busy servers
  - Better performance than static algorithms
- **Disadvantages**:
  - Requires tracking connection count
  - More computational overhead
  - May not account for request complexity
- **Best For**: Long-lived connections, persistent sessions, variable request processing times

---

### 5. **IP-Based (Source IP Hash)**
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.03.02 AM.png' width=500 />

```
Client IP: 192.168.1.100 → Hash → Server A
Client IP: 192.168.1.101 → Hash → Server B
Client IP: 192.168.1.100 → Hash → Server A (same client, same server)
```

- **How It Works**: Uses client's IP address to determine which server receives the request; same client always routes to same server
- **Advantages**:
  - Ensures session persistence without sticky cookies
  - Enables client-side caching
  - Useful for stateful applications
  - Consistent routing for same client
- **Disadvantages**:
  - Uneven distribution if client IPs are skewed
  - Cannot adapt to server load changes
  - Difficult to scale (adding servers changes hash mapping)
- **Best For**: Stateful applications, session persistence, caching scenarios

---

## Algorithm Comparison

| Algorithm | Distribution | Adaptive | Session Persistence | Complexity | Best Use Case |
|-----------|--------------|----------|----------------------|------------|---------------|
| Random | Fair (probabilistic) | No | No | Low | Small uniform deployments |
| Round Robin | Fair (deterministic) | No | No | Low | Uniform servers, stateless apps |
| Weighted Round Robin | Fair (weighted) | No | No | Low | Heterogeneous servers |
| Least Connections | Adaptive | Yes | No | Medium | Long-lived connections |
| IP-Based Hash | Depends on IPs | No | Yes | Medium | Stateful apps, session persistence |

---

## Diagram: Load Balancer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                              │
│              (Web Browsers, Mobile Apps, APIs)              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ HTTP/HTTPS Requests
                         ▼
        ┌────────────────────────────────┐
        │     Load Balancer              │
        │  (Distributes Traffic)         │
        │  - Health Checks               │
        │  - Routing Algorithm           │
        │  - Session Management          │
        └────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    ┌────────┐      ┌────────┐      ┌────────┐
    │Server 1│      │Server 2│      │Server 3│
    │(Active)│      │(Active)│      │(Active)│
    └────────┘      └────────┘      └────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                    ┌─────────────┐
                    │  Database   │
                    │  (Shared)   │
                    └─────────────┘
```

---

## Diagram: Load Balancing Algorithms Comparison

```
Round Robin:
Request 1 → [Server A] ← Request 4
Request 2 → [Server B] ← Request 5
Request 3 → [Server C] ← Request 6

Least Connections:
Server A: 5 connections
Server B: 2 connections ← Request → [Server B]
Server C: 8 connections

IP Hash:
Client 192.168.1.1 → Hash → [Server A]
Client 192.168.1.2 → Hash → [Server B]
Client 192.168.1.1 → Hash → [Server A] (consistent)
```

---

## Key Considerations

- **Health Checks**: Load balancers periodically verify server health; unhealthy servers are removed from rotation
- **Session Persistence**: Some applications require requests from same client to go to same server (sticky sessions)
- **Failover**: When a server fails, load balancer automatically redirects traffic to healthy servers
- **Scalability**: Adding/removing servers should be seamless without disrupting existing connections
- **Geographic Distribution**: Global load balancers can route traffic based on geographic location for reduced latency
