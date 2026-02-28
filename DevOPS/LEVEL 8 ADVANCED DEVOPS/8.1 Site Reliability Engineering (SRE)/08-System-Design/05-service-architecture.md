# Chapter 8.5: Service Architecture

[← Previous: Data Integrity](04-data-integrity.md) | [Back to README](../README.md)

---

## Overview

Service architecture defines how a system is decomposed into components, how they communicate, and how they are deployed. For SRE, architecture directly determines reliability, scalability, debuggability, and operational complexity. This chapter covers the architectural patterns SRE teams work with daily and the design decisions that make systems operable at scale.

---

## 1. Monolith vs Microservices

```
  ┌──────────────────────────────────────────────────────────┐
  │  ARCHITECTURE EVOLUTION                                  │
  │                                                          │
  │  MONOLITH:                                               │
  │  ┌──────────────────────────────────┐                    │
  │  │                                  │                    │
  │  │  Auth │ Orders │ Payments │ UI  │                    │
  │  │       │        │          │      │                    │
  │  │  ──── shared database ────────── │                    │
  │  │                                  │                    │
  │  └──────────────────────────────────┘                    │
  │  One deployable unit. Simple. But:                       │
  │  └─ Scale everything or nothing. One bad module affects │
  │     entire system. Large team coordination required.     │
  │                                                          │
  │  MICROSERVICES:                                          │
  │  ┌──────┐  ┌────────┐  ┌─────────┐  ┌────┐             │
  │  │ Auth │  │ Orders │  │Payments │  │ UI │             │
  │  │ DB1  │  │  DB2   │  │  DB3    │  │CDN │             │
  │  └──┬───┘  └───┬────┘  └────┬────┘  └──┬─┘             │
  │     │          │            │           │               │
  │     └──────────┴────────────┴───────────┘               │
  │              API Gateway / Service Mesh                  │
  │                                                          │
  │  Each service: independent deploy, scale, and fail.     │
  │  But: network complexity, distributed debugging,        │
  │       data consistency across services.                  │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────┬──────────────────┬─────────────────────┐
  │ Factor       │ Monolith         │ Microservices       │
  ├──────────────┼──────────────────┼─────────────────────┤
  │ Simplicity   │ ✓ Simple         │ ✗ Complex           │
  │ Deployment   │ ✗ All-or-nothing │ ✓ Independent       │
  │ Scaling      │ ✗ Uniform scale  │ ✓ Per-service scale │
  │ Debugging    │ ✓ Stack traces   │ ✗ Distributed trace │
  │ Data         │ ✓ ACID in one DB │ ✗ Eventual consist. │
  │ Team         │ ✗ Coordination   │ ✓ Team autonomy     │
  │ Reliability  │ ✗ Blast radius   │ ✓ Fault isolation   │
  │ Operations   │ ✓ One thing      │ ✗ Many things       │
  └──────────────┴──────────────────┴─────────────────────┘
```

---

## 2. Service Communication Patterns

