# Chapter 7.1: Kubernetes Cluster Hardening

## Overview

A Kubernetes cluster has a large attack surface — API server, etcd, kubelet, networking, and workloads. Hardening involves securing each layer: API access, node configuration, network controls, and audit logging. This chapter covers CIS Kubernetes Benchmark recommendations and practical hardening steps.

---

## Kubernetes Attack Surface

```
KUBERNETES CLUSTER ATTACK SURFACE:

  ┌──────────────────────────────────────────────────────────┐
  │                    CONTROL PLANE                          │
  │                                                           │
  │  ┌──────────┐  ┌───────┐  ┌────────────┐  ┌───────────┐ │
  │  │ API      │  │ etcd  │  │ Controller │  │ Scheduler │ │
  │  │ Server   │  │       │  │ Manager    │  │           │ │
  │  │ ○ Auth   │  │ ○ Enc │  │            │  │           │ │
  │  │ ○ RBAC   │  │ ○ TLS │  │            │  │           │ │
  │  │ ○ Audit  │  │ ○ ACL │  │            │  │           │ │
  │  └──────────┘  └───────┘  └────────────┘  └───────────┘ │
  │          │                                                │
  └──────────┼────────────────────────────────────────────────┘
             │ TLS
  ┌──────────┼────────────────────────────────────────────────┐
  │          ▼           WORKER NODES                          │
  │  ┌──────────┐  ┌──────────────────────────────────┐       │
  │  │ Kubelet  │  │ Pods / Containers                 │       │
  │  │ ○ Auth   │  │ ┌─────┐ ┌─────┐ ┌─────┐         │       │
  │  │ ○ TLS    │  │ │Pod A│ │Pod B│ │Pod C│         │       │
  │  │ ○ RO API │  │ └─────┘ └─────┘ └─────┘         │       │
  │  └──────────┘  │ ○ Network Policies                │       │
  │                │ ○ Pod Security Standards           │       │
  │                │ ○ Service Mesh                     │       │
  │                └──────────────────────────────────────┘    │
  └──────────────────────────────────────────────────────────┘
```

---

## API Server Hardening

```yaml
# kube-apiserver security flags

# Authentication
--anonymous-auth=false                        # Disable anonymous access
--oidc-issuer-url=https://accounts.google.com # OIDC authentication
--oidc-client-id=kubernetes
--oidc-username-claim=email

# Authorization
--authorization-mode=RBAC,Node               # RBAC + Node authorization
--enable-admission-plugins=\
    NodeRestriction,\
    PodSecurity,\
    AlwaysPullImages,\
    ServiceAccount

# Audit logging
--audit-log-path=/var/log/kubernetes/audit.log
--audit-log-maxage=30
--audit-log-maxbackup=10
--audit-log-maxsize=100
--audit-policy-file=/etc/kubernetes/audit-policy.yaml

# TLS
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key
--client-ca-file=/etc/kubernetes/pki/ca.crt
--tls-min-version=VersionTLS12

# etcd communication
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key

# Misc
--profiling=false                             # Disable profiling endpoint
--insecure-port=0                             # No HTTP (deprecated, disabled)
```

---

## etcd Security

```bash
# etcd should be:
# 1. Encrypted at rest
# 2. TLS-encrypted in transit
# 3. Accessible only by API server

# Encryption configuration
cat <<EOF > /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: $(head -c 32 /dev/urandom | base64)
      - identity: {}    # Fallback for reading unencrypted data
EOF

# Apply to API server
# --encryption-provider-config=/etc/kubernetes/encryption-config.yaml

# Verify secrets are encrypted
kubectl get secrets -A -o json | kubectl replace -f -
ETCDCTL_API=3 etcdctl get /registry/secrets/default/my-secret \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/peer.crt \
    --key=/etc/kubernetes/pki/etcd/peer.key
# Output should be encrypted (not plaintext)
```

---

