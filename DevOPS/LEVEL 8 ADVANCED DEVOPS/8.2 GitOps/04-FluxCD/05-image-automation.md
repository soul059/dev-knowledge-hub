# Chapter 4.5: Image Automation

[← Previous: HelmRelease](04-helmrelease.md) | [Back to README](../README.md) | [Next → Multi-Tenancy](06-multi-tenancy.md)

---

## Overview

Flux's **Image Automation** feature enables fully automated deployments: when a new container image is pushed to a registry, Flux detects it, updates the image tag in Git, and reconciles the cluster. This closes the loop between CI (build image) and CD (deploy image) without manual steps.

---

## Image Automation Architecture

```
┌──────────────────────────────────────────────────────────────┐
│           IMAGE AUTOMATION PIPELINE                          │
│                                                              │
│  ┌──────────┐     ┌──────────────────┐                      │
│  │ CI Build │────→│ Container        │                      │
│  │ Pipeline │     │ Registry         │                      │
│  └──────────┘     │ (DockerHub, ECR, │                      │
│                   │  GCR, ACR)       │                      │
│                   └────────┬─────────┘                      │
│                            │                                │
│  ┌─────────────────────────▼──────────────────────────┐     │
│  │  IMAGE REFLECTOR CONTROLLER                        │     │
│  │  Scans registry for new tags                       │     │
│  │  CRDs: ImageRepository, ImagePolicy                │     │
│  └─────────────────────────┬──────────────────────────┘     │
│                            │ new tag detected               │
│  ┌─────────────────────────▼──────────────────────────┐     │
│  │  IMAGE AUTOMATION CONTROLLER                       │     │
│  │  Updates image tag in Git repository               │     │
│  │  CRD: ImageUpdateAutomation                        │     │
│  └─────────────────────────┬──────────────────────────┘     │
│                            │ git commit + push              │
│  ┌─────────────────────────▼──────────────────────────┐     │
│  │  SOURCE CONTROLLER                                 │     │
│  │  Detects new commit → produces artifact            │     │
│  └─────────────────────────┬──────────────────────────┘     │
│                            │                                │
│  ┌─────────────────────────▼──────────────────────────┐     │
│  │  KUSTOMIZE CONTROLLER                              │     │
│  │  Applies updated manifests → cluster updated       │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Step 1: ImageRepository

Tells Flux which container registry and image to scan.

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  image: docker.io/myorg/my-app    # Image to scan
  interval: 1m                     # Scan frequency
  secretRef:
    name: registry-credentials     # Auth for private registry
```

### Private Registry Authentication

```yaml
# Create registry credentials
kubectl create secret docker-registry registry-credentials \
  --namespace=flux-system \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mytoken

# For ECR (AWS)
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  image: 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app
  interval: 1m
  provider: aws    # Uses IRSA for auth
```

---

## Step 2: ImagePolicy

Defines the **tag selection strategy** — which tag should be deployed.

### Semver Policy (Most Common)

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: my-app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: my-app
  policy:
    semver:
      range: ">=1.0.0 <2.0.0"     # Latest 1.x.x tag
```

### Alphabetical Policy

```yaml
spec:
  policy:
    alphabetical:
      order: asc                   # Latest alphabetically
```

### Numerical Policy

```yaml
spec:
  policy:
    numerical:
      order: asc                   # Latest numerically
```

### Timestamp-Based (Filter + Numerical)

```yaml
spec:
  filterTags:
    pattern: '^main-[a-f0-9]+-(?P<ts>[0-9]+)'
    extract: '$ts'
  policy:
    numerical:
      order: asc                   # Latest by timestamp
```

---

## Step 3: ImageUpdateAutomation

Configures Flux to **commit image tag updates to Git**.

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageUpdateAutomation
metadata:
  name: my-app-automation
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: fluxbot
        email: flux@example.com
      messageTemplate: |
        Automated image update

        Automation: {{ .AutomationObject }}

        Files changed:
        {{ range $filename, $_ := .Changed.FileChanges -}}
        - {{ $filename }}
        {{ end -}}

        Objects changed:
        {{ range $resource, $changes := .Changed.Objects -}}
        - {{ $resource.Kind }}/{{ $resource.Name }} ({{ $resource.Namespace }})
          {{ range $_, $change := $changes -}}
          {{ $change.OldValue }} -> {{ $change.NewValue }}
          {{ end -}}
        {{ end -}}
    push:
      branch: main                # Push to this branch
  update:
    path: ./clusters/my-cluster   # Path to scan for image markers
    strategy: Setters              # Use image policy markers
```

