# Chapter 5: Docker Swarm

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Docker Swarm (Swarm mode) is Docker's built-in orchestration engine. It turns a group of Docker hosts into a single virtual host with service scheduling, scaling, rolling updates, and encrypted networking — all managed through familiar Docker CLI commands.

---

## 5.1 Swarm Architecture

```
  Docker Swarm Cluster:
  
  +======================== SWARM ========================+
  |                                                        |
  |  +-- Manager 1 --+  +-- Manager 2 --+ +-- Manager 3 -+|
  |  | Raft Leader    |  | Raft Follower | | Raft Follower ||
  |  | API endpoint   |  | Reachable     | | Reachable     ||
  |  | Scheduler      |  |               | |               ||
  |  | State store    |  |               | |               ||
  |  +-------+--------+  +-------+-------+ +------+-------+|
  |          |                   |                |         |
  |  +=======+===================+================+======+ |
  |  |              Gossip Network (Control)              | |
  |  +====================================================+|
  |          |                   |                |         |
  |  +-- Worker 1 ---+  +-- Worker 2 ---+ +-- Worker 3 --+|
  |  | web.1  api.1  |  | web.2  api.2  | | web.3  api.3 ||
  |  | (tasks)       |  | (tasks)       | | (tasks)      ||
  |  +---------------+  +---------------+ +--------------+|
  +========================================================+
  
  Managers: Maintain cluster state (Raft consensus)
  Workers:  Run container workloads (tasks)
  Both:     Managers can also run tasks
```

---

## 5.2 Initialising a Swarm

```bash
# Initialise swarm on first manager
docker swarm init --advertise-addr 192.168.1.10

# Output provides join tokens
# To add a worker to this swarm:
#   docker swarm join --token SWMTKN-1-abc... 192.168.1.10:2377
# To add a manager to this swarm:
#   docker swarm join-token manager

# On worker nodes - join the swarm
docker swarm join \
  --token SWMTKN-1-abc123... \
  192.168.1.10:2377

# Add another manager (run on existing manager first)
docker swarm join-token manager
# Then run the output command on the new manager node

# View cluster status
docker node ls
# ID            HOSTNAME    STATUS    AVAILABILITY   MANAGER STATUS
# abc123 *      manager1    Ready     Active         Leader
# def456        manager2    Ready     Active         Reachable
# ghi789        worker1     Ready     Active
# jkl012        worker2     Ready     Active
```

```
  Recommended Manager Count:
  
  Nodes  Managers  Fault Tolerance
  -----  --------  ---------------
    3       3       1 manager failure
    5       3       1 manager failure
    7       3       1 manager failure
    5       5       2 manager failures
    7       5       2 manager failures
  
  Rule: (N-1)/2 failures tolerated
  Never use even number of managers!
  (Split-brain risk)
```

---

## 5.3 Services and Tasks

```
  Service → Task → Container:
  
  docker service create --replicas 3 --name web nginx
  
  +-- Service: web --+
  |  Desired: 3      |
  |  Image: nginx    |
  +--------+---------+
           |
     +-----+-----+-----+
     |           |           |
  +--v---+   +--v---+   +--v---+
  |Task 1|   |Task 2|   |Task 3|
  |web.1 |   |web.2 |   |web.3 |
  +--+---+   +--+---+   +--+---+
     |           |           |
  +--v---+   +--v---+   +--v---+
  |nginx |   |nginx |   |nginx |
  |ctnr  |   |ctnr  |   |ctnr  |
  +------+   +------+   +------+
  Worker 1   Worker 2   Worker 3
```

```bash
# Create a service
docker service create \
  --name api \
  --replicas 3 \
  --publish 8080:3000 \
  --env NODE_ENV=production \
  --limit-cpu 1 \
  --limit-memory 512M \
  --restart-condition on-failure \
  --restart-max-attempts 3 \
  myregistry/api:v1.0

# List services
docker service ls
# ID      NAME   MODE         REPLICAS   IMAGE
# abc123  api    replicated   3/3        myregistry/api:v1.0

# Inspect service details
docker service inspect --pretty api

# View service tasks (containers)
docker service ps api
# ID      NAME    NODE      STATE     
# t1      api.1   worker1   Running
# t2      api.2   worker2   Running
# t3      api.3   worker3   Running

# View service logs (aggregated from all tasks)
docker service logs --follow api

# Scale a service
docker service scale api=5

# Remove a service
docker service rm api
```

---

## 5.4 Routing Mesh and Load Balancing

```
  Swarm Routing Mesh:
  
  Client Request → port 8080
         |
  +------v------+------+------+
  | ANY node in the swarm      |
  | (even nodes without the    |
  |  service running)          |
  +------+------+------+------+
         |
  +------v------+
  | Ingress     |
  | Network     |
  | (IPVS L4   |
  |  load       |
  |  balancer)  |
  +------+------+
         |
    +----+----+----+
    |         |         |
  +-v---+  +--v--+  +--v--+
  |api.1|  |api.2|  |api.3|
  +-----+  +-----+  +-----+
  
  Key: Publish a port → accessible on ALL nodes
  Built-in round-robin load balancing
```

```bash
# Published port is available on every swarm node
# Even if api only runs on worker1 and worker2,
# hitting worker3:8080 routes to an api container

# Mode options
--publish mode=ingress,target=3000,published=8080  # routing mesh (default)
--publish mode=host,target=3000,published=8080     # host-only (no mesh)
```

---

## 5.5 Rolling Updates

