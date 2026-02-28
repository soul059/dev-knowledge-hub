# Chapter 6.5: Container Registries Security

## Overview

Container registries store and distribute images — making them a critical part of the supply chain. A compromised registry can serve malicious images to all environments. This chapter covers registry hardening, image signing, content trust, and access control.

---

## Registry Architecture

```
CONTAINER REGISTRY SECURITY:

  Developer                    CI/CD Pipeline
     │                              │
     ▼                              ▼
  ┌──────────┐   docker push   ┌────────────────┐
  │ Build     │ ──────────────▶ │   Container    │
  │ Image     │                 │   Registry     │
  └──────────┘                 │                │
                                │ ┌────────────┐ │
  Verification:                 │ │ Scanning   │ │ ─── Trivy / Clair
  ┌──────────┐                 │ │ Engine     │ │
  │ Cosign    │ ◀─verification─│ ├────────────┤ │
  │ / Notary  │                 │ │ Signing    │ │ ─── Cosign / Notary
  └──────────┘                 │ │ Service    │ │
                                │ ├────────────┤ │
  ┌──────────┐   docker pull   │ │ Access     │ │ ─── RBAC / Auth
  │ Runtime   │ ◀──────────────│ │ Control    │ │
  │ (K8s)     │                 │ └────────────┘ │
  └──────────┘                 └────────────────┘
```

---

## Registry Access Control

### AWS ECR

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPush",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:role/ci-cd-role"
      },
      "Action": [
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:BatchCheckLayerAvailability"
      ]
    },
    {
      "Sid": "AllowPull",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:role/eks-node-role"
      },
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:GetAuthorizationToken"
      ]
    },
    {
      "Sid": "DenyUnscannedImages",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "ecr:PutImage",
      "Condition": {
        "StringNotEquals": {
          "ecr:image-tag-mutability": "IMMUTABLE"
        }
      }
    }
  ]
}
```

```bash
# Enable image scanning on push (ECR)
aws ecr put-image-scanning-configuration \
    --repository-name myapp \
    --image-scanning-configuration scanOnPush=true

# Enable tag immutability
aws ecr put-image-tag-mutability \
    --repository-name myapp \
    --image-tag-mutability IMMUTABLE

# Set lifecycle policy (clean old images)
aws ecr put-lifecycle-policy \
    --repository-name myapp \
    --lifecycle-policy-text '{
      "rules": [
        {
          "rulePriority": 1,
          "description": "Keep only 10 images",
          "selection": {
            "tagStatus": "any",
            "countType": "imageCountMoreThan",
            "countNumber": 10
          },
          "action": { "type": "expire" }
        }
      ]
    }'
```

### Harbor (Self-Hosted)

```yaml
# Harbor configuration for security
# harbor.yml
hostname: registry.company.com
https:
  port: 443
  certificate: /certs/registry.crt
  private_key: /certs/registry.key

# Enable vulnerability scanning
trivy:
  ignore_unfixed: true
  security_check: vuln,config,secret
  severity: CRITICAL,HIGH

# Enable content trust (Notary)
notary:
  enabled: true

# Enable automatic scanning
scan:
  scan_on_push: true

# RBAC: Project-level access
# Admin → Full access
# Developer → Push + Pull
# Guest → Pull only
```

---

## Image Signing with Cosign

```
IMAGE SIGNING WORKFLOW:

  Build          Sign           Store          Verify
  ┌────┐      ┌────────┐    ┌──────────┐   ┌──────────┐
  │ CI │ ───▶ │ Cosign │ ──▶│ Registry │ ──▶│ Admission│
  │    │      │ Sign   │    │ +Signature│   │ Controller│
  └────┘      └────────┘    └──────────┘   └──────────┘
                 │                             │
              Private Key               Public Key
              (KMS/Vault)              (Policy Config)
```

```bash
# Install Cosign
go install github.com/sigstore/cosign/v2/cmd/cosign@latest

# Generate key pair
cosign generate-key-pair

# Sign an image
cosign sign --key cosign.key registry.company.com/myapp:1.0.0

# Sign with KMS (recommended for production)
cosign sign --key awskms:///arn:aws:kms:us-east-1:123456789:key/abc-123 \
    registry.company.com/myapp:1.0.0

# Verify an image
cosign verify --key cosign.pub registry.company.com/myapp:1.0.0

# Verify image in CI/CD
cosign verify --key cosign.pub \
    --certificate-identity=ci@company.com \
    --certificate-oidc-issuer=https://github.com/login/oauth \
    registry.company.com/myapp:1.0.0
```

### Keyless Signing (Sigstore / Fulcio)

```bash
# Keyless signing with Sigstore (no key management needed!)
# Uses OIDC identity (GitHub Actions, Google, etc.)

# In GitHub Actions — automatic identity
cosign sign registry.company.com/myapp:1.0.0

# Verify keyless signature
cosign verify \
    --certificate-identity=https://github.com/company/myapp/.github/workflows/build.yml@refs/heads/main \
    --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
    registry.company.com/myapp:1.0.0
