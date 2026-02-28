# Chapter 9.6: Choosing a CI/CD Tool

[← Previous: CI/CD Tool Comparison](05-comparison-matrix.md) | [Back to README](../README.md)

---

## Overview

Choosing the right CI/CD tool depends on your team size, tech stack, cloud provider, compliance needs, and existing ecosystem. This chapter provides a structured decision framework to guide that choice.

---

## Decision Framework

```
  CI/CD TOOL DECISION TREE
  ═════════════════════════

  Start Here
     │
     ▼
  Using GitHub for source code?
  ├── YES → GitHub Actions (primary CI)
  │         └── Need GitOps K8s CD? → + ArgoCD
  │
  └── NO
      │
      ▼
  Want all-in-one platform?
  ├── YES → GitLab CI/CD (self-managed or .com)
  │
  └── NO
      │
      ▼
  Microsoft / Azure ecosystem?
  ├── YES → Azure Pipelines
  │
  └── NO
      │
      ▼
  Need maximum flexibility / self-hosted?
  ├── YES → Jenkins
  │
  └── NO
      │
      ▼
  Need fastest test feedback?
  ├── YES → CircleCI
  │
  └── NO
      │
      ▼
  Building K8s-native platform?
  ├── YES → Tekton (CI) + ArgoCD (CD)
  │
  └── NO → Start with GitHub Actions or GitLab CI
```

---

## Evaluation Criteria

```
  ┌─────────────────────┬───────────────────────────────────────────────┐
  │ Criterion           │ Questions to Ask                              │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Source Control       │ Where is your code? (GitHub/GitLab/Azure/BB) │
  │ Integration          │ Native integration reduces friction          │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Team Size           │ Small (<10): simplicity wins                  │
  │                     │ Medium (10-50): balance features + ease       │
  │                     │ Large (50+): enterprise features matter       │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Cloud Provider      │ AWS → GitHub Actions / GitLab / CircleCI     │
  │                     │ Azure → Azure Pipelines                       │
  │                     │ GCP → GitHub Actions / GitLab / CloudBuild   │
  │                     │ Multi-cloud → Jenkins / Tekton                │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Deployment Target   │ Kubernetes → ArgoCD + any CI                  │
  │                     │ Serverless → Cloud-specific CI                │
  │                     │ VMs → Jenkins / GitLab / any                  │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Compliance          │ Audit trails → GitLab / Azure Pipelines       │
  │                     │ Air-gapped → Jenkins / GitLab self-managed    │
  │                     │ SOC2/FedRAMP → Check vendor certifications    │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Budget              │ Open source → Jenkins, Tekton, ArgoCD         │
  │                     │ Freemium → GitHub Actions, GitLab, CircleCI   │
  │                     │ Enterprise → GitLab Ultimate, Azure DevOps    │
  ├─────────────────────┼───────────────────────────────────────────────┤
  │ Operational         │ Managed → Cloud-hosted tools (Actions/GitLab) │
  │ Capacity            │ Self-managed → Jenkins / Tekton (need ops)    │
  └─────────────────────┴───────────────────────────────────────────────┘
```

---

## Recommendations by Team Profile

### Startup / Small Team (2-15 Engineers)

```
  RECOMMENDED: GitHub Actions + ArgoCD

  Why:
  ├── Zero infrastructure to manage
  ├── Free tier covers small teams
  ├── GitHub integration (already using it)
  ├── Large marketplace for quick setup
  └── ArgoCD for K8s deployments (if applicable)

  Alternative: GitLab CI (if preferring all-in-one)
```

### Mid-Size Company (15-100 Engineers)

```
  RECOMMENDED: GitLab CI/CD (self-managed) or GitHub Actions

  Why:
  ├── Built-in security scanning (GitLab)
  ├── Environment management
  ├── Group-level CI/CD templates
  ├── Compliance features
  └── Self-hosted option for data control

  Alternative: Azure Pipelines (if Microsoft shop)
```

### Enterprise (100+ Engineers)

```
  RECOMMENDED: GitLab Ultimate / Azure DevOps / Jenkins (existing)

  Why:
  ├── Advanced compliance and audit
  ├── Role-based access control
  ├── Template governance
  ├── Multi-project/portfolio management
  ├── Enterprise support contracts
  └── Integration with enterprise tools (LDAP, SSO)

  Alternative: Jenkins (if already invested) with ArgoCD for GitOps
```

### Platform Engineering Team

```
  RECOMMENDED: Tekton + ArgoCD

  Why:
  ├── Kubernetes-native — runs where apps run
  ├── No external dependencies
  ├── Cloud-agnostic / portable
  ├── Fine-grained RBAC via K8s
  └── Foundation for internal developer platform

  Alternative: GitLab CI if K8s-only isn't a requirement
```

---

## Migration Strategies

```
  MIGRATION APPROACHES
  ════════════════════

  1. Big Bang (high risk):
     Old Tool ──────────▶ New Tool
     └── All projects migrate at once

  2. Strangler Fig (recommended):
     Old Tool ─────────────────────▶ (decommission)
         ├── Project A ──▶ New Tool
         ├── Project B ──▶ New Tool
         └── Project C ──▶ New Tool
     └── Gradual migration, project by project

  3. Parallel Run (safest):
     Old Tool ──▶ (keep running)
     New Tool ──▶ (set up alongside)
     └── Compare results, then switch over
```

### Migration Best Practices

