# Chapter 6.6: Jenkins Security

[← Previous: Multibranch Pipelines](05-multibranch-pipelines.md) | [Back to README](../README.md) | [Next: Jenkins Best Practices →](07-jenkins-best-practices.md)

---

## Overview

Jenkins is a high-value target because it holds credentials, accesses production systems, and executes arbitrary code. Securing Jenkins is critical for any CI/CD environment.

---

## Security Architecture

```
  JENKINS SECURITY LAYERS
  ════════════════════════

  ┌─────────────────────────────────────────┐
  │           NETWORK LAYER                 │
  │  • Reverse proxy (Nginx/HAProxy)        │
  │  • TLS termination                      │
  │  • Firewall rules                       │
  │  • VPN / private network                │
  └──────────────────┬──────────────────────┘
                     ▼
  ┌─────────────────────────────────────────┐
  │        AUTHENTICATION LAYER             │
  │  • LDAP / Active Directory              │
  │  • SAML / OAuth / OIDC                  │
  │  • Internal database                    │
  └──────────────────┬──────────────────────┘
                     ▼
  ┌─────────────────────────────────────────┐
  │        AUTHORIZATION LAYER              │
  │  • Role-Based Access Control (RBAC)     │
  │  • Matrix-based security                │
  │  • Project-based authorization          │
  └──────────────────┬──────────────────────┘
                     ▼
  ┌─────────────────────────────────────────┐
  │        EXECUTION LAYER                  │
  │  • Script Security Sandbox              │
  │  • Credential masking                   │
  │  • Agent isolation                      │
  └─────────────────────────────────────────┘
```

---

## Authentication

### LDAP Configuration

```yaml
# JCasC: Jenkins Configuration as Code
jenkins:
  securityRealm:
    ldap:
      configurations:
        - server: "ldap://ldap.example.com"
          rootDN: "dc=example,dc=com"
          userSearchBase: "ou=People"
          userSearch: "uid={0}"
          groupSearchBase: "ou=Groups"
          managerDN: "cn=admin,dc=example,dc=com"
          managerPasswordSecret: "${LDAP_ADMIN_PASSWORD}"
```

### OIDC / OAuth (e.g., GitHub)

```yaml
# JCasC: GitHub OAuth
jenkins:
  securityRealm:
    github:
      githubWebUri: "https://github.com"
      githubApiUri: "https://api.github.com"
      clientID: "${GITHUB_CLIENT_ID}"
      clientSecret: "${GITHUB_CLIENT_SECRET}"
      oauthScopes: "read:org,user:email"
```

---

## Authorization (RBAC)

```
  ROLE-BASED ACCESS CONTROL
  ═════════════════════════

  Role: admin
  ├── Jenkins.ADMINISTER
  ├── All permissions
  └── Members: ops-team

  Role: developer
  ├── Job.BUILD
  ├── Job.READ
  ├── Job.WORKSPACE
  ├── Run.REPLAY
  └── Members: dev-team

  Role: viewer
  ├── Job.READ (read-only)
  ├── Run.READ
  └── Members: stakeholders

  Role: deployer (project-specific)
  ├── Job.BUILD on production jobs only
  └── Members: release-managers
```

### JCasC Authorization

```yaml
# Role Strategy Plugin
jenkins:
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            entries:
              - group: "ops-team"
          - name: "developer"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Read"
              - "Job/Workspace"
            entries:
              - group: "dev-team"
          - name: "viewer"
            permissions:
              - "Overall/Read"
              - "Job/Read"
            entries:
              - group: "stakeholders"
        items:
          - name: "deployer"
            pattern: "production/.*"
            permissions:
              - "Job/Build"
            entries:
              - group: "release-managers"
```

---

## Credentials Management

