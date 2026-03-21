# 4.2 OWASP Top 10 Deep Dive

## Course Overview

This subject provides an **in-depth analysis of every OWASP Top 10 (2021) vulnerability category**, the industry-standard awareness document for web application security. Each unit dissects one category with attack flow diagrams, vulnerable vs. secure code examples, real-world breach case studies, detection techniques, and comprehensive remediation strategies.

> **OWASP Top 10 (2021 Edition):** A01–A10 represent the most critical security risks to web applications, updated through broad industry survey and data analysis.

---

## Table of Contents

### Unit 1: A01 — Broken Access Control
| # | Topic | File |
|---|-------|------|
| 1 | Access Control Concepts | [01-access-control-concepts.md](01-Broken-Access-Control/01-access-control-concepts.md) |
| 2 | Common Weaknesses | [02-common-weaknesses.md](01-Broken-Access-Control/02-common-weaknesses.md) |
| 3 | IDOR Vulnerabilities | [03-idor-vulnerabilities.md](01-Broken-Access-Control/03-idor-vulnerabilities.md) |
| 4 | Privilege Escalation | [04-privilege-escalation.md](01-Broken-Access-Control/04-privilege-escalation.md) |
| 5 | JWT Vulnerabilities | [05-jwt-vulnerabilities.md](01-Broken-Access-Control/05-jwt-vulnerabilities.md) |
| 6 | Testing Methodology | [06-testing-methodology.md](01-Broken-Access-Control/06-testing-methodology.md) |
| 7 | Prevention and Remediation | [07-prevention-and-remediation.md](01-Broken-Access-Control/07-prevention-and-remediation.md) |

### Unit 2: A02 — Cryptographic Failures
| # | Topic | File |
|---|-------|------|
| 1 | Data Classification | [01-data-classification.md](02-Cryptographic-Failures/01-data-classification.md) |
| 2 | Data in Transit Protection | [02-data-in-transit-protection.md](02-Cryptographic-Failures/02-data-in-transit-protection.md) |
| 3 | Data at Rest Protection | [03-data-at-rest-protection.md](02-Cryptographic-Failures/03-data-at-rest-protection.md) |
| 4 | Weak Cryptographic Algorithms | [04-weak-cryptographic-algorithms.md](02-Cryptographic-Failures/04-weak-cryptographic-algorithms.md) |
| 5 | Key Management | [05-key-management.md](02-Cryptographic-Failures/05-key-management.md) |
| 6 | Certificate Validation | [06-certificate-validation.md](02-Cryptographic-Failures/06-certificate-validation.md) |
| 7 | Prevention Strategies | [07-prevention-strategies.md](02-Cryptographic-Failures/07-prevention-strategies.md) |

### Unit 3: A03 — Injection
| # | Topic | File |
|---|-------|------|
| 1 | Injection Types | [01-injection-types.md](03-Injection/01-injection-types.md) |
| 2 | SQL Injection Deep Dive | [02-sql-injection-deep-dive.md](03-Injection/02-sql-injection-deep-dive.md) |
| 3 | NoSQL Injection | [03-nosql-injection.md](03-Injection/03-nosql-injection.md) |
| 4 | OS Command Injection | [04-os-command-injection.md](03-Injection/04-os-command-injection.md) |
| 5 | LDAP Injection | [05-ldap-injection.md](03-Injection/05-ldap-injection.md) |
| 6 | Prevention (Parameterized Queries, Input Validation) | [06-prevention.md](03-Injection/06-prevention.md) |

### Unit 4: A04 — Insecure Design
| # | Topic | File |
|---|-------|------|
| 1 | Secure Design Principles | [01-secure-design-principles.md](04-Insecure-Design/01-secure-design-principles.md) |
| 2 | Threat Modeling | [02-threat-modeling.md](04-Insecure-Design/02-threat-modeling.md) |
| 3 | Secure Design Patterns | [03-secure-design-patterns.md](04-Insecure-Design/03-secure-design-patterns.md) |
| 4 | Attack Surface Reduction | [04-attack-surface-reduction.md](04-Insecure-Design/04-attack-surface-reduction.md) |
| 5 | Defense in Depth Implementation | [05-defense-in-depth.md](04-Insecure-Design/05-defense-in-depth.md) |
| 6 | Security Requirements | [06-security-requirements.md](04-Insecure-Design/06-security-requirements.md) |

### Unit 5: A05 — Security Misconfiguration
| # | Topic | File |
|---|-------|------|
| 1 | Default Configurations | [01-default-configurations.md](05-Security-Misconfiguration/01-default-configurations.md) |
| 2 | Unnecessary Features | [02-unnecessary-features.md](05-Security-Misconfiguration/02-unnecessary-features.md) |
| 3 | Error Handling | [03-error-handling.md](05-Security-Misconfiguration/03-error-handling.md) |
| 4 | Security Headers | [04-security-headers.md](05-Security-Misconfiguration/04-security-headers.md) |
| 5 | Cloud Misconfigurations | [05-cloud-misconfigurations.md](05-Security-Misconfiguration/05-cloud-misconfigurations.md) |
| 6 | Hardening Procedures | [06-hardening-procedures.md](05-Security-Misconfiguration/06-hardening-procedures.md) |

### Unit 6: A06 — Vulnerable and Outdated Components
| # | Topic | File |
|---|-------|------|
| 1 | Component Inventory | [01-component-inventory.md](06-Vulnerable-Components/01-component-inventory.md) |
| 2 | Vulnerability Scanning | [02-vulnerability-scanning.md](06-Vulnerable-Components/02-vulnerability-scanning.md) |
| 3 | Dependency Management | [03-dependency-management.md](06-Vulnerable-Components/03-dependency-management.md) |
| 4 | SCA (Software Composition Analysis) | [04-software-composition-analysis.md](06-Vulnerable-Components/04-software-composition-analysis.md) |
| 5 | Update Strategies | [05-update-strategies.md](06-Vulnerable-Components/05-update-strategies.md) |
| 6 | SBOM (Software Bill of Materials) | [06-software-bill-of-materials.md](06-Vulnerable-Components/06-software-bill-of-materials.md) |

