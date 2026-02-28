# Chapter 3.4: Failure Modes

[← Previous: Risk Assessment](03-risk-assessment.md) | [Back to README](../README.md) | [Next: Designing for Reliability →](05-designing-for-reliability.md)

---

## Overview

Understanding failure modes is critical for building resilient systems. In SRE, we study how systems fail — not just that they fail — to design effective mitigations. Every component, dependency, and interaction introduces potential failure modes that must be systematically identified and addressed.

---

## 1. Failure Mode Classification

```
  ┌────────────────────────────────────────────────────────────┐
  │                FAILURE MODE TAXONOMY                       │
  │                                                            │
  │  By Scope                    By Behaviour                  │
  │  ─────────                   ──────────────                │
  │  ├─ Component failure        ├─ Fail-stop      (clean)    │
  │  ├─ Service failure          ├─ Fail-slow      (degraded) │
  │  ├─ Dependency failure       ├─ Fail-partial   (subset)   │
  │  ├─ Regional failure         ├─ Fail-silent    (no alert) │
  │  └─ Global failure           └─ Byzantine      (random)   │
  │                                                            │
  │  By Cause                    By Duration                   │
  │  ────────                    ───────────                   │
  │  ├─ Hardware                 ├─ Transient      (seconds)  │
  │  ├─ Software                 ├─ Intermittent   (sporadic) │
  │  ├─ Configuration            └─ Permanent      (until fix)│
  │  ├─ Capacity                                               │
  │  ├─ Network                                                │
  │  └─ Human error                                            │
  └────────────────────────────────────────────────────────────┘
```

---

## 2. Common Failure Modes in Distributed Systems

```
  ┌──────────────────────────────────────────────────────────────┐
  │  FAILURE MODE          │ DESCRIPTION          │ EXAMPLE      │
  ├────────────────────────┼──────────────────────┼──────────────┤
  │ Cascading failure      │ One component fails, │ DB slow →    │
  │                        │ overloads others     │ queue fills → │
  │                        │                      │ API timeout  │
  │                        │                      │              │
  │ Thundering herd        │ Many clients retry   │ Cache miss → │
  │                        │ simultaneously       │ all hit DB   │
  │                        │                      │              │
  │ Split brain            │ Network partition    │ Two primaries│
  │                        │ creates inconsistency│ in DB cluster│
  │                        │                      │              │
  │ Resource exhaustion    │ Memory, CPU, disk,   │ OOM kill of  │
  │                        │ connections depleted │ Java process │
  │                        │                      │              │
  │ Poison pill / bad data │ Malformed input      │ One bad msg  │
  │                        │ crashes processing   │ blocks queue │
  │                        │                      │              │
  │ Gray failure           │ Partially working    │ Server at    │
  │                        │ but degraded         │ 50% packet   │
  │                        │                      │ loss         │
  │                        │                      │              │
  │ Metastable failure     │ System can't recover │ High CPU →   │
  │                        │ even after cause     │ longer GC →  │
  │                        │ is removed           │ more queuing │
  └────────────────────────┴──────────────────────┴──────────────┘
```

---

## 3. Cascading Failure Deep Dive

```
  Normal State:
  ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ Service │───▶│ Service │───▶│   DB    │
  │   A     │    │   B     │    │         │
  │  50 rps │    │  50 rps │    │  50 qps │
  └─────────┘    └─────────┘    └─────────┘
  
  Cascade begins:
  ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ Service │───▶│ Service │───▶│   DB    │
  │   A     │    │   B     │    │  ████   │ ← DB slow
  │ 200 rps │    │ 200 rps │    │  overload│   (disk full)
  │  retries│    │ blocked │    │         │
  │  pile up│    │ threads │    │         │
  └─────────┘    └─────────┘    └─────────┘
       ▲              │
       │   timeout     │ thread pool
       │   → retry     │ exhausted
       │              ▼
  ┌───────────────────────┐
  │   FULL OUTAGE         │
  │   All services down   │
  └───────────────────────┘
  
  Prevention:
  ┌─────────┐  circuit  ┌─────────┐  connection ┌─────────┐
  │ Service │──breaker──│ Service │───pool──────│   DB    │
  │   A     │  timeout  │   B     │  limit      │         │
  │ backoff │  bulkhead │ shed    │  timeout    │         │
  └─────────┘           └─────────┘             └─────────┘
```