```
  CREDENTIAL TYPES
  ═════════════════

  ┌──────────────────────┬──────────────────────────────┐
  │ Type                 │ Use Case                     │
  ├──────────────────────┼──────────────────────────────┤
  │ Username/Password    │ Git repos, APIs              │
  │ SSH Key              │ Git clone, server access     │
  │ Secret Text          │ API tokens, passwords        │
  │ Secret File          │ Kubeconfig, certs, keystores │
  │ Certificate (PKCS12) │ TLS client certificates      │
  └──────────────────────┴──────────────────────────────┘

  SCOPE:
  • System   → Available to Jenkins internals only
  • Global   → Available to all pipelines
  • Folder   → Available to jobs in specific folder
```

### Using Credentials in Pipelines

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                // Username/Password
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-registry',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS registry.example.com'
                }

                // Secret Text
                withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
                    sh 'curl -H "Authorization: Bearer $API_KEY" https://api.example.com/deploy'
                }

                // SSH Key
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh 'ssh -i $SSH_KEY deploy@server.example.com "systemctl restart app"'
                }

                // Secret File (e.g., kubeconfig)
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
```

### External Secrets (HashiCorp Vault)

```groovy
// Using HashiCorp Vault Plugin
pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                withVault(
                    configuration: [
                        vaultUrl: 'https://vault.example.com',
                        vaultCredentialId: 'vault-approle'
                    ],
                    vaultSecrets: [
                        [path: 'secret/data/myapp',
                         secretValues: [
                             [envVar: 'DB_PASSWORD', vaultKey: 'db_password'],
                             [envVar: 'API_SECRET', vaultKey: 'api_secret']
                         ]]
                    ]
                ) {
                    sh './deploy.sh'
                }
            }
        }
    }
}
```

---

## Script Security Sandbox

```
  GROOVY SANDBOX
  ══════════════

  Jenkinsfile (Declarative)
  └── Runs in sandbox by default
      ├── Safe method calls → ALLOWED
      ├── Dangerous calls → BLOCKED
      └── Admin approval needed for unsafe methods

  Shared Libraries (Global / Trusted)
  └── Runs OUTSIDE sandbox
      ├── Full Groovy access
      └── Only admins should push to library repos

  COMMON BLOCKED OPERATIONS:
  • new File()         → Use readFile / writeFile steps
  • System.exit()      → Not allowed
  • Runtime.exec()     → Use sh step instead
  • Class.forName()    → Not allowed
  • Thread operations  → Not allowed
```

### Managing Script Approvals

```
  Manage Jenkins → In-process Script Approval

  ┌─────────────────────────────────────────────────┐
  │ Pending Approvals:                              │
  │                                                 │
  │ ☐ method java.util.Map entrySet                 │
  │ ☐ staticMethod org.codehaus.groovy.runtime...   │
  │                                                 │
  │ [ Approve ]  [ Deny ]                           │
  └─────────────────────────────────────────────────┘

  TIP: Review each approval carefully.
       Avoid blanket approvals.
       Move complex logic to trusted shared libraries.
```

---

## CSRF Protection

```groovy
// JCasC: Enable CSRF Protection
jenkins:
  crumbIssuer:
    standard:
      excludeClientIPFromCrumb: false
```

### API Calls With CSRF Token

```bash
# 1. Get crumb
CRUMB=$(curl -s -u admin:token \
    'https://jenkins.example.com/crumbIssuer/api/json' \
    | jq -r '.crumb')

# 2. Use crumb in API call
curl -X POST -u admin:token \
    -H "Jenkins-Crumb: ${CRUMB}" \
    'https://jenkins.example.com/job/my-job/build'
```

---

## Agent Security

```
  AGENT SECURITY BEST PRACTICES
  ══════════════════════════════

  ┌──────────────────────────────────┐
  │ Jenkins Controller               │
  │ • NO builds on controller        │
  │ • Set executors = 0              │
  │ • Protected admin access         │
  └───────────┬──────────────────────┘
              │ Encrypted connection
              ▼
  ┌──────────────────────────────────┐
  │ Build Agents (Ephemeral)         │
  │ • Docker/K8s agents preferred    │
  │ • Fresh environment per build    │
  │ • No persistent credentials      │
  │ • Minimal tools installed        │
  │ • Network isolation              │
  └──────────────────────────────────┘
