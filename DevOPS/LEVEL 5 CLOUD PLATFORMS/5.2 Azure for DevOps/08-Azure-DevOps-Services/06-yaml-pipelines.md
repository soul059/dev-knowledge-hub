# Chapter 6: YAML Pipelines In-Depth

## Overview
This chapter dives deep into **YAML pipeline** features: templates, expressions, conditions, environments, deployment strategies, and multi-stage patterns used in real-world Azure DevOps CI/CD.

---

## 6.1 Templates

Templates allow reusing pipeline logic across multiple pipelines. There are **step templates**, **job templates**, **stage templates**, and **variable templates**.

### Step Template

```yaml
# templates/build-steps.yml
parameters:
  - name: buildConfiguration
    type: string
    default: 'Release'

steps:
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '8.x'

  - script: |
      dotnet restore
      dotnet build --configuration ${{ parameters.buildConfiguration }}
    displayName: 'Restore and Build'

  - script: dotnet test --configuration ${{ parameters.buildConfiguration }} --logger trx
    displayName: 'Run Tests'
```

### Using the Template

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - template: templates/build-steps.yml
    parameters:
      buildConfiguration: 'Release'
```

### Stage Template

```yaml
# templates/deploy-stage.yml
parameters:
  - name: environment
    type: string
  - name: appName
    type: string
  - name: serviceConnection
    type: string

