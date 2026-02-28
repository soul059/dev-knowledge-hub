# Unit 10: Observability

[← Previous: Security](../09-Security/01-security.md) | [Back to README](../README.md) | [Next: Advanced Topics →](../11-Advanced-Topics/01-advanced-topics.md)

---

## Chapter Overview

Observability in Kubernetes involves monitoring, logging, and tracing to understand the health and performance of your applications and cluster. This unit covers **logging architecture**, **Prometheus monitoring**, **Grafana dashboards**, **distributed tracing**, and **health checks (probes)**.

---

## 10.1 The Three Pillars of Observability

```
┌──────────────────────────────────────────────────────────────┐
│          THREE PILLARS OF OBSERVABILITY                       │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   METRICS    │  │    LOGS      │  │   TRACES     │       │
│  │              │  │              │  │              │       │
│  │ What is      │  │ What         │  │ How does a   │       │
│  │ happening?   │  │ happened?    │  │ request flow │       │
│  │              │  │              │  │ through the  │       │
│  │ - CPU usage  │  │ - App logs   │  │ system?      │       │
│  │ - Memory     │  │ - Error msgs │  │              │       │
│  │ - Request    │  │ - Audit      │  │ - Latency    │       │
│  │   rate       │  │   trails     │  │ - Service    │       │
│  │ - Error rate │  │ - System     │  │   deps       │       │
│  │              │  │   events     │  │ - Bottlenecks│       │
│  │              │  │              │  │              │       │
│  │ Tools:       │  │ Tools:       │  │ Tools:       │       │
│  │ Prometheus   │  │ EFK/ELK      │  │ Jaeger       │       │
│  │ Grafana      │  │ Loki         │  │ Zipkin       │       │
│  │ Datadog      │  │ Fluentd      │  │ Tempo        │       │
│  │ metrics-     │  │ Fluent Bit   │  │ OpenTelemetry│       │
│  │ server       │  │              │  │              │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 10.2 Logging in Kubernetes

### Logging Architecture

```
┌──────────────────────────────────────────────────────────────┐
│            KUBERNETES LOGGING ARCHITECTURE                     │
│                                                              │
│  ┌────── Node ──────────────────────────────────────┐        │
│  │                                                  │        │
│  │  ┌──── Pod ─────┐  stdout/stderr                 │        │
│  │  │  Container   │──────────┐                     │        │
│  │  │  (app logs)  │          │                     │        │
│  │  └──────────────┘          ▼                     │        │
│  │                   /var/log/containers/            │        │
│  │                   /var/log/pods/                  │        │
│  │                        │                         │        │
│  │                        ▼                         │        │
│  │  ┌──────────────────────────────────┐            │        │
│  │  │ Node-level Logging Agent         │            │        │
│  │  │ (DaemonSet)                      │            │        │
│  │  │  - Fluentd / Fluent Bit          │            │        │
│  │  │  - Filebeat                      │            │        │
│  │  └────────────────┬─────────────────┘            │        │
│  │                   │                              │        │
│  └───────────────────┼──────────────────────────────┘        │
│                      │                                       │
│                      ▼                                       │
│         ┌────────────────────────┐                            │
│         │   Log Backend          │                            │
│         │  - Elasticsearch       │                            │
│         │  - Loki                │                            │
│         │  - CloudWatch Logs     │                            │
│         │  - Splunk              │                            │
│         └────────────┬───────────┘                            │
│                      │                                       │
│                      ▼                                       │
│         ┌────────────────────────┐                            │
│         │   Visualization        │                            │
│         │  - Kibana              │                            │
│         │  - Grafana             │                            │
│         └────────────────────────┘                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Viewing Logs

