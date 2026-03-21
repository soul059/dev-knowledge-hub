# Unit 9: Cloud Security Assessment — Topic 3: Penetration Testing in Cloud

## Overview

**Cloud penetration testing** evaluates the security of cloud environments by simulating real-world attacks. Unlike traditional penetration testing, cloud pentesting must account for shared responsibility models, cloud provider policies, and cloud-specific attack vectors. Understanding the rules of engagement, legal requirements, and cloud-specific testing techniques is essential for conducting effective and authorized cloud security assessments.

---

## 1. Cloud Pentest Rules

```
CLOUD PROVIDER POLICIES:

AWS:
  → NO prior approval required (as of 2023)
  → Must own/have permission for target account
  → Permitted services: EC2, RDS, Aurora, CloudFront,
    API Gateway, Lambda, Lightsail, ECS, EKS, etc.
  → PROHIBITED: DNS zone walking, DoS/DDoS, port
    flooding, protocol flooding
  → Report abuse to aws-abuse@amazon.com

AZURE:
  → NO prior approval required
  → Must follow Rules of Engagement
  → Permitted: Azure-hosted assets you own
  → PROHIBITED: DoS, port scanning that impacts
    other tenants, social engineering Azure employees
  → Penetration Testing Rules of Engagement:
    https://www.microsoft.com/en-us/msrc/pentest-rules-of-engagement

GCP:
  → NO prior approval required
  → Must own the tested projects
  → Follow Acceptable Use Policy
  → PROHIBITED: DoS attacks, testing other
    customers' resources
  → Report issues to security@google.com

GENERAL RULES:
  [ ] Written authorization from asset owner
  [ ] Define scope (accounts, services, data)
  [ ] Time window for testing
  [ ] Emergency contact information
  [ ] Rules of engagement documented
  [ ] No impact on other tenants
  [ ] No DoS/DDoS testing
  [ ] Handle discovered data per agreement
  [ ] Report findings securely
```

---

## 2. Cloud Attack Methodology

```
CLOUD PENTEST METHODOLOGY:

PHASE 1: RECONNAISSANCE
  → Identify cloud provider (DNS, headers, IPs)
  → Enumerate cloud services in use
  → Find exposed cloud resources
  → Discover S3 buckets, Azure blobs
  → GitHub/GitLab for leaked credentials
  → DNS enumeration for cloud endpoints
  → Certificate transparency logs

PHASE 2: ENUMERATION
  → Service enumeration (authenticated)
  → IAM user and role enumeration
  → Resource discovery
  → Permission enumeration
  → Network mapping
  → API endpoint discovery

PHASE 3: EXPLOITATION
  → Exploit misconfigurations
  → Credential abuse
  → Privilege escalation
  → Data access and exfiltration
  → Lateral movement
  → Service exploitation

PHASE 4: POST-EXPLOITATION
  → Persistence mechanisms
  → Data collection
  → Impact assessment
  → Pivot to other services/accounts
  → Evidence documentation

PHASE 5: REPORTING
  → Executive summary
  → Technical findings
  → Risk ratings
  → Remediation recommendations
  → Evidence and reproduction steps

CLOUD-SPECIFIC ATTACK VECTORS:
  → IMDS exploitation (169.254.169.254)
  → S3 bucket enumeration and access
  → Lambda function code exposure
  → IAM privilege escalation
  → Cross-account role assumption
  → Secrets in environment variables
  → Misconfigured API gateways
  → Container escape to cloud credentials
  → Serverless event injection
  → Cloud metadata abuse
```

---

## 3. Cloud Pentesting Tools

```
RECONNAISSANCE TOOLS:

  CloudEnum:
  → Enumerate cloud resources
  → S3 buckets, Azure blobs, GCP buckets
  → Keyword-based discovery
  
  # python3 cloud_enum.py -k company-name

  S3Scanner:
  → Scan for open S3 buckets
  → List bucket contents
  → Check permissions
  
  # s3scanner --bucket company-backup

  CloudBrute:
  → Multi-cloud resource enumeration
  → AWS, Azure, GCP
  → Wordlist-based
  
EXPLOITATION TOOLS:

  Pacu (AWS):
  → AWS exploitation framework
  → Modular architecture
  → Privilege escalation
  → Data exfiltration
  → Persistence
  
  # Run Pacu
  python3 pacu.py
  > import_keys
  > run iam__enum_permissions
  > run iam__privesc_scan
  > run s3__download_bucket

  PowerZure (Azure):
  → Azure offensive tooling
  → Enumeration and exploitation
  → Privilege escalation
  → Data extraction
  
  ROADtools (Azure AD):
  → Azure AD enumeration
  → ROADrecon: Gather Azure AD data
  → ROADlib: Library for Azure AD ops
  → Interactive exploration

  GCPBucketBrute (GCP):
  → GCP bucket enumeration
  → Permission testing
  
  ScoutSuite:
  → Multi-cloud security assessment
  → Identifies misconfigurations
  → HTML report with findings
  
CREDENTIAL TOOLS:
  → truffleHog: Git history scanning
  → gitleaks: Repository scanning
  → CloudSplaining: AWS IAM analysis
  → enumerate-iam: AWS permission enumeration
```