```bash
# Update service image
docker service update \
  --image myregistry/api:v1.1 \
  --update-parallelism 1 \
  --update-delay 10s \
  --update-failure-action rollback \
  --update-order start-first \
  api

# Monitor the rolling update
docker service ps api
# NAME      NODE      DESIRED STATE  CURRENT STATE
# api.1     worker1   Running        Running 5s
# \_ api.1  worker1   Shutdown       Shutdown 10s    ← old
# api.2     worker2   Running        Running 30s     ← already updated
# api.3     worker3   Running        Running 1m

# Rollback to previous version
docker service rollback api
```

```
  Rolling Update Sequence (parallelism=1, order=start-first):
  
  Time ──────────────────────────────>
  
  api.1: [  v1.0 running  ][v1.1 start][  v1.1 running  ]
                              ↑ new starts before old stops
  api.2: [     v1.0 running     ][v1.1][  v1.1 running  ]
                                     delay
  api.3: [         v1.0 running       ][v1.1][ v1.1 run ]
  
  Zero-downtime: always N or N+1 instances running
```

---

## 5.6 Stacks (Compose in Swarm)

```yaml
# docker-stack.yml
version: "3.8"

services:
  web:
    image: myregistry/web:v1.0
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 1
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
    ports:
      - "80:3000"
    networks:
      - frontend
      - backend

  api:
    image: myregistry/api:v1.0
    deploy:
      replicas: 2
    networks:
      - backend
    secrets:
      - db_password

  db:
    image: postgres:16-alpine
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.storage == ssd
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    secrets:
      - db_password

networks:
  frontend:
  backend:
    internal: true

volumes:
  db_data:

secrets:
  db_password:
    external: true
```

```bash
# Deploy a stack
docker stack deploy -c docker-stack.yml myapp

# List stacks
docker stack ls
# NAME    SERVICES
# myapp   3

# List services in a stack
docker stack services myapp

# List tasks in a stack
docker stack ps myapp

# Update a stack (redeploy with updated file)
docker stack deploy -c docker-stack.yml myapp

# Remove a stack
docker stack rm myapp
```

---

## 5.7 Swarm Secrets

```bash
# Create a secret
echo "supersecretpassword" | docker secret create db_password -

# Create from file
docker secret create tls_cert ./server.crt

# List secrets
docker secret ls
# ID          NAME          CREATED
# abc123      db_password   2 hours ago

# Use secret in a service
docker service create \
  --name api \
  --secret db_password \
  myregistry/api:v1.0

# Inside the container, secret is at:
# /run/secrets/db_password (as a file, in-memory tmpfs)

# Rotate a secret
echo "newpassword" | docker secret create db_password_v2 -
docker service update \
  --secret-rm db_password \
  --secret-add db_password_v2 \
  api
```

```
  Secret Distribution:
  
  +-- Manager --+
  | Raft Store  |  Encrypted at rest
  | (encrypted) |  (AES-256-GCM)
  +------+------+
         |
         | TLS encrypted channel
         |
  +------v------+
  | Worker Node |
  | tmpfs mount |  In-memory only
  | /run/secrets|  Never written to disk
  +-------------+
  
  Secrets never leave the encrypted Raft log
  on managers except to the specific worker
  running a task that needs them
```

---

## 5.8 Node Management

```bash
# Promote worker to manager
docker node promote worker3

# Demote manager to worker
docker node demote manager3

# Drain a node (for maintenance)
docker node update --availability drain worker1
# All tasks moved to other nodes

# Return node to active
docker node update --availability active worker1

# Add labels for placement
docker node update --label-add zone=eu-west worker1
docker node update --label-add storage=ssd worker2

# Remove a node
docker node rm worker3    # must be drained first

# Leave swarm (run on the node)
docker swarm leave         # worker leaves
docker swarm leave --force # manager leaves
```

---

## Summary Table

| Feature | Command / Config |
|---------|-----------------|
| **Init swarm** | `docker swarm init` |
| **Join node** | `docker swarm join --token TOKEN IP:2377` |
| **Create service** | `docker service create --replicas N IMAGE` |
| **Scale** | `docker service scale NAME=N` |
| **Rolling update** | `docker service update --image NEW` |
| **Rollback** | `docker service rollback NAME` |
| **Deploy stack** | `docker stack deploy -c FILE NAME` |
| **Create secret** | `docker secret create NAME FILE` |
| **Drain node** | `docker node update --availability drain` |
| **View tasks** | `docker service ps NAME` |
| **Aggregated logs** | `docker service logs NAME` |

---

## Quick Revision Questions

1. **What is the difference between a manager and a worker node in Docker Swarm?**
   - Managers maintain cluster state via Raft consensus, run the scheduler, and serve the API. Workers execute container tasks. Managers can also run tasks unless drained

2. **How does the routing mesh work?**
   - When you publish a port, it's available on every swarm node. Any node receiving a request on that port routes it via the ingress network (IPVS) to a container running the service, even if that container is on a different node

3. **What does `--update-order start-first` achieve during rolling updates?**
   - It starts the new version of a task before stopping the old one. This ensures there are always N (or N+1) instances running, achieving zero-downtime deployments

4. **How are secrets stored and distributed in Swarm?**
   - Secrets are encrypted in the Raft log (AES-256-GCM) on manager nodes. They're sent over TLS only to workers running tasks that need them, and mounted as in-memory tmpfs files at `/run/secrets/`

5. **What is the purpose of draining a node?**
   - Draining gracefully moves all tasks off a node to other available nodes. Used for maintenance (OS updates, hardware changes) without service interruption

6. **What is a stack and how does it relate to Docker Compose?**
   - A stack is a Swarm deployment from a Compose file. It uses the same `docker-compose.yml` format with the added `deploy` key for Swarm-specific settings like replicas, update strategy, and placement constraints

---

[← Previous: Container Orchestration](04-container-orchestration.md) | [Next: Docker and Kubernetes →](06-docker-and-kubernetes.md)
