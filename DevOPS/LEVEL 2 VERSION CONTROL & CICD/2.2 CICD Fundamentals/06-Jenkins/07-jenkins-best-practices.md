# Chapter 6.7: Jenkins Best Practices

[← Previous: Jenkins Security](06-jenkins-security.md) | [Back to README](../README.md) | [Next: GitHub Actions Workflow Syntax →](../07-GitHub-Actions/01-workflow-syntax.md)

---

## Overview

Jenkins is powerful but can become difficult to manage without discipline. This chapter covers best practices for pipelines, administration, scaling, and maintenance.

---

## Pipeline Best Practices

### 1. Use Declarative Pipelines

```groovy
// ✓ GOOD: Declarative — structured, readable, validated
pipeline {
    agent { docker { image 'node:20-alpine' } }

    stages {
        stage('Build') {
            steps { sh 'npm ci && npm run build' }
        }
        stage('Test') {
            steps { sh 'npm test -- --ci' }
        }
    }
}

// ✕ AVOID: Scripted — harder to maintain, no validation
node {
    stage('Build') {
        sh 'npm ci && npm run build'
    }
    stage('Test') {
        sh 'npm test -- --ci'
    }
}
```

### 2. Keep Jenkinsfiles Short

```
  IDEAL JENKINSFILE STRUCTURE
  ═══════════════════════════

  Jenkinsfile (5-20 lines)
  └── Imports shared library
      └── Calls standardPipeline() with config
          └── Library handles all logic

  INSTEAD OF:
  Jenkinsfile (200+ lines)
  └── Inline build, test, deploy, notify logic
      └── Duplicated across every repo
```

```groovy
// ✓ GOOD: Short Jenkinsfile using shared library
@Library('my-shared-lib') _

standardPipeline(
    appName: 'user-service',
    language: 'node',
    deployTo: ['staging', 'production'],
    slackChannel: '#team-backend'
)
```

### 3. Clean Workspaces

```groovy
pipeline {
    agent any

    options {
        // Automatically clean workspace before checkout
        skipDefaultCheckout()
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        // ...
    }

    post {
        always { cleanWs() }
    }
}
```

### 4. Use Timeouts and Retry

```groovy
pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')  // Global timeout
        retry(2)                              // Retry entire pipeline
    }

    stages {
        stage('Flaky Integration Test') {
            options {
                timeout(time: 10, unit: 'MINUTES')
                retry(3)
            }
            steps {
                sh 'npm run test:integration'
            }
        }

        stage('Deploy') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Approve deployment?'
                }
                sh './deploy.sh'
            }
        }
    }
}
```

### 5. Fail Fast

```groovy
pipeline {
    agent any

    stages {
        stage('Parallel Tests') {
            failFast true
            parallel {
                stage('Unit') { steps { sh 'npm run test:unit' } }
                stage('Lint') { steps { sh 'npm run lint' } }
                stage('Security') { steps { sh 'npm audit' } }
            }
        }
    }
}
```

---

## Administration Best Practices

### Jenkins Configuration as Code (JCasC)

```
  CONFIGURATION MANAGEMENT
  ═════════════════════════

  ANTI-PATTERN:              BEST PRACTICE:
  Manual UI configuration    JCasC (Configuration as Code)
  ├── Not reproducible       ├── Version controlled
  ├── No audit trail         ├── Code review for changes
  ├── Snowflake instances    ├── Reproducible setup
  └── Disaster recovery?     └── Easy disaster recovery
```

```yaml
# jenkins.yaml — Complete JCasC example
jenkins:
  systemMessage: "Jenkins configured via JCasC"
  numExecutors: 0
  securityRealm:
    github:
      clientID: "${GITHUB_CLIENT_ID}"
      clientSecret: "${GITHUB_CLIENT_SECRET}"
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions: ["Overall/Administer"]
            entries:
              - group: "jenkins-admins"
          - name: "developer"
            permissions: ["Overall/Read", "Job/Build", "Job/Read"]
            entries:
              - group: "developers"

  clouds:
    - kubernetes:
        name: "k8s"
        serverUrl: "https://kubernetes.default"
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins:8080"
        templates:
          - name: "default"
            label: "k8s-agent"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
                resourceRequestCpu: "500m"
                resourceRequestMemory: "512Mi"

unclassified:
  location:
    url: "https://jenkins.example.com/"
  slackNotifier:
    teamDomain: "myteam"
    tokenCredentialId: "slack-token"

tool:
  nodejs:
    installations:
      - name: "Node 20"
        properties:
          - installSource:
              installers:
                - nodeJSInstaller:
                    id: "20.11.0"
```

### Backup Strategy

