# Chapter 6.4: Container Runtime Security

## Overview

Securing a running container is as important as building a secure image. Runtime security applies OS-level mechanisms (seccomp, AppArmor, SELinux), restricts capabilities, enforces read-only filesystems, and detects anomalous behavior. This chapter covers how to lock down containers after deployment.

---

## Container Runtime Attack Surface

```
CONTAINER RUNTIME ATTACK SURFACE:

  ┌─────────────────────────────────────────────────┐
  │              HOST OPERATING SYSTEM               │
  │                                                   │
  │  ┌─────────────┐  ┌─────────────┐  ┌──────────┐ │
  │  │ Container A  │  │ Container B  │  │ Cont. C  │ │
  │  │             │  │             │  │          │  │
  │  │ Attack       │  │             │  │          │  │
  │  │ Vectors:     │  │             │  │          │  │
  │  │ ┌──────────┐│  │             │  │          │  │
  │  │ │Process   ││  │             │  │          │  │
  │  │ │Execution ││  │             │  │          │  │
  │  │ ├──────────┤│  │             │  │          │  │
  │  │ │Network   ││  │             │  │          │  │
  │  │ │Access    ││  │             │  │          │  │
  │  │ ├──────────┤│  │             │  │          │  │
  │  │ │Filesystem││  │             │  │          │  │
  │  │ │Write     ││  │             │  │          │  │
  │  │ ├──────────┤│  │             │  │          │  │
  │  │ │Privilege ││  │             │  │          │  │
  │  │ │Escalation││  │             │  │          │  │
  │  │ └──────────┘│  │             │  │          │  │
  │  └─────────────┘  └─────────────┘  └──────────┘ │
  │                                                   │
  │  ┌───────────────────────────────────────────────┐│
  │  │ Linux Kernel (cgroups, namespaces, seccomp)   ││
  │  └───────────────────────────────────────────────┘│
  └─────────────────────────────────────────────────┘
```

---

## Docker Security Options

```bash
# HARDENED docker run command
docker run -d \
    --name myapp \
    --read-only \                          # Read-only filesystem
    --tmpfs /tmp:rw,noexec,nosuid,size=64m \ # Writable tmp with limits
    --cap-drop ALL \                       # Drop ALL Linux capabilities
    --cap-add NET_BIND_SERVICE \           # Add back only what's needed
    --security-opt no-new-privileges:true \ # Prevent privilege escalation
    --security-opt seccomp=seccomp-profile.json \ # Custom seccomp profile
    --security-opt apparmor=docker-custom \       # AppArmor profile
    --pids-limit 100 \                     # Limit PID count (fork bomb prot.)
    --memory 512m \                        # Memory limit
    --cpus 1.0 \                           # CPU limit
    --user 1000:1000 \                     # Run as non-root UID/GID
    --network custom-net \                 # Use custom network (not default)
    -p 8080:8080 \                         # Expose only needed ports
    myapp:1.0.0
```

---

## Linux Capabilities

```
DEFAULT DOCKER CAPABILITIES (Too many!):

  ┌────────────────────────────────────────────────┐
  │           DEFAULT CAPABILITIES                  │
  │                                                  │
  │  CHOWN           SETUID          SETGID          │
  │  DAC_OVERRIDE    FSETID          FOWNER          │
  │  KILL            SETPCAP         NET_BIND_SERVICE│
  │  NET_RAW         SYS_CHROOT     MKNOD           │
  │  AUDIT_WRITE     SETFCAP                        │
  │                                                  │
  │  Many of these are UNNECESSARY for most apps!    │
  └────────────────────────────────────────────────┘

  RECOMMENDED: Drop ALL, add back only what you need

  ┌────────────────────────────────────────────────┐
  │  --cap-drop ALL                                 │
  │  --cap-add NET_BIND_SERVICE   (bind port <1024)│
  │  --cap-add CHOWN              (change file own)│
  │  --cap-add SETUID / SETGID    (switch users)   │
  └────────────────────────────────────────────────┘
```

### Common Capabilities Reference

| Capability | Purpose | When Needed |
|-----------|---------|------------|
| `NET_BIND_SERVICE` | Bind ports below 1024 | Web servers on port 80/443 |
| `CHOWN` | Change file ownership | File management apps |
| `SETUID` / `SETGID` | Switch UID/GID | Apps switching to non-root at startup |
| `NET_RAW` | Raw sockets | Ping, network diagnostic tools |
| `SYS_PTRACE` | Process tracing | Debugging only (NEVER in production) |
| `SYS_ADMIN` | Broad admin access | Almost NEVER needed — red flag |

---

## Seccomp Profiles

