# Chapter 6.3: Jenkins Plugins

[← Previous: Jenkinsfile & Pipeline Syntax](02-jenkinsfile-pipeline-syntax.md) | [Back to README](../README.md) | [Next: Jenkins Shared Libraries →](04-shared-libraries.md)

---

## Overview

**Plugins** are Jenkins' extensibility mechanism — they add new features, integrations, and capabilities to the core platform. With 1800+ plugins available, Jenkins can integrate with virtually any tool in the DevOps ecosystem.

---

## Plugin Architecture

```
  ┌──────────────────────────────────────────────────┐
  │            JENKINS PLUGIN ECOSYSTEM              │
  │                                                  │
  │  ┌────────────────────────────────────┐          │
  │  │         JENKINS CORE              │          │
  │  │  (Scheduling, UI, Security)       │          │
  │  └────────────────┬─────────────────┘          │
  │                   │                              │
  │     ┌─────────────┼─────────────┐               │
  │     │             │             │               │
  │  ┌──▼──┐      ┌──▼──┐      ┌──▼──┐            │
  │  │ SCM │      │Build│      │Notif│            │
  │  │Plugs│      │Plugs│      │Plugs│            │
  │  ├─────┤      ├─────┤      ├─────┤            │
  │  │ Git │      │Maven│      │Slack│            │
  │  │GitHub│     │Grad.│      │Email│            │
  │  │GitLab│     │npm  │      │Teams│            │
  │  └─────┘      └─────┘      └─────┘            │
  │                                                  │
  │  ┌──────┐     ┌──────┐     ┌──────┐            │
  │  │Cloud │     │Test  │     │Secur.│            │
  │  │Plugs │     │Plugs │     │Plugs │            │
  │  ├──────┤     ├──────┤     ├──────┤            │
  │  │Docker│     │JUnit │     │OWASP │            │
  │  │K8s   │     │JaCoCo│     │Cred. │            │
  │  │AWS   │     │Allure│     │Role  │            │
  │  └──────┘     └──────┘     └──────┘            │
  └──────────────────────────────────────────────────┘
```

---

## Essential Plugins by Category

### Source Code Management

| Plugin | Purpose | Usage |
|---|---|---|
| **Git** | Git repository integration | Checkout, branch detection |
| **GitHub Branch Source** | Multibranch for GitHub repos | Org folder scanning |
| **GitLab** | GitLab integration | Webhooks, merge requests |
| **Bitbucket** | Bitbucket integration | PR builds |

### Build Tools

| Plugin | Purpose | Usage |
|---|---|---|
| **Pipeline** | Jenkinsfile support | Core pipeline functionality |
| **Docker Pipeline** | Build in Docker containers | `agent { docker {} }` |
| **Kubernetes** | Dynamic agents on K8s | Pod-based agents |
| **NodeJS** | Node.js tool installer | Auto-install Node versions |
| **Pipeline Utility Steps** | File operations | `readJSON`, `writeFile`, etc. |

### Testing & Quality

| Plugin | Purpose | Usage |
|---|---|---|
| **JUnit** | Test result reporting | `junit 'reports/*.xml'` |
| **JaCoCo** | Java code coverage | Coverage reports |
| **Cobertura** | Code coverage | Multi-language coverage |
| **HTML Publisher** | Publish HTML reports | Coverage, test reports |
| **Warnings Next Gen** | Static analysis results | Lint/SAST reports |

### Notifications & Integrations

| Plugin | Purpose | Usage |
|---|---|---|
| **Slack Notification** | Slack messages | Build status alerts |
| **Email Extension** | Advanced email | Templated notifications |
| **Microsoft Teams** | Teams messages | Build notifications |

---

## Installing Plugins

```
  PLUGIN INSTALLATION METHODS
  ═══════════════════════════

  1. WEB UI
     Manage Jenkins → Plugins → Available Plugins
     Search → Select → Install

  2. CLI
     java -jar jenkins-cli.jar -s http://localhost:8080/ \
       install-plugin git pipeline-stage-view

  3. DOCKER (pre-installed)
     FROM jenkins/jenkins:lts
     RUN jenkins-plugin-cli --plugins \
       git \
       pipeline-stage-view \
       docker-workflow \
       kubernetes \
       blueocean

  4. CONFIGURATION AS CODE (JCasC)
     plugins.txt → jenkins-plugin-cli
```

### Docker with Pre-installed Plugins

```dockerfile
FROM jenkins/jenkins:lts

# Skip setup wizard
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

# Install plugins
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
```