```bash
# ═══════════ kubectl logs ═══════════

# Current logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container>     # Multi-container pod

# Follow logs (stream)
kubectl logs -f <pod-name>

# Last N lines
kubectl logs --tail=100 <pod-name>

# Logs from last hour
kubectl logs --since=1h <pod-name>

# Logs from previous container instance (after crash)
kubectl logs --previous <pod-name>

# Logs from all pods with a label
kubectl logs -l app=web --all-containers=true

# Logs from a deployment (all pods)
kubectl logs deployment/web-app

# Logs with timestamps
kubectl logs --timestamps=true <pod-name>
```

### EFK Stack (Elasticsearch + Fluentd + Kibana)

```
┌──────────────────────────────────────────────────────────────┐
│                  EFK STACK                                    │
│                                                              │
│  Node 1          Node 2          Node 3                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                 │
│  │ Pods     │   │ Pods     │   │ Pods     │                 │
│  │  │       │   │  │       │   │  │       │                 │
│  │  ▼       │   │  ▼       │   │  ▼       │                 │
│  │ Fluentd  │   │ Fluentd  │   │ Fluentd  │                 │
│  │(DaemonSet│   │(DaemonSet│   │(DaemonSet│                 │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘                 │
│       │              │              │                        │
│       └──────────────┼──────────────┘                        │
│                      │                                       │
│                      ▼                                       │
│         ┌────────────────────────┐                            │
│         │    Elasticsearch       │   Index, store,           │
│         │    (StatefulSet)       │   search logs             │
│         └────────────┬───────────┘                            │
│                      │                                       │
│                      ▼                                       │
│         ┌────────────────────────┐                            │
│         │    Kibana              │   Visualize,              │
│         │    (Deployment)        │   query, dashboard        │
│         └────────────────────────┘                            │
│                                                              │
│  Alternative: Loki + Grafana (lighter weight)                │
│  Loki stores logs as compressed chunks (cheaper than ES)     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Application Logging Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│  LOGGING BEST PRACTICES                                      │
│                                                              │
│  1. Write logs to stdout/stderr (not files)                  │
│     → K8s captures container stdout/stderr automatically     │
│                                                              │
│  2. Use structured logging (JSON format)                     │
│     {"level":"error","msg":"failed","time":"2024-01-15"}     │
│                                                              │
│  3. Include correlation IDs (request-id, trace-id)           │
│     → Links logs across microservices                        │
│                                                              │
│  4. Set appropriate log levels                               │
│     DEBUG < INFO < WARN < ERROR < FATAL                      │
│     → Use INFO in production, DEBUG in dev                   │
│                                                              │
│  5. Use sidecar pattern for legacy apps that write to files  │
│     → Sidecar reads file and streams to stdout               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 10.3 Metrics and Prometheus

### Metrics Server

The **metrics-server** collects CPU/memory metrics from kubelets for `kubectl top` and autoscaling.

```bash
# Install metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View resource usage
kubectl top nodes                    # Node CPU/memory
kubectl top pods                     # Pod CPU/memory
kubectl top pods --containers=true   # Per-container
kubectl top pods -n kube-system      # Specific namespace
kubectl top pods --sort-by=cpu       # Sort by CPU
kubectl top pods --sort-by=memory    # Sort by memory
```

### Prometheus Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              PROMETHEUS ARCHITECTURE                          │
│                                                              │
│  ┌─────────── Targets ──────────────────────┐                │
│  │                                          │                │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐  │                │
│  │  │ App Pod  │ │ Node     │ │ kube-    │  │                │
│  │  │:8080/    │ │ Exporter │ │ state-   │  │                │
│  │  │ metrics  │ │:9100/    │ │ metrics  │  │                │
│  │  │          │ │ metrics  │ │:8080/    │  │                │
│  │  └─────┬────┘ └─────┬────┘ │ metrics  │  │                │
│  │        │            │      └─────┬────┘  │                │
│  └────────┼────────────┼────────────┼───────┘                │
│           │  PULL      │  PULL      │  PULL                  │
│           ▼            ▼            ▼                        │
│  ┌──────────────────────────────────────────┐                │
│  │           Prometheus Server               │                │
│  │                                          │                │
│  │  ┌──────────────┐  ┌─────────────────┐   │                │
│  │  │  Scrape      │  │   TSDB          │   │                │
│  │  │  Manager     │  │  (Time Series   │   │                │
│  │  │  (pulls      │  │   Database)     │   │                │
│  │  │   metrics)   │  │                 │   │                │
│  │  └──────────────┘  └─────────────────┘   │                │
│  │                                          │                │
│  │  ┌──────────────┐  ┌─────────────────┐   │                │
│  │  │  Rules       │  │   AlertManager  │──▶│ Slack, Email,  │
│  │  │  Engine      │  │   (alerts)      │   │ PagerDuty      │
│  │  └──────────────┘  └─────────────────┘   │                │
│  └───────────────────────┬──────────────────┘                │
│                          │                                   │
│                          ▼                                   │
│  ┌──────────────────────────────────────────┐                │
│  │           Grafana                         │                │
│  │  (Dashboards and visualization)           │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
│  Key: Prometheus PULLS (scrapes) metrics from targets        │
│       at regular intervals (default 30s)                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Prometheus ServiceMonitor (with Prometheus Operator)

```yaml
# Deploy using kube-prometheus-stack Helm chart
# helm install prometheus prometheus-community/kube-prometheus-stack

