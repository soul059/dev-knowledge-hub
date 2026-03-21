# Unit 8: Container and Serverless Security — Topic 2: Kubernetes Security

## Overview

**Kubernetes (K8s) security** addresses the unique risks of container orchestration platforms. Kubernetes manages container deployment, scaling, and networking, but introduces significant attack surface through its API server, etcd datastore, and complex RBAC system. Securing Kubernetes clusters in cloud environments (EKS, AKS, GKE) requires understanding both Kubernetes-native and cloud-specific security controls.

---

## 1. Kubernetes Attack Surface

```
K8S ATTACK SURFACE:

  ┌──────────────────────────────────────┐
  │         KUBERNETES CLUSTER           │
  │                                      │
  │  CONTROL PLANE:                      │
  │  ┌──────────┐ ┌──────┐ ┌─────────┐ │
  │  │API Server│ │ etcd │ │Scheduler│ │
  │  └────┬─────┘ └──────┘ └─────────┘ │
  │       │                             │
  │  WORKER NODES:                      │
  │  ┌──────────┐ ┌──────────┐         │
  │  │ kubelet  │ │ kube-proxy│         │
  │  │ ┌─────┐  │ │          │         │
  │  │ │Pods │  │ │          │         │
  │  │ └─────┘  │ │          │         │
  │  └──────────┘ └──────────┘         │
  └──────────────────────────────────────┘

ATTACK VECTORS:
  1. Exposed API server (unauthenticated access)
  2. Compromised service account tokens
  3. Container escape to node
  4. Lateral movement between pods
  5. etcd direct access (all secrets stored)
  6. Supply chain (malicious images)
  7. Privilege escalation (RBAC misconfig)
  8. Kubelet API exploitation
  9. Cloud metadata service access from pods
  10. Secrets in environment variables

COMMON ATTACKS:
  → Unauthenticated kubectl access
  → Service account token theft
  → Pod-to-pod lateral movement
  → Host path volume mount escape
  → Privileged container escape
  → IMDS access for cloud credentials
  → etcd data extraction
  → Crypto mining in compromised pods
```

---

## 2. Kubernetes RBAC

```
K8S RBAC:

COMPONENTS:
  → Role: Permissions within a namespace
  → ClusterRole: Cluster-wide permissions
  → RoleBinding: Binds role to user/SA in namespace
  → ClusterRoleBinding: Binds cluster role cluster-wide

DANGEROUS PERMISSIONS:
  → create pods (can mount host paths)
  → create pods/exec (can exec into pods)
  → get secrets (read all secrets)
  → create clusterrolebindings (escalate privileges)
  → impersonate users
  → wildcards on resources or verbs

EXAMPLE SECURE ROLE:
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: production
    name: app-deployer
  rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "update", "patch"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get"]

RBAC BEST PRACTICES:
  [ ] No cluster-admin for regular users
  [ ] Namespace-scoped roles (not cluster)
  [ ] No wildcard permissions (* on verbs/resources)
  [ ] Service accounts per workload
  [ ] Disable auto-mounting SA tokens
  [ ] Regular RBAC audits
  [ ] Use Groups for binding (not individual users)

AUDIT:
  # Find cluster-admin bindings
  kubectl get clusterrolebindings -o json | jq '.items[] | select(.roleRef.name=="cluster-admin") | .subjects'
  
  # List all roles and bindings
  kubectl get roles,rolebindings --all-namespaces
  kubectl get clusterroles,clusterrolebindings
  
  # Check who can do what
  kubectl auth can-i --list --as=system:serviceaccount:default:default
```

---

## 3. Network Policies

```
KUBERNETES NETWORK POLICIES:

DEFAULT BEHAVIOR:
  → ALL pods can communicate with ALL pods
  → No network isolation by default
  → Must explicitly create NetworkPolicies

DEFAULT DENY ALL:
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: default-deny-all
    namespace: production
  spec:
    podSelector: {}
    policyTypes:
    - Ingress
    - Egress

ALLOW SPECIFIC TRAFFIC:
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-frontend-to-api
    namespace: production
  spec:
    podSelector:
      matchLabels:
        app: api
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: frontend
      ports:
      - protocol: TCP
        port: 8080
    policyTypes:
    - Ingress

ALLOW DNS (REQUIRED):
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-dns
    namespace: production
  spec:
    podSelector: {}
    egress:
    - to:
      - namespaceSelector: {}
      ports:
      - protocol: UDP
        port: 53
      - protocol: TCP
        port: 53
    policyTypes:
    - Egress

CLOUD PROVIDER NETWORK PLUGINS:
  AWS EKS: Calico, VPC CNI
  Azure AKS: Azure CNI, Calico
  GCP GKE: Calico, Dataplane V2 (Cilium)
```