```
# plugins.txt
git:latest
pipeline-stage-view:latest
docker-workflow:latest
kubernetes:latest
blueocean:latest
credentials-binding:latest
junit:latest
slack:latest
pipeline-utility-steps:latest
```

---

## Plugin Usage in Pipelines

```groovy
pipeline {
    agent any

    stages {
        // Git Plugin — checkout
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/org/repo.git',
                    credentialsId: 'github-creds'
            }
        }

        // Docker Pipeline Plugin — build in container
        stage('Build') {
            agent {
                docker { image 'node:20' }
            }
            steps {
                sh 'npm ci && npm run build'
            }
        }

        // JUnit Plugin — test reporting
        stage('Test') {
            steps {
                sh 'npm test -- --ci --reporters=jest-junit'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }

        // HTML Publisher Plugin — coverage report
        stage('Coverage') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'coverage/lcov-report',
                    reportFiles: 'index.html',
                    reportName: 'Coverage Report'
                ])
            }
        }

        // Pipeline Utility Steps — read/write files
        stage('Version') {
            steps {
                script {
                    def pkg = readJSON file: 'package.json'
                    echo "Version: ${pkg.version}"
                }
            }
        }
    }

    // Slack Notification Plugin
    post {
        success {
            slackSend channel: '#builds',
                      color: 'good',
                      message: "✓ ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded"
        }
        failure {
            slackSend channel: '#builds',
                      color: 'danger',
                      message: "✕ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED"
        }
    }
}
```

---

## Plugin Management Best Practices

```
  ┌──────────────────────────────────────────────────┐
  │       PLUGIN MANAGEMENT BEST PRACTICES           │
  │                                                  │
  │  1. MINIMIZE PLUGINS                             │
  │     Only install what you actually use            │
  │     More plugins = more attack surface            │
  │                                                  │
  │  2. PIN VERSIONS                                 │
  │     Use specific versions in plugins.txt          │
  │     Don't use :latest in production              │
  │                                                  │
  │  3. UPDATE REGULARLY                             │
  │     Security patches are frequent                 │
  │     Test updates in staging Jenkins first         │
  │                                                  │
  │  4. MONITOR SECURITY ADVISORIES                  │
  │     Jenkins publishes weekly advisories           │
  │     Watch: https://www.jenkins.io/security/       │
  │                                                  │
  │  5. USE CONFIGURATION AS CODE                    │
  │     Reproducible Jenkins setup                   │
  │     Version-controlled plugin list               │
  └──────────────────────────────────────────────────┘
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Plugin fails to install | Dependency conflict | Check required dependencies; update base plugins first |
| Pipeline step not recognized | Plugin not installed or wrong version | Verify plugin is installed and compatible |
| Jenkins won't start after plugin update | Incompatible plugin version | Start in safe mode; disable problematic plugin |
| "Method not permitted" error | Script security sandbox | Approve method in Manage Jenkins → Script Approval |
| Plugin causes memory leak | Buggy plugin version | Check changelogs; downgrade to stable version |
| Too many plugins slow Jenkins | Resource overhead | Audit and remove unused plugins |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Count** | 1800+ plugins available |
| **Install** | Web UI, CLI, Docker, JCasC |
| **Essential** | Git, Pipeline, Docker, JUnit, Credentials |
| **Management** | Pin versions, minimize count, update regularly |
| **Security** | Monitor advisories; use script approval |
| **Best practice** | Define plugins-as-code in `plugins.txt` |

---

## Quick Revision Questions

1. **What are Jenkins plugins?** *Extensions that add functionality to Jenkins — SCM integration, build tools, notifications, cloud providers, and more.*
2. **How do you install plugins in a Docker-based Jenkins?** *Using `jenkins-plugin-cli` with a `plugins.txt` file in the Dockerfile.*
3. **Name three essential Jenkins plugins.** *Git (SCM), Pipeline (Jenkinsfile support), and JUnit (test reporting).*
4. **Why should you minimize the number of plugins?** *Each plugin adds maintenance overhead, potential security vulnerabilities, and resource consumption.*
5. **What is the Pipeline Utility Steps plugin used for?** *File operations in pipeline scripts — such as `readJSON`, `readYaml`, `writeFile`, and `zip`.*
6. **What should you do before updating plugins?** *Test the update on a staging Jenkins instance, review changelogs, and check for known compatibility issues.*

---

[← Previous: Jenkinsfile & Pipeline Syntax](02-jenkinsfile-pipeline-syntax.md) | [Back to README](../README.md) | [Next: Jenkins Shared Libraries →](04-shared-libraries.md)
