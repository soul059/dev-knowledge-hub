# Chapter 2: Prometheus Installation and Configuration

[← Previous: Architecture](01-prometheus-architecture.md) | [Back to README](../README.md) | [Next: PromQL Basics →](03-promql-basics.md)

---

## Overview

This chapter covers installing Prometheus via different methods and understanding the core configuration file.

---

## 2.1 Installation Methods

### Binary Installation

```bash
# Download and extract
wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz
tar xvf prometheus-2.50.0.linux-amd64.tar.gz
cd prometheus-2.50.0.linux-amd64

# Start
./prometheus --config.file=prometheus.yml
```

### Docker Installation

```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v prometheus_data:/prometheus \
  prom/prometheus:v2.50.0
```

### Docker Compose (Full Stack)

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:v2.50.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert_rules.yml:/etc/prometheus/alert_rules.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.27.0
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

  grafana:
    image: grafana/grafana:10.3.0
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter:v1.7.0
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
  grafana_data:
```

### Kubernetes (Helm)

```bash
# Add Prometheus community Helm chart
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set grafana.adminPassword=admin
```

---

## 2.2 Core Configuration File

```yaml
# prometheus.yml — Complete annotated example

# Global settings (apply to all scrape configs unless overridden)
global:
  scrape_interval: 15s          # How often to scrape targets
  evaluation_interval: 15s      # How often to evaluate rules
  scrape_timeout: 10s           # Timeout per scrape

  # Labels added to all time series and alerts
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

# Alert rule files
rule_files:
  - 'recording_rules.yml'
  - 'alert_rules.yml'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 'alertmanager:9093'

# Scrape configurations — what to monitor
scrape_configs:
  # Monitor Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets:
          - 'node-exporter:9100'
    scrape_interval: 30s  # Override global for this job

  # Application metrics
  - job_name: 'myapp'
    metrics_path: '/metrics'       # Default is /metrics
    scheme: 'http'                  # Default is http
    static_configs:
      - targets:
          - 'app1:8080'
          - 'app2:8080'
        labels:
          environment: 'production'  # Extra labels for all targets

  # Application with basic auth
  - job_name: 'secure-app'
    basic_auth:
      username: 'prometheus'
      password: 'secret'
    static_configs:
      - targets: ['secure-app:8443']
    scheme: 'https'
    tls_config:
      insecure_skip_verify: true
```

---

## 2.3 Key Startup Flags

```bash
./prometheus \
  --config.file=prometheus.yml \
  --storage.tsdb.path=/data/prometheus \
  --storage.tsdb.retention.time=30d \
  --storage.tsdb.retention.size=50GB \
  --web.enable-lifecycle \            # Enable /-/reload endpoint
  --web.enable-admin-api \            # Enable admin API (snapshots, etc.)
  --web.listen-address=0.0.0.0:9090
```

| Flag | Purpose | Default |
|------|---------|---------|
| `--config.file` | Config file path | `prometheus.yml` |
| `--storage.tsdb.path` | Data directory | `data/` |
| `--storage.tsdb.retention.time` | How long to keep data | `15d` |
| `--storage.tsdb.retention.size` | Max storage size | unlimited |
| `--web.enable-lifecycle` | Enable config reload via API | disabled |
| `--web.listen-address` | Listen address | `0.0.0.0:9090` |

---

## 2.4 Verifying the Setup

```bash
# Check Prometheus is running
curl http://localhost:9090/-/healthy
# Response: Prometheus Server is Healthy.

# Check config is valid
./promtool check config prometheus.yml

# Check targets
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'

# Reload config without restart
curl -X POST http://localhost:9090/-/reload
# Or send SIGHUP: kill -HUP $(pidof prometheus)
```

---

## Summary Table

| Method | Best For | Complexity |
|--------|----------|------------|
| **Binary** | Development, testing | Low |
| **Docker** | Single-server deployments | Low |
| **Docker Compose** | Full stack (Prom+Grafana+AM) | Medium |
| **Helm/K8s** | Production Kubernetes | Medium-High |
| **Operator** | Large-scale K8s | High |

---

## Quick Revision Questions

1. **What are the three main sections of a `prometheus.yml` configuration file?**
2. **How do you reload Prometheus configuration without restarting the server?**
3. **What is the default scrape interval and retention period?**
4. **Write a scrape config for monitoring a Python app at `myapp:5000/metrics`.**
5. **What command validates a Prometheus config file before applying it?**

---

[← Previous: Architecture](01-prometheus-architecture.md) | [Back to README](../README.md) | [Next: PromQL Basics →](03-promql-basics.md)
