# 🐳 Docker — Comprehensive Study Notes

> **Complete guide to Docker containerization: from fundamentals to production deployment.**

---

## Course Overview

Docker is the industry-standard platform for building, shipping, and running applications inside **containers**. Containers package an application with all its dependencies, ensuring it runs consistently across any environment — development, testing, staging, and production.

These study notes cover Docker from the ground up: container concepts, Docker architecture, images, Dockerfiles, container management, networking, volumes, Docker Compose, security, and production best practices. Each unit includes architecture diagrams, hands-on commands, real-world scenarios, summary tables, and revision questions.

### Prerequisites

- Basic Linux command-line knowledge
- Understanding of client-server architecture
- Familiarity with application deployment concepts

### How to Use These Notes

1. Follow the units **sequentially** — each builds on the previous
2. Practice every command in a local Docker installation
3. Use the **summary tables** for quick revision
4. Attempt the **revision questions** before moving to the next chapter
5. Refer to the ASCII diagrams to solidify architectural understanding

---

## Complete Table of Contents

### Unit 1: Container Concepts
| # | Chapter | File |
|---|---------|------|
| 1 | What Are Containers? | [01-what-are-containers.md](01-Container-Concepts/01-what-are-containers.md) |
| 2 | Containers vs Virtual Machines | [02-containers-vs-vms.md](01-Container-Concepts/02-containers-vs-vms.md) |
| 3 | Container History (chroot, LXC) | [03-container-history.md](01-Container-Concepts/03-container-history.md) |
| 4 | OCI Standards | [04-oci-standards.md](01-Container-Concepts/04-oci-standards.md) |
| 5 | Container Use Cases | [05-container-use-cases.md](01-Container-Concepts/05-container-use-cases.md) |

### Unit 2: Docker Architecture
| # | Chapter | File |
|---|---------|------|
| 1 | Docker Engine Components | [01-docker-engine-components.md](02-Docker-Architecture/01-docker-engine-components.md) |
| 2 | Docker Daemon | [02-docker-daemon.md](02-Docker-Architecture/02-docker-daemon.md) |
| 3 | Docker CLI | [03-docker-cli.md](02-Docker-Architecture/03-docker-cli.md) |
| 4 | containerd and runc | [04-containerd-and-runc.md](02-Docker-Architecture/04-containerd-and-runc.md) |
| 5 | Docker Objects | [05-docker-objects.md](02-Docker-Architecture/05-docker-objects.md) |

### Unit 3: Docker Images
| # | Chapter | File |
|---|---------|------|
| 1 | Image Concepts | [01-image-concepts.md](03-Docker-Images/01-image-concepts.md) |
| 2 | Image Layers and Caching | [02-image-layers-and-caching.md](03-Docker-Images/02-image-layers-and-caching.md) |
| 3 | Pulling and Pushing Images | [03-pulling-and-pushing-images.md](03-Docker-Images/03-pulling-and-pushing-images.md) |
| 4 | Image Tagging | [04-image-tagging.md](03-Docker-Images/04-image-tagging.md) |
| 5 | Docker Hub and Registries | [05-docker-hub-and-registries.md](03-Docker-Images/05-docker-hub-and-registries.md) |
| 6 | Image Inspection | [06-image-inspection.md](03-Docker-Images/06-image-inspection.md) |

### Unit 4: Dockerfile
| # | Chapter | File |
|---|---------|------|
| 1 | Dockerfile Instructions | [01-dockerfile-instructions.md](04-Dockerfile/01-dockerfile-instructions.md) |
| 2 | Build Context | [02-build-context.md](04-Dockerfile/02-build-context.md) |
| 3 | Multi-Stage Builds | [03-multi-stage-builds.md](04-Dockerfile/03-multi-stage-builds.md) |
| 4 | .dockerignore | [04-dockerignore.md](04-Dockerfile/04-dockerignore.md) |
| 5 | Build Arguments and Environment Variables | [05-build-args-and-env.md](04-Dockerfile/05-build-args-and-env.md) |
| 6 | Dockerfile Best Practices | [06-dockerfile-best-practices.md](04-Dockerfile/06-dockerfile-best-practices.md) |
| 7 | Image Optimization | [07-image-optimization.md](04-Dockerfile/07-image-optimization.md) |

### Unit 5: Docker Containers
| # | Chapter | File |
|---|---------|------|
| 1 | Container Lifecycle | [01-container-lifecycle.md](05-Docker-Containers/01-container-lifecycle.md) |
| 2 | Running Containers (docker run) | [02-running-containers.md](05-Docker-Containers/02-running-containers.md) |
| 3 | Container Management Commands | [03-container-management.md](05-Docker-Containers/03-container-management.md) |
| 4 | Executing Commands in Containers | [04-executing-commands.md](05-Docker-Containers/04-executing-commands.md) |
| 5 | Container Logs | [05-container-logs.md](05-Docker-Containers/05-container-logs.md) |
| 6 | Resource Limits (CPU, Memory) | [06-resource-limits.md](05-Docker-Containers/06-resource-limits.md) |
| 7 | Health Checks | [07-health-checks.md](05-Docker-Containers/07-health-checks.md) |

### Unit 6: Docker Networking
| # | Chapter | File |
|---|---------|------|
| 1 | Network Drivers | [01-network-drivers.md](06-Docker-Networking/01-network-drivers.md) |
| 2 | Container DNS | [02-container-dns.md](06-Docker-Networking/02-container-dns.md) |
| 3 | Port Mapping | [03-port-mapping.md](06-Docker-Networking/03-port-mapping.md) |
| 4 | Creating Custom Networks | [04-custom-networks.md](06-Docker-Networking/04-custom-networks.md) |
| 5 | Container Communication | [05-container-communication.md](06-Docker-Networking/05-container-communication.md) |
| 6 | Network Troubleshooting | [06-network-troubleshooting.md](06-Docker-Networking/06-network-troubleshooting.md) |

