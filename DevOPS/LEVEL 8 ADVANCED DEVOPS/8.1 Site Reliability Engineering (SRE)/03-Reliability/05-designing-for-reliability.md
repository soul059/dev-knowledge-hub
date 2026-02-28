# Chapter 3.5: Designing for Reliability

[← Previous: Failure Modes](04-failure-modes.md) | [Back to README](../README.md) | [Next: Redundancy and Replication →](06-redundancy-and-replication.md)

---

## Overview

Designing for reliability means making deliberate architectural choices that allow systems to continue operating — even when individual components fail. Rather than trying to prevent all failures (impossible), SRE-focused design embraces failure as normal and builds systems that are resilient by default.

---

## 1. Core Reliability Design Principles

```
  ┌──────────────────────────────────────────────────────────┐
  │         RELIABILITY DESIGN PRINCIPLES                    │
  │                                                          │
  │  1. Design for failure    → Assume everything fails      │
  │  2. Simplicity            → Simple systems fail less     │
  │  3. Redundancy            → No single points of failure  │
  │  4. Isolation              → Failures don't spread       │
  │  5. Graceful degradation  → Serve partial over nothing   │
  │  6. Defense in depth      → Multiple layers of safety    │
  │  7. Observability         → See what's happening inside  │
  │  8. Automation            → Reduce human error surface   │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Architecture Patterns for Reliability

```
  ┌──────────────────────────────────────────────────────────────┐
  │  PATTERN          │ PURPOSE            │ TRADEOFF            │
  ├───────────────────┼────────────────────┼─────────────────────┤
  │ Load balancing    │ Distribute traffic │ Added complexity    │
  │ Circuit breaker   │ Prevent cascading  │ Potential false     │
  │                   │ failures           │ open states         │
  │ Bulkhead          │ Isolate failures   │ Resource under-     │
  │                   │ to compartments    │ utilization         │
  │ Retry + backoff   │ Handle transient   │ Can amplify load    │
  │                   │ errors             │ without backoff     │
  │ Timeout           │ Prevent indefinite │ May cut off slow    │
  │                   │ blocking           │ but valid requests  │
  │ Queue-based       │ Buffer load spikes │ Added latency,      │
  │ decoupling        │                    │ ordering concerns   │
  │ Idempotency       │ Safe retries       │ Implementation      │
  │                   │                    │ complexity          │
  │ Rate limiting     │ Protect from       │ May reject valid    │
  │                   │ overload           │ traffic             │
  └───────────────────┴────────────────────┴─────────────────────┘
```

---

## 3. Circuit Breaker Pattern

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │               CIRCUIT BREAKER STATES                     │
  │                                                          │
  │         success                   timeout                │
  │    ┌──────────────┐                                      │
  │    │              ▼                                      │
  │  ┌──────┐     failure count    ┌──────┐                  │
  │  │CLOSED│──── > threshold ────▶│ OPEN │                  │
  │  │      │                      │      │                  │
  │  └──────┘                      └──┬───┘                  │
  │    ▲                              │                      │
  │    │ success                      │ timer expires        │
  │    │                              ▼                      │
  │    │                         ┌─────────┐                 │
  │    └─────────────────────────│HALF-OPEN│                 │
  │                    failure   │ (test 1 │──── failure ─┐  │
  │                              │ request)│              │  │
  │                              └─────────┘              │  │
  │                                                  back to │
  │                                                 OPEN     │
  └──────────────────────────────────────────────────────────┘
```

### Implementation Example (Go)

```go
type CircuitBreaker struct {
    maxFailures    int
    resetTimeout   time.Duration
    failures       int
    state          string // "closed", "open", "half-open"
    lastFailureAt  time.Time
    mu             sync.Mutex
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    
    if cb.state == "open" {
        if time.Since(cb.lastFailureAt) > cb.resetTimeout {
            cb.state = "half-open"
        } else {
            cb.mu.Unlock()
            return ErrCircuitOpen
        }
    }
    cb.mu.Unlock()
    
    err := fn()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailureAt = time.Now()
        if cb.failures >= cb.maxFailures {
            cb.state = "open"
        }
        return err
    }
    
    cb.failures = 0
    cb.state = "closed"
    return nil
}
```

---

## 4. Bulkhead Pattern

