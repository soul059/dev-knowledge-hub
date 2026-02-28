# DevSecOps - Comprehensive Study Notes

> **"Security is not a phase — it's a culture woven into every line of code, every pipeline stage, and every deployment."**

---

## Course Overview

DevSecOps integrates security practices within the DevOps process. Instead of treating security as a separate phase at the end of development, DevSecOps embeds security at every stage — from planning and coding to building, testing, deploying, and monitoring. This course covers the complete DevSecOps landscape, including secure coding, automated security testing, container and Kubernetes security, infrastructure hardening, secrets management, and security operations.

### What You'll Learn

- How to **shift security left** and embed it into CI/CD pipelines
- Secure coding practices and vulnerability prevention
- Automated security testing (SAST, DAST, SCA)
- Container and Kubernetes security hardening
- Infrastructure as Code security scanning
- Secrets management and rotation strategies
- Security monitoring, incident response, and compliance

### Prerequisites

- Basic understanding of DevOps concepts (CI/CD, containers, IaC)
- Familiarity with Linux command line
- Basic knowledge of at least one programming language
- Understanding of cloud computing fundamentals

---

## Architecture Overview

```
+------------------------------------------------------------------+
|                     DevSecOps Pipeline                            |
+------------------------------------------------------------------+
|                                                                    |
|  PLAN        CODE        BUILD       TEST       DEPLOY    MONITOR |
|  +------+   +------+   +------+   +--------+  +------+  +------+ |
|  |Threat|   |Secure|   | SAST |   | DAST   |  |  IAC |  | SIEM | |
|  |Model |-->|Code  |-->|      |-->|        |->|Scan  |->|      | |
|  |      |   |Review|   | SCA  |   |Pen Test|  |      |  |  WAF | |
|  +------+   +------+   +------+   +--------+  +------+  +------+ |
|      |          |          |           |           |         |     |
|      v          v          v           v           v         v     |
|  +--------------------------------------------------------------+ |
|  |              Security Feedback Loop (Continuous)              | |
|  +--------------------------------------------------------------+ |
|                                                                    |
|  +--------------------------------------------------------------+ |
|  |  Secrets Management  |  Compliance as Code  |  Governance    | |
|  +--------------------------------------------------------------+ |
+------------------------------------------------------------------+
```

---

## Complete Table of Contents

### Unit 1: DevSecOps Introduction
| # | Chapter | File |
|---|---------|------|
| 1.1 | What is DevSecOps? | [01-what-is-devsecops.md](01-DevSecOps-Introduction/01-what-is-devsecops.md) |
| 1.2 | Shift-Left Security | [02-shift-left-security.md](01-DevSecOps-Introduction/02-shift-left-security.md) |
| 1.3 | Security in the CI/CD Pipeline | [03-security-in-cicd.md](01-DevSecOps-Introduction/03-security-in-cicd.md) |
| 1.4 | Security Culture | [04-security-culture.md](01-DevSecOps-Introduction/04-security-culture.md) |
| 1.5 | Shared Responsibility Model | [05-shared-responsibility-model.md](01-DevSecOps-Introduction/05-shared-responsibility-model.md) |

### Unit 2: Secure Coding Practices
| # | Chapter | File |
|---|---------|------|
| 2.1 | OWASP Top 10 | [01-owasp-top-10.md](02-Secure-Coding-Practices/01-owasp-top-10.md) |
| 2.2 | Common Vulnerabilities (SQL Injection, XSS, CSRF) | [02-common-vulnerabilities.md](02-Secure-Coding-Practices/02-common-vulnerabilities.md) |
| 2.3 | Secure Coding Guidelines | [03-secure-coding-guidelines.md](02-Secure-Coding-Practices/03-secure-coding-guidelines.md) |
| 2.4 | Code Review for Security | [04-code-review-for-security.md](02-Secure-Coding-Practices/04-code-review-for-security.md) |
| 2.5 | Developer Security Training | [05-developer-security-training.md](02-Secure-Coding-Practices/05-developer-security-training.md) |

