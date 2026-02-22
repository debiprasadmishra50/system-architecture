# Event Driven Systems: Aggregator Pattern

## Table of Contents
1. [What is Domain Driven Design](#what-is-domain-driven-design)
2. [What is Aggregator Pattern](#what-is-aggregator-pattern)
3. [What Problem Does It Solve](#what-problem-does-it-solve)
4. [Example](#example)
5. [Aggregator vs API Gateway](#aggregator-vs-api-gateway)
6. [Types of Aggregator](#types-of-aggregator)
7. [Aggregator Pattern with CQRS](#aggregator-pattern-with-cqrs)
8. [Patterns and Strategies for Implementing Aggregator Pattern](#patterns-and-strategies-for-implementing-aggregator-pattern)
9. [When to Use Aggregator and API Gateway](#when-to-use-aggregator-and-api-gateway)

---

## What is Domain Driven Design

Domain Driven Design (DDD) is a software development approach that emphasizes understanding and modeling the business domain. It focuses on:

- **Core Domain**: The unique business logic that differentiates your organization
- **Bounded Contexts**: Clear boundaries around different parts of the system where specific domain models apply
- **Ubiquitous Language**: A shared vocabulary between developers and domain experts
- **Aggregates**: Clusters of domain objects treated as a single unit for data changes
- **Entities and Value Objects**: Building blocks that represent domain concepts

DDD helps create systems that are closely aligned with business requirements and are easier to maintain and evolve. In microservices architectures, DDD principles guide the design of service boundaries and data ownership.

---

## What is Aggregator Pattern
|   ![problem](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.10.52 PM.png)  |  →  |  ![Solution](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.11.24 PM.png)  |
| :---: | :---: | :---: |



The Aggregator Pattern is a microservices design pattern that consolidates data from multiple microservices into a single response. It acts as a mediator that:

- Receives a client request
- Calls multiple backend microservices in parallel or sequence
- Aggregates the responses from these services
- Returns a unified response to the client

The aggregator can be a dedicated service, a controller, or a gateway that orchestrates calls to multiple services to fulfill a single business request.

---

## What Problem Does It Solve

The Aggregator Pattern addresses several critical challenges in microservices architectures:

1. **Chatty Client Problem**: Clients no longer need to make multiple calls to different services; the aggregator handles this complexity
2. **Network Latency**: Reduces the number of round trips between client and backend by batching multiple service calls
3. **Service Coupling**: Decouples clients from knowing about individual service endpoints and their contracts
4. **Data Consistency**: Provides a single point to orchestrate and combine data from multiple sources
5. **Cross-Cutting Concerns**: Centralizes authentication, logging, and error handling for multiple service calls
6. **Reduced Client Complexity**: Clients work with a simplified, unified API instead of managing multiple service interactions

---

## Example

### E-Commerce Order Details Page

Consider an e-commerce platform where displaying an order details page requires data from multiple services:

**Services involved:**
- **Order Service**: Contains order information (order ID, date, status)
- **Product Service**: Contains product details (name, price, description)
- **User Service**: Contains customer information (name, email, address)
- **Payment Service**: Contains payment details (method, status)
- **Inventory Service**: Contains stock information

**Without Aggregator:**
```
Client → Order Service → Product Service → User Service → Payment Service → Inventory Service
(Multiple round trips, complex client logic)
```

**With Aggregator:**
```
Client → Order Aggregator Service
         ├→ Order Service
         ├→ Product Service
         ├→ User Service
         ├→ Payment Service
         └→ Inventory Service
(Single request, aggregator handles orchestration)
```

**Response from Aggregator:**
```json
{
  "order": {
    "id": "ORD-12345",
    "date": "2026-02-06",
    "status": "shipped"
  },
  "customer": {
    "name": "John Doe",
    "email": "john@example.com",
    "address": "123 Main St"
  },
  "products": [
    {
      "id": "PROD-001",
      "name": "Laptop",
      "price": 999.99,
      "quantity": 1
    }
  ],
  "payment": {
    "method": "credit_card",
    "status": "completed"
  },
  "inventory": {
    "available": 5,
    "reserved": 1
  }
}
```

---

## Aggregator vs API Gateway

| Aspect | Aggregator | API Gateway |
|--------|-----------|------------|
| **Purpose** | Aggregates data from multiple services into a single response | Routes requests to appropriate backend services |
| **Scope** | Business logic focused; combines data for specific use cases | Infrastructure focused; handles cross-cutting concerns |
| **Responsibility** | Orchestrates service calls and merges responses | Request routing, rate limiting, authentication, logging |
| **Placement** | Typically deployed as a dedicated service or controller | Sits at the edge of the system, before services |
| **Complexity** | Handles business logic and data transformation | Handles technical concerns and protocol translation |
| **Scalability** | Scales based on business logic complexity | Scales based on request volume |
| **Data Transformation** | Transforms and combines data from multiple sources | Minimal transformation; mainly routing |
| **Error Handling** | Handles service failures and partial responses | Handles request validation and protocol errors |
| **Use Case** | Combining data for specific business operations | General request routing and infrastructure concerns |
| **Example** | Order details aggregator combining order, product, and user data | Kong, AWS API Gateway, Nginx |

---

## Types of Aggregator

### 1. Simple Aggregator
![Simple](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.13.30 PM.png)

A simple aggregator makes parallel calls to multiple services and combines their responses with minimal transformation.

**Characteristics:**
- Calls multiple services in parallel
- Minimal business logic
- Simple data merging
- Fast response times
- Low complexity

**Example:**
```
GET /api/user-profile/{userId}
├→ GET /users/{userId}
├→ GET /preferences/{userId}
└→ GET /notifications/{userId}

Response: { user, preferences, notifications }
```

### 2. Complex Aggregator
![Complex](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.13.36 PM.png)

A complex aggregator implements sophisticated orchestration logic, conditional calls, and data transformation.

**Characteristics:**
- Conditional service calls based on previous responses
- Complex business logic and workflows
- Data transformation and enrichment
- Handles service failures gracefully
- May implement retry logic and circuit breakers
- Potentially slower due to sequential dependencies

**Example:**
```
GET /api/order-details/{orderId}
1. Call Order Service → Get order info
2. If order status is "shipped":
   - Call Tracking Service → Get tracking info
3. Call Product Service → Get product details
4. Call User Service → Get customer info
5. Transform and merge all data
6. Return enriched response
```

---

## Aggregator Pattern with CQRS

The Aggregator Pattern works synergistically with CQRS (Command Query Responsibility Segregation):

**CQRS Basics:**
- **Commands**: Operations that modify state (Create, Update, Delete)
- **Queries**: Operations that read state (Read)

**Integration with Aggregator:**

1. **Query Side**: The aggregator acts as a query orchestrator, combining data from multiple read models optimized for different queries
2. **Command Side**: Commands are routed directly to appropriate services; the aggregator doesn't interfere with state changes
3. **Eventual Consistency**: The aggregator may work with eventually consistent data from different services
4. **Read Model Optimization**: Services can maintain specialized read models, and the aggregator combines them efficiently

**Example:**
```
Query: GET /api/dashboard
Aggregator combines:
- User read model (from User Service)
- Order summary read model (from Order Service)
- Analytics read model (from Analytics Service)
```

---

## Patterns and Strategies for Implementing Aggregator Pattern

### Scatter-Gather Pattern
![SG-pattern](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.15.30 PM.png)

The Scatter-Gather pattern is used when you need to call multiple services in parallel and wait for all responses before aggregating.

**How it works:**
1. **Scatter**: Send requests to multiple services simultaneously
2. **Gather**: Wait for all responses to return
3. **Aggregate**: Combine all responses into a single result

**Advantages:**
- Optimal for independent service calls
- Reduces overall latency compared to sequential calls
- Simple to implement and understand

**Disadvantages:**
- Slowest response is the bottleneck (tail latency problem)
- If one service fails, the entire request fails
- Resource intensive if many services are involved

**Example:**
```
Request: GET /api/product/{productId}
├→ Product Service (100ms)
├→ Review Service (150ms)
├→ Inventory Service (80ms)
└→ Recommendation Service (200ms)

Total Time: ~200ms (max of all calls)
```

### Chained Pattern
![Chained](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.15.54 PM.png)

The Chained Pattern calls services sequentially, where each service call depends on the result of the previous call.

**How it works:**
1. Call Service A
2. Use response from Service A to call Service B
3. Use response from Service B to call Service C
4. Continue until all required data is gathered

**Advantages:**
- Handles dependencies between services naturally
- Reduces unnecessary calls (conditional logic)
- Simpler error handling for dependent operations

**Disadvantages:**
- Slower overall response time (sequential calls)
- Latency accumulates with each call
- Not suitable for independent service calls

**Example:**
```
Request: GET /api/order/{orderId}/shipping-cost
1. Call Order Service → Get order details (100ms)
2. Use order details to call Shipping Service → Get cost (150ms)
3. Use shipping info to call Tax Service → Calculate tax (80ms)

Total Time: ~330ms (sum of all calls)
```

### Branch Pattern
![Branch](../../Resources/06-aggregator-pattern/Screenshot%202026-02-06%20at%209.16.19 PM.png)

The Branch Pattern combines both scatter-gather and chained patterns, creating a tree-like structure of service calls.

**How it works:**
1. Make initial calls in parallel (scatter)
2. Based on responses, make additional conditional calls
3. Gather all results and aggregate

**Advantages:**
- Flexible; handles both dependent and independent calls
- Optimizes latency by parallelizing where possible
- Reduces unnecessary calls through conditional logic

**Disadvantages:**
- More complex to implement
- Harder to reason about execution flow
- Requires careful error handling

**Example:**
```
Request: GET /api/customer/{customerId}/complete-profile
1. Parallel calls:
   - Customer Service → Get customer info (100ms)
   - Order Service → Get order history (120ms)
2. Based on customer type:
   - If premium: Call Loyalty Service (80ms)
   - If new: Call Onboarding Service (60ms)
3. Parallel calls:
   - Preference Service (90ms)
   - Notification Service (70ms)
4. Aggregate all data

Total Time: ~200ms (optimized with parallelization)
```

---

## When to Use Aggregator and API Gateway

### Use Aggregator When:

1. **Multiple Service Dependencies**: Your request requires data from 2+ microservices
2. **Business Logic Needed**: You need to transform, enrich, or combine data from different sources
3. **Specific Use Cases**: You have specific client needs (mobile, web, third-party) requiring different data combinations
4. **Reducing Client Complexity**: Clients shouldn't know about internal service architecture
5. **Cross-Service Orchestration**: You need to coordinate operations across multiple services
6. **Data Consistency**: You need to ensure data consistency across multiple service responses

**Example Scenarios:**
- E-commerce order details page
- User profile dashboard combining multiple data sources
- Recommendation engine combining user, product, and behavior data
- Complex reporting requiring data from multiple services

### Use API Gateway When:

1. **Request Routing**: You need to route requests to different backend services
2. **Cross-Cutting Concerns**: You need centralized authentication, rate limiting, logging
3. **Protocol Translation**: You need to translate between different protocols (REST, gRPC, etc.)
4. **Request/Response Transformation**: You need to transform requests or responses at the edge
5. **Service Discovery**: You need dynamic service discovery and load balancing
6. **Security**: You need centralized security policies and SSL termination

**Example Scenarios:**
- Routing API requests to appropriate microservices
- Enforcing rate limits across all services
- Centralizing authentication and authorization
- Logging and monitoring all API traffic
- Load balancing across service instances

### Combined Approach:

In modern microservices architectures, both patterns are often used together:

```
Client
  ↓
API Gateway (routing, auth, rate limiting)
  ↓
Aggregator Service (business logic, data combination)
  ↓
Multiple Microservices (Order, Product, User, etc.)
```

This layered approach provides:
- **Infrastructure concerns** handled by API Gateway
- **Business logic** handled by Aggregator
- **Clear separation of concerns**
- **Scalability** at each layer
- **Flexibility** to evolve independently

---

## Summary

- The Aggregator Pattern is essential in microservices architectures for combining data from multiple services efficiently. 
- By understanding when to use aggregators, how to implement them with patterns like Scatter-Gather, Chained, and Branch, and how they complement API Gateways, you can design systems that are both performant and maintainable. 
- The key is choosing the right pattern for your specific use case and balancing complexity with performance requirements.


---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
