# Chapter 4: Docker Content Trust

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

Docker Content Trust (DCT) provides cryptographic verification of image integrity and publisher identity. When enabled, Docker only pulls and runs images that have been signed, protecting against tampered or counterfeit images in the supply chain.

---

## 4.1 What is Content Trust?

```
  Without DCT:                   With DCT:
  
  Registry                       Registry
  +----------+                   +----------+
  | Image A  | Pull              | Image A  | Pull
  | (maybe   |----> Container    | + Sig ✓  |----> Verify
  | tampered)|                   +----------+      Signature
  +----------+                        |                |
                                      v                v
  No verification!               Valid? ──> Run ✓
  Could be anything              Invalid? → REJECT ✗
```

```
  Docker Content Trust Components:
  
  +-- The Update Framework (TUF) -------+
  |                                      |
  | Root Key (offline, most important)   |
  |    |                                 |
  |    +-- Targets Key (signs images)    |
  |    |                                 |
  |    +-- Snapshot Key (repo state)     |
  |    |                                 |
  |    +-- Timestamp Key (freshness)     |
  |                                      |
  +--------------------------------------+
  
  Root Key: Keep offline! Controls all other keys
  Targets Key: Signs specific image tags
  Snapshot Key: Tracks signed content
  Timestamp Key: Prevents replay attacks
```

---

## 4.2 Enabling Content Trust

```bash
# Enable DCT globally (environment variable)
export DOCKER_CONTENT_TRUST=1

# Now all push/pull operations require signatures
docker pull nginx:latest        # Only pulls if signed
docker push myimage:v1.0        # Signs before pushing

# Disable temporarily
DOCKER_CONTENT_TRUST=0 docker pull unsigned-image

# Enable per-command
DOCKER_CONTENT_TRUST=1 docker pull nginx:latest
```

```
  DCT Workflow:
  
  Developer (Push):
  +----------+     +----------+     +-----------+
  | Build    |---->| Sign     |---->| Registry  |
  | Image    |     | (private |     | (image +  |
  |          |     |  key)    |     |  signature)|
  +----------+     +----------+     +-----------+
  
  User (Pull):
  +-----------+     +----------+     +----------+
  | Registry  |---->| Verify   |---->| Run      |
  | (image +  |     | (public  |     | Container|
  |  signature)|    |  key)    |     |          |
  +-----------+     +----------+     +----------+
                         |
                    Fails? → Pull blocked!
```

---

## 4.3 Signing Images

```bash
# First push with DCT enabled creates signing keys
export DOCKER_CONTENT_TRUST=1

# Push (creates keys on first use)
docker push myregistry/myimage:v1.0
# First time: prompts for root key passphrase
# and repository signing key passphrase

# Key storage
# ~/.docker/trust/private/  — Private keys (PROTECT THESE!)
# ~/.docker/trust/tuf/      — Metadata

# Delegate signing to CI/CD
docker trust key generate ci-signer
docker trust signer add --key ci-signer.pub ci-signer myregistry/myimage
```

```bash
# View trust data for an image
docker trust inspect --pretty myregistry/myimage

# Revoke trust for a tag
docker trust revoke myregistry/myimage:v1.0
```

---

## 4.4 Notary

```
  Notary Architecture:
  
  Docker CLI
      |
      v
  +---------+     +---------------+
  | Notary  |<--->| Notary Server |
  | Client  |     | (stores TUF   |
  |         |     |  metadata)    |
  +---------+     +------+--------+
                         |
                  +------+--------+
                  | Notary Signer |
                  | (key mgmt)   |
                  +---------------+
  
  Docker Hub uses Notary for DCT
  Self-hosted registries can run Notary too
```

---

## 4.5 Practical DCT Workflow

```bash
# 1. Set up (one-time)
export DOCKER_CONTENT_TRUST=1

# 2. Build
docker build -t myregistry/app:v2.0 .

# 3. Sign and push
docker push myregistry/app:v2.0
# Enter root key passphrase: ********
# Enter repository key passphrase: ********

# 4. Users pull verified images
docker pull myregistry/app:v2.0
# Signature verified ✓

# 5. Attempting to pull unsigned tag fails
docker pull myregistry/app:unsigned-tag
# Error: remote trust data does not exist
```

---

## 4.6 Limitations and Alternatives

```
  DCT Limitations:
  +-----------------------------------------------+
  | • Only verifies tags, not digests              |
  | • No vulnerability information                |
  | • Key management complexity                   |
  | • Not widely adopted outside Docker Hub       |
  +-----------------------------------------------+
  
  Modern Alternatives:
  +-----------------------------------------------+
  | Cosign (Sigstore)                             |
  |   • Keyless signing with OIDC identity        |
  |   • Works with any OCI registry               |
  |   • Integrated with CI/CD (GitHub Actions)    |
  |   • Growing community adoption                |
  +-----------------------------------------------+
  | Notary v2                                     |
  |   • OCI distribution-based                    |
  |   • Simplified key management                 |
  |   • Registry-native signatures                |
  +-----------------------------------------------+
```

```bash
# Cosign example (modern alternative)
# Sign
cosign sign --key cosign.key myregistry/app:v2.0

# Verify
cosign verify --key cosign.pub myregistry/app:v2.0

# Keyless signing (uses OIDC identity)
cosign sign myregistry/app:v2.0
# Opens browser for authentication
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Docker Content Trust** | Cryptographic image signing and verification |
| **TUF** | The Update Framework — security framework behind DCT |
| **Root key** | Master key, keep offline, controls all other keys |
| **Targets key** | Signs specific image tags |
| **DOCKER_CONTENT_TRUST=1** | Enable DCT globally |
| **Notary** | Server-side component for trust metadata |
| **Cosign** | Modern alternative using Sigstore |
| **Image digest** | Content-addressable SHA256 hash (immutable) |

---

## Quick Revision Questions

1. **What does Docker Content Trust protect against?**
   - Tampered images, counterfeit images, and replay attacks. It ensures images come from trusted publishers and haven't been modified

2. **How do you enable Docker Content Trust?**
   - Set `export DOCKER_CONTENT_TRUST=1`. All subsequent pull and push operations will require signed images

3. **What is the root key and why is it important?**
   - The root key is the master key in DCT's trust hierarchy. It controls all other keys. If compromised, an attacker can sign any image. Keep it offline and backed up

4. **What happens when you try to pull an unsigned image with DCT enabled?**
   - The pull fails with an error: "remote trust data does not exist." The image is not downloaded

5. **What is Cosign and how does it differ from DCT?**
   - Cosign (part of Sigstore) is a modern image signing tool that supports keyless signing with OIDC identity, works with any OCI registry, and has simpler key management than DCT

---

[← Previous: Runtime Security](03-runtime-security.md) | [Next: Secrets Management →](05-secrets-management.md)
