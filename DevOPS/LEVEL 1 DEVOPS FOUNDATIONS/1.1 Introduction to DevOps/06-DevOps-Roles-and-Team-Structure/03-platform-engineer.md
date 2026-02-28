# Chapter 6.3 — Platform Engineer

[← Previous: Site Reliability Engineer](02-sre.md) | [Next: Team Topologies →](04-team-topologies.md)

---

## Overview

**Platform Engineering** is the discipline of building and maintaining **Internal Developer Platforms (IDPs)** — self-service toolchains and infrastructure that enable development teams to ship software faster without needing deep infrastructure expertise. While DevOps asks developers to own their infrastructure, Platform Engineering recognizes that **not every developer wants to become a Kubernetes expert** — so platform engineers build golden paths that abstract complexity.

---

## Why Platform Engineering?

```
┌──────────────────────────────────────────────────────────┐
│  THE PROBLEM PLATFORM ENGINEERING SOLVES                 │
│                                                          │
│  DEVOPS PROMISE:       DEVOPS REALITY (at scale):        │
│  "You build it,        "I just want to deploy my app,    │
│   you run it"           not learn Terraform, Kubernetes, │
│                         Helm, ArgoCD, Prometheus,        │
│                         networking, IAM policies..."     │
│                                                          │
│  ┌─────────────────────────────────────────────┐        │
│  │     COGNITIVE LOAD on Development Teams      │        │
│  │                                               │        │
│  │  Without Platform:  WITH Platform:            │        │
│  │  ██████████████████  ████████░░░░░░░░░░░░░   │        │
│  │  App + Infra + CI/CD  App code (platform      │        │
│  │  + Monitoring + K8s    handles the rest)      │        │
│  │  + Security + ...                             │        │
│  └─────────────────────────────────────────────┘        │
│                                                          │
│  Platform Engineering = "Make the right thing easy"      │
└──────────────────────────────────────────────────────────┘
```

---

## Internal Developer Platform (IDP) Architecture

```
┌──────────────────────────────────────────────────────────┐
│  INTERNAL DEVELOPER PLATFORM (IDP)                       │
│                                                          │
│  ┌──────────────────────────────────────────────┐       │
│  │  DEVELOPER PORTAL (e.g., Backstage)           │       │
│  │  ├── Service Catalog                          │       │
│  │  ├── Create New Service (wizard)              │       │
│  │  ├── Documentation Hub                        │       │
│  │  ├── API Catalog                              │       │
│  │  └── Cost Dashboard                           │       │
│  └──────────────────┬───────────────────────────┘       │
│                      │                                   │
│  ┌──────────────────▼───────────────────────────┐       │
│  │  GOLDEN PATHS (Paved Roads)                   │       │
│  │  ├── App Templates (Java, Node, Python)       │       │
│  │  ├── CI/CD Pipeline Templates                 │       │
│  │  ├── Infrastructure Modules (Terraform)       │       │
│  │  ├── Observability Presets                    │       │
│  │  └── Security Baseline Configs                │       │
│  └──────────────────┬───────────────────────────┘       │
│                      │                                   │
│  ┌──────────────────▼───────────────────────────┐       │
│  │  INFRASTRUCTURE LAYER                         │       │
│  │  ├── Kubernetes Clusters                      │       │
│  │  ├── Databases (managed)                      │       │
│  │  ├── Message Queues                           │       │
│  │  ├── Secrets Management                       │       │
│  │  └── Networking & Service Mesh                │       │
│  └──────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────┘
```

---

## Golden Paths

```
┌──────────────────────────────────────────────────────────┐
│  GOLDEN PATHS (Paved Roads)                              │
│                                                          │
│  "The easy way should also be the right way"             │
│                                                          │
│  WHAT:                                                   │
│  Opinionated, pre-built, well-supported workflows        │
│  that teams CHOOSE to follow (not mandated).             │
│                                                          │
│  EXAMPLE — "Deploy a New Microservice":                  │
│                                                          │
│  Step 1: Developer goes to portal                        │
│     │                                                    │
│     ▼                                                    │
│  Step 2: Selects template (e.g., "Java REST API")        │
│     │                                                    │
│     ▼                                                    │
│  Step 3: Fills in: service name, team, tier               │
│     │                                                    │
│     ▼                                                    │
│  Step 4: Platform auto-generates:                        │
│     ├── Git repo with project scaffold                   │
│     ├── CI/CD pipeline (GitHub Actions)                  │
│     ├── Dockerfile + K8s manifests                       │
│     ├── Terraform for database + secrets                 │
│     ├── Prometheus metrics + Grafana dashboard           │
│     ├── PagerDuty integration                            │
│     └── Documentation page in Backstage                  │
│     │                                                    │
│     ▼                                                    │
│  Step 5: Developer writes application code               │
│  (Everything else is handled by the platform!)           │
│                                                          │
│  KEY PRINCIPLE: Optional but incentivized.               │
│  Teams CAN deviate but rarely need to.                   │
└──────────────────────────────────────────────────────────┘
```

