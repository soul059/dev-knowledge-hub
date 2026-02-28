# Chapter 2: Structured Logging

[← Previous: Log Management Concepts](01-log-management-concepts.md) | [Back to README](../README.md) | [Next: Logging in Kubernetes →](03-logging-in-kubernetes.md)

---

## Overview

Structured logging outputs log entries in a machine-parseable format (typically JSON) instead of free-form text. This enables efficient searching, filtering, and analysis at scale.

---

## 2.1 Unstructured vs Structured Logs

```
UNSTRUCTURED (traditional):
  2024-01-15 10:23:45 ERROR Failed to process order 12345 
  for user john@example.com: payment gateway timeout after 30s

STRUCTURED (JSON):
  {
    "timestamp": "2024-01-15T10:23:45.123Z",
    "level": "ERROR",
    "message": "Failed to process order",
    "service": "order-service",
    "order_id": "12345",
    "user_id": "u789",
    "error": "payment gateway timeout",
    "duration_ms": 30000,
    "trace_id": "abc123def456",
    "span_id": "span789"
  }
```

### Comparison

| Aspect | Unstructured | Structured |
|--------|-------------|-----------|
| Human readable | Very easy | Moderate |
| Machine parseable | Requires regex | Native JSON parsing |
| Querying | Full-text search | Field-level filtering |
| Storage efficiency | Moderate | Higher (compression) |
| Schema evolution | Flexible | Needs governance |
| Correlation | Difficult | Easy with IDs |

---

## 2.2 Structured Logging Standards

```
┌──────────────────────────────────────────────────────────┐
│            STRUCTURED LOG FIELD STANDARDS                 │
│                                                          │
│  Required Fields:                                        │
│  ┌────────────────┬─────────────────────────────────┐   │
│  │ timestamp      │ ISO 8601 with timezone          │   │
│  │ level          │ ERROR, WARN, INFO, DEBUG        │   │
│  │ message        │ Human-readable event summary    │   │
│  │ service        │ Name of the service             │   │
│  └────────────────┴─────────────────────────────────┘   │
│                                                          │
│  Recommended Fields:                                     │
│  ┌────────────────┬─────────────────────────────────┐   │
│  │ trace_id       │ Distributed trace identifier    │   │
│  │ span_id        │ Current span identifier         │   │
│  │ request_id     │ Unique request identifier       │   │
│  │ user_id        │ Acting user (if applicable)     │   │
│  │ environment    │ prod, staging, dev              │   │
│  │ version        │ Application version             │   │
│  │ host           │ Hostname or pod name            │   │
│  └────────────────┴─────────────────────────────────┘   │
│                                                          │
│  Context-Specific Fields:                                │
│  ┌────────────────┬─────────────────────────────────┐   │
│  │ http_method    │ GET, POST, PUT, DELETE          │   │
│  │ http_status    │ 200, 404, 500                   │   │
│  │ http_path      │ /api/users                      │   │
│  │ duration_ms    │ Request duration                 │   │
│  │ error_type     │ Exception class name            │   │
│  │ stack_trace    │ Full stack trace (errors only)  │   │
│  └────────────────┴─────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## 2.3 Implementation Examples

### Python (structlog)

```python
import structlog
import logging

# Configure structlog
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.JSONRenderer()
    ],
    wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
)

logger = structlog.get_logger()

# Usage
logger.info("order_created", order_id="12345", user_id="u789", total=99.99)
# Output: {"event":"order_created","order_id":"12345","user_id":"u789",
#          "total":99.99,"level":"info","timestamp":"2024-01-15T10:23:45Z"}

# Context binding (all subsequent logs include these fields)
log = logger.bind(request_id="req-abc", service="order-service")
log.info("processing_payment", amount=99.99)
log.error("payment_failed", error="gateway timeout", duration_ms=30000)
```

### Node.js (pino)

```javascript
const pino = require('pino');

const logger = pino({
  level: 'info',
  timestamp: pino.stdTimeFunctions.isoTime,
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: 'api-server',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION,
  },
});

