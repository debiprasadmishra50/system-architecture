# System Architecture Basics

## Table of Contents
1. [Client Server Architecture](#client-server-architecture)
2. [Network Protocols: IP, TCP, HTTP](#network-protocols-ip-tcp-http)
3. [✅ Database and Storage](./18-database-and-storage.md)
4. [Distributed Systems](#distributed-systems)
5. [Latency vs Throughput](#latency-vs-throughput)
6. [Consistent Hashing](./19-consistent-hashing.md)
7. [CAP Theorem](#cap-theorem)
8. [Load Balancer](./21-load-balancer.md)
9. [Proxy, Forward Proxy, and Reverse Proxy](#proxy-forward-proxy-and-reverse-proxy)
10. [Caching: LRU and LFU](./20-caching.md)
11. [Polling vs Streaming](#polling-vs-streaming)
12. [Publisher-Subscribe Pattern: PUBSUB](./23-pubsub-pattern.md)
13. [Data Replication](./22-data-replication.md)
14. [Partitioning and Sharding](./24-partitioning-and-sharding.md)
15. [Peer-to-Peer Network](./25-peer-to-peer-network.md)
16. [WebSockets](./26-websockets.md)
17. [Rate Limiting and Throttling](./27-rate-limiting-and-throttling.md)
18. [✅ Availability: SLI, SLO, SLA & Error Budgets](./28-availability-sli-slo-sla.md)
19. [✅ Latency](./29-latency.md)
20. [✅ Scalability](./30-scalability.md)
21. [✅ Consistency and CAP Theorem](./31-consistency-and-cap-theorem.md)
22. [✅ Durability / Disaster Recovery](./32-durability.md)
23. [✅ Read/Write Ratio](./33-read-write-ratio.md)
24. [✅ Fault Tolerance, Resilience & Reliability](./34-fault-tolerance-resilience-reliability.md)
25. [✅ Architecture Decision Matrix ✅](./architecture-decision-matrix.md)

---

## Client Server Architecture

- **Definition**: A distributed application architecture that separates concerns between clients (requesters) and servers (providers)
- **Client Role**: Initiates requests, handles user interface, processes user input
- **Server Role**: Receives requests, processes business logic, manages data, sends responses
- **Communication**: Typically uses HTTP/HTTPS protocols over network
- **Advantages**: 
  - Centralized data management
  - Easier to update and maintain server-side logic
  - Scalable - can add more servers
- **Disadvantages**:
  - Single point of failure if server goes down
  - Network latency between client and server
  - Server becomes bottleneck under high load

---

## Network Protocols: IP, TCP, HTTP

### Network Protocols Overview

Network protocols are standardized rules that enable communication between computers over networks. They define how data is formatted, transmitted, and received.

### Common Network Protocols List

- **IP (Internet Protocol)** - Routes and delivers data packets across networks using logical addressing
- **TCP (Transmission Control Protocol)** - Ensures reliable, ordered delivery of data with connection management
- **UDP (User Datagram Protocol)** - Fast, connectionless protocol for speed-critical applications with no delivery guarantee
- **DNS (Domain Name System)** - Translates domain names (example.com) into IP addresses for network routing
- **HTTP (HyperText Transfer Protocol)** - Application protocol for transferring web pages and resources over the internet
- **HTTPS (HTTP Secure)** - Encrypted version of HTTP using SSL/TLS for secure web communication
- **FTP (File Transfer Protocol)** - Transfers files between computers over a network with authentication
- **SMTP (Simple Mail Transfer Protocol)** - Sends emails from clients to mail servers and between mail servers
- **POP3 (Post Office Protocol 3)** - Retrieves emails from mail servers to client devices
- **IMAP (Internet Message Access Protocol)** - Manages emails on mail servers with folder synchronization
- **SSH (Secure Shell)** - Provides secure remote access and command execution on remote servers
- **Telnet** - Unencrypted remote access protocol (deprecated in favor of SSH)
- **DHCP (Dynamic Host Configuration Protocol)** - Automatically assigns IP addresses to devices on a network
- **ARP (Address Resolution Protocol)** - Maps IP addresses to MAC addresses for local network communication
- **ICMP (Internet Control Message Protocol)** - Sends diagnostic messages like ping and traceroute
- **MQTT (Message Queuing Telemetry Transport)** - Lightweight publish-subscribe protocol for IoT and real-time messaging
- **WebSocket** - Enables full-duplex communication over a single TCP connection for real-time applications
- **gRPC** - High-performance RPC framework using HTTP/2 for microservices communication
- **QUIC** - Modern transport protocol combining benefits of TCP and UDP with faster connection establishment

---

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-10 at 5.35.05 PM.png' width='500' />

#### IP (Internet Protocol)

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-10 at 5.33.40 PM.png' width='500' />

- **Definition**: Fundamental protocol responsible for routing and delivering data packets across networks to their destination
- **Version**: IPv4 (32-bit addresses) and IPv6 (128-bit addresses)
- **Function**: Provides logical addressing (IP addresses) and packet routing
- **Connectionless**: Doesn't establish connection before sending data; each packet is independent
- **Unreliable**: No guarantee that packets arrive or arrive in order (relies on higher-layer protocols)
- **Example**: 192.168.1.1 (IPv4) or 2001:0db8:85a3::8a2e:0370:7334 (IPv6)

#### TCP (Transmission Control Protocol)

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-10 at 5.34.09 PM.png' width='500' />

- **Definition**: Connection-oriented protocol that ensures reliable, ordered delivery of data packets
- **Connection**: Establishes connection (three-way handshake) before data transmission
- **Reliability**: Guarantees all packets arrive and in correct order; retransmits lost packets
- **Flow Control**: Manages data transmission rate to prevent overwhelming receiver
- **Error Checking**: Detects and corrects transmission errors
- **Use Cases**: Email (SMTP), File Transfer (FTP), Remote Access (SSH), Web (HTTP/HTTPS)
- **Overhead**: Higher overhead due to connection management and reliability features

#### HTTP (HyperText Transfer Protocol)

- **Definition**: Application-layer protocol for transferring hypertext documents and resources over the web
- **Request-Response**: Client sends request, server sends response; stateless communication
- **Methods**: GET (retrieve), POST (submit), PUT (update), DELETE (remove), PATCH (partial update)
- **Status Codes**: 1xx (informational), 2xx (success), 3xx (redirection), 4xx (client error), 5xx (server error)
- **Stateless**: Each request is independent; server doesn't retain client state between requests
- **HTTPS**: Secure version using SSL/TLS encryption for data protection
- **Versions**: HTTP/1.1 (persistent connections), HTTP/2 (multiplexing), HTTP/3 (QUIC protocol)
- **Use Cases**: Web browsing, REST APIs, Web services

### Protocol Stack Relationship

```
┌─────────────────────────────────────┐
│  Application Layer (HTTP, HTTPS)    │
│  - Web requests, responses          │
│  - Resource transfer                │
├─────────────────────────────────────┤
│  Transport Layer (TCP)              │
│  - Reliable delivery                │
│  - Connection management            │
│  - Error checking                   │
├─────────────────────────────────────┤
│  Internet Layer (IP)                │
│  - Routing                          │
│  - Logical addressing               │
│  - Packet forwarding                │
├─────────────────────────────────────┤
│  Link Layer (Ethernet, WiFi)        │
│  - Physical transmission            │
│  - MAC addressing                   │
└─────────────────────────────────────┘
```

### Comparison: IP vs TCP vs HTTP

| Aspect | IP | TCP | HTTP |
|--------|----|----|------|
| **Layer** | Network (Layer 3) | Transport (Layer 4) | Application (Layer 7) |
| **Purpose** | Routing and addressing | Reliable delivery | Web communication |
| **Connection** | Connectionless | Connection-oriented | Stateless (over TCP) |
| **Reliability** | Unreliable | Reliable | Depends on TCP |
| **Ordering** | No guarantee | Guaranteed order | Guaranteed order |
| **Speed** | Fast | Slower (overhead) | Depends on TCP |
| **Use Case** | All network communication | Reliable data transfer | Web browsing, APIs |
| **Example** | Routing packets to 192.168.1.1 | Email, SSH, FTP | GET /api/users HTTP/1.1 |

### Architect's Perspective: Network Protocols

- **IP**: Foundation of internet communication; every device needs an IP address to communicate
- **TCP**: Choose when reliability is critical (financial transactions, file transfers, email)
- **UDP Alternative**: Use UDP (User Datagram Protocol) when speed matters more than reliability (video streaming, online gaming, VoIP)
- **HTTP/HTTPS**: Standard for web applications; always use HTTPS in production for security
- **Protocol Selection**: Choose based on requirements:
  - Real-time, low-latency: UDP
  - Reliable delivery: TCP
  - Web communication: HTTP/HTTPS
  - IoT/lightweight: MQTT, CoAP
- **Performance**: IP routing efficiency depends on network topology; TCP overhead acceptable for most applications
- **Security**: IP addresses can be spoofed; TCP connections can be hijacked; always use HTTPS for sensitive data
- **Scalability**: IP routing scales globally; TCP connections consume server resources; use connection pooling and load balancing

---

## Distributed Systems

- **Definition**: Multiple independent computers working together to achieve a common goal, appearing as a single system to users
- **Key Challenges**:
  - **Network Failures**: Partial failures where some nodes fail but others continue
  - **Consistency**: Keeping data synchronized across nodes
  - **Coordination**: Managing state across multiple machines

### Vertical Scaling
- **Concept**: Adding more resources (CPU, RAM, disk) to a single machine
- **Advantages**: Simple, no code changes needed
- **Limitations**: Hardware limits, expensive, single point of failure

### Horizontal Scaling
- **Concept**: Adding more machines to distribute load
- **Advantages**: Theoretically unlimited scalability, fault tolerance
- **Challenges**: Complexity in coordination, data distribution, consistency

### Process Management
- **Load Balancing**: Distributing requests across multiple servers
- **Health Checks**: Monitoring server availability
- **Auto-scaling**: Dynamically adding/removing servers based on demand

### Fault Tolerance
- **Redundancy**: Multiple copies of data and services
- **Replication**: Keeping data synchronized across nodes
- **Failover**: Automatic switching to backup when primary fails
- **Recovery**: Restoring failed nodes without data loss

---

## Latency vs Throughput

|  ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.19.53 PM.png)   |   &  | ![image](../../Resources/17-system-architecture-basics/Screenshot%202026-02-08%20at%2011.21.08 PM.png) |
| :---: | :---: | :---: |


| Aspect | Latency | Throughput |
|--------|---------|-----------|
| **Definition** | Time taken for single request to complete | Number of requests processed per unit time |
| **Unit** | Milliseconds (ms) | Requests per second (RPS) |
| **Focus** | Speed of individual operation | Volume of operations |
| **Example** | API response takes 100ms | Server handles 1000 requests/second |
| **Trade-off** | Optimizing for low latency may reduce throughput | High throughput may increase latency |

### Latency Metrics: P90, P95, P99

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-10 at 5.56.49 PM.png' width='500' />

Percentile latencies (P90, P95, P99) measure the distribution of response times across all requests, providing insight into user experience and system performance.

#### Understanding Percentile Latencies

- **P90 (90th Percentile)**: 90% of requests complete within this time; 10% take longer
  - Example: P90 = 100ms means 90% of users experience response time ≤ 100ms
  - Use Case: Acceptable performance for most users
  - Typical Target: 50-200ms for web applications

- **P95 (95th Percentile)**: 95% of requests complete within this time; 5% take longer
  - Example: P95 = 200ms means 95% of users experience response time ≤ 200ms
  - Use Case: Identifies slower requests affecting significant user base
  - Typical Target: 100-500ms for web applications
  - Business Impact: 5% of users experiencing slow responses can affect satisfaction

- **P99 (99th Percentile)**: 99% of requests complete within this time; 1% take longer
  - Example: P99 = 500ms means 99% of users experience response time ≤ 500ms
  - Use Case: Identifies outliers and worst-case scenarios
  - Typical Target: 200-1000ms for web applications
  - Business Impact: Tail latency affects high-value users and can cause timeouts

#### Why Percentiles Matter More Than Averages

| Metric | Value | Interpretation |
|--------|-------|-----------------|
| **Average (Mean)** | 100ms | Misleading if distribution is skewed |
| **P50 (Median)** | 80ms | 50% of requests faster, 50% slower |
| **P90** | 150ms | 90% of users get good experience |
| **P95** | 250ms | 5% of users experience degradation |
| **P99** | 800ms | Tail latency; worst-case scenarios |
| **P99.9** | 2000ms | Extreme outliers; rare but impactful |

**Example Scenario**: E-commerce checkout
- Average latency: 100ms (looks good)
- P90: 120ms (most users happy)
- P95: 300ms (some users notice slowness)
- P99: 2000ms (1% of users experience 2-second delay, may abandon cart)

#### Latency SLOs (Service Level Objectives)

- **Aggressive SLO**: P99 < 100ms (high-frequency trading, real-time gaming)
- **Standard SLO**: P99 < 500ms (web applications, APIs)
- **Relaxed SLO**: P99 < 2000ms (batch processing, background jobs)

#### Monitoring and Optimization

- **Collect All Percentiles**: Track P50, P90, P95, P99, P99.9 to understand full distribution
- **Identify Tail Latency**: P99 spikes indicate specific bottlenecks (database queries, external API calls)
- **Optimize Strategically**:
  - Improve P90: Optimize common code paths
  - Improve P99: Add caching, optimize slow queries, implement timeouts
  - Reduce P99.9: Identify and fix outlier scenarios
- **Tools**: Use APM (Application Performance Monitoring) tools like Datadog, New Relic, or Prometheus to track percentiles
- **Alerting**: Set alerts on P95 and P99 thresholds to catch degradation early

#### Architect's Perspective: Latency Metrics

- **Don't rely on averages**: Always monitor percentiles to understand user experience
- **P99 is critical**: Focus on reducing tail latency; it affects user satisfaction disproportionately
- **Trade-offs**: Improving P99 often requires more resources; balance cost vs. user experience
- **Cascading failures**: High P99 in one service can cause timeouts in dependent services
- **Real-world impact**: A 500ms P99 latency spike during peak traffic can cause 1% of users to experience timeouts
- **Continuous monitoring**: Latency metrics should be tracked continuously; sudden spikes indicate problems

---

**Architect's Perspective**:
- **Low Latency**: Critical for real-time systems (trading, gaming, user experience)
- **High Throughput**: Important for batch processing, data pipelines
- **Balance**: Most systems need both - acceptable latency with high throughput

---

## CAP Theorem
<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-09 at 9.25.07 AM.png' width='500' />

- **Definition**: A fundamental principle stating that distributed systems can guarantee at most two of three properties simultaneously
- **The Three Properties**:
  - **Consistency (C)**: All nodes see the same data at the same time; every read receives the most recent write
  - **Availability (A)**: System remains operational and responsive; every request receives a response (success or failure)
  - **Partition Tolerance (P)**: System continues to operate despite network partitions (communication failures between nodes)

- **Key Insight**: In the presence of a network partition, you must choose between Consistency and Availability
  - **CA Systems**: Prioritize consistency and availability, but fail if network partition occurs
  - **CP Systems**: Prioritize consistency and partition tolerance, sacrificing availability (may return errors during partitions)
  - **AP Systems**: Prioritize availability and partition tolerance, sacrificing strong consistency (may return stale data)

### Example: E-Commerce Order System

**Scenario**: A distributed order processing system with multiple data centers

- **Consistency-focused (CP)**:
  - If network partition occurs between data centers, the system rejects new orders to ensure all centers have identical order data
  - Users experience unavailability but guaranteed data accuracy
  - Example: Banking systems often choose CP to prevent double-charging

- **Availability-focused (AP)**:
  - If network partition occurs, each data center continues accepting orders independently
  - Orders are eventually synchronized when partition heals
  - Users always get a response, but may see inconsistent inventory counts temporarily
  - Example: E-commerce platforms like Amazon choose AP for customer experience

- **Trade-off Decision**:
  - CP choice: Prevents overselling inventory but risks losing sales during outages
  - AP choice: Maximizes sales but requires eventual consistency mechanisms (reconciliation, conflict resolution)

**Architect's Perspective**:
- Most real-world systems operate in AP mode and use eventual consistency patterns
- Choose based on business requirements: financial systems lean CP, user-facing systems lean AP
- Network partitions are inevitable in distributed systems; plan for them rather than avoid them
- Use techniques like conflict-free replicated data types (CRDTs) or event sourcing to handle eventual consistency

---

## Proxy, Forward Proxy, and Reverse Proxy

### Table of Contents
1. [What is a Proxy?](#what-is-a-proxy)
2. [Forward Proxy](#forward-proxy)
3. [Reverse Proxy](#reverse-proxy-gateway)
4. [Comparison: Forward Proxy vs Reverse Proxy](#comparison-forward-proxy-vs-reverse-proxy)
5. [Architect's Perspective](#architects-perspective-proxy)


### What is a Proxy?

- **Definition**: An intermediary server that sits between clients and servers, forwarding requests and responses
- **Purpose**: Acts as a middleman to facilitate communication, add functionality, or control access
- **Position**: Intercepts network traffic and can modify, filter, or redirect it
- **Transparency**: Can be transparent (client unaware) or explicit (client configured)

---

### Forward Proxy

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-09 at 11.41.05 PM.png' width='500' />

#### Definition and Purpose

- **Definition**: A proxy server that sits between clients and the internet, forwarding client requests to external servers
- **Client Perspective**: Clients know about and configure the forward proxy
- **Server Perspective**: External servers see requests coming from the proxy, not the original client
- **Direction**: Client → Forward Proxy → External Server

#### How Forward Proxy Works

```
┌──────────────┐
│   Client 1   │
└──────┬───────┘
       │
┌──────┴───────┐
│   Client 2   │
└──────┬───────┘
       │
       ├─────────────────────────────┐
       │                             │
   ┌───▼──────────────┐              │
   │ Forward Proxy    │              │
   │ (Gateway)        │              │
   └───┬──────────────┘              │
       │                             │
       │ Forwards requests           │
       │ Modifies headers            │
       │ Caches responses            │
       │                             │
   ┌───▼──────────────────────────────┐
   │   External Servers / Internet    │
   │   (Google, Facebook, etc.)       │
   └──────────────────────────────────┘
```

#### Examples of Forward Proxy

- **Corporate Proxy**: Company employees access internet through corporate proxy
  - Monitors employee internet usage
  - Blocks access to certain websites
  - Logs all traffic for compliance
  
- **VPN (Virtual Private Network)**: Encrypts traffic and masks client IP
  - User connects to VPN proxy
  - All traffic routed through VPN server
  - External servers see VPN IP, not user's real IP
  
- **Residential Proxy**: Masks user identity for web scraping or testing
  - Routes requests through residential IP addresses
  - Appears as regular user traffic
  
- **ISP Proxy**: Internet Service Provider proxy for caching and filtering
  - Caches popular content locally
  - Reduces bandwidth usage
  - Filters malicious content

#### Advantages of Forward Proxy

- **Privacy**: Hides client's real IP address from external servers
- **Security**: Filters malicious content, blocks access to dangerous sites
- **Caching**: Stores frequently accessed content locally, reducing bandwidth
- **Access Control**: Restricts which external resources clients can access
- **Monitoring**: Logs and monitors all client internet activity
- **Bandwidth Optimization**: Compresses data, removes ads, reduces traffic
- **Anonymity**: Useful for accessing geo-restricted content

#### Disadvantages of Forward Proxy

- **Performance Overhead**: Adds latency to every request
- **Single Point of Failure**: If proxy goes down, clients lose internet access
- **Maintenance Burden**: Requires configuration and management on client side
- **Complexity**: Clients must be aware of and configure the proxy
- **Potential Bottleneck**: High traffic can overwhelm the proxy
- **Privacy Concerns**: Proxy operator can see all client traffic
- **Compatibility Issues**: Some applications may not work with forward proxies

---

### Reverse Proxy (Gateway)

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-09 at 11.43.02 PM.png' width='500' />

### Definition and Purpose

- **Definition**: A proxy server that sits between clients and backend servers, forwarding client requests to appropriate backend servers
- **Client Perspective**: Clients don't know about the reverse proxy; they think they're talking to a single server
- **Server Perspective**: Backend servers don't know about clients; they think requests come from the proxy
- **Direction**: Client → Reverse Proxy → Backend Servers
- **Alternative Name**: Often called a "Gateway" in API contexts

#### How Reverse Proxy Works

```
┌──────────────┐
│   Client 1   │
└──────┬───────┘
       │
┌──────┴───────┐
│   Client 2   │
└──────┬───────┘
       │
       │ Clients think they're
       │ talking to single server
       │
   ┌───▼──────────────────────────┐
   │  Reverse Proxy / Gateway     │
   │  - Load balancing            │
   │  - SSL/TLS termination       │
   │  - Caching                   │
   │  - Request routing           │
   │  - Rate limiting             │
   └───┬──────────────────────────┘
       │
       ├──────────────┬──────────────┬──────────────┐
       │              │              │              │
   ┌───▼──┐       ┌───▼──┐       ┌───▼──┐       ┌───▼──┐
   │ App  │       │ App  │       │ App  │       │ App  │
   │ Srv1 │       │ Srv2 │       │ Srv3 │       │ SrvN │
   └──────┘       └──────┘       └──────┘       └──────┘
```

#### Examples of Reverse Proxy

- **Load Balancer**: Distributes traffic across multiple backend servers
  - Nginx, HAProxy, AWS ELB/ALB
  - Routes requests based on algorithms (round-robin, least connections)
  - Performs health checks on backend servers
  
- **API Gateway**: Routes API requests to appropriate microservices
  - Kong, AWS API Gateway, Apigee
  - Handles authentication, rate limiting, request transformation
  - Routes `/users/*` to user service, `/orders/*` to order service
  
- **Web Server Reverse Proxy**: Caches static content and forwards dynamic requests
  - Nginx, Apache
  - Serves cached static files directly
  - Forwards dynamic requests to application servers
  
- **CDN (Content Delivery Network)**: Distributes content geographically
  - Cloudflare, Akamai, AWS CloudFront
  - Caches content at edge locations
  - Routes users to nearest server
  
- **SSL/TLS Termination Proxy**: Handles encryption/decryption
  - Offloads SSL/TLS processing from backend servers
  - Reduces computational load on application servers
  - Centralizes certificate management

#### Advantages of Reverse Proxy

- **Load Balancing**: Distributes traffic across multiple backend servers
- **High Availability**: If one backend fails, traffic routes to others
- **Scalability**: Add more backend servers without changing client configuration
- **Security**: Hides backend server details from clients
- **SSL/TLS Termination**: Offloads encryption from backend servers
- **Caching**: Stores frequently accessed content, reducing backend load
- **Request Routing**: Routes requests based on URL, hostname, headers
- **Rate Limiting**: Protects backend from excessive requests
- **Compression**: Compresses responses to reduce bandwidth
- **Transparent to Clients**: Clients unaware of backend complexity

#### Disadvantages of Reverse Proxy

- **Complexity**: More complex to set up and configure than forward proxy
- **Single Point of Failure**: If reverse proxy fails, all traffic is blocked
- **Performance Overhead**: Adds latency to every request
- **Maintenance**: Requires monitoring and management
- **Potential Bottleneck**: High traffic can overwhelm the proxy
- **Debugging Difficulty**: Harder to debug issues when proxy is in the middle
- **Cost**: Reverse proxy infrastructure adds operational cost

---

### Comparison: Forward Proxy vs Reverse Proxy

<img src='../../Resources/17-system-architecture-basics/Screenshot 2026-02-09 at 11.42.51 PM.png' width='500' />

| Aspect | Forward Proxy | Reverse Proxy |
|--------|---------------|---------------|
| **Position** | Between clients and internet | Between clients and backend servers |
| **Client Awareness** | Clients know about proxy | Clients unaware of proxy |
| **Server Awareness** | Servers don't know real client | Servers don't know real client |
| **Primary Purpose** | Client privacy, access control | Load balancing, security, caching |
| **Configuration** | Configured on client side | Configured on server side |
| **Use Case** | Corporate networks, VPNs | Web servers, APIs, CDNs |
| **Hides** | Client IP from servers | Backend servers from clients |
| **Typical Users** | Employees, individuals | Web services, applications |
| **Example** | Corporate proxy, VPN | Nginx, Load Balancer, API Gateway |
| **Traffic Direction** | Outbound (client → internet) | Inbound (internet → backend) |

---

#### Architect's Perspective: Proxy

- **Forward Proxy**: Use when you need to control client outbound traffic, provide privacy, or implement access policies
- **Reverse Proxy**: Use for load balancing, high availability, security, and scalability of backend services
- **Hybrid Approach**: Many systems use both forward and reverse proxies for comprehensive traffic management
- **Security**: Reverse proxies add a security layer by hiding backend infrastructure
- **Performance**: Both can improve performance through caching, but add latency overhead
- **Monitoring**: Implement comprehensive logging and monitoring for both proxy types
- **Redundancy**: Always implement redundant proxies to avoid single point of failure
- **API Gateway Pattern**: Modern microservices architectures typically use reverse proxies as API gateways

---

## Polling vs Streaming

### Table of Contents
1. [What is Polling](#what-is-polling)
2. [How Polling Works](#how-polling-works)
3. [Advantages of Polling](#advantages-of-polling)
4. [Disadvantages of Polling](#disadvantages-of-polling)
5. [Use Cases for Polling](#use-cases-for-polling)
6. [What is Streaming](#what-is-streaming)
7. [How Streaming Works](#how-streaming-works)
8. [Streaming Technologies](#streaming-technologies)
9. [Advantages of Streaming](#advantages-of-streaming)
10. [Disadvantages of Streaming](#disadvantages-of-streaming)
11. [Use Cases for Streaming](#use-cases-for-streaming)
12. [Comparison: Polling vs Streaming](#comparison-polling-vs-streaming)
13. [Architect's Perspective](#architects-perspective-polling-vs-streaming)


---


### What is Polling?

- **Definition**: Client repeatedly requests data from server at fixed intervals
- **Mechanism**: Client sends request → Server responds → Client waits → Client sends request again
- **Frequency**: Polling interval determines how often client checks for updates
- **Latency**: Update latency depends on polling interval (could be seconds to minutes)
- **Simplicity**: Simple to implement, works with standard HTTP

### How Polling Works

```
┌──────────────────────────────────────────────────────┐
│              POLLING MECHANISM                       │
└──────────────────────────────────────────────────────┘

Time →

Client                                    Server
  │                                         │
  ├─────── Request (Is there new data?) ───→│
  │                                         │
  │←────── Response (No new data) ──────────┤
  │                                         │
  │ Wait 5 seconds                          │
  │                                         │
  ├─────── Request (Is there new data?) ───→│
  │                                         │
  │←────── Response (No new data) ──────────┤
  │                                         │
  │ Wait 5 seconds                          │
  │                                         │
  ├─────── Request (Is there new data?) ───→│
  │                                         │
  │←────── Response (Yes! New data) ────────┤
  │                                         │
  │ Process data                            │
  │                                         │
```

### Advantages of Polling

- **Simple Implementation**: Easy to implement with standard HTTP
- **Stateless**: Server doesn't need to maintain client state
- **Firewall Friendly**: Works through firewalls and proxies
- **Browser Compatible**: Works in all browsers without special support
- **No Server Push**: Server doesn't need to initiate connections
- **Predictable**: Consistent resource usage patterns

### Disadvantages of Polling

- **High Latency**: Updates delayed by polling interval
- **Wasted Bandwidth**: Many requests return no new data
- **Server Load**: Continuous requests increase server load
- **Inefficient**: Polling even when no data changes
- **Battery Drain**: Mobile devices drain battery with frequent requests
- **Scalability Issues**: Doesn't scale well with many clients

### Use Cases for Polling

- **Low-Frequency Updates**: Data changes infrequently
- **Simple Applications**: When simplicity is more important than latency
- **Legacy Systems**: Systems that don't support WebSockets
- **Batch Operations**: Checking job status periodically
- **Monitoring**: Periodic health checks, status updates

---

### What is Streaming?

- **Definition**: Server pushes data to client as it becomes available
- **Mechanism**: Persistent connection established → Server sends data when available → Connection remains open
- **Latency**: Near real-time updates, minimal delay
- **Efficiency**: Only sends data when changes occur
- **Complexity**: Requires persistent connection management

### How Streaming Works

```
┌──────────────────────────────────────────────────────┐
│              STREAMING MECHANISM                     │
└──────────────────────────────────────────────────────┘

Time →

Client                                    Server
  │                                         │
  ├─────── Establish Connection ───────────→│
  │←────── Connection Established ──────────┤
  │                                         │
  │ (Connection remains open)               │
  │                                         │
  │←────── Data Update 1 ───────────────────┤
  │ Process data                            │
  │                                         │
  │←────── Data Update 2 ───────────────────┤
  │ Process data                            │
  │                                         │
  │←────── Data Update 3 ───────────────────┤
  │ Process data                            │
  │                                         │
  │ (Connection stays open, waiting)        │
  │                                         │
```

### Streaming Technologies

- **WebSocket**: Full-duplex communication over single TCP connection
  - Persistent connection
  - Low latency
  - Bidirectional communication
  - Example: Real-time chat, live notifications

- **Server-Sent Events (SSE)**: Server pushes updates to client
  - Unidirectional (server → client)
  - Automatic reconnection
  - Works over HTTP
  - Example: Live feeds, notifications

- **gRPC Streaming**: High-performance streaming using Protocol Buffers
  - Bidirectional streaming
  - HTTP/2 based
  - Language-agnostic
  - Example: Microservices communication

- **MQTT**: Lightweight publish-subscribe protocol
  - Publish-subscribe model
  - Low bandwidth
  - Persistent connections
  - Example: IoT devices, real-time sensors

### Advantages of Streaming

- **Low Latency**: Near real-time data delivery
- **Efficient**: Only sends data when changes occur
- **Reduced Bandwidth**: No wasted requests for unchanged data
- **Server Efficient**: Fewer total requests to server
- **Real-time Experience**: Immediate updates for users
- **Scalable**: Better scaling with many clients

### Disadvantages of Streaming

- **Complex Implementation**: Requires persistent connection management
- **Stateful**: Server must maintain client connections
- **Firewall Issues**: May have issues with firewalls/proxies
- **Resource Intensive**: Persistent connections consume server resources
- **Connection Management**: Handling disconnections and reconnections
- **Browser Support**: Requires modern browser features (WebSocket, SSE)

### Use Cases for Streaming

- **Real-Time Applications**: Live chat, notifications, feeds
- **Live Data**: Stock prices, sports scores, weather updates
- **Collaborative Tools**: Real-time document editing, whiteboards
- **Gaming**: Multiplayer games, real-time interactions
- **IoT**: Sensor data, device monitoring
- **Live Streaming**: Video/audio streaming

---

### Comparison: Polling vs Streaming

| Aspect | Polling | Streaming |
|--------|---------|-----------|
| **Latency** | High (depends on interval) | Low (near real-time) |
| **Bandwidth** | High (many empty requests) | Low (only data changes) |
| **Server Load** | High (many requests) | Medium (persistent connections) |
| **Implementation** | Simple | Complex |
| **Stateless** | Yes | No |
| **Firewall Friendly** | Yes | Depends on technology |
| **Browser Support** | All browsers | Modern browsers |
| **Scalability** | Poor | Good |
| **Connection Type** | Multiple short connections | Single persistent connection |
| **Best For** | Infrequent updates | Real-time updates |
| **Example** | Email checking, status updates | Live chat, notifications |

---

### Hybrid Approach: Long Polling

- **Definition**: Client polls server, but server holds response until data is available
- **How it works**:
  - Client sends request
  - Server waits for data (up to timeout)
  - When data available, server responds immediately
  - Client processes data and sends new request
- **Advantages**:
  - Lower latency than regular polling
  - Works with standard HTTP
  - Firewall friendly
- **Disadvantages**:
  - Still less efficient than true streaming
  - Server must manage pending requests
  - Timeout handling complexity

---

#### Architect's Perspective: Polling vs Streaming

- **Choose Polling When**:
  - Updates are infrequent
  - Simplicity is critical
  - Legacy system constraints
  - Low real-time requirements
  - Example: Checking email, periodic status checks

- **Choose Streaming When**:
  - Real-time updates required
  - High frequency of changes
  - Low latency critical
  - Modern technology stack available
  - Example: Live chat, notifications, collaborative tools

- **Choose Long Polling When**:
  - Need real-time feel with HTTP constraints
  - Firewall compatibility required
  - Moderate update frequency
  - Example: Browser-based notifications

- **Implementation Considerations**:
  - **Polling Interval**: Balance between latency and server load
  - **Connection Limits**: Streaming requires managing persistent connections
  - **Fallback Strategy**: Have fallback for streaming failures
  - **Monitoring**: Track connection health and data delivery
  - **Scalability**: Use load balancers and connection pooling for streaming

- **Technology Stack**:
  - Polling: Standard HTTP, REST APIs
  - Streaming: WebSocket, SSE, gRPC, MQTT
  - Hybrid: Long polling with HTTP

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