```
  WHAT TO BACK UP
  ════════════════

  $JENKINS_HOME/
  ├── config.xml                ← Jenkins global config
  ├── credentials.xml           ← Encrypted credentials
  ├── secrets/                  ← Master key & secrets
  │   ├── master.key            ← CRITICAL (needed to decrypt everything)
  │   └── hudson.util.Secret    ← CRITICAL
  ├── jobs/                     ← Job configurations & build history
  │   └── */config.xml
  ├── users/                    ← User configs
  ├── plugins/                  ← Installed plugins
  └── jenkins.yaml              ← JCasC file (if used)

  BACKUP STRATEGY:
  • Daily backup of $JENKINS_HOME
  • Exclude: workspace/, builds/ (large, reproducible)
  • Store backups offsite (S3, Azure Blob, GCS)
  • Test restore procedure quarterly
  • Use ThinBackup plugin for scheduled backups
```

```bash
#!/bin/bash
# backup-jenkins.sh — Simple backup script
JENKINS_HOME="/var/lib/jenkins"
BACKUP_DIR="/backups/jenkins"
DATE=$(date +%Y%m%d_%H%M%S)

tar czf "${BACKUP_DIR}/jenkins_${DATE}.tar.gz" \
    --exclude='workspace' \
    --exclude='*/builds/*/archive' \
    --exclude='*/builds/*/log' \
    "${JENKINS_HOME}"

# Keep last 30 days
find "${BACKUP_DIR}" -name "jenkins_*.tar.gz" -mtime +30 -delete

# Upload to S3
aws s3 cp "${BACKUP_DIR}/jenkins_${DATE}.tar.gz" \
    s3://my-backups/jenkins/
```

---

## Scaling Jenkins

```
  SCALING STRATEGIES
  ══════════════════

  1. VERTICAL SCALING
     ├── More CPU/RAM for controller
     └── Limited ceiling

  2. HORIZONTAL SCALING (Agents)
     ├── Static agents (permanent VMs)
     ├── Cloud agents (AWS EC2, Azure VMs)
     └── Kubernetes agents (best for autoscaling)

  3. HIGH AVAILABILITY
     ├── Jenkins HA (CloudBees)
     ├── Active-passive failover
     └── Shared storage (EFS/NFS)

  KUBERNETES SCALING FLOW:
  ┌──────────┐  Build queued  ┌──────────────┐
  │Controller│──────────────►│ K8s API       │
  │          │               │ Create Pod    │
  └──────────┘               └──────┬───────┘
                                    │
                              ┌─────▼──────┐
                              │ Agent Pod   │
                              │ Runs build  │
                              │ Auto-deleted│
                              └────────────┘
```

### Kubernetes Pod Templates

```groovy
// Dynamic pod template in Jenkinsfile
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:20-alpine
    command: ['sleep', '3600']
    resources:
      requests:
        cpu: '1'
        memory: '2Gi'
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-storage
      mountPath: /var/lib/docker
  volumes:
  - name: docker-storage
    emptyDir: {}
"""
        }
    }

    stages {
        stage('Build') {
            steps {
                container('node') {
                    sh 'npm ci && npm run build'
                }
            }
        }

        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

---

## Monitoring Jenkins

```
  KEY METRICS TO MONITOR
  ═══════════════════════

  Controller Health:
  ├── CPU / Memory usage
  ├── Disk space ($JENKINS_HOME)
  ├── Thread count
  └── Plugin load times

  Build Health:
  ├── Queue length (waiting builds)
  ├── Build duration trends
  ├── Success/failure rates
  ├── Agent utilization
  └── Executor idle time

  TOOLS:
  • Prometheus + Grafana (prometheus plugin)
  • Datadog Jenkins plugin
  • Jenkins built-in monitoring (/monitoring)
```

### Prometheus Metrics

```yaml
# prometheus.yml scrape config
scrape_configs:
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['jenkins.example.com:8080']
    bearer_token_file: /etc/prometheus/jenkins-token
```

---

## Plugin Management

```
  PLUGIN BEST PRACTICES
  ═════════════════════

  ✓ Install only what you need
  ✓ Pin plugin versions in Dockerfile
  ✓ Test updates in staging Jenkins first
  ✓ Use recommended plugins for common tasks
  ✓ Check plugin compatibility before Jenkins upgrades
  ✕ Don't install abandoned/unmaintained plugins
  ✕ Don't auto-update in production
