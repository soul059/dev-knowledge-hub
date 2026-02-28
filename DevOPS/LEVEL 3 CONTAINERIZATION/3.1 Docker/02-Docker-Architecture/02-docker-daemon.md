# Chapter 2: Docker Daemon

> **Unit 2: Docker Architecture** | [Course Home](../README.md)

---

## Overview

The **Docker Daemon** (`dockerd`) is the server-side process that does the heavy lifting in Docker. It listens for API requests, manages Docker objects (images, containers, networks, volumes), and communicates with other daemons for cluster management.

---

## 2.1 What the Docker Daemon Does

```
+----------------------------------------------------------------+
|                      Docker Daemon (dockerd)                    |
|                                                                 |
|  Listens on:                                                    |
|    - /var/run/docker.sock (Unix socket, default)                |
|    - tcp://0.0.0.0:2376   (TCP, for remote access)              |
|                                                                 |
|  +-------------------+  +--------------------+                  |
|  | Image Management  |  | Container Managmnt |                  |
|  | - Build images    |  | - Create/start/stop|                  |
|  | - Pull/push       |  | - Attach/exec      |                  |
|  | - Tag/remove      |  | - Resource limits   |                  |
|  +-------------------+  +--------------------+                  |
|                                                                 |
|  +-------------------+  +--------------------+                  |
|  | Network Management|  | Volume Management  |                  |
|  | - Create networks |  | - Create volumes   |                  |
|  | - Manage drivers  |  | - Mount/unmount    |                  |
|  | - DNS resolution  |  | - Cleanup          |                  |
|  +-------------------+  +--------------------+                  |
|                                                                 |
|  +-------------------+  +--------------------+                  |
|  | Security          |  | Build (BuildKit)   |                  |
|  | - TLS             |  | - Parse Dockerfile |                  |
|  | - Authorization   |  | - Execute steps    |                  |
|  | - Secrets         |  | - Cache layers     |                  |
|  +-------------------+  +--------------------+                  |
+----------------------------------------------------------------+
```

---

## 2.2 Daemon Configuration

The Docker daemon is configured via the file `/etc/docker/daemon.json` (Linux) or through Docker Desktop settings.

### daemon.json Configuration

```json
{
  "data-root": "/var/lib/docker",
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-address-pools": [
    { "base": "172.17.0.0/16", "size": 24 }
  ],
  "dns": ["8.8.8.8", "8.8.4.4"],
  "registry-mirrors": ["https://mirror.example.com"],
  "insecure-registries": ["myregistry.local:5000"],
  "live-restore": true,
  "userland-proxy": false,
  "experimental": false,
  "metrics-addr": "0.0.0.0:9323",
  "debug": false
}
```

### Key Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| `data-root` | Root directory for Docker data | `/var/lib/docker` |
| `storage-driver` | Filesystem driver | `overlay2` |
| `log-driver` | Default logging driver | `json-file` |
| `dns` | DNS servers for containers | Host DNS |
| `live-restore` | Keep containers running if daemon stops | `false` |
| `debug` | Enable debug logging | `false` |
| `tls` | Enable TLS for remote API | `false` |
| `default-runtime` | Default OCI runtime | `runc` |
| `registry-mirrors` | Registry mirror URLs | `[]` |

---

## 2.3 Daemon Startup and Management

```bash
# Start Docker daemon
sudo systemctl start docker

# Stop Docker daemon
sudo systemctl stop docker

# Restart Docker daemon (needed after config changes)
sudo systemctl restart docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Check daemon status
sudo systemctl status docker

# View daemon logs
sudo journalctl -u docker.service -f

# Reload daemon configuration without restart
sudo systemctl reload docker
# OR
sudo kill -SIGHUP $(pidof dockerd)
```

### Daemon Startup Flow

```
  System Boot
       |
       v
  systemd starts docker.service
       |
       v
  +------------------+
  | dockerd starts   |
  |  - Read config   |
  |  - Init storage  |
  |  - Init network  |
  |  - Start API     |
  +--------+---------+
           |
           v
  +------------------+
  | containerd starts|
  |  (if not already |
  |   running)       |
  +--------+---------+
           |
           v
  +------------------+
  | Listen on socket |
  | Ready for        |
  | API requests     |
  +------------------+
```

---

## 2.4 Docker Socket

The Unix socket `/var/run/docker.sock` is the primary communication channel between the Docker CLI and the daemon.

```
  +----------+                           +----------+
  | docker   |  /var/run/docker.sock     | dockerd  |
  | CLI      | ========================>| daemon   |
  +----------+   (Unix domain socket)    +----------+
  
  Permission: srw-rw---- root docker
  
  Anyone in the 'docker' group can control the daemon.
  THIS IS EQUIVALENT TO ROOT ACCESS!
```

### Security Implications

```bash
# The docker socket grants root-equivalent access
# Anyone who can access the socket can:
#   - Mount host filesystem into a container
#   - Access host network
#   - Run privileged containers

# Add user to docker group (use with caution!)
sudo usermod -aG docker $USER

# Check who has access
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker 0 Jan  1 00:00 /var/run/docker.sock
```

