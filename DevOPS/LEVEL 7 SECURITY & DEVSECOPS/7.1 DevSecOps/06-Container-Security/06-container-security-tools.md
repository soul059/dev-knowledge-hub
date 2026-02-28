# Chapter 6.6: Container Security Tools

## Overview

This chapter provides a comprehensive overview of the container security tooling ecosystem — from image scanning to runtime protection. It covers tool selection, integration strategies, and a unified pipeline approach.

---

## Container Security Tool Landscape

```
CONTAINER SECURITY TOOL CATEGORIES:

  BUILD TIME                 DEPLOY TIME              RUNTIME
  ┌──────────────┐          ┌──────────────┐         ┌──────────────┐
  │ Image Scan   │          │ Admission    │         │ Runtime      │
  │ ┌──────────┐ │          │ Control      │         │ Monitoring   │
  │ │ Trivy    │ │          │ ┌──────────┐ │         │ ┌──────────┐ │
  │ │ Grype    │ │          │ │ Kyverno  │ │         │ │ Falco    │ │
  │ │ Snyk     │ │          │ │ OPA/GK   │ │         │ │ Sysdig   │ │
  │ └──────────┘ │          │ └──────────┘ │         │ │ Tetragon │ │
  │              │          │              │         │ └──────────┘ │
  │ Dockerfile   │          │ Signing      │         │              │
  │ Lint         │          │ ┌──────────┐ │         │ Network      │
  │ ┌──────────┐ │          │ │ Cosign   │ │         │ Security     │
  │ │ Hadolint │ │          │ │ Notary   │ │         │ ┌──────────┐ │
  │ │ Dockle   │ │          │ └──────────┘ │         │ │ Cilium   │ │
  │ └──────────┘ │          │              │         │ │ Calico   │ │
  │              │          │ Registry     │         │ └──────────┘ │
  │ SBOM Gen     │          │ ┌──────────┐ │         │              │
  │ ┌──────────┐ │          │ │ Harbor   │ │         │ Forensics    │
  │ │ Syft     │ │          │ │ ECR      │ │         │ ┌──────────┐ │
  │ │ Trivy    │ │          │ │ GCR/GAR  │ │         │ │ Sysdig   │ │
  │ └──────────┘ │          │ └──────────┘ │         │ │ Capture  │ │
  └──────────────┘          └──────────────┘         └──────────────┘
```

---

## Tool Comparison

| Tool | Category | Type | Strengths | Limitations |
|------|----------|------|-----------|-------------|
| **Trivy** | Scanner | OSS | All-in-one (vuln, config, secret, SBOM), fast | Less deep analysis than commercial |
| **Grype** | Scanner | OSS | Pairs with Syft, fast, good accuracy | Vuln scanning only |
| **Snyk Container** | Scanner | Commercial | Developer-friendly, fix advice, IDE plugin | Paid for full features |
| **Clair** | Scanner | OSS | Registry integration, ClairV4 modernized | Complex setup |
| **Hadolint** | Linter | OSS | Dockerfile best practices, ShellCheck integration | Dockerfile only |
| **Dockle** | Linter | OSS | CIS Docker Benchmark, image analysis | Image-level only |
| **Cosign** | Signing | OSS | Sigstore ecosystem, keyless, OCI-native | Newer tooling |
| **Falco** | Runtime | OSS | eBPF-based, real-time, rich rules | Resource overhead |
| **Syft** | SBOM | OSS | Multi-format (SPDX, CycloneDX), fast | SBOM generation only |

---

## Trivy — All-in-One Scanner

```bash
# Vulnerability scan
trivy image --severity HIGH,CRITICAL myapp:1.0.0

# Configuration scan (Dockerfile / K8s manifests)
trivy config ./Dockerfile
trivy config ./k8s/

# Secret detection in image
trivy image --scanners secret myapp:1.0.0

# SBOM generation
trivy image --format spdx-json -o sbom.json myapp:1.0.0

# Full scan pipeline (all scanners)
trivy image --scanners vuln,misconfig,secret \
    --severity HIGH,CRITICAL \
    --format json \
    --output trivy-report.json \
    myapp:1.0.0

# Exit code control for CI/CD
trivy image --exit-code 1 \
    --severity CRITICAL \
    --ignore-unfixed \
    myapp:1.0.0
```

