# Chapter 6: SELinux & AppArmor

[← Previous: Security Hardening](05-security-hardening.md) | [Back to README](../README.md)

---

## Overview

SELinux (Security-Enhanced Linux) and AppArmor are **Mandatory Access Control (MAC)** systems that add an additional security layer beyond traditional Linux permissions. They confine processes to the minimum privileges required, limiting damage from exploits. SELinux is used primarily on RHEL/CentOS/Fedora, while AppArmor is used on Ubuntu/Debian/SUSE.

---

## 6.1 MAC vs DAC

```
┌──────────────────────────────────────────────────────────────┐
│         DAC vs MAC COMPARISON                                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  DAC (Discretionary Access Control)                           │
│  ┌────────────────────────────────────┐                      │
│  │ Traditional Linux permissions      │                      │
│  │ Owner controls access              │                      │
│  │ rwxr-xr-x  user:group             │                      │
│  │ Root = unrestricted                │                      │
│  └────────────────────────────────────┘                      │
│                                                               │
│  MAC (Mandatory Access Control)                               │
│  ┌────────────────────────────────────┐                      │
│  │ System-wide policy, kernel-level   │                      │
│  │ Even root is restricted            │                      │
│  │ Process labeled → confined         │                      │
│  │ "Need to know" principle           │                      │
│  └────────────────────────────────────┘                      │
│                                                               │
│  Request Flow:                                                │
│  ┌──────┐   ┌─────┐   ┌─────┐   ┌──────────┐               │
│  │ User │──>│ DAC │──>│ MAC │──>│ Resource  │               │
│  │ app  │   │check│   │check│   │ (file,    │               │
│  │      │   │     │   │     │   │  socket)  │               │
│  └──────┘   └─────┘   └─────┘   └──────────┘               │
│              PASS?      PASS?      ALLOWED                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 SELinux (RHEL/CentOS/Fedora)

### SELinux Modes

```
┌──────────────────────────────────────────────────────────────┐
│         SELINUX MODES                                         │
├──────────────┬───────────────────────────────────────────────┤
│ Mode         │ Behavior                                      │
├──────────────┼───────────────────────────────────────────────┤
│ Enforcing    │ Policy enforced — violations DENIED & logged  │
│ Permissive   │ Policy NOT enforced — violations LOGGED only  │
│ Disabled     │ SELinux completely off (requires reboot)      │
└──────────────┴───────────────────────────────────────────────┘
```

```bash
# Check current mode
getenforce                     # Enforcing | Permissive | Disabled
sestatus                       # Full status info

# Change mode temporarily (until reboot)
sudo setenforce 0              # Permissive
sudo setenforce 1              # Enforcing

# Change permanently — /etc/selinux/config
SELINUX=enforcing              # enforcing | permissive | disabled
SELINUXTYPE=targeted           # targeted | mls
```

### SELinux Context Labels

```bash
# Every file, process, port has a security context
# Format:  user:role:type:level
# Example: system_u:object_r:httpd_sys_content_t:s0

# View file contexts
ls -Z /var/www/html/
# -rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html

# View process contexts
ps auxZ | grep nginx
# system_u:system_r:httpd_t:s0   root  ... nginx

# View port contexts
sudo semanage port -l | grep http
# http_port_t    tcp  80, 443, 488, 8008, 8009, 8443
```

### Managing File Contexts

```bash
# Change file context temporarily
sudo chcon -t httpd_sys_content_t /path/to/file

# Change recursively
sudo chcon -R -t httpd_sys_content_t /var/www/myapp/

# Restore default context (from policy)
sudo restorecon -v /path/to/file
sudo restorecon -Rv /var/www/html/

# Define permanent file context rule
sudo semanage fcontext -a -t httpd_sys_content_t "/opt/myapp(/.*)?"
sudo restorecon -Rv /opt/myapp/
```

### SELinux Booleans

```bash
# List all booleans
sudo getsebool -a
sudo getsebool -a | grep httpd

# Common examples
sudo setsebool -P httpd_can_network_connect on     # Allow HTTPD → network
sudo setsebool -P httpd_can_network_connect_db on   # Allow HTTPD → database
sudo setsebool -P httpd_enable_homedirs on           # Allow serving home dirs

# -P = persistent across reboots
```

### Managing Port Contexts

```bash
# Allow Nginx on non-standard port
sudo semanage port -a -t http_port_t -p tcp 8080

