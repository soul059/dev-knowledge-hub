# Chapter 2: Monitoring and Observability

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Observability in containerised environments involves three pillars: **metrics**, **logs**, and **traces**. This chapter covers how to instrument Docker hosts and containers for production-grade monitoring, logging, and alerting.

---

## 2.1 The Three Pillars of Observability

```
  Observability Stack:
  
  +----------------+   +----------------+   +----------------+
  |    METRICS     |   |     LOGS       |   |    TRACES      |
  +----------------+   +----------------+   +----------------+
  | CPU, Memory,   |   | stdout/stderr  |   | Request flow   |
  | Network I/O,   |   | App logs       |   | across services|
  | Disk, Errors   |   | Audit logs     |   | Latency, Spans |
  +-------+--------+   +-------+--------+   +-------+--------+
          |                     |                     |
  +-------v--------+   +-------v--------+   +-------v--------+
  | Prometheus      |   | Loki / ELK    |   | Jaeger / Zipkin|
  | + Grafana       |   | + Grafana     |   | + Grafana      |
  +----------------+   +----------------+   +----------------+
          |                     |                     |
          +----------+----------+----------+----------+
                     |                     |
              +------v------+       +------v------+
              |  Dashboards |       |   Alerts    |
              |  Grafana    |       | PagerDuty   |
              +-------------+       +-------------+
```

---

## 2.2 Docker Daemon Metrics

```bash
# Enable Docker metrics endpoint
# /etc/docker/daemon.json
{
  "metrics-addr": "0.0.0.0:9323",
  "experimental": true
}

# Restart Docker
sudo systemctl restart docker

# Verify metrics endpoint
curl http://localhost:9323/metrics

# Key metrics exposed:
# engine_daemon_container_states_containers{state="running"}
# engine_daemon_image_actions_seconds
# process_cpu_seconds_total
# process_resident_memory_bytes
```

---

## 2.3 Container Metrics with cAdvisor

```yaml
# docker-compose.monitoring.yml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.0
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 128M
```

```
  cAdvisor Metrics Flow:
  
  +-------------+     +-------------+     +-------------+
  |  Container  |     |  Container  |     |  Container  |
  |  web-app    |     |  api        |     |  worker     |
  +------+------+     +------+------+     +------+------+
         |                   |                   |
  =======|===================|===================|======
         |    Docker Host cgroups + /proc         |
  =======|===================|===================|======
         |                   |                   |
  +------v-------------------v-------------------v------+
  |                    cAdvisor                          |
  |  Reads cgroup stats, /proc, /sys per container      |
  |  Exposes Prometheus-format metrics on :8080/metrics  |
  +---------------------------+-------------------------+
                              |
                    +---------v----------+
                    |    Prometheus       |
                    |  Scrapes /metrics   |
                    +--------------------+
```

---

## 2.4 Full Monitoring Stack (Prometheus + Grafana)

```yaml
# Complete monitoring stack
services:
  prometheus:
    image: prom/prometheus:v2.48.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prom_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.2.0
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=changeme
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.0
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:v1.7.0
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
    restart: unless-stopped

volumes:
  prom_data:
  grafana_data:
```

```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['host.docker.internal:9323']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'app'
    static_configs:
      - targets: ['api:3000']
```

---

## 2.5 Essential Alerts

```yaml
# prometheus/alert_rules.yml
groups:
  - name: container_alerts
    rules:
      # Container down
      - alert: ContainerDown
        expr: absent(container_last_seen{name="api"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.name }} is down"

      # High CPU usage
      - alert: ContainerHighCPU
        expr: >
          rate(container_cpu_usage_seconds_total{name!=""}[5m]) 
          * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Container {{ $labels.name }} CPU > 80%"

      # High memory usage
      - alert: ContainerHighMemory
        expr: >
          container_memory_usage_bytes{name!=""} 
          / container_spec_memory_limit_bytes{name!=""} 
          * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Container {{ $labels.name }} memory > 85%"

      # Container restarting
      - alert: ContainerRestarting
        expr: >
          increase(container_restart_count[1h]) > 3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Container {{ $labels.name }} restarting frequently"
```

---

## 2.6 Docker Logging Architecture

```
  Logging Architecture:
  
  +--Container--+     +--Container--+     +--Container--+
  | App writes  |     | App writes  |     | App writes  |
  | to stdout/  |     | to stdout/  |     | to stdout/  |
  | stderr      |     | stderr      |     | stderr      |
  +------+------+     +------+------+     +------+------+
         |                   |                   |
  =======|===================|===================|======
         |         Docker Log Driver              |
  =======|===================|===================|======
         |                   |                   |
         +------- Select one driver -------+
                          |
         +----------------+----------------+
         |                |                |
  +------v------+  +------v------+  +------v------+
  | json-file   |  | syslog      |  | fluentd     |
  | (default)   |  |             |  |             |
  +------+------+  +------+------+  +------+------+
         |                |                |
  +------v------+  +------v------+  +------v------+
  | Local Files |  | Syslog      |  | Fluent Bit  |
  | on host     |  | Server      |  | → Loki/ES   |
  +-------------+  +-------------+  +-------------+
```

