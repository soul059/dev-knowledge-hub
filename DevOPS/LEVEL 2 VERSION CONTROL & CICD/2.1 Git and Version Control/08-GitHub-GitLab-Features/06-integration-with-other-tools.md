# Chapter 8.6: Integration with Other Tools

[← Previous: GitHub Pages](05-github-pages.md) | [Back to README](../README.md)

---

## Overview

Git and platforms like GitHub/GitLab integrate with a wide ecosystem of tools — from project management and communication to CI/CD, security scanning, and deployment. These integrations create **automated, connected workflows** that boost developer productivity.

---

## Integration Categories

```
┌──────────────────────────────────────────────────────────────┐
│       GIT INTEGRATION ECOSYSTEM                             │
│                                                              │
│                    ┌──────────────┐                          │
│                    │   Git Repo   │                          │
│                    │ (GitHub/GL)  │                          │
│                    └──────┬───────┘                          │
│                           │                                 │
│       ┌───────────┬───────┼───────┬───────────┐             │
│       │           │       │       │           │             │
│  ┌────▼────┐ ┌────▼────┐ ┌▼─────┐ ┌▼────────┐ ┌▼────────┐ │
│  │ Commun. │ │ Project │ │CI/CD │ │Security│ │ Deploy │ │
│  │         │ │ Mgmt    │ │      │ │        │ │        │ │
│  │ Slack   │ │ Jira    │ │Jenkins│ │Snyk   │ │AWS     │ │
│  │ Teams   │ │ Trello  │ │Circle│ │Depe-  │ │Azure   │ │
│  │ Discord │ │ Asana   │ │Travis│ │ndabot │ │GCP     │ │
│  │         │ │ Linear  │ │      │ │CodeQL │ │Vercel  │ │
│  └─────────┘ └─────────┘ └──────┘ └────────┘ └────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Communication Integrations

### Slack + GitHub

```
┌──────────────────────────────────────────────────────────────┐
│  SLACK + GITHUB INTEGRATION                                 │
│                                                              │
│  In Slack:                                                  │
│  /github subscribe owner/repo                               │
│                                                              │
│  Notifications:                                             │
│  • PR opened / merged / closed                              │
│  • Issues created / assigned                                │
│  • CI/CD status updates                                     │
│  • Deployments                                              │
│  • Code reviews requested                                   │
│                                                              │
│  Actions from Slack:                                        │
│  /github close owner/repo#42                                │
│  /github open owner/repo                                    │
│                                                              │
│  #dev-team channel:                                         │
│  ┌────────────────────────────────────────────┐             │
│  │ 🔔 @alice opened PR #87                     │             │
│  │ "feat: add user search"                    │             │
│  │ [View PR] [Review]                         │             │
│  └────────────────────────────────────────────┘             │
└──────────────────────────────────────────────────────────────┘
```

### Microsoft Teams

```
# Add GitHub connector to a Teams channel:
# Channel → Connectors → GitHub → Configure
# Select events: push, PR, issues, etc.
```

---

## Project Management Integrations

### Jira + GitHub/GitLab

```
┌──────────────────────────────────────────────────────────────┐
│  JIRA + GIT INTEGRATION                                    │
│                                                              │
│  Branch naming with Jira ticket:                            │
│  $ git switch -c feature/PROJ-123-add-search                │
│                                                              │
│  Commit with ticket reference:                              │
│  $ git commit -m "PROJ-123: implement search endpoint"      │
│                                                              │
│  Result in Jira:                                            │
│  ┌──────────────────────────────────────────┐               │
│  │ PROJ-123: Add search functionality       │               │
│  │                                          │               │
│  │ Development:                             │               │
│  │ • Branch: feature/PROJ-123-add-search    │               │
│  │ • 3 commits                              │               │
│  │ • 1 pull request (Open)                  │               │
│  │ • Build: ✅ Passing                       │               │
│  └──────────────────────────────────────────┘               │
│                                                              │
│  Smart Commits:                                             │
│  PROJ-123 #comment Fixed the search bug                     │
│  PROJ-123 #time 2h                                          │
│  PROJ-123 #done                                             │
└──────────────────────────────────────────────────────────────┘
```

---

## CI/CD Tool Integrations

### Jenkins + GitHub

```groovy
// Jenkinsfile
pipeline {
    agent any
    triggers {
        githubPush()   // Trigger on push
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'reports/*.xml'
                }
            }
        }
    }
    post {
        success {
            // Report status back to GitHub
            githubNotify status: 'SUCCESS', description: 'Build passed'
        }
        failure {
            githubNotify status: 'FAILURE', description: 'Build failed'
        }
    }
}
```

### External CI Tools

| Tool | Integration |
|------|-------------|
| **Jenkins** | GitHub plugin, webhooks, status checks |
| **CircleCI** | `.circleci/config.yml`, GitHub app |
| **Travis CI** | `.travis.yml`, GitHub integration |
| **TeamCity** | VCS root pointing to Git repo |
| **Azure Pipelines** | `azure-pipelines.yml`, GitHub connection |

---

## Security Integrations

### Dependabot (GitHub Built-in)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "monthly"
```

