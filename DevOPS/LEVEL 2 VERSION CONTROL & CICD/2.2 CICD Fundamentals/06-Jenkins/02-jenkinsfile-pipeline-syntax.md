# Chapter 6.2: Jenkinsfile & Pipeline Syntax

[← Previous: Jenkins Architecture](01-jenkins-architecture.md) | [Back to README](../README.md) | [Next: Jenkins Plugins →](03-jenkins-plugins.md)

---

## Overview

A **Jenkinsfile** defines your CI/CD pipeline as code, stored in your repository alongside application code. Jenkins supports two syntax styles: **Declarative** (structured, recommended) and **Scripted** (flexible, Groovy-based).

---

## Declarative vs Scripted

```
  DECLARATIVE                      SCRIPTED
  ═══════════                      ════════

  pipeline {                       node {
    agent any                        stage('Build') {
    stages {                           sh 'npm install'
      stage('Build') {               }
        steps {                      stage('Test') {
          sh 'npm install'             sh 'npm test'
        }                           }
      }                            }
    }
  }

  ✓ Structured, opinionated        ✓ Full Groovy power
  ✓ Easier to read/learn           ✓ Flexible logic
  ✓ Built-in validation            ✕ Harder to maintain
  ✓ Blue Ocean compatible          ✕ No structural validation
```

---

## Declarative Pipeline Structure

```groovy
pipeline {
    // WHERE to run
    agent any

    // ENVIRONMENT variables
    environment {
        APP_NAME = 'myapp'
        VERSION  = '1.0.0'
        REGISTRY = 'registry.example.com'
    }

    // BUILD options
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    // TRIGGER rules
    triggers {
        pollSCM('H/5 * * * *')     // Poll every 5 minutes
        cron('H 2 * * 1-5')        // Nightly weekdays
    }

    // BUILD parameters
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }

    // PIPELINE stages
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                sh 'npm test -- --ci --coverage'
            }
            post {
                always {
                    junit 'reports/**/*.xml'
                    publishHTML(target: [
                        reportDir: 'coverage/lcov-report',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh "docker build -t ${REGISTRY}/${APP_NAME}:${VERSION} ."
                sh "docker push ${REGISTRY}/${APP_NAME}:${VERSION}"
            }
        }
    }

    // POST-BUILD actions
    post {
        success {
            slackSend color: 'good', message: "Build #${env.BUILD_NUMBER} succeeded"
        }
        failure {
            slackSend color: 'danger', message: "Build #${env.BUILD_NUMBER} FAILED"
        }
        always {
            cleanWs()   // Clean workspace
        }
    }
}
```

---

## Key Directives

```
  ┌──────────────────────────────────────────────────────┐
  │         DECLARATIVE PIPELINE DIRECTIVES              │
  │                                                      │
  │  pipeline { }        Top-level wrapper               │
  │  ├── agent           Where to execute                │
  │  ├── environment     Environment variables           │
  │  ├── options         Build options/settings          │
  │  ├── parameters      User-input parameters           │
  │  ├── triggers        Automatic triggers              │
  │  ├── tools           Auto-install tools              │
  │  ├── stages          Container for stage blocks      │
  │  │   └── stage       A named phase of the pipeline   │
  │  │       ├── steps   Actual commands to run          │
  │  │       ├── when    Conditional execution           │
  │  │       ├── parallel Parallel sub-stages            │
  │  │       └── post    Per-stage post actions          │
  │  └── post            Post-build actions              │
  │      ├── always      Run regardless                  │
  │      ├── success     Run only on success             │
  │      ├── failure     Run only on failure             │
  │      ├── unstable    Run if tests failed             │
  │      └── changed     Run if status changed           │
  └──────────────────────────────────────────────────────┘
```

---

## Agent Types

```groovy
// No agent at pipeline level — specify per-stage
pipeline {
    agent none

    stages {
        stage('Build (Linux)') {
            agent { label 'linux' }
            steps { sh 'make build' }
        }

        stage('Build (Windows)') {
            agent { label 'windows' }
            steps { bat 'msbuild solution.sln' }
        }

        stage('Docker Build') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-21'
                    args '-v $HOME/.m2:/root/.m2'    // Cache deps
                }
            }
            steps { sh 'mvn package' }
        }
    }
}
```

---

## Conditional Execution (when)

