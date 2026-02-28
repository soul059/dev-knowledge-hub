# Chapter 6: Resource Limits

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Docker uses Linux **cgroups** to limit CPU, memory, and I/O resources for containers. Proper resource limits prevent noisy-neighbor problems, OOM kills, and resource starvation in multi-container environments.

---

## 6.1 Why Resource Limits Matter

```
  WITHOUT limits:                   WITH limits:
  
  +---------------------------+    +---------------------------+
  | Host (8 CPU, 16GB RAM)    |    | Host (8 CPU, 16GB RAM)   |
  |                           |    |                           |
  | +--------+ +--------+    |    | +--------+  +--------+   |
  | | App A  | | App B  |    |    | | App A  |  | App B  |   |
  | | 14GB!! | | OOM!   |    |    | | <=4GB  |  | <=4GB  |   |
  | | 7 CPU  | | starved|    |    | | <=2CPU |  | <=2CPU |   |
  | +--------+ +--------+    |    | +--------+  +--------+   |
  |                           |    |                           |
  | App A hogs everything!    |    | Fair allocation, both     |
  | App B dies                |    | containers stable         |
  +---------------------------+    +---------------------------+
```

---

## 6.2 Memory Limits

```bash
# Hard memory limit
docker run -d --memory=512m myapp
docker run -d --memory=2g myapp
docker run -d -m 256m myapp              # Short form

# Memory + swap limit
docker run -d --memory=512m --memory-swap=1g myapp
# 512MB RAM + 512MB swap = 1GB total

# Disable swap
docker run -d --memory=512m --memory-swap=512m myapp
# --memory-swap = --memory means no swap allowed

# Soft limit (reservation)
docker run -d --memory=1g --memory-reservation=512m myapp
# Guaranteed 512MB, can use up to 1GB

# Kernel memory limit
docker run -d --memory=512m --kernel-memory=50m myapp

# OOM kill disable (dangerous!)
docker run -d --memory=512m --oom-kill-disable myapp
```

```
  Memory Limit Behavior:
  
  Container memory usage
  ^
  |                              ← Hard limit (--memory=512m)
  |                  ___________
  |                /            OOM KILL! Process terminated
  |              /
  |            /
  |          / ← Container uses more memory
  |        /
  |      /   ← Soft limit (--memory-reservation=256m)
  |----/-----
  |  /                          System tries to reclaim
  |/                            memory when above reservation
  +---------------------------------> Time
```

### OOM (Out of Memory) Kill

```bash
# Check if container was OOM killed
docker inspect -f '{{.State.OOMKilled}}' mycontainer
# true

# Check exit code (137 = SIGKILL, often OOM)
docker inspect -f '{{.State.ExitCode}}' mycontainer
# 137

# View kernel OOM events (Linux host)
dmesg | grep -i oom
```

---

## 6.3 CPU Limits

```bash
# Limit to N CPUs
docker run -d --cpus=1.5 myapp        # Use max 1.5 cores
docker run -d --cpus=0.5 myapp        # Use max half a core

# CPU shares (relative weight, default 1024)
docker run -d --cpu-shares=512 app-low    # Half default
docker run -d --cpu-shares=2048 app-high  # Double default

# Pin to specific CPUs
docker run -d --cpuset-cpus="0,1" myapp       # Use CPU 0 and 1
docker run -d --cpuset-cpus="0-3" myapp       # Use CPU 0 through 3

# CPU period and quota (advanced)
docker run -d --cpu-period=100000 --cpu-quota=50000 myapp
# 50000/100000 = 0.5 CPU (50% of one core)
```

```
  --cpus vs --cpu-shares:
  
  --cpus=1.5 (HARD LIMIT):
  Container NEVER uses more than 1.5 cores,
  even if host has idle CPUs.
  
  --cpu-shares=512 (RELATIVE WEIGHT):
  Only matters under contention. If host is idle,
  container can use ALL available CPUs.
  
  +-----------------+----------+----------+
  |                 | Idle Host| Busy Host|
  +-----------------+----------+----------+
  | --cpus=1.5      | 1.5 max  | 1.5 max  |
  | --cpu-shares=512| All CPUs | Fair share|
  +-----------------+----------+----------+
```

---

## 6.4 I/O Limits

```bash
# Block I/O weight (relative, 10-1000, default 500)
docker run -d --blkio-weight=100 myapp

# Device read rate limit
docker run -d --device-read-bps=/dev/sda:10mb myapp

# Device write rate limit
docker run -d --device-write-bps=/dev/sda:5mb myapp

# IOPS limits
docker run -d --device-read-iops=/dev/sda:1000 myapp
docker run -d --device-write-iops=/dev/sda:500 myapp
```

