# Unit 8: Container and Serverless Security — Topic 4: Function Permissions

## Overview

**Function permissions** control what serverless functions can access and who can invoke them. In serverless architectures, each function's IAM role defines its blast radius if compromised. Overpermissioned functions are the #1 serverless security risk. This topic covers how to properly scope function permissions across AWS Lambda, Azure Functions, and GCP Cloud Functions.

---

## 1. Permission Models

```
SERVERLESS PERMISSION TYPES:

EXECUTION ROLE (what the function CAN DO):
  → IAM role attached to the function
  → Defines which cloud resources it can access
  → Should be unique per function
  → Principle of least privilege

INVOCATION POLICY (who CAN CALL the function):
  → Controls who/what can trigger the function
  → API Gateway, S3 events, schedules, etc.
  → Resource-based policies
  → IAM-based or public access

AWS LAMBDA:
  Execution Role:
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "dynamodb:GetItem",
          "dynamodb:PutItem"
        ],
        "Resource": "arn:aws:dynamodb:us-east-1:123456:table/Orders"
      },
      {
        "Effect": "Allow",
        "Action": "logs:*",
        "Resource": "arn:aws:logs:*:*:*"
      }
    ]
  }
  
  Resource-Based Policy (who can invoke):
  {
    "Effect": "Allow",
    "Principal": { "Service": "apigateway.amazonaws.com" },
    "Action": "lambda:InvokeFunction",
    "Resource": "arn:aws:lambda:...:function:my-func",
    "Condition": {
      "ArnLike": {
        "AWS:SourceArn": "arn:aws:execute-api:...:*/*/GET/orders"
      }
    }
  }

AZURE FUNCTIONS:
  → Managed identity (system or user-assigned)
  → RBAC roles on Azure resources
  → Function keys (host, function, master)
  → EasyAuth for authentication
  → IP restrictions

GCP CLOUD FUNCTIONS:
  → Service account per function
  → IAM roles on GCP resources
  → Invoker permission (who can call)
  → allUsers for public (DANGEROUS)
  → Authenticated-only by default (Gen2)
```

---

## 2. Least Privilege Implementation

```
PERMISSION SCOPING STRATEGIES:

PER-FUNCTION ROLES:
  Function: processOrder
  Role: processOrder-role
  Permissions:
  → dynamodb:PutItem on Orders table
  → sqs:SendMessage on notification queue
  → logs:PutLogEvents
  
  Function: getUser
  Role: getUser-role
  Permissions:
  → dynamodb:GetItem on Users table
  → logs:PutLogEvents

  Function: generateReport
  Role: generateReport-role
  Permissions:
  → dynamodb:Query on Orders table (read-only)
  → s3:PutObject on reports bucket
  → logs:PutLogEvents

ANTI-PATTERN (SHARED ROLE):
  ⚠ All functions share: Lambda-FullAccess
  → Any function can access any resource
  → Compromise of one = compromise of all
  → Violates least privilege

RESOURCE-LEVEL RESTRICTIONS:
  GOOD:
  Resource: "arn:aws:s3:::my-specific-bucket/uploads/*"
  
  BAD:
  Resource: "*"  (all S3 buckets!)

CONDITION-BASED RESTRICTIONS:
  {
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::data-bucket/*",
    "Condition": {
      "StringEquals": {
        "aws:PrincipalTag/Environment": "production"
      }
    }
  }

SERVERLESS FRAMEWORK:
  # serverless.yml - per function IAM
  functions:
    processOrder:
      handler: orders.process
      iamRoleStatements:
        - Effect: Allow
          Action:
            - dynamodb:PutItem
          Resource: !GetAtt OrdersTable.Arn
    
    getUser:
      handler: users.get
      iamRoleStatements:
        - Effect: Allow
          Action:
            - dynamodb:GetItem
          Resource: !GetAtt UsersTable.Arn
```

---

## 3. Permission Analysis