```groovy
stage('Deploy to Production') {
    when {
        // Only on main branch
        branch 'main'

        // AND environment is prod
        environment name: 'DEPLOY_TO', value: 'production'

        // Common when conditions:
        // branch 'main'
        // branch pattern: 'release-*', comparator: 'GLOB'
        // tag 'v*'
        // expression { return params.DEPLOY == true }
        // changeset '**/*.java'
        // triggeredBy 'TimerTrigger'
        // not { branch 'dev' }
        // allOf { branch 'main'; environment name: 'ENV', value: 'prod' }
        // anyOf { branch 'main'; branch 'release-*' }
    }
    steps {
        sh './deploy.sh production'
    }
}
```

---

## Parallel Stages

```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            agent { label 'linux' }
            steps { sh 'npm run test:unit' }
        }
        stage('Integration Tests') {
            agent { label 'linux && docker' }
            steps { sh 'npm run test:integration' }
        }
        stage('Lint') {
            agent { label 'linux' }
            steps { sh 'npm run lint' }
        }
    }
}
```

---

## Credentials & Secrets

```groovy
pipeline {
    agent any

    environment {
        // Bind credentials to env variables
        DOCKER_CREDS = credentials('docker-hub-credentials')
        // Creates: DOCKER_CREDS (username:password)
        //          DOCKER_CREDS_USR (username)
        //          DOCKER_CREDS_PSW (password)
    }

    stages {
        stage('Docker Login') {
            steps {
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }

        stage('Use Secret File') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get pods'
                }
            }
        }

        stage('Use SSH Key') {
            steps {
                sshagent(['deploy-key']) {
                    sh 'ssh user@server "deploy.sh"'
                }
            }
        }
    }
}
```

---

## Scripted Pipeline

```groovy
// Scripted pipeline — full Groovy flexibility
node('linux') {
    def appVersion = ''

    try {
        stage('Checkout') {
            checkout scm
            appVersion = sh(
                script: 'cat package.json | jq -r .version',
                returnStdout: true
            ).trim()
            echo "Building version: ${appVersion}"
        }

        stage('Build') {
            sh 'npm ci'
            sh 'npm run build'
        }

        stage('Test') {
            try {
                sh 'npm test -- --ci'
            } catch (err) {
                currentBuild.result = 'UNSTABLE'
            } finally {
                junit 'reports/**/*.xml'
            }
        }

        if (env.BRANCH_NAME == 'main') {
            stage('Deploy') {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh "./deploy.sh ${appVersion}"
            }
        }

        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Always notify
        slackSend color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
                  message: "Build ${env.BUILD_NUMBER}: ${currentBuild.result}"
    }
}
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| "No such DSL method" | Missing plugin or wrong syntax | Install required plugin; check Declarative vs Scripted |
| Pipeline validation errors | Syntax error in Jenkinsfile | Use `Replay` to test; use pipeline linter |
| `sh` command fails | Missing tool on agent | Install tools; use `tools` directive or Docker agent |
| Credentials not found | Wrong ID or scope | Verify credential ID in Manage Jenkins → Credentials |
| Workspace conflicts | Concurrent builds | Use `disableConcurrentBuilds()` or custom workspace |
| `when` condition ignored | Wrong syntax/evaluation | Use `expression { return }` for complex conditions |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Declarative** | Structured, recommended, `pipeline { }` |
| **Scripted** | Full Groovy, `node { }`, flexible logic |
| **Jenkinsfile** | Pipeline-as-code, stored in repo |
| **Key directives** | agent, stages, steps, when, post, environment |
| **Credentials** | `credentials()`, `withCredentials()`, `sshagent()` |
| **Parallel** | `parallel { }` block for concurrent stages |

---

## Quick Revision Questions

1. **What is a Jenkinsfile?** *A text file defining a Jenkins pipeline as code, stored in the repository alongside application code.*
2. **What is the difference between Declarative and Scripted pipelines?** *Declarative is structured with `pipeline {}` — easier to read and validate. Scripted uses `node {}` with full Groovy flexibility.*
3. **What does the `agent` directive do?** *Specifies where the pipeline or stage should execute — on any agent, a labeled agent, in a Docker container, or Kubernetes pod.*
4. **How do you add conditional logic to a stage?** *Using the `when` directive with conditions like `branch 'main'`, `expression {}`, `changeset`, or `tag`.*
5. **How are secrets handled in Jenkins pipelines?** *Using `credentials()` in environment block or `withCredentials()` in steps — secrets are masked in logs.*
6. **What does the `post` section do?** *Defines actions to run after the pipeline or stage completes — with conditions like `always`, `success`, `failure`, `unstable`.*

---

[← Previous: Jenkins Architecture](01-jenkins-architecture.md) | [Back to README](../README.md) | [Next: Jenkins Plugins →](03-jenkins-plugins.md)
