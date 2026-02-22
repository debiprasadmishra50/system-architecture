# Event Driven Systems: Database Per Service

## Table of Contents

1. [What is Database Per Service?](#what-is-database-per-service)
2. [Why Database Per Service?](#why-database-per-service)
3. [What Does It Solve?](#what-does-it-solve)
4. [Pros and Cons of Database Per Service](#pros-and-cons-of-database-per-service)
5. [Implementation Considerations](#implementation-considerations)
6. [When to Use Database Per Service](#when-to-use-database-per-service)

---

## What is Database Per Service?

- Database Per Service is a microservices architectural pattern where each microservice has its own dedicated database instance. 
- Rather than sharing a single monolithic database across multiple services, each service owns and manages its own data store. 
- This database can be of any type (relational, NoSQL, document-based, etc.) that best suits the service's specific needs.

![DB per service](../../Resources/05-database-per-service/Screenshot%202026-02-06%20at%209.01.06 PM.png)

### Key Characteristics:

- **Service Ownership**: Each microservice is responsible for its own data persistence layer
- **Data Isolation**: Services do not directly access other services' databases
- **Technology Flexibility**: Different services can use different database technologies
- **Independent Scaling**: Each service's database can be scaled independently based on its load
- **Loose Coupling**: Services communicate through APIs or events, not through shared databases

---

## Why Database Per Service?

The Database Per Service pattern addresses fundamental challenges in microservices architecture:

### 1. **Service Independence**
In a monolithic architecture with a shared database, services become tightly coupled at the data layer. Changes to the database schema can impact multiple services, making it difficult to deploy services independently. Database Per Service ensures true service autonomy.

### 2. **Technology Diversity**
Different services have different data access patterns and requirements. A service handling time-series data might benefit from a time-series database, while another handling relational data might prefer PostgreSQL. This pattern allows each service to choose the optimal database technology.

### 3. **Scalability**
With a shared database, scaling becomes a bottleneck. Database Per Service allows each service to scale its database independently based on its specific load and performance requirements.

### 4. **Fault Isolation**
If one service's database fails, it doesn't cascade to other services. This improves system resilience and fault tolerance.

### 5. **Development Velocity**
Teams can work independently on their services without coordinating database schema changes with other teams, enabling faster development cycles.

---

## What Does It Solve?

### Problems Addressed:

1. **Database Bottleneck**: Eliminates the single point of failure and performance bottleneck of a shared database
2. **Schema Coupling**: Prevents tight coupling through shared database schemas
3. **Scaling Limitations**: Allows independent scaling of each service's data layer
4. **Technology Lock-in**: Enables services to use the most appropriate database technology
5. **Team Autonomy**: Allows different teams to work independently without database coordination
6. **Deployment Constraints**: Enables independent deployment of services without database migration coordination
7. **Data Consistency Trade-offs**: Forces explicit handling of consistency requirements, leading to better architectural decisions

---

## Pros and Cons of Database Per Service

| **Aspect** | **Pros** | **Cons** |
|---|---|---|
| **Service Independence** | Each service can be deployed independently without database coordination | Requires careful API design for inter-service communication |
| **Technology Flexibility** | Services can use the best-fit database technology (SQL, NoSQL, Graph, etc.) | Increases operational complexity with multiple database types |
| **Scalability** | Each service's database can scale independently based on its load | Requires separate scaling and monitoring for each database |
| **Fault Isolation** | Database failure in one service doesn't affect others | Increases total number of potential failure points |
| **Development Speed** | Teams work independently without schema coordination | Requires strong API contracts and versioning strategies |
| **Performance Optimization** | Each database can be optimized for its specific access patterns | Potential for data duplication and inconsistency |
| **Data Ownership** | Clear ownership and responsibility for data | Difficult to maintain data consistency across services |
| **Operational Simplicity** | Simpler deployment pipeline for individual services | Complex distributed transactions and eventual consistency handling |
| **Cost Efficiency** | Can use cheaper, specialized databases for specific needs | Higher overall infrastructure and licensing costs |
| **Reduced Coupling** | Loose coupling at the data layer | Requires event-driven or API-based communication overhead |
| **Query Flexibility** | Services can optimize queries for their specific needs | Cross-service queries require API calls or data synchronization |
| **Backup & Recovery** | Independent backup strategies per service | More complex backup and disaster recovery procedures |

---

## Implementation Considerations

### Data Consistency Challenges

With Database Per Service, maintaining data consistency across services becomes complex. Consider these approaches:

- **Eventual Consistency**: Accept that data will be eventually consistent across services
- **Saga Pattern**: Use distributed transactions to maintain consistency across services
- **Event Sourcing**: Maintain an event log to reconstruct state and ensure consistency
- **CQRS**: Separate read and write models to handle consistency requirements

### Inter-Service Communication

Services must communicate through:
- **REST APIs**: Synchronous communication for immediate data needs
- **Message Queues**: Asynchronous communication for event-driven workflows
- **Event Streams**: Publish-subscribe patterns for data synchronization

### Data Duplication

To avoid expensive cross-service queries, services may need to maintain copies of data from other services. This requires:
- Event-driven synchronization
- Cache invalidation strategies
- Eventual consistency handling

---

## When to Use Database Per Service

✅ **Use when:**
- Services have different data access patterns
- Services need independent scaling
- Teams need autonomy in development
- Services use different database technologies
- High availability and fault isolation are critical

❌ **Avoid when:**
- Strong ACID transactions across services are required
- Services frequently need to query each other's data
- Team lacks experience with distributed systems
- Operational complexity is a major concern
- Data consistency requirements are strict and immediate

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
