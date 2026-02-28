# Chapter 4: Image Tagging

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

Tags are **human-readable labels** attached to Docker images. A well-designed tagging strategy is essential for version management, deployment pipelines, and rollback capabilities.

---

## 4.1 Tagging Concepts

```
  IMAGE REFERENCE:
  
  repository:tag
  ├────────┤ ├─┤
  │          │
  Image name Version label
  
  Examples:
  nginx:1.25          <-- specific version
  nginx:latest        <-- default tag (not always newest!)
  nginx:1.25-alpine   <-- version + variant
  myapp:v2.0.1        <-- semantic versioning
  myapp:abc123f       <-- git commit hash
  myapp:staging       <-- environment tag
```

### How Tags Work Internally

```
  Tags are POINTERS to image digests:
  
  nginx:latest   ────>  sha256:aaa111...  (Image A)
  nginx:1.25     ────>  sha256:aaa111...  (Same Image A!)
  nginx:1.24     ────>  sha256:bbb222...  (Image B)
  nginx:1.25.3   ────>  sha256:aaa111...  (Same Image A!)
  
  Multiple tags can point to the SAME image.
  Tags are MUTABLE — they can be moved to point to a different image!
  
  Today:     nginx:latest ──> sha256:aaa111
  Tomorrow:  nginx:latest ──> sha256:ccc333  (new build!)
  
  ONLY the digest (sha256:...) is immutable.
```

---

## 4.2 Tagging Commands

```bash
# Tag during build
docker build -t myapp:v1 .
docker build -t myapp:v1 -t myapp:latest .    # Multiple tags

# Tag an existing image
docker tag myapp:v1 myapp:latest
docker tag myapp:v1 registry.io/team/myapp:v1
docker tag myapp:v1 registry.io/team/myapp:production

# List tags
docker images myapp
# REPOSITORY   TAG       IMAGE ID       SIZE
# myapp        v1        a1b2c3d4e5f6   150MB
# myapp        latest    a1b2c3d4e5f6   150MB  <-- same ID!

# Remove a tag (only removes the tag, not the image)
docker rmi myapp:latest    # Removes tag if other tags exist
```

---

## 4.3 Tagging Strategies

### Strategy 1: Semantic Versioning (Recommended)

```
  Version: MAJOR.MINOR.PATCH
  
  myapp:1.0.0   <-- first release
  myapp:1.0.1   <-- patch (bug fix)
  myapp:1.1.0   <-- minor (new feature, backward compatible)
  myapp:2.0.0   <-- major (breaking changes)
  
  Additional floating tags:
  myapp:1       <-- latest 1.x.x (floating)
  myapp:1.1     <-- latest 1.1.x (floating)
  myapp:1.1.0   <-- exact version (pinned)
  myapp:latest  <-- latest build
  
  Tag Hierarchy:
  myapp:latest ──────────────> sha256:abc123
  myapp:2      ──────────────> sha256:abc123
  myapp:2.1    ──────────────> sha256:abc123
  myapp:2.1.3  ──────────────> sha256:abc123
```

### Strategy 2: Git Commit-Based

```bash
# Tag with git commit SHA
docker build -t myapp:$(git rev-parse --short HEAD) .
# Result: myapp:a1b2c3f

# Tag with git branch + commit
docker build -t myapp:main-$(git rev-parse --short HEAD) .
# Result: myapp:main-a1b2c3f

# In CI/CD pipeline:
docker build -t myapp:${CI_COMMIT_SHA:0:7} .
docker tag myapp:${CI_COMMIT_SHA:0:7} myapp:${CI_BRANCH_NAME}
```

### Strategy 3: Date-Based

```bash
# Tag with date
docker build -t myapp:$(date +%Y%m%d) .
# Result: myapp:20240115

# Tag with date + build number
docker build -t myapp:$(date +%Y%m%d)-${BUILD_NUMBER} .
# Result: myapp:20240115-42
```

### Strategy 4: Environment-Based

```
  myapp:dev       <-- development builds
  myapp:staging   <-- staging deployment
  myapp:production <-- production deployment
  
  WARNING: Environment tags are mutable and should be
  used alongside immutable version tags.
```

---

## 4.4 Tagging Best Practices

