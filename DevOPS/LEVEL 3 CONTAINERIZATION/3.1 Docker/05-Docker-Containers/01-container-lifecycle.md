# Chapter 1: Container Lifecycle

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Understanding the **container lifecycle** is fundamental to working with Docker. Containers move through distinct states from creation to removal, and Docker provides commands to control each transition.

---

## 1.1 Container States

```
  +----------------------------------------------------------+
  |                  Container Lifecycle                       |
  |                                                           |
  |  docker create     docker start     (process exits)       |
  |  +----------+      +---------+      +---------+           |
  |  | CREATED  | ---> | RUNNING | ---> | EXITED  |           |
  |  +----------+      +---------+      +---------+           |
  |                       |    ^           |                   |
  |                       |    |           |                   |
  |            docker     |    | docker    | docker rm         |
  |            pause      |    | unpause   |                   |
  |                       v    |           v                   |
  |                    +--------+      +---------+             |
  |                    | PAUSED |      | REMOVED |             |
  |                    +--------+      +---------+             |
  |                                                           |
  |  docker stop / kill ---> EXITED                           |
  |  docker restart -------> RUNNING (stop + start)           |
  |                                                           |
  |  OOM Kill / Error -----> EXITED (with error code)         |
  |                                                           |
  +----------------------------------------------------------+
```

### All Container States

| State | Description | How to Enter |
|-------|-------------|-------------|
| **Created** | Container exists but never started | `docker create` |
| **Running** | Process is executing | `docker start` / `docker run` |
| **Paused** | Process frozen (SIGSTOP) | `docker pause` |
| **Restarting** | Container being restarted | `docker restart` / restart policy |
| **Exited** | Process has stopped | Process exits / `docker stop` |
| **Dead** | Failed removal (stuck) | Error during removal |
| **Removing** | Being deleted | `docker rm` |

---

## 1.2 Creating and Running Containers

```bash
# docker run = docker create + docker start + docker attach
docker run nginx

# Breakdown:
docker create nginx          # Create container (CREATED state)
docker start <container_id>  # Start container (RUNNING state)

# Common run flags
docker run -d nginx                     # Detached mode (background)
docker run -it ubuntu bash              # Interactive + TTY
docker run --name webserver nginx       # Named container
docker run --rm nginx                   # Auto-remove when stopped
docker run -p 8080:80 nginx             # Port mapping
docker run -e KEY=value nginx           # Environment variable
docker run -v /host:/container nginx    # Volume mount
```

```
  docker run lifecycle:
  
  Client                  Daemon                   containerd
  ------                  ------                   ----------
  docker run nginx
       |
       |--- API call --->  Pull image (if needed)
       |                   Create container
       |                        |
       |                        |--- Create task ---> runc create
       |                        |--- Start task --->  runc start
       |                        |
       |<-- Stream logs -------|
       |    (if attached)       |
```

---

## 1.3 Stopping and Removing Containers

```bash
# Graceful stop (SIGTERM, then SIGKILL after timeout)
docker stop mycontainer              # Default: 10s grace period
docker stop -t 30 mycontainer        # 30s grace period

# Forceful stop (immediate SIGKILL)
docker kill mycontainer
docker kill -s SIGQUIT mycontainer   # Custom signal

# Remove stopped container
docker rm mycontainer

# Force remove (even if running)
docker rm -f mycontainer

# Remove all stopped containers
docker container prune
docker rm $(docker ps -aq -f status=exited)
```

```
  docker stop vs docker kill:
  
  docker stop:
  +----------+   SIGTERM   +----------+   10s timeout   +----------+
  | RUNNING  | ----------> | Graceful | -------------> | SIGKILL  |
  |          |             | Shutdown |   if still      | (forced) |
  +----------+             +----------+   running       +----------+
  
  docker kill:
  +----------+   SIGKILL   +----------+
  | RUNNING  | ----------> | EXITED   |
  |          |  (immediate)|          |
  +----------+             +----------+
```

---

## 1.4 Pausing and Restarting

```bash
# Pause (freezes all processes using cgroups freezer)
docker pause mycontainer

# Unpause
docker unpause mycontainer

# Restart (stop + start)
docker restart mycontainer
docker restart -t 5 mycontainer      # 5s grace period before kill
```

---

## 1.5 Container Restart Policies

```bash
# Set restart policy
docker run -d --restart=always nginx
docker run -d --restart=unless-stopped nginx
docker run -d --restart=on-failure:5 nginx

# Update running container's restart policy
docker update --restart=always mycontainer
```

