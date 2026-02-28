# Chapter 6: Docker and Kubernetes

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Kubernetes (K8s) is the industry-standard container orchestration platform. While it supports multiple container runtimes, Docker images are the most common workload format. This chapter covers Kubernetes fundamentals from a Docker user's perspective and how to transition from Docker workflows to Kubernetes.

---

## 6.1 Kubernetes Architecture

```
  Kubernetes Cluster:
  
  +================ Control Plane ==================+
  |                                                   |
  |  +----------+  +----------+  +----------------+  |
  |  | API      |  | Scheduler|  | Controller     |  |
  |  | Server   |  |          |  | Manager        |  |
  |  | (kube-   |  | (places  |  | (reconciles    |  |
  |  |  apiserver)|  pods)    |  |  desired state) |  |
  |  +----------+  +----------+  +----------------+  |
  |                                                   |
  |  +------------+                                   |
  |  | etcd       |  ← Distributed key-value store    |
  |  | (cluster   |    Holds all cluster state         |
  |  |  state)    |                                    |
  |  +------------+                                   |
  +===================================================+
           |                    |
  +--------v--------+  +-------v---------+
  |   Worker Node 1  |  |   Worker Node 2  |
  |                   |  |                   |
  |  +-----+ +-----+ |  |  +-----+ +-----+ |
  |  | Pod | | Pod | |  |  | Pod | | Pod | |
  |  | web | | api | |  |  | web | | api | |
  |  +-----+ +-----+ |  |  +-----+ +-----+ |
  |                   |  |                   |
  |  kubelet          |  |  kubelet          |
  |  kube-proxy       |  |  kube-proxy       |
  |  container runtime|  |  container runtime|
  +-------------------+  +-------------------+
```

---

## 6.2 Docker Concepts → Kubernetes Concepts

```
  Mapping Docker to Kubernetes:
  
  Docker                          Kubernetes
  +-----------+                   +-----------+
  | Container | ───────────────── | Container |  (inside a Pod)
  +-----------+                   +-----------+
  
  +-----------+                   +-----------+
  | docker run| ───────────────── |    Pod    |  (1+ containers)
  +-----------+                   +-----------+
  
  +-----------+                   +------------+
  | Compose   | ───────────────── | Deployment |  (manages Pods)
  | service   |                   +------------+
  +-----------+
  
  +-----------+                   +-----------+
  | -p 8080:80| ───────────────── |  Service  |  (networking)
  +-----------+                   +-----------+
  
  +-----------+                   +-----------+
  | volume    | ───────────────── |    PVC    |  (storage claim)
  +-----------+                   +-----------+
  
  +-----------+                   +-----------+
  | .env file | ───────────────── | ConfigMap |  (config data)
  +-----------+                   +-----------+
  
  +-----------+                   +-----------+
  | secret    | ───────────────── |  Secret   |  (sensitive data)
  +-----------+                   +-----------+
```

| Docker Command | Kubernetes Equivalent |
|---------------|----------------------|
| `docker run nginx` | `kubectl run nginx --image=nginx` |
| `docker ps` | `kubectl get pods` |
| `docker logs` | `kubectl logs POD` |
| `docker exec -it` | `kubectl exec -it POD -- bash` |
| `docker inspect` | `kubectl describe pod POD` |
| `docker rm` | `kubectl delete pod POD` |
| `docker compose up` | `kubectl apply -f manifests/` |
| `docker compose down` | `kubectl delete -f manifests/` |
| `docker images` | `kubectl get images` (via registry) |

---

## 6.3 Pods — The Basic Unit

```yaml
# pod.yml — equivalent to docker run
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
    - name: nginx
      image: nginx:1.25-alpine
      ports:
        - containerPort: 80
      resources:
        limits:
          cpu: "500m"        # 0.5 CPU cores
          memory: "128Mi"    # 128 MB
        requests:
          cpu: "100m"
          memory: "64Mi"
      livenessProbe:         # like Docker HEALTHCHECK
        httpGet:
          path: /health
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 30
      readinessProbe:        # K8s-specific: controls traffic routing
        httpGet:
          path: /ready
          port: 80
        periodSeconds: 5
```

