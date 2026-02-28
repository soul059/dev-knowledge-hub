# Chapter 4: Container Logging

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Docker captures the **STDOUT** and **STDERR** of a container's main process and makes it available through logging drivers. Understanding Docker's logging architecture is essential for monitoring, debugging, and operating containers in production.

---

## 4.1 How Docker Logging Works

```
  Container (PID 1)
  +-----------------------+
  |  Application Process  |
  |                       |
  |  STDOUT ──────────────+──┐
  |  STDERR ──────────────+──┤
  +-----------------------+  |
                              |
                              v
                    +-------------------+
                    |   Docker Daemon   |
                    |   Log Driver      |
                    +-------------------+
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
        +-----------+   +-----------+   +-----------+
        | json-file |   |  syslog   |   | fluentd   |
        | (default) |   |           |   |           |
        +-----------+   +-----------+   +-----------+
```

**Key principle**: Docker only captures STDOUT and STDERR from PID 1. Application logs written to files inside the container are NOT captured by Docker logging.

```dockerfile
# GOOD: Log to STDOUT/STDERR
CMD ["node", "server.js"]
# Application should use: console.log(), console.error()

# Common pattern for nginx:
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log
# Symlinks log files to stdout/stderr
```

---

## 4.2 Log Drivers

```bash
# Check current default log driver
docker info --format '{{.LoggingDriver}}'

# Set default in daemon.json
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}

# Override per container
docker run -d --log-driver=syslog nginx
docker run -d --log-driver=none nginx     # Disable logging
```

### Available Log Drivers

```
  +---------------+-----------------------------------------+-----------+
  | Driver        | Description                             | docker    |
  |               |                                         | logs?     |
  +---------------+-----------------------------------------+-----------+
  | json-file     | JSON format, local file (DEFAULT)       | Yes       |
  | local         | Optimized local storage                 | Yes       |
  | journald      | systemd journal                         | Yes       |
  | syslog        | Syslog daemon                           | No        |
  | fluentd       | Fluentd collector                       | No        |
  | awslogs       | AWS CloudWatch Logs                     | No        |
  | gcplogs       | Google Cloud Logging                    | No        |
  | splunk        | Splunk HTTP Event Collector             | No        |
  | gelf          | Graylog Extended Log Format             | No        |
  | none          | No logging                              | No        |
  +---------------+-----------------------------------------+-----------+
  
  NOTE: Only json-file, local, and journald support
  the `docker logs` command!
```

---

## 4.3 json-file Driver (Default)

```bash
# Default configuration
docker run -d nginx

# With options
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  --log-opt compress=true \
  nginx

# Log file location (Linux)
# /var/lib/docker/containers/<container-id>/<container-id>-json.log
```

```json
// Log format (one JSON object per line):
{"log":"172.17.0.1 - - GET /index.html 200\n","stream":"stdout","time":"2024-01-15T10:30:00.123456789Z"}
{"log":"Error: connection refused\n","stream":"stderr","time":"2024-01-15T10:30:01.234567890Z"}
```

### Log Rotation Options

```
  +------------------+-----------------------------------+
  | Option           | Description                       |
  +------------------+-----------------------------------+
  | max-size         | Max size before rotation           |
  |                  | (e.g., 10m, 100k, 1g)            |
  | max-file         | Max number of log files            |
  | compress         | Compress rotated files (gzip)      |
  | labels           | Add container labels to log        |
  | env              | Add env vars to log                |
  +------------------+-----------------------------------+
  
  IMPORTANT: Without max-size and max-file,
  logs grow UNBOUNDED and can fill disk!
  
  ALWAYS set rotation in production:
  --log-opt max-size=10m --log-opt max-file=3
```

---

## 4.4 docker logs Command

```bash
# Basic usage
docker logs mycontainer

# Follow (stream) logs
docker logs -f mycontainer

# Last N lines
docker logs --tail 100 mycontainer

# Since time
docker logs --since 2024-01-15T10:00:00 mycontainer
docker logs --since 30m mycontainer
docker logs --since 1h mycontainer

# Until time
docker logs --until 2024-01-15T11:00:00 mycontainer

# Show timestamps
docker logs -t mycontainer

# Combine options
docker logs -f --tail 50 -t mycontainer

# Pipe to tools
docker logs mycontainer 2>&1 | grep ERROR
docker logs mycontainer 2>&1 | wc -l
docker logs mycontainer 2>&1 | tail -20
```

---

## 4.5 Application Logging Best Practices

### Direct STDOUT/STDERR

```dockerfile
# Node.js — logs go to stdout by default
CMD ["node", "server.js"]

# Python — force unbuffered output
ENV PYTHONUNBUFFERED=1
CMD ["python", "app.py"]

# Nginx — redirect logs to stdout/stderr
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

# Apache — similar approach
RUN ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log
```

### Structured Logging (JSON)

