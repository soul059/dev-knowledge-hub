# Chapter 9.5: Ansible Tower / AWX

[← Previous: Custom Modules](04-custom-modules.md) | [Next: Performance Tuning →](06-performance-tuning.md)

---

## Overview

**Ansible Tower** (commercial) and **AWX** (open-source upstream) provide a web-based UI, REST API, role-based access control, and centralised management for Ansible automation. They transform Ansible from a command-line tool into an enterprise automation platform.

---

## Tower vs AWX

```
  ┌────────────────────────────────────────────────────┐
  │              Red Hat Ansible Platform               │
  │  ┌──────────────────────────────────────────────┐  │
  │  │          Ansible Automation Controller        │  │
  │  │          (formerly Ansible Tower)              │  │
  │  │                                               │  │
  │  │  ┌─────────────────────────────────────────┐  │  │
  │  │  │              AWX                         │  │  │
  │  │  │        (Open-source upstream)            │  │  │
  │  │  └─────────────────────────────────────────┘  │  │
  │  │  + Enterprise Support                         │  │
  │  │  + Certified Content                          │  │
  │  │  + Long-term Releases                         │  │
  │  └──────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────┘
```

| Feature | AWX | Tower / AAP |
|---------|-----|-------------|
| Cost | Free (open source) | Commercial subscription |
| Support | Community | Red Hat support |
| Releases | Frequent, rolling | LTS (stable) |
| Certification | No | Certified collections |
| Use case | Dev/test, learning | Production enterprise |

---

## Core Components

```
  ┌─────────────────────────────────────────────────────┐
  │                    WEB UI / API                      │
  ├─────────────────────────────────────────────────────┤
  │                                                     │
  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
  │  │Projects  │  │Inventories│  │  Credentials     │  │
  │  │(Git repos)│  │(hosts)   │  │  (vault, SSH,    │  │
  │  │          │  │          │  │   cloud keys)     │  │
  │  └────┬─────┘  └────┬─────┘  └────────┬─────────┘  │
  │       │              │                 │            │
  │       └──────────────┼─────────────────┘            │
  │                      │                              │
  │              ┌───────▼───────┐                      │
  │              │  Job Template │                      │
  │              │  (playbook +  │                      │
  │              │   inventory + │                      │
  │              │   credential) │                      │
  │              └───────┬───────┘                      │
  │                      │                              │
  │              ┌───────▼───────┐                      │
  │              │   Workflow    │                      │
  │              │  (chain jobs) │                      │
  │              └───────┬───────┘                      │
  │                      │                              │
  │              ┌───────▼───────┐                      │
  │              │   Schedules   │                      │
  │              │  (cron-like)  │                      │
  │              └───────────────┘                      │
  │                                                     │
  ├─────────────────────────────────────────────────────┤
  │          RBAC  │  Notifications  │  Audit Logs      │
  └─────────────────────────────────────────────────────┘
```

---

## Key Concepts

### 1. Organizations

```
  Organization: "ACME Corp"
  ├── Teams
  │   ├── "Platform Team"
  │   └── "App Team"
  ├── Users
  │   ├── admin@acme.com
  │   └── dev@acme.com
  ├── Projects
  ├── Inventories
  └── Credentials
```

### 2. Projects (SCM Integration)

```yaml
# Tower syncs playbooks from Git
Project:
  Name: "Web Platform"
  SCM Type: Git
  SCM URL: https://github.com/acme/ansible-web.git
  SCM Branch: main
  SCM Update Options:
    - Update on Launch: true    # Pull latest before each job
    - Clean: true               # Reset local changes
```

### 3. Inventories

```
  Smart Inventory               Static Inventory
  ═══════════════               ════════════════
  Dynamic filter:               Manually defined:
  "os_type:linux AND            [webservers]
   environment:prod"            web01
                                web02
         ▼                           ▼
  Auto-populates hosts          Fixed host list
  from all inventories
```

### 4. Credentials

| Credential Type | Use Case |
|----------------|----------|
| Machine | SSH keys / passwords for managed nodes |
| Source Control | Git auth for projects |
| Vault | Ansible Vault passwords |
| Network | Network device auth |
| Cloud (AWS/Azure/GCP) | API keys for cloud modules |
| Container Registry | Pull container images |
| Custom | User-defined credential types |

### 5. Job Templates

```
  Job Template = Project + Inventory + Credential + Playbook
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────┐
  │  Job Template: "Deploy Web App"             │
  │                                             │
  │  Project:     Web Platform (Git)            │
  │  Playbook:    deploy.yml                    │
  │  Inventory:   Production Servers            │
  │  Credential:  SSH Key (prod)                │
  │  Extra Vars:  app_version: "2.1.0"          │
  │  Limit:       webservers                    │
  │  Verbosity:   0 (Normal)                    │
  │  Forks:       10                            │
  │  Job Type:    Run  (or Check)               │
  └─────────────────────────────────────────────┘
```

### 6. Workflows

```
  ┌──────────┐     Success    ┌──────────┐
  │  Lint &  ├───────────────▶│  Deploy  │
  │  Test    │                │  Staging │
  │          │     Failure    │          │     Success
  └──────────┘───────┐       └──────────┘─────────┐
                     │                             │
               ┌─────▼──────┐              ┌──────▼─────┐
               │  Notify    │              │  Deploy    │
               │  Dev Team  │              │  Prod      │
               └────────────┘              └──────┬─────┘
                                                  │
                                           ┌──────▼─────┐
                                           │  Smoke     │
                                           │  Tests     │
                                           └────────────┘
```

Workflows connect job templates with conditional paths:
- **On Success** → next job
- **On Failure** → notification or rollback
- **Always** → cleanup tasks

---

## RBAC (Role-Based Access Control)

