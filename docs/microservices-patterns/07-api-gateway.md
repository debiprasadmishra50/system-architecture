# Event Driven Systems: Api Gateway

## Table of Contents
1. [What is API Gateway](#what-is-api-gateway)
2. [Problem Definition and Solution](#problem-definition-and-solution)
3. [Limitations of Alternative Patterns](#limitations-of-alternative-patterns)
4. [How API Gateway Solves It](#how-api-gateway-solves-it)
5. [Design and Implementation](#design-and-implementation)
6. [Existing Products](#existing-products)
7. [Custom vs Existing API Gateway](#custom-vs-existing-api-gateway)
8. [GraphQL as API Gateway](#graphql-as-api-gateway)

---

## What is API Gateway

An **API Gateway** is a server that acts as an intermediary between clients and backend microservices. It provides a single entry point for all client requests and handles the complexity of routing, transforming, and managing communication with multiple backend services.

The API Gateway pattern is a critical architectural component in microservices-based systems that abstracts the complexity of service-to-service communication from the client perspective.

![Gateway](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.19.00 PM.png)

---

## Problem Definition and Solution

### The Problem: E-Commerce Application Example

![Without](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.09.35 PM.png)
Consider a modern e-commerce application with the following microservices architecture:

- **Product Service**: Manages product catalog, inventory, and details
- **User Service**: Handles user authentication, profiles, and preferences
- **Order Service**: Manages order creation, tracking, and history
- **Payment Service**: Processes payments and transactions
- **Notification Service**: Sends emails, SMS, and push notifications
- **Review Service**: Manages product reviews and ratings

#### Client-Side Challenges Without API Gateway:
![Without](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.14.45 PM.png)

1. **Multiple Service Discovery**: Clients must know the addresses of all backend services
   ```
   Product Service: api.products.com:8001
   User Service: api.users.com:8002
   Order Service: api.orders.com:8003
   Payment Service: api.payments.com:8004
   ```

2. **Complex Request Orchestration**: A single user action requires multiple service calls
   ```
   Example: User wants to view their order details
   - Call User Service to verify authentication
   - Call Order Service to get order information
   - Call Product Service to get product details
   - Call Review Service to get product reviews
   - Aggregate all responses
   ```

3. **Protocol Mismatch**: Different services might use different protocols (REST, gRPC, WebSocket)

4. **Cross-Cutting Concerns**: Each client must implement:
   - Authentication and authorization
   - Rate limiting
   - Request/response transformation
   - Logging and monitoring
   - Error handling and retries

5. **Latency Issues**: Multiple round trips to different services increase overall response time

6. **Security Vulnerabilities**: Exposing all service endpoints directly to clients increases attack surface

### The Solution: API Gateway Pattern

An API Gateway consolidates all these concerns into a single, centralized component that sits between clients and microservices.

---

## Limitations of Alternative Patterns

### 1. Backend for Frontend (BFF) Pattern
![BFF](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.17.29 PM.png)

**What it does**: Creates service-specific API layers tailored for different client types (web, mobile, desktop).

**Limitations**:
- Still requires clients to know about multiple BFF endpoints
- Doesn't solve cross-cutting concerns like authentication and rate limiting
- Each BFF must implement its own security, logging, and monitoring
- Doesn't handle service discovery or dynamic routing
- Increases operational complexity with multiple BFF instances to maintain

**Example**: 
```
Web Client → Web BFF → Multiple Services
Mobile Client → Mobile BFF → Multiple Services
```
Each BFF still needs to handle routing, transformation, and orchestration independently.

### 2. Proxy Pattern
![Proxy](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.17.39 PM.png)

**What it does**: Acts as a simple pass-through intermediary that forwards requests to backend services.

**Limitations**:
- Only handles basic request forwarding
- No intelligent routing or load balancing
- Cannot aggregate responses from multiple services
- Doesn't provide request/response transformation
- Limited support for cross-cutting concerns
- No built-in authentication or authorization

**Example**:
```
Client → Proxy → Service
```
The proxy simply forwards the request without understanding or modifying it.

### 3. Composite Pattern
![Composite](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.17.50 PM.png)

**What it does**: Combines multiple service responses into a single composite response.

**Limitations**:
- Requires clients to know which services to compose
- No centralized control over composition logic
- Difficult to maintain consistency across different client implementations
- Doesn't handle service discovery or routing
- Each client must implement composition logic independently
- No unified security or rate limiting

**Example**:
```
Client composes:
- Product details from Product Service
- Reviews from Review Service
- Inventory from Inventory Service
```

### 4. Aggregator Pattern
![Aggregator](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.17.57 PM.png)

**What it does**: Aggregates responses from multiple services into a single response.

**Limitations**:
- Requires a dedicated aggregator service for each use case
- Doesn't provide a unified entry point for all clients
- Clients still need to know about multiple aggregators
- Difficult to manage cross-cutting concerns across multiple aggregators
- Increases operational complexity with multiple aggregator instances
- No centralized authentication or authorization

**Example**:
```
Client → Order Aggregator → Product Service, User Service, Order Service
Client → Product Aggregator → Product Service, Review Service, Inventory Service
```

---

## How API Gateway Solves It

The API Gateway pattern provides a **unified, centralized solution** that addresses all the limitations above:

### Complete Solution for E-Commerce Example:

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                              │
│  (Web, Mobile, Desktop, Third-party Integrations)           │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │      API Gateway               │
        │  ┌──────────────────────────┐  │
        │  │ Authentication/AuthZ     │  │
        │  │ Rate Limiting            │  │
        │  │ Request Transformation   │  │
        │  │ Response Aggregation     │  │
        │  │ Service Discovery        │  │
        │  │ Load Balancing           │  │
        │  │ Logging & Monitoring     │  │
        │  │ Caching                  │  │
        │  │ Circuit Breaking         │  │
        │  └──────────────────────────┘  │
        └────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    ┌────────┐      ┌────────┐      ┌────────┐
    │Product │      │ Order  │      │Payment │
    │Service │      │Service │      │Service │
    └────────┘      └────────┘      └────────┘
```

### Key Advantages:

1. **Single Entry Point**: Clients only need to know one endpoint
   ```
   All requests → https://api.ecommerce.com/
   ```

2. **Unified Authentication**: All requests authenticated at gateway level
   ```
   API Gateway validates JWT token once
   All downstream services trust the gateway
   ```

3. **Request Routing**: Intelligent routing based on URL patterns
   ```
   /api/products/* → Product Service
   /api/orders/* → Order Service
   /api/payments/* → Payment Service
   ```

4. **Response Aggregation**: Combine multiple service responses
   ```
   GET /api/orders/123/details
   
   API Gateway:
   1. Calls Order Service for order info
   2. Calls Product Service for product details
   3. Calls Review Service for reviews
   4. Aggregates and returns single response
   ```

5. **Cross-Cutting Concerns**: Centralized implementation
   ```
   - Rate limiting: 1000 requests/hour per user
   - Logging: All requests logged in one place
   - Monitoring: Single dashboard for all traffic
   - Caching: Reduce backend load
   ```

6. **Protocol Translation**: Handle different backend protocols
   ```
   Client (REST) → API Gateway → Product Service (gRPC)
   Client (REST) → API Gateway → Notification Service (WebSocket)
   ```

7. **Service Discovery**: Automatic service location management
   ```
   API Gateway maintains service registry
   Automatically routes to healthy instances
   Handles service failures gracefully
   ```

---

## Design and Implementation

![Design](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.20.45 PM.png)

### Features API Gateway Will Have

#### 1. **Request Routing**
- URL pattern matching and routing to appropriate services
- Path-based routing: `/api/products/*` → Product Service
- Host-based routing: `products.api.com` → Product Service
- Method-based routing: `POST /api/orders` → Order Service

#### 2. **Authentication & Authorization**
- JWT token validation
- OAuth 2.0 support
- API key management
- Role-based access control (RBAC)
- Scope-based authorization

#### 3. **Rate Limiting & Throttling**
- Per-user rate limits
- Per-IP rate limits
- Sliding window algorithm
- Token bucket algorithm
- Quota management

#### 4. **Request/Response Transformation**
- Header manipulation (add/remove/modify)
- Body transformation (JSON to XML, etc.)
- Request validation against schemas
- Response formatting
- Compression (gzip, brotli)

#### 5. **Service Discovery**
- Dynamic service registration
- Health checking
- Load balancing (round-robin, least connections, weighted)
- Circuit breaker integration
- Automatic failover

#### 6. **Caching**
- Response caching based on HTTP headers
- Cache invalidation strategies
- Cache key generation
- TTL management
- Cache warming

#### 7. **Logging & Monitoring**
- Request/response logging
- Performance metrics (latency, throughput)
- Error tracking and alerting
- Distributed tracing (OpenTelemetry)
- Access logs

#### 8. **API Versioning**
- URL-based versioning: `/api/v1/products`
- Header-based versioning: `Accept: application/vnd.api+json;version=1`
- Query parameter versioning: `?version=1`
- Deprecation warnings

#### 9. **Response Aggregation**
- Parallel service calls
- Sequential service calls with dependencies
- Response merging and transformation
- Partial failure handling

#### 10. **Security Features**
- CORS (Cross-Origin Resource Sharing) handling
- HTTPS/TLS enforcement
- DDoS protection
- SQL injection prevention
- XSS protection
- Request size limits

### Deployment Strategy

#### 1. **Deployment Architecture**

```
┌─────────────────────────────────────────────────────────┐
│                    Load Balancer                        │
│              (AWS ELB, Nginx, HAProxy)                  │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    ┌────────┐  ┌────────┐  ┌────────┐
    │API GW  │  │API GW  │  │API GW  │
    │Instance│  │Instance│  │Instance│
    │   1    │  │   2    │  │   3    │
    └────────┘  └────────┘  └────────┘
        │            │            │
        └────────────┼────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    ┌────────┐  ┌────────┐  ┌────────┐
    │Product │  │ Order  │  │Payment │
    │Service │  │Service │  │Service │
    └────────┘  └────────┘  └────────┘
```

#### 2. **Scaling Strategy**

- **Horizontal Scaling**: Deploy multiple API Gateway instances behind a load balancer
- **Auto-scaling**: Scale based on CPU, memory, or request count
- **Regional Deployment**: Deploy in multiple regions for global availability
- **Edge Deployment**: Use CDN edge locations for reduced latency

#### 3. **High Availability**

- **Multi-zone deployment**: Distribute across availability zones
- **Health checks**: Regular health checks with automatic failover
- **Session persistence**: Sticky sessions or distributed session storage
- **Graceful shutdown**: Drain connections before shutdown

#### 4. **Configuration Management**

- **Centralized configuration**: Store routing rules, rate limits in central repository
- **Dynamic updates**: Update configuration without restarting
- **Version control**: Track configuration changes
- **Environment-specific configs**: Different configs for dev, staging, production

#### 5. **Monitoring & Observability**

- **Metrics collection**: Prometheus, CloudWatch, Datadog
- **Log aggregation**: ELK stack, Splunk, CloudWatch Logs
- **Distributed tracing**: Jaeger, Zipkin, AWS X-Ray
- **Alerting**: PagerDuty, Opsgenie integration

---

## Existing Products

### 1. **AWS API Gateway**
- **Type**: Managed service
- **Features**: REST/WebSocket APIs, request validation, caching, throttling, CORS
- **Pricing**: Pay-per-request model
- **Best for**: AWS-native applications, serverless architectures
- **Limitations**: Limited customization, AWS-specific

### 2. **Kong**
- **Type**: Open-source and enterprise
- **Features**: Plugin ecosystem, authentication, rate limiting, service discovery
- **Deployment**: Self-hosted or managed cloud
- **Best for**: Organizations wanting full control and customization
- **Strengths**: Highly extensible, large community

### 3. **Nginx**
- **Type**: Open-source and commercial
- **Features**: Reverse proxy, load balancing, caching, SSL/TLS
- **Deployment**: Self-hosted
- **Best for**: High-performance, lightweight deployments
- **Limitations**: Requires more manual configuration

### 4. **Traefik**
- **Type**: Open-source
- **Features**: Dynamic routing, automatic HTTPS, Docker/Kubernetes integration
- **Deployment**: Self-hosted, container-native
- **Best for**: Containerized and Kubernetes environments
- **Strengths**: Cloud-native, easy configuration

### 5. **Envoy Proxy**
- **Type**: Open-source
- **Features**: Advanced routing, load balancing, observability, service mesh integration
- **Deployment**: Self-hosted
- **Best for**: Complex microservices architectures, service mesh use cases
- **Strengths**: Powerful, used by major tech companies

### 6. **Azure API Management**
- **Type**: Managed service
- **Features**: API versioning, developer portal, analytics, policy engine
- **Pricing**: Tiered pricing model
- **Best for**: Azure-native applications, enterprise API management
- **Limitations**: Azure-specific, higher cost

### 7. **Google Cloud API Gateway**
- **Type**: Managed service
- **Features**: OpenAPI specification support, authentication, monitoring
- **Pricing**: Pay-per-million requests
- **Best for**: Google Cloud Platform users, serverless applications
- **Limitations**: GCP-specific

### 8. **Apigee**
- **Type**: Enterprise API management platform
- **Features**: Advanced analytics, developer portal, monetization, security
- **Pricing**: Enterprise pricing
- **Best for**: Large enterprises, API monetization
- **Strengths**: Comprehensive API lifecycle management

### 9. **Tyk**
- **Type**: Open-source and commercial
- **Features**: API gateway, developer portal, analytics, rate limiting
- **Deployment**: Self-hosted or managed cloud
- **Best for**: Organizations needing open-source with commercial support
- **Strengths**: Easy to deploy, good documentation

### 10. **MuleSoft**
- **Type**: Enterprise integration platform
- **Features**: API gateway, integration, transformation, analytics
- **Pricing**: Enterprise pricing
- **Best for**: Complex enterprise integrations
- **Limitations**: Steep learning curve, expensive

---

## Custom vs Existing API Gateway

### When to Use Existing API Gateway

| Scenario | Reason |
|----------|--------|
| **Startup/MVP Phase** | Faster time-to-market, no infrastructure overhead |
| **Limited DevOps Resources** | Managed services reduce operational burden |
| **Standard Requirements** | Existing products cover 80% of use cases |
| **Cloud-Native Architecture** | AWS/Azure/GCP managed services integrate seamlessly |
| **Compliance Requirements** | Managed services often have built-in compliance features |
| **Cost-Sensitive** | Pay-as-you-go model for variable traffic |
| **Need Developer Portal** | Many products include developer portals and documentation |

### When to Build Custom API Gateway

| Scenario | Reason |
|----------|--------|
| **Unique Business Logic** | Custom routing, transformation, or aggregation rules |
| **Performance Critical** | Need ultra-low latency, custom optimizations |
| **Vendor Lock-in Concerns** | Want complete control and portability |
| **Complex Integration** | Proprietary protocols or legacy system integration |
| **Cost Optimization** | High-volume traffic where custom solution is cheaper |
| **Specific Security Requirements** | Need custom authentication or encryption schemes |
| **Full Customization** | Existing products don't meet specific requirements |

### Hybrid Approach

Many organizations use a **hybrid approach**:

```
┌─────────────────────────────────────────────────────┐
│         Managed API Gateway (AWS/Azure)             │
│  - Public API endpoints                             │
│  - Standard authentication                          │
│  - Rate limiting                                    │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│      Custom API Gateway (Kong/Nginx)                │
│  - Internal service routing                         │
│  - Custom business logic                            │
│  - Service-to-service communication                 │
└────────────────────┬────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    Microservices  Databases   External APIs
```

---

## GraphQL as API Gateway
![GQL](../../Resources/07-api-gateway/Screenshot%202026-02-06%20at%2011.22.18 PM.png)

### What is GraphQL as API Gateway?

- GraphQL can serve as an API Gateway by providing a unified query interface that aggregates data from multiple backend services.

- Instead of REST endpoints, clients query a single GraphQL endpoint that resolves data from various sources.

### Architecture

```
┌──────────────────────────────────────────────────┐
│              GraphQL API Gateway                 │
│  ┌────────────────────────────────────────────┐  │
│  │  Query Resolver                            │  │
│  │  - Resolves User type → User Service       │  │
│  │  - Resolves Order type → Order Service     │  │
│  │  - Resolves Product type → Product Service │  │
│  │  - Resolves Review type → Review Service   │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    User Service  Order Service  Product Service
```

### Example Query

```graphql
query GetOrderDetails($orderId: ID!) {
  order(id: $orderId) {
    id
    status
    createdAt
    user {
      id
      name
      email
    }
    items {
      product {
        id
        name
        price
      }
      quantity
      reviews {
        rating
        comment
      }
    }
    payment {
      method
      status
    }
  }
}
```

This single query aggregates data from:
- Order Service (order details)
- User Service (user information)
- Product Service (product details)
- Review Service (product reviews)
- Payment Service (payment information)

### Pros of GraphQL as API Gateway

1. **Precise Data Fetching**
   - Clients request only the fields they need
   - Reduces over-fetching and under-fetching
   - Lower bandwidth usage

2. **Single Query Language**
   - Unified interface for all data access
   - Eliminates need to learn multiple REST endpoints
   - Easier for frontend developers

3. **Strong Typing**
   - GraphQL schema provides type safety
   - Self-documenting API
   - Better IDE support and autocomplete

4. **Aggregation Built-in**
   - Natural way to combine data from multiple services
   - Resolvers handle service calls transparently
   - Nested queries for related data

5. **Real-time Capabilities**
   - Subscriptions for real-time data updates
   - WebSocket support
   - Efficient change notifications

6. **Introspection**
   - Clients can query the schema itself
   - Automatic documentation generation
   - Better developer experience

7. **Versioning-Free**
   - Add new fields without breaking existing queries
   - Deprecate fields gracefully
   - No need for API versioning

### Cons of GraphQL as API Gateway

1. **Complexity**
   - Steeper learning curve than REST
   - More complex to implement and maintain
   - Requires GraphQL expertise

2. **Performance Challenges**
   - N+1 query problem (multiple database queries for single GraphQL query)
   - Requires careful resolver optimization
   - Batch loading and caching strategies needed

3. **Caching Difficulties**
   - HTTP caching doesn't work well with POST requests
   - Query-based caching is complex
   - Requires custom caching strategies

4. **Monitoring & Debugging**
   - Harder to monitor individual service calls
   - Complex query execution paths
   - Requires specialized GraphQL monitoring tools

5. **File Uploads**
   - Not as straightforward as REST multipart uploads
   - Requires special handling and libraries

6. **Real-time Complexity**
   - Subscriptions add operational complexity
   - Requires WebSocket infrastructure
   - Harder to scale

7. **Overkill for Simple APIs**
   - Unnecessary complexity for simple CRUD operations
   - REST might be simpler for straightforward use cases

### When to Use GraphQL as API Gateway

| Use Case | Reason |
|----------|--------|
| **Mobile Applications** | Precise data fetching reduces bandwidth and battery usage |
| **Multiple Client Types** | Different clients need different data shapes |
| **Complex Data Relationships** | Nested queries handle related data elegantly |
| **Rapid Frontend Development** | Schema-driven development speeds up iteration |
| **Real-time Features** | Subscriptions provide efficient real-time updates |
| **Data Aggregation Heavy** | Multiple service aggregation is natural in GraphQL |
| **Developer Experience Priority** | Strong typing and introspection improve DX |

### When NOT to Use GraphQL as API Gateway

| Use Case | Reason |
|----------|--------|
| **Simple CRUD APIs** | REST is simpler and sufficient |
| **File-Heavy Operations** | REST multipart uploads are more straightforward |
| **Legacy System Integration** | Existing systems may not support GraphQL well |
| **Team Unfamiliar with GraphQL** | Learning curve and expertise requirements |
| **Real-time Not Needed** | Subscriptions add unnecessary complexity |
| **Simple Caching Requirements** | HTTP caching works better with REST |
| **High-Performance Critical** | REST can be faster for simple queries |

### Hybrid Approach: REST + GraphQL

Many organizations use both:

```
┌─────────────────────────────────────────────────────┐
│              API Gateway                            │
│  ┌──────────────────┐      ┌──────────────────┐     │
│  │  REST Endpoints  │      │ GraphQL Endpoint │     │
│  │  /api/users      │      │ /graphql         │     │
│  │  /api/orders     │      │                  │     │
│  │  /api/products   │      │                  │     │
│  └──────────────────┘      └──────────────────┘     │
└─────────────────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    User Service  Order Service  Product Service
```

**Strategy**:
- Use REST for simple, straightforward operations
- Use GraphQL for complex data aggregation and mobile clients
- Share the same backend services and resolvers
- Gradually migrate to GraphQL as team expertise grows

---

## Summary

The **API Gateway pattern** is essential in microservices architectures because it:

1. **Centralizes** cross-cutting concerns (authentication, rate limiting, logging)
2. **Simplifies** client interactions with a single entry point
3. **Enables** service discovery and dynamic routing
4. **Aggregates** responses from multiple services
5. **Improves** security by hiding internal service details
6. **Enhances** performance through caching and optimization

While alternative patterns like BFF, Proxy, Composite, and Aggregator solve specific problems, only the API Gateway provides a comprehensive, centralized solution for all gateway concerns.

**GraphQL as an API Gateway** offers additional benefits for complex data aggregation scenarios but comes with trade-offs in complexity and performance optimization. The choice between REST and GraphQL should be based on specific use cases and team expertise.

Organizations should evaluate existing products (AWS API Gateway, Kong, Nginx, Traefik, Envoy) before building custom solutions, but a hybrid approach combining managed and custom gateways is often optimal for complex architectures.

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