```
  WITHOUT BULKHEAD:
  ┌──────────────────────────────────────────┐
  │  Shared Thread Pool (200 threads)        │
  │                                          │
  │  API endpoint A ─┐                       │
  │  API endpoint B ──┼── All share same ────│──▶ If B is slow,
  │  API endpoint C ─┘   thread pool         │    A and C also
  │                                          │    starve
  └──────────────────────────────────────────┘
  
  WITH BULKHEAD:
  ┌──────────────────────────────────────────┐
  │  Endpoint A: Thread Pool (80 threads)    │──▶ A is isolated
  ├──────────────────────────────────────────┤
  │  Endpoint B: Thread Pool (80 threads)    │──▶ B slow? Only
  │                                          │    B is affected
  ├──────────────────────────────────────────┤
  │  Endpoint C: Thread Pool (40 threads)    │──▶ C keeps working
  └──────────────────────────────────────────┘
```

---

## 5. Graceful Degradation Design

```
  ┌──────────────────────────────────────────────────────────┐
  │         SERVICE DEGRADATION LEVELS                       │
  │                                                          │
  │  Level 0: FULL SERVICE                                   │
  │  ─────────────────────────────────                       │
  │  All features, real-time data, full personalization      │
  │                                                          │
  │  Level 1: REDUCED FEATURES                               │
  │  ─────────────────────────────                           │
  │  Disable recommendations, serve cached search results    │
  │  ("We're experiencing delays — some features limited")   │
  │                                                          │
  │  Level 2: STATIC FALLBACK                                │
  │  ────────────────────────                                │
  │  Serve cached pages, disable writes, read-only mode      │
  │  ("Service is in read-only mode")                        │
  │                                                          │
  │  Level 3: MAINTENANCE PAGE                               │
  │  ───────────────────────                                 │
  │  Static page served from CDN                             │
  │  ("We'll be back shortly")                               │
  │                                                          │
  │  [!] Always prefer partial service over complete outage  │
  └──────────────────────────────────────────────────────────┘
```

### Degradation Configuration Example

```yaml
degradation_levels:
  level_0: # Normal
    features:
      recommendations: true
      real_time_search: true
      personalization: true
      notifications: true
    
  level_1: # Reduced
    trigger: "error_budget < 50% OR latency_p99 > 2s"
    features:
      recommendations: false        # Disable ML inference
      real_time_search: false        # Use cached results
      personalization: false         # Serve generic content
      notifications: true
    
  level_2: # Minimal
    trigger: "error_budget < 10% OR availability < 99.5%"
    features:
      recommendations: false
      real_time_search: false
      personalization: false
      notifications: false
      writes: false                 # Read-only mode
    
  level_3: # Emergency
    trigger: "manual_trigger OR availability < 95%"
    action: "serve_static_page_from_cdn"
```

---

## 6. Designing for Idempotency

```
  ┌──────────────────────────────────────────────────────────┐
  │  WHY IDEMPOTENCY MATTERS                                 │
  │                                                          │
  │  Client ──── Request ────▶ Server                        │
  │         ◄── Timeout ────         (did it succeed?)       │
  │         ──── Retry ─────▶        (safe to retry?)        │
  │                                                          │
  │  HTTP Methods:                                           │
  │  ├─ GET:    Naturally idempotent  ✓                      │
  │  ├─ PUT:    Naturally idempotent  ✓                      │
  │  ├─ DELETE: Naturally idempotent  ✓                      │
  │  └─ POST:   NOT idempotent       ✗ (needs design)       │
  │                                                          │
  │  Making POST idempotent:                                 │
  │  ┌──────────────────────────────────────────┐            │
  │  │ Client generates idempotency_key         │            │
  │  │ Server checks: "seen this key before?"   │            │
  │  │  ├─ YES → return cached result           │            │
  │  │  └─ NO  → process, store key + result    │            │
  │  └──────────────────────────────────────────┘            │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Retry Strategy Design

```
  ┌──────────────────────────────────────────────────────────┐
  │  EXPONENTIAL BACKOFF WITH JITTER                         │
  │                                                          │
  │  Delay                                                   │
  │  ▲                                                       │
  │  │         ╱ Without jitter (coordinated retries)        │
  │  │       ╱                                               │
  │  │     ╱   ⬡ ⬡ ⬡  With jitter (spread out)             │
  │  │   ╱  ⬡                                               │
  │  │  ╱ ⬡                                                  │
  │  │╱⬡                                                     │
  │  └──────────────────────────────────▶ Attempt            │
  │  1    2     3      4       5                             │
  │                                                          │
  │  Formula:                                                │
  │  delay = min(base × 2^attempt + random(0, jitter), max) │
  │                                                          │
  │  Example: base=100ms, jitter=100ms, max=30s              │
  │  Attempt 1: 100-200ms                                    │
  │  Attempt 2: 200-300ms                                    │
  │  Attempt 3: 400-500ms                                    │
  │  Attempt 4: 800-900ms                                    │
  │  Attempt 5: 1600-1700ms                                  │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Designing a Reliable Payment System

