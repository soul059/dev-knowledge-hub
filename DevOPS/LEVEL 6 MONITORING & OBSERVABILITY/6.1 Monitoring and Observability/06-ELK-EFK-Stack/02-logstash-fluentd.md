# Chapter 2: Logstash & Fluentd

[← Previous: Elasticsearch Concepts](01-elasticsearch-concepts.md) | [Back to README](../README.md) | [Next: Kibana →](03-kibana.md)

---

## Overview

Logstash and Fluentd are log processing pipelines that collect, parse, transform, and route log data. Logstash is the "L" in ELK; Fluentd replaces it in the EFK stack.

---

## 2.1 Pipeline Architecture

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  LOGSTASH PIPELINE:     Input → Filter → Output         │
│  FLUENTD PIPELINE:      Source → Filter → Match(Output)  │
│                                                          │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐            │
│  │ INPUT    │───▶│ FILTER   │───▶│ OUTPUT   │            │
│  │          │    │          │    │          │            │
│  │ beats    │    │ grok     │    │ elastic  │            │
│  │ syslog   │    │ mutate   │    │ s3       │            │
│  │ kafka    │    │ date     │    │ kafka    │            │
│  │ file     │    │ geoip    │    │ stdout   │            │
│  │ http     │    │ json     │    │ loki     │            │
│  └─────────┘    └──────────┘    └──────────┘            │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 Logstash Configuration

```ruby
# /etc/logstash/conf.d/pipeline.conf

input {
  beats {
    port => 5044
  }
  
  kafka {
    bootstrap_servers => "kafka:9092"
    topics => ["app-logs"]
    group_id => "logstash-consumers"
    codec => json
  }
}

filter {
  # Parse JSON logs
  json {
    source => "message"
  }
  
  # Parse unstructured logs with grok
  if [type] == "nginx" {
    grok {
      match => {
        "message" => '%{IPORHOST:client_ip} - %{DATA:user} \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:bytes}'
      }
    }
    
    # Parse date
    date {
      match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
      target => "@timestamp"
    }
    
    # GeoIP lookup
    geoip {
      source => "client_ip"
    }
  }
  
  # Add/modify fields
  mutate {
    add_field => { "environment" => "production" }
    remove_field => ["host", "agent"]
    rename => { "level" => "log_level" }
    uppercase => ["log_level"]
  }
  
  # Drop health check logs
  if [request] =~ /health/ {
    drop { }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+yyyy.MM.dd}"
    user => "elastic"
    password => "${ES_PASSWORD}"
  }
  
  # Also send errors to a separate index
  if [log_level] == "ERROR" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "errors-%{+yyyy.MM.dd}"
    }
  }
}
```

---

## 2.3 Fluentd Configuration

```xml
<!-- /etc/fluentd/fluent.conf -->

<!-- Input: Receive from Fluent Bit forwarders -->
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<!-- Input: Tail log files -->
<source>
  @type tail
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.pos
  tag kubernetes.*
  <parse>
    @type json
    time_key time
    time_format %Y-%m-%dT%H:%M:%S.%NZ
  </parse>
</source>

<!-- Filter: Parse JSON from log field -->
<filter kubernetes.**>
  @type parser
  key_name log
  reserve_data true
  <parse>
    @type json
  </parse>
</filter>

<!-- Filter: Add custom fields -->
<filter kubernetes.**>
  @type record_transformer
  <record>
    cluster "production-us-east"
    processed_at ${Time.now.iso8601}
  </record>
</filter>

<!-- Filter: Drop health checks -->
<filter kubernetes.**>
  @type grep
  <exclude>
    key $.kubernetes.container_name
    pattern /health-check/
  </exclude>
</filter>

<!-- Output: Send to Elasticsearch -->
<match kubernetes.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix k8s-logs
  
  <buffer>
    @type file
    path /var/log/fluentd-buffer
    flush_interval 5s
    chunk_limit_size 8MB
    retry_max_interval 30s
    retry_forever true
  </buffer>
</match>

<!-- Output: Send audit logs to S3 -->
<match audit.**>
  @type s3
  s3_bucket audit-logs
  s3_region us-east-1
  path logs/%Y/%m/%d/
  
  <buffer time>
    @type file
    path /var/log/fluentd-s3-buffer
    timekey 3600
    timekey_wait 10m
  </buffer>
</match>
```

---

## 2.4 Logstash vs Fluentd Comparison

| Feature | Logstash | Fluentd |
|---------|----------|---------|
| Language | Java (JRuby) | Ruby (C core) |
| Memory usage | ~500 MB | ~40 MB |
| Configuration | Custom DSL | XML-like tags |
| Plugin ecosystem | 200+ | 800+ |
| Buffering | In-memory + persistent queue | File/memory buffer |
| Best paired with | Elasticsearch (Elastic stack) | Kubernetes, CNCF |
| Performance | Good with tuning | Lightweight, efficient |
| Community | Elastic (commercial) | CNCF (open source) |

---

## 2.5 Grok Patterns (Logstash)

```
# Common grok patterns for log parsing

APACHE: %{COMBINEDAPACHELOG}
→ Parses: 127.0.0.1 - frank [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326

SYSLOG: %{SYSLOGLINE}
→ Parses: Jan 15 10:23:45 hostname sshd[1234]: message

Custom pattern example:
%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}
→ Parses: 2024-01-15T10:23:45 ERROR Something went wrong

# Test grok patterns at: https://grokdebug.herokuapp.com/
# Or with: bin/logstash --config.test_and_exit
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Logstash OOM | Java heap too small | Increase `-Xms` and `-Xmx` in jvm.options |
| Grok parse failures | Pattern doesn't match log format | Test with grok debugger, add `_grokparsefailure` tag handling |
| Fluentd buffer overflow | Output slower than input | Increase buffer size, add more workers |
| Duplicate events | At-least-once delivery | Use document `_id` based on content hash |
| Pipeline backpressure | Slow output destination | Add persistent queue, monitor queue size |

---

## Quick Revision Questions

1. **What are the three stages of a Logstash pipeline?**
2. **How does Fluentd's architecture differ from Logstash?**
3. **What is grok and how is it used to parse unstructured logs?**
4. **Why might you choose Fluentd over Logstash in Kubernetes?**
5. **How do you handle backpressure in a log pipeline?**
6. **How do you route different log types to different outputs?**

---

[← Previous: Elasticsearch Concepts](01-elasticsearch-concepts.md) | [Back to README](../README.md) | [Next: Kibana →](03-kibana.md)
