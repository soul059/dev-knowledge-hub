# Chapter 9.1: Azure Pipelines

[← Previous: GitLab CI/CD Best Practices](../08-GitLab-CICD/08-gitlab-cicd-best-practices.md) | [Back to README](../README.md) | [Next: CircleCI →](02-circleci.md)

---

## Overview

**Azure Pipelines** is Microsoft's CI/CD service within Azure DevOps. It supports any language, platform, and cloud — with deep integration into Azure services, Microsoft-hosted agents, and YAML-based pipeline configuration.

---

## Architecture

```
  AZURE DEVOPS ORGANIZATION
  ══════════════════════════

  Organization
  └── Project
      ├── Repos (Azure Repos / GitHub / Bitbucket)
      ├── Pipelines
      │   ├── Build Pipelines (CI)
      │   └── Release Pipelines (CD)  ← Classic UI (legacy)
      ├── Boards (Work tracking)
      ├── Test Plans
      └── Artifacts (Package feeds)

  YAML Pipeline Execution:
  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Trigger  │───▶│ Pipeline │───▶│  Agent   │
  │ (push/PR)│    │ (YAML)   │    │  Pool    │
  └──────────┘    └──────────┘    └──────────┘
                       │               │
                  ┌────┴────┐     ┌────┴────┐
                  │ Stages  │     │ MS-hosted│
                  │ Jobs    │     │ Self-host│
                  │ Steps   │     │ Scale Set│
                  └─────────┘     └─────────┘
```

---

## Pipeline YAML Structure

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - release/*
  paths:
    include:
      - src/
    exclude:
      - docs/

pr:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

stages:
  - stage: Build
    displayName: 'Build & Test'
    jobs:
      - job: BuildJob
        steps:
          - task: UseDotNet@2
            inputs:
              version: $(dotnetVersion)

          - script: dotnet build --configuration $(buildConfiguration)
            displayName: 'Build'

          - script: dotnet test --configuration $(buildConfiguration) --logger trx
            displayName: 'Test'

          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/*.trx'

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  - stage: Deploy
    displayName: 'Deploy to Azure'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployWeb
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'my-azure-connection'
                    appName: 'my-web-app'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

## Key Concepts

### Hierarchy

```
  Pipeline
  └── Stage (logical boundary — e.g., Build, Test, Deploy)
      └── Job (runs on one agent)
          └── Step (task or script)

  Stage dependencies:
  ┌───────┐    ┌───────┐    ┌──────────┐
  │ Build │───▶│ Test  │───▶│  Deploy  │
  └───────┘    └───────┘    └──────────┘
                              condition:
                              succeeded()
```

### Tasks vs Scripts

```yaml
steps:
  # Script step — inline commands
  - script: |
      echo "Building..."
      npm ci
      npm run build
    displayName: 'Build App'

  # Task step — reusable packaged action
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node.js'

  # PowerShell step
  - powershell: |
      Write-Host "Running on Windows"
    displayName: 'PowerShell Step'

  # Bash step (explicit)
  - bash: echo "Running on any OS"
    displayName: 'Bash Step'
```

### Agent Pools

```
  ┌────────────────────┬──────────────────────────────────┐
  │ Pool Type          │ Details                          │
  ├────────────────────┼──────────────────────────────────┤
  │ Microsoft-hosted   │ Fresh VM each run, pre-installed │
  │                    │ tools, no maintenance            │
  │                    │ ubuntu-latest, windows-latest,   │
  │                    │ macos-latest                     │
  ├────────────────────┼──────────────────────────────────┤
  │ Self-hosted        │ Your infrastructure, persistent  │
  │                    │ cache, custom tools              │
  ├────────────────────┼──────────────────────────────────┤
  │ Scale Set agents   │ Azure VMSS-backed, auto-scaling  │
  │                    │ agents for dynamic demand        │
  └────────────────────┴──────────────────────────────────┘
```

---

## Templates

```yaml
# templates/build-template.yml
parameters:
  - name: buildConfiguration
    type: string
    default: 'Release'
  - name: projects
    type: string
    default: '**/*.csproj'

steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: 'build'
      projects: ${{ parameters.projects }}
      arguments: '--configuration ${{ parameters.buildConfiguration }}'

# Main pipeline using template
stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - template: templates/build-template.yml
            parameters:
              buildConfiguration: 'Release'
```

### Template Types

```
  ┌──────────────────┬────────────────────────────────────┐
  │ Template Type    │ Purpose                            │
  ├──────────────────┼────────────────────────────────────┤
  │ Step template    │ Reusable steps within a job        │
  │ Job template     │ Reusable job definitions           │
  │ Stage template   │ Reusable stage definitions         │
  │ Variable template│ Shared variable definitions        │
  └──────────────────┴────────────────────────────────────┘
