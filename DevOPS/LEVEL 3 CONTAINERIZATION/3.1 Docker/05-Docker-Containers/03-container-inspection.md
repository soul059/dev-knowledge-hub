# Chapter 3: Container Inspection and Debugging

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Inspecting and debugging running (or stopped) containers is a critical skill. Docker provides extensive tools for examining container state, viewing logs, executing commands inside containers, and troubleshooting issues.

---

## 3.1 Listing Containers

```bash
# Running containers
docker ps

# All containers (including stopped)
docker ps -a

# Latest container
docker ps -l

# Custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# Filter containers
docker ps -f status=running
docker ps -f name=web
docker ps -f label=env=production
docker ps -f ancestor=nginx

# Quiet mode (IDs only)
docker ps -q
docker ps -aq          # All container IDs
```

---

## 3.2 Container Inspect

```bash
# Full JSON output
docker inspect mycontainer

# Specific fields with Go template
docker inspect -f '{{.State.Status}}' mycontainer
docker inspect -f '{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect -f '{{.Config.Env}}' mycontainer
docker inspect -f '{{.HostConfig.Memory}}' mycontainer
docker inspect -f '{{.State.Pid}}' mycontainer

# Multiple fields
docker inspect -f 'IP: {{.NetworkSettings.IPAddress}} Status: {{.State.Status}}' mycontainer

# Network details
docker inspect -f '{{json .NetworkSettings.Networks}}' mycontainer | jq

# Mount details
docker inspect -f '{{json .Mounts}}' mycontainer | jq
```

```
  docker inspect output structure:
  
  {
    "Id": "abc123...",
    "State": {
      "Status": "running",        <-- Current state
      "Pid": 12345,               <-- Host PID
      "ExitCode": 0,              <-- Last exit code
      "OOMKilled": false          <-- OOM status
    },
    "Config": {
      "Image": "nginx:alpine",    <-- Image used
      "Env": [...],               <-- Environment vars
      "Cmd": [...],               <-- Command
      "ExposedPorts": {...}       <-- Declared ports
    },
    "NetworkSettings": {
      "IPAddress": "172.17.0.2",  <-- Container IP
      "Ports": {...}              <-- Port mappings
    },
    "HostConfig": {
      "Memory": 536870912,        <-- Memory limit
      "CpuShares": 1024           <-- CPU shares
    },
    "Mounts": [...]               <-- Volume mounts
  }
```

---

## 3.3 Container Logs

```bash
# View all logs
docker logs mycontainer

# Follow logs (like tail -f)
docker logs -f mycontainer

# Last N lines
docker logs --tail 100 mycontainer

# Logs since timestamp
docker logs --since 2024-01-15T10:00:00 mycontainer
docker logs --since 30m mycontainer     # Last 30 minutes
docker logs --since 2h mycontainer       # Last 2 hours

# Show timestamps
docker logs -t mycontainer

# Combined
docker logs -f --tail 50 -t mycontainer
```

```
  Log Flow:
  
  Container Process
  +------------------+
  | STDOUT ──────────+──> docker logs
  | STDERR ──────────+──> docker logs
  +------------------+
         |
         v
  Log Driver (json-file by default)
  /var/lib/docker/containers/<id>/<id>-json.log
```

### Log Drivers

```bash
# Set log driver
docker run -d --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# Available log drivers:
# json-file  (default) — JSON logs on disk
# syslog     — System syslog
# journald   — systemd journal
# fluentd    — Fluentd collector
# awslogs    — AWS CloudWatch
# gcplogs    — Google Cloud Logging
# none       — No logging
```

---

## 3.4 Executing Commands Inside Containers

```bash
# Run command in running container
docker exec mycontainer ls /app

# Interactive shell
docker exec -it mycontainer /bin/bash
docker exec -it mycontainer /bin/sh      # Alpine containers

# Run as specific user
docker exec -u root mycontainer whoami
docker exec -u 0 mycontainer apt-get update

# Set environment variable
docker exec -e DEBUG=true mycontainer printenv

# Working directory
docker exec -w /app mycontainer npm test

# Detached command
docker exec -d mycontainer touch /tmp/marker
```

```
  docker exec vs docker attach:
  
  docker exec:                    docker attach:
  +--------------------+         +--------------------+
  | Starts NEW process |         | Attaches to        |
  | inside container   |         | MAIN process (PID1)|
  |                    |         |                    |
  | Can run any cmd    |         | See main output    |
  | Independent of PID1|         | Ctrl+C may STOP    |
  | Safe to exit       |         | the container!     |
  +--------------------+         +--------------------+
  
  Use exec for debugging, attach for monitoring
```

---

## 3.5 Resource Usage Statistics

