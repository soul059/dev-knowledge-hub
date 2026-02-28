# Chapter 2.6: Lambda (Serverless)

## Overview

AWS Lambda lets you run code **without provisioning or managing servers**. You pay only for compute time consumed — no charge when your code isn't running. Lambda is the core of serverless architecture on AWS.

---

## How Lambda Works

```
┌──────────────────────────────────────────────────────────────┐
│                    AWS LAMBDA                                │
│                                                              │
│  EVENT ──► LAMBDA SERVICE ──► YOUR CODE ──► RESPONSE        │
│                                                              │
│  ┌──────────┐   ┌───────────────────────┐   ┌────────────┐ │
│  │  Event    │   │   Execution           │   │  Output    │ │
│  │  Source   │──►│   Environment         │──►│            │ │
│  │           │   │   ┌─────────────────┐ │   │  Response  │ │
│  │  API GW   │   │   │  Your Function  │ │   │  Logs      │ │
│  │  S3       │   │   │  (Code)         │ │   │  Metrics   │ │
│  │  SQS      │   │   │                 │ │   │            │ │
│  │  DynamoDB │   │   │  Runtime:       │ │   │            │ │
│  │  Schedule │   │   │  Python/Node/   │ │   │            │ │
│  │  ALB      │   │   │  Java/Go/.NET   │ │   │            │ │
│  └──────────┘   │   └─────────────────┘ │   └────────────┘ │
│                 │                         │                  │
│                 │  Memory: 128MB - 10GB   │                  │
│                 │  Timeout: max 15 min    │                  │
│                 │  Temp disk: 512MB-10GB  │                  │
│                 └───────────────────────────┘                │
└──────────────────────────────────────────────────────────────┘
```

---

## Lambda Configuration

| Setting | Range | Default | Notes |
|---------|-------|---------|-------|
| **Memory** | 128 MB – 10,240 MB | 128 MB | CPU scales proportionally |
| **Timeout** | 1 sec – 15 min | 3 sec | Set based on expected duration |
| **Ephemeral Storage** | 512 MB – 10 GB | 512 MB | `/tmp` directory |
| **Concurrency** | 1 – 1000+ | Account limit: 1000 | Scales automatically |
| **Environment Variables** | Key-value pairs | None | Config without code change |
| **Layers** | Up to 5 layers | None | Shared code/dependencies |
| **VPC** | Optional | No VPC | Required for private resources |

---

## Lambda Function Example (Python)

```python
# lambda_function.py
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def lambda_handler(event, context):
    """
    Process API Gateway event and store in DynamoDB
    """
    try:
        # Parse request body
        body = json.loads(event.get('body', '{}'))
        
        # Store in DynamoDB
        table.put_item(Item={
            'id': body['id'],
            'name': body['name'],
            'timestamp': context.aws_request_id
        })
        
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'message': 'Item created', 'id': body['id']})
        }
    except Exception as e:
        print(f"Error: {str(e)}")  # Goes to CloudWatch Logs
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

### Deploy with CLI

```bash
# 1. Create deployment package
zip function.zip lambda_function.py

# 2. Create the function
aws lambda create-function \
  --function-name process-orders \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::123456789012:role/LambdaExecutionRole \
  --environment "Variables={TABLE_NAME=orders,ENV=production}" \
  --memory-size 256 \
  --timeout 30

# 3. Test the function
aws lambda invoke \
  --function-name process-orders \
  --payload '{"body": "{\"id\": \"123\", \"name\": \"test\"}"}' \
  --cli-binary-format raw-in-base64-out \
  output.json

cat output.json

# 4. Update function code
zip function.zip lambda_function.py
aws lambda update-function-code \
  --function-name process-orders \
  --zip-file fileb://function.zip
```

---

## Event Sources

```
┌────────────────────────────────────────────────────────────────┐
│                LAMBDA EVENT SOURCES                            │
│                                                                │
│  SYNCHRONOUS (wait for response):                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │
│  │ API      │  │   ALB    │  │ Cognito  │  │ CloudFront   │ │
│  │ Gateway  │  │          │  │          │  │ (Lambda@Edge)│ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘ │
│                                                                │
│  ASYNCHRONOUS (fire and forget):                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │
│  │    S3    │  │   SNS    │  │CloudWatch│  │  EventBridge │ │
│  │  Events  │  │          │  │  Events  │  │              │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘ │
│                                                                │
│  POLL-BASED (Lambda polls the source):                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │   SQS    │  │ DynamoDB │  │ Kinesis  │                   │
│  │  Queue   │  │ Streams  │  │ Streams  │                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
└────────────────────────────────────────────────────────────────┘
```

### Common Event Source Mappings

```bash
# S3 trigger (async)
aws lambda add-permission \
  --function-name process-images \
  --statement-id s3-permission \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-images-bucket