---

## Step 4: Image Markers in Manifests

Add **marker comments** in your YAML to tell Flux which fields to update:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: docker.io/myorg/my-app:1.2.3  # {"$imagepolicy": "flux-system:my-app"}
```

### Marker Variants

```yaml
# Full image (registry/repo:tag)
image: docker.io/myorg/my-app:1.2.3  # {"$imagepolicy": "flux-system:my-app"}

# Tag only
image: docker.io/myorg/my-app:1.2.3  # {"$imagepolicy": "flux-system:my-app:tag"}

# Name only (registry/repo)
image: docker.io/myorg/my-app:1.2.3  # {"$imagepolicy": "flux-system:my-app:name"}
```

---

## End-to-End Example

```
┌────────────────────────────────────────────────────────┐
│          COMPLETE IMAGE AUTOMATION FLOW                 │
│                                                        │
│  1. Developer pushes code                              │
│     │                                                  │
│     ▼                                                  │
│  2. CI builds image: myorg/my-app:1.3.0                │
│     CI pushes to Docker Hub                            │
│     │                                                  │
│     ▼                                                  │
│  3. Image Reflector scans Docker Hub (every 1m)        │
│     Detects new tag 1.3.0                              │
│     │                                                  │
│     ▼                                                  │
│  4. ImagePolicy selects 1.3.0 (matches >=1.0.0 <2.0)  │
│     │                                                  │
│     ▼                                                  │
│  5. ImageUpdateAutomation:                             │
│     - Updates deployment.yaml:                         │
│       image: myorg/my-app:1.2.3 → myorg/my-app:1.3.0  │
│     - Commits and pushes to Git                        │
│     │                                                  │
│     ▼                                                  │
│  6. Source Controller detects new commit                │
│     │                                                  │
│     ▼                                                  │
│  7. Kustomize Controller applies new manifests         │
│     Cluster now runs 1.3.0                             │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## CLI Commands

```bash
# Install image automation controllers (not included by default)
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  ...

# Create ImageRepository
flux create image repository my-app \
  --image=docker.io/myorg/my-app \
  --interval=1m

# Create ImagePolicy
flux create image policy my-app \
  --image-ref=my-app \
  --select-semver=">=1.0.0 <2.0.0"

# Check scanned tags
flux get image repository my-app
flux get image policy my-app

# Force scan
flux reconcile image repository my-app
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| ImageRepository | Scans a container registry for available tags |
| ImagePolicy | Selects the latest tag based on semver/alphabetical/numerical |
| ImageUpdateAutomation | Commits updated image tags to Git |
| Markers | `# {"$imagepolicy": "ns:name"}` in YAML comments |
| Setter strategy | Uses marker comments to find and replace image tags |
| Extra components | Image controllers require `--components-extra` flag |
| Full automation | CI pushes image → Flux updates Git → Flux deploys |

---

## Quick Revision Questions

1. **What three CRDs make up Flux image automation?**
   > `ImageRepository` (scan registry), `ImagePolicy` (select tag), `ImageUpdateAutomation` (commit to Git).

2. **How does Flux know which field to update in your YAML?**
   > You add marker comments: `# {"$imagepolicy": "flux-system:my-app"}` next to the image field.

3. **What tag selection strategies does ImagePolicy support?**
   > `semver` (semantic versioning range), `alphabetical`, and `numerical`. You can also use `filterTags` with regex for timestamp-based selection.

4. **Are image automation controllers installed by default?**
   > No. You must add `--components-extra=image-reflector-controller,image-automation-controller` during bootstrap.

5. **What does the ImageUpdateAutomation commit message contain?**
   > A configurable template showing which files changed, which objects were updated, and the old/new image values.

6. **How does this differ from ArgoCD Image Updater?**
   > Flux image automation is built-in (as extra controllers), while ArgoCD requires a separate ArgoCD Image Updater addon. Both commit tag changes back to Git.

---

[← Previous: HelmRelease](04-helmrelease.md) | [Back to README](../README.md) | [Next → Multi-Tenancy](06-multi-tenancy.md)
