# Unit 6: A06 - Vulnerable and Outdated Components — Topic 1: Component Inventory

## Overview

You cannot secure what you don't know about. **Component inventory** — knowing every library, framework, runtime, and service your application depends on — is the foundation of managing vulnerable component risk. Modern applications use hundreds of dependencies, and each dependency has its own dependencies (transitive/nested). Without a complete inventory, vulnerabilities in any of these components will go undetected and unpatched.

---

## 1. The Dependency Problem

```
┌─────────────────────────────────────────────────────────────────┐
│               THE DEPENDENCY ICEBERG                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Your application: 1 project                                   │
│  Direct dependencies: ~20-50 packages                          │
│  Transitive dependencies: ~200-1000+ packages                  │
│                                                                 │
│  Example — A simple Express.js app:                            │
│  package.json: 15 dependencies                                 │
│  node_modules: 847 packages installed                          │
│                                                                 │
│  Example — A Spring Boot app:                                  │
│  pom.xml: 10 dependencies                                      │
│  Resolved: 150+ JARs                                           │
│                                                                 │
│  EACH of these packages could have:                            │
│  ├── Known CVEs (publicly disclosed vulnerabilities)           │
│  ├── Zero-day vulnerabilities (unknown)                        │
│  ├── Malicious code (supply chain attacks)                     │
│  ├── Abandoned maintenance (no security updates)               │
│  └── License issues (legal risk)                               │
│                                                                 │
│  If ANY dependency in the tree is vulnerable,                  │
│  YOUR APPLICATION is vulnerable.                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Building a Component Inventory

### Package Manager Inventory

```bash
# ═══════════════════════════════════════
# JavaScript/Node.js
# ═══════════════════════════════════════
# List direct dependencies
cat package.json | jq '.dependencies, .devDependencies'

# List ALL installed packages (including transitive)
npm list --all

# List outdated packages
npm outdated

# Generate package lock (exact versions)
npm install --package-lock-only

# ═══════════════════════════════════════
# Python
# ═══════════════════════════════════════
# List installed packages
pip list
pip freeze > requirements.txt

# Show dependency tree
pip install pipdeptree
pipdeptree

# ═══════════════════════════════════════
# Java (Maven)
# ═══════════════════════════════════════
# Full dependency tree
mvn dependency:tree

# List all dependencies
mvn dependency:list

# ═══════════════════════════════════════
# Java (Gradle)
# ═══════════════════════════════════════
gradle dependencies

# ═══════════════════════════════════════
# .NET
# ═══════════════════════════════════════
dotnet list package
dotnet list package --include-transitive

# ═══════════════════════════════════════
# Ruby
# ═══════════════════════════════════════
bundle list
gem list

# ═══════════════════════════════════════
# Go
# ═══════════════════════════════════════
go list -m all
```

---

## 3. What to Track in Your Inventory

| Field | Description | Example |
|-------|-------------|---------|
| **Component name** | Package/library name | `lodash` |
| **Version** | Exact version installed | `4.17.21` |
| **Latest version** | Current latest version | `4.17.21` |
| **License** | Software license | `MIT` |
| **Direct/Transitive** | How it's included | `Transitive (via express)` |
| **Known CVEs** | Public vulnerabilities | `CVE-2021-23337` |
| **CVSS score** | Severity | `7.2 High` |
| **Last updated** | When maintainer last published | `2021-02-20` |
| **Maintainers** | Number of active maintainers | `3` |
| **Used in** | Which apps/services use it | `api-server, web-frontend` |

---

## 4. Automated Inventory Tools

| Tool | Type | Languages | Features |
|------|------|-----------|----------|
| **Snyk** | SCA | 10+ languages | Vulnerability scanning, fix PRs |
| **Dependabot** | GitHub native | 15+ ecosystems | Auto-update PRs |
| **OWASP Dependency-Check** | Open source | Java, .NET, Ruby, Node, Python | CVE matching |
| **npm audit** | Built-in | Node.js | Vulnerability reports |
| **pip-audit** | Open source | Python | PyPI advisory DB |
| **Trivy** | Open source | Multi-language + containers | Comprehensive scanner |
| **Renovate** | Open source | 50+ managers | Auto-update PRs |
| **Mend (WhiteSource)** | Commercial | 200+ languages | Full SCA platform |
| **Black Duck** | Commercial | 80+ languages | Enterprise SCA |

### Running Inventory Scans

```bash
# npm audit
npm audit
npm audit --json > audit-report.json

# pip-audit
pip install pip-audit
pip-audit

# OWASP Dependency-Check
dependency-check --project "MyApp" --scan ./
# Generates HTML/JSON/XML report

# Trivy (filesystem scan)
trivy fs --security-checks vuln .

# Snyk
snyk test
snyk monitor  # Continuous monitoring
```

---

## 5. Container and Infrastructure Inventory

```bash
# Container image inventory
docker images --format "{{.Repository}}:{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}"

# Scan container images for vulnerabilities
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Grype (Anchore)
grype myapp:latest

# System package inventory (Linux)
dpkg --list              # Debian/Ubuntu
rpm -qa                  # CentOS/RHEL

# Kubernetes pod images
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' | sort -u
```

---

## 6. Component Inventory Policy

```
INVENTORY REQUIREMENTS:

1. COMPLETENESS
   → All direct AND transitive dependencies tracked
   → All runtime, dev, and test dependencies cataloged
   → Container base images and system packages included

2. ACCURACY
   → Exact versions (not ranges like ^1.2.3)
   → Lock files committed to version control
   → Automated scanning, not manual tracking

3. CURRENCY
   → Updated on every dependency change
   → Scanned in CI/CD pipeline
   → Continuous monitoring for new CVEs

4. ACTIONABILITY
   → Vulnerabilities linked to fix versions
   → SLA for patching (Critical: 24hr, High: 7 days)
   → Automated PRs for updates (Dependabot, Renovate)
```

---

## Summary Table

| Ecosystem | Manifest File | Lock File | Inventory Command |
|-----------|--------------|-----------|-------------------|
| Node.js | package.json | package-lock.json | `npm list --all` |
| Python | requirements.txt / pyproject.toml | Pipfile.lock | `pip list` / `pipdeptree` |
| Java Maven | pom.xml | - | `mvn dependency:tree` |
| Java Gradle | build.gradle | gradle.lockfile | `gradle dependencies` |
| .NET | .csproj | packages.lock.json | `dotnet list package` |
| Ruby | Gemfile | Gemfile.lock | `bundle list` |
| Go | go.mod | go.sum | `go list -m all` |

---

## Revision Questions

1. Why is component inventory the foundation of vulnerable component management?
2. What is the difference between direct and transitive dependencies? Why are transitive dependencies dangerous?
3. List commands to enumerate all dependencies in Node.js, Python, Java, and .NET projects.
4. Name five SCA (Software Composition Analysis) tools and their key features.
5. What information should a complete component inventory contain for each dependency?
6. How would you implement automated component inventory in a CI/CD pipeline?

---

*Previous: None (First topic in this unit) | Next: [02-vulnerability-scanning.md](02-vulnerability-scanning.md)*

---

*[Back to README](../README.md)*