Seccomp (Secure Computing Mode) restricts which system calls a container can make.

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": [
    "SCMP_ARCH_X86_64",
    "SCMP_ARCH_X86"
  ],
  "syscalls": [
    {
      "names": [
        "read", "write", "open", "close",
        "stat", "fstat", "lstat",
        "mmap", "mprotect", "munmap",
        "brk", "rt_sigaction", "rt_sigprocmask",
        "ioctl", "access", "pipe",
        "select", "socket", "connect",
        "accept", "sendto", "recvfrom",
        "bind", "listen", "clone",
        "execve", "exit", "exit_group",
        "getpid", "getuid", "getgid",
        "futex", "epoll_create", "epoll_ctl",
        "epoll_wait", "openat", "newfstatat"
      ],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

```bash
# Apply custom seccomp profile
docker run --security-opt seccomp=custom-profile.json myapp

# Check if seccomp is enabled
docker inspect --format='{{.HostConfig.SecurityOpt}}' myapp

# Generate a seccomp profile with OCI runtime (strace-based)
# Use oci-seccomp-bpf-hook to auto-generate profiles
sudo podman run --annotation io.containers.trace-syscall="of:/tmp/profile.json" myapp
```

---

## AppArmor Profiles

```
# /etc/apparmor.d/docker-webapp

#include <tunables/global>

profile docker-webapp flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  # Deny network raw access
  deny network raw,

  # Deny mount
  deny mount,

  # Deny ptrace
  deny ptrace (read, readby),

  # Allow read-only access to app directory
  /app/** r,
  /app/public/** r,

  # Allow write only to specific directories
  /tmp/** rw,
  /app/logs/** w,

  # Deny access to sensitive paths
  deny /etc/shadow r,
  deny /etc/passwd w,
  deny /proc/*/mem rw,
  deny /sys/** w,

  # Allow necessary network operations
  network tcp,
  network udp,
}
```

```bash
# Load AppArmor profile
sudo apparmor_parser -r -W /etc/apparmor.d/docker-webapp

# Run container with profile
docker run --security-opt apparmor=docker-webapp myapp

# Check loaded profiles
sudo aa-status
```

---

## Read-Only Filesystem

```yaml
# docker-compose.yml with read-only filesystem
version: "3.8"

services:
  webapp:
    image: myapp:1.0.0
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=64m
      - /var/run:rw,noexec,nosuid,size=1m
    volumes:
      - app-logs:/app/logs:rw    # Only logs are writable
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    user: "1000:1000"
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
        reservations:
          memory: 256M
          cpus: "0.5"

volumes:
  app-logs:
    driver: local
```

---

## Resource Limits (cgroups)

```
RESOURCE LIMITS — Defense Against DoS:

  ┌──────────────────────────────────────────────────┐
  │ Resource      │ Flag             │ Recommended    │
  ├───────────────┼──────────────────┼────────────────┤
  │ Memory        │ --memory 512m    │ Based on load  │
  │ CPU           │ --cpus 1.0       │ Based on usage │
  │ PIDs          │ --pids-limit 100 │ 50-200         │
  │ Open Files    │ --ulimit nofile  │ 1024:2048      │
  │ Storage       │ --storage-opt    │ Based on needs │
  │ Restart       │ --restart        │ on-failure:5   │
  └──────────────────────────────────────────────────┘
```

```bash
# Set resource limits
docker run -d \
    --memory 512m \
    --memory-swap 512m \          # Same as memory = no swap
    --cpus 1.0 \
    --pids-limit 100 \
    --ulimit nofile=1024:2048 \
    --restart on-failure:5 \
    myapp:1.0.0

# Check resource usage in real time
docker stats myapp
```

---

## Docker Compose Security Hardening

```yaml
# Fully hardened docker-compose.yml
version: "3.8"

services:
  api:
    image: company/api:1.2.3
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=64m
    security_opt:
      - no-new-privileges:true
      - seccomp=seccomp-profile.json
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    user: "1000:1000"
    networks:
      - backend
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
          pids: 100
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  db:
    image: postgres:16.1-alpine
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
      - /run/postgresql:rw,noexec,nosuid
    volumes:
      - db-data:/var/lib/postgresql/data
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETUID
      - SETGID
      - FOWNER
    user: "999:999"
    networks:
      - backend
    # NO ports exposed to host — only accessible from backend network

networks:
  backend:
    driver: bridge
    internal: false

volumes:
  db-data:
    driver: local
```

---

## Runtime Anomaly Detection

```
RUNTIME MONITORING FLOW:

  Container Activity
       │
       ▼
  ┌──────────────┐    ┌──────────────────┐
  │ System Call   │───▶│ Seccomp Filter   │──▶ Block/Allow
  │ Interception  │    └──────────────────┘
  │               │
  │               │    ┌──────────────────┐
  │               │───▶│ Falco / Sysdig   │──▶ Alert
  │               │    │ (eBPF-based)     │    ┌────────┐
  │               │    └──────────────────┘    │ Slack  │
  │               │                            │ PagerD │
  │               │    ┌──────────────────┐    │ SIEM   │
  │               │───▶│ AuditD Logs      │──▶ └────────┘
  └──────────────┘    └──────────────────┘
```

### Falco Rules for Container Monitoring

```yaml
# Custom Falco rules for container security

- rule: Shell Spawned in Container
  desc: Detect shell started inside a container
  condition: >
    spawned_process and container and
    proc.name in (bash, sh, zsh, csh, dash, ash)
  output: >
    Shell spawned in container
    (user=%user.name container=%container.name
    shell=%proc.name parent=%proc.pname
    cmdline=%proc.cmdline image=%container.image.repository)
  priority: WARNING
  tags: [container, shell]

- rule: Write to Sensitive Directory
  desc: Detect writes to /etc, /usr/bin, /usr/sbin
  condition: >
    open_write and container and
    fd.directory in (/etc, /usr/bin, /usr/sbin, /bin, /sbin)
  output: >
    Write to sensitive dir in container
    (file=%fd.name container=%container.name image=%container.image.repository)
  priority: ERROR
  tags: [container, filesystem]

- rule: Outbound Connection to Unusual Port
  desc: Detect container connecting to non-standard ports
  condition: >
    outbound and container and
    not fd.sport in (80, 443, 8080, 8443, 53, 5432, 3306, 6379)
  output: >
    Unexpected outbound connection
    (port=%fd.sport container=%container.name ip=%fd.sip)
  priority: NOTICE
  tags: [container, network]
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Application can't write files | Read-only filesystem | Add `tmpfs` for writable dirs or specific volume mounts |
| Container crashes on start | Dropped necessary capability | Identify needed caps with `--cap-drop ALL` then add one-by-one |
| "Operation not permitted" errors | Seccomp blocking system call | Audit with `strace`, update seccomp profile to allow needed syscalls |
| Container using too much memory | No resource limits set | Set `--memory` and `--memory-swap` limits |
| Fork bomb crashes host | No PID limits | Set `--pids-limit 100` |

---

## Summary Table

| Security Mechanism | What It Does |
|-------------------|-------------|
| **--cap-drop ALL** | Remove all Linux capabilities; add back only needed ones |
| **--read-only** | Prevent filesystem writes (use tmpfs for writable dirs) |
| **no-new-privileges** | Prevent privilege escalation via setuid binaries |
| **Seccomp** | Restrict which syscalls a container can use |
| **AppArmor / SELinux** | Mandatory access control at OS level |
| **Resource limits** | Prevent CPU/memory/PID exhaustion attacks |
| **Falco** | eBPF-based runtime anomaly detection and alerting |

---

## Quick Revision Questions

1. **What does `--cap-drop ALL` do and why is it recommended?**
   > It removes all Linux capabilities from the container. By default, Docker grants 14+ capabilities. Dropping all and adding back only what's needed follows the principle of least privilege. For example, a web server only needs `NET_BIND_SERVICE` to bind port 80.

2. **How does seccomp improve container security?**
   > Seccomp restricts which Linux system calls a container can execute. Docker's default seccomp profile blocks ~44 dangerous syscalls. Custom profiles can whitelist only the exact syscalls your application needs, making exploitation much harder.

3. **Why use `--read-only` with `tmpfs`?**
   > `--read-only` prevents attackers from writing malicious files, dropping webshells, or modifying binaries. `tmpfs` provides writable memory-backed directories (like /tmp) that the application needs, without allowing persistent writes to the filesystem.

4. **What does `no-new-privileges` prevent?**
   > It prevents any process inside the container from gaining additional privileges through setuid/setgid binaries or filesystem capabilities. Even if an attacker finds a setuid binary, they cannot use it to escalate to root.

5. **How does Falco detect container threats at runtime?**
   > Falco uses eBPF to intercept system calls at the kernel level. It applies rules to detect suspicious activities like shell spawning, sensitive file access, unexpected network connections, or privilege escalation attempts, and generates real-time alerts.

6. **What's the difference between capabilities and seccomp?**
   > Capabilities control *what privileged operations* a process can perform (e.g., bind low ports, change file ownership). Seccomp controls *which system calls* a process can make. They complement each other — capabilities restrict privilege level, seccomp restricts the API surface.

---

[← Previous: 6.3 Dockerfile Best Practices](03-dockerfile-best-practices.md) | [Next: 6.5 Container Registries Security →](05-container-registries-security.md)

[Back to Table of Contents](../README.md)