# ServiceMonitor tells Prometheus what to scrape
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
  labels:
    release: prometheus           # Must match Prometheus selector
spec:
  selector:
    matchLabels:
      app: my-app                 # Matches Service labels
  endpoints:
  - port: metrics                 # Service port name
    interval: 30s                 # Scrape interval
    path: /metrics                # Metrics endpoint path
---
# App Service exposing metrics port
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  ports:
  - name: http
    port: 8080
  - name: metrics                 # Metrics port
    port: 9090
  selector:
    app: my-app
```

### Key Prometheus Metrics (PromQL)

```
┌──────────────────────────────────────────────────────────────┐
│            ESSENTIAL PromQL QUERIES                           │
│                                                              │
│  Cluster Level:                                              │
│  ─────────────                                               │
│  # CPU usage per node                                        │
│  100 - (avg by(instance)                                     │
│    (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)    │
│                                                              │
│  # Memory usage per node                                     │
│  node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes │
│                                                              │
│  # Disk usage                                                │
│  node_filesystem_avail_bytes / node_filesystem_size_bytes    │
│                                                              │
│  Pod Level:                                                  │
│  ──────────                                                  │
│  # CPU usage per pod                                         │
│  rate(container_cpu_usage_seconds_total{                      │
│    container!="POD"}[5m])                                    │
│                                                              │
│  # Memory usage per pod                                      │
│  container_memory_working_set_bytes{container!="POD"}        │
│                                                              │
│  # Pod restart count                                         │
│  kube_pod_container_status_restarts_total                    │
│                                                              │
│  Application Level:                                          │
│  ─────────────────                                           │
│  # HTTP request rate                                         │
│  rate(http_requests_total[5m])                               │
│                                                              │
│  # HTTP error rate (5xx)                                     │
│  rate(http_requests_total{status=~"5.."}[5m])                │
│                                                              │
│  # Request latency (p99)                                     │
│  histogram_quantile(0.99,                                    │
│    rate(http_request_duration_seconds_bucket[5m]))            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Alerting Rules

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  labels:
    release: prometheus
spec:
  groups:
  - name: app-alerts
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      for: 5m                    # Must be true for 5 minutes
      labels:
        severity: critical
      annotations:
        summary: "High error rate on {{ $labels.instance }}"
        description: "Error rate is {{ $value }} errors/sec"

    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: warning

    - alert: HighMemoryUsage
      expr: |
        container_memory_working_set_bytes / 
        container_spec_memory_limit_bytes > 0.9
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.pod }} memory > 90% of limit"
```

---

## 10.4 Grafana Dashboards

```
┌──────────────────────────────────────────────────────────────┐
│               GRAFANA SETUP                                   │
│                                                              │
│  Grafana connects to Prometheus as a data source and         │
│  provides rich dashboards for visualization.                 │
│                                                              │
│  Pre-built dashboards (import by ID):                        │
│  ┌────────────────────────────────────────────────────┐      │
│  │ Dashboard                  │ Grafana ID │ For      │      │
│  ├────────────────────────────┼────────────┼──────────┤      │
│  │ K8s Cluster Overview       │ 7249       │ Cluster  │      │
│  │ Node Exporter Full         │ 1860       │ Nodes    │      │
│  │ K8s Pod Monitoring         │ 6336       │ Pods     │      │
│  │ NGINX Ingress Controller   │ 9614       │ Ingress  │      │
│  │ CoreDNS                    │ 5926       │ DNS      │      │
│  │ etcd                       │ 3070       │ etcd     │      │
│  └────────────────────────────┴────────────┴──────────┘      │
│                                                              │
│  Access: kubectl port-forward svc/grafana 3000:80 -n monitor │
│  Default login: admin / prom-operator (kube-prometheus-stack)│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 10.5 Distributed Tracing