aws s3api put-bucket-notification-configuration \
  --bucket my-images-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
      "LambdaFunctionArn": "arn:aws:lambda:...:function:process-images",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {"Key": {"FilterRules": [{"Name": "suffix", "Value": ".jpg"}]}}
    }]
  }'

# SQS trigger (poll-based)
aws lambda create-event-source-mapping \
  --function-name process-orders \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:order-queue \
  --batch-size 10

# Scheduled trigger (EventBridge/CloudWatch)
aws events put-rule \
  --name daily-cleanup \
  --schedule-expression "cron(0 2 * * ? *)"

aws events put-targets \
  --rule daily-cleanup \
  --targets "Id=1,Arn=arn:aws:lambda:...:function:cleanup"
```

---

## Lambda Versions and Aliases

```
┌───────────────────────────────────────────────────────────┐
│            VERSIONS AND ALIASES                           │
│                                                           │
│  $LATEST ──► Mutable, always latest code                 │
│                                                           │
│  Version 1 ──► Immutable snapshot                        │
│  Version 2 ──► Immutable snapshot                        │
│  Version 3 ──► Immutable snapshot (latest published)     │
│                                                           │
│  Aliases (pointers to versions):                         │
│  "prod"  ──────────────► Version 2  (stable)             │
│  "staging" ────────────► Version 3  (testing)            │
│                                                           │
│  Weighted Alias (Canary):                                │
│  "prod" ──► 90% → Version 2                             │
│         ──► 10% → Version 3  (canary testing!)           │
└───────────────────────────────────────────────────────────┘
```

```bash
# Publish a version
aws lambda publish-version \
  --function-name process-orders \
  --description "v1.2 - added validation"

# Create alias pointing to version
aws lambda create-alias \
  --function-name process-orders \
  --name prod \
  --function-version 2

# Canary deployment with weighted alias
aws lambda update-alias \
  --function-name process-orders \
  --name prod \
  --routing-config '{"AdditionalVersionWeights": {"3": 0.1}}'
  # 90% to version 2 (main), 10% to version 3 (canary)
```

---

## Lambda Concurrency

```
┌──────────────────────────────────────────────────────────────┐
│              LAMBDA CONCURRENCY MODEL                        │
│                                                              │
│  Account Limit: 1000 concurrent executions (default)        │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────────────────────┐  │
│  │  UNRESERVED     │  │  RESERVED CONCURRENCY            │  │
│  │  (shared pool)  │  │  (guaranteed for this function)  │  │
│  │                 │  │                                  │  │
│  │  All functions  │  │  Function A: 100 reserved        │  │
│  │  compete for    │  │  Function B: 200 reserved        │  │
│  │  remaining      │  │                                  │  │
│  └─────────────────┘  └──────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  PROVISIONED CONCURRENCY                              │   │
│  │  (pre-warmed instances — NO cold start!)              │   │
│  │                                                        │   │
│  │  Keeps N instances warm and ready                     │   │
│  │  Costs: you pay for provisioned even if unused        │   │
│  │  Use for: latency-sensitive production functions      │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### Cold Start Problem