### Trivy GitHub Actions

```yaml
name: Container Security Scan
on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Trivy Vulnerability Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: sarif
          output: trivy-results.sarif
          severity: CRITICAL,HIGH
          exit-code: 1

      - name: Upload to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: trivy-results.sarif
```

---

## Grype + Syft Pipeline

```bash
# Generate SBOM with Syft
syft myapp:1.0.0 -o spdx-json > sbom.json

# Scan SBOM with Grype
grype sbom:sbom.json --output json > grype-results.json

# Direct image scan
grype myapp:1.0.0 --fail-on critical

# Scan with configuration
grype myapp:1.0.0 \
    --only-fixed \
    --output table \
    --fail-on high
```

```yaml
# .grype.yaml configuration
check-for-app-update: false
fail-on-severity: high
output: "table"
ignore:
  - vulnerability: CVE-2023-xxxxx
    reason: "Not exploitable in our context"
  - package:
      name: openssl
      version: 1.1.1*
      type: deb
```

---

## Dockle — CIS Benchmark Checker

```bash
# Install and run Dockle
dockle myapp:1.0.0

# Example output:
# FATAL  - CIS-DI-0010: Do not store secrets in DOCKERENV
# WARN   - CIS-DI-0001: Create a user for the container
# WARN   - DKL-DI-0006: Avoid latest tag
# INFO   - CIS-DI-0005: Enable Content Trust
# INFO   - CIS-DI-0008: Confirm safety of setuid/setgid files
# PASS   - CIS-DI-0003: Do not install unnecessary packages

# JSON output for CI/CD
dockle --format json --output dockle-results.json myapp:1.0.0

# Set exit code and severity
dockle --exit-code 1 --exit-level fatal myapp:1.0.0
```

---

## Unified Security Pipeline

```yaml
# Complete container security pipeline
name: Container Security Pipeline
on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read
  packages: write
  security-events: write
  id-token: write

jobs:
  # Stage 1: Lint Dockerfile
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          failure-threshold: error

  # Stage 2: Build Image
  build:
    needs: lint
    runs-on: ubuntu-latest
    outputs:
      digest: ${{ steps.build.outputs.digest }}
      image: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Image Metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}

      - name: Build and Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  # Stage 3: Security Scans (parallel)
  vulnerability-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Trivy Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image }}
          format: sarif
          output: trivy.sarif
          severity: CRITICAL,HIGH
          exit-code: 1
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: trivy.sarif

  cis-benchmark:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Dockle CIS Scan
        run: |
          VERSION=$(curl --silent "https://api.github.com/repos/goodwithtech/dockle/releases/latest" | grep '"tag_name"' | sed -E 's/.*"v([^"]+)".*/\1/')
          curl -L -o dockle.tar.gz "https://github.com/goodwithtech/dockle/releases/download/v${VERSION}/dockle_${VERSION}_Linux-64bit.tar.gz"
          tar xzf dockle.tar.gz
          ./dockle --exit-code 1 --exit-level fatal ${{ needs.build.outputs.image }}

  sbom-generate:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          image: ${{ needs.build.outputs.image }}
          format: spdx-json
          output-file: sbom.spdx.json
      - uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.spdx.json

  # Stage 4: Sign (only on main)
  sign:
    needs: [build, vulnerability-scan, cis-benchmark]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: sigstore/cosign-installer@v3
      - name: Sign Image
        run: |
          cosign sign --yes \
            ghcr.io/${{ github.repository }}@${{ needs.build.outputs.digest }}
```

---

## Tool Selection Decision Tree

```
WHICH CONTAINER SECURITY TOOLS DO YOU NEED?

  Start
   │
   ├─ Need vulnerability scanning?
   │   ├─ Budget: Free → Trivy or Grype
   │   ├─ Budget: Paid → Snyk Container
   │   └─ Registry-native → Clair (Harbor) or ECR scanning
   │
   ├─ Need Dockerfile linting?
   │   ├─ During development → Hadolint (IDE plugin)
   │   └─ In CI/CD → Hadolint Action + Dockle
   │
   ├─ Need image signing?
   │   ├─ Keyless (recommended) → Cosign + Sigstore
   │   └─ Key-based → Cosign with KMS
   │
   ├─ Need SBOM generation?
   │   ├─ Standalone → Syft
   │   └─ Integrated → Trivy (scan + SBOM)
   │
   ├─ Need admission control?
   │   ├─ Policy-first → Kyverno
   │   └─ Rego-based → OPA Gatekeeper
   │
   └─ Need runtime protection?
       ├─ Open source → Falco
       ├─ eBPF-native → Tetragon
       └─ Commercial → Sysdig Secure
```

