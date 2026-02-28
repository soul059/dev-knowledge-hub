# Chapter 5: Log Aggregation Patterns

[← Previous: Loki](04-loki.md) | [Back to README](../README.md) | [Next: Elasticsearch Concepts →](../06-ELK-EFK-Stack/01-elasticsearch-concepts.md)

---

## Overview

Log aggregation patterns define how logs flow from applications to centralized storage. Choosing the right pattern affects reliability, cost, latency, and operational complexity.

---

## 5.1 Common Aggregation Patterns

```
┌──────────────────────────────────────────────────────────┐
│            LOG AGGREGATION PATTERNS                      │
│                                                          │
│  Pattern 1: Direct Push                                  │
│  App ──────────────────────────────▶ Log Store           │
│  Simple but couples app to backend                       │
│                                                          │
│  Pattern 2: Agent-Based (most common)                    │
│  App ──▶ stdout ──▶ Agent ──▶ Log Store                 │
│  Decoupled, agent handles retry/buffer                   │
│                                                          │
│  Pattern 3: Agent + Aggregator                           │
│  App ──▶ Agent ──▶ Aggregator ──▶ Log Store             │
│  (Fluent Bit)      (Fluentd)                            │
│  Scalable, two-tier collection                          │
│                                                          │
│  Pattern 4: Agent + Queue + Consumer                     │
│  App ──▶ Agent ──▶ Kafka ──▶ Consumer ──▶ Log Store    │
│  Highly scalable, buffered, multi-consumer              │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Pattern Comparison

| Pattern | Complexity | Reliability | Scalability | Latency | Use Case |
|---------|-----------|-------------|-------------|---------|----------|
| Direct push | Low | Low | Low | Low | Dev, small apps |
| Agent-based | Medium | High | Medium | Low | Most production |
| Agent + Aggregator | Medium | High | High | Medium | Large Kubernetes |
| Agent + Kafka | High | Very High | Very High | Higher | Enterprise scale |

---

## 5.3 Agent + Aggregator Pattern (Two-Tier)

```
┌──────────────────────────────────────────────────────────┐
│  TWO-TIER LOG COLLECTION (Kubernetes)                    │
│                                                          │
│  ┌─── Node 1 ───────┐  ┌─── Node 2 ───────┐           │
│  │ ┌───┐ ┌───┐ ┌───┐│  │ ┌───┐ ┌───┐ ┌───┐│           │
│  │ │Pod│ │Pod│ │Pod││  │ │Pod│ │Pod│ │Pod││           │
│  │ └─┬─┘ └─┬─┘ └─┬─┘│  │ └─┬─┘ └─┬─┘ └─┬─┘│           │
│  │   └──┬──┘     │  │  │   └──┬──┘     │  │           │
│  │ ┌────▼────────▼─┐│  │ ┌────▼────────▼─┐│           │
│  │ │   Fluent Bit   ││  │ │   Fluent Bit   ││ DaemonSet │
│  │ │   (lightweight)││  │ │   (lightweight)││           │
│  │ └───────┬───────┘│  │ └───────┬───────┘│           │
│  └─────────┼────────┘  └─────────┼────────┘           │
│            └──────────┬──────────┘                      │
│                       ▼                                  │
│            ┌────────────────────┐                        │
│            │  Fluentd Aggregator│  Deployment            │
│            │  (buffering,       │  (2-3 replicas)       │
│            │   parsing,         │                        │
│            │   routing)         │                        │
│            └────────┬───────────┘                        │
│                     │                                    │
│          ┌──────────┼──────────┐                        │
│          ▼          ▼          ▼                        │
│     ┌────────┐ ┌────────┐ ┌────────┐                   │
│     │  Loki  │ │   S3   │ │ Elastic│                   │
│     └────────┘ └────────┘ └────────┘                   │
└──────────────────────────────────────────────────────────┘

Fluent Bit: Low memory (~15MB), fast, lightweight
Fluentd:    Feature-rich, 800+ plugins, flexible routing
```

---

## 5.4 Kafka-Based Pipeline

```yaml
# High-scale log pipeline with Kafka buffer

