# Chapter 5.3: Performance Modeling

[← Previous: Load Testing](02-load-testing.md) | [Back to README](../README.md) | [Next: Scaling Strategies →](04-scaling-strategies.md)

---

## Overview

Performance modeling uses mathematical approaches to predict system behavior under varying conditions without running every possible load test. Models help answer "what if" questions: What if traffic doubles? What if we add a caching layer? What if a dependency's latency increases? They complement empirical load testing with analytical insight.

---

## 1. Queueing Theory Fundamentals

```
  ┌──────────────────────────────────────────────────────────┐
  │  QUEUEING MODEL (M/M/1 - simplest)                      │
  │                                                          │
  │  Arrivals          Queue            Server               │
  │  (requests)        (waiting)        (processing)         │
  │                                                          │
  │  ──▶──▶──▶  ┌─┬─┬─┬─┬─┐  ──▶  ┌────────┐  ──▶ Done    │
  │  λ per sec  │ │ │ │ │ │       │ Server │                │
  │             └─┴─┴─┴─┴─┘       │ μ/sec  │                │
  │                                └────────┘                │
  │                                                          │
  │  Key variables:                                          │
  │  λ (lambda) = arrival rate (requests/second)             │
  │  μ (mu)     = service rate (requests/second per server)  │
  │  ρ (rho)    = utilization = λ / μ  (must be < 1)        │
  │                                                          │
  │  Key results (M/M/1):                                    │
  │  Average queue length: Lq = ρ² / (1 - ρ)                │
  │  Average wait time:   Wq = ρ / (μ × (1 - ρ))            │
  │  Average response:    W  = 1 / (μ - λ)                  │
  │                                                          │
  │  [!] As ρ approaches 1.0, wait time → infinity           │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. The Hockey Stick Effect

```
  ┌──────────────────────────────────────────────────────────┐
  │  Response Time vs Utilization (M/M/1 Queue)              │
  │                                                          │
  │  Response                          ╱                     │
  │  Time ▲                          ╱                       │
  │       │                        ╱                         │
  │  500ms│                      ╱   ← "Hockey stick"       │
  │       │                    ╱      latency explodes       │
  │       │                  ╱        above 70-80%           │
  │  200ms│               ╱                                  │
  │       │            ╱                                     │
  │  100ms│      ╱───╱                                       │
  │       │ ╱──╱                                             │
  │   50ms│╱                                                 │
  │       └──────────────────────────────────────▶           │
  │       0%  20%  40%  60%  70%  80%  90% 100%             │
  │                  Utilization (ρ)                          │
  │                                                          │
  │  ρ = 50% → W = 2 × service_time                         │
  │  ρ = 80% → W = 5 × service_time                         │
  │  ρ = 90% → W = 10 × service_time                        │
  │  ρ = 95% → W = 20 × service_time                        │
  │                                                          │
  │  Rule of thumb: Keep utilization below 70% for           │
  │  predictable latency                                     │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Little's Law

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │              L  =  λ  ×  W                               │
  │                                                          │
  │  L = Average number of requests in the system            │
  │  λ = Average arrival rate (requests/second)              │
  │  W = Average time a request spends in the system         │
  │                                                          │
  │  EXAMPLE:                                                │
  │  If λ = 100 req/s and W = 200ms (0.2s):                 │
  │  L = 100 × 0.2 = 20 concurrent requests                 │
  │                                                          │
  │  PRACTICAL USE:                                          │
  │  "How many threads/connections do we need?"              │
  │  threads_needed = throughput × average_latency           │
  │                                                          │
  │  If our payment service handles 500 RPS with             │
  │  100ms average latency:                                  │
  │  threads_needed = 500 × 0.1 = 50 threads                │
  │  With 2x safety buffer: 100 threads                      │
  │                                                          │
  │  If latency increases to 300ms (dependency slow):        │
  │  threads_needed = 500 × 0.3 = 150 threads               │
  │  → Thread pool exhaustion if sized at 100!               │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Amdahl's Law

```
  ┌──────────────────────────────────────────────────────────┐
  │  AMDAHL'S LAW: Limits of parallelization                 │
  │                                                          │
  │  Speedup = 1 / ((1 - P) + P/N)                          │
  │                                                          │
  │  P = Fraction of work that can be parallelized           │
  │  N = Number of parallel processors/instances             │
  │                                                          │
  │  Example: Request processing is 80% parallelizable       │
  │                                                          │
  │  N instances │ Speedup │ Throughput gain                  │
  │  ────────────┼─────────┼──────────────                   │
  │      1       │  1.0x   │  baseline                       │
  │      2       │  1.67x  │  +67%                           │
  │      4       │  2.5x   │  +150%                          │
  │      8       │  3.33x  │  +233%                          │
  │     16       │  4.0x   │  +300%                          │
  │    100       │  4.76x  │  +376%                          │
  │      ∞       │  5.0x   │  +400%  ← MAX (1/(1-0.8))      │
  │                                                          │
  │  Implication: No matter how many servers you add,        │
  │  the serial 20% limits your maximum throughput to        │
  │  5x. Optimize the serial path!                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Universal Scalability Law (USL)

```
  ┌──────────────────────────────────────────────────────────┐
  │  USL extends Amdahl's Law by adding coherence penalty    │
  │                                                          │
  │  C(N) = N / (1 + α(N-1) + β×N×(N-1))                    │
  │                                                          │
  │  N = Number of processors/instances                      │
  │  α = Contention (serialization) parameter                │
  │  β = Coherence (crosstalk/coordination) parameter        │
  │                                                          │
  │  Throughput                                               │
  │  ▲          ╱╲                                           │
  │  │        ╱    ╲  ← Throughput actually DECREASES        │
  │  │      ╱        ╲    after the peak (coherence cost)    │
  │  │    ╱                                                  │
  │  │  ╱  ╱── Linear (ideal)                                │
  │  │╱  ╱                                                   │
  │  │ ╱                                                     │
  │  └──────────────────────────────────▶ N (instances)      │
  │                                                          │
  │  Key insight: Adding too many instances can REDUCE       │
  │  throughput due to coordination overhead (locks,         │
  │  distributed consensus, cache invalidation).             │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Capacity Modeling Worksheet

