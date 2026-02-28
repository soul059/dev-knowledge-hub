# Chapter 5: ELK Stack Deployment

[← Previous: Filebeat & Agents](04-filebeat-agents.md) | [Back to README](../README.md) | [Next: Index Management →](06-index-management.md)

---

## Overview

Deploying the ELK stack in production requires careful planning for high availability, storage, and performance. This chapter covers Docker Compose for development and Kubernetes/Helm for production.

---

## 5.1 Development: Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    healthcheck:
      test: curl -s http://localhost:9200 >/dev/null || exit 1
      interval: 30s
      timeout: 10s
      retries: 5

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    ports:
      - "5044:5044"     # Beats input
      - "9600:9600"     # Monitoring API
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    environment:
      - LS_JAVA_OPTS=-Xms512m -Xmx512m
    depends_on:
      elasticsearch:
        condition: service_healthy

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      elasticsearch:
        condition: service_healthy

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    depends_on:
      elasticsearch:
        condition: service_healthy

volumes:
  es-data:
    driver: local
```

---

## 5.2 Production: Kubernetes with Helm

```bash
# Add Elastic Helm repo
helm repo add elastic https://helm.elastic.co
helm repo update

# Install Elasticsearch (3-node cluster)
helm install elasticsearch elastic/elasticsearch \
  --namespace elastic --create-namespace \
  --set replicas=3 \
  --set minimumMasterNodes=2 \
  --set resources.requests.memory=4Gi \
  --set resources.limits.memory=4Gi \
  --set volumeClaimTemplate.resources.requests.storage=100Gi \
  --set persistence.enabled=true

# Install Kibana
helm install kibana elastic/kibana \
  --namespace elastic \
  --set service.type=LoadBalancer

# Install Filebeat (DaemonSet)
helm install filebeat elastic/filebeat \
  --namespace elastic
```

### Custom Values for ES

```yaml
# elasticsearch-values.yaml
replicas: 3
minimumMasterNodes: 2

resources:
  requests:
    cpu: "2"
    memory: "4Gi"
  limits:
    cpu: "4"
    memory: "4Gi"

volumeClaimTemplate:
  resources:
    requests:
      storage: 200Gi
  storageClassName: gp3

esJavaOpts: "-Xms2g -Xmx2g"

esConfig:
  elasticsearch.yml: |
    cluster.name: "production-logs"
    xpack.security.enabled: true
    xpack.security.transport.ssl.enabled: true

antiAffinity: "hard"   # Spread across nodes

nodeSelector:
  node-type: elasticsearch
```

---

## 5.3 Production Architecture

```
┌──────────────────────────────────────────────────────────┐
│           PRODUCTION ELK ARCHITECTURE                    │
│                                                          │
│  ┌─────┐ ┌─────┐ ┌─────┐                               │
│  │App 1│ │App 2│ │App N│   Applications                 │
│  └──┬──┘ └──┬──┘ └──┬──┘                               │
│     └───────┼───────┘                                    │
│             ▼                                            │
│  ┌────────────────────┐                                  │
│  │ Filebeat DaemonSet  │  Collection tier                │
│  └────────┬───────────┘                                  │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────┐                                  │
│  │ Kafka (optional)    │  Buffering tier                 │
│  │ 3-broker cluster    │                                 │
│  └────────┬───────────┘                                  │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────┐                                  │
│  │ Logstash (3 nodes)  │  Processing tier                │
│  │ Parse + Enrich      │                                 │
│  └────────┬───────────┘                                  │
│           │                                              │
│           ▼                                              │
│  ┌──────────────────────────────────────────────┐       │
│  │ Elasticsearch Cluster                         │       │
│  │ ┌────────┐ ┌────────┐ ┌────────┐             │       │
│  │ │Master 1│ │Master 2│ │Master 3│ Dedicated   │       │
│  │ └────────┘ └────────┘ └────────┘ masters     │       │
│  │ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐         │       │
│  │ │Data 1│ │Data 2│ │Data 3│ │Data N│ Data     │       │
│  │ │(Hot) │ │(Hot) │ │(Warm)│ │(Cold)│ nodes    │       │
│  │ └──────┘ └──────┘ └──────┘ └──────┘         │       │
│  │ ┌────────────┐ ┌────────────┐                │       │
│  │ │Coord Node 1│ │Coord Node 2│ Coordinating  │       │
│  │ └────────────┘ └────────────┘                │       │
│  └──────────────────────────────────────────────┘       │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────┐                                  │
│  │ Kibana (2 replicas) │  Visualization tier             │
│  │ behind load balancer│                                 │
│  └────────────────────┘                                  │
└──────────────────────────────────────────────────────────┘
```

---

## 5.4 Sizing Guidelines

| Component | Small (<50 GB/day) | Medium (50-500 GB/day) | Large (>500 GB/day) |
|-----------|-------------------|----------------------|---------------------|
| ES Masters | 3 × 2 CPU, 4 GB | 3 × 4 CPU, 8 GB | 3 × 8 CPU, 16 GB |
| ES Data (Hot) | 3 × 4 CPU, 16 GB | 5 × 8 CPU, 32 GB | 10+ × 16 CPU, 64 GB |
| ES Data (Warm) | — | 3 × 4 CPU, 16 GB | 5+ × 8 CPU, 32 GB |
| Logstash | 2 × 2 CPU, 4 GB | 3 × 4 CPU, 8 GB | 5+ × 8 CPU, 16 GB |
| Kibana | 1 × 2 CPU, 4 GB | 2 × 4 CPU, 8 GB | 3 × 4 CPU, 8 GB |
| Storage/node | 500 GB SSD | 2 TB SSD | 4 TB SSD |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ES won't start | JVM heap too small | Set `-Xms` and `-Xmx` to 50% of RAM (max 32 GB) |
| Split brain | Odd number of masters needed | Use 3 dedicated master nodes |
| Cluster RED on deploy | Shards not allocated | Wait for recovery, check disk watermarks |
| Kibana can't connect | ES not ready yet | Use health check dependency in compose |
| Performance degraded | Too many small shards | Aim for 20-50 GB per shard |

---

## Quick Revision Questions

1. **What are the main tiers in a production ELK deployment?**
2. **Why do you need 3 dedicated master nodes in production?**
3. **How do you size Elasticsearch JVM heap?**
4. **What is the ideal shard size and how does it affect performance?**
5. **How would you deploy the ELK stack on Kubernetes using Helm?**
6. **What role does Kafka play in an ELK pipeline?**

---

[← Previous: Filebeat & Agents](04-filebeat-agents.md) | [Back to README](../README.md) | [Next: Index Management →](06-index-management.md)
