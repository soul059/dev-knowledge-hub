# Unit 6: Cloud Network Security — Topic 6: WAF Implementation

## Overview

A **Web Application Firewall (WAF)** protects web applications by filtering and monitoring HTTP/HTTPS traffic between the internet and the application. Cloud WAFs inspect Layer 7 traffic to block common web exploits like SQL injection, cross-site scripting (XSS), and other OWASP Top 10 attacks. Proper WAF implementation is essential for protecting public-facing cloud applications.

---

## 1. WAF Fundamentals

```
WAF OPERATION:

  Client → WAF → Load Balancer → Application
  
  WAF inspects every HTTP request:
  → Headers, body, URL, query parameters
  → Cookies, user-agent, content-type
  → Request method, protocol version
  → JSON/XML payloads

DETECTION MODES:
  Positive Security Model (Allowlist):
  → Define what IS allowed
  → Block everything else
  → More secure, harder to implement
  → Best for APIs with known schemas
  
  Negative Security Model (Blocklist):
  → Define what is NOT allowed
  → Allow everything else
  → Easier to implement
  → Risk of bypasses
  → Most common approach

WAF ACTIONS:
  → Allow: Pass request to application
  → Block: Deny request (403/429)
  → Count: Log but don't block (testing)
  → Challenge: CAPTCHA or JS challenge
  → Rate limit: Throttle excessive requests
  → Custom response: Return specific page
```

---

## 2. Cloud WAF Services

```
AWS WAF:

  COMPONENTS:
  → Web ACL: Container for rules
  → Rules: Match conditions + actions
  → Rule Groups: Collection of rules
  → Managed Rule Groups: Pre-built by AWS/vendors

  MANAGED RULE GROUPS:
  → AWS Managed Rules - Core rule set
  → AWS Managed Rules - SQL injection
  → AWS Managed Rules - Known bad inputs
  → AWS Managed Rules - Bot control
  → AWS Managed Rules - Account takeover prevention
  → Third-party: F5, Fortinet, Imperva

  DEPLOYMENT:
  → Amazon CloudFront
  → Application Load Balancer (ALB)
  → Amazon API Gateway
  → AWS AppSync
  → Amazon Cognito

  # Create Web ACL
  aws wafv2 create-web-acl \
    --name my-web-acl \
    --scope REGIONAL \
    --default-action Allow={} \
    --rules file://rules.json \
    --visibility-config \
      SampledRequestsEnabled=true,\
      CloudWatchMetricsEnabled=true,\
      MetricName=my-web-acl

AZURE WAF:

  DEPLOYMENT OPTIONS:
  → Azure Application Gateway (regional)
  → Azure Front Door (global)
  → Azure CDN

  RULE SETS:
  → OWASP CRS 3.2 (Core Rule Set)
  → Bot protection
  → Custom rules
  → Geo-filtering
  → Rate limiting

  MODES:
  → Detection mode (log only)
  → Prevention mode (block)
  → Start in detection, tune, then enable prevention

  # Enable WAF on App Gateway
  az network application-gateway waf-config set \
    --gateway-name myAppGateway \
    --resource-group myRG \
    --enabled true \
    --firewall-mode Prevention \
    --rule-set-type OWASP \
    --rule-set-version 3.2

GCP CLOUD ARMOR:

  FEATURES:
  → Pre-configured WAF rules
  → Custom rules (CEL expressions)
  → Adaptive Protection (ML-based)
  → Bot management
  → Rate limiting
  → Named IP lists
  → Threat intelligence

  PRE-CONFIGURED RULES:
  → sqli: SQL injection
  → xss: Cross-site scripting
  → lfi: Local file inclusion
  → rfi: Remote file inclusion
  → rce: Remote code execution
  → methodenforcement: HTTP method control
  → scannerdetection: Scanner blocking
  → protocolattack: Protocol violations

  # Create security policy with OWASP rules
  gcloud compute security-policies create my-waf-policy

  gcloud compute security-policies rules create 1000 \
    --security-policy=my-waf-policy \
    --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
    --action=deny-403

  gcloud compute security-policies rules create 1100 \
    --security-policy=my-waf-policy \
    --expression="evaluatePreconfiguredExpr('xss-v33-stable')" \
    --action=deny-403
```

---

## 3. WAF Rule Configuration