```
┌─────────────────────────────────────────────────────────────┐
│              COLD START vs WARM START                       │
│                                                             │
│  COLD START (first invocation or after idle):              │
│  ┌──────┐  ┌───────┐  ┌──────┐  ┌────────┐  ┌──────────┐ │
│  │Create│─►│Download│─►│Init  │─►│Handler │─►│ Response │ │
│  │Env   │  │ Code   │  │Runtime│  │Execution│  │          │ │
│  └──────┘  └───────┘  └──────┘  └────────┘  └──────────┘ │
│  ←── Cold start latency ──────►←── Your code ───────────► │
│      (100ms - 10s)                                         │
│                                                             │
│  WARM START (reused environment):                          │
│  ┌────────┐  ┌──────────┐                                  │
│  │Handler │─►│ Response │  ← Much faster!                 │
│  │Execution│  │          │                                  │
│  └────────┘  └──────────┘                                  │
│                                                             │
│  Cold start mitigation:                                    │
│  • Use Provisioned Concurrency                             │
│  • Keep function package small                             │
│  • Use lightweight runtimes (Python/Node > Java)          │
│  • Initialize SDK clients outside handler                  │
│  • Avoid VPC unless needed (adds ~1s)                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Lambda Best Practices

```
✓ Keep functions small and focused (single responsibility)
✓ Initialize SDK clients OUTSIDE the handler (reuse on warm starts)
✓ Use environment variables for configuration
✓ Set appropriate memory (more memory = more CPU = faster)
✓ Set appropriate timeout (don't use max 15 min unless needed)
✓ Use Lambda Layers for shared dependencies
✓ Use Dead Letter Queues (DLQ) for failed async invocations
✓ Use X-Ray for tracing and debugging
✓ Use Powertools for Lambda (logging, tracing, metrics)
✗ Don't use Lambda for long-running tasks (>15 min)
✗ Don't store state in Lambda (use DynamoDB/S3)
✗ Don't use Lambda for constant high-volume processing (use ECS)
```

---

## Serverless Architecture Pattern

```
┌──────────────────────── Serverless Web App ────────────────────┐
│                                                                 │
│  Client ──► CloudFront ──► S3 (Static Frontend)               │
│                │                                                │
│                └──► API Gateway ──► Lambda ──► DynamoDB        │
│                                      │                         │
│                                      ├──► S3 (file storage)   │
│                                      ├──► SES (email)         │
│                                      └──► SNS (notifications) │
│                                                                 │
│  Authentication: Cognito                                       │
│  Monitoring: CloudWatch + X-Ray                                │
│  CI/CD: SAM / CodePipeline                                     │
│                                                                 │
│  Cost: Pay only when users are active!                         │
│  Scaling: Automatic — 0 to thousands of concurrent requests   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Timeout errors | Function takes too long | Increase timeout; optimize code; check external calls |
| Out of memory | Memory too low for workload | Increase memory (also increases CPU) |
| Cold start latency | New execution environment | Use Provisioned Concurrency; optimize package size |
| `ResourceNotFoundException` | Wrong function name/region | Verify ARN, region, and alias |
| Permission denied to DynamoDB/S3 | Missing IAM permissions in role | Add required actions to Lambda execution role |
| Function throttled | Concurrency limit reached | Request limit increase; use reserved concurrency |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Lambda** | Serverless compute. Pay per invocation + duration. Max 15 min timeout. |
| **Event Sources** | Sync (API GW, ALB), Async (S3, SNS), Poll (SQS, DynamoDB Streams). |
| **Versions** | Immutable snapshots. $LATEST is mutable. |
| **Aliases** | Named pointers to versions. Support weighted routing for canary deploys. |
| **Concurrency** | Default 1000/account. Reserved = guaranteed. Provisioned = pre-warmed. |
| **Cold Start** | First invocation latency. Mitigate with Provisioned Concurrency and small packages. |
| **Best Practices** | Single responsibility, init outside handler, env vars for config, DLQ for failures. |

---

## Quick Revision Questions

1. **What is the maximum execution time for a Lambda function?**
   > 15 minutes. For longer tasks, use Step Functions or ECS.

2. **How does Lambda pricing work?**
   > You pay for: number of requests ($0.20/million) + duration ($/GB-second of memory allocated). No charge when idle.

3. **What is a cold start and how do you prevent it?**
   > Latency on first invocation when Lambda creates a new execution environment. Prevent with Provisioned Concurrency.

4. **Why should you initialize SDK clients outside the handler function?**
   > Outside-handler code persists between invocations (warm starts), so the client is reused, saving initialization time.

5. **What is the difference between Reserved and Provisioned Concurrency?**
   > Reserved guarantees capacity but still has cold starts. Provisioned pre-warms instances so there are zero cold starts.

6. **Name three common event sources for Lambda.**
   > API Gateway (sync), S3 events (async), SQS queues (poll-based).

---

[← Previous: 2.5 Elastic Load Balancing](05-elastic-load-balancing.md) | [Next: Unit 3 — VPC Concepts →](../03-Networking-VPC/01-vpc-concepts.md)

[← Back to README](../README.md)
