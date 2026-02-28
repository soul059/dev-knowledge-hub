# Chapter 9.4: AWS X-Ray — Distributed Tracing

## Overview

AWS X-Ray helps you **analyze and debug distributed applications** by tracing requests as they travel through your services. It provides a visual service map, identifies performance bottlenecks, and pinpoints errors across microservices architectures.

---

## X-Ray Architecture

```
┌──── X-Ray Tracing Flow ──────────────────────────────────┐
│                                                           │
│  Client Request                                           │
│       │                                                   │
│       ▼                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│  │ API GW   │───→│ Lambda   │───→│ DynamoDB │            │
│  │ (Trace A)│    │ (Trace A)│    │ (Trace A)│            │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘            │
│       │               │               │                   │
│       ▼               ▼               ▼                   │
│  ┌─────────────────────────────────────────────┐         │
│  │          X-Ray Daemon / SDK                  │         │
│  │  Collects segments + subsegments             │         │
│  │  Sends to X-Ray API via UDP (port 2000)      │         │
│  └──────────────────┬──────────────────────────┘         │
│                     │                                     │
│                     ▼                                     │
│  ┌──────────────────────────────────────────┐            │
│  │           X-Ray Service                  │            │
│  │  ┌─────────────────────────────┐         │            │
│  │  │ Service Map                 │         │            │
│  │  │ ┌────┐    ┌────┐    ┌────┐ │         │            │
│  │  │ │API │───→│ Fn │───→│ DB │ │         │            │
│  │  │ │ GW │    │    │    │    │ │         │            │
│  │  │ └────┘    └────┘    └────┘ │         │            │
│  │  └─────────────────────────────┘         │            │
│  │  ┌─────────────────────────────┐         │            │
│  │  │ Trace Timeline              │         │            │
│  │  │ ├── API GW: 5ms            │         │            │
│  │  │ ├── Lambda: 120ms          │         │            │
│  │  │ │   ├── Init: 45ms         │         │            │
│  │  │ │   ├── DynamoDB: 15ms     │         │            │
│  │  │ │   └── S3 Put: 30ms      │         │            │
│  │  │ └── Total: 125ms          │         │            │
│  │  └─────────────────────────────┘         │            │
│  └──────────────────────────────────────────┘            │
└───────────────────────────────────────────────────────────┘
```

---

## Core Concepts

```
┌──── X-Ray Terminology ───────────────────────────────────┐
│                                                          │
│  Trace:      End-to-end request journey                 │
│              (one Trace ID across all services)          │
│                                                          │
│  Segment:    Work done by a single service              │
│              (e.g., Lambda function execution)           │
│                                                          │
│  Subsegment: Granular breakdown within a segment        │
│              (e.g., DynamoDB call, HTTP request)         │
│                                                          │
│  Annotations: Indexed key-value pairs                   │
│               (searchable: "user=admin")                │
│                                                          │
│  Metadata:   Non-indexed data (JSON objects)            │
│              (debug info, not searchable)                │
│                                                          │
│  Sampling:   Rules to control trace volume              │
│              (e.g., trace 5% of requests)               │
│                                                          │
│  Groups:     Filter traces by expression                │
│              (e.g., fault = true AND http.url = "/api") │
└──────────────────────────────────────────────────────────┘
```

---

## Enabling X-Ray

### Lambda

```yaml
# SAM template — enable Active tracing
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Tracing: Active
      # X-Ray SDK auto-instruments AWS SDK calls
```

```bash
# CLI — enable tracing on existing Lambda
aws lambda update-function-configuration \
  --function-name my-function \
  --tracing-config Mode=Active
```

### ECS / EC2

```bash
# Run X-Ray daemon as sidecar in ECS task definition
# Add to task definition JSON:
{
  "name": "xray-daemon",
  "image": "amazon/aws-xray-daemon",
  "cpu": 32,
  "memoryReservation": 256,
  "portMappings": [{
    "containerPort": 2000,
    "protocol": "udp"
  }]
}

# EC2 — install and run daemon
curl https://s3.us-east-1.amazonaws.com/aws-xray-assets.us-east-1/xray-daemon/aws-xray-daemon-linux-3.x.zip -o xray.zip
unzip xray.zip
./xray -o -n us-east-1
```

### API Gateway

```bash
# Enable tracing on API Gateway stage
aws apigateway update-stage \
  --rest-api-id abc123 \
  --stage-name prod \
  --patch-operations op=replace,path=/tracingEnabled,value=true
```

---

## X-Ray SDK Instrumentation (Python)

