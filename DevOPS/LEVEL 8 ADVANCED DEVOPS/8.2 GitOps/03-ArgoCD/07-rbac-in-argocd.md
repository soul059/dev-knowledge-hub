# Chapter 3.7: RBAC in ArgoCD

[← Previous: Multi-Cluster Management](06-multi-cluster-management.md) | [Back to README](../README.md) | [Next → ArgoCD with Helm](08-argocd-with-helm.md)

---

## Overview

ArgoCD provides a **Role-Based Access Control (RBAC)** system that controls who can perform which actions on which resources. RBAC is configured through the `argocd-rbac-cm` ConfigMap and integrates with SSO providers. This chapter covers policies, roles, projects, and SSO integration.

---

## RBAC Architecture

```
┌──────────────────────────────────────────────────────────┐
│                ArgoCD RBAC Flow                          │
│                                                          │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │  User /  │───→│   SSO / OIDC │───→│  ArgoCD API  │   │
│  │  Group   │    │   Provider   │    │   Server     │   │
│  └──────────┘    └──────────────┘    └──────┬───────┘   │
│                                             │            │
│                                    ┌────────▼────────┐  │
│                                    │  RBAC Policy    │  │
│                                    │  (argocd-rbac-  │  │
│                                    │   cm ConfigMap) │  │
│                                    └────────┬────────┘  │
│                                             │            │
│                         ┌───────────────────┼──────┐    │
│                         ▼                   ▼      ▼    │
│                    ┌─────────┐       ┌──────┐ ┌──────┐  │
│                    │ ALLOW   │       │ DENY │ │ALLOW │  │
│                    │sync app │       │delete│ │read  │  │
│                    │in staging│       │prod  │ │logs  │  │
│                    └─────────┘       └──────┘ └──────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## Policy Syntax

```
p, <role/user/group>, <resource>, <action>, <object>, <allow/deny>
```

### Resources and Actions

| Resource | Actions |
|----------|---------|
| applications | get, create, update, delete, sync, override, action/* |
| applicationsets | get, create, update, delete |
| clusters | get, create, update, delete |
| repositories | get, create, update, delete |
| certificates | get, create, delete |
| accounts | get, update |
| gpgkeys | get, create, delete |
| logs | get |
| exec | create |
| projects | get, create, update, delete |

---

## Configuring RBAC

### argocd-rbac-cm ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  # Default policy for authenticated users
  policy.default: role:readonly

  # CSV-based policy definitions
  policy.csv: |
    # Built-in roles
    # role:readonly  - read-only access to all resources
    # role:admin     - full access to all resources

    # Custom Roles
    # -------------------------------------------------------

    # Dev team: can sync apps in dev project, view everything
    p, role:dev-team, applications, get, dev/*, allow
    p, role:dev-team, applications, sync, dev/*, allow
    p, role:dev-team, applications, action/*, dev/*, allow
    p, role:dev-team, logs, get, dev/*, allow

    # QA team: can sync staging, read-only on prod
    p, role:qa-team, applications, get, */*, allow
    p, role:qa-team, applications, sync, staging/*, allow

    # Ops team: full access
    p, role:ops-team, applications, *, */*, allow
    p, role:ops-team, clusters, *, *, allow
    p, role:ops-team, repositories, *, *, allow
    p, role:ops-team, projects, *, *, allow

    # Release manager: can sync prod
    p, role:release-mgr, applications, get, */*, allow
    p, role:release-mgr, applications, sync, prod/*, allow

    # Map SSO groups to roles
    g, my-org:dev-team, role:dev-team
    g, my-org:qa-team, role:qa-team
    g, my-org:ops-team, role:ops-team
    g, my-org:release-managers, role:release-mgr

    # Map individual user to admin
    g, admin@example.com, role:admin

  # Scopes to use from SSO token (default: '[groups]')
  scopes: '[groups, email]'
```

---

## AppProject RBAC

