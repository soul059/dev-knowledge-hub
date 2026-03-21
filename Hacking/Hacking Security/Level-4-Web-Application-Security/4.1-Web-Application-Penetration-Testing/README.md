# 4.1 Web Application Penetration Testing

## 📋 Course Overview

This subject provides comprehensive coverage of web application security testing, from understanding web fundamentals to advanced exploitation techniques. You'll learn to identify, exploit, and remediate web vulnerabilities using industry-standard tools and methodologies. Content aligns with OWASP Testing Guide and real-world penetration testing engagements.

## 🎯 Learning Objectives

After completing this subject, you will be able to:
- Understand web application architecture and communication protocols
- Perform systematic web application security assessments
- Identify and exploit OWASP Top 10 vulnerabilities
- Test authentication, authorization, and session management
- Exploit SQL injection, XSS, CSRF, and other injection attacks
- Assess API security (REST, GraphQL)
- Use professional web testing tools (Burp Suite, ZAP)
- Provide actionable remediation guidance

## 📚 Table of Contents

### Unit 1: Web Application Fundamentals
| # | Topic | File |
|---|-------|------|
| 1 | HTTP/HTTPS Protocol | [01-http-https-protocol.md](01-Web-Application-Fundamentals/01-http-https-protocol.md) |
| 2 | Request and Response Structure | [02-request-response-structure.md](01-Web-Application-Fundamentals/02-request-response-structure.md) |
| 3 | HTTP Methods | [03-http-methods.md](01-Web-Application-Fundamentals/03-http-methods.md) |
| 4 | Status Codes | [04-status-codes.md](01-Web-Application-Fundamentals/04-status-codes.md) |
| 5 | Headers (Security-Relevant) | [05-security-headers.md](01-Web-Application-Fundamentals/05-security-headers.md) |
| 6 | Cookies and Sessions | [06-cookies-and-sessions.md](01-Web-Application-Fundamentals/06-cookies-and-sessions.md) |
| 7 | Same-Origin Policy | [07-same-origin-policy.md](01-Web-Application-Fundamentals/07-same-origin-policy.md) |

### Unit 2: Web Application Architecture
| # | Topic | File |
|---|-------|------|
| 1 | Client-Server Model | [01-client-server-model.md](02-Web-Application-Architecture/01-client-server-model.md) |
| 2 | Frontend Technologies | [02-frontend-technologies.md](02-Web-Application-Architecture/02-frontend-technologies.md) |
| 3 | Backend Technologies | [03-backend-technologies.md](02-Web-Application-Architecture/03-backend-technologies.md) |
| 4 | Database Interactions | [04-database-interactions.md](02-Web-Application-Architecture/04-database-interactions.md) |
| 5 | API Architectures (REST, GraphQL) | [05-api-architectures.md](02-Web-Application-Architecture/05-api-architectures.md) |
| 6 | Microservices | [06-microservices.md](02-Web-Application-Architecture/06-microservices.md) |
| 7 | CDN and Caching | [07-cdn-and-caching.md](02-Web-Application-Architecture/07-cdn-and-caching.md) |

### Unit 3: Web Reconnaissance
| # | Topic | File |
|---|-------|------|
| 1 | Application Mapping | [01-application-mapping.md](03-Web-Reconnaissance/01-application-mapping.md) |
| 2 | Technology Fingerprinting | [02-technology-fingerprinting.md](03-Web-Reconnaissance/02-technology-fingerprinting.md) |
| 3 | Directory Enumeration | [03-directory-enumeration.md](03-Web-Reconnaissance/03-directory-enumeration.md) |
| 4 | Parameter Discovery | [04-parameter-discovery.md](03-Web-Reconnaissance/04-parameter-discovery.md) |
| 5 | Functionality Mapping | [05-functionality-mapping.md](03-Web-Reconnaissance/05-functionality-mapping.md) |
| 6 | Comment and Metadata Analysis | [06-comment-metadata-analysis.md](03-Web-Reconnaissance/06-comment-metadata-analysis.md) |

### Unit 4: Authentication Testing
| # | Topic | File |
|---|-------|------|
| 1 | Authentication Mechanisms | [01-authentication-mechanisms.md](04-Authentication-Testing/01-authentication-mechanisms.md) |
| 2 | Brute Force Attacks | [02-brute-force-attacks.md](04-Authentication-Testing/02-brute-force-attacks.md) |
| 3 | Password Policies | [03-password-policies.md](04-Authentication-Testing/03-password-policies.md) |
| 4 | Multi-Factor Authentication Testing | [04-mfa-testing.md](04-Authentication-Testing/04-mfa-testing.md) |
| 5 | Session Management | [05-session-management.md](04-Authentication-Testing/05-session-management.md) |
| 6 | OAuth and SSO Testing | [06-oauth-sso-testing.md](04-Authentication-Testing/06-oauth-sso-testing.md) |