---

## 2.5 Remote API Access

```
  LOCAL ACCESS (default):
  
  Client                    Daemon
  +--------+   Unix sock   +--------+
  | docker | ============> |dockerd |
  +--------+               +--------+
  Same machine only
  
  
  REMOTE ACCESS (TCP with TLS):
  
  Client Machine            Server Machine
  +--------+   TCP:2376    +--------+
  | docker | ===========> |dockerd |
  +--------+   TLS/HTTPS   +--------+
  Different machines
  Encrypted + authenticated
```

### Setting Up Remote Access with TLS

```bash
# On the server — configure daemon for remote access
# /etc/docker/daemon.json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "tls": true,
  "tlscacert": "/etc/docker/certs/ca.pem",
  "tlscert": "/etc/docker/certs/server-cert.pem",
  "tlskey": "/etc/docker/certs/server-key.pem",
  "tlsverify": true
}

# On the client — connect to remote daemon
export DOCKER_HOST=tcp://remote-server:2376
export DOCKER_TLS_VERIFY=1
export DOCKER_CERT_PATH=~/.docker/certs

docker ps  # Now connects to remote daemon!
```

---

## 2.6 Live Restore

With `live-restore` enabled, containers continue running even if the daemon stops — critical for production.

```
  WITHOUT live-restore:              WITH live-restore:
  
  Daemon stops                       Daemon stops
       |                                  |
       v                                  v
  All containers STOP               Containers KEEP RUNNING
  Service interruption!              No interruption
       |                                  |
       v                                  v
  Daemon restarts                    Daemon restarts
       |                                  |
       v                                  v
  Containers must                    Daemon reconnects
  be restarted manually              to running containers
```

```json
// /etc/docker/daemon.json
{
  "live-restore": true
}
```

---

## 2.7 Daemon Data Directory

```
  /var/lib/docker/                 (data-root)
  ├── overlay2/                    Storage driver data (image layers)
  │   ├── <layer-id>/
  │   └── ...
  ├── containers/                  Container metadata and logs
  │   ├── <container-id>/
  │   │   ├── config.v2.json
  │   │   ├── hostconfig.json
  │   │   └── <container-id>-json.log
  │   └── ...
  ├── image/                       Image metadata
  │   └── overlay2/
  │       ├── imagedb/
  │       └── layerdb/
  ├── volumes/                     Named volumes
  │   └── <volume-name>/
  │       └── _data/
  ├── network/                     Network configuration
  ├── tmp/                         Temporary files
  ├── trust/                       Content trust data
  ├── buildkit/                    BuildKit cache
  └── plugins/                     Plugin data
```

---

## 2.8 Troubleshooting the Daemon

```bash
# Daemon won't start — check logs
sudo journalctl -u docker.service --no-pager -n 50

# Common issues:

# 1. Config syntax error
sudo dockerd --validate  # Validate daemon.json

# 2. Port conflict
sudo ss -tlnp | grep 2376

# 3. Storage issues
df -h /var/lib/docker  # Check disk space
sudo docker system df  # Check Docker disk usage

# 4. Permission issues
ls -la /var/run/docker.sock

# 5. Debug mode (verbose logging)
# Add to daemon.json: {"debug": true}
sudo systemctl restart docker
sudo journalctl -u docker -f  # Watch debug logs
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Binary | `dockerd` |
| Config File | `/etc/docker/daemon.json` |
| Socket | `/var/run/docker.sock` (Unix), `tcp://host:2376` (remote) |
| Data Directory | `/var/lib/docker/` |
| Service Name | `docker.service` (systemd) |
| Logs | `journalctl -u docker` |
| Key Feature | Manages images, containers, networks, volumes, builds |
| live-restore | Keeps containers running when daemon restarts |
| Security | Socket access = root access; use TLS for remote |

---

## Quick Revision Questions

1. **Where is the Docker daemon configuration file located?**
   - `/etc/docker/daemon.json` on Linux; Docker Desktop settings on Windows/Mac

2. **What is the Docker socket and why is it a security concern?**
   - `/var/run/docker.sock` is the Unix socket for CLI-daemon communication. Anyone with access to it effectively has root access to the host

3. **What does `live-restore: true` do?**
   - It keeps containers running even when the Docker daemon is stopped or restarted, preventing service interruptions during daemon upgrades

4. **How do you enable remote access to the Docker daemon?**
   - Configure `dockerd` to listen on a TCP socket (port 2376) with TLS certificates for encryption and authentication

5. **What storage driver does Docker use by default on modern Linux?**
   - `overlay2` — a union filesystem driver that provides efficient layered storage

6. **How do you troubleshoot a Docker daemon that won't start?**
   - Check logs with `journalctl -u docker`, validate config with `dockerd --validate`, check disk space, and look for port conflicts

---

[← Previous: Docker Engine Components](01-docker-engine-components.md) | [Next: Docker CLI →](03-docker-cli.md)
