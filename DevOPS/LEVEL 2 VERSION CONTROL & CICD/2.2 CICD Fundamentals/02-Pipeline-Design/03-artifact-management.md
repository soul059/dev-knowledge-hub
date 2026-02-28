# Chapter 2.3: Artifact Management

[← Previous: Pipeline as Code](02-pipeline-as-code.md) | [Back to README](../README.md) | [Next: Environment Promotion →](04-environment-promotion.md)

---

## Overview

An **artifact** is the output of a build process — a compiled binary, container image, package, or bundle ready for deployment. **Artifact management** is the practice of storing, versioning, and promoting these outputs through your pipeline. The golden rule: **build once, deploy many times**.

---

## What is an Artifact?

```
┌──────────────────────────────────────────────────────────────┐
│                     BUILD ARTIFACTS                           │
│                                                               │
│  Source Code ──▶ Build Process ──▶ Artifact                  │
│                                                               │
│  Types of Artifacts:                                          │
│  ┌──────────────────────────────────────────────────────────┐│
│  │ Java:     myapp-1.2.0.jar, myapp-1.2.0.war             ││
│  │ Node.js:  dist/ folder, myapp-1.2.0.tgz                ││
│  │ Python:   myapp-1.2.0.whl, myapp-1.2.0.tar.gz          ││
│  │ Go:       myapp (static binary)                          ││
│  │ Docker:   registry.io/myapp:1.2.0 (container image)     ││
│  │ .NET:     myapp.dll, publish/ folder                     ││
│  │ Frontend: dist/ (HTML, CSS, JS bundles)                  ││
│  │ Mobile:   app-release.apk, MyApp.ipa                    ││
│  │ Infra:    terraform plan output, Helm charts             ││
│  └──────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

---

## Artifact Flow Through Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARTIFACT LIFECYCLE                                      │
│                                                                      │
│  ┌────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐    │
│  │ BUILD  │───▶│ PUBLISH  │───▶│  STORE   │───▶│   PROMOTE    │    │
│  │        │    │          │    │          │    │              │    │
│  └────────┘    └──────────┘    └──────────┘    └──────┬───────┘    │
│   Create        Push to         Repository      Pull from          │
│   artifact      registry        (versioned)     store & deploy     │
│                                                      │             │
│                                      ┌───────────────┼───────────┐ │
│                                      ▼               ▼           ▼ │
│                                 ┌────────┐    ┌──────────┐ ┌─────┐│
│                                 │  Dev   │    │ Staging  │ │Prod ││
│                                 └────────┘    └──────────┘ └─────┘│
│                                                                     │
│               SAME artifact in ALL environments                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Artifact Repositories

| Repository | Supports | Self-Hosted | Cloud |
|---|---|---|---|
| **JFrog Artifactory** | All formats (universal) | ✅ | ✅ |
| **Sonatype Nexus** | Maven, npm, Docker, PyPI | ✅ | ✅ |
| **Docker Hub** | Container images | ❌ | ✅ |
| **AWS ECR** | Container images | ❌ | ✅ |
| **Google Artifact Registry** | Containers, packages | ❌ | ✅ |
| **Azure Container Registry** | Container images | ❌ | ✅ |
| **GitHub Packages** | npm, Maven, Docker, NuGet | ❌ | ✅ |
| **npm Registry** | Node.js packages | ❌ | ✅ |
| **PyPI** | Python packages | ❌ | ✅ |

---

## Artifact Versioning Strategies

### Semantic Versioning (SemVer)

```
VERSION: MAJOR.MINOR.PATCH
         │     │     │
         │     │     └── Bug fixes (backward compatible)
         │     └──────── New features (backward compatible)
         └────────────── Breaking changes

Examples:
  1.0.0 → 1.0.1  (patch: bug fix)
  1.0.1 → 1.1.0  (minor: new feature)
  1.1.0 → 2.0.0  (major: breaking change)

Pre-release:
  2.0.0-alpha.1
  2.0.0-beta.3
  2.0.0-rc.1
```

### Git SHA-Based Versioning

```bash
# Use the Git commit SHA as the version tag
docker build -t myapp:$(git rev-parse --short HEAD) .
# Result: myapp:a1b2c3d

# Advantages:
# - Unique per commit
# - Traceable back to exact source code
# - No version conflicts
```

### Hybrid Versioning

```bash
# Combine SemVer + Git SHA + Build Number
VERSION="1.2.0-build.${BUILD_NUMBER}-${GIT_SHA:0:7}"
# Result: 1.2.0-build.42-a1b2c3d
```

---

## Artifact Management in Practice

### Docker Images

```bash
# Build and tag
docker build -t myapp:latest .
docker tag myapp:latest registry.example.com/myapp:1.2.0
docker tag myapp:latest registry.example.com/myapp:$(git rev-parse --short HEAD)

# Push to registry
docker push registry.example.com/myapp:1.2.0
docker push registry.example.com/myapp:a1b2c3d

# Pull for deployment
docker pull registry.example.com/myapp:1.2.0
```

### GitHub Actions — Upload/Download Artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: app-dist
          path: dist/
          retention-days: 30

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-dist
      - run: ./deploy.sh
```

