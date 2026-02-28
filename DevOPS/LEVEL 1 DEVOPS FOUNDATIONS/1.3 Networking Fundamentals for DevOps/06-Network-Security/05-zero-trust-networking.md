# Chapter 5: Zero Trust Networking

## Overview

**Zero Trust** is a security model based on the principle: **"Never trust, always verify."** Unlike traditional perimeter-based security (castle-and-moat), Zero Trust assumes that threats exist both inside and outside the network. Every request must be authenticated, authorized, and encrypted — regardless of where it originates.

---

## 5.1 Traditional vs Zero Trust

```
┌──────────────────────────────────────────────────────────────┐
│       PERIMETER (CASTLE-AND-MOAT) vs ZERO TRUST             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL (Perimeter Security):                           │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  FIREWALL / VPN BOUNDARY                            │     │
│  │  ┌─────────────────────────────────────────────┐    │     │
│  │  │     TRUSTED ZONE (inside = safe)            │    │     │
│  │  │                                             │    │     │
│  │  │  Web ←→ App ←→ DB ←→ Admin                  │    │     │
│  │  │      Everything trusts everything           │    │     │
│  │  │                                             │    │     │
│  │  │  ⚠️ Attacker gets past firewall →            │    │     │
│  │  │     has access to EVERYTHING                │    │     │
│  │  └─────────────────────────────────────────────┘    │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  ZERO TRUST:                                                 │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  NO TRUSTED ZONE — verify every request             │     │
│  │                                                     │     │
│  │  ┌─────┐  auth  ┌─────┐  auth  ┌─────┐             │     │
│  │  │ Web │──mTLS──│ App │──mTLS──│ DB  │             │     │
│  │  └─────┘  ✓     └─────┘  ✓     └─────┘             │     │
│  │     │              │              │                 │     │
│  │     ▼              ▼              ▼                 │     │
│  │  Identity       Identity       Identity             │     │
│  │  verified       verified       verified             │     │
│  │                                                     │     │
│  │  ✅ Attacker compromises Web →                       │     │
│  │     still can't access DB (different identity)      │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Zero Trust Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Verify explicitly** | Authenticate and authorize every request | mTLS, OAuth2, API keys |
| **Least privilege access** | Grant minimum permissions needed | RBAC, short-lived tokens |
| **Assume breach** | Design as if attackers are already inside | Micro-segmentation, encryption |
| **Never trust, always verify** | No implicit trust based on network location | Identity-based access |
| **Minimize blast radius** | Limit damage from any single compromise | Segmentation, rotation |
| **Continuous verification** | Re-validate trust throughout session | Token expiry, anomaly detection |

---

## 5.3 Zero Trust Architecture Components

```
┌──────────────────────────────────────────────────────────────┐
│          ZERO TRUST ARCHITECTURE                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                 CONTROL PLANE                          │  │
│  │  ┌──────────┐  ┌────────────┐  ┌──────────────────┐   │  │
│  │  │ Identity │  │  Policy    │  │  Certificate     │   │  │
│  │  │ Provider │  │  Engine    │  │  Authority       │   │  │
│  │  │ (IdP)    │  │  (OPA/     │  │  (Vault/         │   │  │
│  │  │          │  │   Cedar)   │  │   step-ca)       │   │  │
│  │  └────┬─────┘  └─────┬──────┘  └───────┬──────────┘   │  │
│  │       │              │                 │              │  │
│  └───────┼──────────────┼─────────────────┼──────────────┘  │
│          │              │                 │                  │
│  ┌───────┼──────────────┼─────────────────┼──────────────┐  │
│  │       ▼              ▼                 ▼              │  │
│  │                 DATA PLANE                            │  │
│  │                                                       │  │
│  │  ┌──────────┐  mTLS    ┌──────────┐  mTLS  ┌──────┐  │  │
│  │  │ Service  │─────────▶│ Service  │───────▶│  DB  │  │  │
│  │  │   A      │ verified │   B      │verified│      │  │  │
│  │  └──────────┘          └──────────┘        └──────┘  │  │
│  │       │                      │                        │  │
│  │       ▼                      ▼                        │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Sidecar Proxy (Envoy)                          │ │  │
│  │  │  - Enforces mTLS                                │ │  │
│  │  │  - Checks authorization policy                  │ │  │
│  │  │  - Rotates certificates automatically           │ │  │
│  │  │  - Logs all access attempts                     │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.4 Implementing Zero Trust with Service Mesh (Istio)

