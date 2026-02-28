# Chapter 2.4: Choosing SLIs

[← Previous: SLAs](03-slas.md) | [Back to README](../README.md) | [Next: Setting SLO Targets →](05-setting-slo-targets.md)

---

## Overview

Choosing the right SLIs is one of the most critical steps in building an SRE practice. Poor SLI choices lead to SLOs that don't reflect user experience, resulting in either false confidence or unnecessary firefighting. This chapter provides a systematic framework for selecting SLIs.

---

## 1. The SLI Selection Framework

```
  ┌─────────────────────────────────────────────────────────┐
  │         SLI SELECTION PROCESS                           │
  │                                                         │
  │  Step 1: Identify user journeys                         │
  │     └── "What does the user do with the service?"       │
  │                                                         │
  │  Step 2: Map journeys to service interactions           │
  │     └── "What systems/APIs does each journey touch?"    │
  │                                                         │
  │  Step 3: Choose SLI type for each interaction           │
  │     └── Availability? Latency? Correctness? Freshness? │
  │                                                         │
  │  Step 4: Define good vs total events                    │
  │     └── "What counts as 'good' for this SLI?"           │
  │                                                         │
  │  Step 5: Choose measurement method                      │
  │     └── Logs? Metrics? Probes? Client-side?            │
  │                                                         │
  │  Step 6: Validate the SLI                               │
  │     └── "Does this SLI correlate with user happiness?"  │
  └─────────────────────────────────────────────────────────┘
```

---

## 2. SLI Menu by Service Type

Google's SRE Workbook provides an "SLI Menu" — recommended SLIs based on service type:

```
  ┌──────────────────┬──────────────────────────────────────────┐
  │  SERVICE TYPE    │  RECOMMENDED SLIs                        │
  ├──────────────────┼──────────────────────────────────────────┤
  │                  │  • Availability (non-error responses)    │
  │  Request-Driven  │  • Latency (p50, p95, p99)              │
  │  (HTTP APIs,     │  • Throughput (requests/sec)             │
  │   gRPC services) │                                         │
  ├──────────────────┼──────────────────────────────────────────┤
  │                  │  • Freshness (delay from event to proc)  │
  │  Data Pipeline   │  • Correctness (valid output records)    │
  │  (ETL, streaming,│  • Throughput (records/minute)           │
  │   batch jobs)    │  • Coverage (% of input processed)       │
  ├──────────────────┼──────────────────────────────────────────┤
  │                  │  • Durability (data retrievable)         │
  │  Storage System  │  • Availability (read/write success)     │
  │  (databases,     │  • Latency (read/write time)             │
  │   object stores) │  • Throughput (IOPS)                     │
  ├──────────────────┼──────────────────────────────────────────┤
  │                  │  • Success rate (job completion)          │
  │  Scheduled Jobs  │  • Run duration (within threshold)       │
  │  (cron, batch)   │  • Freshness (completed on time)         │
  │                  │  • Correctness (expected output)          │
  └──────────────────┴──────────────────────────────────────────┘
```

---

## 3. User Journey Mapping

### Example: E-Commerce Platform

```
  User Journey: "Buy a Product"
  
  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  Browse  │──►│  Search  │──►│   View   │──►│  Add to  │
  │  Home    │   │ Products │   │  Product │   │   Cart   │
  └──────────┘   └──────────┘   └──────────┘   └──────────┘
       │              │              │              │
       ▼              ▼              ▼              ▼
  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ SLI:     │   │ SLI:     │   │ SLI:     │   │ SLI:     │
  │ Avail.   │   │ Latency  │   │ Avail. + │   │ Avail.   │
  │ + Latency│   │ + Correct│   │ Latency  │   │          │
  └──────────┘   └──────────┘   └──────────┘   └──────────┘
  
       ┌──────────┐   ┌──────────┐
  ────►│ Checkout │──►│ Payment  │
       │          │   │          │
       └──────────┘   └──────────┘
            │              │
            ▼              ▼
       ┌──────────┐   ┌──────────┐
       │ SLI:     │   │ SLI:     │
       │ Avail.   │   │ Avail. + │
       │ + Latency│   │ Correct. │
       └──────────┘   └──────────┘

  Key Insight: Payment correctness is MORE important than
  homepage latency. Weight SLIs by business impact.
```

---

## 4. Defining Good Events

The most nuanced part of SLI selection is defining what "good" means:

### Availability — What Counts as an Error?