# List port assignments
sudo semanage port -l | grep http_port_t

# Remove custom port
sudo semanage port -d -t http_port_t -p tcp 8080
```

### Troubleshooting SELinux

```bash
# View SELinux denials
sudo ausearch -m AVC -ts recent
sudo ausearch -m AVC,USER_AVC -ts today

# Human-readable analysis
sudo ausearch -m AVC -ts recent | audit2why

# Auto-generate policy module from denial
sudo ausearch -m AVC -ts recent | audit2allow -M mypolicy
sudo semodule -i mypolicy.pp

# Install troubleshooting tools
sudo dnf install setroubleshoot-server
sudo sealert -a /var/log/audit/audit.log

# Check for issues in journal
sudo journalctl -t setroubleshoot
```

```
┌──────────────────────────────────────────────────────────────┐
│         SELINUX TROUBLESHOOTING WORKFLOW                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. App denied? ──> Check audit log                          │
│     sudo ausearch -m AVC -ts recent                          │
│                                                               │
│  2. Is it a label issue?                                      │
│     YES ──> restorecon -Rv /path                             │
│                                                               │
│  3. Is it a boolean?                                          │
│     YES ──> setsebool -P boolean_name on                     │
│                                                               │
│  4. Is it a port issue?                                       │
│     YES ──> semanage port -a -t type -p tcp PORT             │
│                                                               │
│  5. None of the above?                                        │
│     ──> audit2allow to create custom policy                   │
│                                                               │
│  NEVER: setenforce 0 and walk away!                          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 AppArmor (Ubuntu/Debian/SUSE)

### AppArmor Modes

```
┌──────────────────────────────────────────────────────────────┐
│         APPARMOR MODES                                        │
├──────────────┬───────────────────────────────────────────────┤
│ Mode         │ Behavior                                      │
├──────────────┼───────────────────────────────────────────────┤
│ Enforce      │ Policy enforced — violations denied & logged  │
│ Complain     │ Policy NOT enforced — violations logged only  │
│ Disabled     │ Profile not loaded                            │
└──────────────┴───────────────────────────────────────────────┘
```

```bash
# Check status
sudo aa-status
sudo apparmor_status

# Profiles location
ls /etc/apparmor.d/

# Set profile to complain mode
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx

# Set profile to enforce mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# Disable a profile
sudo ln -s /etc/apparmor.d/usr.sbin.nginx /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.nginx

# Reload a profile
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

### AppArmor Profile Structure

```bash
# /etc/apparmor.d/opt.myapp.myapp
# Profile for /opt/myapp/myapp

#include <tunables/global>

