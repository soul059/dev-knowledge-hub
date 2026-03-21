# Unit 8: A08 - Software and Data Integrity Failures — Topic 2: Dependency Integrity

## Overview

Dependency integrity ensures that the third-party code your application uses **has not been tampered with** between the author publishing it and your application consuming it. Without integrity verification, attackers can inject malicious code through compromised packages, man-in-the-middle attacks on registries, or by hijacking maintainer accounts.

---

## 1. Integrity Verification Mechanisms

```
┌─────────────────────────────────────────────────────────────────┐
│           DEPENDENCY INTEGRITY CHAIN                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Author → Registry → Your Build System → Production            │
│    ↕          ↕              ↕                                  │
│  Signing   Checksums     Lock Files                            │
│  GPG/SSH   SHA-256/512   Integrity Hash                        │
│                                                                 │
│  VERIFICATION AT EACH STEP:                                    │
│  1. Author signs the package (GPG, Sigstore)                   │
│  2. Registry stores package with checksum                      │
│  3. Lock file records expected checksum                        │
│  4. Build system verifies checksum matches                     │
│  5. If mismatch → build fails (integrity violation)            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Lock File Integrity

```json
// package-lock.json — Integrity hashes
{
  "packages": {
    "node_modules/express": {
      "version": "4.18.2",
      "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "integrity": "sha512-5/PsL6iGPdfQ/lKM1UuielYgv3BUoJfz1aUwU9vHZ+J7gyvwdQXFEBIEIaxeGf0GIcreATNyBExtalisDbuMg==",
      "license": "MIT"
    }
  }
}

// When you run 'npm ci':
// 1. Downloads express-4.18.2.tgz
// 2. Computes SHA-512 hash
// 3. Compares with "integrity" field
// 4. If mismatch → FAILS (possible tampering)
```

```txt
# Python — requirements.txt with hashes
Flask==2.3.3 \
    --hash=sha256:f69fcd559dc907ed196ab9df0e48471709175e696d6e698dd4dbe940f96ce66b

# pip install --require-hashes -r requirements.txt
# Verifies hash before installing
```

---

## 3. Subresource Integrity (SRI)

```html
<!-- Loading scripts from CDN without SRI (DANGEROUS) -->
<script src="https://cdn.example.com/library.js"></script>
<!-- If CDN is compromised, malicious JS executes -->

<!-- With SRI (SECURE) -->
<script src="https://cdn.example.com/library.js"
    integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
    crossorigin="anonymous"></script>
<!-- Browser verifies hash before executing -->
<!-- If hash doesn't match → script is NOT executed -->

<!-- Generate SRI hash -->
<!-- openssl dgst -sha384 -binary library.js | openssl base64 -A -->
<!-- Or use: https://www.srihash.org -->
```

---

## 4. Package Signing and Verification

```bash
# npm — Package provenance (Sigstore)
npm publish --provenance
# Creates cryptographic attestation linking package to source

# Verify provenance
npm audit signatures
# Checks that packages have valid registry signatures

# Go — Module checksums
# go.sum contains expected hashes for all modules
# go mod verify — checks downloaded modules against go.sum

# Python — PEP 458 (TUF-based signing)
# pip install --require-hashes ensures hash verification

# Container images — cosign (Sigstore)
# Sign image
cosign sign --key cosign.key myregistry.com/myapp:v1.0

# Verify image
cosign verify --key cosign.pub myregistry.com/myapp:v1.0
```

---

## 5. Best Practices

```
DEPENDENCY INTEGRITY CHECKLIST:

□ Use lock files and commit them to version control
□ Use 'npm ci' / 'pip install --require-hashes' in CI/CD
□ Enable SRI for all CDN-loaded scripts/styles
□ Verify package signatures where available
□ Use private registry mirrors for critical dependencies
□ Pin dependencies to exact versions
□ Monitor for package hijacking alerts
□ Use cosign for container image signing
□ Audit lock file changes in code review
□ Enable npm audit signatures
```

---

## Summary Table

| Mechanism | Protects Against | Ecosystem |
|-----------|-----------------|-----------|
| Lock file integrity | Tampered downloads | npm, pip, Go, Ruby |
| SRI | Compromised CDN | Browser/HTML |
| Package signing | Author impersonation | npm (Sigstore), Go |
| Container signing | Image tampering | cosign, Notary |
| Hash verification | MITM on downloads | All |

---

## Revision Questions

1. How do lock files provide integrity verification for dependencies?
2. Implement SRI for a web page loading jQuery and Bootstrap from a CDN.
3. What is Sigstore and how does it enable package provenance?
4. Why should you use `npm ci` instead of `npm install` in CI/CD pipelines?
5. How would you detect if a package was tampered with between the registry and your build system?

---

*Previous: [01-cicd-security.md](01-cicd-security.md) | Next: [03-unsigned-updates.md](03-unsigned-updates.md)*

---

*[Back to README](../README.md)*