```bash
# Live stats for all running containers
docker stats

# Specific container
docker stats mycontainer

# No-stream (snapshot)
docker stats --no-stream

# Custom format
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

```
  docker stats output:
  
  CONTAINER   CPU %   MEM USAGE / LIMIT   MEM %   NET I/O        BLOCK I/O
  webserver   0.50%   25MiB / 512MiB      4.88%   1.2kB / 648B   0B / 0B
  database    2.30%   200MiB / 1GiB       19.53%  5.6kB / 3.1kB  15MB / 8MB
  cache       0.10%   50MiB / 256MiB      19.53%  800B / 400B    0B / 0B
```

---

## 3.6 Container Processes

```bash
# List processes in container
docker top mycontainer

# Output:
# UID   PID    PPID  CMD
# root  12345  12340 nginx: master process
# nginx 12346  12345 nginx: worker process
# nginx 12347  12345 nginx: worker process

# Process details on host
docker inspect -f '{{.State.Pid}}' mycontainer
# 12345

# View from host (Linux)
# ps aux | grep 12345
# ls /proc/12345/
```

---

## 3.7 Copying Files

```bash
# Copy from container to host
docker cp mycontainer:/app/config.json ./config.json
docker cp mycontainer:/var/log/app/ ./logs/

# Copy from host to container
docker cp ./newconfig.json mycontainer:/app/config.json
docker cp ./data/ mycontainer:/app/data/

# Works with running or stopped containers
docker cp stopped_container:/app/crash.log ./
```

---

## 3.8 Container Diff

```bash
# Show filesystem changes since container started
docker diff mycontainer

# Output:
# A /app/data/new-file.txt      (A = Added)
# C /etc/nginx/nginx.conf        (C = Changed)
# D /tmp/old-file.txt            (D = Deleted)
```

---

## 3.9 Debugging Techniques

### Debug a Crashed Container

```bash
# 1. Check exit code and logs
docker ps -a -f name=myapp
docker logs myapp
docker inspect -f '{{.State.ExitCode}}' myapp
docker inspect -f '{{.State.OOMKilled}}' myapp

# 2. Start the container with a shell to investigate
docker run -it --entrypoint /bin/sh myapp:latest

# 3. Inspect the failed container's filesystem
docker cp myapp:/var/log/app.log ./

# 4. Commit crashed container to image for investigation
docker commit myapp debug-myapp
docker run -it debug-myapp /bin/sh
```

### Debug Networking

```bash
# Check container IP
docker inspect -f '{{.NetworkSettings.IPAddress}}' mycontainer

# DNS resolution inside container
docker exec mycontainer nslookup other-container

# Network connectivity
docker exec mycontainer ping -c 3 other-container
docker exec mycontainer curl http://other-container:8080

# Install debugging tools (temporary)
docker exec -u root mycontainer apt-get update
docker exec -u root mycontainer apt-get install -y curl iputils-ping
```

### Debug with a Sidecar

```bash
# Attach a debug container to the same network namespace
docker run -it --network container:mycontainer nicolaka/netshoot

# Available tools in netshoot: curl, dig, nslookup, tcpdump,
# iperf, netstat, ss, ip, traceroute, mtr, etc.
```

---

## 3.10 Container Events

```bash
# Stream real-time events
docker events

# Filter by container
docker events --filter container=mycontainer

# Filter by event type
docker events --filter event=start
docker events --filter event=die
docker events --filter event=oom

# Since timestamp
docker events --since 1h
```

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `docker ps` | List containers | `-a`, `-q`, `-f` |
| `docker inspect` | Detailed container info | `-f` for format |
| `docker logs` | View container output | `-f`, `--tail`, `--since` |
| `docker exec` | Run command inside container | `-it`, `-u`, `-w` |
| `docker stats` | Live resource usage | `--no-stream`, `--format` |
| `docker top` | Container processes | — |
| `docker cp` | Copy files to/from container | — |
| `docker diff` | Filesystem changes | — |
| `docker events` | Real-time event stream | `--filter` |

---

## Quick Revision Questions

1. **What is the difference between `docker exec` and `docker attach`?**
   - `docker exec` starts a new process inside the container (safe for debugging). `docker attach` connects to the container's main process (PID 1), and Ctrl+C may stop the container

2. **How do you view only the last 50 lines of container logs?**
   - `docker logs --tail 50 mycontainer`

3. **How can you investigate a crashed container?**
   - Check logs (`docker logs`), inspect exit code (`docker inspect`), copy files out (`docker cp`), or commit the container to an image and start a shell in it

4. **What does `docker stats` show?**
   - Real-time CPU usage, memory usage and limit, network I/O, and block I/O for running containers

5. **How do you copy a file from a container to the host?**
   - `docker cp mycontainer:/path/to/file ./local-path`

6. **What does `docker diff` show?**
   - Filesystem changes in the container since it was created: Added (A), Changed (C), and Deleted (D) files

---

[← Previous: Running Containers](02-running-containers.md) | [Next: Container Logging →](04-container-logging.md)