### Unit 5: SQL Injection
| # | Topic | File |
|---|-------|------|
| 1 | SQL Injection Types | [01-sql-injection-types.md](05-SQL-Injection/01-sql-injection-types.md) |
| 2 | Detection Techniques | [02-detection-techniques.md](05-SQL-Injection/02-detection-techniques.md) |
| 3 | Exploitation Methodology | [03-exploitation-methodology.md](05-SQL-Injection/03-exploitation-methodology.md) |
| 4 | Data Extraction | [04-data-extraction.md](05-SQL-Injection/04-data-extraction.md) |
| 5 | Authentication Bypass | [05-authentication-bypass.md](05-SQL-Injection/05-authentication-bypass.md) |
| 6 | Automated Tools (sqlmap) | [06-automated-tools-sqlmap.md](05-SQL-Injection/06-automated-tools-sqlmap.md) |
| 7 | Prevention Techniques | [07-prevention-techniques.md](05-SQL-Injection/07-prevention-techniques.md) |

### Unit 6: Cross-Site Scripting (XSS)
| # | Topic | File |
|---|-------|------|
| 1 | XSS Types (Reflected, Stored, DOM-based) | [01-xss-types.md](06-Cross-Site-Scripting-XSS/01-xss-types.md) |
| 2 | XSS Detection | [02-xss-detection.md](06-Cross-Site-Scripting-XSS/02-xss-detection.md) |
| 3 | Payload Crafting | [03-payload-crafting.md](06-Cross-Site-Scripting-XSS/03-payload-crafting.md) |
| 4 | Filter Bypass Techniques | [04-filter-bypass-techniques.md](06-Cross-Site-Scripting-XSS/04-filter-bypass-techniques.md) |
| 5 | Impact and Exploitation | [05-impact-and-exploitation.md](06-Cross-Site-Scripting-XSS/05-impact-and-exploitation.md) |
| 6 | XSS Prevention | [06-xss-prevention.md](06-Cross-Site-Scripting-XSS/06-xss-prevention.md) |
| 7 | Content Security Policy | [07-content-security-policy.md](06-Cross-Site-Scripting-XSS/07-content-security-policy.md) |

### Unit 7: Cross-Site Request Forgery (CSRF)
| # | Topic | File |
|---|-------|------|
| 1 | CSRF Mechanism | [01-csrf-mechanism.md](07-Cross-Site-Request-Forgery-CSRF/01-csrf-mechanism.md) |
| 2 | CSRF vs XSS | [02-csrf-vs-xss.md](07-Cross-Site-Request-Forgery-CSRF/02-csrf-vs-xss.md) |
| 3 | Token-Based Protection | [03-token-based-protection.md](07-Cross-Site-Request-Forgery-CSRF/03-token-based-protection.md) |
| 4 | SameSite Cookies | [04-samesite-cookies.md](07-Cross-Site-Request-Forgery-CSRF/04-samesite-cookies.md) |
| 5 | Testing Methodology | [05-testing-methodology.md](07-Cross-Site-Request-Forgery-CSRF/05-testing-methodology.md) |
| 6 | Prevention Techniques | [06-prevention-techniques.md](07-Cross-Site-Request-Forgery-CSRF/06-prevention-techniques.md) |

### Unit 8: Injection Attacks
| # | Topic | File |
|---|-------|------|
| 1 | Command Injection | [01-command-injection.md](08-Injection-Attacks/01-command-injection.md) |
| 2 | LDAP Injection | [02-ldap-injection.md](08-Injection-Attacks/02-ldap-injection.md) |
| 3 | XML Injection | [03-xml-injection.md](08-Injection-Attacks/03-xml-injection.md) |
| 4 | XXE (XML External Entity) | [04-xxe-attacks.md](08-Injection-Attacks/04-xxe-attacks.md) |
| 5 | Template Injection (SSTI) | [05-template-injection-ssti.md](08-Injection-Attacks/05-template-injection-ssti.md) |
| 6 | Header Injection | [06-header-injection.md](08-Injection-Attacks/06-header-injection.md) |

