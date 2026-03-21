# Docker Security Basics

## Unit 9 - Topic 1 | Container Security

---

## Overview

**Docker** revolutionized software deployment but introduced new security challenges. Containers share the **host kernel**, meaning a container escape can compromise the entire host. Understanding Docker's security model — **namespaces**, **cgroups**, **capabilities**, and **seccomp** — is essential for secure container operations.

---

## 1. Docker Architecture & Security

```
┌──────────────────────────────────────────────────────────────┐
│              DOCKER SECURITY ARCHITECTURE                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │Container │  │Container │  │Container │                  │
│  │   App A  │  │   App B  │  │   App C  │                  │
│  ├──────────┤  ├──────────┤  ├──────────┤                  │
│  │ Libs/Bins│  │ Libs/Bins│  │ Libs/Bins│                  │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘                  │
│        │              │              │                       │
│  ┌─────┴──────────────┴──────────────┴─────┐                │
│  │         Docker Engine (dockerd)          │                │
│  ├─────────────────────────────────────────┤                │
│  │         Container Runtime (runc)         │                │
│  ├─────────────────────────────────────────┤                │
│  │ Namespaces │ Cgroups │ Capabilities │    │                │
│  │ Seccomp    │ AppArmor│ SELinux      │    │                │
│  ├─────────────────────────────────────────┤                │
│  │            HOST KERNEL                   │                │
│  └─────────────────────────────────────────┘                │
│                                                              │
│  ⚠️ Containers share the HOST KERNEL                         │
│  ⚠️ Kernel vulnerability = ALL containers at risk           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Docker Security Defaults

| Mechanism | Description | Default |
|-----------|-------------|---------|
| **Namespaces** | Process, network, mount isolation | ✅ Enabled |
| **Cgroups** | Resource limits (CPU, RAM) | ⚠️ Unlimited |
| **Capabilities** | Reduced root powers | ✅ Limited set |
| **Seccomp** | System call filtering | ✅ Default profile |
| **AppArmor** | MAC confinement | ✅ Default profile |
| **Root user** | Container runs as root | ⚠️ Root by default |
| **Read-only FS** | Immutable filesystem | ❌ Writable |

---

## 3. Docker Security Commands

```bash
# === DANGEROUS PRACTICES ===
# ❌ NEVER do these in production:
docker run --privileged nginx          # Full host access!
docker run -v /:/host nginx            # Host filesystem mounted!
docker run --net=host nginx            # Host network stack!
docker run --pid=host nginx            # Host PID namespace!

# === SECURE PRACTICES ===
# ✅ Run as non-root user
docker run --user 1000:1000 nginx

# ✅ Drop all capabilities, add only needed
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# ✅ Read-only filesystem
docker run --read-only nginx

# ✅ No new privileges
docker run --security-opt=no-new-privileges nginx

# ✅ Resource limits
docker run --memory=256m --cpus=0.5 nginx

# ✅ Temporary filesystem for writable needs
docker run --read-only --tmpfs /tmp:rw,noexec,nosuid nginx

# === INSPECT SECURITY ===
docker inspect --format='{{.HostConfig.Privileged}}' container_name
docker inspect --format='{{.HostConfig.CapAdd}}' container_name
docker inspect --format='{{.Config.User}}' container_name
```

---

## 4. Docker Daemon Security

```bash
# === SECURE DOCKER DAEMON ===

# 1. Don't expose Docker socket over TCP
# ❌ dockerd -H tcp://0.0.0.0:2375        # Unauthenticated access!
# ✅ Use Unix socket (default): /var/run/docker.sock

# 2. Enable user namespaces (remap root)
# /etc/docker/daemon.json
{
    "userns-remap": "default",
    "no-new-privileges": true,
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "icc": false,
    "live-restore": true
}
# userns-remap: container root ≠ host root
# icc: false — disable inter-container communication
# no-new-privileges: prevent privilege escalation

# 3. Restrict who can use Docker
# Docker group = root access!
sudo usermod -aG docker user          # ⚠️ This gives user ROOT equivalent access
# Better: use rootless Docker or Podman
```

---

## 5. Container vs VM Security

| Feature | Container | Virtual Machine |
|---------|:---------:|:---------------:|
| **Isolation** | Kernel namespaces | Full hardware virtualization |
| **Kernel** | Shared with host | Separate kernel |
| **Escape risk** | Higher | Lower |
| **Overhead** | Minimal | Significant |
| **Boot time** | Seconds | Minutes |
| **Attack surface** | Shared kernel | Hypervisor only |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Shared kernel** | Containers share host kernel — kernel vuln = host compromise |
| **--privileged** | NEVER use — gives full host access |
| **Non-root** | Always run containers as non-root user |
| **Capabilities** | Drop ALL, add only what's needed |
| **Read-only** | Use `--read-only` for immutable containers |
| **Docker socket** | Protect — access = root on host |

---

## Quick Revision Questions

1. **Why is `--privileged` dangerous?**
2. **How do containers differ from VMs in terms of security?**
3. **What does `--cap-drop=ALL` do?**
4. **Why is Docker socket access equivalent to root access?**
5. **How do you run a container as a non-root user?**

---

[Next: Container Isolation →](02-container-isolation.md)

---

*Unit 9 - Topic 1 of 5 | [Back to README](../README.md)*
