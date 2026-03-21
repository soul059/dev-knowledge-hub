# 6.3 Cloud Security — Comprehensive Study Notes

## Course Overview

This subject provides a thorough understanding of cloud security across major cloud platforms (AWS, Azure, GCP). It covers cloud-specific threats, shared responsibility models, identity and access management, network security, data protection, container/serverless security, and cloud security assessment. These notes prepare you for securing cloud environments and conducting cloud penetration testing.

---

## Table of Contents

### Unit 1: Cloud Security Fundamentals
1. [Cloud Service Models (IaaS, PaaS, SaaS)](01-Cloud-Security-Fundamentals/01-cloud-service-models.md)
2. [Shared Responsibility Model](01-Cloud-Security-Fundamentals/02-shared-responsibility-model.md)
3. [Cloud Threat Landscape](01-Cloud-Security-Fundamentals/03-cloud-threat-landscape.md)
4. [Cloud vs Traditional Security](01-Cloud-Security-Fundamentals/04-cloud-vs-traditional-security.md)
5. [Cloud Security Benefits and Challenges](01-Cloud-Security-Fundamentals/05-cloud-security-benefits-and-challenges.md)

### Unit 2: AWS Security
1. [AWS Security Architecture](02-AWS-Security/01-aws-security-architecture.md)
2. [IAM (Users, Roles, Policies)](02-AWS-Security/02-iam-users-roles-policies.md)
3. [S3 Security (Bucket Policies, ACLs)](02-AWS-Security/03-s3-security.md)
4. [VPC Security (Security Groups, NACLs)](02-AWS-Security/04-vpc-security.md)
5. [CloudTrail and CloudWatch](02-AWS-Security/05-cloudtrail-and-cloudwatch.md)
6. [AWS Security Services](02-AWS-Security/06-aws-security-services.md)

### Unit 3: Azure Security
1. [Azure Security Architecture](03-Azure-Security/01-azure-security-architecture.md)
2. [Azure AD](03-Azure-Security/02-azure-ad.md)
3. [Azure RBAC](03-Azure-Security/03-azure-rbac.md)
4. [Network Security Groups](03-Azure-Security/04-network-security-groups.md)
5. [Azure Security Center](03-Azure-Security/05-azure-security-center.md)
6. [Azure Sentinel](03-Azure-Security/06-azure-sentinel.md)

### Unit 4: GCP Security
1. [GCP Security Architecture](04-GCP-Security/01-gcp-security-architecture.md)
2. [IAM Fundamentals](04-GCP-Security/02-iam-fundamentals.md)
3. [VPC and Firewall Rules](04-GCP-Security/03-vpc-and-firewall-rules.md)
4. [Cloud Audit Logs](04-GCP-Security/04-cloud-audit-logs.md)
5. [Security Command Center](04-GCP-Security/05-security-command-center.md)

### Unit 5: Cloud IAM Security
1. [Identity Federation](05-Cloud-IAM-Security/01-identity-federation.md)
2. [Principle of Least Privilege](05-Cloud-IAM-Security/02-principle-of-least-privilege.md)
3. [Service Accounts](05-Cloud-IAM-Security/03-service-accounts.md)
4. [API Key Management](05-Cloud-IAM-Security/04-api-key-management.md)
5. [Secrets Management](05-Cloud-IAM-Security/05-secrets-management.md)
6. [Privileged Access Management](05-Cloud-IAM-Security/06-privileged-access-management.md)

### Unit 6: Cloud Network Security
1. [Virtual Network Design](06-Cloud-Network-Security/01-virtual-network-design.md)
2. [Network Segmentation](06-Cloud-Network-Security/02-network-segmentation.md)
3. [Firewall Rules](06-Cloud-Network-Security/03-firewall-rules.md)
4. [Private Connectivity](06-Cloud-Network-Security/04-private-connectivity.md)
5. [DDoS Protection](06-Cloud-Network-Security/05-ddos-protection.md)
6. [WAF Implementation](06-Cloud-Network-Security/06-waf-implementation.md)

### Unit 7: Cloud Data Security
1. [Data Classification](07-Cloud-Data-Security/01-data-classification.md)
2. [Encryption at Rest](07-Cloud-Data-Security/02-encryption-at-rest.md)
3. [Encryption in Transit](07-Cloud-Data-Security/03-encryption-in-transit.md)
4. [Key Management Services](07-Cloud-Data-Security/04-key-management-services.md)
5. [Data Loss Prevention](07-Cloud-Data-Security/05-data-loss-prevention.md)
6. [Database Security](07-Cloud-Data-Security/06-database-security.md)

### Unit 8: Container and Serverless Security
1. [Container Security in Cloud](08-Container-and-Serverless-Security/01-container-security-in-cloud.md)
2. [Kubernetes Security](08-Container-and-Serverless-Security/02-kubernetes-security.md)
3. [Serverless Security](08-Container-and-Serverless-Security/03-serverless-security.md)
4. [Function Permissions](08-Container-and-Serverless-Security/04-function-permissions.md)
5. [Runtime Protection](08-Container-and-Serverless-Security/05-runtime-protection.md)

### Unit 9: Cloud Security Assessment
1. [Cloud Security Posture Management](09-Cloud-Security-Assessment/01-cloud-security-posture-management.md)
2. [Configuration Assessment](09-Cloud-Security-Assessment/02-configuration-assessment.md)
3. [Penetration Testing in Cloud](09-Cloud-Security-Assessment/03-penetration-testing-in-cloud.md)
4. [Compliance Scanning](09-Cloud-Security-Assessment/04-compliance-scanning.md)
5. [Cloud Security Tools](09-Cloud-Security-Assessment/05-cloud-security-tools.md)

---

## Learning Objectives

By completing this subject, you will be able to:
- Understand cloud security concepts and shared responsibility models
- Secure AWS, Azure, and GCP environments
- Implement cloud IAM best practices
- Design secure cloud networks
- Protect data in cloud environments
- Secure containers and serverless deployments
- Conduct cloud security assessments and penetration tests

---

## Key Tools

| Tool | Purpose |
|------|---------|
| AWS CLI | AWS management and enumeration |
| Azure CLI | Azure management and security |
| gcloud CLI | GCP management and security |
| ScoutSuite | Multi-cloud security auditing |
| Prowler | AWS security assessment |
| Pacu | AWS exploitation framework |
| CloudSploit | Cloud security scanning |
| Checkov | Infrastructure as Code scanning |
| kube-bench | Kubernetes security benchmarking |
| Trivy | Container vulnerability scanning |

---

## Subject Statistics

| Component | Count |
|-----------|-------|
| Units | 9 |
| Topic Files | 50 |
| Cloud Platforms | AWS, Azure, GCP |
| Focus Areas | Security architecture, IAM, network, data, containers |

---

*[Back to Level 6: Advanced Topics](../)*
