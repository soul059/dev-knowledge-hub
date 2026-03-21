# Unit 8: A08 - Software and Data Integrity Failures — Topic 5: Code Signing

## Overview

Code signing uses **digital signatures** to verify the authenticity and integrity of software. It guarantees that the code was published by a known author (authentication) and has not been modified since signing (integrity). Code signing is essential for software distribution, updates, drivers, scripts, and container images.

---

## 1. How Code Signing Works

```
┌─────────────────────────────────────────────────────────────────┐
│                CODE SIGNING PROCESS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIGNING (Developer):                                          │
│  1. Developer writes code and builds binary                    │
│  2. Hash of binary is computed (SHA-256)                       │
│  3. Hash is encrypted with developer's PRIVATE key             │
│  4. Encrypted hash = DIGITAL SIGNATURE                         │
│  5. Signature attached to binary (or separate .sig file)       │
│                                                                 │
│  VERIFICATION (User/System):                                   │
│  1. Receive binary + signature                                 │
│  2. Compute hash of binary (SHA-256)                           │
│  3. Decrypt signature with developer's PUBLIC key              │
│  4. Compare: computed hash == decrypted hash?                  │
│  5. If match → authentic and unmodified ✅                     │
│  6. If mismatch → tampered or wrong author ❌                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Signing Methods

### GPG Signing

```bash
# Generate key pair
gpg --full-generate-key

# Sign a file
gpg --detach-sign --armor release-v1.0.tar.gz
# Creates release-v1.0.tar.gz.asc (detached signature)

# Verify signature
gpg --verify release-v1.0.tar.gz.asc release-v1.0.tar.gz
# Output: Good signature from "Developer <dev@example.com>"

# Sign Git commits
git config --global commit.gpgsign true
git commit -S -m "Signed commit"

# Verify Git commit signature
git log --show-signature
```

### Sigstore (Keyless Signing)

```bash
# cosign — Sign container images (Sigstore)
# No key management needed — uses OIDC identity

# Sign with identity (keyless)
cosign sign myregistry.com/myapp:v1.0
# Authenticates via GitHub/Google OIDC
# Signs with ephemeral key
# Records in Rekor transparency log

# Verify
cosign verify myregistry.com/myapp:v1.0

# Sign with key pair (traditional)
cosign generate-key-pair
cosign sign --key cosign.key myregistry.com/myapp:v1.0
cosign verify --key cosign.pub myregistry.com/myapp:v1.0
```

### Platform-Specific Signing

| Platform | Tool | Certificate Type |
|----------|------|-----------------|
| Windows | SignTool | Authenticode (EV certificate) |
| macOS | codesign | Apple Developer ID |
| Android | apksigner | Android keystore |
| iOS | codesign | Apple distribution cert |
| Docker | cosign/Notary | Sigstore or TUF |
| npm | npm publish --provenance | Sigstore OIDC |

---

## 3. Best Practices

```
CODE SIGNING CHECKLIST:
□ Sign ALL release artifacts (binaries, packages, images)
□ Store private keys in HSM or secure key management
□ Use timestamp authority (signature valid after cert expiry)
□ Implement key rotation procedures
□ Record signatures in transparency logs
□ Verify signatures in deployment pipeline
□ Revocation plan if key is compromised
□ Separate signing keys per product/environment
□ Require EV certificates for OS-level trust (Windows, macOS)
□ Sign Git commits and tags
```

---

## Summary Table

| Method | Use Case | Key Management | Transparency |
|--------|----------|----------------|-------------|
| GPG | Git commits, file signing | User-managed keys | Web of trust |
| Sigstore/cosign | Container images, packages | Keyless (OIDC) | Rekor log |
| Authenticode | Windows binaries | CA-issued certificate | CT logs |
| Apple codesign | macOS/iOS apps | Apple Developer ID | Apple notarization |

---

## Revision Questions

1. Explain the code signing process including hashing, encryption, and verification.
2. Compare GPG signing vs Sigstore — when would you use each?
3. Sign a container image with cosign and verify it. Explain the keyless signing flow.
4. Why should signing keys be stored in HSMs? What happens if a signing key is compromised?
5. How do transparency logs (Rekor) add security to the signing process?

---

*Previous: [04-deserialization.md](04-deserialization.md) | Next: [06-integrity-verification.md](06-integrity-verification.md)*

---

*[Back to README](../README.md)*