```
  ┌──────────────────────────────────────────────────────────┐
  │  COMMUNICATION PATTERNS                                  │
  │                                                          │
  │  1. SYNCHRONOUS (Request-Response)                       │
  │                                                          │
  │  ┌──────┐  HTTP/gRPC  ┌──────┐                          │
  │  │Svc A │────────────▶│Svc B │                          │
  │  │      │◀────────────│      │                          │
  │  └──────┘   response  └──────┘                          │
  │  Simple. But: tight coupling, cascade failures.         │
  │                                                          │
  │  2. ASYNCHRONOUS (Event-Driven)                          │
  │                                                          │
  │  ┌──────┐  publish  ┌──────────┐  subscribe  ┌──────┐  │
  │  │Svc A │──────────▶│  Event   │────────────▶│Svc B │  │
  │  └──────┘           │  Bus     │────────────▶│Svc C │  │
  │  Fire-and-forget.   └──────────┘             └──────┘  │
  │  Loose coupling. But: eventual consistency, debugging.  │
  │                                                          │
  │  3. HYBRID (Saga Pattern)                                │
  │                                                          │
  │  ┌────┐   ┌────┐   ┌────┐   ┌────┐                     │
  │  │ T1 │──▶│ T2 │──▶│ T3 │──▶│Done│                     │
  │  └────┘   └────┘   └──┬─┘   └────┘                     │
  │                       │ failure                          │
  │  ┌────┐   ┌────┐   ┌─▼──┐                              │
  │  │ C1 │◀──│ C2 │◀──│ C3 │  Compensating transactions  │
  │  └────┘   └────┘   └────┘  (undo previous steps)       │
  │  Distributed transaction without 2PC overhead.          │
  └──────────────────────────────────────────────────────────┘

  PROTOCOL SELECTION:
  ┌───────────┬─────────────────┬────────────────────────────┐
  │ Protocol  │ Best For        │ SRE Considerations         │
  ├───────────┼─────────────────┼────────────────────────────┤
  │ REST/HTTP │ External APIs   │ Easy to debug, cache-      │
  │           │ CRUD operations │ friendly, well-tooled      │
  │           │                 │                            │
  │ gRPC      │ Internal svc-   │ Lower latency, strong      │
  │           │ to-svc, high    │ typing, streaming, but     │
  │           │ performance     │ harder to debug             │
  │           │                 │                            │
  │ GraphQL   │ Client-facing   │ Flexible queries but       │
  │           │ with varied     │ complexity in caching      │
  │           │ data needs      │ and rate limiting          │
  │           │                 │                            │
  │ Kafka     │ Event streaming │ Durable, ordered, but      │
  │           │ high throughput │ eventual consistency       │
  └───────────┴─────────────────┴────────────────────────────┘
```

---

## 3. API Gateway Pattern

