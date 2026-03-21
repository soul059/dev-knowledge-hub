# Protected Users Group

## Unit 9 - Topic 2 | AD Hardening

---

## Overview

The **Protected Users** security group provides enhanced credential protection for its members by enforcing strict authentication policies. Members cannot use NTLM, CredSSP, or WDigest authentication, and their Kerberos tickets have shorter lifetimes — significantly reducing the attack surface for credential theft.

---

## 1. Protected Users Restrictions

```
PROTECTED USERS GROUP PROTECTIONS:

CREDENTIAL CACHING:
├── ❌ No NTLM hash cached in LSASS
├── ❌ No plaintext credentials cached
├── ❌ No WDigest credentials
├── ❌ No CredSSP delegation
├── ❌ No DES or RC4 keys cached
└── ✅ Only AES keys used

KERBEROS:
├── ❌ Cannot use DES or RC4 encryption
├── ✅ Must use AES (128 or 256)
├── ❌ TGT lifetime: 4 hours (not 10)
├── ❌ TGT cannot be renewed (normally 7 days)
└── ❌ Cannot be delegated (no impersonation)

AUTHENTICATION:
├── ❌ NTLM authentication blocked
├── ❌ No digest authentication
├── ❌ No CredSSP
├── ❌ No WDigest
└── ✅ Kerberos with AES only

IMPACT ON ATTACKS:
├── Pass-the-Hash: ❌ Blocked (no NTLM hash cached)
├── Pass-the-Ticket: ⚠️ Limited (4hr ticket, no renewal)
├── Kerberoasting: ⚠️ Harder (AES only, no RC4)
├── Token theft: ⚠️ Limited (shorter token lifetime)
├── WDigest plaintext: ❌ Blocked
├── Credential delegation: ❌ Blocked
└── NTLM relay: ❌ Blocked
```

---

## 2. Adding Accounts to Protected Users

```powershell
# === ADD USERS TO PROTECTED USERS ===

# Single user:
Add-ADGroupMember -Identity "Protected Users" -Members "t0-admin"

# Multiple users:
$admins = @("t0-admin1", "t0-admin2", "svc_critical")
$admins | ForEach-Object {
  Add-ADGroupMember -Identity "Protected Users" -Members $_
}

# Verify membership:
Get-ADGroupMember -Identity "Protected Users" | select name

# === WHO TO ADD ===
# ✅ Domain Admins
# ✅ Enterprise Admins
# ✅ Schema Admins
# ✅ Tier 0 admin accounts
# ✅ High-value service accounts (if compatible)

# === WHO NOT TO ADD ===
# ❌ Service accounts that need NTLM
# ❌ Accounts used for scheduled tasks with delegation
# ❌ Accounts that need to authenticate to non-Kerberos systems
# ❌ Computer accounts (different protection mechanism)

# === REQUIREMENTS ===
# Domain functional level: Windows Server 2012 R2+
# DC OS: Windows Server 2012 R2+
# All DCs in domain must be 2012 R2+
```

---

## 3. Testing Before Deployment

```powershell
# === PRE-DEPLOYMENT CHECKLIST ===

# 1. Identify all authentication methods used by the account:
# Does it authenticate to:
# - NTLM-only systems? → Will break!
# - Non-Windows systems? → May need NTLM
# - Systems without Kerberos? → Will fail

# 2. Test with monitoring account first:
# Add a test admin account to Protected Users
# Monitor for authentication failures
# Check Event IDs:
# - 4771: Kerberos pre-auth failed
# - 4776: NTLM authentication (should not happen)

# 3. Check for NTLM dependencies:
# Monitor NTLM usage BEFORE adding to Protected Users:
# Enable NTLM auditing:
# GPO → Security Settings → Local Policies → Security Options
# "Network security: Restrict NTLM: Audit NTLM authentication"
# Check logs: Event ID 8001, 8002, 8003, 8004

# 4. Verify Kerberos works for all required services:
klist  # Check current tickets
klist sessions  # All sessions

# === ROLLBACK ===
# If issues occur:
Remove-ADGroupMember -Identity "Protected Users" -Members "user"
# Protections removed immediately at next logon
```

---

## 4. Protected Users vs Other Controls

| Feature | Protected Users | Credential Guard | LAPS |
|---------|:--------------:|:----------------:|:----:|
| **Blocks PtH** | ✅ | ✅ | ✅ (unique passwords) |
| **Blocks NTLM** | ✅ | ❌ | ❌ |
| **Blocks WDigest** | ✅ | ✅ | ❌ |
| **Short TGT** | ✅ (4hr) | ❌ | ❌ |
| **No delegation** | ✅ | ❌ | ❌ |
| **Scope** | Per-user | Per-machine | Per-machine |
| **Requirement** | AD 2012 R2+ | Win 10+ / 2016+ | AD Schema |
| **Breaks NTLM** | ✅ | ❌ | ❌ |
| **Effort** | 🟢 Low | 🟡 Medium | 🟡 Medium |

```
DEFENSE IN DEPTH:
├── Protected Users + Credential Guard = Best protection
├── Protected Users blocks: NTLM, delegation, weak crypto
├── Credential Guard blocks: Hash extraction from LSASS
├── Together: No cached hashes + can't extract from memory
└── Add LAPS: No shared local admin passwords
```

---

## 5. Limitations and Gotchas

```
PROTECTED USERS LIMITATIONS:

1. NTLM IS COMPLETELY BLOCKED:
   ├── Some legacy applications require NTLM
   ├── Some file shares, printers use NTLM
   ├── Some VPNs fall back to NTLM
   └── Test thoroughly before deploying!

2. NO CREDENTIAL DELEGATION:
   ├── "Double hop" is completely blocked
   ├── Can't use CredSSP at all
   ├── Must use Kerberos constrained delegation
   └── Or pass credentials explicitly

3. 4-HOUR TGT:
   ├── Users must re-authenticate every 4 hours
   ├── May be annoying for interactive sessions
   ├── RDP sessions may disconnect
   └── Acceptable for admin accounts (not daily use)

4. NOT RETROACTIVE:
   ├── Protections apply at next logon
   ├── Currently cached credentials aren't cleared
   ├── Must log off and back on for effect
   └── Or reboot the machine

5. SERVICE ACCOUNTS:
   ├── Many service accounts need NTLM
   ├── Adding them to Protected Users may break services
   ├── Use gMSA instead for service accounts
   └── Test in lab environment first!
```

---

## Summary Table

| Protection | What It Does |
|-----------|-------------|
| **No NTLM** | Blocks pass-the-hash and NTLM relay |
| **No WDigest** | No plaintext passwords in memory |
| **AES only** | Harder to crack Kerberos tickets |
| **4-hour TGT** | Limits ticket theft window |
| **No delegation** | Prevents token impersonation |
| **No caching** | No offline credential exposure |

---

## Quick Revision Questions

1. **What authentication protocol is blocked for Protected Users members?**
2. **How long are TGTs for Protected Users members?**
3. **Why can't you add all service accounts to Protected Users?**
4. **What domain functional level is required?**
5. **How does Protected Users complement Credential Guard?**

---

[← Previous: Tiered Administration Model](01-tiered-administration-model.md) | [Next: Credential Guard →](03-credential-guard.md)

---

*Unit 9 - Topic 2 of 6 | [Back to README](../README.md)*
