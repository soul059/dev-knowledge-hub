# Chapter 9.4: Tekton Pipelines

[← Previous: ArgoCD](03-argocd.md) | [Back to README](../README.md) | [Next: CI/CD Tool Comparison →](05-comparison-matrix.md)

---

## Overview

**Tekton** is a cloud-native, Kubernetes-native CI/CD framework. It defines pipelines as Kubernetes custom resources (CRDs), running each step in its own container within a pod. Tekton is the foundation for several platforms including OpenShift Pipelines.

---

## Architecture

```
  TEKTON ARCHITECTURE
  ════════════════════

  Kubernetes Cluster
  ┌──────────────────────────────────────────────┐
  │                                              │
  │  Tekton Controller (watches CRDs)            │
  │       │                                      │
  │       ▼                                      │
  │  ┌──────────┐   ┌──────────┐   ┌─────────┐  │
  │  │ Pipeline │──▶│  Task    │──▶│  Step   │  │
  │  │ (ordered │   │ (unit of │   │ (single │  │
  │  │  tasks)  │   │  work)   │   │ container│  │
  │  └──────────┘   └──────────┘   └─────────┘  │
  │                                              │
  │  Execution:                                  │
  │  ┌──────────────┐   ┌──────────────┐         │
  │  │ PipelineRun  │──▶│  TaskRun     │         │
  │  │ (instance of │   │ (instance of │         │
  │  │  Pipeline)   │   │  Task)       │         │
  │  └──────────────┘   └──────────────┘         │
  │                                              │
  │  TaskRun creates a Pod:                      │
  │  ┌──────────────────────────────┐            │
  │  │ Pod                         │            │
  │  │ ├── Init Container (setup)  │            │
  │  │ ├── Step 1 Container        │            │
  │  │ ├── Step 2 Container        │            │
  │  │ └── Step 3 Container        │            │
  │  │     (shared /workspace vol) │            │
  │  └──────────────────────────────┘            │
  └──────────────────────────────────────────────┘
```

---

## Core Concepts

```
  TEKTON CRD HIERARCHY
  ═════════════════════

  Definitions (reusable):
  ├── Task          → Sequence of steps (runs in one Pod)
  ├── Pipeline      → Ordered collection of Tasks
  └── Trigger       → Listens for events (webhooks)

  Executions (instances):
  ├── TaskRun       → One execution of a Task
  └── PipelineRun   → One execution of a Pipeline

  Supporting:
  ├── Workspace     → Shared storage between Tasks
  ├── Result        → Output from a Task
  └── Param         → Input parameters
```

---

## Task Definition

```yaml
# A Task defines a unit of work
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: build-and-push
spec:
  params:
    - name: image
      type: string
      description: "Image URL to push to"
    - name: dockerfile
      type: string
      default: "./Dockerfile"

  workspaces:
    - name: source
      description: "Source code workspace"

  results:
    - name: image-digest
      description: "Digest of the built image"

  steps:
    - name: build
      image: gcr.io/kaniko-project/executor:latest
      args:
        - --dockerfile=$(params.dockerfile)
        - --context=$(workspaces.source.path)
        - --destination=$(params.image)
        - --digest-file=$(results.image-digest.path)
```

---

## Pipeline Definition

```yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: ci-pipeline
spec:
  params:
    - name: repo-url
      type: string
    - name: image
      type: string

  workspaces:
    - name: shared-workspace
    - name: docker-credentials

  tasks:
    # Clone source code
    - name: fetch-source
      taskRef:
        name: git-clone          # From Tekton Hub
      workspaces:
        - name: output
          workspace: shared-workspace
      params:
        - name: url
          value: $(params.repo-url)

    # Run tests
    - name: run-tests
      taskRef:
        name: run-tests
      runAfter:
        - fetch-source           # Explicit ordering
      workspaces:
        - name: source
          workspace: shared-workspace

    # Build and push image
    - name: build-image
      taskRef:
        name: build-and-push
      runAfter:
        - run-tests
      workspaces:
        - name: source
          workspace: shared-workspace
      params:
        - name: image
          value: $(params.image)

    # Deploy
    - name: deploy
      taskRef:
        name: kubectl-deploy
      runAfter:
        - build-image
      params:
        - name: image-digest
          value: $(tasks.build-image.results.image-digest)
```

---

## PipelineRun (Execution)

```yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: ci-pipeline-run-    # Auto-generate unique name
spec:
  pipelineRef:
    name: ci-pipeline
  params:
    - name: repo-url
      value: "https://github.com/org/app.git"
    - name: image
      value: "registry.example.com/app:latest"
  workspaces:
    - name: shared-workspace
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
    - name: docker-credentials
      secret:
        secretName: docker-registry-creds
```

---

## Triggers

```yaml
# EventListener — receives webhooks
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: github-listener
spec:
  serviceAccountName: tekton-triggers-sa
  triggers:
    - name: github-push
      interceptors:
        - ref:
            name: github
          params:
            - name: secretRef
              value:
                secretName: github-webhook-secret
                secretKey: token
            - name: eventTypes
              value: ["push"]
      bindings:
        - ref: github-push-binding
      template:
        ref: ci-pipeline-template

---
# TriggerBinding — extracts data from webhook payload
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: github-push-binding
spec:
  params:
    - name: repo-url
      value: $(body.repository.clone_url)
    - name: revision
      value: $(body.head_commit.id)

---
# TriggerTemplate — creates PipelineRun from extracted data
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: ci-pipeline-template
spec:
  params:
    - name: repo-url
    - name: revision
  resourcetemplates:
    - apiVersion: tekton.dev/v1
      kind: PipelineRun
      metadata:
        generateName: triggered-run-
      spec:
        pipelineRef:
          name: ci-pipeline
        params:
          - name: repo-url
            value: $(tt.params.repo-url)
```