```
CUSTOM RULES:

AWS WAF CUSTOM RULE EXAMPLES:
  
  Block SQL Injection:
  {
    "Name": "BlockSQLi",
    "Priority": 1,
    "Action": {"Block": {}},
    "Statement": {
      "SqliMatchStatement": {
        "FieldToMatch": {"AllQueryArguments": {}},
        "TextTransformations": [
          {"Priority": 0, "Type": "URL_DECODE"},
          {"Priority": 1, "Type": "HTML_ENTITY_DECODE"}
        ]
      }
    }
  }
  
  Rate Limiting:
  {
    "Name": "RateLimit",
    "Priority": 2,
    "Action": {"Block": {}},
    "Statement": {
      "RateBasedStatement": {
        "Limit": 2000,
        "AggregateKeyType": "IP"
      }
    }
  }
  
  Geo Blocking:
  {
    "Name": "GeoBlock",
    "Priority": 3,
    "Action": {"Block": {}},
    "Statement": {
      "GeoMatchStatement": {
        "CountryCodes": ["CN", "RU", "KP"]
      }
    }
  }

RULE PRIORITY:
  → Lower number = evaluated first
  → First matching rule wins
  → Default action applies if no match
  → Order matters for performance and accuracy

TEXT TRANSFORMATIONS:
  → URL_DECODE: Decode URL encoding
  → HTML_ENTITY_DECODE: Decode HTML entities
  → LOWERCASE: Convert to lowercase
  → COMPRESS_WHITE_SPACE: Normalize whitespace
  → Base64 decode
  → Important: Apply before matching
  → Prevents encoding bypass attacks
```

---

## 4. WAF Tuning and Operations

```
WAF TUNING PROCESS:

PHASE 1: DEPLOY IN LOG/COUNT MODE
  → Enable WAF with logging only
  → No traffic is blocked
  → Collect data on what would be blocked
  → Duration: 1-2 weeks minimum

PHASE 2: ANALYZE FALSE POSITIVES
  → Review logged "would-be-blocked" requests
  → Identify legitimate traffic flagged
  → Common false positives:
    - API endpoints with code in body
    - Search queries with SQL-like terms
    - Rich text editors (HTML in input)
    - File upload endpoints
    - Webhook callbacks

PHASE 3: CREATE EXCEPTIONS
  → Exclude specific paths/parameters
  → Disable specific rules for specific URIs
  → Example: Disable SQLi on /api/search endpoint
  → Document every exception with justification

PHASE 4: ENABLE BLOCKING MODE
  → Switch from count/detect to block/prevent
  → Monitor closely for new false positives
  → Have rollback plan ready
  → Gradual rollout if possible

PHASE 5: CONTINUOUS TUNING
  → Review WAF logs regularly
  → Update rules for new threats
  → Adjust rate limits based on traffic
  → Remove outdated exceptions
  → Test with security scanning tools

WAF LOGGING:
  AWS: CloudWatch Logs, S3, Kinesis Firehose
  Azure: Log Analytics, Storage Account
  GCP: Cloud Logging
  
  Key fields to log:
  → Timestamp
  → Source IP
  → Request URI
  → Rule matched
  → Action taken
  → Request details (headers, body)
```

---

## 5. WAF Assessment

```
WAF SECURITY ASSESSMENT:

TESTING WAF EFFECTIVENESS:
  → Test with known attack payloads
  → Use OWASP ZAP or Burp Suite
  → Try WAF bypass techniques
  → Verify all rule groups active
  → Check for missing coverage

COMMON WAF BYPASSES:
  → Encoding attacks (double URL encode, Unicode)
  → Case manipulation (SeLeCt instead of SELECT)
  → Comment injection (SEL/**/ECT)
  → HTTP parameter pollution
  → Chunked transfer encoding
  → Content-Type manipulation
  → Large payload padding
  → Multipart form data

ASSESSMENT CHECKLIST:
  [ ] WAF deployed on all public web applications
  [ ] OWASP Core Rule Set enabled
  [ ] SQL injection protection active
  [ ] XSS protection active
  [ ] Rate limiting configured
  [ ] Bot detection enabled
  [ ] Custom rules for application-specific threats
  [ ] WAF in blocking mode (not just logging)
  [ ] Logging enabled and monitored
  [ ] Regular rule updates applied
  [ ] False positive exceptions documented
  [ ] WAF bypass testing conducted

COMMON GAPS:
  Gap                             | Risk
  WAF in detection mode only      | High (no actual blocking)
  No rate limiting                | Medium
  Managed rules not updated       | Medium
  No logging configured           | High
  Too many exceptions             | Medium
  API endpoints not covered       | High
  No bot protection               | Medium
  WebSocket traffic not inspected | Low
```

---

## Summary Table

| Feature | AWS WAF | Azure WAF | GCP Cloud Armor |
|---------|---------|-----------|----------------|
| Deployment | CloudFront, ALB, API GW | App GW, Front Door | External LB |
| Rule Sets | Managed + custom | OWASP CRS + custom | Pre-configured + custom |
| Bot Control | Yes (add-on) | Yes | Yes |
| ML Detection | No | No | Adaptive Protection |
| Rate Limiting | Yes | Yes | Yes |
| Pricing | Per rule + requests | Per gateway | Per policy + requests |

---

## Revision Questions

1. What is the difference between positive and negative security models?
2. How do AWS WAF, Azure WAF, and GCP Cloud Armor compare?
3. Why is text transformation important for WAF rules?
4. What is the recommended process for WAF tuning?
5. How can WAF bypasses be tested during security assessments?

---

*Previous: [05-ddos-protection.md](05-ddos-protection.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