Tracing follows a request as it flows through multiple microservices, revealing latency, bottlenecks, and errors.

```
┌──────────────────────────────────────────────────────────────┐
│            DISTRIBUTED TRACING                                │
│                                                              │
│  Request flow through microservices:                         │
│                                                              │
│  Client ──▶ API Gateway ──▶ User Service ──▶ Database        │
│                 │                                            │
│                 └──▶ Order Service ──▶ Payment Service        │
│                         │                                    │
│                         └──▶ Notification Service            │
│                                                              │
│  Trace (one request end-to-end):                             │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ Trace ID: abc-123                                    │    │
│  │                                                      │    │
│  │ ├─ Span: API Gateway      [=====]        25ms        │    │
│  │ │  ├─ Span: User Svc        [===]        15ms        │    │
│  │ │  │  └─ Span: DB Query       [=]         5ms        │    │
│  │ │  └─ Span: Order Svc       [======]      30ms       │    │
│  │ │     ├─ Span: Payment         [====]     20ms       │    │
│  │ │     └─ Span: Notify           [==]      10ms       │    │
│  │ │                                                    │    │
│  │ Total latency: 85ms                                  │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  Tracing Tools:                                              │
│  ┌──────────────┬──────────────────────────────────────┐     │
│  │ Jaeger       │ CNCF, full-featured, UI included     │     │
│  │ Zipkin       │ Older, lighter, Twitter origin       │     │
│  │ Tempo        │ Grafana's tracing backend            │     │
│  │ OpenTelemetry│ Vendor-neutral SDK (replaces         │     │
│  │              │ OpenTracing + OpenCensus)             │     │
│  └──────────────┴──────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### OpenTelemetry Integration

```yaml
# OpenTelemetry Collector (DaemonSet or Deployment)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
spec:
  replicas: 1
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
      - name: collector
        image: otel/opentelemetry-collector:latest
        ports:
        - containerPort: 4317      # gRPC OTLP
        - containerPort: 4318      # HTTP OTLP
        - containerPort: 8888      # Prometheus metrics
        volumeMounts:
        - name: config
          mountPath: /etc/otel
      volumes:
      - name: config
        configMap:
          name: otel-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-config
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    exporters:
      jaeger:
        endpoint: jaeger:14250
      prometheus:
        endpoint: 0.0.0.0:8888
    service:
      pipelines:
        traces:
          receivers: [otlp]
          exporters: [jaeger]
        metrics:
          receivers: [otlp]
          exporters: [prometheus]