```

```dockerfile
# Dockerfile: Pin plugin versions
FROM jenkins/jenkins:lts

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
```

```text
# plugins.txt — Version-pinned
workflow-aggregator:596.v8c21c963d92d
git:5.2.0
docker-workflow:572.v950f58993843
kubernetes:4029.v5712230ccb_f8
configuration-as-code:1775.v810dc950b_514
role-strategy:689.v731678c3e0eb_
credentials-binding:677.vdc9c38cb_15e0
slack:684.v833089650554
prometheus:2.2.3
```

---

## Pipeline Anti-Patterns

```
  ┌─────────────────────────────────┬────────────────────────────────────┐
  │ ANTI-PATTERN                    │ BEST PRACTICE                      │
  ├─────────────────────────────────┼────────────────────────────────────┤
  │ Long inline Jenkinsfiles        │ Shared libraries + short configs   │
  │ Building on controller          │ Always use agents                  │
  │ Manual UI configuration         │ JCasC + version control            │
  │ Storing secrets in code         │ Jenkins credential store / Vault   │
  │ No timeouts                     │ Set global + stage timeouts        │
  │ Ignoring failed tests           │ Fail pipeline on test failures     │
  │ No workspace cleanup            │ cleanWs() in post { always {} }    │
  │ One agent for all builds        │ Tool-specific agents (Docker/K8s)  │
  │ Unbounded build history         │ Discard old builds (keep 30 days)  │
  │ No notifications                │ Slack/email on failure             │
  └─────────────────────────────────┴────────────────────────────────────┘
```

---

## Build History Management

```groovy
pipeline {
    agent any

    options {
        buildDiscarder(logRotator(
            numToKeepStr: '20',       // Keep last 20 builds
            daysToKeepStr: '30',      // Keep builds for 30 days
            artifactNumToKeepStr: '5' // Keep artifacts from last 5
        ))
        timestamps()                   // Add timestamps to console
        disableConcurrentBuilds()      // Prevent parallel runs
    }

    // ...
}
```

---

## Real-World Scenario

> **Scenario**: A company is migrating from a manually-configured Jenkins to an infrastructure-as-code approach.
>
> **Solution**: Export current config using JCasC export. Store `jenkins.yaml` in Git. Use a Docker image with pinned plugins. Deploy Jenkins on Kubernetes with Helm chart. Configure Kubernetes cloud for dynamic agents. Set up Prometheus monitoring. Implement daily backups to S3. Use shared libraries for all pipeline logic. Result: Repeatable setup, disaster recovery in minutes, auto-scaling agents.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Builds stuck in queue | No available executors | Add agents; check agent connectivity |
| Jenkins running out of disk | Build history/workspace accumulation | Configure log rotation; `cleanWs()` in pipelines |
| Slow Jenkins UI | Too many plugins / large build history | Remove unused plugins; discard old builds |
| Plugin conflicts after update | Incompatible versions | Test in staging; pin versions in `plugins.txt` |
| JCasC changes not applied | Config not reloaded | Reload via Manage Jenkins → Configuration as Code |
| Agent disconnects | Network timeout / resource limits | Increase timeout; check agent resource allocation |

---

## Summary Table

| Aspect | Recommendation |
|---|---|
| **Pipelines** | Declarative + shared libraries + short Jenkinsfiles |
| **Configuration** | JCasC (version controlled, reproducible) |
| **Agents** | Ephemeral K8s/Docker agents, 0 executors on controller |
| **Backups** | Daily, exclude workspace, test restores |
| **Plugins** | Pin versions, test updates, minimize count |
| **Monitoring** | Prometheus + Grafana, track queue length and durations |
| **Scaling** | Kubernetes cloud for auto-scaling agents |

---

## Quick Revision Questions

1. **Why use Declarative over Scripted pipelines?** *Declarative pipelines are structured, validated before execution, easier to read, and support the Blue Ocean UI — use Scripted only when Declarative cannot express the logic.*
2. **What is JCasC?** *Jenkins Configuration as Code — defining all Jenkins configuration in YAML files, stored in version control, for reproducible and auditable setup.*
3. **What should you back up in Jenkins?** *`config.xml`, `credentials.xml`, `secrets/` (master key), `jobs/` configs, `plugins/`, and JCasC files. Exclude workspaces and large build archives.*
4. **How do you scale Jenkins builds?** *Use Kubernetes cloud plugin for dynamic agents — pods are created on demand, run the build, and are destroyed automatically.*
5. **What is the recommended executor count on the controller?** *Zero — all builds should run on agents to protect the controller from resource exhaustion and security risks.*
6. **Name three pipeline anti-patterns.** *Building on the controller, long inline Jenkinsfiles (instead of shared libraries), and not setting timeouts on pipelines/stages.*

---

[← Previous: Jenkins Security](06-jenkins-security.md) | [Back to README](../README.md) | [Next: GitHub Actions Workflow Syntax →](../07-GitHub-Actions/01-workflow-syntax.md)
