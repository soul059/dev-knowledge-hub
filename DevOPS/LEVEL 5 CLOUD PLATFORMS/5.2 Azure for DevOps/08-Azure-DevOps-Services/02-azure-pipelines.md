# Chapter 2: Azure Pipelines

## Overview
**Azure Pipelines** is a CI/CD service in Azure DevOps that automatically builds, tests, and deploys code. It supports any language, any platform — including Linux, macOS, and Windows agents — and integrates with GitHub, Azure Repos, Bitbucket, and other source providers.

---

## 2.1 Pipeline Architecture

```
┌────────────── AZURE PIPELINES ───────────────────┐
│                                                    │
│  Source Code                                       │
│  (Azure Repos / GitHub / Bitbucket)                │
│       │                                            │
│       ▼ Trigger (push / PR / schedule)             │
│  ┌──────────────────────────────────────────────┐  │
│  │            PIPELINE                          │  │
│  │                                              │  │
│  │  Stage: Build                                │  │
│  │  ┌────────────────────────────────────────┐  │  │
│  │  │  Job: BuildApp                         │  │  │
│  │  │  ┌──────────────────────────────────┐  │  │  │
│  │  │  │  Step: Install dependencies      │  │  │  │
│  │  │  │  Step: Run unit tests            │  │  │  │
│  │  │  │  Step: Build application         │  │  │  │
│  │  │  │  Step: Publish artifact          │  │  │  │
│  │  │  └──────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────┘  │  │
│  │                                              │  │
│  │  Stage: Deploy-Dev                           │  │
│  │  ┌────────────────────────────────────────┐  │  │
│  │  │  Job: DeployToDev                      │  │  │
│  │  │  ┌──────────────────────────────────┐  │  │  │
│  │  │  │  Step: Download artifact         │  │  │  │
│  │  │  │  Step: Deploy to App Service     │  │  │  │
│  │  │  └──────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────┘  │  │
│  │                                              │  │
│  │  Stage: Deploy-Prod (with approval gate)     │  │
│  │  ┌────────────────────────────────────────┐  │  │
│  │  │  Environment: production               │  │  │
│  │  │  Approval: team-lead                   │  │  │
│  │  │  Job: DeployToProd                     │  │  │
│  │  └────────────────────────────────────────┘  │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  Agent Pools:                                      │
│  ├── Microsoft-hosted (ubuntu-latest, windows-     │
│  │   latest, macos-latest)                         │
│  └── Self-hosted (your own VMs / containers)       │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 2.2 Pipeline Hierarchy

| Component | Description |
|-----------|-------------|
| **Pipeline** | Top-level: defines entire CI/CD process |
| **Stage** | Logical phase (Build, Test, Deploy-Dev, Deploy-Prod) |
| **Job** | Unit of work that runs on one agent |
| **Step** | Individual task (script, task, checkout) |
| **Task** | Pre-built reusable step (e.g., `AzureWebApp@1`) |
| **Trigger** | Event that starts the pipeline |

---

## 2.3 Classic vs YAML Pipelines

```
┌──────── CLASSIC ──────────┬──────── YAML ─────────────┐
│                            │                            │
│  GUI-based editor          │  Code-based (azure-        │
│  (click to configure)      │  pipelines.yml in repo)    │
│                            │                            │
│  Easier for beginners      │  Version controlled        │
│                            │                            │
│  No version control        │  Peer-reviewable (PRs)     │
│  of pipeline definition    │                            │
│                            │                            │
│  Limited reusability       │  Templates & extends       │
│                            │                            │
│  ❌ Being deprecated       │  ✅ Recommended approach   │
│                            │                            │
└────────────────────────────┴────────────────────────────┘
```

---

## 2.4 Basic YAML Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    exclude:
      - docs/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
  - stage: Build
    displayName: 'Build Stage'
    jobs:
      - job: BuildJob
        steps:
          - task: UseDotNet@2
            inputs:
              packageType: 'sdk'
              version: '8.x'

          - script: |
              dotnet restore
              dotnet build --configuration $(buildConfiguration)
              dotnet test --configuration $(buildConfiguration)
            displayName: 'Restore, Build, Test'

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  - stage: DeployDev
    displayName: 'Deploy to Dev'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployWeb
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'my-azure-connection'
                    appName: 'myapp-dev'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

## 2.5 Triggers

| Trigger Type | Description | Example |
|-------------|-------------|---------|
| **CI Trigger** | Runs on code push | `trigger: [main, develop]` |
| **PR Trigger** | Runs on pull request | `pr: [main]` |
| **Scheduled** | Runs on a cron schedule | `schedules: [{cron: "0 3 * * *"}]` |
| **Pipeline** | Runs when another pipeline completes | `resources: {pipelines: [...]}` |
| **Manual** | Triggered manually in UI | `trigger: none` |

---

## 2.6 Agent Pools

```
┌────── AGENT TYPES ──────────────────────────┐
│                                              │
│  MICROSOFT-HOSTED:                           │
│  ┌────────────────────────────────────────┐  │
│  │  • ubuntu-latest (Linux)              │  │
│  │  • windows-latest (Windows Server)    │  │
│  │  • macos-latest (macOS)               │  │
│  │  • Pre-installed tools                │  │
│  │  • Fresh VM each run (no state)       │  │
│  │  • Free tier: 1 parallel job,         │  │
│  │    1800 min/month                     │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  SELF-HOSTED:                                │
│  ┌────────────────────────────────────────┐  │
│  │  • Your own VMs / containers          │  │
│  │  • Custom tools pre-installed         │  │
│  │  • Persistent state between runs      │  │
│  │  • No minute limits                   │  │
│  │  • Access to private networks         │  │
│  └────────────────────────────────────────┘  │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 2.7 Variables and Secrets

