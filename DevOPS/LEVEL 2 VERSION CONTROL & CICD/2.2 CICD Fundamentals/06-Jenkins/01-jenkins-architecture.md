# Chapter 6.1: Jenkins Architecture

[← Previous: Zero-Downtime Deployments](../05-Deployment-Strategies/07-zero-downtime-deployments.md) | [Back to README](../README.md) | [Next: Jenkinsfile & Pipeline Syntax →](02-jenkinsfile-pipeline-syntax.md)

---

## Overview

**Jenkins** is the most widely adopted open-source CI/CD server. It uses a controller-agent architecture where a central controller orchestrates builds that execute on distributed agents.

---

## Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │                 JENKINS ARCHITECTURE                     │
  │                                                          │
  │  ┌────────────────────────────────┐                      │
  │  │      JENKINS CONTROLLER       │                      │
  │  │                                │                      │
  │  │  ┌──────────┐  ┌───────────┐  │                      │
  │  │  │ Web UI   │  │ REST API  │  │                      │
  │  │  └──────────┘  └───────────┘  │                      │
  │  │  ┌──────────┐  ┌───────────┐  │                      │
  │  │  │ Job      │  │ Plugin    │  │                      │
  │  │  │ Scheduler│  │ Manager   │  │                      │
  │  │  └──────────┘  └───────────┘  │                      │
  │  │  ┌──────────┐  ┌───────────┐  │                      │
  │  │  │ Build    │  │ Credential│  │                      │
  │  │  │ Queue    │  │ Store     │  │                      │
  │  │  └──────────┘  └───────────┘  │                      │
  │  └───────────────┬────────────────┘                      │
  │                  │                                       │
  │         ┌────────┼────────┐                              │
  │         │        │        │                              │
  │    ┌────▼───┐┌───▼────┐┌──▼─────┐                       │
  │    │ Agent  ││ Agent  ││ Agent  │                       │
  │    │ Linux  ││ Windows││ Docker │                       │
  │    │        ││        ││        │                       │
  │    │ Java   ││ .NET   ││ Node   │                       │
  │    │ Maven  ││ MSBuild││ npm    │                       │
  │    └────────┘└────────┘└────────┘                       │
  │                                                          │
  │    Agents connect via:                                   │
  │    • SSH (recommended)                                   │
  │    • JNLP/Inbound (agent initiates)                     │
  │    • Docker (ephemeral containers)                      │
  │    • Kubernetes (dynamic pods)                          │
  └──────────────────────────────────────────────────────────┘
