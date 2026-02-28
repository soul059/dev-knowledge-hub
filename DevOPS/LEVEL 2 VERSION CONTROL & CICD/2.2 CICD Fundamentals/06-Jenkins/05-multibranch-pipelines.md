# Chapter 6.5: Multibranch Pipelines

[← Previous: Shared Libraries](04-shared-libraries.md) | [Back to README](../README.md) | [Next: Jenkins Security →](06-jenkins-security.md)

---

## Overview

**Multibranch Pipelines** automatically discover branches (and pull requests) in a source repository and create pipeline jobs for each one. This enables true CI where every branch and PR gets its own build.

---

## How Multibranch Pipelines Work

```
  GIT REPOSITORY                     JENKINS
  ══════════════                     ═══════

  branches/                          Multibranch Pipeline: "my-app"
  ├── main         ──────────────►   ├── main        (Pipeline Job)
  ├── develop      ──────────────►   ├── develop     (Pipeline Job)
  ├── feature/auth ──────────────►   ├── feature/auth(Pipeline Job)
  └── PR #42       ──────────────►   └── PR-42       (Pipeline Job)

         ┌──────────────────────────────────┐
         │  Branch Source (SCM) configured   │
         │  Jenkins scans periodically       │
         │  New branch? → Create job         │
         │  Branch deleted? → Remove job     │
         │  Each branch uses its Jenkinsfile │
         └──────────────────────────────────┘
```

---

## Configuring Multibranch Pipeline

### Via Jenkins UI

1. **New Item** → Enter name → Select **Multibranch Pipeline**
2. Configure **Branch Sources**:
   - GitHub, Bitbucket, GitLab, or plain Git
   - Add credentials
   - Set repository URL
3. **Build Configuration**:
   - Mode: "by Jenkinsfile"
   - Script Path: `Jenkinsfile` (default)
4. **Scan Triggers**:
   - Periodic scan interval (e.g., every 1 minute)
   - Webhook-triggered (recommended)

### Via Jenkins Configuration as Code (JCasC)

```yaml
# jenkins.yaml
jobs:
  - script: >
      multibranchPipelineJob('my-app') {
        branchSources {
          github {
            id('my-app-github')
            repoOwner('my-org')
            repository('my-app')
            credentialsId('github-token')
          }
        }
        factory {
          workflowBranchProjectFactory {
            scriptPath('Jenkinsfile')
          }
        }
        triggers {
          periodicFolderTrigger {
            interval('2m')
          }
        }
        orphanedItemStrategy {
          discardOldItems {
            numToKeep(10)
          }
        }
      }
```

---

## Branch-Specific Behavior

```groovy
// Jenkinsfile — adapts behavior based on branch
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test -- --ci'
            }
        }

        stage('Integration Tests') {
            when { anyOf { branch 'main'; branch 'develop' } }
            steps {
                sh 'npm run test:integration'
            }
        }

        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps {
                sh './deploy.sh staging'
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                input message: 'Deploy to production?'
                sh './deploy.sh production'
            }
        }
    }

    post {
        always { junit 'reports/**/*.xml' }
    }
}
```

```
  BRANCH-BASED STAGE EXECUTION
  ═════════════════════════════

  feature/* :  Build → Unit Tests
  develop   :  Build → Unit Tests → Integration Tests → Deploy Staging
  main      :  Build → Unit Tests → Integration Tests → Deploy Production
  PR-*      :  Build → Unit Tests
```

---

## Pull Request Builds

```groovy
// Jenkinsfile with PR-specific logic
pipeline {
    agent any

    stages {
        stage('Build') {
            steps { sh 'make build' }
        }

        stage('Test') {
            steps { sh 'make test' }
        }

        stage('PR Checks') {
            when { changeRequest() }
            steps {
                // Run additional checks only for PRs
                sh 'make lint'
                sh 'make security-scan'
            }
        }

        stage('Report PR Status') {
            when { changeRequest() }
            steps {
                echo "PR #${env.CHANGE_ID}: ${env.CHANGE_TITLE}"
                echo "Author: ${env.CHANGE_AUTHOR}"
                echo "Target: ${env.CHANGE_TARGET}"
            }
        }
    }
}
```

### PR Environment Variables

| Variable | Description |
|---|---|
| `CHANGE_ID` | PR / MR number |
| `CHANGE_URL` | URL of the pull request |
| `CHANGE_TITLE` | Title of the pull request |
| `CHANGE_AUTHOR` | Author of the pull request |
| `CHANGE_TARGET` | Target branch (e.g., main) |
| `BRANCH_NAME` | Source branch name |

---

## Organization Folders