```
  PERMISSION MODEL
  ══════════════════════════════════════════════════════════

  Organization
    └── Team
         └── User
              └── Role (on a resource)

  Roles:
  ┌──────────┬────────────────────────────────────────┐
  │ Admin    │ Full control over resource             │
  │ Execute  │ Run jobs (job templates)               │
  │ Use      │ Use resource in other contexts         │
  │ Read     │ View-only access                       │
  │ Update   │ Modify resource settings               │
  └──────────┴────────────────────────────────────────┘
```

```yaml
# Example RBAC setup
- Organization: ACME
  Teams:
    - name: Developers
      permissions:
        - resource: "Deploy Staging"
          role: Execute            # Can run staging deploys
        - resource: "Staging Inventory"
          role: Use
    - name: Operations
      permissions:
        - resource: "Deploy Production"
          role: Execute            # Can run prod deploys
        - resource: "All Inventories"
          role: Admin
```

---

## REST API

```bash
# List job templates
curl -u admin:password \
  https://tower.example.com/api/v2/job_templates/

# Launch a job
curl -X POST -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"extra_vars": {"version": "2.1.0"}}' \
  https://tower.example.com/api/v2/job_templates/15/launch/

# Check job status
curl -u admin:password \
  https://tower.example.com/api/v2/jobs/42/
```

### Using the tower-cli / awx CLI

```bash
# Install
pip install awxkit

# Login
awx login --conf.host https://tower.example.com \
          --conf.username admin \
          --conf.password secret

# Launch a job
awx job_templates launch "Deploy Web App" \
    --extra_vars '{"version": "2.1.0"}'

# Monitor job
awx jobs get 42
```

---

## AWX Installation (Docker Compose)

```bash
# Clone AWX
git clone https://github.com/ansible/awx.git
cd awx

# Use the AWX operator for Kubernetes
# Or Docker Compose for development:
cd tools/docker-compose
make docker-compose-build
docker-compose up -d
```

```
  AWX ARCHITECTURE
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────┐
  │            Kubernetes / Docker           │
  │                                         │
  │  ┌──────────┐  ┌──────────┐            │
  │  │  AWX Web │  │  AWX Task│            │
  │  │  (nginx +│  │  (celery │            │
  │  │   django)│  │  workers)│            │
  │  └────┬─────┘  └────┬─────┘            │
  │       │              │                  │
  │  ┌────▼──────────────▼─────┐            │
  │  │      PostgreSQL         │            │
  │  │      Redis              │            │
  │  └────────────────────────┘             │
  └─────────────────────────────────────────┘
```

---

## Notifications

| Type | Destination |
|------|-------------|
| Email | SMTP server |
| Slack | Channel webhook |
| PagerDuty | Incident alerts |
| Webhook | Custom HTTP endpoints |
| IRC | Chat channels |
| Grafana | Annotation events |

```yaml
# Notification attached to Job Template:
Notifications:
  On Start:    Slack #deployments
  On Success:  Email ops@example.com
  On Failure:  PagerDuty + Slack #incidents
```

---

## Real-World Scenario: Self-Service Portal

```
  SELF-SERVICE AUTOMATION
  ══════════════════════════════════════════════════════════

  Developer                    Tower / AWX
  ┌──────────┐     Survey    ┌────────────────────────┐
  │  Click   │──────────────▶│  Job Template:         │
  │  "Deploy"│    Form       │  "Deploy Application"  │
  │  in UI   │               │                        │
  └──────────┘               │  Survey Questions:     │
                             │  - App Version? [___]  │
                             │  - Environment? [▼]    │
                             │  - Notify? [✓]         │
                             │                        │
                             │  ──▶ Runs playbook     │
                             │  ──▶ RBAC enforced     │
                             │  ──▶ Audit logged      │
                             └────────────────────────┘
```

**Surveys** let you prompt users for variables at launch time through a web form, making automation accessible to non-Ansible users.

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Project sync fails | Check SCM URL, credentials, and network access to Git |
| Job stuck in pending | Check available capacity (instances/forks) |
| Credential errors | Verify credential type and stored values |
| RBAC permission denied | Check team/user role assignments on the resource |
| API returns 403 | Token expired or insufficient permissions |
| AWX pods crashing | Check PostgreSQL connectivity and resource limits |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| AWX | Open-source upstream of Tower |
| Tower / AAP | Enterprise product with support |
| Project | Git repository containing playbooks |
| Inventory | Hosts and groups (static or dynamic) |
| Credential | Securely stored authentication data |
| Job Template | Combines project + inventory + credential |
| Workflow | Chains job templates with conditional logic |
| Survey | User-input form at job launch |
| RBAC | Permission model: Org → Team → User → Role |
| API | REST API for programmatic control |
| Notifications | Alerts via Slack, email, PagerDuty, webhooks |

---

## Quick Revision Questions

1. **What is the difference between AWX and Ansible Tower?**
   > AWX is the free, open-source upstream project. Tower (now Ansible Automation Controller) is the commercial product with enterprise support, certified content, and LTS releases.

2. **What three things does a Job Template combine?**
   > A Project (playbook source), an Inventory (target hosts), and a Credential (authentication).

3. **What is a Workflow in Tower/AWX?**
   > A Workflow chains multiple job templates together with conditional paths (on success, on failure, always), allowing complex multi-stage automation.

4. **How does RBAC work in Tower?**
   > Permissions are granted through roles (Admin, Execute, Use, Read) assigned to users or teams on specific resources within an organization.

5. **What is a Survey?**
   > A Survey is a customisable web form that prompts users for variable values at job launch time, enabling self-service automation.

---

[← Previous: Custom Modules](04-custom-modules.md) | [Next: Performance Tuning →](06-performance-tuning.md)
