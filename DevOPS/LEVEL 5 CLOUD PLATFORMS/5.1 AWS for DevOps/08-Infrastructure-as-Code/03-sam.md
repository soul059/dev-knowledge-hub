# Chapter 8.3: AWS SAM — Serverless Application Model

## Overview

AWS SAM (Serverless Application Model) is an **open-source framework** for building serverless applications. It extends CloudFormation with simplified syntax for Lambda functions, API Gateway APIs, DynamoDB tables, and Step Functions. SAM CLI provides local testing and debugging.

---

## SAM Architecture

```
┌──── SAM Workflow ────────────────────────────────────────┐
│                                                           │
│  SAM Template            SAM CLI               AWS        │
│  (template.yaml)                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌────────────┐  │
│  │ AWS::Server- │    │ sam build    │    │ Lambda     │  │
│  │ less::Function│──→│ sam local    │──→│ API GW     │  │
│  │              │    │ sam deploy   │    │ DynamoDB   │  │
│  │ Transform:   │    │              │    │ S3         │  │
│  │ AWS::Server- │    │ Local test   │    │ SQS/SNS   │  │
│  │ less-2016..  │    │ & debug      │    │ Step Fn    │  │
│  └──────────────┘    └──────────────┘    └────────────┘  │
│                                                           │
│  SAM = CloudFormation + Serverless Transform              │
│  ✓ Simplified resource types                             │
│  ✓ Local invoke and debugging                            │
│  ✓ Built-in best practices (tracing, layers, etc.)       │
└───────────────────────────────────────────────────────────┘
```

---

## SAM Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: SAM REST API with DynamoDB

Globals:
  Function:
    Runtime: python3.12
    Timeout: 30
    MemorySize: 256
    Tracing: Active
    Environment:
      Variables:
        TABLE_NAME: !Ref ItemsTable
        STAGE: !Ref Stage

Parameters:
  Stage:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Resources:
  # ─── API Gateway ───
  ApiGateway:
    Type: AWS::Serverless::Api
    Properties:
      StageName: !Ref Stage
      TracingEnabled: true
      Cors:
        AllowMethods: "'GET,POST,PUT,DELETE'"
        AllowHeaders: "'Content-Type,Authorization'"
        AllowOrigin: "'*'"

  # ─── Lambda Functions ───
  GetItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handlers/get_item.handler
      CodeUri: src/
      Description: Get a single item by ID
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref ItemsTable
      Events:
        GetItem:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGateway
            Path: /items/{id}
            Method: GET

  CreateItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handlers/create_item.handler
      CodeUri: src/
      Description: Create a new item
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref ItemsTable
      Events:
        CreateItem:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGateway
            Path: /items
            Method: POST

  ProcessItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handlers/process_item.handler
      CodeUri: src/
      Description: Process items from SQS queue
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref ItemsTable
      Events:
        SQSEvent:
          Type: SQS
          Properties:
            Queue: !GetAtt ItemQueue.Arn
            BatchSize: 10

  # ─── DynamoDB Table ───
  ItemsTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: id
        Type: String
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5

  # ─── SQS Queue ───
  ItemQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 180

Outputs:
  ApiUrl:
    Description: API Gateway endpoint URL
    Value: !Sub "https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/${Stage}"
```

---

## SAM Resource Types

```
┌──── SAM vs CloudFormation Resource Types ────────────────┐
│                                                          │
│  SAM Type                        Creates                │
│  ─────────────────────────       ──────────────────     │
│  AWS::Serverless::Function       Lambda + IAM Role      │
│                                  + Event Source Mapping  │
│                                                          │
│  AWS::Serverless::Api            API Gateway REST API    │
│                                  + Stage + Deployment    │
│                                                          │
│  AWS::Serverless::HttpApi        API Gateway HTTP API    │
│                                  (simpler, cheaper)      │
│                                                          │
│  AWS::Serverless::SimpleTable    DynamoDB Table          │
│                                                          │
│  AWS::Serverless::LayerVersion   Lambda Layer            │
│                                                          │
│  AWS::Serverless::Application    Nested SAM app          │
│                                  (Serverless App Repo)   │
│                                                          │
│  AWS::Serverless::StateMachine   Step Functions          │
│                                                          │
│  ✓ Each SAM type expands to multiple CF resources       │
│  ✓ You can mix SAM and standard CF resources            │
└──────────────────────────────────────────────────────────┘
```

---

## SAM CLI Commands

```bash
# Initialize a new SAM project
sam init
# → Choose template: Quick Start Templates
# → Choose runtime: python3.12
# → Choose template: Hello World

# Build the application
sam build

# Local invoke a single function
sam local invoke GetItemFunction \
  --event events/get-item.json