```
FINDING OVERPERMISSIONED FUNCTIONS:

AWS:
  # List Lambda functions and roles
  aws lambda list-functions \
    --query "Functions[*].{Name:FunctionName,Role:Role}"
  
  # Get role policies
  aws iam list-attached-role-policies --role-name lambda-role
  aws iam list-role-policies --role-name lambda-role
  
  # Get inline policy
  aws iam get-role-policy --role-name lambda-role --policy-name my-policy
  
  # Check for wildcard permissions
  # Look for Action: "*" or Resource: "*"

  # IAM Access Analyzer
  aws accessanalyzer list-findings

AZURE:
  # List function apps
  az functionapp list --query "[].{Name:name, Identity:identity}"
  
  # Check managed identity roles
  az role assignment list --assignee <managed-identity-id>

GCP:
  # List functions and service accounts
  gcloud functions list \
    --format="table(name, serviceAccountEmail)"
  
  # Check SA permissions
  gcloud projects get-iam-policy PROJECT \
    --flatten="bindings[].members" \
    --filter="bindings.members:serviceAccount:SA_EMAIL"

RED FLAGS:
  → Action: "*" (all actions)
  → Resource: "*" (all resources)
  → Action: "s3:*" (all S3 actions)
  → Admin/PowerUser managed policies
  → Same role for multiple functions
  → PassRole permission (privilege escalation)
  → IAM write permissions on function role
```

---

## 4. Event Source Security

```
SECURING EVENT TRIGGERS:

API GATEWAY:
  → Authentication (IAM, Cognito, API Keys)
  → Request validation
  → Rate limiting
  → WAF integration
  → CORS configuration
  → Resource policies

S3 EVENT TRIGGERS:
  → Validate file type and size
  → Scan for malware before processing
  → Don't trust file metadata
  → Process in isolated environment
  → Limit bucket event notifications

SQS/SNS TRIGGERS:
  → Validate message schema
  → Use dead letter queues
  → Message encryption
  → Access policies on queue/topic
  → Don't trust message content

SCHEDULED TRIGGERS:
  → Least privilege for scheduled tasks
  → Audit scheduled function permissions
  → Monitor execution frequency
  → Alert on failures

CROSS-ACCOUNT TRIGGERS:
  → Explicit resource policies
  → Verify source account
  → Condition keys for account restriction
  → Audit cross-account access
```

---

## 5. Assessment

```
FUNCTION PERMISSION ASSESSMENT:

CHECKLIST:
  [ ] Each function has unique IAM role
  [ ] No wildcard (*) permissions
  [ ] Permissions scoped to specific resources
  [ ] Resource-based policies restrict invocation
  [ ] No Admin/PowerUser policies on functions
  [ ] Managed identities used (Azure)
  [ ] Service accounts are function-specific (GCP)
  [ ] Cross-account access justified
  [ ] Event sources authenticated
  [ ] Unused functions disabled/deleted
  [ ] Permission usage analyzed (Access Advisor)

COMMON FINDINGS:
  Finding                          | Risk
  Wildcard permissions (*:*)       | Critical
  Shared role across functions     | High
  Admin managed policy attached    | Critical
  Public function URL (no auth)    | High
  PassRole permission              | High
  Cross-account without conditions | Medium
  Unused permissions               | Medium
  No resource restrictions         | High

REMEDIATION PRIORITY:
  1. Remove wildcard permissions
  2. Create per-function roles
  3. Scope to specific resources
  4. Add authentication to endpoints
  5. Review cross-account access
  6. Remove unused permissions
  7. Add conditions where possible
```

---

## Summary Table

| Provider | Execution Identity | Invocation Control | Best Practice |
|----------|-------------------|-------------------|---------------|
| AWS | IAM Role | Resource Policy | Per-function role |
| Azure | Managed Identity | Function Keys + RBAC | System-assigned MI |
| GCP | Service Account | IAM Invoker | Per-function SA |

---

## Revision Questions

1. What is the difference between execution roles and invocation policies?
2. Why should each serverless function have its own IAM role?
3. What are the red flags when analyzing function permissions?
4. How should event source triggers be secured?
5. What tools can identify overpermissioned serverless functions?

---

*Previous: [03-serverless-security.md](03-serverless-security.md) | Next: [05-runtime-protection.md](05-runtime-protection.md)*

---

*[Back to README](../README.md)*