---

## Backstage Developer Portal (Example)

```yaml
# backstage-template.yaml — Service Template
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: java-microservice
  title: Java Microservice
  description: Creates a production-ready Java microservice
  tags: ['java', 'spring-boot', 'recommended']
spec:
  owner: platform-team
  type: service

  parameters:
    - title: Service Details
      required: ['name', 'owner', 'tier']
      properties:
        name:
          title: Service Name
          type: string
          pattern: '^[a-z][a-z0-9-]*$'
        owner:
          title: Owning Team
          type: string
          ui:field: OwnerPicker
        tier:
          title: Service Tier
          type: string
          enum: ['tier-1-critical', 'tier-2-important', 'tier-3-standard']
          description: Determines SLO targets and on-call requirements
        description:
          title: Description
          type: string

  steps:
    - id: generate
      name: Generate Project
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          owner: ${{ parameters.owner }}

    - id: publish
      name: Create GitHub Repository
      action: publish:github
      input:
        repoUrl: github.com?owner=myorg&repo=${{ parameters.name }}
        defaultBranch: main

    - id: create-infrastructure
      name: Provision Infrastructure
      action: terraform:apply
      input:
        module: modules/microservice
        variables:
          service_name: ${{ parameters.name }}
          tier: ${{ parameters.tier }}

    - id: register
      name: Register in Backstage
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: /catalog-info.yaml
```

---

## DevOps Engineer vs SRE vs Platform Engineer

```
┌──────────────────────────────────────────────────────────┐
│  COMPARING THE THREE ROLES                               │
│                                                          │
│  ┌────────────┬──────────────┬───────────────────┐      │
│  │ DevOps Eng │     SRE      │ Platform Engineer │      │
│  ├────────────┼──────────────┼───────────────────┤      │
│  │ CI/CD &    │ Reliability  │ Developer         │      │
│  │ Automation │ & Uptime     │ Experience        │      │
│  ├────────────┼──────────────┼───────────────────┤      │
│  │ Serves:    │ Serves:      │ Serves:           │      │
│  │ Dev teams  │ End users    │ Dev teams         │      │
│  │ & org      │ (via SLOs)   │ (as customers)    │      │
│  ├────────────┼──────────────┼───────────────────┤      │
│  │ Builds:    │ Builds:      │ Builds:           │      │
│  │ Pipelines  │ Resilience   │ Self-service      │      │
│  │ & infra    │ & monitoring │ platform          │      │
│  ├────────────┼──────────────┼───────────────────┤      │
│  │ Measures:  │ Measures:    │ Measures:         │      │
│  │ DORA       │ SLOs, error  │ Dev productivity, │      │
│  │ metrics    │ budgets      │ adoption rate     │      │
│  ├────────────┼──────────────┼───────────────────┤      │
│  │ Key skill: │ Key skill:   │ Key skill:        │      │
│  │ Automation │ Distributed  │ Product thinking  │      │
│  │            │ systems      │ (platform=product)│      │
│  └────────────┴──────────────┴───────────────────┘      │
│                                                          │
│  Many orgs blend these roles. That's fine.               │
│  What matters is WHICH problems you're solving.          │
└──────────────────────────────────────────────────────────┘
```

---

## Platform Team Key Metrics

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Developer Satisfaction** | How happy devs are with platform | > 4/5 survey score |
| **Onboarding Time** | Time for new service to reach production | < 1 day |
| **Golden Path Adoption** | % of teams using platform templates | > 80% |
| **Self-Service Rate** | % of requests handled without tickets | > 90% |
| **Platform Reliability** | Uptime of platform services | 99.9%+ |
| **Cognitive Load** | Survey: ease of shipping to production | Improving trend |

---

## Real-World Scenario: Building an IDP

