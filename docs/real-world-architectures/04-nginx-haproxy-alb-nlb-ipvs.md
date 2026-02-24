# System Architecture: NGINX HAproxy ALB NLB IPVS

## Overview

Load balancing is critical for distributing traffic across multiple servers. This document covers five major load balancing solutions: NGINX, HAProxy, AWS Application Load Balancer (ALB), AWS Network Load Balancer (NLB), and IPVS (IP Virtual Server).

---

## Individual Solutions

### NGINX

**NGINX** is a high-performance, open-source web server and reverse proxy that excels at load balancing. It operates at Layer 7 (Application) and Layer 4 (Transport) and can handle millions of concurrent connections with minimal resource consumption.

**Key Characteristics:**
- Event-driven, asynchronous architecture
- Lightweight and fast
- Excellent for HTTP/HTTPS load balancing
- Can be deployed on-premises or in cloud
- Rich ecosystem with modules and extensions
- Configuration-based (no coding required)

---

### HAProxy

**HAProxy** (High Availability Proxy) is a free, open-source load balancer and proxy server designed for high availability and performance. It operates at Layer 4 and Layer 7, supporting both TCP and HTTP protocols.

**Key Characteristics:**
- Extremely fast and reliable
- Advanced health checking capabilities
- Excellent for TCP load balancing
- Strong support for session persistence
- Detailed statistics and monitoring
- Can handle millions of connections efficiently

---

### AWS Application Load Balancer (ALB)

**ALB** is a managed AWS service that operates at Layer 7 (Application Layer). It's designed for modern application architectures, particularly microservices and containerized applications.

**Key Characteristics:**
- Fully managed by AWS (no infrastructure management)
- Layer 7 routing (path-based, hostname-based, header-based)
- Native integration with AWS services (ECS, Lambda, EC2)
- Auto-scaling support
- Built-in DDoS protection
- Pay-per-use pricing model

---

### AWS Network Load Balancer (NLB)

**NLB** is a managed AWS service operating at Layer 4 (Transport Layer). It's optimized for extreme performance and low latency, handling millions of requests per second.

**Key Characteristics:**
- Ultra-high performance (millions of RPS)
- Extreme low latency (<100 microseconds)
- Layer 4 TCP/UDP load balancing
- Connection pooling and multiplexing
- Ideal for non-HTTP protocols (gaming, IoT, real-time)
- Fully managed by AWS

---

### IPVS (IP Virtual Server)

**IPVS** is a Linux kernel-based load balancing solution that operates at Layer 4. It's part of the Linux Virtual Server (LVS) project and provides in-kernel load balancing with minimal overhead.

Check this https://aws.plainenglish.io/awss-0-01-hour-secret-we-replaced-elb-with-this-1990s-tech-and-handled-10x-more-traffic-d772bfe5d352

**Key Characteristics:**
- Kernel-level implementation (extremely fast)
- Minimal CPU overhead
- Supports multiple load balancing algorithms
- Direct Server Return (DSR) capability
- No application-level processing
- Requires Linux kernel support

---

## Comparison Table

| Aspect | NGINX | HAProxy | ALB | NLB | IPVS |
|--------|-------|---------|-----|-----|------|
| **Layer** | L4/L7 | L4/L7 | L7 | L4 | L4 |
| **Deployment** | Self-hosted | Self-hosted | Managed (AWS) | Managed (AWS) | Self-hosted (Linux) |
| **Performance** | Very High | Very High | High | Ultra-High | Ultra-High |
| **Throughput** | 100K+ RPS | 100K+ RPS | 100K+ RPS | 1M+ RPS | 1M+ RPS |
| **Latency** | Low (ms) | Low (ms) | Medium (ms) | Ultra-Low (<100µs) | Ultra-Low (<100µs) |
| **Setup Complexity** | Medium | Medium | Low | Low | High |
| **Operational Overhead** | High | High | Low | Low | High |
| **Cost** | Free (OSS) | Free (OSS) | $16-32/month + data | $16-32/month + data | Free (OSS) |
| **Scaling** | Manual | Manual | Auto-scaling | Auto-scaling | Manual |
| **Health Checks** | Basic | Advanced | Advanced | Advanced | Basic |
| **Session Persistence** | Good | Excellent | Good | Good | Good |
| **Protocol Support** | HTTP/HTTPS/TCP | HTTP/HTTPS/TCP/UDP | HTTP/HTTPS | TCP/UDP | TCP/UDP |
| **Routing Flexibility** | Good | Excellent | Excellent | Limited | Limited |
| **AWS Integration** | None | None | Native | Native | None |
| **Monitoring** | Basic | Detailed | CloudWatch | CloudWatch | Basic |
| **SSL/TLS Termination** | Yes | Yes | Yes | Yes | No |
| **Learning Curve** | Medium | Medium | Low | Low | High |

