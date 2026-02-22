# Load Balancer: How Load Balancers Work

## Table of Contents
1. [What is a Load Balancer](#what-is-a-load-balancer)
2. [Why Load Balancers are Essential](#why-load-balancers-are-essential)
3. [How Load Balancers Work](#how-load-balancers-work)
4. [Load Balancing Algorithms](#load-balancing-algorithms)
5. [Types of Load Balancers](#types-of-load-balancers)
6. [Load Balancer Architecture](#load-balancer-architecture)

---

## What is a Load Balancer

- **Definition**: A system or device that distributes incoming network traffic across multiple backend servers
- **Purpose**: Ensures no single server becomes a bottleneck and improves overall system reliability
- **Position**: Sits between clients and backend servers, acting as a reverse proxy
- **Responsibility**: Routes requests intelligently to available servers based on configured algorithms
- **Transparency**: Clients communicate with the load balancer's IP/domain, unaware of backend server details

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-08 at 11.26.42 PM.png' width='500' />

---

## Why Load Balancers are Essential

- **Scalability**: Distribute load across multiple servers to handle increased traffic
- **High Availability**: If one server fails, traffic is automatically redirected to healthy servers
- **Performance**: Reduce response times by distributing requests evenly
- **Fault Tolerance**: System continues operating even when individual servers fail
- **Maintenance**: Take servers offline for updates without disrupting service
- **Geographic Distribution**: Route traffic to geographically closest servers for reduced latency
- **Security**: Hide backend server details and act as a security barrier

---

## How Load Balancers Work

### Request Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         REQUEST FLOW                            │
└─────────────────────────────────────────────────────────────────┘

    ┌──────────┐
    │  Client  │
    └────┬─────┘
         │ 1. HTTP/HTTPS Request
         │
    ┌────▼──────────────────┐
    │  Load Balancer        │
    │  ┌──────────────────┐ │
    │  │ 3. Algorithm     │ │
    │  │    Decision      │ │
    │  └──────────────────┘ │
    └────┬──────────────────┘
         │ 4. Forward Request
         │
    ┌────┴─────────────────────────────────────┐
    │                                          │
┌───▼────┐  ┌──────────┐  ┌──────────┐  ┌──────▼───┐
│Backend │  │ Backend  │  │ Backend  │  │ Backend  │
│Server 1│  │ Server 2 │  │ Server 3 │  │ Server N │
└───┬────┘  └────┬─────┘  └────┬─────┘  └──────┬───┘
    │ 5. Process Request       │               │
    │ 6. Generate Response     │               │
    │                          │               │
    └──────────────┬───────────┴───────────────┘
                   │ 6. Response to Load Balancer
                   │
              ┌────▼──────────────────┐
              │  Load Balancer        │
              │  (Forwards Response)  │
              └────┬──────────────────┘
                   │ 7. HTTP Response
                   │
              ┌────▼─────┐
              │  Client  │
              └──────────┘
```

1. **Client Request**: Client sends HTTP/HTTPS request to load balancer's IP address or domain
2. **Load Balancer Receives**: Load balancer receives the incoming request
3. **Algorithm Decision**: Load balancer applies its configured algorithm to select a backend server
4. **Request Forwarding**: Load balancer forwards the request to the selected backend server
5. **Backend Processing**: Backend server processes the request and generates a response
6. **Response Return**: Backend server sends response back to load balancer
7. **Client Response**: Load balancer forwards response to the client
8. **Connection Handling**: Load balancer may maintain or close the connection based on configuration

### Key Mechanisms

- **Connection Multiplexing**: Load balancer maintains connections to multiple backend servers
- **Request Routing**: Examines request headers, URLs, or other criteria to make routing decisions
- **State Management**: Tracks active connections and server availability
- **Timeout Handling**: Manages connection timeouts and retries
- **Logging**: Records traffic patterns for monitoring and debugging

---

## Load Balancing Algorithms

### Round Robin
- **How it works**: Distributes requests sequentially to each server in order
- **Advantages**: Simple, fair distribution, no server state needed
- **Disadvantages**: Doesn't account for server capacity or current load
- **Use case**: Servers with similar capacity and performance

### Weighted Round Robin
- **How it works**: Assigns weights to servers; servers with higher weights receive more requests
- **Advantages**: Accounts for different server capacities
- **Disadvantages**: Still doesn't consider current server load
- **Use case**: Heterogeneous server environments with different hardware specs

### Least Connections
- **How it works**: Routes request to server with fewest active connections
- **Advantages**: Adapts to current server load, good for long-lived connections
- **Disadvantages**: Doesn't consider connection duration or server processing time
- **Use case**: WebSocket connections, persistent connections, streaming

### Weighted Least Connections
- **How it works**: Combines least connections with server weights
- **Advantages**: Accounts for both server capacity and current load
- **Disadvantages**: More complex to implement
- **Use case**: Mixed server capacities with varying connection types

### IP Hash
- **How it works**: Hashes client IP address to determine backend server
- **Advantages**: Same client always routes to same server, no session state needed
- **Disadvantages**: Uneven distribution if many clients share same IP, doesn't adapt to load
- **Use case**: Session persistence without sticky sessions, cache locality

### Least Response Time
- **How it works**: Routes to server with fastest average response time and fewest connections
- **Advantages**: Optimizes for user experience and server efficiency
- **Disadvantages**: Requires monitoring response times, more overhead
- **Use case**: Performance-critical applications

### Random
- **How it works**: Randomly selects a backend server
- **Advantages**: Simple, good distribution with many requests
- **Disadvantages**: No optimization, may overload some servers
- **Use case**: Simple scenarios with many requests

### Resource-Based
- **How it works**: Routes based on server resource availability (CPU, memory, disk)
- **Advantages**: Prevents overloading resource-constrained servers
- **Disadvantages**: Requires real-time resource monitoring
- **Use case**: Heterogeneous environments with varying resource constraints

---

## Types of Load Balancers

### Layer 4 (Transport Layer) Load Balancers
- **Protocol**: TCP/UDP level
- **Decision Criteria**: Source IP, destination IP, port numbers
- **Speed**: Very fast, minimal processing
- **Use Cases**: 
  - High-performance scenarios
  - Non-HTTP protocols (databases, gaming)
  - Extreme throughput requirements
- **Examples**: AWS Network Load Balancer (NLB), HAProxy (TCP mode)

### Layer 7 (Application Layer) Load Balancers
- **Protocol**: HTTP/HTTPS level
- **Decision Criteria**: URL paths, hostnames, HTTP headers, request content
- **Capabilities**:
  - Content-based routing
  - SSL/TLS termination
  - Request rewriting
  - Advanced health checks
- **Trade-off**: More intelligent but slightly higher latency
- **Use Cases**:
  - Microservices routing
  - API gateways
  - Content-based routing
- **Examples**: AWS Application Load Balancer (ALB), Nginx, Apache

### Hardware Load Balancers
- **Implementation**: Dedicated physical devices
- **Advantages**: High performance, enterprise features, redundancy
- **Disadvantages**: Expensive, requires physical space, less flexible
- **Examples**: F5 BIG-IP, Citrix NetScaler, Radcom

### Software Load Balancers
- **Implementation**: Software running on standard servers
- **Advantages**: Cost-effective, flexible, easy to scale
- **Disadvantages**: Consumes server resources, potential single point of failure
- **Examples**: Nginx, HAProxy, Apache, Envoy

### Cloud Load Balancers
- **Implementation**: Managed services provided by cloud providers
- **Advantages**: Automatic scaling, high availability, integrated with cloud services
- **Disadvantages**: Vendor lock-in, less control
- **Examples**: AWS ELB/ALB/NLB, Google Cloud Load Balancer, Azure Load Balancer

---

## Load Balancer Architecture

### Single Load Balancer
```
Clients → Load Balancer → Backend Servers
```
- **Limitation**: Load balancer itself becomes single point of failure

### Redundant Load Balancers (Active-Passive)
```
Clients → Load Balancer 1 (Active)
       ↘ Load Balancer 2 (Passive/Standby)
         → Backend Servers
```
- **How it works**: Primary load balancer handles traffic; secondary takes over if primary fails
- **Detection**: Heartbeat mechanism detects primary failure
- **Failover**: Virtual IP (VIP) switches to secondary load balancer
- **Advantages**: High availability for load balancer itself
- **Disadvantages**: Secondary resources idle during normal operation

### Redundant Load Balancers (Active-Active)
```
Clients → Load Balancer 1 → Backend Servers
       ↘ Load Balancer 2 ↗
```
- **How it works**: Both load balancers actively handle traffic
- **Distribution**: DNS or anycast routes clients to both
- **Advantages**: Better resource utilization, no idle capacity
- **Disadvantages**: More complex, requires careful synchronization

### Geographic Load Balancing
```
Clients (US) → US Load Balancer → US Backend Servers
Clients (EU) → EU Load Balancer → EU Backend Servers
```
- **How it works**: Route clients to geographically closest data center
- **Benefits**: Reduced latency, improved performance
- **Implementation**: DNS-based routing, GeoDNS, or global load balancer

---

## Architect's Perspective

- **Choose Algorithm**: Match algorithm to application characteristics (connection type, server capacity, response time)
- **Layer Selection**: Use Layer 4 for extreme performance, Layer 7 for intelligent routing
- **Redundancy**: Always implement redundant load balancers for production systems
- **Session Management**: Prefer centralized session stores or stateless design over sticky sessions
- **Monitoring**: Implement comprehensive health checks and monitoring
- **Scaling**: Load balancers should scale horizontally; avoid them becoming bottlenecks
- **Security**: Use load balancers for SSL/TLS termination and DDoS protection
- **Testing**: Load test to ensure load balancer can handle peak traffic

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