### CodeQL (GitHub Code Scanning)

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        language: ['javascript', 'python']
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

### Other Security Tools

| Tool | Purpose |
|------|---------|
| **Snyk** | Vulnerability scanning for dependencies |
| **SonarQube** | Code quality and security analysis |
| **Trivy** | Container and filesystem vulnerability scanner |
| **GitGuardian** | Detect secrets/credentials in code |
| **FOSSA** | License compliance checking |

---

## Cloud Deployment Integrations

```
┌──────────────────────────────────────────────────────────────┐
│  DEPLOY FROM GIT TO CLOUD                                   │
│                                                              │
│  Git Push → CI/CD → Deploy                                  │
│                                                              │
│  Platform        Integration Method                         │
│  ────────        ──────────────────                         │
│  AWS             CodePipeline, GitHub Actions + AWS CLI     │
│  Azure           Azure Pipelines, GitHub Actions + az cli   │
│  GCP             Cloud Build, GitHub Actions + gcloud       │
│  Vercel          Automatic deploy on push (zero config)     │
│  Netlify         Automatic deploy on push (zero config)     │
│  Heroku          GitHub integration or git push heroku main │
│  DigitalOcean    App Platform connects to GitHub repo       │
│  Railway         Automatic deploy from GitHub               │
└──────────────────────────────────────────────────────────────┘
```

### Vercel Example

```bash
# 1. Import GitHub repo in Vercel dashboard
# 2. Vercel automatically:
#    • Builds on every push to main
#    • Creates preview deployments for PRs
#    • Adds deployment status to PR checks

# PR Comment from Vercel bot:
# ┌─────────────────────────────────────────┐
# │ ✅ Preview: https://my-app-pr-42.vercel.app │
# │ 📊 Build: 45s | Size: 245 KB               │
# └─────────────────────────────────────────┘
```

---

## IDE Integrations

```
┌──────────────────────────────────────────────────────────────┐
│  GIT IN YOUR IDE                                            │
│                                                              │
│  VS Code:                                                   │
│  • Built-in Git support (Source Control panel)              │
│  • GitLens extension (blame, history, compare)              │
│  • GitHub Pull Requests extension                           │
│  • GitHub Copilot                                           │
│                                                              │
│  JetBrains (IntelliJ, WebStorm, PyCharm):                   │
│  • Built-in Git integration                                 │
│  • Diff viewer and merge tool                               │
│  • GitHub/GitLab PR support                                 │
│  • Git blame annotations                                    │
│                                                              │
│  CLI Tools:                                                 │
│  • GitHub CLI (gh): gh pr create, gh issue list             │
│  • GitLab CLI (glab): glab mr create, glab issue list      │
│  • lazygit: Terminal UI for Git                             │
│  • tig: Text-mode interface for Git                         │
└──────────────────────────────────────────────────────────────┘
```

### GitHub CLI Examples

```bash
# Create a PR from terminal
$ gh pr create --title "feat: add search" --body "Closes #42"

# List open PRs
$ gh pr list

# Check out a PR locally
$ gh pr checkout 87

# Create an issue
$ gh issue create --title "Bug: login fails" --label bug

# View repo in browser
$ gh repo view --web
```

---

## Webhooks (Custom Integrations)

