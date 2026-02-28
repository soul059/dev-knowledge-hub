# Chapter 1: Log Management Concepts

[← Previous: Provisioning Dashboards](../04-Grafana/08-provisioning-dashboards.md) | [Back to README](../README.md) | [Next: Structured Logging →](02-structured-logging.md)

---

## Overview

Logs are timestamped records of discrete events that happened within an application or system. They are the most detailed telemetry signal and essential for debugging and root cause analysis.

---

## 1.1 What Are Logs?

```
┌──────────────────────────────────────────────────────────┐
│                    LOG ENTRY ANATOMY                     │
│                                                          │
│  2024-01-15T10:23:45.123Z  INFO  api-server  req-handler │
│  ├── Timestamp              │     │           │          │
│  │   When it happened       │     │           │          │
│  ├── Level ─────────────────┘     │           │          │
│  │   Severity of the event        │           │          │
│  ├── Source ──────────────────────┘           │          │
│  │   Which service/component                  │          │
│  └── Context ────────────────────────────────┘          │
│      Additional details                                  │
│                                                          │
│  Message: "GET /api/users 200 145ms"                    │
│  Metadata: {trace_id: "abc123", user_id: "u456"}        │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Log Levels

```
┌──────────────────────────────────────────────────────────┐
│  SEVERITY LEVELS (most to least severe)                  │
│                                                          │
│  FATAL/EMERGENCY  ████████████  System unusable          │
│  ERROR            ████████      Operation failed         │
│  WARN             ██████        Potential issues         │
│  INFO             ████          Normal operations        │
│  DEBUG            ██            Detailed diagnostics     │
│  TRACE            █             Very fine-grained        │
│                                                          │
│  Production: Usually INFO and above                      │
│  Debugging:  DEBUG or TRACE temporarily                  │
└──────────────────────────────────────────────────────────┘
```

| Level | When to Use | Example |
|-------|------------|---------|
| FATAL | Application cannot continue | "Database connection pool exhausted, shutting down" |
| ERROR | Operation failed, needs attention | "Payment processing failed: timeout" |
| WARN | Unexpected but recoverable | "Retry attempt 3/5 for external API" |
| INFO | Normal business operations | "Order #12345 created successfully" |
| DEBUG | Developer diagnostics | "Cache miss for key user:456, querying DB" |
| TRACE | Very detailed flow tracking | "Entering function processOrder with args..." |

---

## 1.3 Log Lifecycle

```
┌──────────────────────────────────────────────────────────┐
│                  LOG LIFECYCLE                            │
│                                                          │
│  Generate ──▶ Collect ──▶ Transport ──▶ Store ──▶ Query  │
│                                                          │
│  App writes    Agent       Ship to      Index &   Search │
│  log entry     reads       central      retain   analyze │
│  (stdout,      (Fluent     system       (ES,             │
│   file)        Bit,        (Kafka       Loki,            │
│                Filebeat)   optional)    S3)              │
│                                                          │
│  ┌─────┐   ┌──────────┐  ┌────────┐  ┌──────┐  ┌─────┐│
│  │ App │──▶│ Collector │─▶│Pipeline│─▶│Store │─▶│Query││
│  └─────┘   └──────────┘  └────────┘  └──────┘  └─────┘│
│                                                          │
│  Controls at each stage:                                │
│  • Generate: Log level, structured format               │
│  • Collect: Filtering, parsing                          │
│  • Transport: Buffering, compression                    │
│  • Store: Retention, indexing strategy                  │
│  • Query: Full-text search, aggregation                 │
└──────────────────────────────────────────────────────────┘
```

---

## 1.4 Log Destinations

```
  Application
       │
       ├──▶ stdout/stderr (12-factor app, containers)
       │     └──▶ Container runtime captures
       │          └──▶ Log collector ships
       │
       ├──▶ File (/var/log/app.log)
       │     └──▶ Log rotation (logrotate)
       │          └──▶ Filebeat/Fluentd reads
       │
       ├──▶ Syslog (UDP/TCP port 514)
       │     └──▶ Central syslog server
       │
       └──▶ Direct API (HTTP POST)
             └──▶ Logging service (Loki, Datadog)
```

### Best Practice: stdout for Containers

```
# 12-Factor App: Log to stdout
# Let the platform handle collection

# Docker captures stdout → /var/lib/docker/containers/
# Kubernetes → kubectl logs pod-name
# Collector (Fluent Bit) → reads from container runtime
```

---

## 1.5 Log Volume Considerations

```
┌──────────────────────────────────────────────────────────┐
│  LOG VOLUME ESTIMATION                                   │
│                                                          │
│  Inputs:                                                 │
│    Requests/sec:     1,000                               │
│    Log lines/req:    5 (access + app logic)              │
│    Avg line size:    500 bytes                            │
│                                                          │
│  Calculation:                                            │
│    1,000 req/s × 5 lines × 500 bytes = 2.5 MB/s         │
│    Per day:  2.5 MB/s × 86,400 = ~216 GB/day            │
│    30 days retention: ~6.5 TB                            │
│                                                          │
│  Cost implications:                                      │
│    Ingestion: $0.50/GB  → $108/day                      │
│    Storage:   $0.03/GB  → $195/month (6.5TB)            │
│                                                          │
│  Strategy: Log smartly, not everything                   │
│    ✅ Log business events, errors, warnings              │
│    ✅ Sample debug logs in production                    │
│    ❌ Don't log every health check                      │
│    ❌ Don't log sensitive data (PII, secrets)           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.6 Logs vs Metrics vs Traces

| Aspect | Logs | Metrics | Traces |
|--------|------|---------|--------|
| Data type | Text events | Numeric time series | Request spans |
| Cardinality | Unlimited | Must be bounded | Per-request |
| Storage cost | High | Low | Medium |
| Query speed | Slower (text search) | Fast (numeric) | Medium |
| Best for | Debugging, audit | Alerting, trends | Request flow |
| Retention | Days-weeks | Months-years | Days-weeks |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Logs missing | App writing to wrong destination | Check log output config, verify stdout/file |
| Log volume too high | Debug level in production | Set log level to INFO for production |
| Can't find relevant logs | No structure or correlation IDs | Implement structured logging with trace IDs |
| Logs delayed | Buffer/queue backlog | Check collector health, increase buffer |
| Sensitive data in logs | No log sanitization | Add PII scrubbing filters in pipeline |

---

## Quick Revision Questions

1. **What are the standard log severity levels and when should each be used?**
2. **What are the stages of the log lifecycle?**
3. **Why should containerized applications log to stdout?**
4. **How would you estimate log storage requirements for a service?**
5. **What are the trade-offs between logs and metrics for monitoring?**
6. **What strategies reduce log volume without losing important information?**

---

[← Previous: Provisioning Dashboards](../04-Grafana/08-provisioning-dashboards.md) | [Back to README](../README.md) | [Next: Structured Logging →](02-structured-logging.md)