# Step 1: Fluent Bit → Kafka
[OUTPUT]
    Name        kafka
    Match       *
    Brokers     kafka:9092
    Topics      logs
    Format      json

# Step 2: Kafka Consumer → Loki/Elasticsearch
# Using Vector, Logstash, or custom consumer

# Benefits:
# - Kafka absorbs traffic spikes (buffering)
# - Multiple consumers can read same logs
# - Replay capability (re-process old logs)
# - Decouples producers from consumers
```

---

## 5.5 Log Processing Pipeline

```
┌──────────────────────────────────────────────────────────┐
│              LOG PROCESSING STAGES                       │
│                                                          │
│  Raw log ──▶ Parse ──▶ Enrich ──▶ Filter ──▶ Route     │
│                                                          │
│  Parse:    Extract fields from raw text                  │
│            JSON, regex, grok, logfmt                     │
│                                                          │
│  Enrich:   Add metadata                                  │
│            Kubernetes labels, geo-IP, hostname           │
│                                                          │
│  Filter:   Drop/modify logs                              │
│            Remove health checks, mask PII                │
│                                                          │
│  Route:    Send to appropriate destination               │
│            Errors → PagerDuty, All → Loki, Audit → S3   │
│                                                          │
│  Example Fluentd pipeline:                               │
│  <filter **>                                             │
│    @type parser                                          │
│    key_name log                                          │
│    <parse>                                               │
│      @type json                                          │
│    </parse>                                              │
│  </filter>                                               │
│                                                          │
│  <filter **>                                             │
│    @type record_transformer                              │
│    <record>                                              │
│      cluster "production-us-east"                        │
│    </record>                                             │
│  </filter>                                               │
│                                                          │
│  <match audit.**>                                        │
│    @type s3                                              │
│    s3_bucket audit-logs                                  │
│  </match>                                                │
│                                                          │
│  <match **>                                              │
│    @type loki                                            │
│    url http://loki:3100                                  │
│  </match>                                                │
└──────────────────────────────────────────────────────────┘
```

---

## 5.6 Log Collection Tools Comparison

| Tool | Language | Memory | Plugins | Best For |
|------|----------|--------|---------|----------|
| Fluent Bit | C | ~15 MB | 100+ | Edge/node collection |
| Fluentd | Ruby | ~40 MB | 800+ | Aggregation, routing |
| Filebeat | Go | ~30 MB | Limited | Elasticsearch ecosystem |
| Vector | Rust | ~20 MB | 100+ | High performance |
| Promtail | Go | ~25 MB | Limited | Loki specifically |
| Logstash | Java | ~500 MB | 200+ | Heavy transformation |

---

## 5.7 Best Practices

```
✅ DO:
  • Use structured (JSON) logging at the source
  • Collect at the node level (DaemonSet)
  • Buffer logs to handle spikes
  • Add correlation IDs (trace_id, request_id)
  • Set retention policies per log type
  • Monitor your logging pipeline itself

❌ DON'T:
  • Don't log everything at DEBUG in production
  • Don't use high-cardinality labels in Loki
  • Don't let log collector OOM (set memory limits)
  • Don't skip log rotation on nodes
  • Don't push logs directly from app to storage
  • Don't store logs without retention policies
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Log lag increasing | Pipeline bottleneck | Add more aggregator replicas, check back-pressure |
| Duplicate logs | At-least-once delivery | Use idempotent writes or dedup at query time |
| Missing logs | Buffer overflow/dropped | Increase buffer size, add persistent queue |
| High cost | Logging everything | Filter health checks, reduce debug logging |
| Pipeline failure | Single point of failure | Add Kafka buffer, HA for aggregators |

---

## Quick Revision Questions

1. **What are the four main log aggregation patterns?**
2. **When would you use a two-tier (Fluent Bit + Fluentd) architecture?**
3. **What benefits does Kafka provide in a log pipeline?**
4. **What are the four stages of log processing?**
5. **Compare Fluent Bit, Fluentd, and Vector — when would you use each?**
6. **What are three best practices for production log aggregation?**

---

[← Previous: Loki](04-loki.md) | [Back to README](../README.md) | [Next: Elasticsearch Concepts →](../06-ELK-EFK-Stack/01-elasticsearch-concepts.md)