```
  TRIGGER FLOW
  ═════════════

  GitHub Webhook
       │
       ▼
  EventListener (Kubernetes Service)
       │
       ▼
  Interceptor (validates, filters)
       │
       ▼
  TriggerBinding (extracts params from payload)
       │
       ▼
  TriggerTemplate (creates PipelineRun)
       │
       ▼
  PipelineRun executes Pipeline
```

---

## Tekton Hub (Reusable Tasks)

```
  TEKTON HUB (hub.tekton.dev)
  ════════════════════════════

  Popular catalog tasks:
  ├── git-clone        → Clone Git repository
  ├── kaniko           → Build container images (no Docker daemon)
  ├── buildah          → Build OCI images
  ├── kubectl-deploy   → Deploy to Kubernetes
  ├── helm-upgrade     → Helm chart deployment
  ├── golang-test      → Run Go tests
  ├── npm              → Node.js npm operations
  └── maven            → Java Maven build/test

  Install:
  kubectl apply -f https://hub.tekton.dev/tekton/task/git-clone/0.9/raw
```

---

## Tekton vs Other CI/CD Tools

```
  ┌─────────────────┬──────────────────────────────────┐
  │ Feature         │ Tekton                           │
  ├─────────────────┼──────────────────────────────────┤
  │ Execution model │ Kubernetes-native (CRDs + Pods)  │
  │ Scaling         │ Kubernetes-native auto-scaling   │
  │ Config format   │ Kubernetes YAML (CRDs)           │
  │ Portability     │ Any Kubernetes cluster           │
  │ UI              │ Tekton Dashboard (optional)      │
  │ Vendor lock-in  │ None — CNCF/Linux Foundation     │
  │ Best for        │ K8s-native teams, platform teams │
  │ Learning curve  │ Steep (requires K8s knowledge)   │
  └─────────────────┴──────────────────────────────────┘
```

---

## Real-World Scenario

> **Scenario**: A platform team building an internal developer platform on Kubernetes wants CI/CD that is cloud-agnostic and doesn't require external SaaS dependencies.
>
> **Solution**: Deploy Tekton on the existing Kubernetes cluster. Create catalog Tasks for common operations (build, test, scan, deploy). Define Pipeline templates that product teams reuse. Set up EventListeners for GitHub webhooks. Store credentials as Kubernetes Secrets mounted as workspaces. Use Tekton Dashboard for visibility. The entire CI/CD system runs inside Kubernetes with no external dependencies — portable across any K8s distribution (EKS, GKE, AKS, on-prem).

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| TaskRun stuck Pending | No PVC available or scheduling issues | Check PVC provisioner and node resources |
| Step fails with permission error | Container security context | Set `securityContext` in step or use appropriate service account |
| Workspace data not shared | Wrong workspace binding | Ensure same workspace name is used in Pipeline and Task |
| Trigger not firing | EventListener not exposed | Create Ingress or use `kubectl port-forward` |
| "Task not found" | Task not installed in cluster | Apply Task YAML from Tekton Hub |
| PipelineRun OOMKilled | Resource limits too low | Increase step resource `requests`/`limits` |

---

## Summary Table

| Concept | Description |
|---|---|
| Task | Sequence of steps running in one Pod |
| Step | Single container executing a command |
| Pipeline | Ordered collection of Tasks |
| PipelineRun | One execution of a Pipeline |
| Workspace | Shared volume between Tasks |
| Result | Output value passed between Tasks |
| EventListener | Receives webhooks and creates PipelineRuns |
| TriggerBinding | Extracts parameters from event payloads |
| TriggerTemplate | Creates resources from extracted parameters |
| Tekton Hub | Catalog of reusable community Tasks |

---

## Quick Revision Questions

1. **What is Tekton?** *A Kubernetes-native CI/CD framework that defines pipelines as Kubernetes CRDs, running each step as a container in a Pod.*
2. **What's the difference between a Task and a Pipeline?** *A Task is a sequence of steps running in a single Pod. A Pipeline is an ordered collection of Tasks, where each Task runs in its own Pod.*
3. **How does Tekton run steps?** *Each step runs as a separate container within a single Pod. Steps share a workspace volume for data passing within a Task.*
4. **What is a PipelineRun?** *A concrete execution instance of a Pipeline — it provides parameter values and workspace bindings, creating TaskRuns for each Task.*
5. **How do Tekton Triggers work?** *EventListener receives webhooks → Interceptors validate/filter → TriggerBinding extracts params from payload → TriggerTemplate creates a PipelineRun.*
6. **When should you choose Tekton over other CI/CD tools?** *When you want fully Kubernetes-native CI/CD with no external dependencies, cloud-agnostic portability, and are building an internal developer platform.*

---

[← Previous: ArgoCD](03-argocd.md) | [Back to README](../README.md) | [Next: CI/CD Tool Comparison →](05-comparison-matrix.md)