```python
# Install: pip install aws-xray-sdk

from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch all supported libraries (boto3, requests, etc.)
patch_all()

# Custom subsegment
def process_order(order_id):
    # Automatic tracing of DynamoDB call (patched)
    table = boto3.resource('dynamodb').Table('Orders')
    order = table.get_item(Key={'id': order_id})

    # Custom subsegment for business logic
    with xray_recorder.in_subsegment('process_payment') as subsegment:
        subsegment.put_annotation('order_id', order_id)
        subsegment.put_annotation('customer', order['customer_id'])
        subsegment.put_metadata('order_details', order, 'payment')

        result = charge_payment(order)
        return result
```

---

## Sampling Rules

```bash
# Create custom sampling rule
aws xray create-sampling-rule --cli-input-json '{
  "SamplingRule": {
    "RuleName": "api-sampling",
    "Priority": 1000,
    "FixedRate": 0.05,
    "ReservoirSize": 1,
    "ServiceName": "my-api",
    "ServiceType": "*",
    "Host": "*",
    "HTTPMethod": "*",
    "URLPath": "/api/*",
    "ResourceARN": "*",
    "Version": 1
  }
}'

# FixedRate: 0.05 = 5% of requests after reservoir
# ReservoirSize: 1 = first 1 request per second always traced
# Lower Priority number = higher precedence
```

```
┌──── Sampling Strategy ───────────────────────────────────┐
│                                                          │
│  Default: Reservoir=1, Rate=5%                          │
│                                                          │
│  1000 requests/second:                                   │
│  ├── 1 traced (reservoir)                               │
│  └── 999 × 5% = ~50 traced                             │
│  = ~51 traces/second (instead of 1000)                  │
│                                                          │
│  Custom rules can:                                       │
│  • Trace 100% of errors: URLPath="/error*", Rate=1.0   │
│  • Trace 1% of health checks: URLPath="/health"        │
│  • Trace 10% of API calls: URLPath="/api/*"            │
└──────────────────────────────────────────────────────────┘
```

---

## Querying Traces

```bash
# Get trace summaries (filter by annotation)
aws xray get-trace-summaries \
  --start-time $(date -d '-1 hour' +%s) \
  --end-time $(date +%s) \
  --filter-expression 'annotation.order_id = "ORD-123"'

# Filter expressions:
# service("my-api")
# responsetime > 5
# fault = true
# http.url CONTAINS "/api/orders"
# annotation.customer = "premium"
# edge("api", "database") { responsetime > 1 }
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| No traces appearing | Tracing not enabled | Enable Active tracing on Lambda/API GW |
| Missing downstream calls | SDK not patched | Call `patch_all()` or patch specific libraries |
| X-Ray daemon errors | IAM permissions missing | Add `xray:PutTraceSegments`, `xray:PutTelemetryRecords` |
| Too many traces | Sampling rate too high | Reduce `FixedRate` or `ReservoirSize` in sampling rules |
| Broken trace chain | Trace header not propagated | Ensure `X-Amzn-Trace-Id` header is forwarded between services |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **X-Ray** | Distributed tracing for microservices. Service map + traces. |
| **Segments** | Work done by one service. Subsegments for granular detail. |
| **Annotations** | Indexed key-value pairs (searchable in console). |
| **Sampling** | Controls trace volume. Reservoir + fixed rate. |
| **SDK** | Instrument code. `patch_all()` auto-traces AWS SDK calls. |
| **Daemon** | Sidecar process that buffers and sends segments to X-Ray. |

---

## Quick Revision Questions

1. **What is the difference between a segment and a subsegment?**
   > A segment represents work done by a single service. A subsegment provides granular detail within a segment (e.g., a DynamoDB call or HTTP request).

2. **What are annotations vs metadata in X-Ray?**
   > Annotations are indexed key-value strings that you can search/filter by. Metadata is non-indexed JSON data for debugging, not searchable.

3. **How does X-Ray sampling work?**
   > A reservoir guarantees N traces per second. Beyond that, a fixed rate (e.g., 5%) determines what percentage of additional requests are traced.

4. **How do you enable X-Ray for Lambda?**
   > Set `Tracing: Active` in the function configuration. The X-Ray SDK is included in the Lambda runtime.

5. **What is the X-Ray daemon?**
   > A process that listens on UDP port 2000, buffers trace segments, and sends them to the X-Ray API. Required for EC2/ECS (not needed for Lambda).

---

[← Previous: 9.3 Dashboards](03-cloudwatch-dashboards.md) | [Next: 9.5 CloudTrail →](05-cloudtrail.md)

[← Back to README](../README.md)
