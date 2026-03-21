# Unit 8: Container and Serverless Security — Topic 3: Serverless Security

## Overview

**Serverless computing** (Functions-as-a-Service) allows running code without managing servers. While serverless eliminates infrastructure management, it introduces unique security challenges around function permissions, event triggers, dependency management, and observability. Understanding serverless security is essential as organizations increasingly adopt Lambda, Azure Functions, and Cloud Functions.

---

## 1. Serverless Security Landscape

```
SERVERLESS SERVICES:

  AWS Lambda
  Azure Functions
  GCP Cloud Functions / Cloud Run
  Cloudflare Workers
  Vercel Functions

SERVERLESS CHARACTERISTICS:
  → No server management
  → Event-driven execution
  → Automatic scaling
  → Pay-per-execution
  → Short-lived (minutes max)
  → Stateless (by design)

SHARED RESPONSIBILITY (SERVERLESS):
  Cloud Provider:
  → Physical security
  → Host OS patching
  → Runtime environment
  → Container isolation
  → Network infrastructure
  → Scaling and availability
  
  Customer:
  → Application code security
  → Function permissions (IAM)
  → Dependency management
  → Data handling
  → Authentication/authorization
  → Secrets management
  → Input validation

SECURITY ADVANTAGES:
  → No OS to patch
  → No server to harden
  → Reduced attack surface
  → Automatic scaling (DDoS resilient)
  → Short-lived execution context
  → Provider handles isolation

SECURITY CHALLENGES:
  → Overpermissioned function roles
  → Vulnerable dependencies
  → Injection attacks
  → Event data tampering
  → Cold start information leakage
  → Difficulty in monitoring
  → Increased attack surface via triggers
  → Third-party library supply chain
```

---

## 2. Serverless Attack Vectors

```
ATTACK VECTORS:

1. INJECTION ATTACKS:
   → Function receives untrusted input
   → SQL injection via event data
   → Command injection in function code
   → SSRF through function network access
   → NoSQL injection (DynamoDB, Cosmos DB)
   
   Example (vulnerable Lambda):
   def handler(event, context):
       user_input = event['name']
       # VULNERABLE - command injection
       os.system(f"echo Hello {user_input}")
       
   Attack: {"name": "; cat /etc/passwd"}

2. OVERPERMISSIONED ROLES:
   → Lambda with AdministratorAccess
   → Function can access ANY AWS resource
   → Single compromise = full account access
   → Common: Copy-paste IAM policies
   
   BAD:
   { "Effect": "Allow", "Action": "*", "Resource": "*" }
   
   GOOD:
   { "Effect": "Allow", 
     "Action": ["dynamodb:PutItem", "dynamodb:GetItem"],
     "Resource": "arn:aws:dynamodb:*:*:table/myTable" }

3. DEPENDENCY VULNERABILITIES:
   → Outdated npm/pip packages
   → Known CVEs in dependencies
   → Supply chain attacks
   → Typosquatting packages

4. EVENT DATA POISONING:
   → Malicious data in S3 events
   → Tampered API Gateway requests
   → Poisoned SQS/SNS messages
   → Crafted CloudWatch events
   → All event sources are potential attack vectors

5. INFORMATION DISCLOSURE:
   → Verbose error messages
   → Environment variables exposed
   → Logs containing sensitive data
   → Cold start behavior differences
   → Timing attacks
```

---

## 3. Serverless Security Best Practices

```
SECURITY CONTROLS:

1. IAM LEAST PRIVILEGE:
   → One role per function
   → Minimum required permissions
   → Resource-level restrictions
   → No wildcard (*) permissions
   → Use policy conditions
   
   # AWS SAM template
   MyFunction:
     Type: AWS::Serverless::Function
     Properties:
       Policies:
         - DynamoDBReadPolicy:
             TableName: !Ref MyTable
         - S3ReadPolicy:
             BucketName: !Ref MyBucket

2. INPUT VALIDATION:
   → Validate ALL event data
   → Use schema validation (JSON Schema)
   → Sanitize before processing
   → Never trust event source data
   → Type checking and bounds checking

3. DEPENDENCY MANAGEMENT:
   → Pin dependency versions
   → Regular vulnerability scanning
   → Use Snyk, npm audit, pip-audit
   → Minimal dependencies
   → Lock files (package-lock.json, Pipfile.lock)

4. SECRETS MANAGEMENT:
   → Use Secrets Manager / Key Vault
   → Never in environment variables (visible in console)
   → Cache secrets in memory (not on disk)
   → Use IAM for service-to-service auth
   → Encrypt sensitive environment variables

5. MONITORING AND LOGGING:
   → Enable CloudWatch / Azure Monitor / Cloud Logging
   → Log function invocations
   → Alert on errors and anomalies
   → Distributed tracing (X-Ray, App Insights)
   → Don't log sensitive data
   
6. NETWORK SECURITY:
   → Deploy in VPC when accessing internal resources
   → No public internet access unless needed
   → Use VPC endpoints for AWS services
   → NAT Gateway for outbound access
```