## Kubelet Hardening

```yaml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration

# Authentication
authentication:
  anonymous:
    enabled: false           # No anonymous access
  webhook:
    enabled: true            # Use API server for auth
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt

# Authorization
authorization:
  mode: Webhook              # Use API server RBAC

# TLS
tlsCertFile: /var/lib/kubelet/pki/kubelet.crt
tlsPrivateKeyFile: /var/lib/kubelet/pki/kubelet.key
tlsCipherSuites:
  - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

# Security
readOnlyPort: 0              # Disable read-only port (10255)
protectKernelDefaults: true  # Protect kernel tunable
makeIPTablesUtilChains: true
eventRecordQPS: 5

# Resource management
evictionHard:
  memory.available: "200Mi"
  nodefs.available: "10%"
```

---

## Audit Policy

```yaml
# /etc/kubernetes/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Don't log requests to certain non-resource URL paths
  - level: None
    nonResourceURLs:
      - /healthz*
      - /readyz*
      - /livez*

  # Don't log watch requests
  - level: None
    verbs: ["watch", "list"]

  # Log Secret access at Metadata level
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]

  # Log pod exec/attach at RequestResponse level
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["pods/exec", "pods/attach", "pods/portforward"]

  # Log RBAC changes
  - level: RequestResponse
    resources:
      - group: "rbac.authorization.k8s.io"
        resources: ["clusterroles", "clusterrolebindings", "roles", "rolebindings"]

  # Log all other resources at Metadata level
  - level: Metadata
    resources:
      - group: ""
      - group: "apps"
      - group: "batch"
```

---

## CIS Kubernetes Benchmark — Key Controls

```
CIS BENCHMARK CATEGORIES:

  ┌────────────────────────────────────────────────────────┐
  │ 1. Control Plane Components                            │
  │    ├─ 1.1 API Server (22 checks)                      │
  │    ├─ 1.2 Controller Manager (7 checks)                │
  │    ├─ 1.3 Scheduler (2 checks)                         │
  │    └─ 1.4 etcd (7 checks)                              │
  │                                                         │
  │ 2. Worker Node Security                                 │
  │    ├─ 2.1 Kubelet (14 checks)                          │
  │    └─ 2.2 Configuration Files (8 checks)               │
  │                                                         │
  │ 3. Policies                                             │
  │    ├─ 3.1 RBAC and Service Accounts                    │
  │    ├─ 3.2 Pod Security Standards                        │
  │    ├─ 3.3 Network Policies                              │
  │    └─ 3.4 Secrets Management                           │
  │                                                         │
  │ 4. Managed Kubernetes (EKS/AKS/GKE)                    │
  │    ├─ 4.1 Control Plane (managed by provider)          │
  │    └─ 4.2 Worker Nodes (your responsibility)           │
  └────────────────────────────────────────────────────────┘
```

### Running kube-bench

```bash
# kube-bench checks CIS Kubernetes Benchmark compliance
# Run as a Job in your cluster
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: kube-bench
  namespace: kube-system
spec:
  template:
    spec:
      hostPID: true
      containers:
        - name: kube-bench
          image: aquasec/kube-bench:latest
          command: ["kube-bench"]
          volumeMounts:
            - name: var-lib-kubelet
              mountPath: /var/lib/kubelet
              readOnly: true
            - name: etc-kubernetes
              mountPath: /etc/kubernetes
              readOnly: true
      restartPolicy: Never
      volumes:
        - name: var-lib-kubelet
          hostPath:
            path: /var/lib/kubelet
        - name: etc-kubernetes
          hostPath:
            path: /etc/kubernetes
  backoffLimit: 0
EOF

# Check results
kubectl logs job/kube-bench -n kube-system

# Run for specific sections
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro \
    aquasec/kube-bench run --targets=master
```

---

## Managed Kubernetes Hardening (EKS)

