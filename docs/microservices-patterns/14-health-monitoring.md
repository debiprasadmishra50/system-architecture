# Event Driven Systems: Health Monitoring

## Table of Contents
1. [What is Health Monitoring](#what-is-health-monitoring)
2. [How Kubernetes Achieves It](#how-kubernetes-achieves-it)
3. [Kubernetes Probes](#kubernetes-probes)
4. [Problems It Solves](#problems-it-solves)

## What is Health Monitoring

- Health monitoring is the continuous process of checking the status and well-being of applications and services running in a distributed system. 

- It involves regularly assessing whether services are functioning correctly, responding to requests, and maintaining their expected behavior.

![image](../../Resources/14-health-monitoring/Screenshot%202026-02-07%20at%2010.20.34 AM.png)

In the context of microservices and containerized environments, health monitoring ensures that:
- Services are alive and responsive
- Services can handle incoming traffic
- Services are ready to serve requests
- Services recover gracefully from failures

Health monitoring goes beyond simple "is the process running" checks. It validates that services are actually healthy and capable of performing their intended functions, not just that they're technically alive.

## How Kubernetes Achieves It

Kubernetes implements health monitoring through a built-in probing mechanism that continuously evaluates the state of containers and pods. The Kubernetes control plane uses these probes to make intelligent decisions about pod lifecycle management.

### Kubernetes Health Monitoring Strategy

![image](../../Resources/14-health-monitoring/Screenshot%202026-02-07%20at%2010.18.17 AM.png)

1. **Continuous Probing**: Kubelet (the node agent) periodically executes probes against containers to determine their health status
2. **Automatic Recovery**: Based on probe results, Kubernetes automatically restarts unhealthy containers or removes pods from service
3. **Traffic Management**: Unhealthy pods are removed from service endpoints, preventing traffic from reaching failing services
4. **Graceful Shutdown**: Kubernetes allows services time to shut down gracefully before forcefully terminating them

### How It Works

```
┌────────────────────────────────────────────────────────┐
│                    Kubernetes Node                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Pod with Container                  │  │
│  │                                                  │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │         Application Container              │  │  │
│  │  │  (Running Service/Application)             │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  │                      ▲                           │  │
│  │                      │ Probes                    │  │
│  │                      │                           │  │
│  └──────────────────────┼───────────────────────────┘  │
│                         │                              │
│  ┌──────────────────────▼───────────────────────────┐  │
│  │           Kubelet (Health Checker)               │  │
│  │  - Executes Liveness Probes                      │  │
│  │  - Executes Readiness Probes                     │  │
│  │  - Executes Startup Probes                       │  │
│  └──────────────────────┬───────────────────────────┘  │
│                         │                              │
└─────────────────────────┼──────────────────────────────┘
                          │
                          ▼
                  ┌──────────────┐
                  │ Control Plane│
                  │ - Restarts   │
                  │ - Removes    │
                  │ - Reschedules│
                  └──────────────┘
```

## Kubernetes Probes

Kubernetes uses three types of probes to monitor container health:

### 1. Liveness Probe

**Purpose**: Determines if a container is still running and should be restarted if it fails.

**When it's used**: 
- Detects deadlocks or infinite loops
- Identifies containers that are technically running but not functioning
- Triggers automatic container restart

**Example**:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

If the probe fails 3 times consecutively, Kubernetes restarts the container.

### 2. Readiness Probe

**Purpose**: Determines if a container is ready to accept traffic and serve requests.

**When it's used**:
- Checks if the application has completed initialization
- Verifies database connections are established
- Confirms dependencies are available
- Prevents traffic routing to unprepared containers

**Example**:
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 2
```

If the probe fails, the pod is removed from the service's load balancer, but the container is NOT restarted.

### 3. Startup Probe

**Purpose**: Determines if an application has successfully started, especially for slow-starting applications.

**When it's used**:
- Applications with long initialization times
- Services that need to load large datasets on startup
- Legacy applications that take time to become ready

**Example**:
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

Once the startup probe succeeds, liveness and readiness probes take over. This prevents premature restarts during initialization.

### Probe Implementation Methods

Kubernetes supports three ways to implement probes:

| Method | Description | Use Case |
|--------|-------------|----------|
| **HTTP GET** | Makes an HTTP GET request to a specified path and port. Success = 2xx or 3xx status code | REST APIs, web services |
| **TCP Socket** | Attempts to establish a TCP connection to a specified port. Success = connection established | Databases, message queues, any TCP service |
| **Exec** | Executes a command inside the container. Success = exit code 0 | Custom health checks, scripts |

### Probe Configuration Parameters

- **initialDelaySeconds**: Time to wait before starting probes (allows app startup)
- **periodSeconds**: How often to run the probe (default: 10)
- **timeoutSeconds**: How long to wait for a response (default: 1)
- **successThreshold**: Consecutive successes needed to mark as healthy (default: 1)
- **failureThreshold**: Consecutive failures before taking action (default: 3)

## Problems It Solves

### 1. **Automatic Failure Detection and Recovery**

**Problem**: In traditional systems, failed services continue to receive traffic, causing cascading failures and poor user experience.

**Solution**: Kubernetes probes detect failures immediately and restart unhealthy containers automatically, reducing downtime and manual intervention.

### 2. **Prevents Traffic to Unhealthy Services**

**Problem**: Load balancers may route requests to services that are technically running but not functioning properly, causing request failures.

**Solution**: Readiness probes ensure only truly healthy services receive traffic. Failed readiness probes remove pods from the service endpoint, preventing bad requests.

### 3. **Handles Slow-Starting Applications**

**Problem**: Liveness probes might restart applications before they finish initializing, causing restart loops.

**Solution**: Startup probes give applications time to initialize without triggering restarts, while still detecting genuinely stuck processes.

### 4. **Reduces Manual Monitoring Overhead**

**Problem**: Without automated health checks, operators must manually monitor services and restart failed instances.

**Solution**: Kubernetes automates health monitoring and recovery, reducing operational burden and human error.

### 5. **Enables Self-Healing Infrastructure**

**Problem**: Failed services remain in the cluster, consuming resources and potentially causing issues.

**Solution**: Kubernetes automatically removes and reschedules unhealthy pods, maintaining cluster health without manual intervention.

### 6. **Improves System Resilience**

**Problem**: Cascading failures occur when one service failure affects dependent services.

**Solution**: Quick detection and recovery of failed services prevent cascading failures and maintain overall system stability.

### 7. **Supports Zero-Downtime Deployments**

**Problem**: Deploying new versions without proper health checks can cause service disruptions.

**Solution**: Readiness probes ensure new instances are fully ready before receiving traffic, enabling smooth rolling updates.

## Best Practices for Health Monitoring

1. **Define meaningful health checks**: Probes should check actual service functionality, not just process existence
2. **Set appropriate timeouts**: Balance between quick failure detection and avoiding false positives
3. **Use all three probe types**: Combine startup, readiness, and liveness probes for comprehensive monitoring
4. **Implement graceful shutdown**: Handle SIGTERM signals to allow in-flight requests to complete
5. **Monitor probe metrics**: Track probe success/failure rates to identify systemic issues
6. **Test probes thoroughly**: Ensure probes accurately reflect service health in various scenarios

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
