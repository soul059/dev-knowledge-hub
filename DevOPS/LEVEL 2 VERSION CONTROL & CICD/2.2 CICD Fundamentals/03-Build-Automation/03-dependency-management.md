# Chapter 3.3: Dependency Management

[← Previous: Compilation and Packaging](02-compilation-packaging.md) | [Back to README](../README.md) | [Next: Build Optimization →](04-build-optimization.md)

---

## Overview

**Dependency management** handles external libraries your project relies on. In CI/CD, proper dependency management ensures reproducible builds, avoids security vulnerabilities, and speeds up pipelines through caching.

---

## Dependency Resolution Flow

```
┌───────────────────────────────────────────────────────────────┐
│              DEPENDENCY RESOLUTION                             │
│                                                                │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐          │
│  │ Manifest   │───▶│  Resolver  │───▶│  Download  │          │
│  │ File       │    │            │    │ & Install  │          │
│  └────────────┘    └────────────┘    └────────────┘          │
│   package.json      Find compatible    Fetch from             │
│   pom.xml           versions           registry               │
│   requirements.txt                                             │
│                         │                                      │
│                         ▼                                      │
│                   ┌────────────┐                               │
│                   │  Lockfile  │  Records exact resolved       │
│                   │            │  versions for reproducibility │
│                   └────────────┘                               │
│                   package-lock.json                            │
│                   pom.xml (with versions)                      │
│                   poetry.lock                                   │
└───────────────────────────────────────────────────────────────┘
```

---

## Lockfiles — The Key to Reproducibility

```
WITHOUT LOCKFILE (Non-deterministic)
════════════════════════════════════
  package.json: "lodash": "^4.17.0"

  Monday build:   installs lodash 4.17.20  ✅
  Tuesday build:  installs lodash 4.17.21  ❌ (new bug!)
  
  Same code, different result!

WITH LOCKFILE (Deterministic)
════════════════════════════════════
  package-lock.json: "lodash": "4.17.20"

  Monday build:   installs lodash 4.17.20  ✅
  Tuesday build:  installs lodash 4.17.20  ✅
  
  Same code, same result. Always.
```

| Ecosystem | Manifest | Lockfile | CI Install Command |
|---|---|---|---|
| npm | `package.json` | `package-lock.json` | `npm ci` |
| Yarn | `package.json` | `yarn.lock` | `yarn install --frozen-lockfile` |
| pnpm | `package.json` | `pnpm-lock.yaml` | `pnpm install --frozen-lockfile` |
| Python (pip) | `requirements.txt` | `requirements.txt` (pinned) | `pip install -r requirements.txt` |
| Python (Poetry) | `pyproject.toml` | `poetry.lock` | `poetry install --no-interaction` |
| Java (Maven) | `pom.xml` | (versions in pom) | `mvn dependency:resolve` |
| Go | `go.mod` | `go.sum` | `go mod download` |
| Rust | `Cargo.toml` | `Cargo.lock` | `cargo build` |
| Ruby | `Gemfile` | `Gemfile.lock` | `bundle install` |

---

## Dependency Vulnerability Scanning

```
┌──────────────────────────────────────────────────────────────────┐
│          DEPENDENCY SECURITY SCANNING                             │
│                                                                   │
│  Your App                                                         │
│    │                                                              │
│    ├── express@4.18.2              ✅ No known vulnerabilities    │
│    ├── lodash@4.17.20             ✅ OK                          │
│    ├── axios@0.21.1               ❌ CVE-2021-3749 (HIGH)        │
│    │   └── follow-redirects@1.13  ❌ CVE-2022-0536 (MEDIUM)     │
│    └── jsonwebtoken@8.5.1         ⚠️  CVE-2022-23529 (MEDIUM)   │
│                                                                   │
│  Tools: npm audit, Snyk, Dependabot, OWASP dep-check,           │
│         Trivy, Grype, Safety (Python)                            │
└──────────────────────────────────────────────────────────────────┘
```

```bash
# Scan for vulnerabilities
npm audit                           # Node.js
pip-audit                           # Python
mvn org.owasp:dependency-check-maven:check  # Java
snyk test                           # Any language (Snyk CLI)
trivy fs .                          # File system scan
```

### CI Pipeline Integration
```yaml
# Fail pipeline on high-severity vulnerabilities
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm audit --audit-level=high   # Fails if HIGH/CRITICAL found
```

---

## Automated Dependency Updates

