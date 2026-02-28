# Chapter 3: Docker CLI

> **Unit 2: Docker Architecture** | [Course Home](../README.md)

---

## Overview

The **Docker CLI** (`docker`) is the primary user interface for interacting with Docker. It translates user commands into REST API calls sent to the Docker daemon. Mastering the CLI is fundamental to working with Docker effectively.

---

## 3.1 CLI Architecture

```
  +------------------------------------------------------------------+
  |  User types: docker run -d -p 80:80 nginx                        |
  +------------------------------------------------------------------+
       |
       v
  +------------------------------------------------------------------+
  |  Docker CLI (docker binary)                                       |
  |                                                                   |
  |  1. Parse command and flags                                       |
  |  2. Validate arguments                                            |
  |  3. Construct API request                                         |
  |  4. Send to Docker daemon                                         |
  |  5. Display response to user                                      |
  +------------------------------------------------------------------+
       |
       |  POST /v1.43/containers/create
       |  Host: /var/run/docker.sock
       |  Body: {"Image": "nginx", "ExposedPorts": {...}, ...}
       v
  +------------------------------------------------------------------+
  |  Docker Daemon (dockerd)                                          |
  |  Processes the request and returns response                       |
  +------------------------------------------------------------------+
```

---

## 3.2 Command Structure

Docker CLI follows a consistent pattern:

```
  docker  [management-command]  [sub-command]  [options]  [arguments]

  Examples:
  docker  container             run            -d -p 80:80  nginx
  docker  image                 pull                        ubuntu:22.04
  docker  network               create         --driver bridge  mynet
  docker  volume                create                      mydata
  
  Legacy (still works):
  docker  run  -d -p 80:80  nginx
  docker  pull  ubuntu:22.04
```

### Management Commands

```
  $ docker --help
  
  Management Commands:
  +----------------+------------------------------------------+
  | Command        | Description                              |
  +----------------+------------------------------------------+
  | container      | Manage containers                        |
  | image          | Manage images                            |
  | network        | Manage networks                          |
  | volume         | Manage volumes                           |
  | system         | Manage Docker                            |
  | builder        | Manage builds                            |
  | compose        | Docker Compose                           |
  | context        | Manage contexts                          |
  | manifest       | Manage manifests                         |
  | plugin         | Manage plugins                           |
  | trust          | Manage trust on images                   |
  | swarm          | Manage Swarm                             |
  | node           | Manage Swarm nodes                       |
  | service        | Manage Swarm services                    |
  | stack          | Manage Swarm stacks                      |
  | config         | Manage Swarm configs                     |
  | secret         | Manage Swarm secrets                     |
  +----------------+------------------------------------------+
```

---

## 3.3 Essential Commands Reference

### Container Commands

```bash
# Create and run a container
docker run [OPTIONS] IMAGE [COMMAND]
docker run -d --name web -p 8080:80 nginx          # Detached, named, port-mapped
docker run -it ubuntu bash                          # Interactive with terminal
docker run --rm alpine echo "hello"                 # Auto-remove when done

# List containers
docker ps                                           # Running only
docker ps -a                                        # All (including stopped)
docker ps -q                                        # IDs only
docker ps --format "table {{.Names}}\t{{.Status}}"  # Custom format

# Container lifecycle
docker start <container>
docker stop <container>
docker restart <container>
docker pause <container>
docker unpause <container>
docker kill <container>                             # Force stop (SIGKILL)

# Remove containers
docker rm <container>                               # Remove stopped container
docker rm -f <container>                            # Force remove (even running)
docker container prune                              # Remove all stopped

# Inspect and manage
docker inspect <container>                          # Full JSON details
docker logs <container>                             # View logs
docker logs -f <container>                          # Follow/tail logs
docker stats                                        # Live resource usage
docker top <container>                              # Running processes
docker exec -it <container> bash                    # Execute command inside
docker cp <container>:/path /local/path             # Copy files out
docker cp /local/path <container>:/path             # Copy files in
docker diff <container>                             # Filesystem changes
```

### Image Commands

```bash
# Pull and push
docker pull nginx:latest
docker push myregistry/myimage:v1

# List and inspect
docker images                                       # List all images
docker image inspect nginx                          # Full details
docker image history nginx                          # Layer history

# Build
docker build -t myapp:v1 .                          # Build from Dockerfile
docker build -t myapp:v1 -f Dockerfile.prod .       # Specify Dockerfile

# Remove
docker rmi nginx                                    # Remove image
docker image prune                                  # Remove dangling images
docker image prune -a                               # Remove all unused images

# Tag
docker tag myapp:v1 myregistry/myapp:v1
docker tag myapp:v1 myapp:latest
```

### System Commands

```bash
# System info
docker info                                         # Full system information
docker version                                      # Version details

# Disk usage
docker system df                                    # Docker disk usage
docker system df -v                                 # Verbose disk usage

# Cleanup
docker system prune                                 # Remove unused data
docker system prune -a --volumes                    # AGGRESSIVE cleanup

# Events
docker system events                                # Real-time event stream
docker system events --filter type=container        # Filter events
```

