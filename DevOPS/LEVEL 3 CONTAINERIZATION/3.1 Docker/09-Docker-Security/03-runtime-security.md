# Chapter 3: Runtime Security

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

Runtime security focuses on protecting containers while they're running. This includes restricting privileges, filtering system calls, applying mandatory access controls, and using read-only filesystems to limit what a compromised container can do.

---

## 3.1 Linux Capabilities

```
  Root Powers Broken Into Capabilities:
  
  Traditional: root = ALL powers
  Capabilities: root = selected powers
  
  Docker Default Capabilities (kept):
  +-----+-----------------------------------+
  | Cap | Purpose                           |
  +-----+-----------------------------------+
  | CHOWN        | Change file ownership     |
  | DAC_OVERRIDE | Bypass file permissions   |
  | FSETID       | Set file SUID/SGID        |
  | FOWNER       | Bypass owner checks       |
  | KILL         | Send signals              |
  | SETGID       | Set group ID              |
  | SETUID       | Set user ID               |
  | SETPCAP      | Modify capabilities       |
  | NET_BIND     | Bind ports < 1024         |
  | NET_RAW      | Use raw sockets           |
  | SYS_CHROOT   | Use chroot                |
  | MKNOD        | Create special files      |
  | AUDIT_WRITE  | Write to audit log        |
  | SETFCAP      | Set file capabilities     |
  +-----+-----------------------------------+
  
  Docker Drops (blocked by default):
  SYS_ADMIN, SYS_PTRACE, SYS_MODULE,
  NET_ADMIN, SYS_RAWIO, SYS_TIME, etc.
```

```bash
# Drop ALL capabilities, add only what's needed
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx

# Drop specific dangerous capabilities
docker run --cap-drop NET_RAW --cap-drop SYS_CHROOT myapp

# Check container capabilities
docker exec mycontainer cat /proc/1/status | grep Cap

# NEVER use --privileged (grants ALL capabilities + devices)
docker run --privileged myimage  # DANGEROUS!
```

```
  Capability Strategy:
  
  ✗ Default: 14 capabilities (too many)
  ✓ Better:  Drop ALL, add specific ones
  ✗ NEVER:   --privileged (ALL capabilities)
  
  --privileged gives:
  ├── ALL Linux capabilities
  ├── Access to ALL host devices
  ├── No seccomp filtering
  ├── No AppArmor profile
  └── Can mount host filesystems
  
  = Basically root on the host!
```

---

## 3.2 Seccomp Profiles

```bash
# Seccomp = Secure Computing Mode
# Filters system calls the container can make

# Docker's default seccomp profile blocks ~44 syscalls including:
# - reboot, mount, umount, swapon
# - init_module, delete_module (kernel modules)
# - settimeofday, clock_settime
# - acct (process accounting)

# Check if seccomp is active
docker inspect --format='{{.HostConfig.SecurityOpt}}' mycontainer

# Run with custom seccomp profile
docker run --security-opt seccomp=custom-profile.json myimage

# Disable seccomp (NOT recommended)
docker run --security-opt seccomp=unconfined myimage
```

```json
// Example: Restrictive seccomp profile (custom-profile.json)
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["read", "write", "open", "close",
                "stat", "fstat", "mmap", "mprotect",
                "brk", "exit_group", "execve"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

```
  Seccomp Decision:
  
  Syscall made by container
         |
         v
  +-------------------+
  | Seccomp Profile   |
  | (allowlist/deny)  |
  +-------------------+
         |
    +----+----+
    |         |
  Allowed   Blocked
    |         |
    v         v
  Execute   EPERM
  syscall   (Permission
            denied)
```

---

## 3.3 AppArmor and SELinux

```bash
# AppArmor (Ubuntu/Debian default)
# Check active profile
docker inspect --format='{{.AppArmorProfile}}' mycontainer
# Output: docker-default

# Run with custom AppArmor profile
docker run --security-opt apparmor=my-custom-profile myimage

# SELinux (RHEL/CentOS/Fedora)
# Enable SELinux labels on volumes
docker run -v /data:/data:Z myimage   # Private label
docker run -v /data:/data:z myimage   # Shared label

# Check SELinux context
docker inspect --format='{{.ProcessLabel}}' mycontainer
```

---

## 3.4 Read-Only Root Filesystem

```bash
# Make container's root filesystem read-only
docker run --read-only myimage

# Usually need some writable areas
docker run --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --tmpfs /var/run:rw,noexec,nosuid \
  -v appdata:/app/data \
  myimage