```
  ┌─────────┐    ┌─────────────┐    ┌──────────────┐
  │ Client  │───▶│  API GW     │───▶│  Payment     │
  │         │    │ • rate limit │    │  Service     │
  │ idempot.│    │ • auth      │    │ • circuit brk│
  │ key in  │    │ • timeout:5s│    │ • idempotent │
  │ header  │    └─────────────┘    │ • retry 3x   │
  └─────────┘                       └──────┬───────┘
                                           │
                            ┌──────────────┼──────────────┐
                            ▼              ▼              ▼
                      ┌──────────┐  ┌──────────┐  ┌──────────┐
                      │ Primary  │  │ Message  │  │ Payment  │
                      │ DB       │  │ Queue    │  │ Gateway  │
                      │ (write)  │  │ (async)  │  │ (Stripe) │
                      │ + replica│  │ DLQ      │  │ timeout  │
                      └──────────┘  └──────────┘  └──────────┘
  
  Reliability features:
  ├─ Idempotency: Client sends unique key, prevents double-charge
  ├─ Circuit breaker: On Stripe (opens after 5 failures in 30s)
  ├─ Retry with backoff: 3 attempts with exponential backoff
  ├─ Dead letter queue: Failed payments sent to DLQ for review
  ├─ DB replication: Primary with synchronous replica
  ├─ Rate limiting: 100 req/s per user at API gateway
  └─ Timeout: 5s at gateway, 10s at payment service
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Circuit breaker opens too frequently" | Increase failure threshold or window. Ensure health check endpoints aren't counted. |
| "Retries are overwhelming downstream services" | Add jitter to backoff. Set retry budget (max 10% of total traffic). |
| "Graceful degradation not kicking in" | Automate degradation triggers. Don't rely on human decisions during incidents. |
| "Bulkhead pools are hard to size" | Start at equal split, then adjust based on 30 days of traffic patterns. |

---

## Summary Table

| Pattern | Purpose | Key Config |
|---------|---------|------------|
| **Circuit Breaker** | Stop cascading failures | Threshold, timeout, half-open test |
| **Bulkhead** | Isolate failure domains | Thread pool sizes per endpoint |
| **Graceful Degradation** | Partial service > no service | Feature flags, degradation levels |
| **Idempotency** | Safe retries | Client-generated unique keys |
| **Retry + Backoff** | Handle transient errors | Base delay, multiplier, jitter, max |
| **Rate Limiting** | Prevent overload | Requests/second per client |
| **Timeout** | Prevent thread blocking | Set at each service boundary |

---

## Quick Revision Questions

1. **What are the three states of a circuit breaker?**
   > Closed (normal, requests pass through), Open (requests fail fast), Half-Open (test one request to check recovery).

2. **How does the bulkhead pattern prevent cascading failures?**
   > It isolates each service/endpoint with its own resource pool (thread pool, connection pool). If one pool is exhausted, others continue operating normally.

3. **What is graceful degradation and why is it preferred over complete failure?**
   > Progressively disabling non-critical features to maintain core functionality. Users get partial service, which is always better than a complete outage.

4. **Why is jitter important in retry strategies?**
   > Without jitter, all clients retry at the same time (thundering herd), amplifying the load. Jitter spreads retries randomly across time.

5. **How do you make a POST request idempotent?**
   > Client generates a unique idempotency key and sends it with each request. Server checks if this key was already processed; if yes, returns the cached result without re-processing.

6. **What is the exponential backoff formula?**
   > delay = min(base × 2^attempt + random(0, jitter), max_delay). Example: 100ms, 200ms, 400ms, 800ms... up to max 30s.

---

[← Previous: Failure Modes](04-failure-modes.md) | [Back to README](../README.md) | [Next: Redundancy and Replication →](06-redundancy-and-replication.md)
