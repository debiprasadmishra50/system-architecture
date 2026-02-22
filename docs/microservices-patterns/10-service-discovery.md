# Event Driven Systems: Service Discovery

## Table of Contents
1. [What is Service Discovery in Microservices](#what-is-service-discovery-in-microservices)
2. [Why Service Discovery](#why-service-discovery)
3. [What Problem Does It Solve](#what-problem-does-it-solve)
4. [Service Registry and Patterns](#service-registry-and-patterns)
5. [Service Discovery Types](#service-discovery-types)
6. [Why It's Necessary for Microservices](#why-its-necessary-for-microservices)
7. [Tools for Service Discovery](#tools-for-service-discovery)
8. [Simple Example Using Eureka](#simple-example-using-eureka)

---

## What is Service Discovery in Microservices

- Service Discovery is a mechanism that automatically detects and locates services in a microservices architecture. 

- It enables services to find and communicate with each other dynamically without hardcoding IP addresses or hostnames.

- In a microservices environment, services are deployed across multiple servers, containers, or cloud instances. 

- Service Discovery maintains a registry of all available services and their network locations (IP addresses and ports), allowing services to query this registry to find other services they need to communicate with.

**Key Characteristics:**
- **Dynamic Registration**: Services automatically register themselves when they start
- **Automatic Deregistration**: Services are removed from the registry when they stop
- **Health Checking**: Continuous monitoring of service health and availability
- **Load Balancing**: Distribution of requests across multiple instances of a service
- **Fault Tolerance**: Automatic handling of service failures and unavailability

---

## Why Service Discovery

Service Discovery is essential in microservices architectures for several reasons:

1. **Dynamic Environment**: In containerized and cloud-native environments, services are constantly being created, destroyed, and moved. Service Discovery handles this dynamism automatically.

2. **Scalability**: As services scale up or down, Service Discovery automatically updates the registry, allowing other services to discover new instances without manual intervention.

3. **Decoupling**: Services don't need to know the exact location of other services. They query the Service Discovery system instead, reducing tight coupling.

4. **Resilience**: When a service instance fails, Service Discovery removes it from the registry, preventing requests from being sent to dead instances.

5. **Operational Efficiency**: Eliminates the need for manual configuration and DNS updates when services are deployed or moved.

6. **Multi-Environment Support**: Works seamlessly across different deployment environments (on-premises, cloud, hybrid).

---

## What Problem Does It Solve

### Traditional Monolithic Approach Problems

In traditional monolithic applications, all components run on the same server with fixed addresses. However, in microservices:

**Problems Without Service Discovery:**

1. **Hardcoded Addresses**: Services would need hardcoded IP addresses and ports
   - Difficult to manage when services move or scale
   - Requires manual updates and redeployment

2. **Service Unavailability**: No automatic detection of failed service instances
   - Requests sent to dead services cause failures
   - Manual intervention required to remove failed instances

3. **Scaling Challenges**: Adding new service instances requires manual configuration
   - Load balancing configuration must be updated manually
   - No automatic distribution of traffic to new instances

4. **Configuration Management Nightmare**: Managing service locations across multiple environments
   - Different configurations for dev, staging, and production
   - Error-prone manual updates

5. **Network Complexity**: In containerized environments, IP addresses change frequently
   - Containers are ephemeral and get new IPs on restart
   - Traditional DNS is too slow for dynamic environments

**Service Discovery Solves These By:**
- Automatically registering and deregistering services
- Maintaining an up-to-date registry of available services
- Providing health checks to detect failures
- Enabling automatic load balancing
- Supporting dynamic scaling without manual intervention

---

## Service Registry and Patterns

### What is a Service Registry

A Service Registry is a database that maintains information about all available services in the system. It stores:
- Service name
- Service instances (IP address, port, metadata)
- Health status
- Service metadata (version, tags, etc.)

### Service Registry Patterns

#### 1. **Client-Side Service Discovery**
![iamge](../../Resources/10-service-discovery/Screenshot%202026-02-07%20at%2012.19.06 AM.png)

The client is responsible for querying the Service Registry and selecting a service instance.

**How it works:**
- Client queries the Service Registry to get available instances
- Client selects an instance (using load balancing logic)
- Client makes a direct request to the selected instance

**Advantages:**
- Simple to implement
- No additional network hop
- Client has full control over load balancing logic

**Disadvantages:**
- Client must implement service discovery logic
- Client must handle load balancing
- Client must implement retry logic for failed instances

**Example Tools:** Eureka (Netflix), Consul

#### 2. **Server-Side Service Discovery**
![image](../../Resources/10-service-discovery/Screenshot%202026-02-07%20at%2012.19.43 AM.png)

A load balancer or API Gateway queries the Service Registry and routes requests to available instances.

**How it works:**
- Client sends request to a load balancer/API Gateway
- Load balancer queries the Service Registry
- Load balancer selects an instance and routes the request
- Load balancer returns the response to the client

**Advantages:**
- Client doesn't need to implement discovery logic
- Centralized load balancing
- Easier to change load balancing strategy

**Disadvantages:**
- Additional network hop through load balancer
- Load balancer becomes a potential bottleneck
- Load balancer must be highly available

**Example Tools:** Kubernetes Service, AWS ELB, Nginx

#### 3. **Self-Registration Pattern**
![image](../../Resources/10-service-discovery/Screenshot%202026-02-07%20at%2012.18.13 AM.png)

Services register themselves with the Service Registry when they start.

**How it works:**
- Service starts and registers itself with the registry
- Service sends periodic heartbeats to keep registration alive
- Service deregisters itself when shutting down

**Advantages:**
- Service has control over its registration
- Simple to implement
- Works well with dynamic environments

**Disadvantages:**
- Service must implement registration logic
- Service must handle heartbeats and deregistration
- If service crashes, it may leave stale entries

#### 4. **Third-Party Registration Pattern**
![image](../../Resources/10-service-discovery/Screenshot%202026-02-07%20at%2012.26.26 AM.png)

An external component (registrar) registers services with the Service Registry.

**How it works:**
- External registrar monitors service deployments
- Registrar automatically registers new services
- Registrar removes services when they're undeployed
- Registrar handles health checks and deregistration

**Advantages:**
- Services don't need to implement registration logic
- Centralized management of registrations
- Works well with container orchestration platforms

**Disadvantages:**
- Additional component to manage
- Registrar must be highly available
- More complex setup

**Example:** Kubernetes automatically registers services

---

## Service Discovery Types

### 1. **DNS-Based Service Discovery**

Uses DNS to resolve service names to IP addresses.

**How it works:**
- Services register with a DNS server
- Clients query DNS to resolve service names
- DNS returns IP addresses of available instances

**Advantages:**
- Simple and widely supported
- Works across different platforms
- No additional client libraries needed

**Disadvantages:**
- DNS caching can cause stale entries
- Limited load balancing capabilities
- Slow updates in dynamic environments

**Where it's used:**
- Kubernetes (via CoreDNS)
- Traditional infrastructure
- Simple deployments

**Example:**
```
service-name.namespace.svc.cluster.local → 10.0.0.1, 10.0.0.2, 10.0.0.3
```

### 2. **API-Based Service Discovery**

Services query an API endpoint to discover other services.

**How it works:**
- Services query a Service Registry API
- API returns list of available instances
- Client selects an instance and makes a request

**Advantages:**
- More control over discovery logic
- Can return rich metadata about services
- Supports complex load balancing strategies
- Real-time updates

**Disadvantages:**
- Requires client library implementation
- Additional API calls for discovery
- Client must handle failures

**Where it's used:**
- Netflix Eureka
- Consul
- Spring Cloud Discovery
- Custom implementations

**Example:**
```
GET /api/services/user-service
Response: [
  { "host": "10.0.0.1", "port": 8080, "status": "UP" },
  { "host": "10.0.0.2", "port": 8080, "status": "UP" }
]
```

### 3. **Configuration-Based Service Discovery**

Service locations are configured in a centralized configuration system.

**How it works:**
- Service locations are stored in a configuration file or system
- Services read configuration to discover other services
- Configuration is updated when services are deployed

**Advantages:**
- Simple to understand
- Works in any environment
- No additional infrastructure needed

**Disadvantages:**
- Manual updates required
- Not suitable for dynamic environments
- Doesn't scale well

**Where it's used:**
- Development environments
- Static deployments
- Legacy systems

**Example:**
```yaml
services:
  user-service: http://10.0.0.1:8080
  order-service: http://10.0.0.2:8080
  payment-service: http://10.0.0.3:8080
```

### 4. **Load Balancer-Based Service Discovery**

A load balancer maintains the registry and routes requests.

**How it works:**
- Load balancer maintains list of service instances
- Clients send requests to load balancer
- Load balancer routes to available instances

**Advantages:**
- Centralized management
- Transparent to clients
- Built-in load balancing

**Disadvantages:**
- Load balancer is a single point of failure
- Additional network hop
- Less flexible for client-side logic

**Where it's used:**
- Kubernetes Services
- AWS ELB/ALB
- Nginx/HAProxy
- Cloud load balancers

---

## Why It's Necessary for Microservices

### 1. **Ephemeral Nature of Services**

In microservices architectures, services are ephemeral:
- Containers are created and destroyed frequently
- Services are deployed and undeployed regularly
- IP addresses change on every restart
- Service instances scale up and down dynamically

Service Discovery handles this by automatically updating the registry as services come and go.

### 2. **Distributed Communication**

Microservices need to communicate across the network:
- Services are deployed on different servers/containers
- Network locations are not fixed
- Services need to find each other dynamically

Service Discovery provides a mechanism for services to locate each other without hardcoding addresses.

### 3. **High Availability and Resilience**

Service Discovery enables high availability:
- Detects failed service instances
- Removes failed instances from the registry
- Routes requests only to healthy instances
- Supports automatic failover

### 4. **Scalability**

Service Discovery enables horizontal scaling:
- New instances are automatically registered
- Load is distributed across instances
- Scaling is transparent to clients
- No manual configuration needed

### 5. **Operational Simplicity**

Service Discovery reduces operational complexity:
- No manual service location management
- Automatic handling of service movements
- Centralized visibility of all services
- Simplified deployment process

### 6. **Multi-Environment Support**

Service Discovery works across different environments:
- Same code works in dev, staging, and production
- No environment-specific configuration needed
- Supports hybrid and multi-cloud deployments

---

## Tools for Service Discovery

### 1. **Eureka (Netflix)**

A REST-based service registry built by Netflix.

**Features:**
- Client-side service discovery
- Self-registration pattern
- Health checking
- Load balancing support
- Highly available

**Use Cases:**
- Spring Boot applications
- Netflix microservices
- Java-based systems

**Pros:**
- Mature and battle-tested
- Good Spring Cloud integration
- Resilient to network partitions

**Cons:**
- Java-centric
- Requires client library
- Less feature-rich than Consul

### 2. **Consul (HashiCorp)**

A distributed service mesh and service discovery platform.

**Features:**
- DNS and HTTP API-based discovery
- Health checking
- Key-value store
- Service mesh capabilities
- Multi-datacenter support

**Use Cases:**
- Polyglot microservices
- Multi-datacenter deployments
- Service mesh implementations

**Pros:**
- Language-agnostic
- Rich feature set
- Excellent documentation

**Cons:**
- More complex than Eureka
- Requires more operational overhead
- Steeper learning curve

### 3. **Kubernetes Service Discovery**

Built-in service discovery in Kubernetes.

**Features:**
- DNS-based discovery
- Automatic service registration
- Load balancing
- Health checking
- Service mesh integration

**Use Cases:**
- Kubernetes deployments
- Container orchestration
- Cloud-native applications

**Pros:**
- Built-in to Kubernetes
- No additional tools needed
- Highly scalable

**Cons:**
- Kubernetes-specific
- Limited to Kubernetes environments
- Less flexible than dedicated tools

### 4. **Zookeeper (Apache)**

A distributed coordination service used for service discovery.

**Features:**
- Distributed configuration management
- Service registration
- Leader election
- Health checking

**Use Cases:**
- Hadoop ecosystems
- Kafka deployments
- Distributed systems

**Pros:**
- Mature and stable
- Good for distributed coordination
- Wide adoption

**Cons:**
- Complex to set up
- Requires Java
- Steeper learning curve

### 5. **etcd (CoreOS)**

A distributed key-value store used for service discovery.

**Features:**
- Distributed configuration
- Service registration
- Health checking
- Watch mechanism for changes

**Use Cases:**
- Kubernetes (uses etcd internally)
- Distributed systems
- Configuration management

**Pros:**
- Simple and lightweight
- Good performance
- Used by Kubernetes

**Cons:**
- Requires additional tooling for service discovery
- Less feature-rich than Consul
- Smaller ecosystem

### 6. **AWS Service Discovery**

AWS's managed service discovery solution.

**Features:**
- Integration with AWS services
- DNS and API-based discovery
- Health checking
- Auto-scaling support

**Use Cases:**
- AWS deployments
- ECS/EKS clusters
- AWS-native applications

**Pros:**
- Managed service (no operational overhead)
- Good AWS integration
- Scalable

**Cons:**
- AWS-specific
- Less flexible than open-source solutions
- Vendor lock-in

---

## Simple Example Using Eureka

### Setup: Eureka Server

**pom.xml (Maven Dependencies)**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**EurekaServerApplication.java**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**application.yml (Eureka Server Configuration)**
```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

### Setup: Eureka Client (User Service)

**pom.xml (Maven Dependencies)**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**UserServiceApplication.java**
```java
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

**application.yml (User Service Configuration)**
```yaml
spring:
  application:
    name: user-service

server:
  port: 8081

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**UserController.java**
```java
@RestController
@RequestMapping("/users")
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = new User(id, "John Doe", "john@example.com");
        return ResponseEntity.ok(user);
    }
}
```

### Setup: Eureka Client (Order Service - Calling User Service)

**application.yml (Order Service Configuration)**
```yaml
spring:
  application:
    name: order-service

server:
  port: 8082

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**OrderController.java (Using Service Discovery)**
```java
@RestController
@RequestMapping("/orders")
public class OrderController {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        // Method 1: Using DiscoveryClient
        List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
        if (instances.isEmpty()) {
            return ResponseEntity.notFound().build();
        }
        
        ServiceInstance instance = instances.get(0);
        String userServiceUrl = "http://" + instance.getHost() + ":" + instance.getPort();
        
        // Call user service
        User user = restTemplate.getForObject(userServiceUrl + "/users/1", User.class);
        
        Order order = new Order(id, user, "PENDING");
        return ResponseEntity.ok(order);
    }
    
    @GetMapping("/{id}/with-ribbon")
    public ResponseEntity<Order> getOrderWithRibbon(@PathVariable Long id) {
        // Method 2: Using @LoadBalanced RestTemplate (automatic load balancing)
        User user = restTemplate.getForObject("http://user-service/users/1", User.class);
        
        Order order = new Order(id, user, "PENDING");
        return ResponseEntity.ok(order);
    }
}
```

**RestTemplateConfig.java (Load Balanced RestTemplate)**
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### How It Works

1. **Eureka Server starts** on `http://localhost:8761`
2. **User Service starts** and registers itself with Eureka
   - Service name: `user-service`
   - Instance URL: `http://localhost:8081`
3. **Order Service starts** and registers itself with Eureka
   - Service name: `order-service`
   - Instance URL: `http://localhost:8082`
4. **Order Service discovers User Service** using Eureka
   - Queries Eureka for `user-service` instances
   - Gets list of available instances
   - Calls User Service using discovered URL
5. **Eureka Dashboard** available at `http://localhost:8761`
   - Shows all registered services
   - Shows health status of instances
   - Shows instance metadata

### Testing the Example

**Start Eureka Server:**
```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8761"
```

**Start User Service:**
```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"
```

**Start Order Service:**
```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8082"
```

**Test Service Discovery:**
```bash
# Get order (which internally calls user service via Eureka)
curl http://localhost:8082/orders/1

# View Eureka dashboard
open http://localhost:8761
```

### Key Points

- **Automatic Registration**: Services automatically register with Eureka on startup
- **Automatic Deregistration**: Services automatically deregister on shutdown
- **Health Checks**: Eureka periodically checks service health
- **Load Balancing**: `@LoadBalanced` RestTemplate automatically load balances across instances
- **Resilience**: If a service instance fails, Eureka removes it from the registry
- **Scalability**: New instances are automatically discovered and added to load balancing

---

## Summary

Service Discovery is a critical component of microservices architectures that enables:
- **Dynamic service location management** without hardcoding addresses
- **Automatic scaling** with transparent load balancing
- **High availability** through health checking and failover
- **Operational simplicity** by eliminating manual configuration

By using tools like Eureka, Consul, or Kubernetes Service Discovery, organizations can build resilient, scalable microservices systems that adapt to changing infrastructure and load conditions automatically.

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
