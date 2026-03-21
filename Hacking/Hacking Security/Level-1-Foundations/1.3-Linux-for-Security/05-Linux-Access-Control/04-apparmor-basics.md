# AppArmor Basics

## Unit 5 - Topic 4 | Linux Access Control

---

## Overview

**AppArmor** is a MAC implementation that uses **path-based profiles** to confine applications. Unlike SELinux's label-based approach, AppArmor defines what each program can access using filesystem paths. AppArmor is the default MAC system on **Ubuntu**, **Debian**, and **SUSE** — it's generally easier to learn and manage than SELinux.

---

## 1. AppArmor vs SELinux

| Feature | AppArmor | SELinux |
|---------|:--------:|:------:|
| **Approach** | Path-based | Label-based |
| **Complexity** | ★★★ (simpler) | ★★★★★ (complex) |
| **Default on** | Ubuntu, Debian, SUSE | RHEL, CentOS, Fedora |
| **Profile location** | /etc/apparmor.d/ | /etc/selinux/ |
| **Learning mode** | complain mode | permissive mode |
| **Scope** | Per-application profiles | System-wide policy |

---

## 2. AppArmor Status and Modes

```bash
# Check status
aa-status
# or
apparmor_status

# Output shows:
# - Loaded profiles
# - Profiles in enforce mode
# - Profiles in complain mode
# - Unconfined processes

# MODES:
# enforce  — Blocks and logs violations ✅
# complain — Logs violations but doesn't block (learning mode)
# unconfined — No restrictions (profile not loaded)

# Switch modes
aa-enforce /etc/apparmor.d/usr.sbin.nginx     # Set to enforce
aa-complain /etc/apparmor.d/usr.sbin.nginx    # Set to complain
aa-disable /etc/apparmor.d/usr.sbin.nginx     # Disable profile

# Reload all profiles
systemctl reload apparmor
apparmor_parser -r /etc/apparmor.d/           # Reload specific profiles
```

---

## 3. AppArmor Profile Structure

```bash
# Profile location: /etc/apparmor.d/usr.sbin.nginx

#include <tunables/global>

/usr/sbin/nginx {
    #include <abstractions/base>
    #include <abstractions/nameservice>

    # === CAPABILITIES ===
    capability net_bind_service,
    capability setuid,
    capability setgid,

    # === NETWORK ===
    network inet stream,              # Allow TCP IPv4
    network inet6 stream,             # Allow TCP IPv6

    # === FILE ACCESS ===
    /etc/nginx/** r,                  # Read nginx config
    /var/log/nginx/** w,              # Write to log files
    /var/www/html/** r,               # Read web content
    /run/nginx.pid rw,                # PID file
    /usr/sbin/nginx mr,               # Execute self

    # === DENY RULES ===
    deny /etc/shadow r,               # Explicitly deny shadow
    deny /etc/passwd w,               # Deny writing to passwd
    deny /home/** rwx,                # No access to home dirs

    # === PERMISSIONS KEY ===
    # r = read, w = write, x = execute
    # m = mmap, k = lock, l = link
    # ** = recursive (all subdirs)
}
```

---

## 4. Creating Custom Profiles

```bash
# STEP 1: Generate a profile skeleton
aa-genprof /usr/local/bin/myapp
# This runs the app and monitors what it accesses

# STEP 2: Or manually create profile
cat > /etc/apparmor.d/usr.local.bin.myapp << 'EOF'
#include <tunables/global>

/usr/local/bin/myapp {
    #include <abstractions/base>

    /usr/local/bin/myapp mr,
    /etc/myapp/** r,
    /var/log/myapp/** w,
    /var/lib/myapp/** rw,
    /tmp/myapp-* rw,

    deny /etc/shadow r,
    deny /etc/passwd w,
    deny /home/** rwx,
}
EOF

# STEP 3: Load profile
apparmor_parser -r /etc/apparmor.d/usr.local.bin.myapp

# STEP 4: Start in complain mode to test
aa-complain /etc/apparmor.d/usr.local.bin.myapp

# STEP 5: Monitor violations
journalctl | grep apparmor | grep ALLOWED

# STEP 6: Switch to enforce when ready
aa-enforce /etc/apparmor.d/usr.local.bin.myapp
```

---

## 5. Troubleshooting AppArmor

```bash
# View AppArmor denials
journalctl | grep "apparmor=\"DENIED\""
dmesg | grep apparmor

# Example denial:
# apparmor="DENIED" operation="open" profile="/usr/sbin/nginx"
# name="/home/user/secret.txt" requested_mask="r"

# Translation: nginx tried to read /home/user/secret.txt
# AppArmor profile doesn't allow this → DENIED

# Fix: Add rule to profile (if legitimate)
# /home/user/secret.txt r,
# Then reload: apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

---

## 6. Important Abstractions

| Abstraction | Purpose |
|-------------|---------|
| `base` | Basic system access (libc, locales) |
| `nameservice` | DNS, NSS, LDAP resolution |
| `authentication` | PAM, NSS, auth modules |
| `apache2-common` | Common Apache access patterns |
| `ssl_certs` | SSL certificate access |
| `user-tmp` | User temporary file access |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **AppArmor** | Path-based MAC for Linux |
| **Profiles** | Define allowed file/network/capability access |
| **Enforce** | Blocks violations |
| **Complain** | Logs but doesn't block (learning mode) |
| **aa-genprof** | Auto-generate profiles |
| **Abstractions** | Reusable access pattern includes |

---

## Quick Revision Questions

1. **How does AppArmor differ from SELinux?**
2. **What is the difference between enforce and complain mode?**
3. **Where are AppArmor profiles stored?**
4. **How do you create a profile for a new application?**
5. **How do you troubleshoot an AppArmor denial?**

---

[← Previous: SELinux Basics](03-selinux-basics.md) | [Next: Linux Capabilities →](05-linux-capabilities.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*