---

## 4. Secrets and Admission Control

```
SECRETS MANAGEMENT:

K8S SECRETS (BUILT-IN):
  → Base64 encoded (NOT encrypted by default!)
  → Stored in etcd
  → Enable encryption at rest for etcd
  → Use external secrets managers instead

EXTERNAL SECRETS:
  → AWS Secrets Manager + External Secrets Operator
  → Azure Key Vault + CSI Secrets Store Driver
  → GCP Secret Manager + External Secrets
  → HashiCorp Vault + Vault Agent Injector
  
  # External Secrets Operator
  apiVersion: external-secrets.io/v1beta1
  kind: ExternalSecret
  metadata:
    name: db-credentials
  spec:
    refreshInterval: 1h
    secretStoreRef:
      name: aws-sm
      kind: SecretStore
    target:
      name: db-credentials
    data:
    - secretKey: password
      remoteRef:
        key: prod/db/password

ADMISSION CONTROL:

  Pod Security Standards (PSS):
  → Privileged: Unrestricted (avoid!)
  → Baseline: Minimally restrictive
  → Restricted: Heavily restricted (recommended)
  
  # Enforce restricted standard
  kubectl label namespace production \
    pod-security.kubernetes.io/enforce=restricted

  OPA Gatekeeper / Kyverno:
  → Custom admission policies
  → Deny privileged containers
  → Require resource limits
  → Enforce image registry
  → Require labels/annotations
  → Block host networking

  Binary Authorization (GCP):
  → Only deploy signed images
  → Attestation-based verification
  → Break-glass for emergencies
```

---

## 5. Assessment

```
KUBERNETES SECURITY ASSESSMENT:

TOOLS:
  kube-bench:
  → CIS Kubernetes Benchmark
  → Checks control plane and node config
  → Automated compliance checking
  
  # Run kube-bench
  kubectl apply -f kube-bench-job.yaml
  kubectl logs kube-bench-xxxxx

  kube-hunter:
  → Penetration testing for K8s
  → Discovers attack surface
  → Tests for known vulnerabilities
  
  kubeaudit:
  → Audit K8s cluster security
  → Check pod security settings
  → Identify misconfigurations

  Trivy:
  → Image vulnerability scanning
  → K8s manifest scanning
  → IaC scanning

CHECKLIST:
  [ ] API server not publicly exposed
  [ ] RBAC enabled and properly configured
  [ ] Network policies enforce segmentation
  [ ] Pod security standards enforced
  [ ] Secrets encrypted at rest (etcd)
  [ ] External secrets manager used
  [ ] Image scanning before deployment
  [ ] Admission controllers configured
  [ ] Audit logging enabled
  [ ] Node pools hardened
  [ ] Auto-upgrade enabled
  [ ] IMDS access restricted from pods

COMMON FINDINGS:
  Finding                          | Risk
  API server publicly accessible   | Critical
  cluster-admin bound broadly      | Critical
  No network policies              | High
  Secrets not encrypted in etcd    | High
  Privileged pods allowed          | High
  No admission control             | Medium
  IMDS accessible from pods        | High
  No image scanning                | Medium
  Audit logging disabled           | High
```

---

## Summary Table

| Security Area | Control | Tools |
|--------------|---------|-------|
| RBAC | Least privilege roles | kubectl, rback |
| Network | Network Policies | Calico, Cilium |
| Pods | Security Standards | PSS, OPA |
| Secrets | External manager | Vault, ESO |
| Images | Scanning + signing | Trivy, Cosign |
| Audit | Logging + benchmark | kube-bench, Falco |

---

## Revision Questions

1. What are the most dangerous Kubernetes RBAC permissions?
2. Why is default deny important for network policies?
3. How should secrets be managed in Kubernetes?
4. What is admission control and what policies should it enforce?
5. What tools are used for Kubernetes security assessment?

---

*Previous: [01-container-security-in-cloud.md](01-container-security-in-cloud.md) | Next: [03-serverless-security.md](03-serverless-security.md)*

---

*[Back to README](../README.md)*