```
  ┌──────────────────────────────────────────────────────────┐
  │  SERVICE: Order Processing API                           │
  │                                                          │
  │  Current state:                                          │
  │  ├─ Single instance throughput: 200 RPS (tested)         │
  │  ├─ Average latency: 50ms                                │
  │  ├─ Serial fraction: ~30% (DB writes, locks)             │
  │  ├─ Current instances: 4                                 │
  │  ├─ Current throughput: 650 RPS (not 800 — contention)   │
  │  └─ Current utilization: 65%                             │
  │                                                          │
  │  Question: How many instances for 2000 RPS?              │
  │                                                          │
  │  Using Amdahl's Law:                                     │
  │  Max speedup = 1 / (1-0.7) = 3.33x                      │
  │  Max throughput per instance = 200 × 3.33 = 666 RPS      │
  │  Wait — that's the ceiling, not per-instance!            │
  │                                                          │
  │  For 2000 RPS:                                           │
  │  Need to scale the serial path too:                      │
  │  ├─ Optimize DB writes (batching): serial → 15%          │
  │  ├─ New max speedup: 1 / (1-0.85) = 6.67x               │
  │  ├─ New max throughput: 200 × 6.67 = 1334 RPS per inst  │
  │  ├─ But USL coherence: ~70% efficiency at 10 nodes      │
  │  ├─ Effective: 10 × 200 × 0.7 = 1400 RPS                │
  │  ├─ Need 15 instances: 15 × 200 × 0.67 = 2010 RPS      │
  │  └─ With 30% buffer: 20 instances                        │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Using Little's Law to Size a Connection Pool

```
  Problem: Payment service intermittently times out
  
  Given:
  ├─ Peak throughput: 500 RPS
  ├─ Average DB query time: 20ms
  ├─ Connection pool size: 20
  ├─ DB calls per request: 3
  
  Applying Little's Law:
  L = λ × W
  L = (500 × 3) × 0.020 = 30 concurrent connections needed
  
  Problem found:
  Pool size is 20, but we need 30 concurrent connections!
  ├─ 10 requests waiting for connections at any time
  ├─ Wait time adds to latency
  ├─ Under spikes, queue grows and timeouts occur
  
  Solution:
  ├─ Increase pool to 50 (30 needed + buffer)
  ├─ Also: optimize queries to reduce avg time to 10ms
  ├─ New: L = 1500 × 0.010 = 15 (well within 50 pool)
  └─ Result: Timeouts eliminated, p99 latency cut by 60%
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Model predictions don't match real tests" | Models are approximations. Calibrate with load test data. Use models for direction, load tests for precision. |
| "Adding more servers doesn't improve throughput" | USL/Amdahl's: you've hit the serialization limit. Optimize the serial path (DB, locks, consensus). |
| "Thread pool is always exhausted" | Use Little's Law: threads = throughput × latency. If dependency latency spikes, threads consumed scales linearly. |
| "Can't estimate serial fraction" | Run load test with 1, 2, 4, 8 instances. Plot throughput. The curve shape reveals serialization and coherence parameters. |

---

## Summary Table

| Concept | Formula | Key Insight |
|---------|---------|-------------|
| **Utilization** | ρ = λ/μ | Keep below 70% for stable latency |
| **Little's Law** | L = λ × W | Relates concurrency, throughput, latency |
| **Amdahl's Law** | S = 1/((1-P)+P/N) | Serial fraction limits max speedup |
| **USL** | C(N) = N/(1+α(N-1)+βN(N-1)) | Too many nodes can reduce throughput |
| **Hockey Stick** | Latency vs utilization | Nonlinear — small util increase → huge latency jump |

---

## Quick Revision Questions

1. **State Little's Law and give a practical example.**
   > L = λ × W. If throughput is 1000 RPS and avg latency is 100ms, there are 100 concurrent requests in-flight. This determines thread pool and connection pool sizing.

2. **What is the hockey stick effect in queueing theory?**
   > As utilization approaches 100%, response time increases exponentially (not linearly). At 80% utilization, latency is 5x the service time. This is why we target <70% utilization.

3. **Explain Amdahl's Law and its implication for scaling.**
   > Speedup is limited by the serial (non-parallelizable) fraction. If 30% of work is serial, max speedup is 3.33x regardless of how many servers you add. Optimize the serial path.

4. **How does USL differ from Amdahl's Law?**
   > USL adds a "coherence" penalty representing coordination overhead between nodes. This means adding too many instances can actually decrease throughput, not just plateau.

5. **How would you size a database connection pool?**
   > Using Little's Law: connections_needed = RPS × DB_calls_per_request × avg_query_time. Add 50-100% buffer for spikes and variability.

---

[← Previous: Load Testing](02-load-testing.md) | [Back to README](../README.md) | [Next: Scaling Strategies →](04-scaling-strategies.md)