---

## 3.4 Command Flags Cheat Sheet

### `docker run` Flags

| Flag | Short | Description | Example |
|------|-------|-------------|---------|
| `--detach` | `-d` | Run in background | `docker run -d nginx` |
| `--interactive` | `-i` | Keep STDIN open | `docker run -i ubuntu` |
| `--tty` | `-t` | Allocate pseudo-TTY | `docker run -t ubuntu` |
| `--name` | | Assign container name | `--name myapp` |
| `--publish` | `-p` | Map port | `-p 8080:80` |
| `--volume` | `-v` | Mount volume | `-v /data:/app/data` |
| `--env` | `-e` | Set environment variable | `-e DB_HOST=db` |
| `--network` | | Connect to network | `--network mynet` |
| `--rm` | | Auto-remove when stopped | `docker run --rm alpine` |
| `--restart` | | Restart policy | `--restart unless-stopped` |
| `--memory` | `-m` | Memory limit | `-m 512m` |
| `--cpus` | | CPU limit | `--cpus 1.5` |
| `--workdir` | `-w` | Working directory | `-w /app` |
| `--user` | `-u` | Run as user | `-u 1000:1000` |
| `--hostname` | `-h` | Container hostname | `-h myhost` |
| `--env-file` | | Load env vars from file | `--env-file .env` |

---

## 3.5 Docker Context — Managing Multiple Environments

```bash
# List contexts
docker context ls

# Create context for remote Docker host
docker context create production \
  --docker "host=tcp://prod-server:2376,ca=ca.pem,cert=cert.pem,key=key.pem"

# Switch context
docker context use production

# Run command against specific context
docker --context production ps
```

```
  +------------------+     +-----------------+     +----------------+
  |  docker context  |     | docker context  |     | docker context |
  |  "default"       |     | "staging"       |     | "production"   |
  |  local daemon    |     | staging server  |     | prod cluster   |
  +------------------+     +-----------------+     +----------------+
         |                        |                       |
         v                        v                       v
  Local dockerd           Staging dockerd          Prod dockerd
```

---

## 3.6 Formatting Output

Docker supports Go templates for custom output formatting:

```bash
# Container listing with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# Image listing
docker images --format "{{.Repository}}:{{.Tag}} - {{.Size}}"

# Inspect specific fields
docker inspect --format '{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect --format '{{.State.Status}}' mycontainer
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer

# JSON output for scripting
docker inspect mycontainer | jq '.[0].State'
```

---

## 3.7 Useful CLI Patterns

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove all dangling images
docker image prune -f

# Get IP of a container
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name

# Follow logs with timestamps
docker logs -f --timestamps mycontainer

# Monitor resource usage
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Export/import containers
docker export mycontainer > container.tar
docker import container.tar myimage:restored

# Save/load images
docker save myimage:v1 > image.tar
docker load < image.tar
```

---

## 3.8 Shell Aliases for Productivity

```bash
# Add to ~/.bashrc or ~/.zshrc
alias d='docker'
alias dc='docker compose'
alias dps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dpsa='docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dimg='docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"'
alias dlogs='docker logs -f'
alias dexec='docker exec -it'
alias dstats='docker stats --no-stream'
alias dprune='docker system prune -af --volumes'
alias dstop='docker stop $(docker ps -q) 2>/dev/null'
```

---

## Summary Table

| Command Category | Key Commands |
|-----------------|-------------|
| **Run** | `docker run`, `docker start`, `docker stop`, `docker rm` |
| **List** | `docker ps`, `docker images`, `docker volume ls`, `docker network ls` |
| **Inspect** | `docker inspect`, `docker logs`, `docker stats`, `docker top` |
| **Execute** | `docker exec`, `docker attach` |
| **Build** | `docker build`, `docker tag`, `docker push`, `docker pull` |
| **Clean** | `docker system prune`, `docker image prune`, `docker container prune` |
| **System** | `docker info`, `docker version`, `docker system df` |

---

## Quick Revision Questions

1. **What is the general structure of a Docker CLI command?**
   - `docker [management-command] [sub-command] [options] [arguments]` — e.g., `docker container run -d nginx`

2. **How do you run a container in the background and map port 8080 to port 80?**
   - `docker run -d -p 8080:80 nginx`

3. **What is the difference between `docker stop` and `docker kill`?**
   - `docker stop` sends SIGTERM and waits (default 10s) for graceful shutdown, then sends SIGKILL. `docker kill` sends SIGKILL immediately

4. **How do you get an interactive shell inside a running container?**
   - `docker exec -it <container> bash` (or `sh` for Alpine-based images)

5. **What does `docker system prune -a --volumes` do?**
   - Removes ALL stopped containers, ALL unused images (not just dangling), ALL unused networks, and ALL unused volumes — aggressive cleanup

6. **How do you manage multiple Docker hosts from a single CLI?**
   - Using `docker context` — create contexts for each host, switch with `docker context use <name>`

---

[← Previous: Docker Daemon](02-docker-daemon.md) | [Next: containerd and runc →](04-containerd-and-runc.md)