```
  ┌──────────────────────────────────────────────────────────┐
  │  API GATEWAY                                             │
  │                                                          │
  │  ┌────────────┐                                          │
  │  │  Mobile    │──┐                                       │
  │  └────────────┘  │     ┌─────────────────────────────┐  │
  │  ┌────────────┐  ├────▶│       API GATEWAY           │  │
  │  │  Web App   │──┤     │                             │  │
  │  └────────────┘  │     │  ├─ Authentication          │  │
  │  ┌────────────┐  │     │  ├─ Rate Limiting           │  │
  │  │  Partner   │──┘     │  ├─ Request Routing         │  │
  │  └────────────┘        │  ├─ Load Balancing          │  │
  │                        │  ├─ Response Caching        │  │
  │                        │  ├─ Request/Response        │  │
  │                        │  │  Transformation          │  │
  │                        │  ├─ Circuit Breaking        │  │
  │                        │  └─ Observability           │  │
  │                        │     (logging, tracing)      │  │
  │                        └──────────┬──────────────────┘  │
  │                                   │                      │
  │                    ┌──────────────┼──────────────┐       │
  │                    ▼              ▼              ▼       │
  │               ┌────────┐    ┌────────┐    ┌────────┐    │
  │               │ Users  │    │ Orders │    │ Pay    │    │
  │               │ Service│    │ Service│    │ Service│    │
  │               └────────┘    └────────┘    └────────┘    │
  │                                                          │
  │  GATEWAY vs SERVICE MESH:                                │
  │  Gateway = North-South traffic (external → internal)    │
  │  Service Mesh = East-West traffic (service → service)   │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Service Mesh Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  SERVICE MESH (e.g., Istio, Linkerd)                     │
  │                                                          │
  │  ┌─────────────────────┐   ┌─────────────────────┐      │
  │  │  Pod A              │   │  Pod B              │      │
  │  │  ┌──────┐ ┌───────┐ │   │  ┌───────┐ ┌──────┐│      │
  │  │  │ App  │ │ Proxy │─┼───┼─▶│ Proxy │ │ App  ││      │
  │  │  │      │ │(Envoy)│ │   │  │(Envoy)│ │      ││      │
  │  │  └──────┘ └───────┘ │   │  └───────┘ └──────┘│      │
  │  └─────────────────────┘   └─────────────────────┘      │
  │            ▲                          ▲                   │
  │            │          ┌───────┐       │                   │
  │            └──────────│Control│───────┘                   │
  │                       │ Plane │                           │
  │                       └───────┘                           │
  │                                                          │
  │  WHAT THE PROXY HANDLES (so your app doesn't):          │
  │  ├─ mTLS encryption (service-to-service)                │
  │  ├─ Load balancing (L7, per-request)                    │
  │  ├─ Circuit breaking                                    │
  │  ├─ Retries with exponential backoff                    │
  │  ├─ Timeouts                                            │
  │  ├─ Distributed tracing headers                         │
  │  ├─ Traffic splitting (canary)                          │
  │  └─ Metrics collection (latency, errors, traffic)       │
  │                                                          │
  │  SRE BENEFIT: Consistent reliability features across    │
  │  all services without changing application code.        │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Observability Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  THREE PILLARS OF OBSERVABILITY                          │
  │                                                          │
  │  ┌──────────────────────────────────────────────┐        │
  │  │  METRICS (what is happening)                 │        │
  │  │  ├─ Counters (requests, errors)             │        │
  │  │  ├─ Gauges (CPU, memory, queue depth)       │        │
  │  │  ├─ Histograms (latency distribution)       │        │
  │  │  └─ Tool: Prometheus + Grafana              │        │
  │  └──────────────────────────────────────────────┘        │
  │                                                          │
  │  ┌──────────────────────────────────────────────┐        │
  │  │  LOGS (what happened, detailed)              │        │
  │  │  ├─ Structured (JSON with correlation IDs)  │        │
  │  │  ├─ Centralized (all services → one place)  │        │
  │  │  ├─ Queryable (search, filter, aggregate)   │        │
  │  │  └─ Tool: ELK Stack / Loki                  │        │
  │  └──────────────────────────────────────────────┘        │
  │                                                          │
  │  ┌──────────────────────────────────────────────┐        │
  │  │  TRACES (request path across services)       │        │
  │  │  ├─ Distributed tracing (span per service)  │        │
  │  │  ├─ Latency breakdown per hop               │        │
  │  │  ├─ Error propagation visualization         │        │
  │  │  └─ Tool: Jaeger / Tempo / Zipkin           │        │
  │  └──────────────────────────────────────────────┘        │
  │                                                          │
  │  EXAMPLE TRACE:                                          │
  │  ┌─ API Gateway (12ms) ──────────────────────────────┐  │
  │  │  ┌─ Auth Service (3ms) ──┐                        │  │
  │  │  └───────────────────────┘                        │  │
  │  │  ┌─ Order Service (45ms) ─────────────────────┐   │  │
  │  │  │  ┌─ DB Query (8ms) ──┐                     │   │  │
  │  │  │  └───────────────────┘                     │   │  │
  │  │  │  ┌─ Payment Service (30ms) ────────────┐   │   │  │
  │  │  │  │  ┌─ Stripe API (25ms) ──────────┐   │   │   │  │
  │  │  │  │  └─────────────────────────────┘   │   │   │  │
  │  │  │  └────────────────────────────────────┘   │   │  │
  │  │  └───────────────────────────────────────────┘   │  │
  │  └──────────────────────────────────────────────────┘  │
  │  Total: 57ms   Bottleneck: Stripe API (25ms, 44%)     │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Design for Operability Checklist

```yaml
# SRE Service Design Review Checklist
service_design_review:
  service_name: "order-service"
  
  reliability:
    - name: "Health endpoints"
      status: "implemented"
      details: "/healthz (liveness), /ready (readiness)"
    - name: "Graceful shutdown"
      status: "implemented"
      details: "Drain connections on SIGTERM, 30s grace"
    - name: "Circuit breakers"
      status: "implemented"
      details: "On all external calls (payment, inventory)"
    - name: "Retry logic"
      status: "implemented"
      details: "Exponential backoff, max 3 retries, idempotent"
    - name: "Timeouts"
      status: "implemented"
      details: "5s per external call, 30s total request"
    - name: "Rate limiting"
      status: "implemented"
      details: "100 req/s per user, 10K global"
      
  observability:
    - name: "Structured logging"
      status: "implemented"
      details: "JSON with trace_id, user_id, request_id"
    - name: "Metrics"
      status: "implemented"
      details: "RED metrics: Rate, Errors, Duration"
    - name: "Distributed tracing"
      status: "implemented"
      details: "OpenTelemetry SDK, Jaeger export"
    - name: "Dashboards"
      status: "implemented"
      details: "Service health, SLO burn rate, dependencies"
    - name: "Alerts"
      status: "implemented"
      details: "SLO-based burn rate alerts, error budget"
      
  deployment:
    - name: "Container image"
      status: "implemented"
      details: "Distroless base, multi-stage build"
    - name: "Config externalized"
      status: "implemented"
      details: "ConfigMaps + Secrets, no hardcoded values"
    - name: "Rollback tested"
      status: "implemented"
      details: "N-1 compatibility verified in CI"
    - name: "Canary support"
      status: "implemented"
      details: "Argo Rollouts with automated analysis"
      
  data:
    - name: "Backup strategy"
      status: "implemented"
      details: "Automated daily + PITR, tested monthly"
    - name: "Idempotent writes"
      status: "implemented"
      details: "Idempotency key on all mutations"
    - name: "Schema migrations"
      status: "implemented"
      details: "Expand-contract pattern, backward compatible"