```

---

## 10.6 Health Checks (Probes)

Kubernetes probes let the kubelet know the health status of containers.

```
┌──────────────────────────────────────────────────────────────┐
│                THREE TYPES OF PROBES                          │
│                                                              │
│  ┌────────────────┬──────────────────────────────────────┐   │
│  │ Probe          │ Purpose                              │   │
│  ├────────────────┼──────────────────────────────────────┤   │
│  │ Liveness       │ Is the container ALIVE?              │   │
│  │                │ Failure → restart container           │   │
│  │                │ Use: detect deadlocks, hangs          │   │
│  ├────────────────┼──────────────────────────────────────┤   │
│  │ Readiness      │ Is the container READY for traffic?  │   │
│  │                │ Failure → remove from Service         │   │
│  │                │   endpoints (no traffic sent)        │   │
│  │                │ Use: wait for startup, DB connection  │   │
│  ├────────────────┼──────────────────────────────────────┤   │
│  │ Startup        │ Has the container STARTED?           │   │
│  │                │ Failure → restart container           │   │
│  │                │ Disables liveness/readiness until     │   │
│  │                │   startup succeeds                   │   │
│  │                │ Use: slow-starting applications       │   │
│  └────────────────┴──────────────────────────────────────┘   │
│                                                              │
│  Probe Flow:                                                 │
│                                                              │
│  Container starts                                            │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────┐ Passes ┌──────────┐                            │
│  │ Startup  │───────▶│ Liveness │──── Fails ──▶ RESTART      │
│  │ Probe    │        │ Probe    │              container     │
│  │          │        └──────────┘                             │
│  │  Fails   │        ┌──────────┐                            │
│  │    │     │        │Readiness │──── Fails ──▶ Remove from  │
│  │    ▼     │        │ Probe    │              Service       │
│  │ RESTART  │        └──────────┘              endpoints     │
│  └──────────┘                                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Probe Mechanisms

```
┌──────────────────────────────────────────────────────────────┐
│              PROBE MECHANISMS                                 │
│                                                              │
│  ┌──────────┬──────────────────────────────────────────┐     │
│  │ Method   │ How it works                             │     │
│  ├──────────┼──────────────────────────────────────────┤     │
│  │ HTTP GET │ Send HTTP GET to container port/path     │     │
│  │          │ Success: 200-399 status code             │     │
│  ├──────────┼──────────────────────────────────────────┤     │
│  │ TCP      │ Open TCP connection to port              │     │
│  │ Socket   │ Success: connection established          │     │
│  ├──────────┼──────────────────────────────────────────┤     │
│  │ Exec     │ Run command inside container             │     │
│  │ Command  │ Success: exit code 0                     │     │
│  ├──────────┼──────────────────────────────────────────┤     │
│  │ gRPC     │ gRPC health check protocol               │     │
│  │          │ Success: SERVING status                   │     │
│  └──────────┴──────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Complete Probe Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-probes
spec:
  containers:
  - name: app
    image: my-app:1.0
    ports:
    - containerPort: 8080

    # Startup probe: for slow-starting apps
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5        # Wait before first check
      periodSeconds: 10             # Check every 10s
      failureThreshold: 30          # 30 failures = restart
      # Total startup time allowed: 5 + (10 * 30) = 305 seconds

    # Liveness probe: detect deadlocks
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: "probe"
      initialDelaySeconds: 0        # Start after startup passes
      periodSeconds: 15             # Check every 15s
      timeoutSeconds: 3             # Timeout per check
      failureThreshold: 3           # 3 failures = restart
      successThreshold: 1           # 1 success = healthy

    # Readiness probe: ready for traffic?
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
      successThreshold: 1
---
# TCP Socket probe example (databases)
apiVersion: v1
kind: Pod
metadata:
  name: db
spec:
  containers:
  - name: postgres
    image: postgres:16
    ports:
    - containerPort: 5432
    livenessProbe:
      tcpSocket:
        port: 5432
      periodSeconds: 20
    readinessProbe:
      tcpSocket:
        port: 5432
      periodSeconds: 10
---
# Exec probe example
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:7
    livenessProbe:
      exec:
        command:
        - redis-cli
        - ping
      periodSeconds: 10
    readinessProbe:
      exec:
        command:
        - redis-cli
        - ping
      periodSeconds: 5
```