### Unit 9: Access Control Testing
| # | Topic | File |
|---|-------|------|
| 1 | IDOR (Insecure Direct Object Reference) | [01-idor.md](09-Access-Control-Testing/01-idor.md) |
| 2 | Horizontal Privilege Escalation | [02-horizontal-privilege-escalation.md](09-Access-Control-Testing/02-horizontal-privilege-escalation.md) |
| 3 | Vertical Privilege Escalation | [03-vertical-privilege-escalation.md](09-Access-Control-Testing/03-vertical-privilege-escalation.md) |
| 4 | Function-Level Access Control | [04-function-level-access-control.md](09-Access-Control-Testing/04-function-level-access-control.md) |
| 5 | JWT Security Testing | [05-jwt-security-testing.md](09-Access-Control-Testing/05-jwt-security-testing.md) |
| 6 | Authorization Bypass | [06-authorization-bypass.md](09-Access-Control-Testing/06-authorization-bypass.md) |

### Unit 10: Other Web Vulnerabilities
| # | Topic | File |
|---|-------|------|
| 1 | File Upload Vulnerabilities | [01-file-upload-vulnerabilities.md](10-Other-Web-Vulnerabilities/01-file-upload-vulnerabilities.md) |
| 2 | Path Traversal | [02-path-traversal.md](10-Other-Web-Vulnerabilities/02-path-traversal.md) |
| 3 | SSRF (Server-Side Request Forgery) | [03-ssrf.md](10-Other-Web-Vulnerabilities/03-ssrf.md) |
| 4 | Deserialization Vulnerabilities | [04-deserialization-vulnerabilities.md](10-Other-Web-Vulnerabilities/04-deserialization-vulnerabilities.md) |
| 5 | Business Logic Flaws | [05-business-logic-flaws.md](10-Other-Web-Vulnerabilities/05-business-logic-flaws.md) |
| 6 | Race Conditions | [06-race-conditions.md](10-Other-Web-Vulnerabilities/06-race-conditions.md) |

### Unit 11: API Security Testing
| # | Topic | File |
|---|-------|------|
| 1 | API Reconnaissance | [01-api-reconnaissance.md](11-API-Security-Testing/01-api-reconnaissance.md) |
| 2 | API Authentication Testing | [02-api-authentication-testing.md](11-API-Security-Testing/02-api-authentication-testing.md) |
| 3 | Rate Limiting | [03-rate-limiting.md](11-API-Security-Testing/03-rate-limiting.md) |
| 4 | API-Specific Vulnerabilities | [04-api-specific-vulnerabilities.md](11-API-Security-Testing/04-api-specific-vulnerabilities.md) |
| 5 | GraphQL Security | [05-graphql-security.md](11-API-Security-Testing/05-graphql-security.md) |
| 6 | REST API Testing | [06-rest-api-testing.md](11-API-Security-Testing/06-rest-api-testing.md) |

### Unit 12: Web Testing Tools
| # | Topic | File |
|---|-------|------|
| 1 | Burp Suite Fundamentals | [01-burp-suite-fundamentals.md](12-Web-Testing-Tools/01-burp-suite-fundamentals.md) |
| 2 | OWASP ZAP | [02-owasp-zap.md](12-Web-Testing-Tools/02-owasp-zap.md) |
| 3 | Browser Developer Tools | [03-browser-developer-tools.md](12-Web-Testing-Tools/03-browser-developer-tools.md) |
| 4 | Automated Scanners | [04-automated-scanners.md](12-Web-Testing-Tools/04-automated-scanners.md) |
| 5 | Custom Scripts | [05-custom-scripts.md](12-Web-Testing-Tools/05-custom-scripts.md) |
| 6 | Tool Chaining | [06-tool-chaining.md](12-Web-Testing-Tools/06-tool-chaining.md) |

## 🔧 Key Tools

| Tool | Purpose | Category |
|------|---------|----------|
| Burp Suite | Web proxy, scanner, repeater | Primary Testing |
| OWASP ZAP | Open-source web proxy/scanner | Primary Testing |
| sqlmap | Automated SQL injection | Injection Testing |
| XSStrike | Advanced XSS detection | XSS Testing |
| Nikto | Web server scanner | Reconnaissance |
| Gobuster/ffuf | Directory & file brute-forcing | Reconnaissance |
| Wfuzz | Web fuzzer | Fuzzing |
| Postman/Insomnia | API testing | API Testing |
| JWT_Tool | JWT manipulation | Token Testing |
| Nuclei | Template-based scanner | Automated Scanning |
| Browser DevTools | Request inspection & debugging | Manual Testing |
| cURL/httpie | HTTP request crafting | Manual Testing |

## 📊 Subject Statistics

| Metric | Count |
|--------|-------|
| Total Units | 12 |
| Total Topics | 76 |
| Estimated Study Time | 60-80 hours |
| Hands-On Labs | 50+ exercises |
| Key Tools | 12+ |

---

*Part of the [Hacking & Security Course](../../HACKING-SECURITY-PROMPTS.md) — Level 4: Web Application Security*