```
  Unstructured:                    Structured (JSON):
  
  "User login failed             {"timestamp":"2024-01-15T10:30:00Z",
   at 10:30:00"                    "level":"error",
                                   "message":"User login failed",
  Hard to parse,                   "user_id":"12345",
  hard to search                   "ip":"192.168.1.100"}
  
                                  Easy to parse,
                                  searchable, filterable
```

### 12-Factor App Logging

```
  +-------------------+
  | Application       |
  | (writes to stdout)|
  +--------+----------+
           |
           v
  +--------+----------+
  | Docker Log Driver  |
  | (captures stream)  |
  +--------+----------+
           |
           v
  +--------+----------+
  | Log Aggregator     |
  | (ELK, Loki, etc.) |
  +--------+----------+
           |
           v
  +--------+----------+
  | Dashboard          |
  | (Kibana, Grafana)  |
  +-------------------+
  
  Principle: Application should NEVER manage
  log routing, storage, or rotation.
```

---

## 4.6 Production Logging with Fluentd

```bash
# Run Fluentd container
docker run -d \
  --name fluentd \
  -p 24224:24224 \
  -v /fluentd/conf:/fluentd/etc \
  fluent/fluentd:v1.16

# Run app containers with fluentd driver
docker run -d \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag="app.{{.Name}}" \
  myapp
```

```
  Centralized Logging Architecture:
  
  +--------+  +--------+  +--------+
  | App 1  |  | App 2  |  | App 3  |   Containers
  +---+----+  +---+----+  +---+----+
      |           |           |
      +-----+-----+-----+-----+
            |             |
            v             v
      +----------+  +----------+
      | Fluentd  |  | Fluentd  |   Collectors
      +----+-----+  +-----+----+
           |               |
           +-------+-------+
                   |
                   v
            +-----------+
            | Elastic   |              Storage
            | Search    |
            +-----+-----+
                  |
                  v
            +-----------+
            | Kibana    |              Dashboard
            +-----------+
```

---

## 4.7 Log Aggregation Options

| Stack | Components | Best For |
|-------|-----------|----------|
| **ELK** | Elasticsearch + Logstash + Kibana | Full-featured, enterprise |
| **EFK** | Elasticsearch + Fluentd + Kibana | Kubernetes-native |
| **Loki + Grafana** | Loki + Promtail + Grafana | Lightweight, label-based |
| **Datadog** | Agent + Dashboard | SaaS, all-in-one |
| **CloudWatch** | AWS native | AWS environments |

---

## 4.8 Troubleshooting Logs

```bash
# Container produces no logs
docker inspect -f '{{.HostConfig.LogConfig.Type}}' mycontainer
# Check if log driver is "none"

# Logs are too large
docker system df -v
# Check container log sizes

# Find large log files (Linux)
find /var/lib/docker/containers -name "*-json.log" -size +100M

# Truncate a log file (emergency)
truncate -s 0 /var/lib/docker/containers/<id>/<id>-json.log

# Check if app is logging to file instead of stdout
docker exec mycontainer find /var/log -name "*.log"
```

---

## Summary Table

| Topic | Description | Key Points |
|-------|-------------|------------|
| **Log Source** | STDOUT/STDERR of PID 1 | Apps must log to stdout |
| **Default Driver** | json-file | Supports `docker logs` |
| **Log Rotation** | max-size, max-file | MUST configure in production |
| **docker logs** | View container logs | `-f`, `--tail`, `--since` |
| **Structured Logging** | JSON format | Easier parsing and searching |
| **Centralized Logging** | Fluentd/ELK/Loki | Production requirement |
| **12-Factor** | App logs to stdout only | Platform handles routing |

---

## Quick Revision Questions

1. **What does Docker capture from a container for logging?**
   - Docker captures STDOUT and STDERR from the container's main process (PID 1). Log files written inside the container are NOT captured

2. **What is the default log driver and what are its risks?**
   - The default is `json-file`. Without log rotation settings (`max-size`, `max-file`), logs grow unbounded and can fill up the host disk

3. **Which log drivers support `docker logs` command?**
   - Only `json-file`, `local`, and `journald` support `docker logs`. Other drivers (syslog, fluentd, awslogs) do not

4. **Why should applications log to STDOUT rather than files?**
   - Following the 12-Factor App methodology, applications should treat logs as event streams. The platform (Docker, Kubernetes) handles log routing, aggregation, and storage

5. **How do you redirect Nginx logs to Docker's logging system?**
   - Create symlinks: `ln -sf /dev/stdout /var/log/nginx/access.log` and `ln -sf /dev/stderr /var/log/nginx/error.log`

6. **What is the purpose of structured (JSON) logging?**
   - Structured logs are machine-parseable, making them easy to search, filter, and analyze in log aggregation systems like ELK or Loki

---

[← Previous: Container Inspection](03-container-inspection.md) | [Next: Container Networking Basics →](05-container-networking-basics.md)
