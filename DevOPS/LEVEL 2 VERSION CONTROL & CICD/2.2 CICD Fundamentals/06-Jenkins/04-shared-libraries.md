# Chapter 6.4: Jenkins Shared Libraries

[← Previous: Jenkins Plugins](03-jenkins-plugins.md) | [Back to README](../README.md) | [Next: Multibranch Pipelines →](05-multibranch-pipelines.md)

---

## Overview

**Shared Libraries** allow you to define reusable pipeline code in a separate Git repository and import it into any Jenkinsfile. This eliminates duplication across projects and centralizes pipeline logic.

---

## Why Shared Libraries?

```
  WITHOUT SHARED LIBRARIES          WITH SHARED LIBRARIES
  ═════════════════════════         ══════════════════════

  Repo A: Jenkinsfile               Shared Library (Git repo):
  ├── buildDocker()  ← duplicated   ├── vars/buildDocker.groovy
  ├── runTests()     ← duplicated   ├── vars/runTests.groovy
  └── deploy()       ← duplicated   └── vars/deploy.groovy

  Repo B: Jenkinsfile               Repo A: Jenkinsfile
  ├── buildDocker()  ← duplicated   └── @Library('shared') _
  ├── runTests()     ← duplicated       buildDocker()
  └── deploy()       ← duplicated       runTests()
                                         deploy()
  Repo C: Jenkinsfile
  ├── buildDocker()  ← duplicated   Repo B: Jenkinsfile
  ├── runTests()     ← duplicated   └── @Library('shared') _
  └── deploy()       ← duplicated       buildDocker()
                                         runTests()
  FIX A BUG: Change 10+ repos              deploy()
  WITH LIBRARY: Change 1 repo
```

---

## Library Structure

```
  shared-library/                    (Git repository)
  ├── vars/                          Global pipeline steps
  │   ├── buildDocker.groovy         → call as buildDocker()
  │   ├── runTests.groovy            → call as runTests()
  │   ├── deploy.groovy              → call as deploy()
  │   └── standardPipeline.groovy    → call as standardPipeline()
  ├── src/                           Class-based code (optional)
  │   └── com/
  │       └── example/
  │           ├── Docker.groovy      Utility classes
  │           └── SlackNotifier.groovy
  ├── resources/                     Static files
  │   ├── scripts/
  │   │   └── deploy.sh
  │   └── templates/
  │       └── email.html
  └── README.md
```

---

## Creating Shared Steps (vars/)

### Simple Step

```groovy
// vars/buildDocker.groovy
def call(String imageName, String tag = 'latest') {
    sh "docker build -t ${imageName}:${tag} ."
    sh "docker push ${imageName}:${tag}"
}
```

### Step with Closure Body

```groovy
// vars/withNotification.groovy
def call(String channel = '#builds', Closure body) {
    try {
        body()
        slackSend channel: channel, color: 'good',
                  message: "✓ ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded"
    } catch (Exception e) {
        slackSend channel: channel, color: 'danger',
                  message: "✕ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED: ${e.message}"
        throw e
    }
}
```

### Step with Map Parameters

```groovy
// vars/deploy.groovy
def call(Map config = [:]) {
    def environment = config.environment ?: 'dev'
    def image = config.image
    def namespace = config.namespace ?: 'default'

    echo "Deploying ${image} to ${environment} (namespace: ${namespace})"

    sh """
        kubectl set image deployment/${config.app} \
            ${config.app}=${image} \
            -n ${namespace}
        kubectl rollout status deployment/${config.app} \
            -n ${namespace} --timeout=300s
    """
}
```

---

## Full Reusable Pipeline