### Maven (Java)

```xml
<!-- pom.xml — Deploy to Nexus -->
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>https://nexus.example.com/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>https://nexus.example.com/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

```bash
# Publish artifact
mvn deploy

# Download artifact
mvn dependency:resolve
```

---

## Artifact Retention & Cleanup

```
┌──────────────────────────────────────────────────┐
│         ARTIFACT RETENTION POLICY                 │
│                                                   │
│  ┌─────────────────────────────────────────────┐ │
│  │ Type           │ Retention    │ Cleanup     │ │
│  │────────────────┼──────────────┼─────────────│ │
│  │ Dev/PR builds  │ 7 days       │ Auto-delete │ │
│  │ Main builds    │ 90 days      │ Auto-delete │ │
│  │ Release tags   │ Forever      │ Manual only │ │
│  │ Snapshots      │ 30 days      │ Auto-delete │ │
│  └─────────────────────────────────────────────┘ │
│                                                   │
│  ⚠️ Without retention: registries fill up fast!  │
│  1 Docker image ≈ 100-500MB                      │
│  50 builds/day × 365 days = 18,000+ images       │
└──────────────────────────────────────────────────┘
```

### Cleanup Example

```bash
# Docker: Remove untagged images older than 30 days
docker image prune --filter "until=720h" --force

# AWS ECR lifecycle policy (JSON)
{
  "rules": [{
    "rulePriority": 1,
    "description": "Keep last 10 images",
    "selection": {
      "tagStatus": "any",
      "countType": "imageCountMoreThan",
      "countNumber": 10
    },
    "action": { "type": "expire" }
  }]
}
```

---

## Artifact Security

```
┌──────────────────────────────────────────────────────────────┐
│            ARTIFACT SECURITY                                  │
│                                                               │
│  1. SIGNING                                                   │
│     Sign artifacts to prove authenticity                      │
│     └── Docker Content Trust / cosign / GPG                  │
│                                                               │
│  2. SCANNING                                                  │
│     Scan for vulnerabilities before deployment                │
│     └── Trivy, Snyk, Grype, Clair                            │
│                                                               │
│  3. IMMUTABILITY                                              │
│     Never overwrite a released artifact                       │
│     └── v1.2.0 once published = final                        │
│                                                               │
│  4. ACCESS CONTROL                                            │
│     Limit who can push/pull artifacts                         │
│     └── RBAC on registry, scoped tokens                      │
│                                                               │
│  5. PROVENANCE                                                │
│     Track where an artifact came from                         │
│     └── SLSA framework, build attestations                   │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Scan Docker image with Trivy
trivy image registry.example.com/myapp:1.2.0

# Sign image with cosign
cosign sign --key cosign.key registry.example.com/myapp:1.2.0
```

---

## Troubleshooting Artifact Management

| Issue | Cause | Solution |
|---|---|---|
| Artifact not found in deploy | Wrong name/version reference | Match artifact names exactly between stages |
| Registry out of space | No retention policy | Add lifecycle/retention rules |
| Slow artifact download | Large artifacts, far registry | Use regional mirrors, optimize image size |
| "Artifact already exists" | Re-publishing same version | Use unique versions (SHA-based) or allow overwrite for snapshots |
| Vulnerability in artifact | Outdated base image / deps | Run security scanning, update dependencies |

---

## Summary Table

| Concept | Description |
|---|---|
| **Artifact** | Output of build process (binary, image, package) |
| **Build Once** | Same artifact deployed to all environments |
| **Versioning** | SemVer, Git SHA, or hybrid strategies |
| **Repository** | Centralized store (Artifactory, ECR, Nexus) |
| **Retention** | Auto-delete old artifacts; keep releases |
| **Security** | Sign, scan, immutability, access control |
| **Promotion** | Move artifact from dev → staging → production |
| **Config** | Externalized — not baked into artifact |

---

## Quick Revision Questions

1. **What is a build artifact?**  
   *The output of a build process — a compiled binary, container image, or packaged bundle ready for deployment.*

2. **Why should you "build once, deploy everywhere"?**  
   *To ensure the exact same code is running in all environments. Rebuilding per environment can introduce subtle differences.*

3. **What is Semantic Versioning and what do the three numbers represent?**  
   *MAJOR.MINOR.PATCH — major for breaking changes, minor for new features (backward compatible), patch for bug fixes.*

4. **Why are artifact retention policies important?**  
   *Without them, registries fill up with old artifacts, wasting storage and increasing costs. Policies auto-clean while preserving releases.*

5. **Name three artifact security practices.**  
   *Signing (authenticity), scanning (vulnerabilities), and immutability (no overwriting released versions).*

6. **What is the difference between a snapshot and a release artifact?**  
   *Snapshots are in-development versions that can be overwritten; release artifacts are final, immutable versions.*

---

[← Previous: Pipeline as Code](02-pipeline-as-code.md) | [Back to README](../README.md) | [Next: Environment Promotion →](04-environment-promotion.md)