```

---

## Controller vs Agent

| Component | Controller | Agent |
|---|---|---|
| **Role** | Orchestration, scheduling, UI | Execute builds |
| **Runs** | Always running | On-demand or permanent |
| **Stores** | Configurations, logs, plugins | Build workspace (temporary) |
| **Resources** | Moderate CPU/RAM | High CPU/RAM for builds |
| **Count** | 1 (HA: 2) | Many (scale horizontally) |
| **Execute builds?** | No (best practice) | Yes |

---

## Key Components

```
  ┌──────────────────────────────────────────────┐
  │           JENKINS COMPONENTS                 │
  │                                              │
  │  JOB / PROJECT                               │
  │  ├── Freestyle Project  (GUI configured)     │
  │  ├── Pipeline           (Jenkinsfile)        │
  │  ├── Multibranch Pipeline (per branch)       │
  │  └── Organization Folder (GitHub/GitLab org) │
  │                                              │
  │  BUILD                                       │
  │  ├── A single execution of a job             │
  │  ├── Has a build number (#1, #2, ...)        │
  │  └── Produces artifacts, logs, test results  │
  │                                              │
  │  WORKSPACE                                   │
  │  ├── Directory where build runs              │
  │  ├── Source code checked out here            │
  │  └── Cleaned between builds (configurable)   │
  │                                              │
  │  PLUGIN                                      │
  │  ├── Extends Jenkins functionality           │
  │  ├── 1800+ plugins available                 │
  │  └── Installed via Plugin Manager UI         │
  │                                              │
  │  CREDENTIAL                                  │
  │  ├── Securely stored secrets                 │
  │  ├── Types: username/password, SSH key,      │
  │  │   secret text, certificate                │
  │  └── Scoped: Global, Folder, or Job          │
  └──────────────────────────────────────────────┘
```

---

## Installation

```bash
# Docker (recommended for quick setup)
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Ubuntu/Debian
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

# Helm (Kubernetes)
helm repo add jenkins https://charts.jenkins.io
helm install jenkins jenkins/jenkins \
  --set controller.serviceType=LoadBalancer
```

---

## Essential Plugins

| Plugin | Purpose |
|---|---|
| **Pipeline** | Enables Jenkinsfile-based pipelines |
| **Git** | Git SCM integration |
| **Blue Ocean** | Modern UI for pipelines |
| **Docker Pipeline** | Build inside Docker containers |
| **Kubernetes** | Dynamic agents on Kubernetes |
| **Credentials Binding** | Inject secrets into builds |
| **Pipeline Utility Steps** | File ops, JSON/YAML parsing |
| **Slack Notification** | Send build notifications |
| **JUnit** | Test result reporting |
| **Cobertura / JaCoCo** | Code coverage reporting |

---

## Agent Configuration

```groovy
// Jenkinsfile — Specifying agents
pipeline {
    // Run on any available agent
    agent any

    // Run on agent with specific label
    agent { label 'linux && docker' }

    // Run in Docker container
    agent {
        docker {
            image 'node:20'
            args '-v /tmp:/tmp'
        }
    }

    // Run on Kubernetes pod
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: node
                image: node:20
                command: ['sleep', 'infinity']
              - name: docker
                image: docker:24-dind
                securityContext:
                  privileged: true
            """
        }
    }
}
```

---

## Jenkins Directory Structure

```
  $JENKINS_HOME/
  ├── config.xml              # Global configuration
  ├── credentials.xml         # Encrypted credentials
  ├── secrets/                # Encryption keys
  ├── plugins/                # Installed plugins (.jpi)
  ├── users/                  # User configurations
  ├── jobs/                   # Job configurations
  │   ├── my-project/
  │   │   ├── config.xml      # Job config
  │   │   ├── builds/         # Build history
  │   │   │   ├── 1/
  │   │   │   │   ├── log     # Build log
  │   │   │   │   └── build.xml
  │   │   │   └── 2/
  │   │   └── workspace/      # Working directory
  │   └── my-pipeline/
  │       └── config.xml
  ├── nodes/                  # Agent configurations
  └── logs/                   # Jenkins logs
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Jenkins slow / high memory | Too many builds/plugins | Limit build history; prune old builds; audit plugins |
| Agent disconnects frequently | Network issues / SSH timeout | Use JNLP; configure keep-alive; check firewall |
| Builds stuck in queue | No available agents | Add agents; check agent labels match job requirements |
| Plugin conflicts | Incompatible plugin versions | Use Plugin Manager; update incrementally; test first |
| Disk full | Build logs and workspaces accumulate | Configure build retention; clean workspaces |
| Controller runs builds | No agents configured | Never run builds on controller; add dedicated agents |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Architecture** | Controller + Agent (distributed) |
| **Controller** | Scheduling, UI, configuration, plugin management |
| **Agents** | Execute builds; connect via SSH, JNLP, Docker, K8s |
| **Configuration** | Jenkins Home directory; XML-based |
| **Extensibility** | 1800+ plugins |
| **Installation** | Docker, native packages, Helm chart |

---

## Quick Revision Questions

1. **What is the Jenkins controller?** *The central server that manages scheduling, UI, configurations, and plugin management — it should not execute builds directly.*
2. **What is a Jenkins agent?** *A worker machine that connects to the controller and executes build jobs.*
3. **What are the agent connection methods?** *SSH (recommended), JNLP/Inbound (agent-initiated), Docker containers, Kubernetes pods.*
4. **Where is Jenkins configuration stored?** *In the JENKINS_HOME directory, primarily as XML files.*
5. **What is the difference between Freestyle and Pipeline projects?** *Freestyle uses GUI configuration; Pipeline uses Jenkinsfile (code) for defining the build process.*
6. **Why should builds NOT run on the controller?** *For security (builds execute arbitrary code) and performance (builds consume resources needed for orchestration).*

---

[← Previous: Zero-Downtime Deployments](../05-Deployment-Strategies/07-zero-downtime-deployments.md) | [Back to README](../README.md) | [Next: Jenkinsfile & Pipeline Syntax →](02-jenkinsfile-pipeline-syntax.md)