```

```
  Read-Only Filesystem:
  
  +-- Container Filesystem --+
  | / (read-only)            |
  | /bin  (read-only)        |
  | /etc  (read-only)        |
  | /app  (read-only)        |
  |                          |
  | /tmp  (tmpfs - writable) |
  | /var/run (tmpfs - write) |
  | /app/data (volume - rw)  |
  +--------------------------+
  
  Benefits:
  ✓ Attacker can't modify binaries
  ✓ Can't install backdoors
  ✓ Can't modify configs
  ✓ Immutable infrastructure
```

```yaml
# Docker Compose
services:
  api:
    image: myapi
    read_only: true
    tmpfs:
      - /tmp:size=64m
      - /var/run
    volumes:
      - appdata:/app/data
```

---

## 3.5 No New Privileges

```bash
# Prevent processes from gaining additional privileges
docker run --security-opt no-new-privileges myimage

# This prevents:
# - SUID binaries from escalating privileges
# - setuid/setgid operations
# - Gaining capabilities through execve
```

```yaml
# Docker Compose
services:
  api:
    security_opt:
      - no-new-privileges:true
```

---

## 3.6 Resource Limits as Security

```bash
# Prevent resource exhaustion attacks (DoS)

# Memory limit (OOM killed if exceeded)
docker run --memory=512m --memory-swap=512m myimage

# CPU limit
docker run --cpus=1.0 myimage

# Process limit (fork bomb protection)
docker run --pids-limit=100 myimage

# No-restart to prevent restart loops
docker run --restart=on-failure:5 myimage  # Max 5 retries
```

```
  Resource Limits Security:
  
  Without limits:        With limits:
  
  +----------+           +----------+
  | Container|           | Container|
  | Fork bomb|           | Fork bomb|
  |  ||||||| |           | |||| MAX |
  |  ||||||| |           +----------+
  |  |||||||→ HOST DOWN! 
  +----------+           Host protected ✓
  
  --pids-limit=100  → Max 100 processes
  --memory=512m     → Max 512MB RAM
  --cpus=1.0        → Max 1 CPU core
```

---

## 3.7 Runtime Security Checklist

```
  Container Runtime Security:
  
  +--+--------------------------------------------------+
  | ✓| Run as non-root user (USER in Dockerfile)        |
  | ✓| Drop ALL capabilities, add only needed           |
  | ✓| Use read-only root filesystem                    |
  | ✓| Set no-new-privileges                            |
  | ✓| Apply resource limits (memory, CPU, PIDs)        |
  | ✓| Keep default seccomp profile                     |
  | ✓| Keep default AppArmor/SELinux profile             |
  | ✓| Never use --privileged                            |
  | ✓| Don't mount Docker socket into containers        |
  | ✓| Use tmpfs for writable temp directories          |
  +--+--------------------------------------------------+
```

---

## Summary Table

| Mechanism | Purpose | Docker Flag |
|-----------|---------|-------------|
| **Capabilities** | Granular root powers | `--cap-drop ALL --cap-add X` |
| **Seccomp** | System call filtering | `--security-opt seccomp=` |
| **AppArmor** | Mandatory access control | `--security-opt apparmor=` |
| **SELinux** | Security labels | Volume `:z` / `:Z` flags |
| **Read-only FS** | Prevent file modification | `--read-only` |
| **No new privileges** | Block privilege escalation | `--security-opt no-new-privileges` |
| **Resource limits** | Prevent DoS | `--memory`, `--cpus`, `--pids-limit` |
| **Non-root user** | Limit access | `--user 1000:1000` |

---

## Quick Revision Questions

1. **What does `--cap-drop ALL --cap-add NET_BIND_SERVICE` do?**
   - Removes all Linux capabilities and only adds back the ability to bind to ports below 1024. The container has minimal privileges

2. **Why should you never use `--privileged`?**
   - It grants ALL Linux capabilities, access to all host devices, disables seccomp and AppArmor, and allows mounting host filesystems. Essentially gives root on the host

3. **What is seccomp and what does Docker's default profile do?**
   - Seccomp filters system calls. Docker's default profile blocks ~44 dangerous syscalls like reboot, mount, loading kernel modules, and changing system time

4. **How do you make a container's filesystem read-only?**
   - Use `--read-only` flag. Add `--tmpfs` for directories that need to be writable (like /tmp) and volumes for persistent data

5. **What does `no-new-privileges` prevent?**
   - Prevents processes inside the container from gaining additional privileges through SUID binaries, setuid, or execve. Blocks privilege escalation attempts

6. **How do resource limits improve security?**
   - They prevent resource exhaustion attacks (fork bombs, memory leaks) from crashing the host or starving other containers

---

[← Previous: Image Security](02-image-security.md) | [Next: Docker Content Trust →](04-docker-content-trust.md)