stages:
  - stage: Deploy_${{ parameters.environment }}
    displayName: 'Deploy to ${{ parameters.environment }}'
    jobs:
      - deployment: DeployApp
        environment: ${{ parameters.environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: ${{ parameters.serviceConnection }}
                    appName: ${{ parameters.appName }}
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

```yaml
# azure-pipelines.yml — multi-environment deploy
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - template: templates/build-steps.yml

  - template: templates/deploy-stage.yml
    parameters:
      environment: 'dev'
      appName: 'myapp-dev'
      serviceConnection: 'azure-dev'

  - template: templates/deploy-stage.yml
    parameters:
      environment: 'staging'
      appName: 'myapp-staging'
      serviceConnection: 'azure-staging'

  - template: templates/deploy-stage.yml
    parameters:
      environment: 'production'
      appName: 'myapp-prod'
      serviceConnection: 'azure-prod'
```

---

## 6.2 Expressions and Conditions

```
┌────── EXPRESSION TYPES ───────────────────┐
│                                            │
│  Compile-time:  ${{ expression }}           │
│  • Evaluated before pipeline runs          │
│  • Used in template parameters             │
│                                            │
│  Runtime:       $[ expression ]             │
│  • Evaluated during pipeline execution     │
│  • Used in variable definitions            │
│                                            │
│  Macro:         $(variable)                 │
│  • Simple variable substitution            │
│  • Expanded just before task runs          │
│                                            │
└────────────────────────────────────────────┘
```

### Common Conditions

```yaml
# Run only on main branch
- script: echo "Production deploy"
  condition: eq(variables['Build.SourceBranchName'], 'main')

# Run even if previous step failed
- script: echo "Always runs"
  condition: always()

# Run only if previous steps succeeded
- script: echo "Only on success"
  condition: succeeded()

# Run only if a variable is set
- script: echo "Feature flag enabled"
  condition: eq(variables['ENABLE_FEATURE'], 'true')

# Compound condition
- script: echo "Main + Release build"
  condition: and(
    eq(variables['Build.SourceBranchName'], 'main'),
    eq(variables['Build.Reason'], 'IndividualCI')
  )
```

---

## 6.3 Deployment Strategies

```
┌────── DEPLOYMENT STRATEGIES ──────────────┐
│                                            │
│  runOnce:                                  │
│  ┌──────────────────────────────────────┐  │
│  │  Deploy once to all targets          │  │
│  │  Simplest strategy                   │  │
│  │  Good for: dev/test environments     │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  rolling:                                  │
│  ┌──────────────────────────────────────┐  │
│  │  Deploy to N targets at a time       │  │
│  │  e.g., 25% of VMs per batch          │  │
│  │  Good for: VM-based deployments      │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  canary:                                   │
│  ┌──────────────────────────────────────┐  │
│  │  Deploy to small % first (e.g., 10%) │  │
│  │  Monitor → if healthy → full deploy  │  │
│  │  Good for: risk-sensitive production │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

```yaml
# Canary deployment
jobs:
  - deployment: CanaryDeploy
    environment: 'production'
    strategy:
      canary:
        increments: [10, 50]  # 10% then 50% then 100%
        deploy:
          steps:
            - script: echo "Deploying canary"
        routeTraffic:
          steps:
            - script: echo "Routing $(Strategy.Increment)% traffic"
        postRouteTraffic:
          steps:
            - script: echo "Running health checks"
        on:
          success:
            steps:
              - script: echo "Canary succeeded"
          failure:
            steps:
              - script: echo "Rolling back"
```

---

## 6.4 Environments and Approvals

```
┌────── ENVIRONMENT GATES ──────────────────┐
│                                            │
│  Environment: production                   │
│  ┌──────────────────────────────────────┐  │
│  │  Approvals:                         │  │
│  │  └── Approver: team-lead@company    │  │
│  │      Timeout: 72 hours              │  │
│  │                                     │  │
│  │  Checks:                            │  │
│  │  ├── Business hours only            │  │
│  │  ├── Required template check        │  │
│  │  ├── Azure Monitor alert check      │  │
│  │  └── REST API health check          │  │
│  │                                     │  │
│  │  Exclusive Lock: single deploy      │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Pipeline reaches Deploy_production        │
│       │                                    │
│       ▼                                    │
│  ⏸ Waiting for approval...                │
│       │                                    │
│       ▼ (approved)                         │
│  ✅ Checks pass → Deploy executes          │
│                                            │
└────────────────────────────────────────────┘
```

---

## 6.5 Multi-Stage Pipeline Pattern

```yaml
# Complete multi-stage pipeline
trigger:
  branches:
    include: [main]

variables:
  - group: 'shared-variables'
  - name: buildConfig
    value: 'Release'

stages:
  # ── BUILD ──
  - stage: Build
    jobs:
      - job: BuildAndTest
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
              arguments: '--configuration $(buildConfig)'
          - task: DotNetCoreCLI@2
            inputs:
              command: 'test'
              arguments: '--configuration $(buildConfig) --logger trx'
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/*.trx'
          - task: DotNetCoreCLI@2
            inputs:
              command: 'publish'
              arguments: '--configuration $(buildConfig) --output $(Build.ArtifactStagingDirectory)'
          - publish: $(Build.ArtifactStagingDirectory)
            artifact: drop

  # ── DEPLOY DEV ──
  - stage: DeployDev
    dependsOn: Build
    jobs:
      - deployment: Deploy
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'azure-dev'
                    appName: 'myapp-dev'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'

  # ── DEPLOY STAGING ──
  - stage: DeployStaging
    dependsOn: DeployDev
    jobs:
      - deployment: Deploy
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'azure-staging'
                    appName: 'myapp-staging'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'

  # ── DEPLOY PRODUCTION (with approval) ──
  - stage: DeployProd
    dependsOn: DeployStaging
    condition: eq(variables['Build.SourceBranchName'], 'main')
    jobs:
      - deployment: Deploy
        environment: 'production'  # Has approval gate configured
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'azure-prod'
                    appName: 'myapp-prod'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

## 6.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Template not found | Wrong path or file not committed | Verify relative path from pipeline YAML |
| Expression syntax error | Missing `${{ }}` or wrong function | Check docs for compile vs runtime syntax |
| Stage skipped | `condition` evaluated to false | Debug with `dependsOn` and conditions |
| Approval timeout | Approver didn't respond in time | Increase timeout or add backup approvers |
| Artifact not downloaded | Wrong artifact name in `download` | Match name used in `publish` step |
| Environment not found | Environment not created in project | Create environment in Pipelines > Environments |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Templates** | Reuse steps, jobs, stages across pipelines |
| **Expressions** | `${{ }}` (compile), `$[ ]` (runtime), `$()` (macro) |
| **Conditions** | `succeeded()`, `always()`, `eq()`, `and()`, `or()` |
| **Deployment Strategies** | runOnce, rolling, canary |
| **Environments** | Named targets with approvals and checks |
| **Multi-Stage** | Build → Dev → Staging → Prod in one YAML |

---

## Quick Revision Questions

1. **What are the three types of expressions in YAML pipelines?**
   > Compile-time `${{ }}` (evaluated before run), runtime `$[ ]` (evaluated during run), and macro `$()` (variable substitution at task time).

2. **What is a pipeline template?**
   > A reusable YAML file containing steps, jobs, or stages with parameters, allowing DRY (Don't Repeat Yourself) pipeline definitions.

3. **What is a canary deployment?**
   > Deploying to a small percentage of targets first (e.g., 10%), monitoring health, then gradually increasing to 100%.

4. **How do environment approvals work?**
   > When a pipeline deploys to an environment with approvals, it pauses until designated approvers approve (or the timeout expires).

5. **What's the difference between `dependsOn` and `condition`?**
   > `dependsOn` defines stage ordering (which stages must complete first). `condition` determines whether the stage actually runs based on expressions.

6. **How do you pass artifacts between stages?**
   > Use `publish` in the build stage and `download: current` with the artifact name in deployment stages.

---

[⬅ Previous: Azure Artifacts](05-azure-artifacts.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Monitor ➡](../09-Monitoring-and-Logging/01-azure-monitor.md)
