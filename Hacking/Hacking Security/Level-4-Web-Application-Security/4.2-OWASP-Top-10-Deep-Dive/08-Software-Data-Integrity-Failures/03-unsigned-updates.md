# Unit 8: A08 - Software and Data Integrity Failures — Topic 3: Unsigned Updates

## Overview

Auto-update mechanisms that don't verify the **authenticity and integrity** of updates can be exploited to distribute malware. If an application downloads and installs updates without cryptographic verification, an attacker who compromises the update server, performs a MITM attack, or DNS hijacks the update domain can push malicious code to all users.

---

## 1. Update Integrity Risks

```
ATTACK SCENARIOS:

1. COMPROMISED UPDATE SERVER
   Attacker gains access to update distribution server
   → Replaces legitimate update with malicious version
   → All users who update receive malware
   Example: ASUS Live Update (2019) — 500K+ users

2. MAN-IN-THE-MIDDLE
   Update downloaded over HTTP (not HTTPS)
   → Attacker intercepts and replaces package
   Example: evilgrade framework exploits insecure updaters

3. DNS HIJACKING
   Redirect update domain to attacker's server
   → Users download malicious update from fake server

4. COMPROMISED SIGNING KEY
   Attacker steals code signing private key
   → Signs malicious updates with legitimate key
   Example: Stuxnet used stolen Realtek/JMicron certificates
```

---

## 2. Secure Update Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURE UPDATE FLOW                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Developer builds update                                    │
│  2. Update signed with private key (offline/HSM)               │
│  3. Signed update uploaded to distribution server              │
│  4. Client checks for updates (over HTTPS + cert pinning)      │
│  5. Client downloads update + signature                        │
│  6. Client verifies signature with embedded public key         │
│  7. If signature valid → install update                        │
│  8. If signature invalid → reject and alert                    │
│                                                                 │
│  CRITICAL REQUIREMENTS:                                        │
│  ├── HTTPS for all update communications                       │
│  ├── Code signing with strong algorithm (RSA-2048+ or Ed25519)│
│  ├── Public key embedded in application (not downloaded)       │
│  ├── Verify signature BEFORE installation                      │
│  ├── Version check (prevent downgrade attacks)                 │
│  ├── Multiple signing keys (threshold signing)                 │
│  └── Transparency log (public record of updates)               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Code Signing Implementation

```python
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding, rsa

# SIGNING (build server)
def sign_update(update_path, private_key_path):
    with open(private_key_path, 'rb') as f:
        private_key = serialization.load_pem_private_key(f.read(), password=None)
    
    with open(update_path, 'rb') as f:
        update_data = f.read()
    
    signature = private_key.sign(
        update_data,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
    
    with open(update_path + '.sig', 'wb') as f:
        f.write(signature)

# VERIFICATION (client application)
def verify_update(update_path, signature_path, public_key_pem):
    public_key = serialization.load_pem_public_key(public_key_pem)
    
    with open(update_path, 'rb') as f:
        update_data = f.read()
    with open(signature_path, 'rb') as f:
        signature = f.read()
    
    try:
        public_key.verify(
            signature,
            update_data,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        return True  # Signature valid
    except Exception:
        return False  # TAMPERED — do not install
```

---

## 4. The Update Framework (TUF)

```
TUF — Industry standard for secure software updates
Used by: PyPI, RubyGems, Docker Content Trust, Uptane (vehicles)

KEY CONCEPTS:
- Multiple signing roles (root, targets, snapshot, timestamp)
- Key threshold (requires N of M keys to sign)
- Expiration on metadata (prevents freeze attacks)
- Consistent snapshots (prevents mix-and-match attacks)

ROLES:
Root     → Trusts other roles, rarely changes
Targets  → Signs the actual update files
Snapshot → Signs current state of all targets
Timestamp→ Proves freshness (prevents freeze attacks)
```

---

## 5. Prevention Checklist

```
□ All updates delivered over HTTPS with certificate validation
□ Updates cryptographically signed before distribution
□ Client verifies signature before installation
□ Public key embedded in application (not fetched remotely)
□ Version checking prevents downgrade attacks
□ Signing keys stored in HSM or air-gapped systems
□ Key rotation procedure documented and tested
□ Update metadata has expiration (freshness)
□ Failed verification triggers alert, not silent failure
□ Consider TUF for critical update infrastructure
```

---

## Summary Table

| Control | Prevents | Implementation |
|---------|----------|----------------|
| HTTPS | MITM interception | TLS 1.2+, certificate pinning |
| Code signing | Tampered updates | RSA/Ed25519 signatures |
| Version check | Downgrade attacks | Compare version numbers |
| Key management | Key compromise | HSM, rotation, threshold |
| TUF | Comprehensive update attacks | Framework adoption |

---

## Revision Questions

1. What are four ways an attacker can compromise a software update mechanism?
2. Design a secure auto-update system with signing, verification, and rollback.
3. Implement code signing and verification in Python.
4. What is The Update Framework (TUF) and what attacks does it prevent?
5. How did the ASUS Live Update attack work and what controls would have prevented it?

---

*Previous: [02-dependency-integrity.md](02-dependency-integrity.md) | Next: [04-deserialization.md](04-deserialization.md)*

---

*[Back to README](../README.md)*