```yaml
# Variable types
variables:
  # Inline variable
  buildConfig: 'Release'
  
  # Variable group (linked from Library)
  - group: 'my-variable-group'
  
  # Template expression (compile time)
  - name: isProd
    value: ${{ eq(variables['Build.SourceBranchName'], 'main') }}

# Runtime expression
steps:
  - script: echo "Config is $(buildConfig)"

  # Conditional step
  - script: echo "Deploying to production"
    condition: eq(variables['Build.SourceBranchName'], 'main')
```

> **Secret Variables**: Mark as secret in Variable Groups or use Azure Key Vault integration. Secrets are masked in logs and cannot be echoed.

---

## 2.8 Service Connections

```bash
# Service connections authenticate pipelines to external services

# Azure Resource Manager (ARM) connection
# → Deploy to Azure subscriptions
# → Uses Service Principal or Managed Identity

# Docker Registry connection
# → Push/pull images to ACR, Docker Hub

# Kubernetes connection
# → Deploy to AKS clusters

# GitHub connection
# → Access GitHub repos
```

---

## 2.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Pipeline not triggering | Wrong branch filter in trigger | Check `trigger:` includes your branch |
| "No hosted parallelism" | Free tier needs approval | Request free parallelism from MS |
| Secret not available | Variable not marked as secret properly | Use Variable Groups or Key Vault |
| Agent offline | Self-hosted agent service stopped | Restart agent service on the host |
| Artifact not found | Wrong `artifactName` or path | Verify `PublishBuildArtifacts` output |
| Permission denied | Service connection not authorized | Authorize connection for the pipeline |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Pipeline** | Automated CI/CD: build → test → deploy |
| **YAML Pipelines** | Version-controlled, peer-reviewable (recommended) |
| **Stages/Jobs/Steps** | Hierarchical: Stage > Job > Step |
| **Triggers** | CI (push), PR, Scheduled, Pipeline, Manual |
| **Agents** | Microsoft-hosted (managed) or Self-hosted (your VMs) |
| **Service Connections** | Authenticate to Azure, Docker, K8s, GitHub |
| **Variables** | Inline, Groups, Key Vault; secrets are masked |

---

## Quick Revision Questions

1. **What is the hierarchy in a YAML pipeline?**
   > Pipeline → Stage → Job → Step. Stages are logical phases, Jobs run on agents, Steps are individual tasks.

2. **What is the difference between Microsoft-hosted and self-hosted agents?**
   > Microsoft-hosted agents are managed VMs (fresh each run, pre-installed tools, minute limits). Self-hosted agents are your own machines (persistent state, custom tools, no limits).

3. **How do you protect secrets in pipelines?**
   > Use secret variables in Variable Groups or Azure Key Vault integration. Secrets are masked in logs and cannot be echoed.

4. **What triggers can start a pipeline?**
   > CI (code push), PR (pull request), scheduled (cron), pipeline completion, or manual trigger.

5. **Why are YAML pipelines preferred over Classic?**
   > YAML pipelines are version-controlled in the repo, peer-reviewable via PRs, support templates, and are the recommended approach going forward.

6. **What is a service connection?**
   > A configured authentication link between Azure Pipelines and an external service (Azure, Docker Hub, Kubernetes, GitHub).

---

[⬅ Previous: Azure Repos](01-azure-repos.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Boards ➡](03-azure-boards.md)
