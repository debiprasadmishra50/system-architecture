# Event Driven Systems: Strangler Fig Pattern

## Table of Contents
1. [What is Strangler Pattern Design Strategy?](#what-is-strangler-pattern)
2. [Real-World Examples](#real-world-examples)
3. [How to Implement the Pattern](#implementation)
4. [Facade Layer](#facade-layer)
5. [Challenges with Strangler Pattern](#challenges)

---

## What is Strangler Pattern Design Strategy?

- The **Strangler Fig Pattern** is a software modernization strategy that allows you to gradually replace an old system with a new one without requiring a complete rewrite or a "big bang" migration. 
- The pattern is named after the strangler fig tree, which grows around an existing tree and eventually replaces it.

![Strangler Pattern](../../Resources/04-strangler-fig/Screenshot%202026-02-06%20at%208.46.22 PM.png)

### Key Characteristics

- **Gradual Migration**: Replace legacy systems incrementally rather than all at once
- **Parallel Execution**: Old and new systems run side-by-side during the transition
- **Risk Reduction**: Minimize the risk of complete system failure by migrating feature by feature
- **Continuous Delivery**: Deploy new features independently without waiting for the entire system migration
- **Reversibility**: Easy to rollback if issues arise with the new implementation

### Core Principles

1. **Incremental Replacement**: Identify specific features or modules to migrate first
2. **Facade Layer**: Introduce an intermediary layer that routes requests to either the old or new system
3. **Feature Parity**: Ensure new system functionality matches the old system before full migration
4. **Monitoring & Validation**: Continuously monitor both systems to ensure correctness
5. **Gradual Traffic Shifting**: Slowly increase traffic to the new system as confidence grows

### Benefits

- **Lower Risk**: Failures affect only a portion of the system
- **Faster Time to Value**: New features can be deployed independently
- **Team Flexibility**: Different teams can work on legacy and new systems simultaneously
- **Reduced Downtime**: No need for scheduled maintenance windows
- **Data Consistency**: Easier to maintain data consistency with parallel systems
- **Learning Opportunity**: Teams can learn the new technology stack gradually

---

## Real-World Examples 

### Amazon's Monolith to Microservices Migration
![Amazon's Migration](../../Resources/04-strangler-fig/Screenshot%202026-02-06%20at%208.47.07 PM.png)

Amazon is one of the most famous examples of successfully applying the Strangler Fig Pattern. In the early 2000s, Amazon's entire platform was built as a monolithic application. As the company scaled, they needed to break it down into microservices.

**How Amazon Applied the Pattern:**

1. **Initial State**: Single monolithic application handling all operations (product catalog, orders, payments, shipping, etc.)

2. **Strangling Process**:
   - Started with the **Product Catalog Service** - extracted product information into a separate microservice
   - Created a facade layer (API Gateway) that routed catalog requests to the new service
   - Gradually migrated other services: Order Service, Payment Service, Shipping Service, etc.
   - Each service was independently deployable and scalable

3. **Facade Implementation**:
   - Used API Gateway to route requests intelligently
   - Maintained backward compatibility with existing clients
   - Implemented feature flags to control traffic routing

4. **Results**:
   - Achieved independent scaling of services based on demand
   - Enabled different teams to own different services
   - Improved deployment frequency from quarterly to daily releases
   - Better fault isolation - failure in one service doesn't crash the entire platform

**Timeline**: This migration took several years (2000s-2010s) but allowed Amazon to become the cloud infrastructure leader with AWS.

### Netflix's Legacy System Modernization

Netflix faced a critical challenge in 2008 when their DVD rental business experienced a major database corruption incident. This prompted them to migrate from a monolithic Java application to a microservices architecture using the Strangler Fig Pattern.

**How Netflix Applied the Pattern:**

1. **Initial Challenge**: 
   - Monolithic application was difficult to scale
   - Database corruption caused significant downtime
   - Needed to support rapid feature development

2. **Strangling Strategy**:
   - **Phase 1**: Extracted the **Recommendation Engine** into a separate service
     - This was a compute-intensive service that benefited from independent scaling
     - Used a facade layer to route recommendation requests
   
   - **Phase 2**: Extracted **User Authentication & Authorization**
     - Created a dedicated identity service
     - All requests went through this service via the facade layer
   
   - **Phase 3**: Extracted **Content Delivery & Streaming**
     - Separated video streaming logic from the monolith
     - Enabled independent optimization for streaming performance
   
   - **Phase 4**: Extracted **Billing & Subscription Management**
     - Separated payment processing from core application logic

3. **Facade Implementation**:
   - Built **Zuul** (API Gateway) to route requests to appropriate services
   - Implemented intelligent routing based on request type
   - Maintained backward compatibility with client applications

4. **Results**:
   - Scaled from handling thousands to millions of concurrent users
   - Reduced deployment time from weeks to minutes
   - Enabled independent team ownership of services
   - Improved system resilience - service failures are isolated
   - Achieved 99.99% uptime SLA

**Key Insight**: Netflix's migration took approximately 5-7 years but transformed them from a struggling DVD rental company to a global streaming giant capable of handling 200+ million subscribers.

---

## How to Implement the Pattern

### Step-by-Step Implementation Guide

#### Step 1: Identify Strangling Candidates

```
Analysis Phase:
├── Identify loosely coupled modules
├── Find services with clear boundaries
├── Prioritize based on:
│   ├── Business value
│   ├── Technical complexity
│   ├── Team capacity
│   └── Risk level
└── Create migration roadmap
```

**Example**: If you have a monolithic e-commerce system, start with:
- Product Catalog (read-heavy, clear boundaries)
- User Authentication (used by all services)
- Inventory Management (moderate complexity)

#### Step 2: Design the Facade Layer

The facade layer acts as a router between the old and new systems.

```
Client Request
    ↓
Facade Layer (Router)
    ├─→ Route to Old System (Legacy)
    └─→ Route to New System (Modern)
    ↓
Response
```

**Implementation Considerations**:
- Use API Gateway or reverse proxy
- Implement request routing logic
- Add feature flags for traffic control
- Log all routing decisions for debugging

#### Step 3: Build the New Service

```
New Service Development:
├── Implement core functionality
├── Ensure feature parity with legacy
├── Add comprehensive logging
├── Implement health checks
├── Create integration tests
└── Deploy to staging environment
```

**Best Practices**:
- Use the same data models initially
- Implement the same business logic
- Add extensive monitoring and alerting
- Create automated tests for all scenarios

#### Step 4: Implement Data Synchronization

```
Data Flow:
┌─────────────────────────────────────┐
│     Shared Data Store               │
│  (Database/Message Queue)           │
└─────────────────────────────────────┘
        ↑                    ↑
        │                    │
    Old System          New System
    (Read/Write)        (Read/Write)
```

**Strategies**:
- **Dual-Write Pattern**: Write to both old and new systems
- **Event Sourcing**: Publish events that both systems consume
- **Change Data Capture (CDC)**: Automatically sync data changes
- **Message Queue**: Use async messaging for data consistency

#### Step 5: Implement Routing Logic

```javascript
// Example Facade Layer Routing Logic
function routeRequest(request) {
  const featureFlag = getFeatureFlag(request.feature);
  const userPercentage = getUserPercentageForNewSystem();
  
  if (featureFlag.enabled && userPercentage > Math.random() * 100) {
    return routeToNewSystem(request);
  } else {
    return routeToLegacySystem(request);
  }
}
```

**Routing Strategies**:
- **Percentage-Based**: Route X% of traffic to new system
- **User-Based**: Route specific users to new system
- **Feature-Based**: Route specific features to new system
- **Time-Based**: Route based on time of day or day of week
- **Canary Deployment**: Route to new system for testing before full rollout

#### Step 6: Monitor and Validate

```
Monitoring Strategy:
├── Compare responses from both systems
├── Track error rates
├── Monitor latency
├── Validate data consistency
├── Alert on anomalies
└── Collect metrics for analysis
```

**Key Metrics**:
- Response time comparison
- Error rate comparison
- Data consistency checks
- User experience metrics
- System resource utilization

#### Step 7: Gradual Traffic Migration

```
Traffic Migration Timeline:
Week 1:  1% → New System  (99% → Legacy)
Week 2:  5% → New System  (95% → Legacy)
Week 3: 10% → New System  (90% → Legacy)
Week 4: 25% → New System  (75% → Legacy)
Week 5: 50% → New System  (50% → Legacy)
Week 6: 75% → New System  (25% → Legacy)
Week 7: 90% → New System  (10% → Legacy)
Week 8: 100% → New System (0% → Legacy)
```

#### Step 8: Decommission Legacy System

```
Decommissioning Checklist:
☐ All traffic routed to new system
☐ No errors in new system
☐ Data migration complete
☐ Backup of legacy system created
☐ Documentation updated
☐ Team trained on new system
☐ Legacy system shutdown scheduled
☐ Monitoring for issues continues
```

---

## Facade Layer 

### What is a Facade Layer?

The **Facade Layer** is an intermediary component that sits between clients and the actual services. It provides a unified interface and handles the complexity of routing requests to either the old or new system.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Applications                      │
│              (Web, Mobile, Third-party APIs)                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    FACADE LAYER                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Request Router                                      │   │
│  │  ├─ Feature Flag Evaluation                          │   │
│  │  ├─ User Segmentation                                │   │
│  │  ├─ Traffic Percentage Control                       │   │
│  │  └─ Request Transformation                           │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Response Aggregation & Validation                   │   │
│  │  ├─ Response Transformation                          │   │
│  │  ├─ Error Handling                                   │   │
│  │  ├─ Response Caching                                 │   │
│  │  └─ Logging & Monitoring                             │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
         ┌───────────────┴───────────────┐
         ↓                               ↓
┌──────────────────────┐      ┌──────────────────────┐
│   LEGACY SYSTEM      │      │   NEW SYSTEM         │
│  (Monolith/Old)      │      │  (Microservice)      │
│                      │      │                      │
│ ├─ Product Catalog   │      │ ├─ Product Service   │
│ ├─ Orders            │      │ ├─ Order Service     │
│ ├─ Payments          │      │ ├─ Payment Service   │
│ └─ Shipping          │      │ └─ Shipping Service  │
└──────────────────────┘      └──────────────────────┘
```

### Facade Layer Responsibilities

1. **Request Routing**
   - Determine which system should handle the request
   - Apply routing rules based on feature flags, user segments, or traffic percentage
   - Transform requests if needed for compatibility

2. **Response Handling**
   - Aggregate responses from multiple services if needed
   - Transform responses to match expected format
   - Handle errors gracefully
   - Cache responses when appropriate

3. **Monitoring & Logging**
   - Log all routing decisions
   - Track response times and error rates
   - Monitor system health
   - Alert on anomalies

4. **Feature Flag Management**
   - Enable/disable features without redeployment
   - Control traffic routing dynamically
   - A/B testing support
   - Gradual rollout capabilities

### Implementation Technologies

#### API Gateway Solutions

**Kong**:
```yaml
# Kong Configuration Example
services:
  - name: product-service
    url: http://new-product-service:8080
    routes:
      - name: product-route
        paths:
          - /api/products
        plugins:
          - name: rate-limiting
            config:
              minute: 1000
          - name: request-transformer
            config:
              add:
                headers:
                  - X-Service-Version:v2
```

**AWS API Gateway**:
```json
{
  "type": "AWS::ApiGateway::Resource",
  "properties": {
    "RestApiId": {"Ref": "ApiGateway"},
    "ParentId": {"Ref": "RootResource"},
    "PathPart": "products"
  }
}
```

**NGINX**:
```nginx
upstream legacy_system {
    server legacy.example.com:8080;
}

upstream new_system {
    server new-service.example.com:8080;
}

server {
    listen 80;
    server_name api.example.com;

    location /api/products {
        # Route based on feature flag
        if ($feature_flag_enabled) {
            proxy_pass http://new_system;
        }
        proxy_pass http://legacy_system;
    }
}
```

#### Feature Flag Solutions

**LaunchDarkly**:
```javascript
const client = new LaunchDarkly.LDClient('sdk-key');

async function routeRequest(request, user) {
  const useNewSystem = await client.variation(
    'use-new-product-service',
    user,
    false
  );
  
  if (useNewSystem) {
    return callNewProductService(request);
  } else {
    return callLegacyProductService(request);
  }
}
```

**Custom Feature Flag Service**:
```javascript
class FeatureFlagService {
  constructor(configService) {
    this.config = configService;
  }

  isFeatureEnabled(featureName, context) {
    const feature = this.config.getFeature(featureName);
    
    if (!feature.enabled) return false;
    
    // Check user segment
    if (feature.userSegments && !feature.userSegments.includes(context.userId)) {
      return false;
    }
    
    // Check traffic percentage
    if (feature.trafficPercentage < 100) {
      const hash = this.hashUserId(context.userId);
      return (hash % 100) < feature.trafficPercentage;
    }
    
    return true;
  }

  hashUserId(userId) {
    // Consistent hashing for same user
    return userId.split('').reduce((acc, char) => {
      return ((acc << 5) - acc) + char.charCodeAt(0);
    }, 0);
  }
}
```

### Facade Layer Example Implementation

```javascript
// Facade Layer Implementation
class StranglerFacade {
  constructor(legacyService, newService, featureFlagService) {
    this.legacyService = legacyService;
    this.newService = newService;
    this.featureFlagService = featureFlagService;
    this.logger = new Logger('StranglerFacade');
  }

  async handleRequest(request, context) {
    const startTime = Date.now();
    
    try {
      // Determine which service to use
      const useNewService = this.featureFlagService.isFeatureEnabled(
        'use-new-product-service',
        context
      );

      let response;
      if (useNewService) {
        response = await this.callNewService(request);
        this.logger.info('Routed to new service', {
          path: request.path,
          duration: Date.now() - startTime
        });
      } else {
        response = await this.callLegacyService(request);
        this.logger.info('Routed to legacy service', {
          path: request.path,
          duration: Date.now() - startTime
        });
      }

      return response;
    } catch (error) {
      this.logger.error('Request failed', {
        error: error.message,
        path: request.path,
        duration: Date.now() - startTime
      });
      
      // Fallback to legacy system if new system fails
      return this.callLegacyService(request);
    }
  }

  async callNewService(request) {
    try {
      const response = await this.newService.handle(request);
      this.validateResponse(response);
      return response;
    } catch (error) {
      this.logger.warn('New service failed, falling back to legacy', {
        error: error.message
      });
      return this.callLegacyService(request);
    }
  }

  async callLegacyService(request) {
    return this.legacyService.handle(request);
  }

  validateResponse(response) {
    if (!response || !response.status) {
      throw new Error('Invalid response format');
    }
  }
}
```

---

## Challenges with Strangler Pattern 

### 1. Data Consistency Challenges

**Problem**: Maintaining data consistency between old and new systems during parallel execution.

**Scenarios**:
- User updates data in new system, but old system still serves stale data
- Dual-write failures cause data divergence
- Race conditions between systems

**Solutions**:

```javascript
// Dual-Write with Compensation
class DataSyncService {
  async updateUserProfile(userId, data) {
    const legacyResult = await this.legacyDB.update(userId, data);
    
    try {
      const newResult = await this.newDB.update(userId, data);
      return newResult;
    } catch (error) {
      // Compensate: rollback legacy update
      await this.legacyDB.rollback(userId, legacyResult.version);
      throw error;
    }
  }
}

// Event-Driven Synchronization
class EventSyncService {
  async publishDataChangeEvent(entity, change) {
    const event = {
      entityType: entity.type,
      entityId: entity.id,
      changeType: change.type,
      data: change.data,
      timestamp: Date.now()
    };
    
    // Both systems subscribe to this event
    await this.eventBus.publish('data-changed', event);
  }
}
```

**Best Practices**:
- Use event sourcing for immutable audit trail
- Implement eventual consistency patterns
- Use distributed transactions carefully
- Create reconciliation jobs to detect and fix inconsistencies
- Monitor data divergence metrics

### 2. Increased Operational Complexity

**Problem**: Running two systems simultaneously increases operational overhead.

**Challenges**:
- Monitoring two systems instead of one
- Debugging issues across system boundaries
- Managing deployments for both systems
- Training teams on both architectures

**Solutions**:

```yaml
# Unified Monitoring Setup
monitoring:
  metrics:
    - name: response_time_comparison
      query: |
        (new_system_response_time - legacy_system_response_time) / legacy_system_response_time
      alert_threshold: 20%  # Alert if new system is 20% slower
    
    - name: error_rate_comparison
      query: |
        new_system_error_rate - legacy_system_error_rate
      alert_threshold: 1%  # Alert if error rate increases by 1%
    
    - name: data_consistency
      query: |
        count(records_in_new_system != records_in_legacy_system)
      alert_threshold: 0  # Alert on any inconsistency

  dashboards:
    - name: strangler-migration-dashboard
      panels:
        - traffic_distribution
        - error_rates_comparison
        - latency_comparison
        - data_consistency_status
```

**Best Practices**:
- Implement comprehensive logging and tracing
- Use distributed tracing (Jaeger, Zipkin)
- Create unified dashboards for both systems
- Automate health checks and alerts
- Document runbooks for common issues

### 3. Performance Overhead

**Problem**: The facade layer adds latency and the dual-write pattern impacts performance.

**Impact**:
- Extra network hop through facade layer
- Dual-write operations take longer
- Increased database load
- Potential timeout issues

**Solutions**:

```javascript
// Caching Strategy
class CachingFacade {
  constructor(cache, legacyService, newService) {
    this.cache = cache;
    this.legacyService = legacyService;
    this.newService = newService;
  }

  async handleRequest(request) {
    const cacheKey = this.generateCacheKey(request);
    
    // Check cache first
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      return cached;
    }

    // Route to appropriate service
    const response = await this.routeRequest(request);
    
    // Cache the response
    await this.cache.set(cacheKey, response, {
      ttl: 300  // 5 minutes
    });

    return response;
  }

  generateCacheKey(request) {
    return `${request.method}:${request.path}:${JSON.stringify(request.query)}`;
  }
}

// Async Dual-Write
class AsyncDualWriteService {
  async updateData(data) {
    // Write to new system synchronously
    const newResult = await this.newService.update(data);
    
    // Write to legacy system asynchronously
    this.legacyService.update(data).catch(error => {
      this.logger.error('Async legacy update failed', error);
      // Trigger reconciliation job
      this.reconciliationQueue.enqueue({
        type: 'update',
        data: data,
        timestamp: Date.now()
      });
    });

    return newResult;
  }
}
```

**Best Practices**:
- Implement caching at multiple levels
- Use async operations where possible
- Optimize database queries
- Monitor and optimize facade layer performance
- Consider read replicas for heavy read operations

### 4. Testing Complexity

**Problem**: Testing becomes more complex with two systems running in parallel.

**Challenges**:
- Need to test both systems independently
- Need to test routing logic
- Need to test data consistency
- Need to test failure scenarios

**Solutions**:

```javascript
// Comprehensive Testing Strategy
describe('Strangler Facade', () => {
  describe('Routing Logic', () => {
    it('should route to new system when feature flag is enabled', async () => {
      const facade = new StranglerFacade(
        mockLegacyService,
        mockNewService,
        mockFeatureFlagService
      );

      mockFeatureFlagService.isFeatureEnabled.mockReturnValue(true);

      await facade.handleRequest(mockRequest, mockContext);

      expect(mockNewService.handle).toHaveBeenCalled();
      expect(mockLegacyService.handle).not.toHaveBeenCalled();
    });

    it('should fallback to legacy system when new system fails', async () => {
      mockNewService.handle.mockRejectedValue(new Error('Service unavailable'));

      await facade.handleRequest(mockRequest, mockContext);

      expect(mockLegacyService.handle).toHaveBeenCalled();
    });
  });

  describe('Data Consistency', () => {
    it('should detect data divergence between systems', async () => {
      const dataConsistencyChecker = new DataConsistencyChecker(
        legacyDB,
        newDB
      );

      const inconsistencies = await dataConsistencyChecker.findInconsistencies();

      expect(inconsistencies.length).toBe(0);
    });
  });

  describe('Performance', () => {
    it('should not exceed latency threshold', async () => {
      const startTime = Date.now();
      await facade.handleRequest(mockRequest, mockContext);
      const duration = Date.now() - startTime;

      expect(duration).toBeLessThan(100);  // 100ms threshold
    });
  });
});
```

**Best Practices**:
- Create integration tests for both systems
- Test routing logic thoroughly
- Implement chaos engineering tests
- Test failure scenarios and fallbacks
- Create load tests to verify performance

### 5. Team Coordination and Knowledge

**Problem**: Teams need to maintain expertise in both old and new systems.

**Challenges**:
- Knowledge silos between teams
- Difficulty in onboarding new team members
- Risk of losing expertise when team members leave
- Conflicting priorities between legacy and new system work

**Solutions**:

```markdown
# Team Structure for Strangler Migration

## Strangler Team (Cross-functional)
- Facade Layer Developers
- Data Synchronization Specialists
- DevOps/Infrastructure Engineers
- QA/Testing Engineers
- Product Manager

## Legacy System Team
- Maintains existing functionality
- Fixes bugs in legacy system
- Supports data migration
- Gradually reduces scope

## New System Team
- Develops new microservices
- Implements feature parity
- Optimizes performance
- Prepares for full migration

## Knowledge Sharing
- Weekly sync meetings
- Shared documentation
- Pair programming sessions
- Code reviews across teams
- Runbooks and playbooks
```

**Best Practices**:
- Create cross-functional teams
- Document all decisions and learnings
- Conduct regular knowledge-sharing sessions
- Implement pair programming
- Create comprehensive runbooks

### 6. Rollback Complexity

**Problem**: Rolling back changes becomes complex when data has been modified in both systems.

**Scenarios**:
- New system has a critical bug
- Data corruption in new system
- Performance issues in new system
- Need to revert to legacy system

**Solutions**:

```javascript
// Rollback Strategy
class RollbackManager {
  async initiateRollback(reason) {
    this.logger.warn('Initiating rollback', { reason });

    try {
      // Step 1: Stop routing to new system
      await this.featureFlagService.disableFeature('use-new-system');
      this.logger.info('Stopped routing to new system');

      // Step 2: Verify legacy system is healthy
      const legacyHealth = await this.legacyService.healthCheck();
      if (!legacyHealth.ok) {
        throw new Error('Legacy system is not healthy');
      }

      // Step 3: Reconcile data
      await this.dataReconciliationService.reconcile();
      this.logger.info('Data reconciliation complete');

      // Step 4: Verify data consistency
      const inconsistencies = await this.dataConsistencyChecker.check();
      if (inconsistencies.length > 0) {
        throw new Error(`Found ${inconsistencies.length} data inconsistencies`);
      }

      // Step 5: Monitor for issues
      await this.monitoringService.startIntensiveMonitoring(300000);  // 5 minutes

      this.logger.info('Rollback completed successfully');
    } catch (error) {
      this.logger.error('Rollback failed', error);
      // Alert on-call engineer
      await this.alertingService.alert('CRITICAL', 'Rollback failed: ' + error.message);
      throw error;
    }
  }

  async createBackupBeforeMigration() {
    // Create snapshots of both systems
    const legacySnapshot = await this.legacyService.createSnapshot();
    const newSnapshot = await this.newService.createSnapshot();

    return {
      timestamp: Date.now(),
      legacySnapshot,
      newSnapshot
    };
  }
}
```

**Best Practices**:
- Create backups before major migrations
- Test rollback procedures regularly
- Document rollback steps clearly
- Have a clear decision criteria for rollback
- Practice rollback scenarios in staging

### 7. Cost Implications

**Problem**: Running two systems simultaneously increases infrastructure costs.

**Costs**:
- Double database infrastructure
- Additional servers for new system
- Increased network bandwidth
- Additional monitoring and logging
- Extra development and operations effort

**Solutions**:

```yaml
# Cost Optimization Strategy
cost_management:
  infrastructure:
    - use_shared_databases_initially: true
    - implement_auto_scaling: true
    - use_spot_instances_for_new_system: true
    - decommission_legacy_gradually: true
  
  timeline:
    phase_1: "Months 1-3: 10% new system, 90% legacy (minimal cost increase)"
    phase_2: "Months 4-6: 50% new system, 50% legacy (peak cost)"
    phase_3: "Months 7-9: 90% new system, 10% legacy (cost decreasing)"
    phase_4: "Month 10+: 100% new system (cost reduced)"
  
  monitoring:
    - track_infrastructure_costs_daily
    - compare_cost_per_transaction
    - identify_optimization_opportunities
    - forecast_cost_savings
```

**Best Practices**:
- Plan migration timeline to minimize peak costs
- Use shared infrastructure where possible
- Implement aggressive auto-scaling
- Monitor costs continuously
- Calculate ROI of migration

---

## Summary

The Strangler Fig Pattern is a powerful strategy for modernizing legacy systems without the risk of a complete rewrite. By gradually replacing components while maintaining parallel execution, organizations can:

- **Reduce Risk**: Migrate incrementally with easy rollback options
- **Maintain Continuity**: Keep the system running during migration
- **Enable Innovation**: Deploy new features independently
- **Improve Quality**: Test thoroughly before full migration

However, success requires careful planning, robust monitoring, strong team coordination, and a clear understanding of the challenges involved. The pattern is most effective when:

1. Clear boundaries exist between system components
2. Teams have the expertise to manage parallel systems
3. Adequate monitoring and testing infrastructure is in place
4. Business stakeholders understand the timeline and costs
5. Data consistency strategies are well-defined

By addressing these challenges proactively, organizations can successfully modernize their systems while maintaining stability and delivering continuous value to users.


---


<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