```

### GitHub Actions — Sign and Push

```yaml
name: Build, Sign, Push
on:
  push:
    branches: [main]

permissions:
  contents: read
  packages: write
  id-token: write    # Required for keyless signing

jobs:
  build-sign:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: sigstore/cosign-installer@v3

      - name: Login to Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Sign Image (Keyless)
        run: |
          cosign sign --yes \
            ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
```

---

## Docker Content Trust (DCT)

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Push a trusted image (automatically signs with Notary)
docker push registry.company.com/myapp:1.0.0

# Pull only trusted images
docker pull registry.company.com/myapp:1.0.0
# Fails if image is not signed!

# Delegate signing to CI/CD
docker trust signer add --key ci-key.pub cibot registry.company.com/myapp
docker trust sign registry.company.com/myapp:1.0.0

# Inspect trust data
docker trust inspect --pretty registry.company.com/myapp
```

---

## Admission Control (Kubernetes)

```yaml
# Kyverno policy: Only allow signed images
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  background: false
  rules:
    - name: check-image-signature
      match:
        any:
          - resources:
              kinds: ["Pod"]
      verifyImages:
        - imageReferences:
            - "ghcr.io/company/*"
            - "registry.company.com/*"
          attestors:
            - entries:
                - keyless:
                    issuer: "https://token.actions.githubusercontent.com"
                    subject: "https://github.com/company/*"

---
# OPA Gatekeeper: Only allow images from trusted registries
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata:
  name: allowed-repos
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    repos:
      - "ghcr.io/company/"
      - "registry.company.com/"
      # Block: docker.io, quay.io, etc.
```

---

## Tag Immutability and Digest Pinning

```
TAG MUTABILITY RISK:

  Time T1:  myapp:v1.0 ──▶ Image A (safe, scanned)
                             sha256:abc123...

  Time T2:  Attacker pushes malicious image with SAME tag
            myapp:v1.0 ──▶ Image B (compromised!)
                             sha256:xyz789...

  SOLUTION: Use immutable tags or digest pinning

  Immutable: myapp:v1.0  ← Cannot be overwritten
  Digest:    myapp@sha256:abc123...  ← Always the exact image
```

```yaml
# Kubernetes deployment with digest pinning
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          # Use digest instead of tag
          image: registry.company.com/myapp@sha256:a1b2c3d4e5f6...
          imagePullPolicy: Always
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `denied: requested access to the resource is denied` | Missing registry permissions | Check IAM role / RBAC; ensure push/pull permissions |
| Cosign verify fails | Wrong public key or unsigned image | Confirm key matches; sign the image first |
| DCT blocks pull | Image not signed | Sign with `docker trust sign` or disable DCT temporarily |
| Scan results not appearing | Scan-on-push not enabled | Enable scan on push in registry (ECR/Harbor) settings |
| Old images accumulating | No lifecycle policy | Set retention rules: keep N images; expire old tags |

---

## Summary Table

| Security Control | Purpose |
|-----------------|---------|
| **RBAC on registry** | Limit who can push/pull images |
| **Scan on push** | Automatically scan images when pushed |
| **Tag immutability** | Prevent tag overwriting attacks |
| **Image signing (Cosign)** | Verify image authenticity and integrity |
| **Keyless signing (Sigstore)** | Sign images using CI/CD identity, no keys to manage |
| **Admission control** | Block unsigned/unscanned images in Kubernetes |
| **Digest pinning** | Reference exact image content, immune to tag changes |

---

## Quick Revision Questions

1. **Why is tag immutability important?**
   > Without immutability, an attacker (or even accidental push) can replace a tagged image with a different one. All deployments using that tag would pull the compromised image. Immutable tags prevent overwrites, ensuring the same tag always references the same image.

2. **How does Cosign keyless signing work?**
   > Keyless signing uses OIDC identity providers (like GitHub Actions) instead of managing long-lived keys. Cosign gets a short-lived certificate from Sigstore's Fulcio CA, proving the signer's identity. The signing event is logged in Rekor (transparency log). Verification checks the OIDC issuer and subject identity.

3. **What is digest pinning and when should you use it?**
   > Digest pinning references an image by its content hash (`@sha256:...`) instead of a mutable tag. The digest is immutable — it always points to the exact same image content. Use it in production deployments, especially in Kubernetes, for guaranteed reproducibility.

4. **How do admission controllers enforce registry security in Kubernetes?**
   > Admission controllers (Kyverno, OPA Gatekeeper) intercept pod creation requests and validate the container image against policies: allowed registries, required signatures, scan results. Non-compliant pods are rejected before they can run.

5. **What is the principle difference between Docker Content Trust and Cosign?**
   > DCT uses Notary (TUF-based) and is Docker-specific, requiring a Notary server. Cosign is OCI-native, stores signatures alongside images in any OCI registry, supports keyless signing via Sigstore, and works with Kubernetes admission controllers. Cosign is the modern recommended approach.

---

[← Previous: 6.4 Runtime Security](04-runtime-security.md) | [Next: 6.6 Container Security Tools →](06-container-security-tools.md)

[Back to Table of Contents](../README.md)
