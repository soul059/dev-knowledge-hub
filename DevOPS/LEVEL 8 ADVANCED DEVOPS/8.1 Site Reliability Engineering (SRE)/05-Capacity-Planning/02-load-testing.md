# Chapter 5.2: Load Testing

[← Previous: Capacity Forecasting](01-capacity-forecasting.md) | [Back to README](../README.md) | [Next: Performance Modeling →](03-performance-modeling.md)

---

## Overview

Load testing validates that a system can handle expected (and beyond-expected) traffic volumes while maintaining SLO targets. It is essential for capacity planning, performance tuning, and discovering bottlenecks before they cause production incidents. Without load testing, capacity planning is guesswork.

---

## 1. Types of Performance Testing

```
  ┌──────────────────────────────────────────────────────────┐
  │  TYPE            │ PURPOSE                │ DURATION      │
  ├──────────────────┼────────────────────────┼───────────────┤
  │ Smoke Test       │ Verify basic function  │ Minutes       │
  │                  │ under minimal load     │               │
  │                  │                        │               │
  │ Load Test        │ Expected production    │ 30-60 min     │
  │                  │ traffic levels         │               │
  │                  │                        │               │
  │ Stress Test      │ Beyond expected load   │ 30-60 min     │
  │                  │ until breaking point   │               │
  │                  │                        │               │
  │ Spike Test       │ Sudden traffic burst   │ 10-15 min     │
  │                  │ (e.g., viral content)  │               │
  │                  │                        │               │
  │ Soak/Endurance   │ Sustained load over    │ 4-24 hours    │
  │ Test             │ extended period (find  │               │
  │                  │ memory leaks, conn     │               │
  │                  │ pool exhaustion)       │               │
  │                  │                        │               │
  │ Breakpoint Test  │ Gradually increase     │ Until failure │
  │                  │ until system fails     │               │
  │                  │ (find max capacity)    │               │
  └──────────────────┴────────────────────────┴───────────────┘
```

---

## 2. Load Test Profiles

```
  ┌──────────────────────────────────────────────────────────────┐
  │   LOAD TEST PROFILE:                                        │
  │                                                              │
  │   RPS                                                        │
  │   ▲       ┌─────────────────────┐                            │
  │   │       │   Steady state      │                            │
  │ 10K│ ramp /│   (measure here)    │\  ramp down               │
  │   │    / │                     │ \                           │
  │   │   /  │                     │  \                          │
  │   │  /   │                     │   \                         │
  │   │ /    │                     │    \                        │
  │   └──────┴─────────────────────┴─────▶ Time                  │
  │   5min   │      30 min         │5min                         │
  │          │                     │                             │
  │ STRESS TEST PROFILE:                                        │
  │                                                              │
  │   RPS                    ╱╱╱ breaking                        │
  │   ▲              ╱╱╱╱╱          point                        │
  │   │        ╱╱╱╱╱                                             │
  │ 20K│  ╱╱╱╱╱                                                  │
  │   │╱╱                                                        │
  │ 10K│                                                         │
  │   └──────────────────────────────────▶ Time                  │
  │   Gradually increase until errors > 1%                      │
  │                                                              │
  │ SPIKE TEST PROFILE:                                         │
  │                                                              │
  │   RPS    │ spike │                                           │
  │   ▲      │       │                                           │
  │ 50K│     │       │                                           │
  │   │      │       │                                           │
  │ 10K│─────┘       └──────                                     │
  │   └──────────────────────▶ Time                              │
  │   Normal → 5x spike → return to normal                      │
  └──────────────────────────────────────────────────────────────┘
```

---

## 3. Load Testing Tools

```
  ┌──────────────────────────────────────────────────────────┐
  │  TOOL            │ LANGUAGE │ KEY FEATURES               │
  ├──────────────────┼──────────┼────────────────────────────┤
  │ k6 (Grafana)     │ JS       │ Developer-friendly, CI/CD │
  │                  │          │ integration, cloud run     │
  │                  │          │                            │
  │ Locust           │ Python   │ Distributed, real-time UI │
  │                  │          │ code-based scenarios       │
  │                  │          │                            │
  │ Gatling          │ Scala    │ High performance, detailed │
  │                  │          │ HTML reports               │
  │                  │          │                            │
  │ JMeter           │ Java     │ GUI, extensive protocols   │
  │                  │          │ (JDBC, LDAP, FTP)          │
  │                  │          │                            │
  │ wrk / vegeta     │ Go       │ CLI, extremely fast,      │
  │                  │          │ simple HTTP testing        │
  │                  │          │                            │
  │ Artillery        │ Node.js  │ YAML config, cloud-native │
  └──────────────────┴──────────┴────────────────────────────┘
```

---

## 4. k6 Load Test Example

```javascript
// load-test.js — k6 script for API load testing
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const latencyTrend = new Trend('response_time');

// Test configuration
export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 VUs
    { duration: '5m', target: 100 },   // Stay at 100 VUs
    { duration: '2m', target: 500 },   // Ramp to 500 VUs
    { duration: '5m', target: 500 },   // Stay at 500 VUs
    { duration: '2m', target: 1000 },  // Ramp to 1000 VUs
    { duration: '5m', target: 1000 },  // Stay at 1000 VUs
    { duration: '3m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],  // SLO targets
    errors: ['rate<0.01'],                            // <1% error rate
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  // Simulate real user behavior
  const endpoints = [
    { url: 'https://api.example.com/products', weight: 50 },
    { url: 'https://api.example.com/cart', weight: 30 },
    { url: 'https://api.example.com/checkout', weight: 20 },
  ];

  const endpoint = weightedRandom(endpoints);
  const res = http.get(endpoint.url, {
    headers: { 'Authorization': 'Bearer test-token' },
    tags: { endpoint: endpoint.url },
  });

  // Validate response
  const success = check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  errorRate.add(!success);
  latencyTrend.add(res.timings.duration);

  sleep(Math.random() * 3 + 1); // 1-4s think time
}
```