```

---

## 7. Real-World Scenario

### [SCENARIO] Migrating from Monolith to Microservices

```
  SITUATION: E-commerce company with a 5-year-old monolith
  experiencing reliability issues and slow deployments.
  
  SYMPTOMS:
  ├─ Deploy takes 45 minutes (entire app redeploys)
  ├─ Any module failure crashes entire application
  ├─ Can't scale search without scaling payments
  ├─ 15+ developers stepping on each other's code
  └─ MTTR: 35 minutes (hard to isolate failures)
  
  ┌──────────────────────────────────────────────────────────┐
  │  STRANGLER FIG MIGRATION PATTERN                         │
  │                                                          │
  │  Phase 1: ADD API GATEWAY (Month 1-2)                   │
  │  ┌────────────────────────────────────────────┐          │
  │  │ API Gateway (new)                          │          │
  │  │ ├─ Routes 100% to monolith initially       │          │
  │  │ ├─ Provides consistent external API        │          │
  │  │ └─ Enables gradual traffic shifting        │          │
  │  └────────────────────────────────────────────┘          │
  │                                                          │
  │  Phase 2: EXTRACT FIRST SERVICE (Month 3-4)             │
  │  ┌──────────────┐  ┌──────────────────────────┐          │
  │  │ Auth Service │  │ Monolith (minus auth)   │          │
  │  │ (extracted)  │  │ Orders+Payments+Search  │          │
  │  └──────────────┘  └──────────────────────────┘          │
  │  Start with low-risk, well-bounded service.             │
  │                                                          │
  │  Phase 3: EXTRACT BY BUSINESS DOMAIN (Month 5-12)       │
  │  ┌──────┐ ┌────────┐ ┌─────────┐ ┌────────┐             │
  │  │ Auth │ │ Search │ │ Orders │ │  Pay   │             │
  │  └──────┘ └────────┘ └─────────┘ └────────┘             │
  │  Each extraction: extract → dual-write → validate →     │
  │  switch traffic → decommission monolith code.           │
  │                                                          │
  │  Phase 4: DECOMMISSION MONOLITH (Month 12-15)           │
  │  Remove remaining monolith code, clean up.              │
  │                                                          │
  │  KEY SRE PRACTICES DURING MIGRATION:                     │
  │  ├─ Feature flags on every new service route            │
  │  ├─ Shadow traffic (send to both, compare results)      │
  │  ├─ SLO tracking per service from day 1                 │
  │  ├─ Service mesh for consistent observability           │
  │  └─ Rollback plan for every extraction step             │
  │                                                          │
  │  RESULTS AFTER 15 MONTHS:                                │
  │  ├─ Deploy time: 45 min → 3 min per service            │
  │  ├─ Deploy frequency: weekly → 10x daily               │
  │  ├─ MTTR: 35 min → 4 min                               │
  │  ├─ Blast radius: 100% → single service (5-15%)        │
  │  └─ Availability: 99.9% → 99.97%                       │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Distributed debugging is too hard" | Implement distributed tracing (OpenTelemetry). Ensure all services propagate trace context. Correlate logs with trace IDs. |