### Probe Configuration Parameters

```
┌──────────────────────────────────────────────────────────────┐
│           PROBE PARAMETERS                                    │
│                                                              │
│  ┌──────────────────────┬──────────┬─────────────────────┐   │
│  │ Parameter            │ Default  │ Description          │   │
│  ├──────────────────────┼──────────┼─────────────────────┤   │
│  │ initialDelaySeconds  │ 0        │ Wait before 1st     │   │
│  │                      │          │ probe                │   │
│  │ periodSeconds        │ 10       │ Time between probes  │   │
│  │ timeoutSeconds       │ 1        │ Probe timeout        │   │
│  │ successThreshold     │ 1        │ Consecutive          │   │
│  │                      │          │ successes to pass    │   │
│  │ failureThreshold     │ 3        │ Consecutive failures │   │
│  │                      │          │ before action        │   │
│  └──────────────────────┴──────────┴─────────────────────┘   │
│                                                              │
│  Timing: First probe at initialDelaySeconds,                 │
│  then every periodSeconds.                                   │
│  Restart after failureThreshold * periodSeconds of failures. │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 10.7 Kubernetes Events

Events are API objects that record significant occurrences in the cluster.

```bash
# View events
kubectl get events                          # All events in default ns
kubectl get events --sort-by=.lastTimestamp  # Sorted by time
kubectl get events -n kube-system           # System events
kubectl get events --field-selector type=Warning  # Only warnings
kubectl get events -w                       # Watch for new events

# Events for a specific pod
kubectl describe pod <pod-name>             # Events section at bottom

# Common events:
# Scheduled     - Pod assigned to node
# Pulling       - Pulling container image
# Pulled        - Image pulled successfully
# Created       - Container created
# Started       - Container started
# Unhealthy     - Probe failed
# Killing       - Container being killed
# BackOff       - Back-off restarting failed container
# FailedMount   - Volume mount failed
```

---

## Real-World Scenario: Full Observability Stack

```yaml
# Application Deployment with comprehensive probes and monitoring
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
      annotations:
        prometheus.io/scrape: "true"       # Auto-discovery
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: api
        image: api-server:2.0
        ports:
        - name: http
          containerPort: 8080
        - name: metrics
          containerPort: 9090

        startupProbe:
          httpGet:
            path: /healthz
            port: 8080
          failureThreshold: 30
          periodSeconds: 10

        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          periodSeconds: 15
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
          failureThreshold: 3

        resources:
          requests:
            cpu: 250m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi

        env:
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector:4317"
        - name: OTEL_SERVICE_NAME
          value: "api-server"
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| `kubectl top` shows "metrics not available" | Check if metrics-server is installed | Install metrics-server; check for TLS issues in args |
| Liveness probe failing, pod restarting | `kubectl describe pod` → Events | Increase `initialDelaySeconds` or `failureThreshold`; fix health endpoint |
| Readiness probe failing, no traffic | `kubectl get endpoints <svc>` | Check readiness endpoint; ensure app is truly ready |
| Prometheus not scraping targets | Check ServiceMonitor labels match Prometheus selector | Verify `release` label; check `kubectl get servicemonitor` |
| No logs from container | `kubectl logs <pod> --previous` | Check if container is crashing; check image entrypoint |
| Grafana can't connect to Prometheus | Check Prometheus service URL | Verify Prometheus service name and port |

