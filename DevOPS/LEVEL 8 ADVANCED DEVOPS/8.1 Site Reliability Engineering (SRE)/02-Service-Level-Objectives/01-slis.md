# Chapter 2.1: SLIs (Service Level Indicators)

[← Previous: SRE Team Structure](../01-SRE-Fundamentals/05-sre-team-structure.md) | [Back to README](../README.md) | [Next: SLOs →](02-slos.md)

---

## Overview

Service Level Indicators (SLIs) are carefully defined quantitative measures of some aspect of the level of service that is provided. They are the raw measurements that tell you how your system is actually performing from the user's perspective.

> **SLIs answer: "How is my service performing right now?"**

---

## 1. What is an SLI?

An SLI is a **ratio** of good events to total events, expressed as a percentage:

```
              Good Events
  SLI  =  ──────────────── × 100%
             Total Events

  Example:
    Successful requests:  9,970
    Total requests:      10,000
    
    SLI = 9,970 / 10,000 × 100% = 99.7%
```

### Why a Ratio?

```
  ┌─────────────────────────────────────────────────────┐
  │  A ratio (0-100%) is:                               │
  │                                                     │
  │  ✓ Easy to understand ("99.9% of requests succeed") │
  │  ✓ Directly comparable to SLOs                      │
  │  ✓ Independent of traffic volume                    │
  │  ✓ Naturally bounded (0% to 100%)                   │
  │                                                     │
  │  vs. raw count ("5 errors") which doesn't tell you  │
  │  if 5 out of 10 or 5 out of 1,000,000              │
  └─────────────────────────────────────────────────────┘
```

---

## 2. Common SLI Categories

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    COMMON SLI TYPES                          │
  ├──────────────┬───────────────────────────────────────────────┤
  │  AVAILABILITY│  Was the service up and able to respond?      │
  │              │  Good event: successful response (non-5xx)    │
  │              │  Total: all requests                          │
  ├──────────────┼───────────────────────────────────────────────┤
  │  LATENCY     │  How fast did the service respond?            │
  │              │  Good event: response within threshold        │
  │              │  Total: all requests                          │
  ├──────────────┼───────────────────────────────────────────────┤
  │  THROUGHPUT  │  How much data/work was processed?            │
  │              │  Good event: request processed within time    │
  │              │  Total: all requests received                 │
  ├──────────────┼───────────────────────────────────────────────┤
  │  CORRECTNESS │  Did the service return the right answer?     │
  │              │  Good event: correct response                 │
  │              │  Total: all responses                         │
  ├──────────────┼───────────────────────────────────────────────┤
  │  FRESHNESS   │  How up-to-date is the data?                 │
  │              │  Good event: data updated within threshold    │
  │              │  Total: all data checks                       │
  ├──────────────┼───────────────────────────────────────────────┤
  │  DURABILITY  │  Is stored data safe from loss?               │
  │              │  Good event: data retrievable when requested  │
  │              │  Total: all stored items                      │
  └──────────────┴───────────────────────────────────────────────┘
```

---

## 3. SLI Examples by Service Type

### 3.1 Web API / HTTP Service

```
  Availability SLI:
    Good = HTTP responses with status < 500
    Total = All HTTP requests
    
    SLI = (Total Requests - 5xx Errors) / Total Requests × 100%

  Latency SLI:
    Good = Requests completed in < 200ms
    Total = All HTTP requests
    
    SLI = Requests < 200ms / Total Requests × 100%
```

### 3.2 Data Pipeline

```
  Freshness SLI:
    Good = Records processed within 10 minutes of creation
    Total = All records
    
    SLI = Fresh Records / Total Records × 100%

  Correctness SLI:
    Good = Records that match expected output
    Total = All processed records
    
    SLI = Correct Records / Total Records × 100%
```

### 3.3 Storage System

```
  Durability SLI:
    Good = Objects successfully retrieved on read
    Total = All read requests for existing objects
    
    SLI = Successful Reads / Total Reads × 100%

  Availability SLI:
    Good = Successful write acknowledgements
    Total = All write attempts
```

---

## 4. Where to Measure SLIs

The measurement point matters enormously. Measuring at different points gives different views of user experience:

```
  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │   User's Browser/App                                    │
  │   ┌─────────────┐                                      │
  │   │  Client-side │ ◄── BEST: Measures actual user      │
  │   │  measurement │     experience (Real User            │
  │   └──────┬──────┘     Monitoring / RUM)                │
  │          │                                              │
  │          ▼                                              │
  │   ┌─────────────┐                                      │
  │   │  Load       │ ◄── GOOD: Measures at your edge      │
  │   │  Balancer   │     (includes network within         │
  │   │  logs       │      your infra)                     │
  │   └──────┬──────┘                                      │
  │          │                                              │
  │          ▼                                              │
  │   ┌─────────────┐                                      │
  │   │  Application│ ◄── OK: Misses LB overhead and       │
  │   │  server     │     network issues                   │
  │   │  metrics    │                                      │
  │   └──────┬──────┘                                      │
  │          │                                              │
  │          ▼                                              │
  │   ┌─────────────┐                                      │
  │   │  Synthetic  │ ◄── SUPPLEMENT: Controlled probes    │
  │   │  monitoring │     from external locations          │
  │   │  (probes)   │     (doesn't reflect real users)     │
  │   └─────────────┘                                      │
  │                                                         │
  │   [!] Measure as close to the user as possible!         │
  └─────────────────────────────────────────────────────────┘
```

---

## 5. SLI Specification vs Implementation

### Specification (what you want to measure)

```yaml
# SLI Specification — conceptual definition
sli:
  name: "API Availability"
  description: "Proportion of successful HTTP requests"
  good_event: "HTTP response with status code < 500"
  total_event: "All HTTP requests received"
  measurement_point: "Load balancer access logs"
```

### Implementation (how you actually measure it)

```yaml
# Prometheus query implementing the SLI
# Availability SLI
- record: sli:availability:ratio_rate5m
  expr: |
    sum(rate(http_requests_total{status!~"5.."}[5m]))
    /
    sum(rate(http_requests_total[5m]))

# Latency SLI (proportion under 200ms)
- record: sli:latency:ratio_rate5m
  expr: |
    sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m]))
    /
    sum(rate(http_request_duration_seconds_count[5m]))
