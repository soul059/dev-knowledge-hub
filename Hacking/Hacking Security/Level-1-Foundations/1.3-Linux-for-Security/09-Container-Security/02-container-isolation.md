# Container Isolation

## Unit 9 - Topic 2 | Container Security

---

## Overview

Container isolation relies on **Linux kernel features** — namespaces for resource visibility, cgroups for resource limits, and security modules for access control. Understanding how these work together reveals both the **strengths and limitations** of container isolation.

---

## 1. Linux Namespaces (Isolation)

```
┌──────────────────────────────────────────────────────────────┐
│              NAMESPACE ISOLATION PER CONTAINER                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NAMESPACE    │ WHAT IT ISOLATES                              │
│  ─────────────┼──────────────────────────────────────────     │
│  PID          │ Process IDs (container sees own PIDs)        │
│  NET          │ Network stack (own IPs, ports, routes)       │
│  MNT          │ Mount points (own filesystem)                │
│  UTS          │ Hostname and domain name                     │
│  IPC          │ Inter-process communication                  │
│  USER         │ User/Group IDs (UID/GID mapping)            │
│  CGROUP       │ Cgroup membership view                       │
│                                                              │
│  Each container gets ALL 7 namespaces                        │
│  Process in container cannot "see" outside its namespace     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# View container's namespaces
ls -la /proc/$(docker inspect -f '{{.State.Pid}}' container_name)/ns/

# Enter container's namespace
nsenter -t $(docker inspect -f '{{.State.Pid}}' container_name) -m -u -i -n -p bash

# Verify PID namespace isolation
docker exec container_name ps aux     # Only container processes
ps aux                                 # All host processes (more)
```

---

## 2. Cgroups (Resource Limits)

```bash
# === SET RESOURCE LIMITS ===
# Memory limit
docker run --memory=256m --memory-swap=512m nginx
# Container killed (OOM) if it exceeds 256MB RAM

# CPU limit
docker run --cpus=1.5 nginx           # 1.5 CPU cores max
docker run --cpu-shares=512 nginx      # Relative CPU weight

# Process limit
docker run --pids-limit=100 nginx      # Max 100 processes

# I/O limit
docker run --device-read-bps=/dev/sda:10mb nginx

# === VERIFY LIMITS ===
docker stats                           # Real-time resource usage
docker inspect container_name | grep -A 5 "Memory"

# === WHY LIMITS MATTER ===
# Without limits:
# • Fork bomb in container → exhausts host PIDs
# • Memory leak → crashes host
# • CPU-intensive process → starves other containers
```

---

## 3. Seccomp Profiles

```bash
# Seccomp filters system calls available to container
# Docker's default profile blocks ~44 dangerous syscalls

# Blocked by default:
# • mount, umount     — filesystem manipulation
# • reboot            — system reboot
# • kexec_load        — load new kernel
# • ptrace            — process debugging
# • bpf               — kernel BPF operations
# • unshare           — namespace creation

# Run with default seccomp (automatic)
docker run nginx

# Run with NO seccomp (dangerous!)
docker run --security-opt seccomp=unconfined nginx

# Run with custom seccomp profile
docker run --security-opt seccomp=custom-profile.json nginx

# View default profile
# /etc/docker/seccomp/default.json
```

```json
// Example custom seccomp profile (restrictive)
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "syscalls": [
        {
            "names": ["read", "write", "open", "close", "stat",
                      "fstat", "mmap", "mprotect", "munmap",
                      "brk", "exit_group", "execve"],
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}
```

---

## 4. Container Escape Vectors

| Vector | Description | Mitigation |
|--------|-------------|-----------|
| **Privileged mode** | Full host access | Never use `--privileged` |
| **Docker socket mount** | API access = root | Don't mount socket |
| **Kernel exploits** | Shared kernel vulnerability | Patch host kernel |
| **Sensitive mounts** | `/proc`, `/sys` writable | Use read-only mounts |
| **Capabilities** | Excessive capabilities | `--cap-drop=ALL` |
| **Host PID/NET** | `--pid=host`, `--net=host` | Use container namespaces |

```bash
# ATTACK: Container escape via privileged mode
docker run --privileged -it alpine sh
# Inside container:
mount /dev/sda1 /mnt                  # Mount host filesystem!
chroot /mnt                            # Access host as root!

# ATTACK: Escape via Docker socket
docker run -v /var/run/docker.sock:/var/run/docker.sock -it alpine sh
# Inside container:
apk add docker-cli
docker run --privileged -v /:/host alpine chroot /host
# Full host access!

# DEFENSE: Verify container security
docker inspect container_name | jq '.[0].HostConfig | {
    Privileged, CapAdd, CapDrop, SecurityOpt,
    PidMode, NetworkMode, UsernsMode
}'
```

---

## 5. Rootless Containers

```bash
# === ROOTLESS DOCKER ===
# Run Docker daemon without root
dockerd-rootless-setuptool.sh install
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock

# Benefits:
# • Docker daemon runs as regular user
# • Container root ≠ host root (user namespace)
# • Limits blast radius of container escape

# === PODMAN (Rootless by default) ===
podman run --rm -it alpine sh
# Podman = daemonless, rootless container engine
# Drop-in replacement for Docker
# No daemon = no single point of attack
```

---

## Summary Table

| Mechanism | Purpose | Default in Docker |
|-----------|---------|:---:|
| **PID namespace** | Process isolation | ✅ |
| **NET namespace** | Network isolation | ✅ |
| **Cgroups** | Resource limits | ⚠️ Unlimited |
| **Seccomp** | Syscall filtering | ✅ Default profile |
| **Capabilities** | Privilege control | ✅ Limited set |
| **User namespace** | UID remapping | ❌ Off by default |

---

## Quick Revision Questions

1. **Name the 7 Linux namespaces used by Docker.**
2. **What happens without memory limits on a container?**
3. **What does seccomp do and what syscalls does it block?**
4. **How can an attacker escape from a privileged container?**
5. **What is rootless Docker and why is it more secure?**

---

[← Previous: Docker Security Basics](01-docker-security-basics.md) | [Next: Image Security →](03-image-security.md)

---

*Unit 9 - Topic 2 of 5 | [Back to README](../README.md)*