```
  DO:                                    DON'T:
  
  ✓ Use specific version tags           ✗ Rely on :latest in production
    myapp:v1.2.3                          myapp:latest
  
  ✓ Tag with immutable identifiers      ✗ Use only mutable tags
    myapp:abc123f (git SHA)               myapp:stable
  
  ✓ Apply multiple tags                 ✗ Use a single tag per image
    myapp:v1.2.3                        
    myapp:v1.2                          
    myapp:v1                            
    myapp:latest                        
  
  ✓ Document your tagging strategy      ✗ Tag inconsistently
  
  ✓ Pin versions in Dockerfiles         ✗ Use FROM node:latest
    FROM node:20.10.0-alpine              FROM node
```

### Why `latest` Is Dangerous in Production

```
  Scenario: Two deployments pull "myapp:latest"
  
  Day 1: Deploy myapp:latest  ──> gets v1.2.3 (works!)
  Day 2: Team pushes new build ──> latest now = v1.3.0
  Day 3: Scale up, new pod pulls myapp:latest ──> gets v1.3.0
  
  RESULT: Some pods run v1.2.3, some run v1.3.0!
  
  +----------+  +----------+  +----------+
  | Pod 1    |  | Pod 2    |  | Pod 3    |
  | v1.2.3   |  | v1.2.3   |  | v1.3.0   |  <-- DIFFERENT!
  +----------+  +----------+  +----------+
  
  FIX: Always use specific tags in production:
  image: myapp:v1.2.3    (deterministic)
```

---

## 4.5 CI/CD Tagging Example

```bash
#!/bin/bash
# build-and-tag.sh — CI/CD pipeline tagging

IMAGE="registry.io/myteam/myapp"
VERSION="1.2.3"
GIT_SHA=$(git rev-parse --short HEAD)
DATE=$(date +%Y%m%d)

# Build the image
docker build -t ${IMAGE}:${GIT_SHA} .

# Apply multiple tags
docker tag ${IMAGE}:${GIT_SHA} ${IMAGE}:${VERSION}
docker tag ${IMAGE}:${GIT_SHA} ${IMAGE}:latest
docker tag ${IMAGE}:${GIT_SHA} ${IMAGE}:${DATE}-${GIT_SHA}

# Push all tags
docker push ${IMAGE}:${GIT_SHA}
docker push ${IMAGE}:${VERSION}
docker push ${IMAGE}:latest
docker push ${IMAGE}:${DATE}-${GIT_SHA}

# Result — same image, four tags:
# registry.io/myteam/myapp:a1b2c3f
# registry.io/myteam/myapp:1.2.3
# registry.io/myteam/myapp:latest
# registry.io/myteam/myapp:20240115-a1b2c3f
```

---

## Summary Table

| Tagging Strategy | Example | Best For | Drawback |
|-----------------|---------|----------|----------|
| Semantic Version | `v1.2.3` | Releases, public images | Manual version management |
| Git SHA | `abc123f` | CI/CD, traceability | Not human-friendly |
| Date-Based | `20240115` | Nightly builds | No semantic meaning |
| Environment | `production` | Deployment pipelines | Mutable, can cause drift |
| Floating | `v1`, `v1.2` | Consumers wanting latest patch | Can change unexpectedly |

| Tag Property | Description |
|-------------|-------------|
| Mutable | Tags can be moved to different images |
| Multiple | One image can have many tags |
| Default | `:latest` is used when no tag is specified |
| Convention | Not enforced — purely organizational |

---

## Quick Revision Questions

1. **What is the difference between a tag and a digest?**
   - A tag is a mutable, human-readable label (e.g., `v1.0`). A digest is an immutable SHA256 hash of the image manifest, guaranteeing exact content

2. **Why should you avoid using `:latest` in production deployments?**
   - `latest` is mutable and can change at any time. Different nodes might pull different versions, causing inconsistency. Always use specific version tags

3. **How do you apply multiple tags to a single image build?**
   - Use multiple `-t` flags: `docker build -t myapp:v1 -t myapp:latest .` or use `docker tag` after building

4. **What tagging strategy provides the best traceability to source code?**
   - Git commit SHA tagging (e.g., `myapp:a1b2c3f`) — directly maps to the exact code that built the image

5. **What happens when you `docker rmi myapp:v1` if the image also has the tag `myapp:latest`?**
   - Only the `v1` tag is removed. The image remains because it still has the `latest` tag. The image is only deleted when all tags are removed

6. **What is a floating tag and when is it useful?**
   - A floating tag like `myapp:v1` or `myapp:v1.2` is updated to always point to the latest patch release. Useful for consumers who want automatic patch updates

---

[← Previous: Pulling and Pushing Images](03-pulling-and-pushing-images.md) | [Next: Docker Hub and Registries →](05-docker-hub-and-registries.md)