# Start local API Gateway
sam local start-api
# → http://127.0.0.1:3000/items/123

# Generate sample event
sam local generate-event apigateway aws-proxy \
  --method GET --path /items/123 > events/get-item.json

# Deploy (guided — interactive prompts)
sam deploy --guided
# → Stack name, region, confirm changes, capabilities

# Deploy (subsequent — uses samconfig.toml)
sam deploy

# View logs
sam logs --name GetItemFunction --stack-name my-api --tail

# Delete stack
sam delete --stack-name my-api

# Sync (hot-reload during development)
sam sync --watch --stack-name my-api-dev
```

---

## SAM Policy Templates

```yaml
# Instead of writing full IAM policies, use built-in templates:
Policies:
  # DynamoDB
  - DynamoDBReadPolicy:
      TableName: !Ref MyTable
  - DynamoDBCrudPolicy:
      TableName: !Ref MyTable

  # S3
  - S3ReadPolicy:
      BucketName: !Ref MyBucket
  - S3CrudPolicy:
      BucketName: !Ref MyBucket

  # SQS
  - SQSSendMessagePolicy:
      QueueName: !GetAtt MyQueue.QueueName
  - SQSPollerPolicy:
      QueueName: !GetAtt MyQueue.QueueName

  # SNS
  - SNSPublishMessagePolicy:
      TopicArn: !Ref MyTopic

  # Step Functions
  - StepFunctionsExecutionPolicy:
      StateMachineName: !GetAtt MyStateMachine.Name

  # Secrets Manager / SSM
  - SSMParameterReadPolicy:
      ParameterName: !Ref MyParam
```

---

## Local Testing & Debugging

```
┌──── SAM Local ───────────────────────────────────────────┐
│                                                          │
│  sam local invoke                                        │
│  └─ Runs Lambda in Docker container locally              │
│     Simulates Lambda runtime environment                 │
│                                                          │
│  sam local start-api                                     │
│  └─ Starts local HTTP server simulating API Gateway      │
│     Hot-reloads on code changes                          │
│                                                          │
│  sam local start-lambda                                  │
│  └─ Starts local Lambda endpoint                         │
│     Invoke via AWS SDK pointed at localhost:3001         │
│                                                          │
│  Requirements:                                           │
│  • Docker must be installed and running                  │
│  • SAM CLI pulls Lambda runtime Docker images            │
│                                                          │
│  Debugging (VS Code):                                    │
│  sam local invoke -d 5858 GetItemFunction                │
│  → Attach VS Code debugger to port 5858                 │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `sam build` fails | Missing dependencies | Check `requirements.txt` or `package.json` in CodeUri dir |
| Local invoke timeout | Function takes too long locally | Increase timeout in template or use `--timeout` flag |
| Docker not found | Docker Desktop not running | Start Docker Desktop before using `sam local` |
| Deploy fails: capabilities | Template creates IAM resources | Use `--capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND` |
| API returns 403 | Missing CORS configuration | Add `Cors` property to `AWS::Serverless::Api` |
| Hot reload not working | Not using `sam sync --watch` | Use `sam sync --watch` for development iteration |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **SAM** | CloudFormation extension for serverless. Simplified syntax. |
| **Transform** | `AWS::Serverless-2016-10-31` enables SAM resource types |
| **Globals** | Default settings shared across all functions |
| **Policy Templates** | Pre-built IAM policies: `DynamoDBCrudPolicy`, `S3ReadPolicy` |
| **SAM CLI** | Build, local invoke, start-api, deploy, sync, logs |
| **Local Testing** | Docker-based Lambda/API simulation with debugger support |

---

## Quick Revision Questions

1. **What is SAM's relationship to CloudFormation?**
   > SAM is an extension of CloudFormation. SAM templates use the `AWS::Serverless` Transform and are converted to standard CloudFormation during deployment.

2. **What does `sam local start-api` do?**
   > Starts a local HTTP server that simulates API Gateway, routing requests to Lambda functions running in Docker containers.

3. **What are SAM policy templates?**
   > Pre-built IAM policy templates like `DynamoDBCrudPolicy` that generate minimal required permissions, avoiding the need to write full IAM policy documents.

4. **What is the Globals section for?**
   > Defines default settings (runtime, timeout, memory, env vars) shared across all `AWS::Serverless::Function` resources in the template.

5. **How do you debug a SAM function locally?**
   > Use `sam local invoke -d <port> FunctionName` to start in debug mode, then attach your IDE's debugger to that port.

---

[← Previous: 8.2 CDK](02-cdk.md) | [Next: 8.4 Terraform with AWS →](04-terraform-with-aws.md)

[← Back to README](../README.md)