```bash
# Create from YAML
kubectl apply -f pod.yml

# List pods
kubectl get pods
# NAME   READY   STATUS    RESTARTS   AGE
# web    1/1     Running   0          30s

# Describe (detailed info)
kubectl describe pod web

# Logs
kubectl logs web
kubectl logs web --follow

# Execute command
kubectl exec -it web -- /bin/sh

# Delete
kubectl delete pod web
```

---

## 6.4 Deployments — Managing Replicas

```yaml
# deployment.yml — equivalent to docker service create --replicas
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # At most 1 pod down during update
      maxSurge: 1            # At most 1 extra pod during update
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myregistry/api:v1.0
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: password
          resources:
            limits:
              cpu: "1"
              memory: "512Mi"
```

```bash
# Apply deployment
kubectl apply -f deployment.yml

# Check rollout status
kubectl rollout status deployment/api

# Scale
kubectl scale deployment/api --replicas=5

# Update image (triggers rolling update)
kubectl set image deployment/api api=myregistry/api:v1.1

# View rollout history
kubectl rollout history deployment/api

# Rollback
kubectl rollout undo deployment/api
kubectl rollout undo deployment/api --to-revision=2
```

```
  Deployment manages ReplicaSet manages Pods:
  
  +-- Deployment: api ---+
  |  replicas: 3         |
  |  strategy: Rolling   |
  +----------+-----------+
             |
  +----------v-----------+
  |  ReplicaSet: api-abc |
  |  replicas: 3         |
  +----+------+------+---+
       |      |      |
  +----v+ +---v-+ +--v--+
  |Pod 1| |Pod 2| |Pod 3|
  |api  | |api  | |api  |
  +-----+ +-----+ +-----+
```

---

## 6.5 Services — Networking and Discovery

```yaml
# service.yml — equivalent to docker service publish
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api          # Routes to pods with label app=api
  type: ClusterIP     # Internal only (default)
  ports:
    - port: 80        # Service port
      targetPort: 3000 # Container port
---
# LoadBalancer service — exposes externally
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  type: LoadBalancer   # Cloud provider creates LB
  ports:
    - port: 80
      targetPort: 3000
```

```
  Kubernetes Service Types:
  
  +==================+==================+===================+
  |    ClusterIP     |    NodePort      |   LoadBalancer     |
  +==================+==================+===================+
  | Internal only    | ClusterIP +      | NodePort +         |
  |                  | port on each node| cloud load balancer|
  +------------------+------------------+-------------------+
  |  api:80          | api:80           | api:80             |
  | (cluster only)   | NodeIP:30080     | LB_IP:80           |
  +------------------+------------------+-------------------+
  
  DNS: Pods can reach services by name
  api.default.svc.cluster.local → ClusterIP of api service
  (or just "api" within same namespace)
```

---

## 6.6 ConfigMaps and Secrets

```yaml
# configmap.yml — equivalent to .env files
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  config.json: |
    {
      "port": 3000,
      "debug": false
    }
---
# secret.yml — equivalent to Docker secrets
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
stringData:
  password: supersecret
  connection-string: "postgres://app:secret@db:5432/mydb"
```

```yaml
# Using ConfigMap and Secret in a Deployment
spec:
  containers:
    - name: api
      image: myregistry/api:v1.0
      envFrom:
        - configMapRef:
            name: app-config       # All keys as env vars
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: password
      volumeMounts:
        - name: config-volume
          mountPath: /app/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

---

## 6.7 Converting Docker Compose to Kubernetes

```
  Docker Compose          Kubernetes Manifests
  
  docker-compose.yml  →   deployment.yml  (per service)
                          service.yml     (per exposed port)
                          configmap.yml   (env vars)
                          secret.yml      (secrets)
                          pvc.yml         (volumes)
                          ingress.yml     (routing rules)
```

```yaml
# Docker Compose
services:
  web:
    image: myapp:v1.0
    ports:
      - "80:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - data:/app/data
    deploy:
      replicas: 3

