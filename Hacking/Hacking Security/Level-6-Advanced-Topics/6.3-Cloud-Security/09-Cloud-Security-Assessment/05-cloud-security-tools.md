# Unit 9: Cloud Security Assessment — Topic 5: Cloud Security Tools

## Overview

**Cloud security tools** encompass the broad ecosystem of native, open-source, and commercial solutions used to secure, assess, and monitor cloud environments. This topic provides a comprehensive reference of essential cloud security tools organized by category, with practical usage examples for each. Mastering these tools is essential for cloud security professionals.

---

## 1. Multi-Cloud Security Assessment

```
COMPREHENSIVE ASSESSMENT TOOLS:

SCOUTSUITE:
  Category: Multi-cloud security auditing
  Type: Open source (Python)
  Clouds: AWS, Azure, GCP, Alibaba, Oracle
  
  # Install
  pip install scoutsuite
  
  # Run AWS assessment
  scout aws --report-dir ./aws-report
  
  # Run Azure assessment
  scout azure --cli --report-dir ./azure-report
  
  # Run GCP assessment
  scout gcp --user-account --report-dir ./gcp-report
  
  Output: Interactive HTML report
  Categories: Danger, Warning, Good
  Coverage: IAM, Storage, Compute, Network, Logging

PROWLER:
  Category: Security assessment and compliance
  Type: Open source (Python)
  Clouds: AWS, Azure, GCP
  
  # Install
  pip install prowler
  
  # Full AWS scan
  prowler aws
  
  # Specific compliance
  prowler aws --compliance cis_2.0_aws
  prowler aws --compliance pci_3.2.1_aws
  
  # Specific checks
  prowler aws -c ec2_instance_public_ip
  
  # Output formats
  prowler aws -M json csv html
  
  Coverage: 300+ checks, CIS/PCI/HIPAA/NIST

STEAMPIPE:
  Category: Cloud resource querying
  Type: Open source
  Clouds: AWS, Azure, GCP + many more
  
  # Install
  brew install turbot/tap/steampipe
  steampipe plugin install aws azure gcp
  
  # Interactive SQL queries
  steampipe query
  > select name, region, public_access from aws_s3_bucket
  > select title, state from azure_compute_virtual_machine
  
  # Run compliance benchmark
  steampipe check aws_compliance.benchmark.cis_v200

CLOUDSPLOIT:
  Category: Multi-cloud security scanning
  Type: Open source core
  Clouds: AWS, Azure, GCP, Oracle
  
  # Clone and run
  git clone https://github.com/aquasecurity/cloudsploit
  node index.js --cloud aws
```

---

## 2. Cloud Provider-Specific Tools

```
AWS SECURITY TOOLS:

  PACU:
  → AWS exploitation framework
  → Privilege escalation scanning
  → Data exfiltration modules
  → Persistence mechanisms
  
  # Install
  pip install pacu
  python3 pacu.py
  > import_keys --all
  > run iam__enum_permissions
  > run iam__privesc_scan

  CLOUDSPLAINING:
  → AWS IAM security assessment
  → Identifies privilege escalation risks
  → Generates HTML report
  
  # Scan IAM
  cloudsplaining download --profile default
  cloudsplaining scan --input-file account_auth.json

  ENUMERATE-IAM:
  → Brute-force IAM permissions
  → Discover what an access key can do
  
  python enumerate-iam.py --access-key AKIA... --secret-key ...

  AWS CLI SECURITY:
  # Credential report
  aws iam generate-credential-report
  
  # Security Hub findings
  aws securityhub get-findings --filters '{...}'
  
  # GuardDuty findings
  aws guardduty list-findings --detector-id ID

AZURE SECURITY TOOLS:

  ROADTOOLS:
  → Azure AD enumeration
  → roadrecon: Gather Azure AD data
  → roadlib: Library for Azure AD
  
  # Gather data
  roadrecon gather --access-token $TOKEN
  roadrecon gui  # Interactive exploration

  AZUREHOUND:
  → BloodHound for Azure
  → Attack path visualization
  → Privilege escalation discovery
  → Azure AD and Azure Resource paths
  
  # Collect data
  azurehound -u user@domain.com -p password list

  MICROBURST:
  → Azure security assessment toolkit
  → Service enumeration
  → Privilege escalation
  → Data access testing
  
  Import-Module MicroBurst.psm1
  Invoke-EnumerateAzureBlobs -Base "company"

  POWERZURE:
  → Azure offensive tooling
  → Enumeration and exploitation
  
  Import-Module PowerZure.ps1
  Get-AzureTargets

GCP SECURITY TOOLS:

  GCPBUCKETBRUTE:
  → GCP bucket enumeration
  → Permission testing
  
  python3 gcpbucketbrute.py -k company -w wordlist.txt

  GCPHOUND:
  → GCP enumeration
  → Permission analysis
```

---

## 3. Infrastructure as Code Security