```
SCENARIO: 200-developer org, 40 microservices, growing fast

BEFORE:
├── Every team configures their own K8s manifests
├── 5 different CI/CD approaches across teams
├── New service takes 2-3 weeks to reach production
├── "How do I deploy?" questions flood Slack daily
├── Security misconfigurations found in production
└── DevOps team (3 people) is bottleneck for everything

PLATFORM ENGINEERING APPROACH:

Quarter 1 — Foundation:
├── Deploy Backstage as developer portal
├── Create service catalog (register all 40 services)
├── Build first golden path: "Node.js microservice"
├── Include: repo template, CI/CD, K8s manifests, monitoring
└── Pilot with 2 willing teams

Quarter 2 — Expand:
├── Add golden paths for Java, Python
├── Build Terraform modules (database, cache, queue)
├── Create self-service secrets management
├── Add cost visibility per team/service in portal
└── Onboard 10 more teams

Quarter 3 — Mature:
├── Add API catalog and documentation hub
├── Implement security baseline scanning
├── Build environment promotion workflow
├── Add SLO dashboard integration
└── Gather developer feedback, iterate

AFTER:
├── New service: idea → production in 4 hours (from 3 weeks)
├── 85% golden path adoption (voluntary)
├── Security baselines baked in by default
├── DevOps/Platform team scales to support 200+ devs
├── Developer satisfaction: 3.1 → 4.5 / 5.0
└── Zero "how do I deploy?" questions in Slack
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Low platform adoption | Too rigid, doesn't fit real use cases | Treat platform as product; get user feedback; iterate |
| "Mandatory platform" resistance | Mandate instead of enabling | Make golden paths optional but compelling |
| Platform becomes bottleneck | Platform team reviews all changes | Build self-service; CI/CD for platform itself |
| Platform too complex | Building for edge cases first | Start simple; solve the 80% use case; extend later |
| Devs work around the platform | Platform doesn't solve real pain | Shadow developers; identify actual friction points |
| Platform team burnout | Too much scope, too few people | Prioritize ruthlessly; say no to non-core requests |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Platform Engineering** | Building internal developer platforms for self-service |
| **IDP** | Internal Developer Platform — tools + workflows for dev teams |
| **Golden Paths** | Pre-built, opinionated, optional workflows for common tasks |
| **Backstage** | Open-source developer portal by Spotify |
| **Service Catalog** | Registry of all services, owners, docs, APIs |
| **Product Thinking** | Treat the platform as a product; developers are customers |
| **Cognitive Load** | Platform reduces the knowledge burden on dev teams |
| **Self-Service** | Developers provision what they need without tickets |

---

## Quick Revision Questions

1. **What is Platform Engineering and why has it emerged?**
   <details><summary>Answer</summary>Platform Engineering is the discipline of building Internal Developer Platforms (IDPs) that provide self-service toolchains for development teams. It emerged because DevOps at scale creates cognitive overload — developers are expected to know Kubernetes, Terraform, CI/CD, monitoring, security, etc. Platform engineering abstracts this complexity behind golden paths, making "the right way the easy way."</details>

2. **What are golden paths and why should they be optional?**
   <details><summary>Answer</summary>Golden paths (paved roads) are pre-built, opinionated workflows for common tasks like creating a new microservice or deploying to production. They should be optional because: 1) Mandates create resistance. 2) Edge cases will always exist. 3) Voluntary adoption signals genuine value. 4) Developer autonomy matters for innovation. The goal is to make golden paths so good that teams choose them naturally.</details>

3. **How does Platform Engineer differ from DevOps Engineer and SRE?**
   <details><summary>Answer</summary>DevOps Engineer focuses on CI/CD pipelines and automation for teams. SRE focuses on reliability, SLOs, and incident management for end users. Platform Engineer focuses on developer experience, building self-service platforms that treat developers as customers. The key differentiator: platform engineers apply product thinking — they do user research, measure adoption, and iterate like a product team.</details>

4. **What is Backstage and how does it serve as a developer portal?**
   <details><summary>Answer</summary>Backstage is an open-source developer portal created by Spotify. It provides: 1) Service Catalog — registry of all services with owners, docs, APIs. 2) Software Templates — scaffolding wizards for new services. 3) TechDocs — documentation hub. 4) Plugin ecosystem — integrate any tool (CI/CD, monitoring, cost). It serves as the single pane of glass for developer self-service.</details>

5. **What metrics should a platform team track?**
   <details><summary>Answer</summary>Key metrics: 1) Developer Satisfaction — survey score (target >4/5). 2) Onboarding Time — new service to production (target <1 day). 3) Golden Path Adoption — % teams using templates (target >80%). 4) Self-Service Rate — requests without tickets (target >90%). 5) Platform Reliability — uptime of platform services (99.9%+). 6) Cognitive Load — surveyed ease of shipping to production.</details>

6. **What are common mistakes when building an IDP?**
   <details><summary>Answer</summary>1) Mandating the platform instead of making it compelling. 2) Building for edge cases first instead of the 80% use case. 3) Not getting developer feedback (building in isolation). 4) Making it too complex from day one. 5) Platform team becoming a bottleneck (not enough self-service). 6) Not treating it as a product (no roadmap, no user research, no iteration). Start simple, pilot early, iterate based on real usage.</details>

---

[← Previous: Site Reliability Engineer](02-sre.md) | [Next: Team Topologies →](04-team-topologies.md)