# Equivalent Kubernetes (3 files)
# --- deployment.yml ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: myapp:v1.0
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
          volumeMounts:
            - name: data
              mountPath: /app/data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: web-data
---
# --- service.yml ---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 3000
---
# --- pvc.yml ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

```bash
# Tools for conversion
# Kompose — auto-convert compose to K8s
kompose convert -f docker-compose.yml
# Generates deployment, service, and PVC files

# Apply all manifests in a directory
kubectl apply -f k8s/

# Delete all
kubectl delete -f k8s/
```

---

## 6.8 When to Move from Docker to Kubernetes

```
  Migration Decision:
  
  Stay with Docker Compose/Swarm if:
  +-----------------------------------------+
  | • Single host or small cluster (< 5)    |
  | • Small team without K8s experience     |
  | • Simple stateless applications         |
  | • Budget constraints (K8s ops cost)     |
  | • Fast time-to-market priority          |
  +-----------------------------------------+
  
  Move to Kubernetes if:
  +-----------------------------------------+
  | • Multi-team, many microservices        |
  | • Need auto-scaling (HPA/VPA)          |
  | • Multi-cloud or hybrid cloud          |
  | • Advanced deployment (canary, A/B)    |
  | • Service mesh requirements (Istio)    |
  | • Compliance needs (RBAC, policies)    |
  | • Large scale (50+ containers)         |
  +-----------------------------------------+
  
  Consider managed K8s to reduce ops burden:
  EKS (AWS) | AKS (Azure) | GKE (Google)
```

---

## Summary Table

| Docker Concept | Kubernetes Equivalent | Purpose |
|---------------|----------------------|---------|
| `docker run` | Pod | Run a container |
| `docker service` | Deployment | Manage replicated containers |
| `-p 8080:3000` | Service (LoadBalancer) | Expose networking |
| `volume` | PersistentVolumeClaim | Persistent storage |
| `.env` file | ConfigMap | Configuration data |
| `docker secret` | Secret | Sensitive data |
| `docker-compose.yml` | Multiple YAML manifests | Full app definition |
| `docker service update` | `kubectl set image` | Rolling update |
| `docker service rollback` | `kubectl rollout undo` | Rollback |
| `docker service scale` | `kubectl scale` or HPA | Scaling |
| `docker network` | NetworkPolicy + CNI | Network isolation |
| `docker stack deploy` | `kubectl apply -f dir/` | Deploy full stack |

---

## Quick Revision Questions

1. **What are the main components of the Kubernetes control plane?**
   - API Server (all communication), Scheduler (places pods on nodes), Controller Manager (reconciles desired vs actual state), and etcd (distributed key-value store for all cluster state)

2. **How does a Kubernetes Deployment differ from a Docker service?**
   - Both manage replicated containers with rolling updates. Kubernetes Deployments manage ReplicaSets (for revision history), support more advanced strategies, and integrate with the broader K8s ecosystem (HPA, RBAC, NetworkPolicy)

3. **What is the difference between a liveness probe and a readiness probe?**
   - Liveness checks if the container is alive — failure causes a restart (like Docker HEALTHCHECK). Readiness checks if the container can accept traffic — failure removes it from the Service endpoints without restarting

4. **How do Kubernetes Services provide load balancing?**
   - A Service has a stable ClusterIP. `kube-proxy` on each node sets up iptables/IPVS rules that distribute traffic to healthy pods matching the Service's label selector. DNS resolves the service name to the ClusterIP

5. **What tool can convert Docker Compose files to Kubernetes manifests?**
   - Kompose (`kompose convert`) generates Deployments, Services, and PVCs from a `docker-compose.yml`. Results often need manual refinement but provide a solid starting point

6. **When should you stay with Docker Compose/Swarm instead of adopting Kubernetes?**
   - Small teams, few services, single host or small cluster, budget constraints, and when simplicity and fast deployment outweigh the operational overhead of running Kubernetes

---

[← Previous: Docker Swarm](05-docker-swarm.md) | [Course Home](../README.md)