---

## 4. Common Cloud Exploits

```
EXPLOITATION TECHNIQUES:

IMDS EXPLOITATION:
  # Access EC2 metadata (v1)
  curl http://169.254.169.254/latest/meta-data/
  curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
  
  # Returns: AccessKeyId, SecretAccessKey, Token
  # Use these to access AWS services as the instance role
  
  # IMDSv2 (requires token)
  TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
    -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
  curl -H "X-aws-ec2-metadata-token: $TOKEN" \
    http://169.254.169.254/latest/meta-data/

IAM PRIVILEGE ESCALATION (AWS):
  → iam:CreatePolicyVersion (create new policy)
  → iam:SetDefaultPolicyVersion (activate policy)
  → iam:AttachUserPolicy (attach admin policy)
  → iam:CreateLoginProfile (create console access)
  → iam:UpdateLoginProfile (reset password)
  → iam:PassRole + ec2:RunInstances (launch with role)
  → lambda:CreateFunction + iam:PassRole (exec as role)
  → sts:AssumeRole (assume cross-account role)

S3 BUCKET EXPLOITATION:
  # List bucket contents
  aws s3 ls s3://target-bucket --no-sign-request
  
  # Download public data
  aws s3 cp s3://target-bucket/sensitive.csv ./
  
  # Check bucket ACL
  aws s3api get-bucket-acl --bucket target-bucket
  
  # Check bucket policy
  aws s3api get-bucket-policy --bucket target-bucket

AZURE AD EXPLOITATION:
  → Password spraying against Azure AD
  → Token theft via device code phishing
  → Consent phishing (malicious app registration)
  → JWT token manipulation
  → Service principal key extraction

LATERAL MOVEMENT:
  → Cross-account role assumption
  → Service account impersonation
  → Shared credential abuse
  → VPC peering exploitation
  → Shared storage access
```

---

## 5. Cloud Pentest Reporting

```
REPORT STRUCTURE:

EXECUTIVE SUMMARY:
  → Overall risk rating
  → Key findings summary
  → Business impact
  → Recommendations priority

METHODOLOGY:
  → Scope and boundaries
  → Tools and techniques used
  → Testing timeline
  → Limitations

FINDINGS:
  For each finding:
  → Title and ID
  → Severity (Critical/High/Medium/Low)
  → CVSS score (if applicable)
  → Description
  → Evidence (screenshots, outputs)
  → Impact assessment
  → Remediation recommendation
  → References

EXAMPLE FINDINGS:
  FINDING: CRIT-001
  Title: S3 Bucket Publicly Accessible with Sensitive Data
  Severity: Critical
  Description: The bucket "company-backups" is configured
  with public read access. It contains database backups with
  PII including names, emails, and SSNs.
  Impact: Complete exposure of customer PII
  Remediation: Enable S3 Block Public Access, review bucket
  policy, enable encryption, audit for data exposure period

  FINDING: HIGH-002
  Title: EC2 Instance Role with AdministratorAccess
  Severity: High
  Description: The EC2 instance "web-server-01" has an
  attached IAM role with AdministratorAccess policy.
  SSRF vulnerability could lead to full account compromise
  via IMDS credential theft.
  Impact: Full AWS account compromise
  Remediation: Create least-privilege role with only
  required permissions. Enable IMDSv2.

REMEDIATION TRACKING:
  Finding | Severity | Status     | Owner  | Due Date
  CRIT-001| Critical | In Progress| DevOps | 7 days
  HIGH-002| High     | Open       | Cloud  | 14 days
  MED-003 | Medium   | Fixed      | SecOps | Completed
```

---

## Summary Table

| Phase | Activities | Key Tools |
|-------|-----------|-----------|
| Recon | Resource discovery, credential search | CloudEnum, truffleHog |
| Enumeration | IAM, network, service enumeration | Pacu, ScoutSuite |
| Exploitation | Privilege escalation, data access | Pacu, PowerZure |
| Post-Exploit | Persistence, lateral movement | Cloud CLI tools |
| Reporting | Findings, remediation | Report templates |

---

## Revision Questions

1. What are the cloud provider policies for penetration testing?
2. How does cloud penetration testing methodology differ from traditional?
3. What is IMDS exploitation and how can it be prevented?
4. What IAM permissions enable privilege escalation in AWS?
5. What should a cloud penetration test report include?

---

*Previous: [02-configuration-assessment.md](02-configuration-assessment.md) | Next: [04-compliance-scanning.md](04-compliance-scanning.md)*

---

*[Back to README](../README.md)*