### Unit 7: A07 — Identification and Authentication Failures
| # | Topic | File |
|---|-------|------|
| 1 | Credential Stuffing | [01-credential-stuffing.md](07-Authentication-Failures/01-credential-stuffing.md) |
| 2 | Brute Force Protection | [02-brute-force-protection.md](07-Authentication-Failures/02-brute-force-protection.md) |
| 3 | Session Management | [03-session-management.md](07-Authentication-Failures/03-session-management.md) |
| 4 | Password Requirements | [04-password-requirements.md](07-Authentication-Failures/04-password-requirements.md) |
| 5 | MFA Implementation | [05-mfa-implementation.md](07-Authentication-Failures/05-mfa-implementation.md) |
| 6 | Account Recovery | [06-account-recovery.md](07-Authentication-Failures/06-account-recovery.md) |

### Unit 8: A08 — Software and Data Integrity Failures
| # | Topic | File |
|---|-------|------|
| 1 | CI/CD Security | [01-cicd-security.md](08-Software-Data-Integrity-Failures/01-cicd-security.md) |
| 2 | Dependency Integrity | [02-dependency-integrity.md](08-Software-Data-Integrity-Failures/02-dependency-integrity.md) |
| 3 | Unsigned Updates | [03-unsigned-updates.md](08-Software-Data-Integrity-Failures/03-unsigned-updates.md) |
| 4 | Deserialization | [04-deserialization.md](08-Software-Data-Integrity-Failures/04-deserialization.md) |
| 5 | Code Signing | [05-code-signing.md](08-Software-Data-Integrity-Failures/05-code-signing.md) |
| 6 | Integrity Verification | [06-integrity-verification.md](08-Software-Data-Integrity-Failures/06-integrity-verification.md) |

### Unit 9: A09 — Security Logging and Monitoring Failures
| # | Topic | File |
|---|-------|------|
| 1 | Logging Requirements | [01-logging-requirements.md](09-Security-Logging-Monitoring-Failures/01-logging-requirements.md) |
| 2 | Audit Trails | [02-audit-trails.md](09-Security-Logging-Monitoring-Failures/02-audit-trails.md) |
| 3 | Monitoring and Alerting | [03-monitoring-and-alerting.md](09-Security-Logging-Monitoring-Failures/03-monitoring-and-alerting.md) |
| 4 | Incident Response Integration | [04-incident-response-integration.md](09-Security-Logging-Monitoring-Failures/04-incident-response-integration.md) |
| 5 | Log Injection Prevention | [05-log-injection-prevention.md](09-Security-Logging-Monitoring-Failures/05-log-injection-prevention.md) |
| 6 | SIEM Integration | [06-siem-integration.md](09-Security-Logging-Monitoring-Failures/06-siem-integration.md) |

### Unit 10: A10 — Server-Side Request Forgery (SSRF)
| # | Topic | File |
|---|-------|------|
| 1 | SSRF Attack Vectors | [01-ssrf-attack-vectors.md](10-Server-Side-Request-Forgery/01-ssrf-attack-vectors.md) |
| 2 | Cloud Metadata Attacks | [02-cloud-metadata-attacks.md](10-Server-Side-Request-Forgery/02-cloud-metadata-attacks.md) |
| 3 | Internal Service Access | [03-internal-service-access.md](10-Server-Side-Request-Forgery/03-internal-service-access.md) |
| 4 | SSRF Filter Bypass | [04-ssrf-filter-bypass.md](10-Server-Side-Request-Forgery/04-ssrf-filter-bypass.md) |
| 5 | Prevention Techniques | [05-prevention-techniques.md](10-Server-Side-Request-Forgery/05-prevention-techniques.md) |
| 6 | Network Segmentation | [06-network-segmentation.md](10-Server-Side-Request-Forgery/06-network-segmentation.md) |

---

## Learning Objectives

By completing this subject, you will be able to:

- ✅ Explain every OWASP Top 10 (2021) category with real-world breach examples
- ✅ Identify and exploit each vulnerability class in controlled environments
- ✅ Write both vulnerable and secure code for each category
- ✅ Perform access control, injection, and SSRF testing methodically
- ✅ Implement cryptographic best practices for data in transit and at rest
- ✅ Apply threat modeling and secure design principles from the start
- ✅ Harden applications against misconfigurations and component vulnerabilities
- ✅ Design logging, monitoring, and incident response pipelines
- ✅ Verify software and data integrity across CI/CD pipelines
- ✅ Prepare for OSCP, eWPT, and OWASP-based certification exams

---

## Key Tools

| Tool | Purpose |
|------|---------|
| Burp Suite | Web proxy and scanner |
| OWASP ZAP | Open-source web scanner |
| sqlmap | SQL injection automation |
| Nuclei | Template-based vulnerability scanner |
| ffuf | Web fuzzer |
| JWT_Tool | JWT analysis and exploitation |
| Semgrep | Static analysis / SAST |
| Trivy / Snyk | Dependency and container scanning |
| ELK Stack | Logging and SIEM |

---

## Subject Statistics

| Metric | Value |
|--------|-------|
| **Units** | 10 |
| **Topic Files** | 62 |
| **OWASP Categories** | A01–A10 (2021) |
| **Level** | 4 — Web Application Security |

---

*[Back to Level 4](../)*