```
  ┌─────────────────────────────────────────────────────────┐
  │  HTTP Status Codes — Include or Exclude?                │
  │                                                         │
  │  ┌─────────┬──────────────┬──────────────────────────┐  │
  │  │ Status  │ Include in   │ Reasoning                │  │
  │  │ Code    │ Error Count? │                          │  │
  │  ├─────────┼──────────────┼──────────────────────────┤  │
  │  │ 2xx     │ No (good)    │ Successful response      │  │
  │  │ 3xx     │ No (good)    │ Expected redirect        │  │
  │  │ 400     │ No           │ Client sent bad request  │  │
  │  │ 401/403 │ No           │ Auth issue (client-side) │  │
  │  │ 404     │ Depends      │ Maybe broken link (ours) │  │
  │  │ 408     │ No           │ Client timeout           │  │
  │  │ 429     │ YES          │ We're rate-limiting users │  │
  │  │ 499     │ No           │ Client cancelled         │  │
  │  │ 500     │ YES          │ Server error (our fault) │  │
  │  │ 502     │ YES          │ Bad gateway (our infra)  │  │
  │  │ 503     │ YES          │ Service unavailable      │  │
  │  │ 504     │ YES          │ Gateway timeout          │  │
  │  └─────────┴──────────────┴──────────────────────────┘  │
  │                                                         │
  │  [!] 429 is debatable — if you're shedding load due to  │
  │  capacity issues, it's an availability problem.         │
  └─────────────────────────────────────────────────────────┘
```

### Latency — Setting Thresholds

```
  Latency SLI: "Proportion of requests faster than X"
  
  But what is X?
  
  ┌────────────────────────────────────────────────────┐
  │  Method: Analyze historical latency distribution   │
  │                                                    │
  │  Requests ▲                                        │
  │           │  ████                                  │
  │           │  █████                                 │
  │           │  ██████                                │
  │           │  ████████                              │
  │           │  █████████                             │
  │           │  ██████████                            │
  │           │  ███████████  █                        │
  │           │  ████████████ ██ █  █    █             │
  │           └──────────────────────────────── ms     │
  │              50  100  200  500  1s  2s  5s         │
  │                                                    │
  │  Good threshold choices:                           │
  │    p50 threshold: 100ms (fast for most users)      │
  │    p99 threshold: 500ms (catches tail latency)     │
  │                                                    │
  │  SLIs:                                             │
  │    "95% of requests complete in < 100ms" (p50 SLI) │
  │    "99% of requests complete in < 500ms" (p99 SLI) │
  └────────────────────────────────────────────────────┘
```

---

## 5. Measurement Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │           SLI MEASUREMENT METHODS                        │
  ├──────────────────┬───────────────────────────────────────┤
  │                  │                                       │
  │  SERVER-SIDE     │  Prometheus metrics, application logs │
  │  METRICS         │  ┌────────────────────────────────┐  │
  │                  │  │  + Easy to implement            │  │
  │                  │  │  + High granularity             │  │
  │                  │  │  - Misses client-side issues    │  │
  │                  │  └────────────────────────────────┘  │
  │                  │                                       │
  │  LOAD BALANCER   │  LB access logs (ALB, NGINX, Envoy) │
  │  LOGS            │  ┌────────────────────────────────┐  │
  │                  │  │  + Sees all traffic             │  │
  │                  │  │  + Includes server errors       │  │
  │                  │  │  - Misses client-side issues    │  │
  │                  │  └────────────────────────────────┘  │
  │                  │                                       │
  │  CLIENT-SIDE     │  RUM (Real User Monitoring)          │
  │  MEASUREMENT     │  ┌────────────────────────────────┐  │
  │                  │  │  + Truest user experience       │  │
  │                  │  │  + Catches network issues       │  │
  │                  │  │  - Harder to implement          │  │
  │                  │  │  - Sampling / privacy concerns  │  │
  │                  │  └────────────────────────────────┘  │
  │                  │                                       │
  │  SYNTHETIC       │  Blackbox probes (Pingdom, etc.)     │
  │  MONITORING      │  ┌────────────────────────────────┐  │
  │                  │  │  + Detects outages fast         │  │
  │                  │  │  + Consistent baseline          │  │
  │                  │  │  - Doesn't reflect real users   │  │
  │                  │  │  - Limited coverage             │  │
  │                  │  └────────────────────────────────┘  │
  └──────────────────┴───────────────────────────────────────┘
  
  RECOMMENDATION: Use LB logs as primary + synthetic as supplement
```

---

## 6. SLI Implementation Examples

### Prometheus Recording Rules

```yaml
# Availability SLI — proportion of non-5xx responses
groups:
- name: sli_recording_rules
  rules:
  - record: sli:api_availability:ratio_5m
    expr: |
      sum(rate(http_requests_total{code!~"5.."}[5m])) by (service)
      /
      sum(rate(http_requests_total[5m])) by (service)

  # Latency SLI — proportion of requests under 200ms
  - record: sli:api_latency_200ms:ratio_5m
    expr: |
      sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m])) by (service)
      /
      sum(rate(http_request_duration_seconds_count[5m])) by (service)

  # Latency SLI — proportion of requests under 1s
  - record: sli:api_latency_1s:ratio_5m
    expr: |
      sum(rate(http_request_duration_seconds_bucket{le="1.0"}[5m])) by (service)
      /
      sum(rate(http_request_duration_seconds_count[5m])) by (service)