Projects provide an additional layer of access control:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications

  # Who can access this project
  roles:
  - name: deployer
    description: Can sync production apps
    policies:
    - p, proj:production:deployer, applications, sync, production/*, allow
    - p, proj:production:deployer, applications, get, production/*, allow
    groups:
    - my-org:release-managers

  - name: viewer
    description: Read-only access
    policies:
    - p, proj:production:viewer, applications, get, production/*, allow
    groups:
    - my-org:dev-team
    - my-org:qa-team

  # Allowed source repos
  sourceRepos:
  - 'https://github.com/myorg/production-config.git'

  # Allowed destination clusters/namespaces
  destinations:
  - server: https://prod-cluster-api:6443
    namespace: 'app-*'

  # Deny-listed cluster resources
  clusterResourceBlacklist:
  - group: ''
    kind: Namespace
  - group: rbac.authorization.k8s.io
    kind: ClusterRole

  # Allowed namespace resources
  namespaceResourceWhitelist:
  - group: apps
    kind: Deployment
  - group: ''
    kind: Service
  - group: ''
    kind: ConfigMap
  - group: networking.k8s.io
    kind: Ingress
```

---

## SSO Integration

### OIDC with Keycloak / Dex

```yaml
# argocd-cm ConfigMap — SSO setup
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com

  # OIDC configuration (e.g., Keycloak)
  oidc.config: |
    name: Keycloak
    issuer: https://keycloak.example.com/realms/myorg
    clientID: argocd
    clientSecret: $oidc.keycloak.clientSecret
    requestedScopes:
      - openid
      - profile
      - email
      - groups
    requestedIDTokenClaims:
      groups:
        essential: true
```

### SSO Group → ArgoCD Role Flow

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  1. User logs in via SSO                                 │
│     └──→ SSO returns JWT with groups claim               │
│                                                          │
│  2. ArgoCD reads groups from JWT                         │
│     JWT: { "groups": ["my-org:dev-team"] }               │
│                                                          │
│  3. RBAC maps group to role                              │
│     g, my-org:dev-team, role:dev-team                    │
│                                                          │
│  4. Policy grants permissions to role                    │
│     p, role:dev-team, applications, sync, dev/*, allow   │
│                                                          │
│  5. User can sync apps in dev project only               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Local Accounts

```yaml
# argocd-cm ConfigMap — local accounts
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Create a local account for CI pipeline
  accounts.ci-bot: apiKey, login
  accounts.ci-bot.enabled: "true"

  # Viewer account (login only, no API key)
  accounts.viewer: login
```

```bash
# Set password for local account
argocd account update-password \
  --account ci-bot \
  --new-password '<secure-password>'

# Generate API key for CI
argocd account generate-token --account ci-bot

# List accounts
argocd account list

# Get account info
argocd account get --account ci-bot
```

---

## Testing RBAC Policies

```bash
# Test if a user/role can perform an action
argocd admin settings rbac can role:dev-team sync applications 'dev/*' \
  --policy-file policy.csv

# Expected output: Yes / No

# Validate the full policy
argocd admin settings rbac validate \
  --policy-file policy.csv
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| RBAC ConfigMap | `argocd-rbac-cm` holds policy.csv and policy.default |
| Policy format | `p, <subject>, <resource>, <action>, <object>, allow/deny` |
| Group mapping | `g, <sso-group>, <role>` maps SSO groups to ArgoCD roles |
| AppProject RBAC | Projects define roles, allowed sources, destinations |
| SSO integration | OIDC config in `argocd-cm`, group claims for mapping |
| Local accounts | Defined in `argocd-cm`, useful for CI bots |
| Testing | `argocd admin settings rbac can ...` to validate |

---

## Quick Revision Questions

1. **Where is ArgoCD RBAC configured?**
   > In the `argocd-rbac-cm` ConfigMap in the `argocd` namespace, under the `policy.csv` key.

2. **What is the syntax for an RBAC policy rule?**
   > `p, <subject>, <resource>, <action>, <object>, allow/deny`

3. **How do you map an SSO group to an ArgoCD role?**
   > Use the `g` (group) statement: `g, my-org:dev-team, role:dev-team`

4. **What additional access control does an AppProject provide?**
   > Projects restrict allowed source repos, destination clusters/namespaces, permitted resource kinds, and define project-scoped roles.

5. **How do you create a local account for a CI bot?**
   > Add `accounts.<name>: apiKey, login` to the `argocd-cm` ConfigMap, then generate a token with `argocd account generate-token`.

6. **How do you test RBAC policies without applying them?**
   > Use `argocd admin settings rbac can <role> <action> <resource> <object> --policy-file policy.csv`.

---

[← Previous: Multi-Cluster Management](06-multi-cluster-management.md) | [Back to README](../README.md) | [Next → ArgoCD with Helm](08-argocd-with-helm.md)