```bash
# View container logs
docker logs mycontainer
docker logs --tail 100 --follow mycontainer
docker logs --since 1h mycontainer
docker logs --timestamps mycontainer

# Configure logging driver per container
docker run -d \
  --log-driver json-file \
  --log-opt max-size=50m \
  --log-opt max-file=5 \
  myapp:v1.0

# Configure default logging driver
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5",
    "labels": "app,environment",
    "tag": "{{.Name}}/{{.ID}}"
  }
}
```

---

## 2.7 Centralized Logging with Loki

```yaml
# Loki stack for log aggregation
services:
  loki:
    image: grafana/loki:2.9.0
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped

  promtail:
    image: grafana/promtail:2.9.0
    container_name: promtail
    volumes:
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail/config.yml:/etc/promtail/config.yml:ro
    command: -config.file=/etc/promtail/config.yml
    restart: unless-stopped

volumes:
  loki_data:
```

```yaml
# promtail/config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/**/*.log
    pipeline_stages:
      - docker: {}
      - labels:
          container_name:
```

---

## 2.8 Application-Level Metrics

```python
# Python app with Prometheus metrics
from prometheus_client import Counter, Histogram, start_http_server
import time

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 5.0]
)

# Expose /metrics endpoint on port 9090
start_http_server(9090)

# Instrument your handlers
def handle_request(method, endpoint):
    start = time.time()
    status = process_request()   # Your logic
    duration = time.time() - start
    
    REQUEST_COUNT.labels(method, endpoint, status).inc()
    REQUEST_LATENCY.labels(method, endpoint).observe(duration)
```

---

## 2.9 Key Metrics to Monitor

```
  RED Method (for services):
  
  +==============+================================+
  |   Metric     |   What to Watch                |
  +==============+================================+
  |   Rate       | Requests per second            |
  +--------------+--------------------------------+
  |   Errors     | Error rate (% of 5xx)          |
  +--------------+--------------------------------+
  |   Duration   | Response time percentiles      |
  |              | (p50, p95, p99)                |
  +==============+================================+
  
  USE Method (for resources):
  
  +==============+================================+
  |   Metric     |   What to Watch                |
  +==============+================================+
  | Utilization  | CPU %, Memory %, Disk %        |
  +--------------+--------------------------------+
  | Saturation   | Queue depth, wait time         |
  +--------------+--------------------------------+
  | Errors       | OOM kills, disk errors         |
  +==============+================================+
```

```bash
# Quick container stats
docker stats --no-stream --format \
  "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Check for OOM kills
docker inspect mycontainer | grep -i oom
# "OOMKilled": true  ← container ran out of memory
```

---

## Summary Table

| Component | Tool | Purpose |
|-----------|------|---------|
| **Host metrics** | Node Exporter | CPU, memory, disk, network |
| **Container metrics** | cAdvisor | Per-container resource usage |
| **Docker daemon** | Built-in metrics | Engine-level stats |
| **App metrics** | Prometheus client | Business/app-level metrics |
| **Metrics storage** | Prometheus | Time-series database |
| **Log collection** | Promtail / Fluent Bit | Ship logs to aggregator |
| **Log storage** | Loki / Elasticsearch | Searchable log storage |
| **Visualization** | Grafana | Dashboards for all data |
| **Alerting** | Alertmanager | Route alerts to channels |
| **Quick check** | `docker stats` | CLI real-time overview |

---

## Quick Revision Questions

1. **What are the three pillars of observability?**
   - Metrics (quantitative measurements over time), Logs (discrete events with details), and Traces (request flow across services showing latency and dependencies)

2. **What is cAdvisor and what does it do?**
   - Container Advisor reads cgroup stats from the host to expose per-container CPU, memory, network, and disk metrics in Prometheus format for scraping

3. **Why must you configure log rotation in production?**
   - Without rotation (`max-size` / `max-file`), container logs grow unbounded and can fill the disk, crashing the Docker host and all containers

4. **Explain the RED method for monitoring services.**
   - Rate (requests per second), Errors (percentage of failed requests), Duration (latency distribution, especially p95/p99). These three metrics capture service health

5. **How does Prometheus discover and collect container metrics?**
   - Prometheus scrapes HTTP `/metrics` endpoints at configured intervals. cAdvisor, Node Exporter, and apps expose these endpoints; `scrape_configs` in `prometheus.yml` defines targets

6. **What happens when a container is OOM-killed?**
   - The kernel kills the container's main process when it exceeds its memory limit. Docker marks `OOMKilled: true`. The restart policy determines if it restarts automatically

---

[← Previous: Production Best Practices](01-production-best-practices.md) | [Next: CI/CD with Docker →](03-ci-cd-with-docker.md)