```
┌──────────────────────────────────────────────────────────────┐
│  WEBHOOKS — Connect Git to anything!                        │
│                                                              │
│  GitHub Event ──► HTTP POST ──► Your Server                 │
│                                                              │
│  Settings → Webhooks → Add webhook                          │
│  • Payload URL: https://myserver.com/hooks/github           │
│  • Content type: application/json                           │
│  • Secret: shared-secret-for-verification                   │
│  • Events: Push, PR, Issues, etc.                           │
│                                                              │
│  Use cases:                                                 │
│  • Custom notifications                                     │
│  • Trigger external builds                                  │
│  • Update dashboards                                        │
│  • Sync with external systems                               │
│  • Auto-deploy on push                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Full Development Workflow

```
1. Developer picks Jira ticket PROJ-123
2. Creates branch: feature/PROJ-123-search
3. Pushes code → GitHub Actions runs CI
4. Opens PR → Slack notifies #dev-team
5. CodeQL runs security scan
6. Reviewer approves in VS Code (GitHub PR extension)
7. PR merged → Vercel auto-deploys
8. Jira ticket auto-transitions to "Done"
```

### Scenario 2: Monitoring Dependencies

```
1. Dependabot detects vulnerable dependency
2. Opens PR with version bump
3. CI runs tests on the PR
4. Developer reviews and merges
5. Slack notifies team about the update
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Webhook not firing | Check payload URL, verify events selected, check secret |
| Jira not linking commits | Include ticket ID in branch name or commit message |
| Slack notifications too noisy | Filter events: `/github subscribe owner/repo reviews comments` |
| Deployment failing | Check CI logs, verify secrets/credentials, check permissions |
| Integration permissions error | Re-authorize the GitHub/GitLab app; check OAuth scopes |

---

## Summary Table

| Category | Tools |
|----------|-------|
| Communication | Slack, Teams, Discord |
| Project Management | Jira, Trello, Asana, Linear |
| CI/CD (External) | Jenkins, CircleCI, Travis CI |
| Security | Dependabot, Snyk, CodeQL, SonarQube |
| Deployment | Vercel, Netlify, AWS, Azure, GCP |
| IDE | VS Code, JetBrains, GitLens |
| CLI | GitHub CLI (gh), GitLab CLI (glab), lazygit |
| Custom | Webhooks, GitHub Apps, APIs |

---

## Quick Revision Questions

1. **What are the main categories of Git integrations?**
   <details><summary>Answer</summary>Communication (Slack, Teams), Project Management (Jira, Linear), CI/CD (Jenkins, CircleCI), Security (Dependabot, Snyk, CodeQL), Cloud Deployment (Vercel, AWS, Netlify), IDE (VS Code, JetBrains), and Custom (Webhooks, APIs).</details>

2. **How does Jira integrate with Git?**
   <details><summary>Answer</summary>By including Jira ticket IDs (e.g., PROJ-123) in branch names, commit messages, and PR titles. Jira automatically links these to the ticket, showing branches, commits, PRs, and build status. Smart Commits allow logging time and transitioning tickets from commit messages.</details>

3. **What is Dependabot?**
   <details><summary>Answer</summary>Dependabot is a GitHub built-in tool that automatically checks dependencies for known vulnerabilities and outdated versions. It opens PRs with version bumps, runs CI on those PRs, and supports multiple ecosystems (npm, pip, Docker, etc.). Configured via `.github/dependabot.yml`.</details>

4. **What is a webhook and when would you use one?**
   <details><summary>Answer</summary>A webhook is an HTTP callback — when an event occurs in GitHub/GitLab (push, PR, issue), it sends an HTTP POST to a URL you specify. Use webhooks for custom integrations: triggering external builds, updating dashboards, syncing with external systems, or custom notifications.</details>

5. **Name three ways GitHub CLI (`gh`) improves workflow.**
   <details><summary>Answer</summary>1) Create PRs from the terminal without opening a browser (`gh pr create`). 2) Check out PRs locally for testing (`gh pr checkout 87`). 3) Create and manage issues from the command line (`gh issue create`). It also supports viewing CI status, managing repos, and running GitHub Actions workflows.</details>

6. **How do platforms like Vercel integrate with GitHub?**
   <details><summary>Answer</summary>Vercel connects to a GitHub repository and automatically: builds on every push to the main branch, creates preview deployments for every PR (with unique URLs), adds deployment status checks to PRs, and provides instant rollback. It requires minimal configuration (often zero-config for popular frameworks).</details>

---

[← Previous: GitHub Pages](05-github-pages.md) | [Back to README](../README.md)