```
  Restart Policies:
  
  +------------------+------------------------------------------+
  | Policy           | Behavior                                 |
  +------------------+------------------------------------------+
  | no               | Never restart (default)                  |
  | always           | Always restart, even manual stop +       |
  |                  | restarts on daemon startup               |
  | unless-stopped   | Like always, but NOT after manual stop   |
  | on-failure[:N]   | Restart only on non-zero exit code       |
  |                  | Optional max retry count N               |
  +------------------+------------------------------------------+
  
  always vs unless-stopped:
  
  docker stop mycontainer
  systemctl restart docker
  
  always:         Container restarts ✓
  unless-stopped: Container stays stopped ✗
```

---

## 1.6 Container Exit Codes

```
  +------+--------------------------------------------+
  | Code | Meaning                                    |
  +------+--------------------------------------------+
  |  0   | Success (normal exit)                      |
  |  1   | General error                              |
  |  2   | Misuse of shell command                    |
  | 126  | Command not executable                     |
  | 127  | Command not found                          |
  | 128  | Invalid exit argument                      |
  | 130  | Terminated by Ctrl+C (128 + SIGINT=2)      |
  | 137  | Killed by SIGKILL (128 + SIGKILL=9)        |
  |      | Often means OOM killed                     |
  | 139  | Segmentation fault (128 + SIGSEGV=11)      |
  | 143  | Terminated by SIGTERM (128 + SIGTERM=15)    |
  +------+--------------------------------------------+
```

```bash
# Check exit code
docker inspect --format='{{.State.ExitCode}}' mycontainer

# Check if OOM killed
docker inspect --format='{{.State.OOMKilled}}' mycontainer
```

---

## 1.7 Container Lifecycle Commands Reference

```bash
# Create
docker create --name myapp nginx        # Create without starting
docker run -d --name myapp nginx        # Create and start

# Inspect state
docker ps                               # Running containers
docker ps -a                            # All containers
docker inspect myapp                    # Full details
docker inspect -f '{{.State.Status}}' myapp  # Just status

# Interact
docker exec -it myapp bash             # Run command inside
docker attach myapp                     # Attach to main process
docker logs myapp                       # View logs
docker logs -f myapp                    # Follow logs

# Control
docker start myapp                      # Start stopped container
docker stop myapp                       # Graceful stop
docker kill myapp                       # Force stop
docker restart myapp                    # Restart
docker pause myapp                      # Freeze
docker unpause myapp                    # Unfreeze

# Modify
docker rename myapp newname             # Rename
docker update --memory 512m myapp       # Update resources

# Cleanup
docker rm myapp                         # Remove stopped
docker rm -f myapp                      # Force remove
docker container prune                  # Remove all stopped
```

---

## Summary Table

| State | Entry Action | Exit Action | Key Behavior |
|-------|-------------|-------------|-------------|
| **Created** | `docker create` | `docker start` / `docker rm` | Container ready, not running |
| **Running** | `docker start` / `docker run` | `docker stop/kill/pause` | Process executing |
| **Paused** | `docker pause` | `docker unpause` | Process frozen via cgroups |
| **Exited** | Process exits / `docker stop` | `docker start` / `docker rm` | Process stopped, filesystem preserved |
| **Removed** | `docker rm` | — | Container fully deleted |

---

## Quick Revision Questions

1. **What is the difference between `docker create` and `docker run`?**
   - `docker create` only creates the container (CREATED state). `docker run` creates, starts, and optionally attaches to the container in one command

2. **What happens when you `docker stop` a container?**
   - Docker sends SIGTERM to the main process, waits a grace period (default 10s), then sends SIGKILL if the process hasn't exited

3. **What does exit code 137 indicate?**
   - Exit code 137 = 128 + 9 (SIGKILL). The process was forcefully killed, often due to an Out-Of-Memory (OOM) condition or `docker kill`

4. **What is the difference between `always` and `unless-stopped` restart policies?**
   - `always` restarts the container even after a manual `docker stop` when the daemon restarts. `unless-stopped` does not restart manually stopped containers after a daemon restart

5. **How does `docker pause` work internally?**
   - It uses the cgroups freezer subsystem to freeze all processes in the container. Processes are suspended in place and resume exactly where they left off when unpaused

6. **What is the `--rm` flag used for?**
   - `--rm` automatically removes the container and its anonymous volumes when the container exits, useful for temporary/one-off containers

---

[← Previous Unit: Image Optimization](../04-Dockerfile/07-image-optimization.md) | [Next: Running Containers →](02-running-containers.md)