```

```yaml
# JCasC: Disable builds on controller
jenkins:
  numExecutors: 0
  labelString: "controller"

  nodes:
    - permanent:
        name: "build-agent-1"
        remoteFS: "/home/jenkins"
        launcher:
          ssh:
            host: "agent1.example.com"
            credentialsId: "agent-ssh-key"
        nodeProperties:
          - envVars:
              env:
                - key: "AGENT_ROLE"
                  value: "build"
```

---

## Security Checklist

```
  ☐ Enable authentication (LDAP/OIDC/SAML)
  ☐ Configure RBAC with least privilege
  ☐ Set controller executors to 0
  ☐ Enable CSRF protection
  ☐ Use HTTPS with valid certificate
  ☐ Store credentials in Jenkins credential store
  ☐ Enable audit logging
  ☐ Regularly update Jenkins and plugins
  ☐ Review script approvals carefully
  ☐ Use ephemeral agents (Docker/K8s)
  ☐ Limit plugin installations to essentials
  ☐ Restrict agent-to-controller access
  ☐ Configure Content Security Policy headers
  ☐ Back up Jenkins master key securely
```

---

## Real-World Scenario

> **Scenario**: A company needs Jenkins to comply with SOC 2 audit requirements.
>
> **Solution**: Enable GitHub OAuth for authentication. Configure RBAC with separate roles (admin, developer, viewer, deployer). Store all secrets in HashiCorp Vault via the Vault plugin. Enable audit trail plugin for all actions. Set controller executors to 0 and use ephemeral Kubernetes agents. Enable CSRF protection and HTTPS. Set up automated plugin update checks.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| "Access Denied" for user | Insufficient permissions | Check role assignments in RBAC configuration |
| Credential not found | Scope mismatch | Ensure credential scope (Global/Folder) matches job location |
| Script blocked by sandbox | Unsafe Groovy method | Move to trusted shared library or request admin approval |
| CSRF token rejected | Crumb not included | Fetch crumb from `/crumbIssuer/api/json` and include in header |
| Agent connection refused | Firewall or wrong port | Open JNLP port (50000) or use SSH-based agents |
| Secrets visible in logs | Not using `withCredentials` | Wrap secret usage in `withCredentials` block for masking |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Authentication** | LDAP, SAML, OIDC, GitHub OAuth |
| **Authorization** | RBAC, Matrix-based, Project-based |
| **Credentials** | Jenkins store, HashiCorp Vault, folder-scoped |
| **Sandbox** | Groovy sandbox for untrusted code |
| **CSRF** | Crumb issuer enabled by default |
| **Agents** | Ephemeral Docker/K8s, no builds on controller |

---

## Quick Revision Questions

1. **What authentication methods does Jenkins support?** *Internal database, LDAP, Active Directory, SAML, OAuth/OIDC (e.g., GitHub OAuth).*
2. **What is RBAC in Jenkins?** *Role-Based Access Control — assigning permissions (build, read, admin) to roles, then mapping users/groups to roles.*
3. **How are credentials protected in pipelines?** *Using `withCredentials` blocks that bind secrets to environment variables and mask them in console output.*
4. **What is the Groovy sandbox?** *A security mechanism that restricts which Groovy methods pipeline scripts can call — dangerous methods require admin approval.*
5. **Why should the controller have 0 executors?** *To prevent builds from running on the controller, which could compromise credentials, configuration, and the Jenkins instance itself.*
6. **How does CSRF protection work in Jenkins?** *Jenkins issues a crumb (token) that must be included in API requests to prevent cross-site request forgery attacks.*

---

[← Previous: Multibranch Pipelines](05-multibranch-pipelines.md) | [Back to README](../README.md) | [Next: Jenkins Best Practices →](07-jenkins-best-practices.md)
