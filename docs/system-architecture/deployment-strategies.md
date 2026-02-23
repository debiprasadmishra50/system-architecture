# Deployment Strategies: Techniques for Safe and Reliable Releases

## Table of Contents
1. [What is Deployment](#what-is-deployment)
2. [Rolling Deployment](#rolling-deployment)
    - [Overview](#overview)
    - [Benefits & Tradeoffs](#benefits--tradeoffs)
    - [Real-World Use Case](#real-world-use-case)
3. [Blue-Green Deployment](#blue-green-deployment)
    - [Overview](#overview-1)
    - [Benefits & Tradeoffs](#benefits--tradeoffs-1)
    - [Real-World Use Case](#real-world-use-case-1)
4. [Canary Deployment](#canary-deployment)
    - [Overview](#overview-2)
    - [Benefits & Tradeoffs](#benefits--tradeoffs-2)
    - [Real-World Use Case](#real-world-use-case-2)
5. [Feature Flag](#feature-flag)
    - [Overview](#overview-3)
    - [Benefits & Tradeoffs](#benefits--tradeoffs-3)
    - [Real-World Use Case](#real-world-use-case-3)
6. [A/B Testing Deployment](#ab-testing-deployment)
    - [Overview](#overview-4)
    - [Benefits & Tradeoffs](#benefits--tradeoffs-4)
    - [Real-World Use Case](#real-world-use-case-4)
7. [Comparison Summary](#comparison-summary)
    - [Decision Matrix](#decision-matrix)
8. [Hybrid Approaches](#hybrid-approaches)

---

## What is Deployment

**Deployment** is the process of releasing new versions of software to production environments. It involves:

- **Code Release**: Moving tested code from development to production
- **Minimal Downtime**: Ensuring users experience no or minimal service interruption
- **Risk Mitigation**: Reducing the impact of potential bugs or issues
- **Rollback Capability**: Ability to revert to previous versions if problems occur
- **Gradual Rollout**: Deploying to a subset of users before full release (optional)

<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 12.55.18 PM.png' width=400 />

**Key Goals**:
- Zero or near-zero downtime
- Quick rollback if issues arise
- Gradual validation of new features
- Minimal user impact

---

## Rolling Deployment
<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 12.57.08 PM.png' width=800 />

### Overview

Gradually replace old version instances with new version instances, one at a time or in small batches.

```
┌─────────────────────────────────────────────────────────────┐
│                    ROLLING DEPLOYMENT                       │
└─────────────────────────────────────────────────────────────┘

Time →

Initial State:
[V1] [V1] [V1] [V1]  (4 instances running v1)

Step 1:
[V2] [V1] [V1] [V1]  (1 instance upgraded)

Step 2:
[V2] [V2] [V1] [V1]  (2 instances upgraded)

Step 3:
[V2] [V2] [V2] [V1]  (3 instances upgraded)

Final State:
[V2] [V2] [V2] [V2]  (All instances running v2)

Load Balancer continuously routes traffic to healthy instances
```

### Benefits & Tradeoffs

| Aspect | Details |
|--------|---------|
| **Benefits** | • No downtime during deployment<br>• Gradual rollout reduces risk<br>• Easy rollback (keep old version running)<br>• Resource efficient (no duplicate infrastructure)<br>• Simple to implement |
| **Tradeoffs** | • Deployment takes longer (sequential updates)<br>• Mixed versions running simultaneously (compatibility issues)<br>• Database schema changes require careful planning<br>• Harder to test with multiple versions live<br>• Rollback requires re-deploying old version |

### Real-World Use Case

**Who Uses It**: Netflix, Uber, most microservices-based companies

**How It Works**:
- Netflix deploys updates to 1-2 instances at a time
- Monitors metrics (error rates, latency) for 5-10 minutes
- If healthy, continues rolling out to next batch
- If issues detected, stops and rolls back
- Entire deployment takes 30-60 minutes for large fleets

---

## Blue-Green Deployment
<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 12.58.42 PM.png' width=800 />

### Overview

Maintain two identical production environments (Blue and Green). Deploy to inactive environment, then switch traffic instantly.

```
┌─────────────────────────────────────────────────────────────┐
│                  BLUE-GREEN DEPLOYMENT                      │
└─────────────────────────────────────────────────────────────┘

BEFORE DEPLOYMENT:
┌──────────────────┐         ┌──────────────────┐
│   BLUE (Active)  │         │  GREEN (Idle)    │
│   [V1] [V1]      │         │  [V1] [V1]       │
└────────┬─────────┘         └──────────────────┘
         │
    ┌────▼──────┐
    │Load       │
    │Balancer   │
    └───────────┘
         │
    ┌────▼─────┐
    │ Users    │
    └──────────┘

DURING DEPLOYMENT:
┌──────────────────┐         ┌──────────────────┐
│   BLUE (Active)  │         │  GREEN (Deploy)  │
│   [V1] [V1]      │         │  [V2] [V2]       │
└────────┬─────────┘         └──────────────────┘
         │
    ┌────▼──────┐
    │Load       │
    │Balancer   │
    └───────────┘
         │
    ┌────▼─────┐
    │ Users    │
    └──────────┘

AFTER DEPLOYMENT (Switch):
┌──────────────────┐         ┌──────────────────┐
│   BLUE (Idle)    │         │  GREEN (Active)  │
│   [V1] [V1]      │         │  [V2] [V2]       │
└──────────────────┘         └────────┬─────────┘
                                      │
                                 ┌────▼──────┐
                                 │Load       │
                                 │Balancer   │
                                 └───────────┘
                                      │
                                 ┌────▼─────┐
                                 │ Users    │
                                 └──────────┘
```

### Benefits & Tradeoffs

| Aspect | Details |
|--------|---------|
| **Benefits** | • Instant traffic switch (zero downtime)<br>• Easy rollback (switch back to Blue)<br>• Full environment testing before switch<br>• No version compatibility issues<br>• Supports database migrations |
| **Tradeoffs** | • Requires 2x infrastructure (expensive)<br>• Data synchronization between environments<br>• Stateful data must be shared/synced<br>• Higher operational complexity<br>• Larger resource footprint |

### Real-World Use Case

**Who Uses It**: AWS, GitHub, Shopify, large enterprises with high availability requirements

**How It Works**:
- GitHub maintains two identical production clusters
- New code deployed to inactive cluster
- Automated tests run against new cluster
- Once validated, DNS/load balancer switches traffic
- Old cluster kept running for 24 hours for quick rollback
- Entire process takes 15-30 minutes

---

## Canary Deployment
<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 1.01.11 PM.png' width=800 />

### Overview

Deploy new version to a small subset of users (canaries), monitor metrics, then gradually increase traffic percentage.

```
┌─────────────────────────────────────────────────────────────┐
│                   CANARY DEPLOYMENT                         │
└─────────────────────────────────────────────────────────────┘

Stage 1 (5% traffic):
┌─────────────────────────────────────────────────────────────┐
│ [V2] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] │
│  5%                          95%                            │
└─────────────────────────────────────────────────────────────┘

Stage 2 (25% traffic):
┌─────────────────────────────────────────────────────────────┐
│ [V2] [V2] [V2] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] [V1] │
│  25%                         75%                            │
└─────────────────────────────────────────────────────────────┘

Stage 3 (50% traffic):
┌─────────────────────────────────────────────────────────────┐
│ [V2] [V2] [V2] [V2] [V2] [V2] [V1] [V1] [V1] [V1] [V1] [V1] │
│  50%                         50%                            │
└─────────────────────────────────────────────────────────────┘

Stage 4 (100% traffic):
┌─────────────────────────────────────────────────────────────┐
│ [V2] [V2] [V2] [V2] [V2] [V2] [V2] [V2] [V2] [V2] [V2] [V2] │
│  100%                                                       │
└─────────────────────────────────────────────────────────────┘

Monitoring at each stage:
• Error rates
• Latency
• Resource usage
• User complaints
```

### Benefits & Tradeoffs

| Aspect | Details |
|--------|---------|
| **Benefits** | • Catches bugs early with minimal user impact<br>• Real-world validation before full rollout<br>• Gradual traffic increase reduces risk<br>• Easy rollback (reduce traffic to old version)<br>• Metrics-driven decision making |
| **Tradeoffs** | • Requires sophisticated monitoring/alerting<br>• Longer deployment time (hours to days)<br>• Complex traffic routing logic needed<br>• Multiple versions running simultaneously<br>• Requires feature flag or traffic splitting capability |

### Real-World Use Case

**Who Uses It**: Google, Facebook, Uber, companies with millions of users

**How It Works**:
- Google deploys new search algorithm to 1% of users
- Monitors click-through rates, dwell time, bounce rate
- If metrics improve, increases to 5%, then 10%, then 50%, then 100%
- If metrics degrade, immediately rolls back to 0%
- Entire process takes 1-2 weeks for major features
- Automated alerts trigger rollback if error rate exceeds threshold

---

## Feature Flag
<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 1.01.57 PM.png' width=800 />

### Overview

Deploy code with new features disabled by default. Enable features for specific users/groups without redeployment.

```
┌─────────────────────────────────────────────────────────────┐
│                    FEATURE FLAG DEPLOYMENT                  │
└─────────────────────────────────────────────────────────────┘

Code Deployment (All instances):
┌──────────────────────────────────────────────────────────────┐
│ [V2 with Feature X disabled] [V2 with Feature X disabled]    │
│ [V2 with Feature X disabled] [V2 with Feature X disabled]    │
└──────────────────────────────────────────────────────────────┘

Feature Flag Configuration (Dynamic):
┌──────────────────────────────────────────────────────────────┐
│ Feature X:                                                   │
│   - Enabled for: Internal users, Beta testers (10%)          │
│   - Disabled for: Regular users (90%)                        │
└──────────────────────────────────────────────────────────────┘

Runtime Decision:
┌──────────────────────────────────────────────────────────────┐
│ if (featureFlag.isEnabled('feature_x', userId)) {            │
│   // Show new feature                                        │
│ } else {                                                     │
│   // Show old feature                                        │
│ }                                                            │
└──────────────────────────────────────────────────────────────┘

Gradual Rollout (No redeployment):
Time 1: 10% enabled  → 90% disabled
Time 2: 25% enabled  → 75% disabled
Time 3: 50% enabled  → 50% disabled
Time 4: 100% enabled → 0% disabled
```

### Benefits & Tradeoffs

| Aspect | Details |
|--------|---------|
| **Benefits** | • No redeployment needed for rollout<br>• Instant enable/disable without restart<br>• Decouples deployment from feature release<br>• Easy A/B testing and experimentation<br>• Supports gradual rollout and rollback<br>• Works with any deployment strategy |
| **Tradeoffs** | • Code complexity (conditional logic everywhere)<br>• Feature flag management overhead<br>• Dead code accumulation (old flags)<br>• Testing complexity (multiple code paths)<br>• Requires external flag management system |

### Real-World Use Case

**Who Uses It**: Slack, Airbnb, Stripe, most modern SaaS companies

**How It Works**:
- Slack deploys new UI component with flag disabled
- Enables for 5% of users via LaunchDarkly (flag service)
- Monitors user feedback and metrics
- Gradually increases percentage: 10% → 25% → 50% → 100%
- If issues found, disables instantly without redeployment
- Old code path removed after 2-3 weeks of 100% rollout

---

## A/B Testing Deployment
<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 1.02.53 PM.png' width=800 />

<img src='../../Resources/20-deployment-strategies/Screenshot 2026-02-23 at 1.03.21 PM.png' width=800 />

### Overview

Deploy two versions simultaneously and route users randomly to measure which performs better based on business metrics.

```
┌─────────────────────────────────────────────────────────────┐
│                  A/B TESTING DEPLOYMENT                     │
└─────────────────────────────────────────────────────────────┘

Deployment:
┌──────────────────────────────────────────────────────────────┐
│ Version A (Control)    │    Version B (Variant)              │
│ [Old Algorithm]        │    [New Algorithm]                  │
│ [V1] [V1] [V1]         │    [V2] [V2] [V2]                   │
└──────────────────────────────────────────────────────────────┘

Traffic Split (50/50):
┌──────────────────────────────────────────────────────────────┐
│ User 1 → Hash(user_id) → Version A                           │
│ User 2 → Hash(user_id) → Version B                           │
│ User 3 → Hash(user_id) → Version A                           │
│ User 4 → Hash(user_id) → Version B                           │
│ ...                                                          │
│ 50% users see Version A  |  50% users see Version B          │
└──────────────────────────────────────────────────────────────┘

Metrics Collection:
┌──────────────────────────────────────────────────────────────┐
│ Version A:                 │ Version B:                      │
│ • Conversion: 3.2%         │ • Conversion: 3.8%              │
│ • Avg Order: $45           │ • Avg Order: $52                │
│ • Bounce Rate: 35%         │ • Bounce Rate: 28%              │
│ • Revenue: $1.2M           │ • Revenue: $1.4M                │
└──────────────────────────────────────────────────────────────┘

Decision:
Version B wins → Promote to 100% (or keep A if no improvement)
```

### Benefits & Tradeoffs

| Aspect | Details |
|--------|---------|
| **Benefits** | • Data-driven decisions (statistical validation)<br>• Measures business impact, not just technical metrics<br>• Identifies winning variant before full rollout<br>• Supports experimentation culture<br>• Can run multiple A/B tests simultaneously |
| **Tradeoffs** | • Requires statistical significance (time/traffic)<br>• Both versions must be maintained<br>• User experience inconsistency (different versions)<br>• Requires analytics infrastructure<br>• Longer time to decision (need sufficient data) |

### Real-World Use Case

**Who Uses It**: Amazon, Netflix, Airbnb, e-commerce and SaaS companies

**How It Works**:
- Amazon tests new checkout flow (Version B) vs current (Version A)
- Routes 50% of traffic to each version
- Collects metrics: conversion rate, cart abandonment, revenue
- Runs test for 2 weeks to achieve statistical significance
- If Version B shows 2% higher conversion, promotes to 100%
- If no improvement, keeps Version A and tries different variant
- Runs 100+ A/B tests simultaneously across platform

---

## Comparison Summary

| Strategy | Downtime | Speed | Risk | Infrastructure | Complexity | Best For |
|----------|----------|-------|------|-----------------|------------|----------|
| **Rolling** | None | Slow (30-60 min) | Medium | 1x | Low | Gradual, safe rollouts |
| **Blue-Green** | None | Fast (5-15 min) | Low | 2x | Medium | Critical systems, instant rollback |
| **Canary** | None | Medium (hours-days) | Very Low | 1.2x | High | Large user bases, risk-averse |
| **Feature Flag** | None | Instant | Very Low | 1x | Medium | Decoupled releases, experimentation |
| **A/B Testing** | None | Slow (days-weeks) | Low | 1.5x | High | Business optimization, data-driven |

### Decision Matrix

**Use Rolling Deployment when**:
- Resources are limited
- Deployment frequency is low
- Rollback is acceptable
- Database changes are minimal

**Use Blue-Green Deployment when**:
- Zero downtime is critical
- Instant rollback is required
- Infrastructure budget allows 2x resources
- Database migrations are needed

**Use Canary Deployment when**:
- User base is large
- Risk tolerance is low
- Monitoring infrastructure exists
- Gradual validation is preferred

**Use Feature Flags when**:
- Decoupling deployment from release is needed
- Experimentation is frequent
- Instant rollback is required
- Multiple features deploy simultaneously

**Use A/B Testing when**:
- Business metrics matter more than technical metrics
- Data-driven decisions are required
- Experimentation culture exists
- Statistical validation is needed

---

## Hybrid Approaches

Most organizations combine strategies:

- **Rolling + Feature Flags**: Deploy code with flags disabled, gradually enable
- **Canary + A/B Testing**: Canary to 5%, then A/B test variants at 50/50
- **Blue-Green + Feature Flags**: Switch to Green, enable features gradually
- **Rolling + Canary**: Rolling deployment with canary monitoring at each stage