| "Service-to-service latency is too high" | Use gRPC instead of REST for internal calls. Add connection pooling. Consider service mesh sidecar caching. Reduce serialization overhead. |
| "Too many microservices to manage" | Consolidate related services. Platform team provides shared infrastructure (service mesh, observability). Use internal developer platforms. |
| "Data consistency across services" | Use saga pattern for distributed transactions. Implement eventual consistency with reconciliation. Consider event sourcing. |
| "Service discovery not working reli| Use Kubernetes DNS for K8s workloads. Implement health-check-based discovery. Add circuit breakers for unavailable services. |

---

## Summary Table

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| **Monolith** | Single deployable unit | Small teams, early stage, simple domain |
| **Microservices** | Independent services per domain | Large teams, complex domain, need independent scaling |
| **API Gateway** | Single entry point for clients | External API consumers, cross-cutting concerns |
| **Service Mesh** | Infrastructure layer for service comms | Many services, need consistent reliability/security |
| **Event-Driven** | Async communication via events | Loose coupling, high throughput, eventual consistency OK |
| **Saga Pattern** | Distributed transactions | Cross-service workflows that need consistency |
| **Strangler Fig** | Incremental migration | Moving from monolith to microservices safely |

---

## Quick Revision Questions

1. **When should you choose microservices over a monolith?**
   > When you have multiple teams needing independent deployment, different services requiring different scaling characteristics, and a complex domain that benefits from bounded contexts. Don't adopt microservices prematurely — the operational complexity is significant.

2. **What is the strangler fig pattern?**
   > A migration strategy where you incrementally extract functionality from a monolith into new services, routing traffic through an API gateway. The monolith gradually shrinks until it can be fully decommissioned. Named after the strangler fig vine that grows around a tree.

3. **What problems does a service mesh solve?**
   > It provides consistent cross-cutting infrastructure for all service-to-service communication: mTLS encryption, load balancing, circuit breaking, retries, timeouts, distributed tracing, and traffic management — all without changing application code.

4. **What are the three pillars of observability?**
   > Metrics (what is happening — counters, gauges, histograms), Logs (what happened in detail — structured, centralized, queryable), and Traces (request path across services — latency breakdown, error propagation).

5. **What is the saga pattern and when do you need it?**
   > A pattern for managing distributed transactions across microservices. Instead of a single ACID transaction, each service executes a local transaction and publishes an event. If any step fails, compensating transactions undo previous steps. Use it when a business process spans multiple services.

6. **What should every service include for SRE operability?**
   > Health endpoints (liveness + readiness), structured logging with correlation IDs, RED metrics (Rate, Errors, Duration), distributed tracing, graceful shutdown, circuit breakers on external calls, timeouts, retry logic with backoff, and externalized configuration.

---

[← Previous: Data Integrity](04-data-integrity.md) | [Back to README](../README.md)
