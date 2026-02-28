# Kubernetes — Comprehensive Study Notes

```
  _  __     _                          _            
 | |/ /   _| |__   ___ _ __ _ __   ___| |_ ___  ___ 
 | ' / | | | '_ \ / _ \ '__| '_ \ / _ \ __/ _ \/ __|
 | . \ |_| | |_) |  __/ |  | | | |  __/ ||  __/\__ \
 |_|\_\__,_|_.__/ \___|_|  |_| |_|\___|\__\___||___/
```

## Course Overview

Kubernetes (K8s) is the de-facto standard for container orchestration, automating the deployment, scaling, and management of containerized applications. These study notes provide an in-depth, hands-on guide covering everything from foundational concepts to advanced administration — structured for both exam preparation and real-world usage.

**Prerequisites:** Basic understanding of containers (Docker), Linux CLI, YAML syntax, and networking fundamentals.

**What You Will Learn:**

- How Kubernetes works under the hood (architecture, control plane, data plane)
- Deploying and managing applications using Pods, Deployments, Services, and more
- Networking, storage, configuration, and security in Kubernetes
- Observability, autoscaling, Helm, Operators, and GitOps workflows
- Cluster administration, troubleshooting, and production best practices

---

## Complete Table of Contents

### Unit 1: Kubernetes Concepts
| # | Chapter | Link |
|---|---------|------|
| 1.1 | What is Kubernetes? | [Read →](01-Kubernetes-Concepts/01-what-is-kubernetes.md) |
| 1.2 | Why Orchestration? | [Read →](01-Kubernetes-Concepts/01-what-is-kubernetes.md#why-orchestration) |
| 1.3 | Kubernetes Features | [Read →](01-Kubernetes-Concepts/01-what-is-kubernetes.md#kubernetes-features) |
| 1.4 | Kubernetes Distributions | [Read →](01-Kubernetes-Concepts/01-what-is-kubernetes.md#kubernetes-distributions) |
| 1.5 | CNCF Ecosystem | [Read →](01-Kubernetes-Concepts/01-what-is-kubernetes.md#the-cncf-ecosystem) |

### Unit 2: Kubernetes Architecture
| # | Chapter | Link |
|---|---------|------|
| 2.1 | Control Plane Components | [Read →](02-Kubernetes-Architecture/01-architecture-overview.md) |
| 2.2 | Node Components | [Read →](02-Kubernetes-Architecture/01-architecture-overview.md#node-components-data-plane) |
| 2.3 | Add-ons | [Read →](02-Kubernetes-Architecture/01-architecture-overview.md#add-ons) |
| 2.4 | Communication Flow | [Read →](02-Kubernetes-Architecture/01-architecture-overview.md#communication-flow) |

### Unit 3: Pods
| # | Chapter | Link |
|---|---------|------|
| 3.1 | Pod Concepts | [Read →](03-Pods/01-pods.md) |
| 3.2 | Pod Lifecycle | [Read →](03-Pods/01-pods.md#pod-lifecycle) |
| 3.3 | Multi-Container Pods | [Read →](03-Pods/01-pods.md#multi-container-pods) |
| 3.4 | Pod YAML Structure | [Read →](03-Pods/01-pods.md#pod-yaml-structure) |
| 3.5 | Pod Management Commands | [Read →](03-Pods/01-pods.md#pod-management-commands) |
| 3.6 | Static Pods | [Read →](03-Pods/01-pods.md#static-pods) |

### Unit 4: Workload Resources
| # | Chapter | Link |
|---|---------|------|
| 4.1 | ReplicaSets | [Read →](04-Workload-Resources/01-workload-resources.md) |
| 4.2 | Deployments | [Read →](04-Workload-Resources/01-workload-resources.md#deployments) |
| 4.3 | StatefulSets | [Read →](04-Workload-Resources/01-workload-resources.md#statefulsets) |
| 4.4 | DaemonSets | [Read →](04-Workload-Resources/01-workload-resources.md#daemonsets) |
| 4.5 | Jobs and CronJobs | [Read →](04-Workload-Resources/01-workload-resources.md#jobs-and-cronjobs) |
| 4.6 | Choosing the Right Workload | [Read →](04-Workload-Resources/01-workload-resources.md#choosing-the-right-workload-type) |

### Unit 5: Services and Networking
| # | Chapter | Link |
|---|---------|------|
| 5.1 | Service Types | [Read →](05-Services-And-Networking/01-services-and-networking.md) |
| 5.2 | Service Discovery | [Read →](05-Services-And-Networking/01-services-and-networking.md#service-discovery) |
| 5.3 | DNS in Kubernetes | [Read →](05-Services-And-Networking/01-services-and-networking.md#dns-in-kubernetes) |
| 5.4 | Ingress Controllers & Resources | [Read →](05-Services-And-Networking/01-services-and-networking.md#ingress) |
| 5.5 | Network Policies | [Read →](05-Services-And-Networking/01-services-and-networking.md#network-policies) |
| 5.6 | CNI Plugins Overview | [Read →](05-Services-And-Networking/01-services-and-networking.md#cni-plugins-overview) |

### Unit 6: Storage
| # | Chapter | Link |
|---|---------|------|
| 6.1 | Volumes | [Read →](06-Storage/01-storage.md) |
| 6.2 | Persistent Volumes (PV) | [Read →](06-Storage/01-storage.md#persistent-volumes-pv) |
| 6.3 | Persistent Volume Claims (PVC) | [Read →](06-Storage/01-storage.md#persistent-volume-claims-pvc) |
| 6.4 | Storage Classes | [Read →](06-Storage/01-storage.md#storage-classes) |
| 6.5 | Dynamic Provisioning | [Read →](06-Storage/01-storage.md#dynamic-provisioning) |
| 6.6 | CSI Drivers | [Read →](06-Storage/01-storage.md#csi-drivers) |

### Unit 7: Configuration
| # | Chapter | Link |
|---|---------|------|
| 7.1 | ConfigMaps | [Read →](07-Configuration/01-configuration.md) |
| 7.2 | Secrets | [Read →](07-Configuration/01-configuration.md#secrets) |
| 7.3 | Environment Variables | [Read →](07-Configuration/01-configuration.md#environment-variables) |
| 7.4 | Mounting Configs as Volumes | [Read →](07-Configuration/01-configuration.md#mounting-configs-as-volumes) |
| 7.5 | Immutable ConfigMaps/Secrets | [Read →](07-Configuration/01-configuration.md#immutable-configmaps-and-secrets) |

### Unit 8: Scheduling
| # | Chapter | Link |
|---|---------|------|
| 8.1 | Node Selectors | [Read →](08-Scheduling/01-scheduling.md) |
| 8.2 | Node Affinity / Anti-Affinity | [Read →](08-Scheduling/01-scheduling.md#node-affinity-and-anti-affinity) |
| 8.3 | Pod Affinity / Anti-Affinity | [Read →](08-Scheduling/01-scheduling.md#pod-affinity-and-anti-affinity) |
| 8.4 | Taints and Tolerations | [Read →](08-Scheduling/01-scheduling.md#taints-and-tolerations) |
| 8.5 | Resource Requests and Limits | [Read →](08-Scheduling/01-scheduling.md#resource-requests-and-limits) |
| 8.6 | Priority and Preemption | [Read →](08-Scheduling/01-scheduling.md#priority-and-preemption) |

### Unit 9: Security
| # | Chapter | Link |
|---|---------|------|
| 9.1 | RBAC | [Read →](09-Security/01-security.md) |
| 9.2 | Service Accounts | [Read →](09-Security/01-security.md#service-accounts) |
| 9.3 | Pod Security Standards | [Read →](09-Security/01-security.md#pod-security-standards) |
| 9.4 | Security Contexts | [Read →](09-Security/01-security.md#security-contexts) |
| 9.5 | Network Policies (Security) | [Read →](09-Security/01-security.md#network-policies-for-security) |
| 9.6 | Secrets Management | [Read →](09-Security/01-security.md#secrets-management) |
| 9.7 | Image Security | [Read →](09-Security/01-security.md#image-security) |

### Unit 10: Observability
| # | Chapter | Link |
|---|---------|------|
| 10.1 | Logging Architecture | [Read →](10-Observability/01-observability.md) |
| 10.2 | Metrics with Prometheus | [Read →](10-Observability/01-observability.md#metrics-with-prometheus) |
| 10.3 | Monitoring with Grafana | [Read →](10-Observability/01-observability.md#monitoring-with-grafana) |
| 10.4 | Distributed Tracing | [Read →](10-Observability/01-observability.md#distributed-tracing) |
| 10.5 | Health Checks (Probes) | [Read →](10-Observability/01-observability.md#health-checks-probes) |

### Unit 11: Advanced Topics
| # | Chapter | Link |
|---|---------|------|
| 11.1 | Helm Package Manager | [Read →](11-Advanced-Topics/01-advanced-topics.md) |
| 11.2 | Custom Resource Definitions | [Read →](11-Advanced-Topics/01-advanced-topics.md#custom-resource-definitions-crds) |
| 11.3 | Operators | [Read →](11-Advanced-Topics/01-advanced-topics.md#operators) |
| 11.4 | Autoscaling (HPA, VPA, CA) | [Read →](11-Advanced-Topics/01-advanced-topics.md#autoscaling) |
| 11.5 | GitOps with ArgoCD / Flux | [Read →](11-Advanced-Topics/01-advanced-topics.md#gitops) |
| 11.6 | Service Mesh (Istio Basics) | [Read →](11-Advanced-Topics/01-advanced-topics.md#service-mesh-istio-basics) |

### Unit 12: Kubernetes Administration
| # | Chapter | Link |
|---|---------|------|
| 12.1 | Cluster Setup | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md) |
| 12.2 | Cluster Upgrades | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md#cluster-upgrades) |
| 12.3 | Backup and Disaster Recovery | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md#backup-and-disaster-recovery) |
| 12.4 | Troubleshooting | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md#troubleshooting) |
| 12.5 | Performance Tuning | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md#performance-tuning) |
| 12.6 | Multi-Tenancy | [Read →](12-Kubernetes-Administration/01-kubernetes-administration.md#multi-tenancy) |

---

## How to Use These Notes

1. **Sequential Study:** Work through units 1–12 in order for a structured learning path.
2. **Quick Reference:** Jump to any topic via the table of contents above.
3. **Hands-On Practice:** Follow configuration examples using Minikube, kind, or a cloud provider.
4. **Revision:** Use the summary tables and quick revision questions at the end of each chapter.
5. **Exam Prep:** These notes align with the CKA/CKAD exam domains.

---

## Recommended Lab Setup

| Tool | Purpose |
|------|---------|
| **Minikube** | Single-node local cluster for learning |
| **kind** | Kubernetes in Docker — lightweight multi-node clusters |
| **kubeadm** | Production-grade cluster setup |
| **kubectl** | CLI for interacting with clusters |
| **Lens / k9s** | GUI / TUI for cluster management |
| **Helm** | Package manager for Kubernetes |

```bash
# Quick start with Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker --cpus=2 --memory=4096

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl
kubectl version --client
```

---

## Key Resources

- [Official Kubernetes Docs](https://kubernetes.io/docs/)
- [Kubernetes GitHub](https://github.com/kubernetes/kubernetes)
- [CNCF Landscape](https://landscape.cncf.io/)
- [CKA Exam Curriculum](https://github.com/cncf/curriculum)
- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

---

> **Start your journey → [Unit 1: Kubernetes Concepts](01-Kubernetes-Concepts/01-what-is-kubernetes.md)**