---

## 4. Failure Mode and Effects Analysis (FMEA)

```yaml
# FMEA Template for a Microservices System
fmea:
  system: "E-Commerce Checkout Service"
  date: "2024-01-15"
  
  analyses:
    - component: "Payment Gateway Connection"
      failure_mode: "Connection timeout"
      cause: "Gateway latency spike or network partition"
      effect_local: "Payment requests queue up"
      effect_system: "Checkout fails, lost revenue"
      severity: 5      # 1-5
      occurrence: 3    # 1-5  
      detection: 2     # 1-5 (1=easy, 5=hard to detect)
      rpn: 30          # Risk Priority Number = S × O × D
      mitigation:
        - "Circuit breaker with 10s timeout"
        - "Fallback to secondary gateway"
        - "Async payment retry queue"
      residual_rpn: 6  # After mitigation (5 × 1 × 1.2 ≈ 6)

    - component: "Inventory Database"
      failure_mode: "Primary node crash"
      cause: "Disk failure or OOM"
      effect_local: "Stock checks fail"
      effect_system: "Overselling or blocked checkout"
      severity: 4
      occurrence: 2
      detection: 1
      rpn: 8
      mitigation:
        - "Automated failover to replica (< 30s)"
        - "Read from replica for stock checks"
      residual_rpn: 2
```

### Risk Priority Number (RPN)

```
  RPN = Severity × Occurrence × Detection
  
  ┌─────────────────────────────────────────────┐
  │ RPN Range │ Priority   │ Action             │
  ├───────────┼────────────┼────────────────────┤
  │  1 - 20   │ Low        │ Monitor            │
  │ 21 - 50   │ Medium     │ Plan mitigation    │
  │ 51 - 80   │ High       │ Prioritize fix     │
  │ 81 - 125  │ Critical   │ Immediate action   │
  └───────────┴────────────┴────────────────────┘
```

---

## 5. Gray Failures

```
  ┌──────────────────────────────────────────────────────────┐
  │ GRAY FAILURE: Partial or degraded state that evades      │
  │ traditional binary health checks                        │
  │                                                          │
  │ Examples:                                                │
  │  ├─ NIC dropping 5% of packets (but "up")               │
  │  ├─ Disk with intermittent I/O errors                   │
  │  ├─ CPU thermal throttling under load                   │
  │  └─ DNS resolution succeeding but slowly                │
  │                                                          │
  │ Why dangerous:                                           │
  │  ├─ Health checks say "OK"                               │
  │  ├─ Metrics may look normal in averages                  │
  │  ├─ Affects subset of requests → hard to reproduce       │
  │  └─ Can persist for days/weeks unnoticed                 │
  │                                                          │
  │ Detection:                                               │
  │  ├─ Monitor p99/p999 latency, not just averages          │
  │  ├─ Differential analysis across peers                   │
  │  ├─ Canary/synthetic probes from multiple vantage points│
  │  └─ Track per-instance error rates                       │
  └──────────────────────────────────────────────────────────┘
```

### Prometheus Query to Detect Outlier Instances

```promql
# Find instances with error rate 3x higher than the fleet average
(
  sum(rate(http_requests_total{status=~"5.."}[5m])) by (instance)
  /
  sum(rate(http_requests_total[5m])) by (instance)
)
>
3 * (
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
)
```

---

## 6. Metastable Failures

```
  Normal ──────────── Overload ──────────── Metastable
  (stable)            (trigger)             (stuck)
  
  ┌──────────┐      ┌──────────┐       ┌──────────┐
  │ CPU: 40% │      │ CPU: 95% │       │ CPU: 90% │
  │ Queue: 0 │ ──►  │ Queue: 1K│  ──►  │ Queue: 5K│
  │ Latency: │  ↑   │ Latency: │   ↑   │ Latency: │
  │  50ms    │  │   │  2000ms  │   │   │  5000ms  │
  └──────────┘  │   └──────────┘   │   └──────────┘
                │                   │
           Traffic spike       Cause removed,
           (temporary)         but system can't
                               drain the queue
                               fast enough
  
  Key characteristic: Removing the trigger does NOT
  restore the system — manual intervention required
  
  Solutions:
  ├─ Bounded queues (drop excess work)
  ├─ Admission control (reject early)
  ├─ Load shedding (prioritize critical requests)
  └─ Circuit breakers with recovery cooldown
```