### Unit 7: Docker Volumes
| # | Chapter | File |
|---|---------|------|
| 1 | Data Persistence Concepts | [01-data-persistence.md](07-Docker-Volumes/01-data-persistence.md) |
| 2 | Volume Types | [02-volume-types.md](07-Docker-Volumes/02-volume-types.md) |
| 3 | Volume Management | [03-volume-management.md](07-Docker-Volumes/03-volume-management.md) |
| 4 | Backup and Restore | [04-backup-and-restore.md](07-Docker-Volumes/04-backup-and-restore.md) |
| 5 | Volume Drivers | [05-volume-drivers.md](07-Docker-Volumes/05-volume-drivers.md) |

### Unit 8: Docker Compose
| # | Chapter | File |
|---|---------|------|
| 1 | Compose Concepts | [01-compose-concepts.md](08-Docker-Compose/01-compose-concepts.md) |
| 2 | docker-compose.yml Structure | [02-compose-file-structure.md](08-Docker-Compose/02-compose-file-structure.md) |
| 3 | Services, Networks, Volumes | [03-services-networks-volumes.md](08-Docker-Compose/03-services-networks-volumes.md) |
| 4 | Environment Variables | [04-environment-variables.md](08-Docker-Compose/04-environment-variables.md) |
| 5 | Compose Commands | [05-compose-commands.md](08-Docker-Compose/05-compose-commands.md) |
| 6 | Multi-Environment Setups | [06-multi-environment-setups.md](08-Docker-Compose/06-multi-environment-setups.md) |
| 7 | Compose Best Practices | [07-compose-best-practices.md](08-Docker-Compose/07-compose-best-practices.md) |

### Unit 9: Docker Security
| # | Chapter | File |
|---|---------|------|
| 1 | Security Considerations | [01-security-considerations.md](09-Docker-Security/01-security-considerations.md) |
| 2 | Running as Non-Root | [02-running-as-non-root.md](09-Docker-Security/02-running-as-non-root.md) |
| 3 | Image Scanning | [03-image-scanning.md](09-Docker-Security/03-image-scanning.md) |
| 4 | Secrets Management | [04-secrets-management.md](09-Docker-Security/04-secrets-management.md) |
| 5 | Security Best Practices | [05-security-best-practices.md](09-Docker-Security/05-security-best-practices.md) |
| 6 | Docker Bench Security | [06-docker-bench-security.md](09-Docker-Security/06-docker-bench-security.md) |

### Unit 10: Docker in Production
| # | Chapter | File |
|---|---------|------|
| 1 | Container Orchestration Overview | [01-container-orchestration.md](10-Docker-In-Production/01-container-orchestration.md) |
| 2 | Docker Swarm Basics | [02-docker-swarm-basics.md](10-Docker-In-Production/02-docker-swarm-basics.md) |
| 3 | Logging Strategies | [03-logging-strategies.md](10-Docker-In-Production/03-logging-strategies.md) |
| 4 | Monitoring Containers | [04-monitoring-containers.md](10-Docker-In-Production/04-monitoring-containers.md) |
| 5 | CI/CD Integration | [05-cicd-integration.md](10-Docker-In-Production/05-cicd-integration.md) |
| 6 | Production Best Practices | [06-production-best-practices.md](10-Docker-In-Production/06-production-best-practices.md) |

---

## Quick Reference

### Essential Docker Commands

```bash
# Images
docker pull <image>            # Download an image
docker build -t <name> .       # Build from Dockerfile
docker images                  # List images
docker rmi <image>             # Remove an image

# Containers
docker run <image>             # Create and start container
docker ps                      # List running containers
docker ps -a                   # List all containers
docker stop <container>        # Stop a container
docker rm <container>          # Remove a container
docker exec -it <c> bash       # Shell into container

# Volumes & Networks
docker volume create <name>    # Create a volume
docker network create <name>   # Create a network

# Compose
docker compose up -d           # Start services
docker compose down            # Stop and remove services
docker compose logs            # View service logs
```

### Docker Architecture at a Glance

```
+---------------------------------------------------+
|                   Docker Client                    |
|              (docker CLI / API calls)              |
+---------------------------+-----------------------+
                            |  REST API
+---------------------------v-----------------------+
|                   Docker Daemon                    |
|                    (dockerd)                       |
|  +-------------+  +-----------+  +--------------+ |
|  |   Images    |  | Containers|  |   Networks   | |
|  +-------------+  +-----------+  +--------------+ |
|  +--------------+  +----------------------------+ |
|  |   Volumes    |  |        containerd          | |
|  +--------------+  +-------------+--------------+ |
|                                  |                 |
|                    +-------------v--------------+  |
|                    |           runc             |  |
|                    +----------------------------+  |
+---------------------------------------------------+
|                  Host Operating System             |
|          (Linux Kernel / Namespaces / cgroups)     |
+---------------------------------------------------+
```

---

## Study Tips

1. **Install Docker Desktop** on your machine and practice every command
2. **Build real projects** — containerize a web app, database, and reverse proxy
3. **Read official docs** alongside these notes: [docs.docker.com](https://docs.docker.com)
4. **Draw diagrams** to reinforce architecture concepts
5. **Break things intentionally** — the best way to learn troubleshooting

---

*Happy Containerizing!* 🐳
