# Image Security

## Unit 9 - Topic 3 | Container Security

---

## Overview

**Container images** are the blueprints for containers. A compromised image means every container spawned from it is compromised. Image security covers **building secure images**, **scanning for vulnerabilities**, **signing images**, and **managing image supply chains**.

---

## 1. Image Security Risks

```
┌──────────────────────────────────────────────────────────────┐
│              IMAGE SECURITY RISKS                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. VULNERABLE BASE IMAGES                                   │
│     Using outdated or unpatched base images                  │
│                                                              │
│  2. MALICIOUS IMAGES                                         │
│     Backdoored images from untrusted registries              │
│                                                              │
│  3. EMBEDDED SECRETS                                         │
│     API keys, passwords, tokens in image layers              │
│                                                              │
│  4. EXCESSIVE PACKAGES                                       │
│     Unnecessary software increases attack surface            │
│                                                              │
│  5. ROOT USER                                                │
│     Running as root inside the container                     │
│                                                              │
│  6. SUPPLY CHAIN ATTACKS                                     │
│     Compromised dependencies or build pipeline               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Building Secure Dockerfiles

```dockerfile
# === INSECURE Dockerfile ===
FROM ubuntu:latest              # ❌ Mutable tag, large image
RUN apt-get update && apt-get install -y python3 curl wget vim
COPY . /app                     # ❌ Copies everything including secrets
ENV API_KEY=sk-1234567890       # ❌ Secret in environment variable!
USER root                       # ❌ Running as root
CMD ["python3", "/app/server.py"]

# === SECURE Dockerfile ===
FROM python:3.11-slim@sha256:abc123...   # ✅ Specific version + digest
RUN groupadd -r appuser && \
    useradd -r -g appuser appuser        # ✅ Create non-root user
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appuser . .         # ✅ Proper ownership
RUN chmod -R 555 /app                     # ✅ Read-only application
USER appuser                              # ✅ Run as non-root
HEALTHCHECK --interval=30s CMD curl -f http://localhost:8080/ || exit 1
EXPOSE 8080
CMD ["python3", "server.py"]
```

---

## 3. Multi-Stage Builds (Minimize Attack Surface)

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

# Stage 2: Production (minimal image)
FROM scratch                     # ✅ Empty base image!
COPY --from=builder /app/server /server
USER 65534:65534                 # ✅ Nobody user
EXPOSE 8080
ENTRYPOINT ["/server"]

# Result: Final image has ONLY the compiled binary
# No shell, no package manager, no extra tools = minimal attack surface
```

---

## 4. Image Scanning

```bash
# === TRIVY (Most popular, free) ===
trivy image nginx:latest
trivy image --severity HIGH,CRITICAL myapp:v1
trivy image --format json -o results.json myapp:v1

# === Docker Scout ===
docker scout cves nginx:latest
docker scout quickview nginx:latest

# === Grype ===
grype nginx:latest
grype nginx:latest --only-fixed        # Show only fixable vulns

# === Scan during CI/CD ===
# GitHub Actions example:
# - name: Scan image
#   uses: aquasecurity/trivy-action@master
#   with:
#     image-ref: myapp:${{ github.sha }}
#     exit-code: 1                       # Fail build on HIGH/CRITICAL
#     severity: HIGH,CRITICAL

# === SCAN FILESYSTEM (for secrets) ===
trivy fs --security-checks secret .     # Scan for leaked secrets
trufflehog docker --image myapp:latest  # Deep secret scanning
```

---

## 5. Image Best Practices

| Practice | Why |
|----------|-----|
| **Use minimal base images** | `alpine`, `slim`, `distroless`, `scratch` |
| **Pin versions** | `python:3.11-slim` not `python:latest` |
| **Use digests** | `nginx@sha256:abc...` — immutable reference |
| **Multi-stage builds** | Remove build tools from final image |
| **Non-root user** | `USER appuser` in Dockerfile |
| **No secrets in images** | Use Docker secrets or env injection |
| **Scan regularly** | Trivy, Grype, Docker Scout |
| **Sign images** | Docker Content Trust / cosign |
| **.dockerignore** | Exclude .git, secrets, test files |

```bash
# .dockerignore example
.git
.env
*.key
*.pem
node_modules
tests/
docker-compose.yml
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Base image** | Use minimal (alpine/slim/distroless) |
| **Secrets** | Never embed in image layers |
| **Scanning** | Trivy, Grype — scan in CI/CD |
| **Multi-stage** | Build tools ≠ production image |
| **Non-root** | Always `USER nonroot` |
| **Pin versions** | Avoid `:latest` tag |

---

## Quick Revision Questions

1. **Why should you avoid `FROM ubuntu:latest`?**
2. **How does a multi-stage build improve security?**
3. **What tool scans container images for vulnerabilities?**
4. **Why are secrets in Docker images dangerous even if deleted?**
5. **What is the most minimal Docker base image possible?**

---

[← Previous: Container Isolation](02-container-isolation.md) | [Next: Runtime Security →](04-runtime-security.md)

---

*Unit 9 - Topic 3 of 5 | [Back to README](../README.md)*