```

### Grafana Dashboard Query

```promql
# Error rate (inverse of availability SLI)
1 - (
  sum(rate(http_requests_total{status!~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
)
```

---

## 6. SLI Best Practices

```
  ┌──────────────────────────────────────────────────────┐
  │              SLI BEST PRACTICES                      │
  │                                                      │
  │  1. MEASURE WHAT USERS EXPERIENCE                    │
  │     ✗ CPU usage, memory usage (causes, not symptoms) │
  │     ✓ Request success rate, latency (user-facing)    │
  │                                                      │
  │  2. USE RATIOS (0-100%)                              │
  │     ✗ "5 errors per minute"                          │
  │     ✓ "99.95% of requests successful"                │
  │                                                      │
  │  3. KEEP IT SIMPLE (3-5 SLIs per service)            │
  │     ✗ 20 SLIs covering every metric                  │
  │     ✓ Availability + Latency + one domain-specific   │
  │                                                      │
  │  4. MEASURE AT THE EDGE                              │
  │     ✗ Application-level metrics only                 │
  │     ✓ Load balancer logs + client-side monitoring     │
  │                                                      │
  │  5. SEPARATE GOOD/BAD, NOT JUST ERROR/TOTAL          │
  │     ✗ Error count                                    │
  │     ✓ (Total - Errors) / Total                       │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Defining SLIs for an E-Commerce Platform

```
  Service: Product Search API
  
  ┌────────────────────────────────────────────────────────┐
  │  SLI 1: Availability                                   │
  │  ────────────────────                                  │
  │  Good: Non-5xx responses from the search endpoint      │
  │  Total: All search requests                            │
  │  Formula: non-5xx / total × 100%                       │
  │  Target range: 99.9% - 99.99%                          │
  │                                                        │
  │  SLI 2: Latency                                        │
  │  ───────────────                                       │
  │  Good: Search results returned in < 300ms              │
  │  Total: All search requests                            │
  │  Formula: requests_under_300ms / total × 100%          │
  │  Target range: 95% - 99%                               │
  │                                                        │
  │  SLI 3: Correctness (domain-specific)                  │
  │  ────────────────────────────────                      │
  │  Good: Search results contain the queried product      │
  │  Total: All search queries for in-stock products       │
  │  Formula: correct_results / total_queries × 100%       │
  │  Note: Measured via periodic synthetic queries          │
  └────────────────────────────────────────────────────────┘
```

---

## 8. Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| Using CPU as an SLI | CPU usage doesn't directly correlate with user experience | Use latency and error rate instead |
| Too many SLIs | Creates noise, hard to track | 3-5 per service max |
| Measuring at the server only | Misses network/LB issues | Add load balancer and client-side metrics |
| Not distinguishing client vs server errors | 4xx errors are usually the client's fault | Exclude 4xx from availability SLI |
| Average latency as SLI | Averages hide tail latency | Use percentiles (p95, p99) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SLI Definition** | Quantitative measure of service performance expressed as a ratio |
| **Formula** | Good Events / Total Events × 100% |
| **Common Types** | Availability, Latency, Throughput, Correctness, Freshness, Durability |
| **Measurement Point** | As close to the user as possible (client-side > LB > server) |
| **Best Practice** | 3-5 SLIs per service, user-centric, ratio-based |
| **Implementation** | Prometheus recording rules, log analysis, synthetic monitoring |
| **Key Insight** | SLIs measure symptoms (user experience) not causes (CPU, memory) |

---

## Quick Revision Questions

1. **What is the standard formula for an SLI?**
   > SLI = Good Events / Total Events × 100%. For example, availability = successful requests / total requests.

2. **Why are SLIs expressed as ratios rather than raw counts?**
   > Ratios are independent of traffic volume, naturally bounded (0-100%), and directly comparable to SLO targets.

3. **Name four common types of SLIs and give an example of each.**
   > Availability (non-5xx responses), Latency (requests under 200ms), Correctness (accurate search results), Freshness (data updated within 10 min).

4. **Where should SLIs be measured, and why?**
   > As close to the user as possible — ideally at the load balancer or client-side. Server-side metrics miss network and infrastructure issues.

5. **Why is average latency a poor SLI?**
   > Averages hide tail latency. A p99 of 5s can be masked by a 50ms average. Percentiles (p95, p99) reveal the worst user experiences.

6. **What is the difference between an SLI specification and an SLI implementation?**
   > Specification defines *what* to measure conceptually. Implementation defines *how* to measure it with specific tools and queries.

---

[← Previous: SRE Team Structure](../01-SRE-Fundamentals/05-sre-team-structure.md) | [Back to README](../README.md) | [Next: SLOs →](02-slos.md)