```yaml
# 1. Abstract logic into scripts (tool-agnostic)
# scripts/build.sh
#!/bin/bash
set -euo pipefail
npm ci
npm run build
npm run test

# 2. Thin CI config calls scripts
# GitHub Actions:
- run: ./scripts/build.sh

# GitLab CI:
script: ./scripts/build.sh

# Jenkins:
steps { sh './scripts/build.sh' }

# This makes migration = rewriting orchestration YAML only
```

---

## Cost Comparison

```
  ┌─────────────────┬──────────────────────────────────────┐
  │ Tool            │ Pricing Model                        │
  ├─────────────────┼──────────────────────────────────────┤
  │ Jenkins         │ Free (self-host infrastructure cost) │
  │ GitHub Actions  │ 2,000 min/mo free, then $0.008/min   │
  │ GitLab CI       │ 400 min/mo free, tiers from $29/user │
  │ Azure Pipelines │ 1 parallel job free, $40/parallel    │
  │ CircleCI        │ 6,000 credits/mo free, usage-based   │
  │ ArgoCD          │ Free (self-host)                     │
  │ Tekton          │ Free (self-host)                     │
  └─────────────────┴──────────────────────────────────────┘

  Hidden costs to consider:
  ├── Self-hosted: Infrastructure + ops engineer time
  ├── Cloud-hosted: Minutes/credits at scale
  ├── Enterprise tiers: Per-user licensing
  └── Migration: Engineering time to switch tools
```

---

## Future-Proofing

```
  TRENDS IN CI/CD (2024+)
  ════════════════════════

  1. GitOps becoming standard for K8s deployments
     → ArgoCD / Flux adoption growing fast

  2. AI-assisted CI/CD
     → Auto-fix failing tests, smart test selection
     → GitHub Copilot for CI config

  3. Platform Engineering
     → Internal developer platforms (IDP)
     → Backstage + Tekton + ArgoCD

  4. Supply chain security
     → SLSA, Sigstore, in-toto attestations
     → Built into CI pipelines

  5. Ephemeral environments
     → Preview environments per PR/MR
     → Standard across all tools

  GUIDING PRINCIPLE:
  Choose a tool that fits today but can evolve.
  Minimize lock-in by keeping logic in scripts.
```

---

## Real-World Scenario

> **Scenario**: A fintech company (40 engineers) is using Jenkins with 200+ freestyle jobs. Builds are slow, config is inconsistent, and the team spends 2 days/month on Jenkins maintenance.
>
> **Solution**: Migrate to GitLab CI/CD (self-managed) for regulatory compliance. Phase 1: Set up GitLab and migrate 5 pilot projects using the strangler fig pattern. Phase 2: Create shared CI templates for build/test/security scanning at the group level. Phase 3: Migrate remaining projects over 3 months, converting freestyle jobs to `.gitlab-ci.yml`. Phase 4: Decommission Jenkins. Keep build logic in shell scripts for portability. Result: 80% reduction in CI/CD maintenance, consistent security scanning across all projects, full audit trail for compliance.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Analysis paralysis | Too many tools | Start with source control alignment (GitHub→Actions) |
| Migration takes too long | Big bang approach | Use strangler fig — migrate project by project |
| Team pushback | Unfamiliar tool | Run training sessions; pilot with willing team |
| Unexpected costs | Usage-based pricing surprises | Monitor usage, set alerts, use self-hosted runners |
| Feature gaps | New tool missing old tool features | Document requirements before migrating; check roadmap |
| Multi-tool complexity | Different tools per team | Standardize org-wide; allow exceptions with justification |

---

## Summary Table

| Decision Factor | Recommendation |
|---|---|
| GitHub + small team | GitHub Actions |
| All-in-one + compliance | GitLab CI/CD |
| Azure ecosystem | Azure Pipelines |
| Maximum flexibility | Jenkins |
| Fast test feedback | CircleCI |
| Kubernetes GitOps | ArgoCD |
| Cloud-agnostic K8s CI | Tekton |
| Minimize lock-in | Scripts + thin CI config |
| Migration approach | Strangler fig (gradual) |

---

## Quick Revision Questions

1. **What is the most important factor when choosing a CI/CD tool?** *Source control integration — choosing a tool native to your SCM platform (GitHub→Actions, GitLab→GitLab CI) reduces friction and provides the best developer experience.*
2. **How does team size affect tool selection?** *Small teams benefit from managed/cloud tools (low ops overhead). Large enterprises need governance, compliance, and template management (GitLab Ultimate, Azure DevOps).*
3. **What is the strangler fig migration pattern?** *Gradually migrating projects from the old CI/CD tool to the new one, project by project, rather than switching everything at once. Reduces risk and allows learning.*
4. **How do you minimize vendor lock-in?** *Keep build/test/deploy logic in shell scripts or Makefiles. Use CI/CD tool YAML only for orchestration (triggers, ordering, environments). This makes migration a matter of rewriting config, not logic.*
5. **When should you combine CI and CD tools?** *When deploying to Kubernetes with GitOps — use a CI tool (GitHub Actions, GitLab CI) for build/test and ArgoCD for deployment. Each tool excels at its role.*
6. **What hidden costs should you consider?** *Self-hosted infrastructure and operations time, cloud-hosted minutes at scale, per-user enterprise licensing, and engineering time for migration.*

---

[← Previous: CI/CD Tool Comparison](05-comparison-matrix.md) | [Back to README](../README.md)
