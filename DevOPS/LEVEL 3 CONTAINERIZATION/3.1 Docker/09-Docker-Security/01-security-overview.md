# Chapter 1: Security Overview

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

Docker security encompasses multiple layers: the host OS, the Docker daemon, container isolation, images, and the application itself. Understanding the attack surface and defense mechanisms is essential for running containers safely.

---

## 1.1 Docker Security Layers

```
  Defense in Depth:
  
  +--------------------------------------------------+
  | Layer 5: APPLICATION SECURITY                     |
  |   Input validation, auth, dependency scanning     |
  +--------------------------------------------------+
  | Layer 4: IMAGE SECURITY                           |
  |   Trusted images, scanning, minimal base images   |
  +--------------------------------------------------+
  | Layer 3: CONTAINER RUNTIME SECURITY               |
  |   Non-root, capabilities, seccomp, AppArmor      |
  +--------------------------------------------------+
  | Layer 2: DOCKER DAEMON SECURITY                   |
  |   Socket access, TLS, rootless mode               |
  +--------------------------------------------------+
  | Layer 1: HOST OS SECURITY                         |
  |   Kernel updates, hardened OS, access control     |
  +--------------------------------------------------+
  
  Security is only as strong as the WEAKEST layer
```

---

## 1.2 Container Isolation Mechanisms

```
  Linux Security Features Used by Docker:
  
  +-- Namespaces (Isolation) ----------+
  | PID:  Container sees only its       |
  |       own processes (PID 1+)        |
  | NET:  Own network stack, IPs        |
  | MNT:  Own filesystem view          |
  | UTS:  Own hostname                 |
  | IPC:  Own shared memory            |
  | USER: Own user ID mapping          |
  +------------------------------------+
  
  +-- cgroups (Resource Limits) -------+
  | CPU:    Limit CPU usage            |
  | Memory: Limit RAM + OOM control   |
  | I/O:    Limit disk bandwidth      |
  | PIDs:   Limit process count       |
  +------------------------------------+
  
  +-- Security Modules ----------------+
  | seccomp:    System call filtering  |
  | AppArmor:   Mandatory access ctrl  |
  | SELinux:    Security labels        |
  | Capabilities: Granular root powers |
  +------------------------------------+
```

---

## 1.3 The Root Problem

```
  Container Root vs Host Root:
  
  By default, container root = host root (UID 0)
  
  Container              Host
  +-------------+        +------------------+
  | root (UID 0)|  ===   | root (UID 0)     |
  | Process runs|        | Same user!       |
  | as UID 0    |        |                  |
  +-------------+        +------------------+
  
  If container escapes → ROOT on host!
  
  Mitigations:
  1. Run as non-root user (USER in Dockerfile)
  2. Use user namespace remapping
  3. Drop Linux capabilities
  4. Use rootless Docker
  5. Read-only root filesystem
```

```bash
# Check what user a container runs as
docker run --rm alpine id
# uid=0(root) gid=0(root) ← Running as root!

docker run --rm --user 1000:1000 alpine id
# uid=1000 gid=1000 ← Non-root

docker run --rm python:3.12-slim id
# uid=0(root) ← Many images default to root
```

---

## 1.4 Attack Surface Overview

```
  Docker Attack Vectors:
  
  +-- Image Attacks --------------------+
  | • Malicious/backdoored images       |
  | • Vulnerable base images            |
  | • Exposed secrets in layers         |
  | • Unpatched dependencies            |
  +-------------------------------------+
  
  +-- Runtime Attacks ------------------+
  | • Container escape (kernel exploit) |
  | • Privileged container abuse        |
  | • Docker socket access             |
  | • Resource exhaustion (DoS)         |
  +-------------------------------------+
  
  +-- Network Attacks ------------------+
  | • Exposed ports / services          |
  | • Unencrypted communication         |
  | • ARP spoofing on bridge networks   |
  | • DNS spoofing                      |
  +-------------------------------------+
  
  +-- Supply Chain Attacks -------------+
  | • Compromised CI/CD pipeline        |
  | • Typosquatting image names         |
  | • Dependency confusion              |
  +-------------------------------------+
```

---

## 1.5 Security Scanning Quick Start

```bash
# Scan image for vulnerabilities
docker scout cves nginx:latest
docker scout recommendations nginx:latest

# Using Trivy (popular open-source scanner)
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest

# Using Grype
docker run --rm \
  anchore/grype nginx:latest

# Using Snyk
docker scan nginx:latest  # Docker Desktop integration
```

```
  Vulnerability Severity Levels:
  
  CRITICAL  ███████████  Fix immediately!
  HIGH      ████████     Fix soon
  MEDIUM    ██████       Plan to fix
  LOW       ███          Monitor
  UNKNOWN   █            Investigate
  
  Focus on CRITICAL and HIGH first
  Many LOW/MEDIUM are unexploitable in context
```

---

## Summary Table

| Security Layer | Key Mechanisms | Example |
|----------------|---------------|---------|
| **Host OS** | Kernel updates, hardened OS | Ubuntu with unattended-upgrades |
| **Docker daemon** | Socket permission, TLS | Rootless mode, TLS on TCP |
| **Container isolation** | Namespaces, cgroups | PID/NET/MNT namespaces |
| **Runtime security** | Capabilities, seccomp | Drop ALL, add only needed |
| **Image security** | Base image, scanning | Alpine/distroless, Trivy scan |
| **Application** | Auth, input validation | OWASP best practices |
| **Supply chain** | Image signing, provenance | Docker Content Trust |

---

## Quick Revision Questions

1. **What Linux kernel features provide container isolation?**
   - Namespaces (PID, NET, MNT, UTS, IPC, USER) for isolation and cgroups for resource limits

2. **Why is running as root in a container dangerous?**
   - Container root (UID 0) equals host root by default. If an attacker escapes the container, they have root access to the host

3. **What are the main categories of Docker attack vectors?**
   - Image attacks (malicious/vulnerable images), runtime attacks (container escape, privileged access), network attacks (exposed ports, unencrypted traffic), and supply chain attacks (compromised CI/CD)

4. **What is seccomp in Docker?**
   - A Linux kernel feature that filters system calls. Docker's default seccomp profile blocks approximately 44 dangerous syscalls

5. **Name three tools for scanning Docker images for vulnerabilities.**
   - Docker Scout, Trivy (Aqua Security), Grype (Anchore), and Snyk

---

[← Previous Unit: Docker Compose](../08-Docker-Compose/07-compose-in-production.md) | [Next: Image Security →](02-image-security.md)
