# Chapter 7.5: AWS CodeArtifact — Managed Artifact Repository

## Overview

AWS CodeArtifact is a **fully managed artifact repository service** for storing, publishing, and sharing software packages. It works with popular package managers like npm, pip, Maven, NuGet, and Gradle, providing a central place to manage dependencies.

---

## CodeArtifact Architecture

```
┌──── CodeArtifact ────────────────────────────────────────┐
│                                                           │
│  Developers / CI/CD                 CodeArtifact          │
│  ┌──────────────────┐          ┌──────────────────────┐  │
│  │ npm install      │          │ Domain: my-org       │  │
│  │ pip install      │─────────→│                      │  │
│  │ mvn install      │          │ ┌──────────────────┐ │  │
│  │ nuget install    │          │ │ Repo: internal   │ │  │
│  └──────────────────┘          │ │ (your packages)  │ │  │
│                                │ └────────┬─────────┘ │  │
│  If not found locally:        │          │ upstream   │  │
│  Fetches from upstream ───────→│ ┌────────▼─────────┐ │  │
│                                │ │ Repo: npm-store  │ │  │
│                                │ │ (cached public)  │ │  │
│                                │ └────────┬─────────┘ │  │
│                                │          │ upstream   │  │
│                                │ ┌────────▼─────────┐ │  │
│                                │ │ npmjs.com        │ │  │
│                                │ │ (external)       │ │  │
│                                │ └──────────────────┘ │  │
│                                └──────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

---

## Core Concepts

```
┌──── Hierarchy ───────────────────────────────────────────┐
│                                                          │
│  Domain                                                  │
│  └─ Organizational boundary (one per org/account)        │
│     ├─ Repository (internal packages)                    │
│     │  └─ Packages: @myorg/utils, @myorg/auth           │
│     │                                                    │
│     ├─ Repository (npm-store — upstream cache)           │
│     │  └─ Upstream: npmjs.com                            │
│     │  └─ Cached: react, express, lodash                │
│     │                                                    │
│     └─ Repository (pypi-store — upstream cache)          │
│        └─ Upstream: pypi.org                             │
│        └─ Cached: boto3, flask, requests                │
│                                                          │
│  ⚠ Domains are regional (not global)                   │
│  ✓ Cross-account sharing via domain policies            │
└──────────────────────────────────────────────────────────┘
```

---

## Setup Commands

```bash
# Create a domain
aws codeartifact create-domain --domain my-org

# Create an internal repository
aws codeartifact create-repository \
  --domain my-org \
  --repository internal

# Create upstream repository (caches public packages)
aws codeartifact create-repository \
  --domain my-org \
  --repository npm-store \
  --upstreams repositoryName=npmjs-upstream

# Connect internal repo to npm-store as upstream
aws codeartifact associate-external-connection \
  --domain my-org \
  --repository npm-store \
  --external-connection public:npmjs

aws codeartifact update-repository \
  --domain my-org \
  --repository internal \
  --upstreams repositoryName=npm-store
```

---

## Authenticating Package Managers

```bash
# Get auth token (valid for 12 hours by default)
export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token \
  --domain my-org \
  --domain-owner 123456789012 \
  --query authorizationToken \
  --output text)

# ─── npm ───
aws codeartifact login --tool npm \
  --domain my-org \
  --domain-owner 123456789012 \
  --repository internal

# ─── pip ───
aws codeartifact login --tool pip \
  --domain my-org \
  --domain-owner 123456789012 \
  --repository internal

# ─── Maven (settings.xml) ───
# Configure in ~/.m2/settings.xml with auth token
aws codeartifact get-repository-endpoint \
  --domain my-org \
  --repository internal \
  --format maven

# ─── NuGet ───
aws codeartifact login --tool dotnet \
  --domain my-org \
  --domain-owner 123456789012 \
  --repository internal
```

---

## CodeBuild Integration

```yaml
# buildspec.yml — authenticate to CodeArtifact during build
version: 0.2

phases:
  pre_build:
    commands:
      # Login npm to CodeArtifact
      - aws codeartifact login --tool npm
          --domain my-org
          --domain-owner 123456789012
          --repository internal
      # OR use pip
      - aws codeartifact login --tool pip
          --domain my-org
          --domain-owner 123456789012
          --repository internal
  install:
    commands:
      - npm ci     # Uses CodeArtifact as registry
  build:
    commands:
      - npm run build
      - npm test
```

---

## Domain Policies (Cross-Account)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountRead",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:root"
      },
      "Action": [
        "codeartifact:GetAuthorizationToken",
        "codeartifact:GetRepositoryEndpoint",
        "codeartifact:ReadFromRepository"
      ],
      "Resource": "*"
    }
  ]
}
```

```bash
# Apply domain policy
aws codeartifact put-domain-permissions-policy \
  --domain my-org \
  --policy-document file://domain-policy.json
```

---

## Package Management

```bash
# List packages in repository
aws codeartifact list-packages \
  --domain my-org \
  --repository internal

# Publish npm package
npm publish --registry https://my-org-123456789012.d.codeartifact.us-east-1.amazonaws.com/npm/internal/

# Delete package version
aws codeartifact delete-package-versions \
  --domain my-org \
  --repository internal \
  --format npm \
  --package my-utils \
  --versions 1.0.0

# Copy package between repos
aws codeartifact copy-package-versions \
  --domain my-org \
  --source-repository staging \
  --destination-repository production \
  --format npm \
  --package my-utils \
  --versions 2.0.0
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `401 Unauthorized` | Token expired (12-hour default) | Re-run `aws codeartifact login` to refresh token |
| Package not found | Not in repo and no upstream | Configure upstream connection to public registry |
| Cross-account access denied | Missing domain policy | Add domain permissions for the external account |
| Slow first install | Package not cached yet | First request fetches from upstream; cached afterwards |
| Can't publish | Missing `codeartifact:PublishPackageVersion` | Add publish permissions to IAM role/user |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CodeArtifact** | Managed artifact repo for npm, pip, Maven, NuGet packages |
| **Domain** | Organizational boundary. One per org. Regional. |
| **Upstream** | Chain repos: internal → cache → public registry |
| **Auth Token** | 12-hour default. Use `aws codeartifact login` per tool |
| **Cross-Account** | Domain policies grant access to other AWS accounts |
| **CI/CD** | Authenticate in CodeBuild `pre_build` phase |

---

## Quick Revision Questions

1. **What is a CodeArtifact domain?**
   > An organizational container for repositories. Controls ownership, access policy, and billing. One domain typically per organization.

2. **How do upstream repositories work?**
   > When a package isn't found in the current repo, CodeArtifact fetches it from upstream repos in chain order, caching it locally for future requests.

3. **How long is the default auth token valid?**
   > 12 hours. After expiration, you need to re-authenticate using `aws codeartifact login`.

4. **Which package formats does CodeArtifact support?**
   > npm (JavaScript), pip/twine (Python), Maven/Gradle (Java), NuGet (.NET), Swift, and generic.

5. **How do you share packages across AWS accounts?**
   > Apply a domain permissions policy that grants the external account permissions like `GetAuthorizationToken` and `ReadFromRepository`.

---

[← Previous: 7.4 CodePipeline](04-codepipeline.md) | [Next: 7.6 GitHub Actions with AWS →](06-github-actions-with-aws.md)

[← Back to README](../README.md)