```
┌──────────────────────────────────────────────────────────────┐
│       AUTOMATED DEPENDENCY UPDATES                            │
│                                                               │
│  ┌────────────────┐  detects   ┌────────────────┐           │
│  │  Dependency    │──────────▶│  Creates PR    │           │
│  │  Bot           │           │  with update   │           │
│  │                │           │                │           │
│  │  • Dependabot  │           │  "Bump lodash  │           │
│  │  • Renovate    │           │   4.17.20 →    │           │
│  │  • Snyk        │           │   4.17.21"     │           │
│  └────────────────┘           └───────┬────────┘           │
│                                        │                    │
│                                        ▼                    │
│                               ┌────────────────┐           │
│                               │  CI runs tests │           │
│                               │  automatically │           │
│                               └───────┬────────┘           │
│                                       │                     │
│                                  ✅ Pass → Auto-merge      │
│                                  ❌ Fail → Notify team     │
└──────────────────────────────────────────────────────────────┘
```

### Dependabot Configuration
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "team-leads"
    labels:
      - "dependencies"

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

## Private Registries

```
┌──────────────────────────────────────────────────────────────────┐
│        PRIVATE REGISTRY SETUP                                     │
│                                                                   │
│  Public Registry          Private Registry                       │
│  ┌──────────────┐         ┌──────────────┐                      │
│  │  npmjs.com   │         │  Artifactory │                      │
│  │  pypi.org    │         │  Nexus       │                      │
│  │  Maven Ctrl  │         │  GitHub Pkgs │                      │
│  └──────┬───────┘         └──────┬───────┘                      │
│         │                        │                               │
│         └────────┬───────────────┘                               │
│                  ▼                                                │
│         ┌──────────────┐                                         │
│         │  Your Build  │                                         │
│         └──────────────┘                                         │
│                                                                   │
│  Use a proxy/mirror to cache public packages and host private   │
└──────────────────────────────────────────────────────────────────┘
```

```bash
# npm — configure private registry
npm config set registry https://nexus.example.com/repository/npm-group/
echo "//nexus.example.com/:_authToken=${NPM_TOKEN}" > .npmrc

# pip — configure private index
pip install --index-url https://nexus.example.com/repository/pypi/simple/ mypackage

# Maven — settings.xml
# <mirrors><mirror>
#   <id>nexus</id>
#   <url>https://nexus.example.com/repository/maven-public/</url>
#   <mirrorOf>*</mirrorOf>
# </mirror></mirrors>
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| "Package not found" | Wrong registry URL | Check `.npmrc`, pip config, Maven settings |
| Version conflict | Incompatible dependency ranges | Use lockfile, resolve manually with `overrides` |
| Vulnerability alert blocks build | Known CVE in dependency | Update dep, or add exception if false positive |
| Slow dependency install | No caching | Cache `node_modules`, `~/.m2`, `~/.cache/pip` |
| Lockfile out of sync | Edited manifest without updating lock | Run `npm install` / `poetry lock` and commit |

---

## Summary Table

| Concept | Description |
|---|---|
| **Manifest** | Declares dependencies and version ranges (package.json) |
| **Lockfile** | Records exact resolved versions for reproducibility |
| **CI Install** | Use lockfile-based install (`npm ci`, `--frozen-lockfile`) |
| **Vulnerability Scan** | Automated detection of known CVEs in dependencies |
| **Auto-Updates** | Bots like Dependabot/Renovate create PRs for updates |
| **Private Registry** | Host internal packages, proxy/cache public ones |

---

## Quick Revision Questions

1. **What is the difference between a manifest file and a lockfile?**  
   *A manifest (package.json) declares dependency ranges; a lockfile (package-lock.json) records exact resolved versions for reproducible builds.*

2. **Why should you use `npm ci` instead of `npm install` in CI?**  
   *`npm ci` installs exactly from the lockfile, ensuring deterministic builds and failing if the lockfile is out of sync.*

3. **Name three dependency vulnerability scanning tools.**  
   *npm audit, Snyk, Dependabot, OWASP Dependency-Check, Trivy, pip-audit.*

4. **What is Dependabot and how does it work?**  
   *A GitHub bot that monitors dependencies, detects available updates, and automatically creates pull requests to update them.*

5. **Why would you use a private package registry?**  
   *To host proprietary internal packages, cache public packages for speed/reliability, and control which packages are allowed.*

6. **What happens if you don't commit your lockfile?**  
   *Builds become non-deterministic — different runs may install different versions, causing "works on my machine" issues.*

---

[← Previous: Compilation and Packaging](02-compilation-packaging.md) | [Back to README](../README.md) | [Next: Build Optimization →](04-build-optimization.md)