---

## 5. What to Measure During Load Tests

```
  ┌──────────────────────────────────────────────────────────┐
  │  METRIC             │ WHY                │ TARGET         │
  ├─────────────────────┼────────────────────┼────────────────┤
  │ Throughput (RPS)     │ System capacity    │ > expected peak│
  │ Latency p50          │ Typical experience │ < 200ms        │
  │ Latency p95          │ Slow requests      │ < 500ms        │
  │ Latency p99          │ Tail latency       │ < 1000ms       │
  │ Error rate           │ Reliability        │ < 0.1%         │
  │ CPU utilization      │ Compute headroom   │ < 70%          │
  │ Memory utilization   │ Resource headroom  │ < 80%          │
  │ DB connections       │ Pool exhaustion    │ < 80%          │
  │ DB query latency     │ Backend bottleneck │ < 50ms avg     │
  │ Network I/O          │ Bandwidth limits   │ < 70% of limit │
  │ GC pause time        │ JVM/Go pauses      │ < 100ms p99    │
  │ Queue depth          │ Backpressure       │ Not growing    │
  └─────────────────────┴────────────────────┴────────────────┘
```

---

## 6. Load Testing Best Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │  DO:                                                     │
  │  ✓ Test in an environment matching production            │
  │  ✓ Use realistic traffic patterns (not just one endpoint)│
  │  ✓ Include think time between requests (simulates users) │
  │  ✓ Monitor system metrics alongside test results         │
  │  ✓ Run soak tests to find memory leaks and connection    │
  │    pool exhaustion                                       │
  │  ✓ Store results and compare across runs                 │
  │  ✓ Integrate into CI/CD for regression detection         │
  │                                                          │
  │  DON'T:                                                  │
  │  ✗ Test from the same machine as the system under test   │
  │  ✗ Test with cached/warm data only (cold start matters)  │
  │  ✗ Ignore database state (empty DB ≠ production DB)     │
  │  ✗ Run against production without traffic controls       │
  │  ✗ Use synthetic data that doesn't match real patterns   │
  │  ✗ Skip recording baseline before making changes         │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Discovering a Connection Pool Bottleneck

```
  Load Test Results:
  
  RPS ▲
      │                        ┌── Error rate spikes
      │              ┌─────────┤   above 500 RPS
  800 │         ╱────┘         │
      │    ╱───╱               │
  500 │───╱                    │ ← p99 latency doubles
      │╱                       │
      └────────────────────────┴──────▶ VUs
      0   100   200   300   400   500
  
  Observation at 500 VUs:
  ├─ RPS plateaus at 800 (doesn't increase with more VUs)
  ├─ p99 latency jumps from 200ms to 4000ms
  ├─ Error rate increases from 0.1% to 15%
  ├─ CPU: 35% (not the bottleneck)
  ├─ Memory: 50% (not the bottleneck)
  ├─ DB connection pool: 100/100 (BOTTLENECK!)
  
  Root cause: Connection pool maxed at 100 connections.
  Application threads waiting for available connections.
  
  Fix:
  ├─ Increased connection pool to 200
  ├─ Added connection pooler (PgBouncer)
  ├─ Optimized slow queries holding connections too long
  └─ Re-test: 1500 RPS with <200ms p99 latency
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Load test results differ from production" | Match environment: same instance types, data size, network topology. |
| "Can't generate enough traffic" | Use distributed load generation (k6 cloud, distributed Locust). |
| "Latency looks good but users complain" | Check client-side metrics. Load tests often don't include DNS, TLS handshake, rendering. |
| "Results aren't reproducible" | Control variables: same time, same data, same infrastructure. Document everything. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Load Test** | Expected traffic levels, sustained |
| **Stress Test** | Beyond expected, find breaking point |
| **Spike Test** | Sudden traffic burst |
| **Soak Test** | Extended duration, find leaks |
| **Virtual Users (VUs)** | Simulated concurrent users |
| **Think Time** | Pause between requests (realism) |
| **Breakpoint** | Maximum capacity before SLO violation |

---

## Quick Revision Questions

1. **What's the difference between a load test and a stress test?**
   > Load test validates performance at expected traffic levels. Stress test pushes beyond expected to find the breaking point.

2. **Why are soak tests important?**
   > They reveal problems that only appear over time: memory leaks, connection pool exhaustion, disk space growth, log file accumulation.

3. **Name five metrics you should monitor during a load test.**
   > Throughput (RPS), latency (p50/p95/p99), error rate, CPU utilization, and database connection pool usage.

4. **Why is think time important in load tests?**
   > Real users pause between actions (reading, typing). Without think time, tests simulate unrealistic constant-request bots, overestimating required capacity.

5. **What is the ideal CPU utilization target during peak load testing?**
   > Below 70% sustained. This leaves headroom for traffic spikes, failure absorption (N+1), and prevents CPU throttling.

---

[← Previous: Capacity Forecasting](01-capacity-forecasting.md) | [Back to README](../README.md) | [Next: Performance Modeling →](03-performance-modeling.md)