```groovy
// vars/standardPipeline.groovy
def call(Map config) {
    pipeline {
        agent { docker { image config.buildImage ?: 'node:20' } }

        environment {
            REGISTRY = config.registry ?: 'registry.example.com'
            APP_NAME = config.appName
        }

        stages {
            stage('Install') {
                steps { sh 'npm ci' }
            }

            stage('Lint') {
                steps { sh 'npm run lint' }
            }

            stage('Test') {
                steps { sh 'npm test -- --ci --coverage' }
                post {
                    always {
                        junit 'reports/**/*.xml'
                    }
                }
            }

            stage('Build Image') {
                when { branch 'main' }
                steps {
                    sh "docker build -t ${REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} ."
                    sh "docker push ${REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}"
                }
            }

            stage('Deploy') {
                when { branch 'main' }
                steps {
                    deploy(
                        app: APP_NAME,
                        image: "${REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}",
                        environment: 'staging'
                    )
                }
            }
        }

        post {
            always { cleanWs() }
            failure {
                slackSend channel: config.slackChannel ?: '#builds',
                          color: 'danger',
                          message: "✕ ${APP_NAME} build FAILED"
            }
        }
    }
}
```

### Using the Standard Pipeline

```groovy
// Jenkinsfile (in application repo)
@Library('my-shared-library') _

standardPipeline(
    appName: 'user-service',
    buildImage: 'node:20',
    registry: 'registry.example.com',
    slackChannel: '#team-backend'
)
```

---

## Configuring Shared Libraries

```
  CONFIGURATION METHODS
  ═════════════════════

  1. GLOBAL (Manage Jenkins → System → Global Pipeline Libraries)
     ✓ Available to all pipelines
     ✓ Implicitly trusted (no sandbox)

  2. FOLDER-LEVEL
     ✓ Available to jobs in that folder only
     ✓ Scoped access control

  3. DYNAMIC (@Library in Jenkinsfile)
     @Library('my-lib@main') _
     @Library('my-lib@v1.2.0') _     ← Pin to version
     @Library('my-lib@feature-x') _  ← Use branch
```

---

## Using Classes (src/)

```groovy
// src/com/example/Docker.groovy
package com.example

class Docker implements Serializable {
    def steps

    Docker(steps) { this.steps = steps }

    def build(String image, String tag) {
        steps.sh "docker build -t ${image}:${tag} ."
    }

    def push(String image, String tag) {
        steps.sh "docker push ${image}:${tag}"
    }

    def buildAndPush(String image, String tag) {
        build(image, tag)
        push(image, tag)
    }
}
```

```groovy
// Using class in vars/ step
// vars/dockerBuild.groovy
import com.example.Docker

def call(String image, String tag) {
    def docker = new Docker(this)
    docker.buildAndPush(image, tag)
}
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Library not found | Not configured in Jenkins | Add to Global Pipeline Libraries settings |
| Class not found (src/) | Wrong package path | Ensure directory matches package declaration |
| "Scripts not permitted" | Sandbox restrictions | Use global library (trusted) or approve scripts |
| Changes not reflected | Library cached | Use `@Library('lib@branch')` with specific commit/tag |
| `Serializable` error | Non-serializable object in class | Implement `Serializable`; use `@NonCPS` for helper methods |
| Multiple libraries conflict | Name collision in vars/ | Use unique step names; prefix with library name |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Reusable pipeline code shared across projects |
| **Location** | Separate Git repository |
| **vars/** | Global pipeline steps (functions) |
| **src/** | Groovy classes for complex logic |
| **resources/** | Static files (scripts, templates) |
| **Import** | `@Library('name') _` in Jenkinsfile |

---

## Quick Revision Questions

1. **What is a Jenkins Shared Library?** *A Git repository containing reusable pipeline code (steps and classes) that can be imported into any Jenkinsfile.*
2. **What goes in the `vars/` directory?** *Global pipeline steps — Groovy files whose filename becomes the step name (e.g., `vars/buildDocker.groovy` → `buildDocker()`).*
3. **What goes in the `src/` directory?** *Groovy classes with full OOP support, organized by package (e.g., `src/com/example/Docker.groovy`).*
4. **How do you import a shared library?** *Add `@Library('library-name') _` at the top of the Jenkinsfile.*
5. **How do you pin a library to a specific version?** *Use `@Library('library-name@v1.2.0') _` to reference a specific tag, branch, or commit.*
6. **What is the benefit of a `standardPipeline` step?** *It provides a full reusable pipeline — application Jenkinsfiles become just a few lines of configuration.*

---

[← Previous: Jenkins Plugins](03-jenkins-plugins.md) | [Back to README](../README.md) | [Next: Multibranch Pipelines →](05-multibranch-pipelines.md)