```
IaC SECURITY SCANNING:

TFSEC:
  Category: Terraform security scanner
  Type: Open source
  
  # Install
  brew install tfsec
  
  # Scan
  tfsec .
  tfsec . --format json
  tfsec . --minimum-severity HIGH

CHECKOV:
  Category: IaC security scanner
  Type: Open source (Bridgecrew)
  Supports: Terraform, CloudFormation, K8s, Helm, ARM
  
  # Install
  pip install checkov
  
  # Scan Terraform
  checkov -d .
  checkov --framework terraform -d .
  
  # Specific check
  checkov -d . --check CKV_AWS_18  # S3 public access

TERRASCAN:
  Category: IaC security scanner
  Type: Open source (Tenable)
  
  # Scan
  terrascan scan -t terraform
  terrascan scan -t k8s -f deployment.yaml

KICS:
  Category: IaC security scanner
  Type: Open source (Checkmarx)
  Supports: Terraform, CloudFormation, Ansible, Docker, K8s
  
  # Scan
  kics scan -p ./terraform-directory

OPA (OPEN POLICY AGENT):
  Category: Policy engine
  Type: Open source (CNCF)
  Use: Policy-as-code for any system
  
  # Evaluate policy
  opa eval -d policy.rego -i input.json "data.policy.deny"

SENTINEL (HASHICORP):
  Category: Policy-as-code for Terraform Cloud
  Type: Commercial
  
  # Example policy
  import "tfplan/v2" as tfplan
  main = rule {
    all tfplan.planned_values.resources as _, r {
      r.values.encrypted is true
    }
  }
```

---

## 4. Container and Kubernetes Security

```
CONTAINER SECURITY TOOLS:

TRIVY:
  Category: Vulnerability scanner
  Type: Open source (Aqua)
  Scans: Container images, filesystem, IaC, K8s
  
  # Scan image
  trivy image nginx:latest
  trivy image --severity HIGH,CRITICAL myapp:1.0
  
  # Scan filesystem
  trivy fs .
  
  # Scan K8s cluster
  trivy k8s --report summary cluster

GRYPE:
  Category: Vulnerability scanner
  Type: Open source (Anchore)
  
  # Scan image
  grype nginx:latest
  grype myapp:1.0 --only-fixed

KUBE-BENCH:
  Category: CIS Kubernetes benchmark
  Type: Open source (Aqua)
  
  # Run as job
  kubectl apply -f kube-bench-job.yaml
  kubectl logs kube-bench-xxxxx

KUBE-HUNTER:
  Category: K8s penetration testing
  Type: Open source (Aqua)
  
  # Run remotely
  kube-hunter --remote <cluster-ip>
  
  # Run inside cluster
  kubectl apply -f kube-hunter-job.yaml

KUBEAUDIT:
  Category: K8s security auditing
  Type: Open source
  
  # Audit all
  kubeaudit all
  kubeaudit privileged
  kubeaudit rootfs

FALCO:
  Category: Runtime security
  Type: Open source (CNCF)
  
  # Install via Helm
  helm install falco falcosecurity/falco
```

---

## 5. Security Tool Selection

```
TOOL SELECTION GUIDE:

BY ASSESSMENT TYPE:
  Configuration audit → ScoutSuite, Prowler
  Compliance check → Prowler, Steampipe
  Penetration test → Pacu, PowerZure, ROADtools
  IaC scanning → Checkov, tfsec, Terrascan
  Container scanning → Trivy, Grype
  K8s assessment → kube-bench, kube-hunter
  IAM analysis → CloudSplaining, enumerate-iam
  Runtime monitoring → Falco, cloud-native tools
  Secret scanning → truffleHog, gitleaks

BY CLOUD:
  AWS: Prowler, Pacu, ScoutSuite, CloudSplaining
  Azure: ROADtools, AzureHound, MicroBurst, PowerZure
  GCP: ScoutSuite, gcpdiag, GCPBucketBrute
  Multi: ScoutSuite, Steampipe, Checkov

RECOMMENDED TOOLKIT:
  ┌────────────────────────────────┐
  │  CLOUD SECURITY TOOLKIT       │
  │                                │
  │  Assessment: ScoutSuite       │
  │  Compliance: Prowler          │
  │  Querying: Steampipe          │
  │  IaC: Checkov + tfsec        │
  │  Containers: Trivy            │
  │  K8s: kube-bench              │
  │  Secrets: gitleaks            │
  │  AWS Pentest: Pacu            │
  │  Azure Pentest: ROADtools     │
  │  Runtime: Falco               │
  └────────────────────────────────┘

INTEGRATION:
  → CI/CD pipeline: Checkov, Trivy, gitleaks
  → Scheduled scanning: Prowler, ScoutSuite
  → Real-time monitoring: Falco, cloud-native
  → Reporting: Steampipe, custom dashboards
  → Ticketing: Integration with Jira, ServiceNow
```

---

## Summary Table

| Category | Tool | Type | Best For |
|----------|------|------|----------|
| Multi-cloud audit | ScoutSuite | Open source | Comprehensive assessment |
| Compliance | Prowler | Open source | CIS/PCI/HIPAA checks |
| Cloud query | Steampipe | Open source | SQL-based investigation |
| AWS pentest | Pacu | Open source | AWS exploitation |
| Azure pentest | ROADtools | Open source | Azure AD assessment |
| IaC scanning | Checkov | Open source | Pre-deployment checks |
| Container scan | Trivy | Open source | Image vulnerabilities |
| K8s benchmark | kube-bench | Open source | CIS K8s checks |
| Runtime | Falco | Open source | Container monitoring |
| Secrets | gitleaks | Open source | Credential scanning |

---

## Revision Questions

1. What open-source tools provide multi-cloud security assessment?
2. How do Pacu and ROADtools differ in their approach to cloud pentesting?
3. What IaC security tools should be integrated into CI/CD pipelines?
4. How does Trivy differ from kube-bench in container security?
5. What is the recommended cloud security toolkit for a security team?

---

*Previous: [04-compliance-scanning.md](04-compliance-scanning.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