### Unit 3: Static Application Security Testing (SAST)
| # | Chapter | File |
|---|---------|------|
| 3.1 | SAST Concepts | [01-sast-concepts.md](03-SAST/01-sast-concepts.md) |
| 3.2 | SAST Tools (SonarQube, Checkmarx, Semgrep) | [02-sast-tools.md](03-SAST/02-sast-tools.md) |
| 3.3 | Integration in CI/CD | [03-sast-cicd-integration.md](03-SAST/03-sast-cicd-integration.md) |
| 3.4 | False Positive Management | [04-false-positive-management.md](03-SAST/04-false-positive-management.md) |
| 3.5 | Fixing Vulnerabilities | [05-fixing-vulnerabilities.md](03-SAST/05-fixing-vulnerabilities.md) |

### Unit 4: Dynamic Application Security Testing (DAST)
| # | Chapter | File |
|---|---------|------|
| 4.1 | DAST Concepts | [01-dast-concepts.md](04-DAST/01-dast-concepts.md) |
| 4.2 | DAST Tools (OWASP ZAP, Burp Suite) | [02-dast-tools.md](04-DAST/02-dast-tools.md) |
| 4.3 | Automated Scanning | [03-automated-scanning.md](04-DAST/03-automated-scanning.md) |
| 4.4 | API Security Testing | [04-api-security-testing.md](04-DAST/04-api-security-testing.md) |
| 4.5 | Integration in Pipelines | [05-dast-pipeline-integration.md](04-DAST/05-dast-pipeline-integration.md) |

### Unit 5: Software Composition Analysis (SCA)
| # | Chapter | File |
|---|---------|------|
| 5.1 | Third-Party Dependencies Risks | [01-third-party-risks.md](05-SCA/01-third-party-risks.md) |
| 5.2 | SCA Tools (Snyk, Dependabot, OWASP Dependency-Check) | [02-sca-tools.md](05-SCA/02-sca-tools.md) |
| 5.3 | Vulnerability Databases (CVE, NVD) | [03-vulnerability-databases.md](05-SCA/03-vulnerability-databases.md) |
| 5.4 | License Compliance | [04-license-compliance.md](05-SCA/04-license-compliance.md) |
| 5.5 | Dependency Updates | [05-dependency-updates.md](05-SCA/05-dependency-updates.md) |

### Unit 6: Container Security
| # | Chapter | File |
|---|---------|------|
| 6.1 | Image Security Scanning | [01-image-security-scanning.md](06-Container-Security/01-image-security-scanning.md) |
| 6.2 | Base Image Selection | [02-base-image-selection.md](06-Container-Security/02-base-image-selection.md) |
| 6.3 | Dockerfile Best Practices | [03-dockerfile-best-practices.md](06-Container-Security/03-dockerfile-best-practices.md) |
| 6.4 | Runtime Security | [04-runtime-security.md](06-Container-Security/04-runtime-security.md) |
| 6.5 | Container Registries Security | [05-container-registries-security.md](06-Container-Security/05-container-registries-security.md) |
| 6.6 | Tools (Trivy, Clair, Anchore) | [06-container-security-tools.md](06-Container-Security/06-container-security-tools.md) |

### Unit 7: Kubernetes Security
| # | Chapter | File |
|---|---------|------|
| 7.1 | Cluster Hardening | [01-cluster-hardening.md](07-Kubernetes-Security/01-cluster-hardening.md) |
| 7.2 | RBAC Implementation | [02-rbac-implementation.md](07-Kubernetes-Security/02-rbac-implementation.md) |
| 7.3 | Network Policies | [03-network-policies.md](07-Kubernetes-Security/03-network-policies.md) |
| 7.4 | Pod Security Standards | [04-pod-security-standards.md](07-Kubernetes-Security/04-pod-security-standards.md) |
| 7.5 | Secrets Management | [05-k8s-secrets-management.md](07-Kubernetes-Security/05-k8s-secrets-management.md) |
| 7.6 | Security Scanning Tools | [06-security-scanning-tools.md](07-Kubernetes-Security/06-security-scanning-tools.md) |
| 7.7 | Runtime Protection (Falco) | [07-runtime-protection-falco.md](07-Kubernetes-Security/07-runtime-protection-falco.md) |