```

### Log-Based SLI (using BigQuery / SQL)

```sql
-- Monthly availability SLI from load balancer logs
SELECT
  DATE_TRUNC(timestamp, MONTH) AS month,
  COUNTIF(status_code < 500) / COUNT(*) * 100 AS availability_pct,
  COUNT(*) AS total_requests,
  COUNTIF(status_code >= 500) AS error_count
FROM
  `project.dataset.lb_access_logs`
WHERE
  timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY
  month
ORDER BY
  month DESC;
```

---

## 7. Validating SLI Choices

```
  ┌──────────────────────────────────────────────────────┐
  │            SLI VALIDATION CHECKLIST                   │
  │                                                      │
  │  □ Does the SLI correlate with user complaints?      │
  │    (When SLI drops, do support tickets increase?)    │
  │                                                      │
  │  □ Does the SLI capture the most impactful failures? │
  │    (Would the worst outage show up in this SLI?)     │
  │                                                      │
  │  □ Is the SLI measurable with current tools?         │
  │    (Can we actually query this data?)                │
  │                                                      │
  │  □ Is the SLI independent of traffic volume?         │
  │    (Does it stay meaningful at 10x or 0.1x traffic?) │
  │                                                      │
  │  □ Can we set a meaningful SLO target for this SLI?  │
  │    (Is there a clear "good enough" threshold?)       │
  │                                                      │
  │  □ Does the SLI avoid perverse incentives?           │
  │    (Will optimizing for this SLI make things better?)│
  └──────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Choosing SLIs for a Video Streaming Platform

```
  Service: Video Streaming Platform (like Netflix/YouTube)
  
  User Journeys:
  1. Browse catalog → Search → Play video → Watch
  2. Sign up → Subscribe → Manage account
  
  ┌──────────────────────────────────────────────────────┐
  │  Journey 1: Watch a Video                            │
  │                                                      │
  │  Step          │ SLI Type     │ Definition            │
  │  ──────────────│──────────────│──────────────────     │
  │  Browse catalog│ Availability │ Non-5xx responses     │
  │               │ Latency      │ Page load < 2s        │
  │                │              │                       │
  │  Search        │ Latency      │ Results < 500ms       │
  │               │ Correctness  │ Relevant results      │
  │                │              │                       │
  │  Start playback│ Availability │ Video starts playing  │
  │               │ Latency      │ Start time < 3s       │
  │                │              │                       │
  │  Watch video   │ Quality      │ Buffering < 1%        │
  │               │ Availability │ No mid-stream errors  │
  │                │              │                       │
  │  PRIORITY: Playback SLIs > Browse SLIs              │
  │  (Users tolerate slow browse but not buffering)     │
  └──────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "SLI doesn't catch outages we know about" | Map past incidents to SLI values. If SLI didn't move, it's measuring the wrong thing |
| "SLI is always 100%, never shows problems" | Threshold may be too loose, or measurement point too narrow |
| "We have 20+ SLIs and can't manage them" | Consolidate to 3-5 per service. Focus on user journeys, not internal metrics |
| "Different teams define 'error' differently" | Standardize across the org: publish an SLI definition guide |
| "Can't measure client-side latency" | Start with LB logs. Add RUM incrementally. Synthetic monitoring as bridge. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SLI Menu** | Recommended SLIs per service type (request-driven, pipeline, storage) |
| **User Journey Mapping** | Trace user actions → service interactions → SLIs |
| **Good Event Definition** | What counts as success (exclude client errors, include server errors) |
| **Measurement Point** | Client-side > LB > Server > Synthetic |
| **Latency Thresholds** | Use percentiles (p50, p99), not averages |
| **Validation** | SLI must correlate with user complaints and capture real failures |
| **Implementation** | Prometheus recording rules, log-based queries, RUM |

---

## Quick Revision Questions

1. **What are the six steps in the SLI selection framework?**
   > Identify user journeys → Map to service interactions → Choose SLI type → Define good/total events → Choose measurement method → Validate the SLI.

2. **What SLIs are recommended for a data pipeline?**
   > Freshness (processing delay), Correctness (valid outputs), Throughput (records/sec), Coverage (% of input processed).

3. **Should HTTP 400 errors count against your availability SLI? Why?**
   > No — 400 errors are client-caused (bad requests). Including them would penalize you for user mistakes.

4. **Why is average latency a bad SLI choice?**
   > Averages hide tail latency. Use percentiles — "95% of requests < 200ms" catches the worst user experiences.

5. **How do you validate that an SLI is measuring the right thing?**
   > Check if it correlates with user complaints, captures known outages, and doesn't create perverse incentives.

---

[← Previous: SLAs](03-slas.md) | [Back to README](../README.md) | [Next: Setting SLO Targets →](05-setting-slo-targets.md)