---

## 7. Real-World Scenario

### [SCENARIO] Cascading Failure in Microservices

```
  Architecture:
  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
  │ Mobile │──▶│  API   │──▶│ Order  │──▶│Payment │
  │  App   │   │Gateway │   │Service │   │Service │
  └────────┘   └────────┘   └────────┘   └────────┘
                                              │
                                              ▼
                                         ┌────────┐
                                         │ Stripe │
                                         │  API   │
                                         └────────┘
  
  Timeline:
  14:00 - Stripe API latency spikes from 200ms to 15s
  14:01 - Payment Service threads blocked waiting for Stripe
  14:02 - Payment Service connection pool exhausted (200/200)
  14:03 - Order Service calls to Payment start timing out (30s)
  14:05 - Order Service thread pool exhausted
  14:07 - API Gateway upstream timeout → 504s to all users
  14:10 - Mobile app retries create 3x normal traffic
  14:15 - Full outage declared
  
  Post-incident mitigations applied:
  ├─ Payment Service: circuit breaker (5s timeout, 50% threshold)
  ├─ Order Service: bulkhead pattern (separate thread pools)
  ├─ API Gateway: rate limiting + request shedding
  ├─ Mobile App: exponential backoff with jitter
  └─ Result: Stripe outage now causes graceful degradation
       (payments queued, browsing unaffected)
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Can't distinguish gray failure from normal variance" | Compare per-instance metrics against fleet aggregates. Use percentile ratios (p99/p50). |
| "Cascading failures happen too fast to react" | Automate circuit breakers and load shedding. Human response time (minutes) is too slow. |
| "Hard to reproduce intermittent failures" | Implement per-request tracing with baggage. Log correlation IDs for forensic analysis. |
| "System enters metastable state after incidents" | Add bounded queues and admission control. Design explicit recovery procedures. |

---

## Summary Table

| Failure Mode | Key Characteristic | Primary Mitigation |
|-------------|-------------------|-------------------|
| **Cascading** | Failure propagates across services | Circuit breakers, timeouts, bulkheads |
| **Thundering herd** | Simultaneous retries amplify load | Jittered backoff, cache stampede locks |
| **Split brain** | Network partition → inconsistency | Consensus protocols (Raft/Paxos) |
| **Gray failure** | Partial failure evades health checks | Percentile monitoring, peer comparison |
| **Metastable** | Can't self-recover after trigger removed | Bounded queues, load shedding |
| **Poison pill** | One bad input blocks processing | Dead letter queues, input validation |

---

## Quick Revision Questions

1. **What is the difference between a fail-stop and a gray failure?**
   > Fail-stop: component completely stops and is easily detected. Gray failure: component partially works, evading health checks while degrading service quality.

2. **Explain cascading failure with an example.**
   > Service A depends on Service B which depends on DB. DB slows down → B's threads block → A's retries overwhelm B → entire chain fails. Prevention: circuit breakers, timeouts, bulkheads.

3. **What is a metastable failure and why is it dangerous?**
   > A state where the system remains degraded even after the initial trigger is removed. Dangerous because it requires manual intervention — the system cannot self-heal.

4. **Calculate the RPN: Severity=4, Occurrence=3, Detection=4.**
   > RPN = 4 × 3 × 4 = 48 (Medium priority — plan mitigation).

5. **How do you detect gray failures that health checks miss?**
   > Monitor p99/p999 latency, compare per-instance metrics against fleet averages, use synthetic probes from multiple vantage points, track differential error rates.

6. **What is the thundering herd problem and how do you prevent it?**
   > When many clients simultaneously retry or request the same resource (e.g., after cache expiry). Prevent with: jittered exponential backoff, cache stampede protection (locking), request coalescing.

---

[← Previous: Risk Assessment](03-risk-assessment.md) | [Back to README](../README.md) | [Next: Designing for Reliability →](05-designing-for-reliability.md)