// Usage
logger.info({ orderId: '12345', userId: 'u789' }, 'Order created');
// Output: {"level":"info","time":"2024-01-15T10:23:45.123Z",
//          "service":"api-server","orderId":"12345","msg":"Order created"}

// Child logger with bound context
const reqLogger = logger.child({ requestId: 'req-abc', traceId: 'trace-123' });
reqLogger.info({ path: '/api/orders' }, 'Request received');
```

### Go (zerolog)

```go
package main

import (
    "os"
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func main() {
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnixMs
    logger := zerolog.New(os.Stdout).With().
        Timestamp().
        Str("service", "api-server").
        Logger()

    logger.Info().
        Str("order_id", "12345").
        Str("user_id", "u789").
        Float64("total", 99.99).
        Msg("Order created")
}
```

---

## 2.4 Contextual Logging

```
┌──────────────────────────────────────────────────────────┐
│          CONTEXTUAL LOGGING PATTERN                      │
│                                                          │
│  Request enters ──▶ Create logger with context          │
│                      │                                   │
│                      ▼                                   │
│  ┌─────────────────────────────────────┐                │
│  │ logger = base_logger.bind(          │                │
│  │   request_id = "req-abc",           │                │
│  │   trace_id   = "trace-xyz",         │                │
│  │   user_id    = "u789",              │                │
│  │   method     = "POST",              │                │
│  │   path       = "/api/orders"        │                │
│  │ )                                   │                │
│  └──────────────┬──────────────────────┘                │
│                 │                                        │
│    ┌────────────┼────────────┐                          │
│    ▼            ▼            ▼                          │
│  Handler    Service      Repository                     │
│  logger     logger       logger                         │
│  .info()    .info()      .info()                        │
│                                                          │
│  ALL logs automatically include request_id, trace_id    │
│  No need to pass context manually everywhere            │
└──────────────────────────────────────────────────────────┘
```

---

## 2.5 Log Correlation with Traces

```json
// Log from service A
{
  "timestamp": "2024-01-15T10:23:45.100Z",
  "service": "api-gateway",
  "message": "Forwarding request to order-service",
  "trace_id": "abc123",
  "span_id": "span-001"
}

// Log from service B (same trace_id!)
{
  "timestamp": "2024-01-15T10:23:45.150Z",
  "service": "order-service",
  "message": "Processing order",
  "trace_id": "abc123",
  "span_id": "span-002",
  "parent_span_id": "span-001"
}

// In Grafana: Search by trace_id="abc123" to see ALL logs
// for this request across ALL services
```

---

## 2.6 What NOT to Log

```
❌ NEVER LOG:
  • Passwords or authentication tokens
  • Credit card numbers (PCI-DSS violation)
  • Social Security Numbers
  • Personal health information (HIPAA)
  • Full request/response bodies with PII
  • API keys or secrets

✅ SAFE TO LOG:
  • Hashed or masked identifiers
  • Request metadata (method, path, status, duration)
  • Business event summaries
  • Error messages and stack traces
  • Performance metrics (latency, queue depth)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| JSON parsing failures | Mixed structured/unstructured logs | Ensure all log output is JSON |
| Missing fields in some logs | Inconsistent logger setup | Use shared logger config across modules |
| Sensitive data in logs | No sanitization | Add field scrubbing processor |
| Large log entries | Stack traces or full payloads | Truncate or sample verbose fields |
| Hard to correlate across services | No trace_id | Add OpenTelemetry context propagation |

---

## Quick Revision Questions

1. **What are the key advantages of structured logging over unstructured?**
2. **What fields should every structured log entry include?**
3. **How does contextual logging work and why is it important?**
4. **How do you correlate logs across microservices?**
5. **What types of data should never appear in logs?**
6. **Compare structlog (Python), pino (Node.js), and zerolog (Go) — what do they have in common?**

---

[← Previous: Log Management Concepts](01-log-management-concepts.md) | [Back to README](../README.md) | [Next: Logging in Kubernetes →](03-logging-in-kubernetes.md)