---

## 4. Cloud Provider Serverless Security

```
AWS LAMBDA SECURITY:

  → Function URL authentication (IAM or None)
  → Resource-based policies
  → VPC deployment option
  → Lambda Layers for shared dependencies
  → Reserved concurrency limits
  → Dead letter queues for failures
  → AWS X-Ray for tracing
  → CodeGuru for code analysis
  → Lambda SnapStart (Java) for cold start
  
  # Restrict Lambda URL access
  aws lambda add-permission \
    --function-name my-func \
    --action lambda:InvokeFunctionUrl \
    --principal "*" \
    --function-url-auth-type AWS_IAM

AZURE FUNCTIONS SECURITY:

  → Authentication/Authorization (Easy Auth)
  → Managed identity for Azure resource access
  → Key Vault integration for secrets
  → VNet integration
  → Private endpoints
  → Deployment slots for safe deployment
  → Application Insights monitoring
  → IP restrictions

GCP CLOUD FUNCTIONS SECURITY:

  → IAM-based invocation control
  → VPC connector for internal access
  → Ingress settings (all, internal-only, internal+LB)
  → Service account per function
  → Secret Manager integration
  → Cloud Audit Logs

  # Restrict to authenticated only
  gcloud functions deploy my-func \
    --no-allow-unauthenticated \
    --service-account=my-func-sa@proj.iam.gserviceaccount.com \
    --ingress-settings=internal-only
```

---

## 5. Assessment

```
SERVERLESS SECURITY ASSESSMENT:

CHECKLIST:
  [ ] Each function has dedicated IAM role
  [ ] Roles follow least privilege
  [ ] No wildcard permissions
  [ ] Input validation on all event data
  [ ] Dependencies scanned for vulnerabilities
  [ ] Dependencies pinned to specific versions
  [ ] Secrets in Secrets Manager (not env vars)
  [ ] Functions deployed in VPC (if needed)
  [ ] Authentication on function URLs/endpoints
  [ ] Logging and monitoring enabled
  [ ] Concurrency limits set
  [ ] Error handling doesn't leak information

COMMON FINDINGS:
  Finding                          | Risk
  Admin IAM role on function       | Critical
  No input validation              | High
  Secrets in environment variables | High
  Outdated dependencies            | Medium
  Function URL without auth        | High
  No VPC deployment                | Medium
  Verbose error responses          | Medium
  No monitoring/alerting           | Medium
  No concurrency limits            | Low

TOOLS:
  → SLS-DevTools: Serverless monitoring
  → Serverless Security Top 10 (OWASP)
  → Prowler (AWS serverless checks)
  → Snyk (dependency scanning)
  → Dashbird (Lambda monitoring)
  → Lumigo (serverless debugging)
```

---

## Summary Table

| Risk | Mitigation | Priority |
|------|-----------|----------|
| Overpermissioned role | Least privilege IAM | Critical |
| Injection attacks | Input validation | High |
| Vulnerable dependencies | Scanning, pinning | High |
| Secrets exposure | Secrets Manager | High |
| No authentication | IAM/API key auth | High |
| Lack of monitoring | CloudWatch/tracing | Medium |

---

## Revision Questions

1. How does the shared responsibility model differ for serverless?
2. What are the most common serverless attack vectors?
3. Why is IAM least privilege especially important for serverless?
4. How should secrets be managed in serverless functions?
5. What tools can assess serverless security?

---

*Previous: [02-kubernetes-security.md](02-kubernetes-security.md) | Next: [04-function-permissions.md](04-function-permissions.md)*

---

*[Back to README](../README.md)*