```yaml
# 1. Enable strict mTLS for entire mesh
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system  # Mesh-wide
spec:
  mtls:
    mode: STRICT  # All traffic must be mTLS

---
# 2. Authorization Policy — only frontend can call backend
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: backend-policy
  namespace: backend
spec:
  selector:
    matchLabels:
      app: api-server
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/frontend/sa/nginx"]  # Service identity
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/*"]

---
# 3. Authorization Policy — deny all by default
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: backend
spec:
  {}  # Empty spec = deny all traffic

---
# 4. Authorization Policy — allow health checks
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-health
  namespace: backend
spec:
  selector:
    matchLabels:
      app: api-server
  rules:
    - to:
        - operation:
            methods: ["GET"]
            paths: ["/health", "/ready"]
```

---

## 5.5 Identity-Based Access (BeyondCorp Model)

```
┌──────────────────────────────────────────────────────────────┐
│       GOOGLE BEYONDCORP MODEL                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Traditional VPN Access:                                     │
│  User → VPN → Internal Network → All Resources              │
│  ⚠️ VPN = keys to the kingdom                                │
│                                                              │
│  BeyondCorp (Zero Trust):                                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │  User Request                                          │  │
│  │     │                                                  │  │
│  │     ▼                                                  │  │
│  │  ┌──────────────────┐                                  │  │
│  │  │ Access Proxy     │ ← All requests go through proxy  │  │
│  │  │ (Identity-Aware) │                                  │  │
│  │  └────────┬─────────┘                                  │  │
│  │           │                                            │  │
│  │     Check all factors:                                 │  │
│  │     ✓ User identity (SSO/MFA)                          │  │
│  │     ✓ Device trust (managed? patched?)                 │  │
│  │     ✓ Location/context                                 │  │
│  │     ✓ Resource sensitivity                             │  │
│  │     ✓ Time of day / behavior                           │  │
│  │           │                                            │  │
│  │     ┌─────▼─────┐                                      │  │
│  │     │ Allow?    │                                      │  │
│  │     │ Yes → ──▶ Specific Resource (not whole network)  │  │
│  │     │ No  → ──▶ Denied                                 │  │
│  │     └───────────┘                                      │  │
│  │                                                        │  │
│  │  No VPN needed. Works from any network.                │  │
│  │  Access is per-resource, not per-network.              │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Implementations:                                            │
│  - Google BeyondCorp Enterprise                              │
│  - Cloudflare Access                                         │
│  - Zscaler Private Access                                    │
│  - Tailscale                                                 │
│  - Teleport                                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.6 Zero Trust for DevOps Pipelines

```
┌──────────────────────────────────────────────────────────────┐
│       ZERO TRUST CI/CD PIPELINE                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │ Developer │    │  CI/CD   │    │Production│               │
│  │   Push    │───▶│ Pipeline │───▶│  Deploy  │               │
│  └──────────┘    └──────────┘    └──────────┘               │
│       │               │               │                     │
│       ▼               ▼               ▼                     │
│  ✓ Signed commits  ✓ OIDC tokens   ✓ Short-lived creds     │
│  ✓ Branch protect  ✓ No long-lived  ✓ Workload identity     │
│  ✓ Code review       secrets       ✓ Least privilege       │
│  ✓ MFA enabled     ✓ Isolated      ✓ Audit logging         │
│                      runners       ✓ mTLS                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### GitHub Actions with OIDC (No Stored Secrets)

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