---

## Detailed Analysis

### Differences

**Protocol Support:**
- NGINX & HAProxy: Flexible, support multiple protocols
- ALB: HTTP/HTTPS only
- NLB: TCP/UDP, any protocol
- IPVS: TCP/UDP only

**Management:**
- NGINX & HAProxy: Require manual configuration and updates
- ALB & NLB: Fully managed, automatic updates
- IPVS: Kernel-level, requires system administration

**Routing Capabilities:**
- ALB: Advanced L7 routing (path, hostname, headers)
- HAProxy: Advanced L7 routing with custom rules
- NGINX: Good L7 routing with modules
- NLB & IPVS: Basic L4 routing only

### Benefits

**NGINX:**
- Lightweight and resource-efficient
- Excellent for web applications
- Large community and ecosystem
- Easy configuration syntax
- Can run on minimal hardware

**HAProxy:**
- Superior health checking
- Excellent for high-availability setups
- Detailed statistics and monitoring
- Strong TCP load balancing
- Proven in production at scale

**ALB:**
- Zero operational overhead
- Automatic scaling
- Native AWS integration
- Advanced routing for microservices
- Built-in security features

**NLB:**
- Extreme performance for non-HTTP workloads
- Ultra-low latency
- Handles millions of connections
- Ideal for real-time applications
- Automatic scaling

**IPVS:**
- Kernel-level performance
- Minimal resource consumption
- No application overhead
- Direct Server Return (DSR) for efficiency
- Extremely fast packet processing

### Trade-offs

**NGINX:**
- ✓ Lightweight | ✗ Requires manual management
- ✓ Flexible | ✗ Less advanced health checks
- ✓ Free | ✗ Operational overhead

**HAProxy:**
- ✓ Advanced features | ✗ Steeper learning curve
- ✓ Excellent monitoring | ✗ Manual scaling
- ✓ Free | ✗ Requires expertise

**ALB:**
- ✓ Zero ops | ✗ AWS lock-in
- ✓ Auto-scaling | ✗ L7 only (not suitable for all protocols)
- ✓ Easy setup | ✗ Higher cost at scale

**NLB:**
- ✓ Ultra-performance | ✗ AWS lock-in
- ✓ Low latency | ✗ Limited routing options
- ✓ Auto-scaling | ✗ Higher cost for high throughput

**IPVS:**
- ✓ Kernel performance | ✗ Complex setup
- ✓ Minimal overhead | ✗ Limited routing
- ✓ Free | ✗ Requires Linux expertise

### Cost Analysis

**NGINX & HAProxy:**
- **Infrastructure:** Server costs (EC2, VPS, bare metal)
- **Licensing:** Free (open-source)
- **Operational:** Engineering time for management
- **Typical:** $50-500/month depending on infrastructure

**ALB:**
- **Base:** $16.20/month per ALB
- **Data Processing:** $0.006 per LCU-hour
- **Typical:** $50-200/month for moderate traffic

**NLB:**
- **Base:** $16.20/month per NLB
- **Data Processing:** $0.006 per LCU-hour
- **Typical:** $100-500/month for high throughput

**IPVS:**
- **Infrastructure:** Server costs (bare metal or VM)
- **Licensing:** Free (open-source)
- **Operational:** Engineering time
- **Typical:** $100-1000/month depending on infrastructure

---

## Where to Use Each

### NGINX
- **Best for:** Web applications, microservices, API gateways
- **Scenarios:** 
  - HTTP/HTTPS load balancing
  - Reverse proxy for web services
  - Content delivery
  - When you need flexibility and control
  - Cost-sensitive deployments

### HAProxy
- **Best for:** High-availability clusters, TCP load balancing
- **Scenarios:**
  - Database load balancing (MySQL, PostgreSQL)
  - TCP protocol load balancing
  - Complex health checking requirements
  - When you need detailed statistics
  - On-premises deployments