```
  ORGANIZATION FOLDER
  ═══════════════════

  GitHub Organization: "my-org"
  └── Organization Folder Job in Jenkins
      ├── repo-a/           (Multibranch Pipeline)
      │   ├── main
      │   ├── develop
      │   └── PR-5
      ├── repo-b/           (Multibranch Pipeline)
      │   ├── main
      │   └── feature/x
      └── repo-c/           (Multibranch Pipeline)
          └── main

  → Automatically discovers ALL repos with a Jenkinsfile
  → Creates multibranch pipelines per repository
  → Great for microservices / many-repo setups
```

```yaml
# JCasC: Organization Folder
jobs:
  - script: >
      organizationFolder('my-org') {
        organizations {
          github {
            repoOwner('my-org')
            credentialsId('github-token')
          }
        }
        projectFactories {
          workflowMultiBranchProjectFactory {
            scriptPath('Jenkinsfile')
          }
        }
      }
```

---

## Branch Discovery & Filtering

```groovy
// Discover specific branches only
multibranchPipelineJob('my-app') {
    branchSources {
        git {
            remote('https://github.com/my-org/my-app.git')
            credentialsId('github-token')
            includes('main develop release/*')
            excludes('feature/experimental-*')
        }
    }
}
```

### Trait Configuration

| Trait | Purpose |
|---|---|
| Discover branches | Find branches in the repo |
| Discover PRs (origin) | Find PRs from repo collaborators |
| Discover PRs (forks) | Find PRs from forks (trust settings) |
| Filter by name (regex) | Include/exclude branches by pattern |
| Clean before checkout | Fresh workspace per build |
| Suppress SCM trigger | Disable automatic builds for certain branches |

---

## Webhooks for Instant Triggers

```
  WEBHOOK FLOW
  ════════════

  Developer pushes to branch
         │
         ▼
  GitHub sends webhook → Jenkins /github-webhook/
         │
         ▼
  Jenkins identifies branch → Triggers pipeline job
         │
         ▼
  Pipeline runs Jenkinsfile from that branch
         │
         ▼
  Status reported back to GitHub commit
```

### GitHub Webhook Setup

1. Repository → Settings → Webhooks → Add webhook
2. Payload URL: `https://jenkins.example.com/github-webhook/`
3. Content type: `application/json`
4. Events: "Pushes" and "Pull requests"
5. Install **GitHub Branch Source** plugin in Jenkins

---

## Real-World Scenario

> **Scenario**: A team with 5 microservices wants each service to auto-build feature branches and PRs.
>
> **Solution**: Create an **Organization Folder** pointing to the GitHub organization. Jenkins auto-discovers all repos with a `Jenkinsfile`. Each repo uses the shared library for a standard pipeline. Feature branches run build + test. PRs get extra lint + security checks. Only `main` deploys to production.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Branch not discovered | Scan hasn't run | Trigger "Scan Now" or reduce scan interval |
| Old branches remain | Orphan strategy not set | Configure orphaned item strategy (e.g., keep 30 days) |
| PR builds not triggered | Missing GitHub plugin/webhook | Install GitHub Branch Source + configure webhook |
| Wrong Jenkinsfile used | Script path incorrect | Verify "Script Path" in Build Configuration |
| Too many branches building | No filter | Add branch filter (include/exclude pattern) |
| Credential errors | Token expired or insufficient scope | Regenerate token with `repo` scope |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Auto-discover branches/PRs and create pipeline jobs |
| **Discovery** | Periodic scan or webhook-triggered |
| **Jenkinsfile** | Each branch uses its own Jenkinsfile from that branch |
| **PRs** | Build and report status back to SCM |
| **Organization Folders** | Auto-discover all repos in a GitHub org |
| **Filtering** | Include/exclude branches by name pattern |

---

## Quick Revision Questions

1. **What is a Multibranch Pipeline?** *A Jenkins job type that automatically discovers branches and PRs in a repository and creates a pipeline job for each one.*
2. **How does Jenkins discover new branches?** *Through periodic SCM scanning or webhook notifications from the SCM provider.*
3. **How do you run different stages per branch?** *Use the `when { branch 'name' }` directive to conditionally execute stages.*
4. **What is an Organization Folder?** *A Jenkins job that discovers all repositories in a GitHub/GitLab organization and creates multibranch pipelines for each repo with a Jenkinsfile.*
5. **How do you handle stale branches?** *Configure the orphaned item strategy to automatically remove jobs for deleted branches after a set period.*
6. **What environment variable holds the PR number?** *`CHANGE_ID` — available when the build is triggered by a pull request.*

---

[← Previous: Shared Libraries](04-shared-libraries.md) | [Back to README](../README.md) | [Next: Jenkins Security →](06-jenkins-security.md)
