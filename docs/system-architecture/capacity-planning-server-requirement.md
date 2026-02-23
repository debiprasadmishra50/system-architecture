# Web Server Framework Performance Guide

## Table of Contents

- [Key Concepts](#key-concepts)
  - [Little's Law](#littles-law)
  - [Single Core Throughput (I/O-bound, rule of thumb)](#single-core-throughput-io-bound-rule-of-thumb)
- [System Requirements Formula](#system-requirements-formula)
  - [Input: req/s + latency → find hardware](#input-reqs--latency--find-hardware)
  - [Input: concurrent requests + latency → find hardware](#input-concurrent-requests--latency--find-hardware)
- [Framework Comparison](#framework-comparison)
- [Full Framework Performance List (Descending)](#full-framework-performance-list-descending)
- [Performance Tiers](#performance-tiers)
- [Architecture Recommendations](#architecture-recommendations)
- [Capacity Planning: 100,000 concurrent requests, p99 = 250ms](#capacity-planning-100000-concurrent-requests-p99--250ms)
- [Capacity Planning: 100,000 requests/sec, p99 = 250ms](#capacity-planning-100000-requestssec-p99--250ms)
- [Server Capacity Requirements](#server-capacity-requirements)
  - [Scenario 1: 100K Requests/Second](#scenario-1-100k-requestssecond)
  - [Scenario 2: 100K Concurrent Requests/Second](#scenario-2-100k-concurrent-requestssecond)

## Key Concepts

### Little's Law
Used to derive system requirements from two known inputs.

```
Concurrent = Throughput × Latency

Throughput = Concurrent / Latency
```

### Single Core Throughput (I/O-bound, rule of thumb)
- Each framework has a different req/s per core ceiling
- All numbers below assume **I/O-bound workloads** (DB calls, network, etc.)
- CPU-bound workloads will be significantly lower

---

## System Requirements Formula

    Considering a NodeJS Express Server with 2000 req/s per core throughput 

### Input: req/s + latency → find hardware

| Step | Formula | Example (10,000 req/s, p99=250ms) |
|---|---|---|
| 1. Concurrent in-flight | `Throughput × Latency` | 10,000 × 0.25 = **2,500** |
| 2. Cores (raw) | `Throughput ÷ req/s per core` | 10,000 ÷ 2,000 = **5** |
| 3. Cores (with headroom) | `Raw cores × 1.5` | 5 × 1.5 = **~8** |
| 4. RAM | `Concurrent × ~2MB` | 2,500 × 2MB = **~8GB** |

### Input: concurrent requests + latency → find hardware

| Step | Formula | Example (10,000 concurrent, p99=250ms) |
|---|---|---|
| 1. Throughput needed | `Concurrent / Latency` | 10,000 / 0.25 = **40,000 req/s** |
| 2. Cores (raw) | `Throughput ÷ req/s per core` | 40,000 ÷ 2,000 = **20** |
| 3. Cores (with headroom) | `Raw cores × 1.5` | 20 × 1.5 = **~30** |
| 4. RAM | `Concurrent × ~2MB` | 10,000 × 2MB = **~32GB** |

### So:

    10,000 people @ 1 req/s → 8-core machine

    10,000 concurrent requests → ~20-core machine or 3x 8-core machine

---

## Framework Comparison
### Input: 10,000 concurrent requests, p99=250ms → Throughput needed: 40,000 req/s

| Framework | Language | req/s per core | Cores (raw) | Cores (1.5x) | RAM |
|---|---|---|---|---|---|
| Actix Web | Rust | ~12,000 | 4 | **~6** | 16GB |
| Fiber | Go | ~6,000 | 7 | **~11** | 16GB |
| Gin | Go | ~5,500 | 8 | **~12** | 16GB |
| Fastify | Node.js | ~4,500 | 9 | **~14** | 16GB |
| ASP.NET Core | C# | ~4,000 | 10 | **~15** | 24GB |
| Spring WebFlux | Java | ~3,000 | 14 | **~21** | 32GB |
| Spring Boot MVC | Java | ~2,500 | 16 | **~24** | 48GB |
| Express.js | Node.js | ~2,000 | 20 | **~30** | 32GB |
| NestJS | Node.js | ~1,500 | 27 | **~40** | 32GB |
| FastAPI | Python | ~700 | 58 | **~87** | 32GB |

---

## Full Framework Performance List (Descending)

| Rank | Framework | Language | req/s per core |
|---|---|---|---|
| 1 | Actix Web | Rust | ~12,000 |
| 2 | Hyper | Rust | ~11,000 |
| 3 | Warp | Rust | ~10,500 |
| 4 | Axum | Rust | ~10,000 |
| 5 | Drogon | C++ | ~9,500 |
| 6 | Crow | C++ | ~9,000 |
| 7 | Oat++ | C++ | ~8,500 |
| 8 | Vertx | Java | ~7,000 |
| 9 | Netty | Java | ~6,500 |
| 10 | Fiber | Go | ~6,000 |
| 11 | Gin | Go | ~5,500 |
| 12 | Echo | Go | ~5,200 |
| 13 | Chi | Go | ~5,000 |
| 14 | Fastify | Node.js | ~4,500 |
| 15 | Rocket | Rust | ~4,000 |
| 16 | ASP.NET Core (Minimal API) | C# | ~4,000 |
| 17 | ASP.NET Core (Controllers) | C# | ~3,500 |
| 18 | Ktor | Kotlin | ~3,200 |
| 19 | Spring WebFlux | Java | ~3,000 |
| 20 | Hono | Bun/Node | ~2,800 |
| 21 | Elysia | Bun | ~2,800 |
| 22 | uWSGI | Python | ~2,700 |
| 23 | Spring Boot (MVC) | Java | ~2,500 |
| 24 | Quarkus | Java | ~2,500 |
| 25 | Micronaut | Java | ~2,300 |
| 26 | Play Framework | Scala | ~2,200 |
| 27 | Helidon | Java | ~2,000 |
| 28 | Express.js | Node.js | ~2,000 |
| 29 | Koa | Node.js | ~1,900 |
| 30 | Hapi | Node.js | ~1,700 |
| 31 | NestJS | Node.js | ~1,500 |
| 32 | Sinatra | Ruby | ~1,200 |
| 33 | Litestar | Python | ~1,100 |
| 34 | FastAPI | Python | ~700 |
| 35 | Django (async) | Python | ~600 |
| 36 | Flask | Python | ~500 |
| 37 | Rails (Puma) | Ruby | ~500 |
| 38 | Laravel | PHP | ~400 |
| 39 | Symfony | PHP | ~350 |
| 40 | Django (sync) | Python | ~300 |
| 41 | CodeIgniter | PHP | ~300 |
| 42 | CakePHP | PHP | ~250 |

---

## Performance Tiers

| Tier | Frameworks | req/s per core | Why |
|---|---|---|---|
| 🥇 Rust/C++ | Actix, Hyper, Axum, Drogon | 8,000–12,000 | No GC, zero-cost abstractions, bare metal |
| 🥈 Go | Fiber, Gin, Echo, Chi | 5,000–6,000 | Goroutines, lightweight runtime |
| 🥉 Java/C# Reactive | Vertx, Netty, ASP.NET Core | 3,000–7,000 | Reactive pipelines, JVM warmed up |
| 4 | Node.js (Fastify/Hono) | 2,800–4,500 | Optimized event loop |
| 5 | Java/Kotlin (blocking) | 2,000–2,500 | Thread pool, GC pressure |
| 6 | Node.js (Express/Nest) | 1,500–2,000 | Middleware overhead |
| 7 | Python | 300–1,100 | GIL is the ceiling |
| 8 | PHP/Ruby | 250–500 | Legacy threading models |

---

## Architecture Recommendations

### When vertical scaling isn't enough → Go horizontal

```
CDN → Load Balancer → N × (multi-core server) → Redis Cache → DB
```

### Levers to pull before adding hardware

- **Caching (Redis)** — cache hits never reach your server
- **CDN** — static/cacheable responses offloaded entirely
- **Reduce latency** — lower p99 = fewer concurrent in-flight = fewer cores needed
- **Switch framework** — Express → Fastify alone gives 2x throughput per core

---

## Capacity Planning: 100,000 concurrent requests, p99 = 250ms
```
Concurrent Req = 100000
Latency = 250ms = 0.25s

Throughput = 100000 / 0.25 = 400,000 req/s

NodeJS I/O bound throughput = 2000 req/s per core

Cores(RAW) needed = 400,000 / 2000 = 200 cores

Cores(With Overhead) = 200 * 1.5 = 300 cores

RAM needed = 100,000 concurrent * 2MB = 200GB
```

    So You would need a single machine with 300 cores and 200 GB RAM Minimum
    OR
    You would need 38 8-core machines each with 6GB RAM to handle the load


## Capacity Planning: 100,000 requests/sec, p99 = 250ms
```
Throughput = 100,000 req/s
Latency = 250ms = 0.25s

Concurrent Req = 100,000 * 0.25 = 25,000

NodeJS I/O bound throughput = 2000 req/s per core

Cores(RAW) needed = 25000 / 2000 = 12.5 cores

Cores(With Overhead) = 12.5 * 1.5 = 18.75 cores ~ 19 cores

RAM needed = 25000 concurrent * 2MB = 50GB

```

    So You would need a single machine with 19 cores and 50 GB RAM Minimum
    OR
    You would need 3 8-core machines each with 8GB RAM to handle the load

---

|  Throughput (req/s)   |   Concurrent Req (req/sec) (in flight)  | Cores(Raw) | Cores(Actual) | RAM(GB) | Machines (nos)
| :---: | :---: | :---: | :---: | :---: | :---: |
|  100000   |   25000  | 12.5 | 19 | 50 | 3x 8-core 8GB RAM |
|  400000   |  100000   | 200 | 300 | 200 | 38x 8-core 6GB RAM |

---


## Server Capacity Requirements

### Scenario 1: 100K Requests/Second
| Metric | Value |
|:---|:---:|
| **Throughput** | 100,000 req/s |
| **Concurrent Requests** | 25,000 |
| **CPU Cores (Raw)** | 12.5 |
| **CPU Cores (Actual)** | 19 |
| **RAM** | 50 GB |
| **Infrastructure** | 3 × 8-core, 8 GB RAM |

### Scenario 2: 100K Concurrent Requests/Second
| Metric | Value |
|:---|:---:|
| **Throughput** | 400,000 req/s |
| **Concurrent Requests** | 100,000 |
| **CPU Cores (Raw)** | 200 |
| **CPU Cores (Actual)** | 300 |
| **RAM** | 200 GB |
| **Infrastructure** | 38 × 8-core, 6 GB RAM |

---


*All req/s per core figures are approximations under realistic I/O-bound conditions.*

*Benchmark your specific workload for accurate capacity planning.*