---

## 6.5 PIDs Limit

```bash
# Limit number of processes in container
docker run -d --pids-limit=100 myapp

# Prevents fork bombs and runaway process creation
# Default: unlimited (or daemon default)
```

---

## 6.6 Updating Resources at Runtime

```bash
# Update running container's resources
docker update --memory=1g --cpus=2 mycontainer

# Update restart policy
docker update --restart=always mycontainer

# Update multiple containers
docker update --memory=512m container1 container2 container3
```

---

## 6.7 Monitoring Resource Usage

```bash
# Real-time stats
docker stats
docker stats mycontainer

# Snapshot (non-streaming)
docker stats --no-stream

# Custom format
docker stats --no-stream --format \
  "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}"

# Check configured limits
docker inspect -f 'Memory: {{.HostConfig.Memory}}' mycontainer
docker inspect -f 'CPUs: {{.HostConfig.NanoCpus}}' mycontainer
```

```
  docker stats output:
  
  NAME        CPU %    MEM USAGE / LIMIT    MEM %    NET I/O         BLOCK I/O
  webapp      1.25%    128MiB / 512MiB      25.00%   1.5kB / 800B    5MB / 0B
  database    5.50%    450MiB / 1GiB        43.95%   10kB / 8kB      50MB / 30MB
  cache       0.30%    64MiB / 256MiB       25.00%   500B / 200B     0B / 0B
```

---

## 6.8 Production Resource Configuration

```bash
# Typical production container
docker run -d \
  --name production-app \
  --memory=1g \
  --memory-reservation=512m \
  --cpus=2 \
  --pids-limit=200 \
  --restart=unless-stopped \
  --health-cmd="curl -f http://localhost:3000/health || exit 1" \
  --health-interval=30s \
  -p 3000:3000 \
  myapp:2.1.0

# Sizing Guidelines:
#
# +------------------+----------+--------+
# | Workload Type    | Memory   | CPUs   |
# +------------------+----------+--------+
# | Small API        | 256-512m | 0.5-1  |
# | Medium Web App   | 512m-1g  | 1-2    |
# | Database         | 1-4g     | 2-4    |
# | ML Inference     | 2-8g     | 2-4    |
# | Build Container  | 2-4g     | 2-4    |
# +------------------+----------+--------+
```

---

## Summary Table

| Resource | Flag | Default | Effect |
|----------|------|---------|--------|
| **Memory (hard)** | `--memory` / `-m` | Unlimited | OOM kill if exceeded |
| **Memory (soft)** | `--memory-reservation` | Unlimited | System reclaims above this |
| **Swap** | `--memory-swap` | 2× memory | Total memory + swap |
| **CPU (hard)** | `--cpus` | Unlimited | Max CPU time |
| **CPU (relative)** | `--cpu-shares` | 1024 | Weight under contention |
| **CPU (pinning)** | `--cpuset-cpus` | All | Specific cores only |
| **PIDs** | `--pids-limit` | Unlimited | Max processes |
| **Block I/O** | `--device-*-bps/iops` | Unlimited | Disk bandwidth/IOPS |

---

## Quick Revision Questions

1. **What happens when a container exceeds its memory limit?**
   - Docker's OOM killer terminates the container's main process. The container enters EXITED state with exit code 137 (128 + SIGKILL)

2. **What is the difference between `--cpus` and `--cpu-shares`?**
   - `--cpus` is a hard limit (container never exceeds it). `--cpu-shares` is a relative weight that only takes effect under CPU contention — idle CPUs can still be used

3. **How do you prevent a container from using swap?**
   - Set `--memory-swap` equal to `--memory`: `docker run --memory=512m --memory-swap=512m myapp`

4. **How do you update resource limits on a running container?**
   - Use `docker update`: `docker update --memory=1g --cpus=2 mycontainer`

5. **What is `--memory-reservation` and how does it differ from `--memory`?**
   - `--memory` is a hard limit (OOM kill above it). `--memory-reservation` is a soft limit — the system tries to keep the container near this value but won't kill it for exceeding the reservation

6. **Why should you set `--pids-limit`?**
   - To prevent fork bombs and runaway process creation that could exhaust the host's PID space and affect all other containers and the host system

---

[← Previous: Container Networking Basics](05-container-networking-basics.md) | [Next: Container Cleanup →](07-container-cleanup.md)
