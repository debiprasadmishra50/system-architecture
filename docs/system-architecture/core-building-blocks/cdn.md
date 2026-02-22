# CDN (Content Delivery Network)

## Table of Contents
1. [What is a CDN](#what-is-a-cdn)
2. [Problems It Solves](#problems-it-solves)
3. [Why CDN is Important](#why-cdn-is-important)
   - [Fast Content Delivery](#fast-content-delivery)
   - [Reduced Server Load](#reduced-server-load)
   - [DDoS Protection](#ddos-protection)
4. [How CDN Works](#how-cdn-works)
   - [Step-by-Step Process](#step-by-step-process)
   - [YouTube Example](#youtube-example)
5. [CDN Server Hierarchy](#cdn-server-hierarchy)
   - [Origin Servers](#origin-servers)
   - [Regional Servers](#regional-servers)
   - [Edge Servers](#edge-servers)
6. [CDN Providers Comparison](#cdn-providers-comparison)
7. [Diagram: CDN Architecture](#diagram-cdn-architecture)
8. [Diagram: Content Delivery Flow](#diagram-content-delivery-flow)

---

## What is a CDN

- **Definition**: A geographically distributed network of servers that work together to deliver content to users with high availability and high performance
- **Core Function**: Caches and serves content from servers located closer to end users, reducing latency and bandwidth usage
- **Content Types**: Static assets (images, CSS, JavaScript), videos, documents, APIs, and dynamic content
- **Transparency**: Users access content through optimized routes; origin servers remain protected and offloaded

---

## Problems It Solves

### 1. **High Latency**
- Eliminates long-distance data transmission delays
- Serves content from geographically proximate servers
- Reduces round-trip time (RTT) for users worldwide

### 2. **Bandwidth Congestion**
- Distributes traffic across multiple servers globally
- Reduces strain on origin server bandwidth
- Prevents bottlenecks during traffic spikes

### 3. **Server Overload**
- Offloads traffic from origin servers
- Caches frequently accessed content at edge locations
- Allows origin servers to focus on dynamic content generation

### 4. **Poor User Experience**
- Delivers content faster to users in different regions
- Reduces page load times and buffering
- Improves application responsiveness

### 5. **Single Point of Failure**
- Provides redundancy through distributed servers
- Ensures service availability even if origin server fails
- Automatic failover to healthy edge servers

---

## Why CDN is Important

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.12.28 AM.png' width=700 />

### Fast Content Delivery
- **Geographic Distribution**: Servers positioned worldwide reduce distance between users and content
- **Caching Strategy**: Frequently accessed content cached at edge servers eliminates origin server requests
- **Performance Metrics**: Typical latency reduction of 50-80% compared to direct origin access
- **User Experience**: Faster load times increase engagement and reduce bounce rates

### Reduced Server Load
- **Traffic Offloading**: CDN absorbs majority of traffic, protecting origin servers
- **Bandwidth Savings**: Origin server bandwidth usage reduced by 60-90%
- **Cost Efficiency**: Fewer origin servers needed to handle same traffic volume
- **Scalability**: Origin servers can handle more concurrent users without degradation

### DDoS Protection
- **Traffic Filtering**: CDN absorbs and filters malicious traffic before reaching origin
- **Distributed Defense**: Attack traffic distributed across CDN infrastructure
- **Rate Limiting**: Automatic detection and blocking of suspicious traffic patterns
- **Mitigation**: Protects origin server from being overwhelmed during attacks

---

## How CDN Works

### Step-by-Step Process

1. **User Request**: User requests content (e.g., video, image) from application
2. **DNS Resolution**: DNS query resolves to nearest CDN edge server
3. **Edge Server Check**: Edge server checks if content exists in cache
4. **Cache Hit**: If cached, content served directly to user (fast path)
5. **Cache Miss**: If not cached, edge server requests from parent/regional server
6. **Origin Fetch**: If not in regional cache, request goes to origin server
7. **Content Delivery**: Content travels back through hierarchy to edge server
8. **Caching**: Content cached at each level for future requests
9. **User Delivery**: Content delivered to user from nearest cached location
10. **Subsequent Requests**: Future requests served from cache (significantly faster)

### YouTube Example

```
User in India requests video from YouTube
    ↓
DNS resolves to nearest CDN edge server (Mumbai)
    ↓
Edge server checks cache for video
    ↓
Cache Miss → Request goes to regional server (Asia-Pacific)
    ↓
Regional server checks cache
    ↓
Cache Miss → Request goes to origin server (YouTube HQ)
    ↓
Origin server streams video content
    ↓
Content cached at regional server (Asia-Pacific)
    ↓
Content cached at edge server (Mumbai)
    ↓
Video delivered to user in India (fast)
    ↓
Next user in India requests same video
    ↓
Served from Mumbai edge server cache (very fast, ~10-50ms latency)
```

**Benefits in this scenario:**
- First user: ~200-300ms latency (origin fetch)
- Subsequent users: ~10-50ms latency (edge cache)
- Origin server bandwidth: Reduced by 95%+
- Video quality: Adaptive bitrate based on user's connection

---

## CDN Server Hierarchy

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.16.14 AM.png' width=600 />
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.16.25 AM.png' width=600 />

### Origin Servers
- **Location**: Company's primary data center
- **Role**: Source of truth for all content
- **Responsibility**: Generate dynamic content, store original files
- **Load**: Minimal traffic (only cache misses and updates)
- **Example**: YouTube's main servers in California

### Regional Servers
- **Location**: Major geographic regions (continents)
- **Role**: Intermediate cache layer between origin and edge
- **Responsibility**: Cache popular content, reduce origin load
- **Coverage**: Serve multiple countries in region
- **Example**: CDN servers in Singapore serving Asia-Pacific region

### Edge Servers
- **Location**: Distributed globally, close to end users
- **Role**: First point of contact for user requests
- **Responsibility**: Serve cached content, handle user requests
- **Coverage**: City-level or ISP-level distribution
- **Example**: CDN servers in Mumbai, Delhi, Bangalore serving India

**Hierarchy Flow:**
```
User → Edge Server (Mumbai) → Regional Server (Singapore) → Origin Server (California)
```

---

## CDN Providers Comparison

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-14 at 11.17.08 AM.png' width=900 />

| Provider | Pros | Cons |
|----------|------|------|
| **Cloudflare** | • Global network (200+ cities)<br>• DDoS protection included<br>• Affordable pricing<br>• Easy setup<br>• Free tier available | • Limited customization<br>• Performance varies by region<br>• Smaller network than competitors |
| **Akamai** | • Largest global network<br>• Excellent performance<br>• Advanced security features<br>• Enterprise support | • Expensive pricing<br>• Complex configuration<br>• Steep learning curve<br>• Overkill for small projects |
| **AWS CloudFront** | • Integrated with AWS ecosystem<br>• Competitive pricing<br>• Excellent documentation<br>• Flexible caching rules | • Vendor lock-in<br>• Complex pricing model<br>• Requires AWS account<br>• Less intuitive UI |
| **Fastly** | • High performance<br>• Real-time purging<br>• Instant configuration updates<br>• Good for video streaming | • Higher cost than Cloudflare<br>• Smaller network than Akamai<br>• Less enterprise features<br>• Smaller community |
| **Google Cloud CDN** | • Integrated with Google Cloud<br>• Good performance<br>• Competitive pricing<br>• Strong video support | • Requires Google Cloud account<br>• Vendor lock-in<br>• Smaller network than Akamai<br>• Less mature than competitors |
| **Bunny CDN** | • Very affordable<br>• Good performance<br>• Transparent pricing<br>• Good for video streaming | • Smaller network<br>• Limited advanced features<br>• Smaller support community<br>• Less enterprise-focused |

---

## Diagram: CDN Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         GLOBAL CDN NETWORK                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              ORIGIN SERVER (California)                  │   │
│  │         (Source of Truth, Dynamic Content)               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            ↑                                    │
│                    (Cache Misses)                               │
│                            │                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           REGIONAL SERVERS (Intermediate Cache)          │   │
│  │  ┌─────────────────┐  ┌─────────────────┐                │   │
│  │  │ Europe Region   │  │ Asia Region     │                │   │
│  │  │ (Frankfurt)     │  │ (Singapore)     │                │   │
│  │  └─────────────────┘  └─────────────────┘                │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            ↑                                    │
│                    (Regional Misses)                            │
│                            │                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         EDGE SERVERS (Closest to Users)                  │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │   │
│  │  │ Mumbai   │ │ London   │ │ Sydney   │ │ Toronto  │     │   │
│  │  │ (India)  │ │ (Europe) │ │(Australia)│ │(Canada) │     │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            ↑                                    │
│                    (User Requests)                              │
│                            │                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    END USERS                             │   │
│  │  👤 India  👤 UK  👤 Australia  👤 Canada                │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Diagram: Content Delivery Flow

```
SCENARIO: User in India requests YouTube video

┌───────────────────────────────────────────────┐
│ FIRST REQUEST (Cache Miss)                    │
├───────────────────────────────────────────────┤
│                                               │
│  User (India)                                 │
│      │                                        │
│      │ 1. Request video                       │
│      ↓                                        │
│  Edge Server (Mumbai)                         │
│      │                                        │
│      │ 2. Cache miss                          │
│      ↓                                        │
│  Regional Server (Singapore)                  │
│      │                                        │
│      │ 3. Cache miss                          │
│      ↓                                        │
│  Origin Server (California)                   │
│      │                                        │
│      │ 4. Fetch video                         │
│      ↓                                        │
│  Regional Server (Singapore) - Cache video    │
│      │                                        │
│      │ 5. Send to edge                        │
│      ↓                                        │
│  Edge Server (Mumbai) - Cache video           │
│      │                                        │
│      │ 6. Deliver to user                     │
│      ↓                                        │
│  User (India) - Receives video (~200-300ms)   │
│                                               │
└───────────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│ SUBSEQUENT REQUESTS (Cache Hit)           │
├───────────────────────────────────────────┤
│                                           │
│  User (India)                             │
│      │                                    │
│      │ 1. Request same video              │
│      ↓                                    │
│  Edge Server (Mumbai) - Cache hit!        │
│      │                                    │
│      │ 2. Serve from cache                │
│      ↓                                    │
│  User (India) - Receives video (~10-50ms) │
│                                           │
│  ✓ 95%+ faster than first request         │
│  ✓ Origin server not contacted            │
│  ✓ Minimal bandwidth usage                │
│                                           │
└───────────────────────────────────────────┘
```

---

## Key Considerations

- **Cache Invalidation**: Plan strategy for updating cached content (TTL, purging, versioning)
- **Cost Optimization**: Monitor bandwidth usage and choose appropriate CDN tier
- **Geographic Coverage**: Ensure CDN has servers in regions where users are located
- **Security**: Enable DDoS protection, WAF, and SSL/TLS encryption
- **Performance Monitoring**: Track cache hit ratio, latency, and user experience metrics
- **Fallback Strategy**: Configure origin server failover if CDN becomes unavailable
- **Content Types**: Different content may require different caching strategies
- **Compliance**: Ensure CDN provider complies with data residency and privacy regulations