permissions:
  id-token: write   # Required for OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Authenticate with OIDC — no stored AWS keys!
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-deploy
          aws-region: us-east-1
          # No access keys! Uses short-lived OIDC token

      - name: Deploy
        run: |
          aws sts get-caller-identity  # Verify identity
          terraform apply -auto-approve
```

### AWS IAM Role for GitHub OIDC

```hcl
# Trust policy — only allow specific repo/branch
resource "aws_iam_role" "github_deploy" {
  name = "github-deploy"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = "arn:aws:iam::123456789:oidc-provider/token.actions.githubusercontent.com"
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
        }
        StringLike = {
          # Only main branch of specific repo can assume this role
          "token.actions.githubusercontent.com:sub" = "repo:myorg/myrepo:ref:refs/heads/main"
        }
      }
    }]
  })
}

# Attach minimal permissions
resource "aws_iam_role_policy_attachment" "deploy" {
  role       = aws_iam_role.github_deploy.name
  policy_arn = aws_iam_policy.deploy_minimal.arn
}
```

---

## 5.7 Secret Management in Zero Trust

```
┌──────────────────────────────────────────────────────────────┐
│         SECRET MANAGEMENT HIERARCHY                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ❌ WORST: Hardcoded in code / environment variables          │
│     password = "admin123"                                    │
│                                                              │
│  ⚠️  BETTER: Encrypted secrets in CI/CD                       │
│     GitHub Secrets, GitLab CI Variables                       │
│                                                              │
│  ✅ GOOD: Centralized secret manager                          │
│     AWS Secrets Manager, HashiCorp Vault                     │
│     → Rotation, auditing, access control                     │
│                                                              │
│  ✅ BEST: No secrets at all (workload identity)               │
│     OIDC federation, IAM roles, service accounts             │
│     → Short-lived tokens, no credentials to steal            │
│                                                              │
│  Zero Trust Secret Principles:                               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ 1. Use workload identity (OIDC) when possible       │     │
│  │ 2. Rotate secrets automatically (≤90 days)          │     │
│  │ 3. Never store secrets in code or config files      │     │
│  │ 4. Use short-lived tokens (hours, not months)       │     │
│  │ 5. Audit all secret access                          │     │
│  │ 6. Use different secrets per environment            │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Services can't communicate after mTLS | Certificate not issued or expired | Check cert-manager / Vault CA status |
| OIDC authentication fails | Trust policy misconfigured | Verify audience and subject claims |
| Over-restrictive policies blocking legit traffic | Deny-all without proper allow rules | Add specific allow policies, check logs |
| mTLS not enforcing in Istio | PeerAuthentication in PERMISSIVE mode | Change to STRICT mode |
| Users locked out after removing VPN | Identity-aware proxy not configured | Deploy proxy (Cloudflare Access, etc.) first |
| Secret rotation breaks services | Hard-coded old credentials | Use dynamic secrets from Vault/Secrets Manager |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Zero Trust | Never trust, always verify — no trusted zones |
| BeyondCorp | Identity-aware access proxy, no VPN needed |
| mTLS | Both client and server authenticate — service mesh |
| Least Privilege | Minimum permissions, short-lived credentials |
| OIDC Federation | No stored secrets — identity-based access |
| Service Mesh | Istio/Linkerd enforce mTLS + authorization |
| Workload Identity | Services prove identity without static secrets |
| Micro-Segmentation | Every workload isolated, explicit allow |

---

## Quick Revision Questions

1. **How does Zero Trust differ from traditional perimeter security?**
2. **What are the core principles of Zero Trust networking?**
3. **Explain how a service mesh (Istio) implements Zero Trust between microservices.**
4. **What is the BeyondCorp model and how does it eliminate VPNs?**
5. **How does OIDC federation allow CI/CD pipelines to access cloud resources without stored credentials?**
6. **Why are short-lived tokens better than long-lived API keys in a Zero Trust model?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Network Segmentation](04-network-segmentation.md) | [README](../README.md) | [Common Vulnerabilities →](06-common-vulnerabilities.md) |
