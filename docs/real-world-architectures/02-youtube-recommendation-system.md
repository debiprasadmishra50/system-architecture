# System Architecture: Youtube Recommendation System

## Table of Contents

1. [What is a Recommendation System](#what-is-a-recommendation-system)
2. [How Recommendations are Achieved](#how-recommendations-are-achieved)
3. [YouTube's Scaling Challenge](#youtubes-scaling-challenge)
4. [Three-Stage Recommendation Pipeline](#three-stage-recommendation-pipeline)
   - [Stage 1: Candidate Generation](#stage-1-candidate-generation)
   - [Stage 2: Ranking](#stage-2-ranking)
   - [Stage 3: Signal Inputs](#stage-3-signal-inputs)
5. [Signal Processing Pipeline](#signal-processing-pipeline)

---

## What is a Recommendation System

A recommendation system predicts and suggests content that users are most likely to engage with based on their behavior, preferences, and contextual information.

**Core Objectives:**
- Maximize user engagement and watch time
- Discover new content for users
- Reduce decision fatigue through personalization
- Improve content discoverability for creators

---

## How Recommendations are Achieved

Recommendation systems leverage multiple techniques working in concert:

**Machine Learning Approaches:**
- **Collaborative Filtering**: Identify users with similar behavior patterns and recommend content they liked
- **Content-Based Filtering**: Recommend similar content based on video metadata and features
- **Hybrid Models**: Combine collaborative and content-based approaches
- **Deep Learning**: Neural networks capture complex user-content relationships

**Key Techniques:**
- Matrix factorization for latent factor discovery
- Embedding-based similarity matching
- Real-time ranking with contextual features
- Multi-armed bandit algorithms for exploration vs. exploitation

---

## YouTube's Scaling Challenge

**The Problem:**
- 800+ million videos in catalog
- Billions of users with unique preferences
- Millisecond-level latency requirements
- Real-time personalization at scale

**Why Brute Force Fails:**
- Cannot score all 800M videos for every user in real-time
- Computational cost prohibitive
- Latency would exceed acceptable thresholds

**Solution: Two-Stage Filtering Pipeline**
```
800M Videos → Candidate Generation → Hundreds → Ranking → Top N Results
```

---

## Three-Stage Recommendation Pipeline

<img src='../../Resources/real-world-arch-resources/02-youtube-recommendation-system/Screenshot 2026-02-22 at 12.56.43 PM.png' width=700 />

### Stage 1: Candidate Generation

**Objective:** Reduce 800M videos to ~100-500 candidates in milliseconds

<img src='../../Resources/real-world-arch-resources/02-youtube-recommendation-system/Screenshot 2026-02-22 at 12.36.07 PM.png' width=800 />

**Two-Tower Neural Network Architecture:**

```
┌────────────────────────────────────────────────────────────┐
│                    Two-Tower Model                         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  User Tower                          Video Tower           │
│  ┌──────────────────┐               ┌──────────────────┐   │
│  │ User Features    │               │ Video Features   │   │
│  │ - Watch History  │               │ - Title/Tags     │   │
│  │ - Search Queries │               │ - Category       │   │
│  │ - Demographics   │               │ - Metadata       │   │
│  └────────┬─────────┘               └────────┬─────────┘   │
│           │                                  │             │
│  ┌────────▼─────────┐               ┌────────▼─────────┐   │
│  │ Dense Layers     │               │ Dense Layers     │   │
│  │ (ReLU, Dropout)  │               │ (ReLU, Dropout)  │   │
│  └────────┬─────────┘               └────────┬─────────┘   │
│           │                                  │             │
│  ┌────────▼─────────┐               ┌────────▼─────────┐   │
│  │ User Embedding   │               │ Video Embedding  │   │
│  │ (d-dimensional)  │               │ (d-dimensional)  │   │
│  └────────┬─────────┘               └────────┬─────────┘   │
│           │                                  │             │
│           └──────────────┬───────────────────┘             │
│                          │                                 │
│                  ┌───────▼────────┐                        │
│                  │ Dot Product    │                        │
│                  │ Similarity     │                        │
│                  └────────────────┘                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**How It Works:**
- User tower encodes user context into a dense embedding
- Video tower encodes video features into a dense embedding
- Dot product computes similarity score between user and video
- Top-K videos with highest similarity scores selected as candidates

**Advantages:**
- Efficient: O(1) lookup with pre-computed video embeddings
- Scalable: Embeddings can be indexed (e.g., using approximate nearest neighbor search)
- Real-time: Millisecond-level inference
- Captures collaborative patterns through learned embeddings

**Training:**
- Positive examples: Videos user watched/engaged with
- Negative examples: Random videos from corpus
- Loss function: Softmax cross-entropy or triplet loss

---

### Stage 2: Ranking

**Objective:** Score and order ~100-500 candidates to produce final top-N recommendations
<img src='../../Resources/real-world-arch-resources/02-youtube-recommendation-system/Screenshot 2026-02-22 at 12.40.34 PM.png' width=750 />

**Deep Ranking Model:**

```
┌──────────────────────────────────────────────────────┐
│              Deep Ranking Neural Network             │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Input Features                                      │
│  ├─ User Features (embeddings, demographics)         │
│  ├─ Video Features (embeddings, metadata)            │
│  ├─ Interaction Features (watch time, clicks)        │
│  ├─ Contextual Features (time, device, location)     │
│  └─ Real-time Signals (trending, freshness)          │
│           │                                          │
│  ┌────────▼──────────────────────────────────────┐   │
│  │ Feature Concatenation & Normalization         │   │
│  └────────┬──────────────────────────────────────┘   │
│           │                                          │
│  ┌────────▼──────────────────────────────────────┐   │
│  │ Hidden Layers (ReLU, Batch Norm, Dropout)     │   │
│  │ - Layer 1: 1024 units                         │   │
│  │ - Layer 2: 512 units                          │   │
│  │ - Layer 3: 256 units                          │   │
│  └────────┬──────────────────────────────────────┘   │
│           │                                          │
│  ┌────────▼──────────────────────────────────────┐   │
│  │ Output Layer (Sigmoid)                        │   │
│  │ Prediction: Watch Time / Click Probability    │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Ranking Objectives:**
- Predict expected watch time (primary metric)
- Predict click-through rate (CTR)
- Predict engagement (likes, shares, comments)
- Balance relevance with diversity

**Additional Ranking Parameters:**
- **Freshness**: Boost recently uploaded videos
- **Diversity**: Avoid recommending too many videos from same creator
- **Quality Signals**: Video ratings, viewer satisfaction
- **Creator Metrics**: Channel authority, upload frequency
- **Exploration**: Occasionally recommend less-certain videos
- **Business Rules**: Enforce content policies, regional restrictions

**Training Data:**
- Logged user interactions (watch, skip, like, dislike)
- Implicit feedback (watch duration, return visits)
- Explicit feedback (ratings, comments)
- Negative examples (videos user skipped quickly)

---

### Stage 3: Signal Inputs

<img src='../../Resources/real-world-arch-resources/02-youtube-recommendation-system/Screenshot 2026-02-22 at 12.47.37 PM.png' width=700 />

Recommendation quality depends on diverse, high-quality signals:

#### Explicit Signals: Conscious Actions

User deliberately provides preference information:

- **Likes/Dislikes**: Direct quality feedback
- **Subscriptions**: Channel preferences
- **Playlists**: Curated content collections
- **Search Queries**: Intent-driven content discovery
- **Comments**: Engagement and interest indicators
- **Ratings**: Explicit quality assessment

**Characteristics:**
- High confidence but sparse (users rarely rate)
- Directly expresses user intent
- Subject to bias (users rate extreme experiences)

#### Implicit Signals: Behavioral Patterns

Inferred from user actions without explicit feedback:

- **Watch History**: Content consumed
- **Watch Duration**: Engagement depth
- **Pause/Resume Patterns**: Content relevance
- **Repeat Viewing**: Content value
- **Click Patterns**: Interest in recommendations
- **Session Duration**: Overall engagement
- **Return Frequency**: Long-term interest

**Characteristics:**
- Abundant and continuous
- Noisy (watch duration ≠ satisfaction)
- Captures actual behavior vs. stated preferences
- Requires careful interpretation

#### Real-Time Events

Signals that change rapidly and require immediate incorporation:

- **Trending Videos**: Viral content gaining momentum
- **Breaking News**: Time-sensitive information
- **Live Events**: Concurrent user interest
- **Seasonal Content**: Temporal relevance
- **Creator Activity**: New uploads, live streams
- **Social Signals**: Shares, mentions, discussions

**Characteristics:**
- High velocity and short lifespan
- Critical for discovery and relevance
- Requires low-latency signal processing
- Often drives engagement spikes

#### Contextual Signals: The Moment

Environmental and temporal factors influencing recommendations:

- **Time of Day**: Morning vs. evening preferences
- **Day of Week**: Weekday vs. weekend behavior
- **Device Type**: Mobile vs. desktop consumption patterns
- **Network Quality**: Bandwidth-dependent content selection
- **Location**: Geographic and cultural preferences
- **Language**: User language preferences
- **Session Context**: What user was watching before
- **User State**: New user vs. long-time user

**Characteristics:**
- Highly personalized and dynamic
- Captures situational preferences
- Improves relevance through context awareness
- Enables adaptive recommendations

<img src='../../Resources/real-world-arch-resources/02-youtube-recommendation-system/Screenshot 2026-02-22 at 12.47.58 PM.png' width=800 />

---

## Signal Processing Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Signal Collection                        │
├─────────────────────────────────────────────────────────────┤
│  Explicit    │  Implicit    │  Real-Time   │  Contextual    │
│  Signals     │  Signals     │  Events      │  Signals       │
└────┬─────────┴──────┬───────┴──────┬───────┴────────┬───────┘
     │                │              │                │
     └────────────────┼──────────────┼────────────────┘
                      │
          ┌───────────▼────────────┐
          │ Feature Engineering    │
          │ - Normalization        │
          │ - Aggregation          │
          │ - Embedding            │
          └───────────┬────────────┘
                      │
          ┌───────────▼────────────┐
          │ Candidate Generation   │
          │ (Two-Tower Model)      │
          └───────────┬────────────┘
                      │
          ┌───────────▼────────────┐
          │ Ranking                │
          │ (Deep Ranking Model)   │
          └───────────┬────────────┘
                      │
          ┌───────────▼────────────┐
          │ Post-Processing        │
          │ - Diversity            │
          │ - Business Rules       │
          │ - Personalization      │
          └───────────┬────────────┘
                      │
          ┌───────────▼────────────┐
          │ Final Recommendations  │
          │ (Top-N Videos)         │
          └────────────────────────┘
```

---

## Key Takeaways

- **Two-stage approach** reduces computational complexity from O(n) to O(log n)
- **Candidate generation** uses efficient embedding-based retrieval
- **Ranking** applies sophisticated deep learning for fine-grained scoring
- **Multi-signal fusion** captures diverse aspects of user preferences
- **Real-time processing** enables dynamic, contextual recommendations
- **Scalability** achieved through staged filtering and efficient indexing