```

---

## Environments & Approvals

```yaml
# Deployment job with environment
- stage: Production
  jobs:
    - deployment: DeployProd
      environment: 'production'         # Tracks deployments
      strategy:
        runOnce:
          deploy:
            steps:
              - script: ./deploy.sh

# Environment settings (in Azure DevOps UI):
# • Required approvers
# • Branch restrictions
# • Business hours check
# • Exclusive lock (one deployment at a time)
```

---

## Variables & Secrets

```yaml
# Pipeline-level variables
variables:
  - name: appName
    value: 'my-app'
  - group: 'production-secrets'         # Variable group (from Library)
  - name: secretKey
    value: $(MY_SECRET)                 # From pipeline variables (UI)

# Stage-level variables
stages:
  - stage: Deploy
    variables:
      environment: 'production'

# Runtime expressions
steps:
  - script: echo $(appName)            # Macro syntax — compile time
  - script: echo $[variables.appName]  # Runtime expression
```

```
  VARIABLE PRECEDENCE (highest to lowest):
  ═══════════════════════════════════════

  1. Queue-time variables (set at run)
  2. Pipeline-level variables
  3. Stage-level variables
  4. Job-level variables
  5. Variable groups
  6. Template parameters
```

---

## Service Connections

```
  ┌──────────────┐        ┌──────────────────┐
  │  Pipeline    │───SC──▶│  Azure (ARM)     │
  │              │───SC──▶│  Docker Hub      │
  │              │───SC──▶│  Kubernetes      │
  │              │───SC──▶│  AWS             │
  └──────────────┘        └──────────────────┘

  SC = Service Connection
  Created in: Project Settings → Service Connections
  Auth methods: Service Principal, Managed Identity,
                Workload Identity Federation (recommended)
```

---

## Real-World Scenario

> **Scenario**: A .NET team using Azure wants CI/CD with staging and production environments, requiring approval for production deployments.
>
> **Solution**: Create `azure-pipelines.yml` with three stages: Build (compile + test + publish artifacts), Deploy-Staging (auto-deploy on main), and Deploy-Production (manual approval via environment check). Use Azure Web App task for deployment, service connection with Workload Identity Federation, and variable groups for environment-specific config. Configure the production environment with required approvers in the Azure DevOps UI.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Pipeline not triggering | Wrong trigger/pr config | Check branch filters and path filters |
| "No hosted parallelism" | Free tier — need to request | Request free parallelism or use self-hosted |
| Task not found | Extension not installed | Install extension from Marketplace |
| Variable not resolved | Wrong syntax or scope | Use `$(var)` for compile-time, `$[var]` for runtime |
| Deployment blocked | Pending approval | Check environment approvals in Azure DevOps |
| Slow builds | Microsoft-hosted cold start | Use self-hosted agents with cached dependencies |

---

## Summary Table

| Feature | Azure Pipelines |
|---|---|
| Config file | `azure-pipelines.yml` |
| Hierarchy | Pipeline → Stage → Job → Step |
| Agents | Microsoft-hosted, self-hosted, Scale Sets |
| Templates | Step, Job, Stage, Variable templates |
| Environments | With approvals, checks, deployment history |
| Secrets | Variable groups, Azure Key Vault integration |
| Marketplace | Tasks (extensions from VS Marketplace) |
| Best for | Azure-native, .NET, enterprise teams |

---

## Quick Revision Questions

1. **What is the hierarchy of an Azure Pipeline?** *Pipeline → Stage → Job → Step. Stages are logical boundaries, jobs run on agents, and steps are individual tasks or scripts.*
2. **What are the agent pool types?** *Microsoft-hosted (fresh VMs, pre-installed tools), Self-hosted (your infrastructure), and Scale Set agents (Azure VMSS auto-scaling).*
3. **How do templates work in Azure Pipelines?** *Templates are reusable YAML files (step, job, stage, or variable level) included with `template:` keyword, accepting typed parameters.*
4. **How are deployments tracked?** *Using `deployment` jobs with `environment:` — environments track history and can have approval checks, branch restrictions, and exclusive locks.*
5. **What authentication method is recommended for Azure?** *Workload Identity Federation via service connections — no stored secrets, uses OIDC-based trust.*
6. **How does Azure Pipelines compare to GitHub Actions?** *Azure Pipelines uses tasks (vs actions), has explicit stages (vs implicit), uses agent pools (vs runners), and has deeper Azure/enterprise integration.*

---

[← Previous: GitLab CI/CD Best Practices](../08-GitLab-CICD/08-gitlab-cicd-best-practices.md) | [Back to README](../README.md) | [Next: CircleCI →](02-circleci.md)