/opt/myapp/myapp {
    #include <abstractions/base>
    #include <abstractions/nameservice>

    # Allow reading config
    /etc/myapp/** r,

    # Allow read/write to data directory
    /var/lib/myapp/** rw,

    # Allow writing logs
    /var/log/myapp/** w,

    # Allow network access
    network tcp,

    # Allow reading shared libraries
    /usr/lib/** mr,

    # Deny everything else (implicit)
}
```

```
┌──────────────────────────────────────────────────────────────┐
│         APPARMOR PERMISSION FLAGS                             │
├──────────┬───────────────────────────────────────────────────┤
│ Flag     │ Permission                                        │
├──────────┼───────────────────────────────────────────────────┤
│ r        │ Read                                              │
│ w        │ Write                                             │
│ a        │ Append                                            │
│ x        │ Execute                                           │
│ ix       │ Inherit execute (child inherits parent profile)   │
│ px       │ Discrete profile execute (child has own profile)  │
│ ux       │ Unconfined execute (child runs unconfined)        │
│ m        │ Memory map executable                             │
│ k        │ Lock (file locking)                               │
│ l        │ Link                                              │
└──────────┴───────────────────────────────────────────────────┘
```

### Creating a New Profile

```bash
# Install utilities
sudo apt install apparmor-utils

# Auto-generate profile (interactive)
sudo aa-genprof /opt/myapp/myapp
# 1. Run the app in another terminal
# 2. Come back and press 'S' to scan logs
# 3. Review and allow/deny each access
# 4. Press 'F' to finish

# Generate from logs (after running in complain mode)
sudo aa-logprof
```

### Troubleshooting AppArmor

```bash
# Check for denials in syslog
sudo grep "apparmor" /var/log/syslog | tail -20

# Or journalctl
sudo journalctl -k | grep "apparmor.*DENIED"

# Put profile in complain mode to diagnose
sudo aa-complain /etc/apparmor.d/profile.name

# Update profile from logs
sudo aa-logprof

# Switch back to enforce
sudo aa-enforce /etc/apparmor.d/profile.name
```

---

## 6.4 SELinux vs AppArmor Comparison

```
┌──────────────────────────────────────────────────────────────┐
│         SELINUX vs APPARMOR COMPARISON                        │
├──────────────┬────────────────────┬──────────────────────────┤
│ Feature      │ SELinux            │ AppArmor                 │
├──────────────┼────────────────────┼──────────────────────────┤
│ Distros      │ RHEL, CentOS,      │ Ubuntu, Debian,          │
│              │ Fedora, Amazon     │ SUSE, Arch               │
├──────────────┼────────────────────┼──────────────────────────┤
│ Approach     │ Label-based        │ Path-based               │
│              │ (security context) │ (filesystem paths)       │
├──────────────┼────────────────────┼──────────────────────────┤
│ Complexity   │ Steep learning     │ Easier to learn          │
│              │ curve              │                          │
├──────────────┼────────────────────┼──────────────────────────┤
│ Granularity  │ Very fine-grained  │ Moderate granularity     │
│              │ (files, ports,     │ (file paths primarily)   │
│              │ processes, network)│                          │
├──────────────┼────────────────────┼──────────────────────────┤
│ Default      │ All processes      │ Only profiled apps       │
│ scope        │ confined           │ are confined             │
├──────────────┼────────────────────┼──────────────────────────┤
│ Modes        │ Enforcing,         │ Enforce, Complain,       │
│              │ Permissive,        │ Disabled (per-profile)   │
│              │ Disabled (global)  │                          │
├──────────────┼────────────────────┼──────────────────────────┤
│ Docker       │ Native support     │ Native support           │
│ integration  │ (--security-opt)   │ (--security-opt)         │
├──────────────┼────────────────────┼──────────────────────────┤
│ Troubleshoot │ ausearch,          │ aa-logprof,              │
│              │ audit2why,         │ aa-complain,             │
│              │ sealert            │ syslog/journal           │
└──────────────┴────────────────────┴──────────────────────────┘
```

---

## 6.5 Docker & Container Integration

### SELinux with Docker

```bash
# Docker automatically labels containers
# Run with read-only volume and SELinux relabel
docker run -v /data:/data:z nginx         # :z = shared label
docker run -v /data:/data:Z nginx         # :Z = private (per-container)

# Check container SELinux context
docker inspect --format '{{.ProcessLabel}}' container_name

# Custom SELinux policy for container
# Write .te file → compile → install
```

### AppArmor with Docker

```bash
# Docker uses default apparmor profile: docker-default
docker run --security-opt apparmor=docker-default nginx

# Run with custom profile
docker run --security-opt apparmor=my-custom-profile nginx

# Run without AppArmor (not recommended)
docker run --security-opt apparmor=unconfined nginx

# Load custom profile
sudo apparmor_parser -r -W /etc/apparmor.d/my-container-profile
```

### Custom AppArmor Profile for Container

```bash
# /etc/apparmor.d/docker-nginx-custom
#include <tunables/global>

profile docker-nginx-custom flags=(attach_disconnected,mediate_deleted) {
    #include <abstractions/base>

    network inet tcp,
    network inet udp,

    deny mount,
    deny ptrace,

    /usr/sbin/nginx ix,
    /etc/nginx/** r,
    /var/log/nginx/** w,
    /var/cache/nginx/** rw,
    /run/nginx.pid w,

    # Deny access to sensitive host paths
    deny /proc/sys/** w,
    deny /sys/** w,
}
```

---

## 6.6 Real-World Scenario: Diagnosing SELinux Denial for Nginx

```bash
#!/usr/bin/env bash
# selinux-fix.sh — Diagnose and fix SELinux issues for web apps
# Usage: sudo ./selinux-fix.sh

set -euo pipefail

APP_DIR="/opt/webapp"
APP_PORT=8080
LOG="/var/log/selinux-fix.log"

log() { echo "$(date '+%F %T') $*" | tee -a "$LOG"; }

log "=== SELinux Web App Fix Script ==="

# 1. Check SELinux status
MODE=$(getenforce)
log "Current SELinux mode: $MODE"

if [ "$MODE" = "Disabled" ]; then
    log "SELinux is disabled. Enable it in /etc/selinux/config and reboot."
    exit 1
fi

# 2. Check for recent denials
log "Checking recent AVC denials..."
DENIALS=$(ausearch -m AVC -ts recent 2>/dev/null | grep -c "denied" || true)
log "Found $DENIALS recent denials"

if [ "$DENIALS" -gt 0 ]; then
    log "Recent denials:"
    ausearch -m AVC -ts recent 2>/dev/null | audit2why | tee -a "$LOG"
fi

# 3. Fix file contexts for web app
log "Setting file contexts for $APP_DIR..."
semanage fcontext -a -t httpd_sys_content_t "${APP_DIR}(/.*)?" 2>/dev/null || \
    log "Context rule already exists"
restorecon -Rv "$APP_DIR" | tee -a "$LOG"
log "File contexts set"

# 4. Allow non-standard port
log "Allowing HTTP on port $APP_PORT..."
EXISTING=$(semanage port -l | grep "http_port_t" | grep -c "$APP_PORT" || true)
if [ "$EXISTING" -eq 0 ]; then
    semanage port -a -t http_port_t -p tcp "$APP_PORT"
    log "Port $APP_PORT added to http_port_t"
else
    log "Port $APP_PORT already allowed"
fi

# 5. Enable common booleans
log "Enabling required booleans..."
BOOLEANS=(
    "httpd_can_network_connect"
    "httpd_can_network_connect_db"
)
for bool in "${BOOLEANS[@]}"; do
    CURRENT=$(getsebool "$bool" | awk '{print $3}')
    if [ "$CURRENT" = "off" ]; then
        setsebool -P "$bool" on
        log "Enabled: $bool"
    else
        log "Already enabled: $bool"
    fi
done

# 6. Verify
log "Verification:"
log "  File context: $(ls -Zd "$APP_DIR")"
log "  Port: $(semanage port -l | grep http_port_t | head -1)"
log "  Mode: $(getenforce)"

# 7. Test (if nginx/httpd is running)
if systemctl is-active httpd &>/dev/null || systemctl is-active nginx &>/dev/null; then
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:${APP_PORT}/" || echo "000")
    log "  HTTP test (port $APP_PORT): $HTTP_CODE"
fi

log "=== Fix Complete ==="
log "If issues persist, check: ausearch -m AVC -ts recent | audit2why"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Nginx can't read files | Wrong SELinux label | `restorecon -Rv /var/www/html` or `chcon -t httpd_sys_content_t` |
| App can't bind to port 8080 | Port not in SELinux policy | `semanage port -a -t http_port_t -p tcp 8080` |
| App can't connect to DB | Boolean not set | `setsebool -P httpd_can_network_connect_db on` |
| Docker volume permission denied | Missing `:z` or `:Z` | Use `docker run -v /path:/path:z image` |
| AppArmor blocking app | Profile too restrictive | `aa-complain profile`, test, `aa-logprof`, `aa-enforce` |
| Not sure what's blocked | Need audit details | SELinux: `ausearch -m AVC`; AppArmor: `journalctl -k | grep DENIED` |

---

## Summary Table

| Feature | SELinux | AppArmor |
|---------|---------|----------|
| Used on | RHEL/CentOS/Fedora | Ubuntu/Debian/SUSE |
| Check status | `getenforce`, `sestatus` | `aa-status` |
| View labels/profiles | `ls -Z`, `ps auxZ` | `aa-status` |
| Fix file labels | `restorecon -Rv` | N/A (path-based) |
| Allow port | `semanage port -a` | Edit profile |
| Enable feature | `setsebool -P` | Edit profile |
| Debug mode | `setenforce 0` (permissive) | `aa-complain profile` |
| View denials | `ausearch -m AVC` | `grep apparmor /var/log/syslog` |
| Generate policy | `audit2allow` | `aa-genprof` |
| Docker flag | `-v path:path:z` | `--security-opt apparmor=` |

---

## Quick Revision Questions

1. **What is the difference between DAC and MAC?**
2. **What are the three SELinux modes and when would you use each?**
3. **How do you fix an SELinux denial for Nginx serving files from a custom directory?**
4. **What is the difference between SELinux `chcon` and `restorecon`?**
5. **How do you create a custom AppArmor profile for a new application?**
6. **How do SELinux and AppArmor integrate with Docker containers?**

---

[← Previous: Security Hardening](05-security-hardening.md) | [Back to README](../README.md)