### ALB
- **Best for:** AWS-native microservices and containerized apps
- **Scenarios:**
  - ECS/Kubernetes clusters on AWS
  - Microservices with path-based routing
  - Lambda integration
  - When you want zero operational overhead
  - Modern application architectures

### NLB
- **Best for:** Extreme performance and low-latency requirements
- **Scenarios:**
  - Gaming servers
  - Real-time applications
  - IoT platforms
  - Non-HTTP protocols
  - When latency is critical
  - Millions of concurrent connections

### IPVS
- **Best for:** On-premises high-performance clusters
- **Scenarios:**
  - Kubernetes clusters (kube-proxy uses IPVS)
  - Bare-metal deployments
  - When kernel-level performance is needed
  - Direct Server Return (DSR) architectures
  - Extreme scale on-premises

---

## Software Architect Perspective: Scenario Breakdown

### Scenario 1: Early-Stage SaaS (100-1K users)
**Use:** NGINX
- **Why:** Low cost, easy to manage, sufficient performance
- **Setup:** Single NGINX instance with failover
- **Cost:** Minimal infrastructure

### Scenario 2: Growing Web Application (1K-100K users)
**Use:** NGINX or ALB
- **Why:** NGINX for cost control; ALB for AWS convenience
- **Setup:** NGINX cluster or ALB with auto-scaling
- **Decision:** Choose ALB if already on AWS, NGINX if multi-cloud

### Scenario 3: High-Availability Database Cluster
**Use:** HAProxy
- **Why:** Superior TCP load balancing, advanced health checks
- **Setup:** HAProxy pair with keepalived for HA
- **Cost:** Moderate infrastructure

### Scenario 4: Microservices on AWS (ECS/EKS)
**Use:** ALB
- **Why:** Native integration, path-based routing, auto-scaling
- **Setup:** ALB → ECS/EKS services
- **Cost:** Managed, predictable

### Scenario 5: Real-Time Gaming/IoT Platform
**Use:** NLB
- **Why:** Ultra-low latency, millions of concurrent connections
- **Setup:** NLB → backend servers
- **Cost:** Higher but justified by performance

### Scenario 6: On-Premises Kubernetes Cluster
**Use:** IPVS (via kube-proxy)
- **Why:** Kernel-level performance, no additional overhead
- **Setup:** Kubernetes with IPVS mode enabled
- **Cost:** Free, minimal overhead

### Scenario 7: Multi-Region Global Application
**Use:** ALB/NLB + Route 53 (AWS) or NGINX + DNS
- **Why:** Geographic distribution with local load balancing
- **Setup:** Regional load balancers + global routing
- **Cost:** Scales with regions

### Scenario 8: Legacy On-Premises Infrastructure
**Use:** HAProxy or NGINX
- **Why:** No cloud dependency, proven reliability
- **Setup:** Redundant pair with shared storage
- **Cost:** Infrastructure-dependent

---

## Decision Matrix

```
Need extreme latency (<100µs)?
├─ Yes → NLB or IPVS
└─ No → Continue

On AWS?
├─ Yes → ALB (L7) or NLB (L4)
└─ No → Continue

Need L7 routing (paths, hostnames)?
├─ Yes → NGINX or HAProxy
└─ No → HAProxy or IPVS

Need advanced health checks?
├─ Yes → HAProxy
└─ No → NGINX, ALB, or NLB

Cost-sensitive?
├─ Yes → NGINX or IPVS
└─ No → ALB or NLB (managed convenience)
```

---

## Summary

| Use Case | Recommended | Rationale |
|----------|-------------|-----------|
| Web apps, APIs | NGINX | Lightweight, flexible, free |
| Database LB, TCP | HAProxy | Advanced features, proven |
| AWS microservices | ALB | Native integration, auto-scaling |
| Real-time, gaming | NLB | Ultra-low latency, high throughput |
| Kubernetes on-prem | IPVS | Kernel performance, zero overhead |
| Cost-critical | NGINX/IPVS | Free, minimal infrastructure |
| Zero-ops preference | ALB/NLB | Fully managed by AWS |
| Complex routing | HAProxy/NGINX | Advanced L7 capabilities |