```bash
# EKS-specific hardening

# 1. Enable control plane logging
aws eks update-cluster-config \
    --name my-cluster \
    --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enabled":true}]}'

# 2. Restrict public endpoint access
aws eks update-cluster-config \
    --name my-cluster \
    --resources-vpc-config \
        endpointPublicAccess=true,\
        publicAccessCidrs="10.0.0.0/8",\
        endpointPrivateAccess=true

# 3. Enable secrets encryption
aws eks create-cluster \
    --name my-cluster \
    --encryption-config '[{"resources":["secrets"],"provider":{"keyArn":"arn:aws:kms:..."}}]'

# 4. Use managed node groups with custom launch template
aws eks create-nodegroup \
    --cluster-name my-cluster \
    --nodegroup-name secure-nodes \
    --node-role arn:aws:iam::123456789:role/eks-node-role \
    --launch-template id=lt-0123456789,version=1
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Unauthorized` accessing API server | Anonymous auth disabled + missing credentials | Configure kubeconfig with proper certs or OIDC token |
| kube-bench shows FAIL for etcd encryption | Secrets not encrypted at rest | Apply EncryptionConfiguration and restart API server |
| Kubelet read-only port accessible | `readOnlyPort: 10255` still set | Set `readOnlyPort: 0` in kubelet config |
| Audit logs filling disk | Too verbose audit policy | Use `None` level for high-frequency read-only ops |
| CIS bench fails on managed K8s | Control plane not accessible | Use managed-specific kube-bench profile (`--benchmark eks`) |

---

## Summary Table

| Hardening Area | Key Actions |
|---------------|-------------|
| **API Server** | Disable anonymous auth, enable RBAC, audit logging, TLS 1.2+ |
| **etcd** | Encrypt at rest, TLS in transit, restrict access to API server only |
| **Kubelet** | Disable anonymous/read-only, webhook auth, protect kernel defaults |
| **Audit Logging** | Log secrets access, exec/attach, RBAC changes |
| **CIS Benchmark** | Run kube-bench regularly; remediate FAIL items |
| **Managed K8s** | Enable control plane logging, restrict public endpoint, encrypt secrets |

---

## Quick Revision Questions

1. **What are the key components to harden in a Kubernetes control plane?**
   > API Server (authentication, authorization, audit, TLS), etcd (encryption at rest, TLS, ACLs), Controller Manager (secure port, profiling disabled), and Scheduler (secure port, profiling disabled). The API server is the most critical as it's the gateway to the entire cluster.

2. **Why should anonymous authentication be disabled on the API server?**
   > Anonymous access allows unauthenticated requests to the API server. An attacker who can reach the API server network could enumerate resources, discover cluster metadata, or exploit misconfigured RBAC rules. Disabling `--anonymous-auth` forces all requests to authenticate.

3. **What does etcd encryption at rest protect against?**
   > If an attacker gains access to etcd data files (via backup theft, disk access, or etcd exploit), unencrypted secrets are readable in plaintext. Encryption at rest ensures Kubernetes Secrets are encrypted in the etcd store, requiring the encryption key to read them.

4. **What does kube-bench check?**
   > kube-bench runs the CIS Kubernetes Benchmark checks against your cluster. It verifies API server flags, kubelet configuration, etcd settings, file permissions, RBAC policies, and network policies. It reports PASS/FAIL/WARN for each check with remediation guidance.

5. **What is the Kubernetes audit policy and why is it important?**
   > The audit policy defines which API requests are logged and at what detail level (None, Metadata, Request, RequestResponse). It's critical for detecting unauthorized access, investigating incidents, and compliance. Key events to audit include secrets access, pod exec/attach, and RBAC changes.

---

[← Previous: 6.6 Container Security Tools](../06-Container-Security/06-container-security-tools.md) | [Next: 7.2 RBAC Implementation →](02-rbac-implementation.md)

[Back to Table of Contents](../README.md)