### Unit 8: Infrastructure Security
| # | Chapter | File |
|---|---------|------|
| 8.1 | IaC Security Scanning | [01-iac-security-scanning.md](08-Infrastructure-Security/01-iac-security-scanning.md) |
| 8.2 | Cloud Security Posture | [02-cloud-security-posture.md](08-Infrastructure-Security/02-cloud-security-posture.md) |
| 8.3 | Compliance as Code | [03-compliance-as-code.md](08-Infrastructure-Security/03-compliance-as-code.md) |
| 8.4 | Security Benchmarks (CIS) | [04-security-benchmarks-cis.md](08-Infrastructure-Security/04-security-benchmarks-cis.md) |
| 8.5 | Tools (Checkov, tfsec, Prowler) | [05-infrastructure-security-tools.md](08-Infrastructure-Security/05-infrastructure-security-tools.md) |

### Unit 9: Secrets Management
| # | Chapter | File |
|---|---------|------|
| 9.1 | Secrets Challenges | [01-secrets-challenges.md](09-Secrets-Management/01-secrets-challenges.md) |
| 9.2 | HashiCorp Vault | [02-hashicorp-vault.md](09-Secrets-Management/02-hashicorp-vault.md) |
| 9.3 | Cloud Secrets Managers | [03-cloud-secrets-managers.md](09-Secrets-Management/03-cloud-secrets-managers.md) |
| 9.4 | Secret Rotation | [04-secret-rotation.md](09-Secrets-Management/04-secret-rotation.md) |
| 9.5 | Integration Patterns | [05-integration-patterns.md](09-Secrets-Management/05-integration-patterns.md) |
| 9.6 | Best Practices | [06-secrets-best-practices.md](09-Secrets-Management/06-secrets-best-practices.md) |

### Unit 10: Security Operations
| # | Chapter | File |
|---|---------|------|
| 10.1 | Security Monitoring | [01-security-monitoring.md](10-Security-Operations/01-security-monitoring.md) |
| 10.2 | Threat Detection | [02-threat-detection.md](10-Security-Operations/02-threat-detection.md) |
| 10.3 | Incident Response | [03-incident-response.md](10-Security-Operations/03-incident-response.md) |
| 10.4 | Penetration Testing | [04-penetration-testing.md](10-Security-Operations/04-penetration-testing.md) |
| 10.5 | Bug Bounty Programs | [05-bug-bounty-programs.md](10-Security-Operations/05-bug-bounty-programs.md) |
| 10.6 | Security Compliance (SOC2, ISO 27001) | [06-security-compliance.md](10-Security-Operations/06-security-compliance.md) |

---

## How to Use These Notes

1. **Sequential Study**: Follow the units in order for a structured learning path
2. **Quick Reference**: Jump to specific topics using the Table of Contents
3. **Revision**: Use the summary tables and quick revision questions at the end of each chapter
4. **Hands-On**: Follow the configuration examples and commands in each chapter
5. **Real-World**: Apply the scenarios and use cases to your own projects

## Key Tools Covered

| Category | Tools |
|----------|-------|
| SAST | SonarQube, Checkmarx, Semgrep |
| DAST | OWASP ZAP, Burp Suite |
| SCA | Snyk, Dependabot, OWASP Dependency-Check |
| Container Security | Trivy, Clair, Anchore |
| Kubernetes Security | Falco, kube-bench, OPA/Gatekeeper |
| IaC Security | Checkov, tfsec, Prowler |
| Secrets | HashiCorp Vault, AWS Secrets Manager, Azure Key Vault |
| Monitoring | ELK Stack, Prometheus, Grafana |

---

## Quick Start

Begin with **[Unit 1: DevSecOps Introduction](01-DevSecOps-Introduction/01-what-is-devsecops.md)** to understand the foundational concepts, then progress through each unit sequentially.

---

*Last updated: February 2026*