---

## Recommended Tool Stack by Team Size

| Team Size | Image Scanning | Linting | Signing | SBOM | Runtime |
|-----------|---------------|---------|---------|------|---------|
| **Small (1-5)** | Trivy | Hadolint | Cosign keyless | Trivy | — |
| **Medium (5-20)** | Trivy + Snyk | Hadolint + Dockle | Cosign + KMS | Syft + Grype | Falco |
| **Enterprise (20+)** | Snyk + Trivy | Hadolint + Dockle | Cosign + Notary | Syft + SBOM DB | Sysdig Secure |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Trivy scan timeout on large images | Image has many layers / packages | Use `--timeout 15m`; scan base images separately |
| Grype misses vulnerabilities | Outdated database | Run `grype db update` before each scan |
| Hadolint rules too strict | Default rules prevent common patterns | Create `.hadolint.yaml` with appropriate ignores |
| Dockle false positives | Base image inherits issues | Report on base image issues separately; focus on app layers |
| Cosign sign fails in CI | Missing OIDC token permissions | Add `id-token: write` permission in GitHub Actions |

---

## Summary Table

| Tool | Category | Key Capability |
|------|----------|---------------|
| **Trivy** | All-in-one scanner | Vulnerabilities + misconfig + secrets + SBOM |
| **Grype** | Vulnerability scanner | Fast, pairs with Syft for SBOM-based scanning |
| **Hadolint** | Dockerfile linter | Best practices and ShellCheck integration |
| **Dockle** | CIS benchmark checker | Checks built images against CIS Docker Benchmark |
| **Cosign** | Image signing | Keyless signing with Sigstore, OCI-native |
| **Syft** | SBOM generator | SPDX and CycloneDX output, multi-ecosystem |
| **Falco** | Runtime security | eBPF-based syscall monitoring and alerting |

---

## Quick Revision Questions

1. **What makes Trivy an "all-in-one" container security tool?**
   > Trivy scans for vulnerabilities, misconfigurations, secrets, and generates SBOMs — all in a single tool. It supports container images, filesystems, Git repositories, and Kubernetes manifests. This eliminates the need for multiple separate tools in smaller teams.

2. **Why pair Syft with Grype instead of using them independently?**
   > Syft generates a detailed SBOM (Software Bill of Materials) listing every component. Grype then scans this SBOM against vulnerability databases. This separation allows you to store the SBOM as an artifact and re-scan it later without rebuilding the image, enabling historical vulnerability tracking.

3. **What does Dockle check that Hadolint doesn't?**
   > Hadolint lints the Dockerfile source (static analysis of instructions). Dockle checks the *built image* against CIS Docker Benchmark best practices — it detects runtime issues like running as root, stored secrets in environment variables, setuid/setgid binaries, and content trust status that only exist in the final image.

4. **How should a team decide between Trivy and Snyk Container?**
   > Trivy is free, open-source, and excellent for CI/CD integration. Snyk Container offers a developer-friendly UI, fix recommendations, IDE integration, and enterprise features (license compliance, reporting, priority scoring). Small teams typically start with Trivy; enterprises often use both — Trivy for CI/CD gates and Snyk for developer workflows.

5. **What is the recommended minimum container security pipeline?**
   > At minimum: (1) Hadolint to lint the Dockerfile, (2) Trivy to scan the built image for vulnerabilities, (3) Cosign to sign the verified image. This covers Dockerfile quality, vulnerability detection, and image integrity. Add Dockle, Syft/SBOM, and Falco as the team matures.

---

[← Previous: 6.5 Container Registries Security](05-container-registries-security.md) | [Next: 7.1 Cluster Hardening →](../07-Kubernetes-Security/01-cluster-hardening.md)

[Back to Table of Contents](../README.md)