```bash
# Observability debugging commands
kubectl top nodes && kubectl top pods        # Resource usage
kubectl describe pod <pod> | tail -20        # Recent events
kubectl get events --sort-by='.lastTimestamp' | tail -20
kubectl logs <pod> --tail=50                 # Recent logs
kubectl exec <pod> -- wget -qO- localhost:8080/healthz  # Manual probe
kubectl get endpoints <service>              # Check healthy endpoints
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Three Pillars** | Metrics, Logs, Traces — all needed for full observability |
| **kubectl logs** | View container stdout/stderr; use `-f`, `--tail`, `--since`, `--previous` |
| **EFK Stack** | Elasticsearch + Fluentd + Kibana for centralized logging |
| **Loki** | Lightweight log aggregation (labels-based, no full-text index) |
| **metrics-server** | Basic CPU/memory metrics for `kubectl top` and HPA |
| **Prometheus** | Pull-based metrics collection; PromQL queries; CNCF graduated |
| **Grafana** | Dashboard visualization for Prometheus (and other sources) |
| **AlertManager** | Handles Prometheus alerts → Slack, email, PagerDuty |
| **Liveness Probe** | Is the container alive? Failure = restart container |
| **Readiness Probe** | Is the container ready? Failure = remove from endpoints |
| **Startup Probe** | Has the container started? Disables other probes during startup |
| **OpenTelemetry** | Vendor-neutral SDK for metrics, logs, and traces |
| **Events** | Cluster-level occurrences (scheduled, pulled, unhealthy, etc.) |

---

## Quick Revision Questions

1. **What is the difference between liveness and readiness probes?**
   <details><summary>Answer</summary>**Liveness probe**: Checks if the container is alive. If it fails, kubelet restarts the container. Used to detect deadlocks or hung processes. **Readiness probe**: Checks if the container is ready to receive traffic. If it fails, the pod is removed from Service endpoints (no traffic routed to it), but the container is NOT restarted. Used during startup, DB migrations, or when temporarily overloaded.</details>

2. **Why should you use a startup probe for slow-starting applications?**
   <details><summary>Answer</summary>Without a startup probe, you'd need to set high `initialDelaySeconds` on the liveness probe, which delays deadlock detection after startup. With a startup probe, liveness and readiness probes are disabled until startup succeeds. Example: `failureThreshold: 30, periodSeconds: 10` gives 300 seconds for startup. Once the app starts, liveness kicks in immediately with its own (shorter) settings.</details>

3. **How does Prometheus collect metrics, and how is it different from push-based systems?**
   <details><summary>Answer</summary>Prometheus uses a **pull model**: it scrapes HTTP endpoints (like `/metrics`) at regular intervals. This is different from push-based systems (like StatsD, Datadog Agent) where applications push metrics to a collector. Pull advantages: Prometheus can detect if a target is down (failed scrape), easier to manage (no client config needed), and simpler debugging. Service discovery (via ServiceMonitor, annotations) tells Prometheus what to scrape.</details>

4. **What happens to traffic when a readiness probe fails?**
   <details><summary>Answer</summary>The pod is removed from the Service's `Endpoints` list. The kube-proxy / CNI plugin stops routing traffic to that pod. Other ready pods continue receiving traffic. Once the readiness probe passes again, the pod is re-added to endpoints. The container is NOT restarted — only traffic routing is affected. Check with: `kubectl get endpoints <service-name>`.</details>

5. **What is the EFK stack and what does each component do?**
   <details><summary>Answer</summary>**E**lasticsearch: Stores, indexes, and enables searching of log data. **F**luentd: Runs as a DaemonSet on each node, collects container logs from `/var/log/containers/`, processes/transforms them, and forwards to Elasticsearch. **K**ibana: Web UI for querying, visualizing, and dashboarding log data from Elasticsearch. Alternative: Loki + Grafana (lighter — Loki only indexes labels, not full text).</details>

6. **How would you set up a basic Prometheus alert for pod crash loops?**
   <details><summary>Answer</summary>Create a PrometheusRule with an alert expression:
   ```yaml
   alert: PodCrashLooping
   expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
   for: 5m
   labels:
     severity: warning
   ```
   This triggers when the restart counter increases over 15 minutes, sustained for 5 minutes. AlertManager then routes the alert to configured receivers (Slack, email, PagerDuty) based on labels like severity.</details>

---

[← Previous: Security](../09-Security/01-security.md) | [Back to README](../README.md) | [Next: Advanced Topics →](../11-Advanced-Topics/01-advanced-topics.md)
