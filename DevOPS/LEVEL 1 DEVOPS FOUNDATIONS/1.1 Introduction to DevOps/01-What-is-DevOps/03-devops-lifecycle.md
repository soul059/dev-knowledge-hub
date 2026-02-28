# Chapter 1.3 — The DevOps Lifecycle

[← Previous: DevOps vs Traditional IT](02-devops-vs-traditional-it.md) | [Next: DevOps Culture and Mindset →](04-devops-culture-and-mindset.md)

---

## Overview

The DevOps lifecycle is a continuous, iterative loop that connects development and operations activities. Often represented as an **infinity loop (∞)**, it illustrates how software flows from idea to production and how feedback loops drive continuous improvement.

---

## The DevOps Infinity Loop

```
            ┌──────────────────────────────────────────┐
            │              DevOps Lifecycle             │
            │                                          │
            │         DEV                OPS           │
            │     ┌─────────┐      ┌─────────┐        │
            │    ╱           ╲    ╱           ╲       │
            │   │  Plan       │  │  Deploy     │      │
            │   │      Code   │  │      Operate│      │
            │    ╲           ╱    ╲           ╱       │
            │     └─────────┘      └─────────┘        │
            │     ┌─────────┐      ┌─────────┐        │
            │    ╱           ╲    ╱           ╲       │
            │   │  Build      │  │  Monitor   │       │
            │   │      Test   │  │      Learn │       │
            │    ╲           ╱    ╲           ╱       │
            │     └─────────┘      └─────────┘        │
            │                                          │
            │        ◄──── Continuous Flow ────►       │
            └──────────────────────────────────────────┘
```

---

## The 8 Phases of the DevOps Lifecycle

### Phase 1: PLAN

```
+----------------------------------------------------------+
|  PLAN                                                     |
|  ═════                                                    |
|                                                           |
|  Inputs:                    Outputs:                      |
|  ┌─────────────────┐       ┌──────────────────────┐      |
|  │ Business needs   │       │ User stories          │      |
|  │ Customer feedback│  ──►  │ Sprint backlog         │      |
|  │ Monitoring data  │       │ Roadmap                │      |
|  │ Incident reports │       │ Architecture decisions │      |
|  └─────────────────┘       └──────────────────────┘      |
|                                                           |
|  Tools: Jira, Azure Boards, Trello, GitHub Issues         |
+----------------------------------------------------------+
```

**Key Activities:**
- Define requirements and user stories
- Prioritize backlog items
- Sprint planning and capacity planning
- Architecture and design reviews

### Phase 2: CODE

```
+----------------------------------------------------------+
|  CODE                                                     |
|  ═════                                                    |
|                                                           |
|  Developer Workflow:                                      |
|                                                           |
|  main ─────────●────────────●──────── (stable)            |
|                 \          /                               |
|  feature/login   ●──●──●─●   (branch, commit, PR)        |
|                                                           |
|  Best Practices:                                          |
|  ├── Feature branching                                    |
|  ├── Code reviews (Pull Requests)                         |
|  ├── Pair/mob programming                                 |
|  └── Coding standards & linting                           |
|                                                           |
|  Tools: Git, GitHub, GitLab, Bitbucket, VS Code           |
+----------------------------------------------------------+
```

**Key Commands:**
```bash
# Create feature branch
git checkout -b feature/user-auth

# Stage and commit changes
git add .
git commit -m "feat: add user authentication"

# Push and create PR
git push origin feature/user-auth
```

### Phase 3: BUILD

```
+----------------------------------------------------------+
|  BUILD                                                    |
|  ══════                                                   |
|                                                           |
|  Source Code ──► Compile ──► Package ──► Artifact         |
|                                                           |
|  ┌──────────┐   ┌──────────┐   ┌──────────────────┐     |
|  │ .java    │   │ javac    │   │ app-1.0.jar      │     |
|  │ .py      │──►│ pip      │──►│ app:latest (img) │     |
|  │ .js      │   │ npm      │   │ app-1.0.tar.gz   │     |
|  └──────────┘   └──────────┘   └──────────────────┘     |
|                                                           |
|  Artifact Repository: Nexus, JFrog, Docker Registry       |
|  Tools: Maven, Gradle, npm, pip, Docker                   |
+----------------------------------------------------------+
```

**Key Activities:**
- Compile source code
- Resolve dependencies
- Create build artifacts (JAR, Docker image, binary)
- Version the artifacts

### Phase 4: TEST

```
+----------------------------------------------------------+
|  TEST                                                     |
|  ═════                                                    |
|                                                           |
|  The Testing Pyramid:                                     |
|                                                           |
|              /\                                           |
|             /  \        E2E / UI Tests (Few, Slow)        |
|            /    \       - Selenium, Cypress                |
|           /──────\                                        |
|          /        \     Integration Tests (Some)          |
|         /          \    - API tests, DB tests             |
|        /────────────\                                     |
|       /              \   Unit Tests (Many, Fast)          |
|      /________________\  - JUnit, pytest, Jest            |
|                                                           |
|  Additional Testing:                                      |
|  ├── Security scanning (SAST, DAST)                       |
|  ├── Performance / Load testing                           |
|  ├── Contract testing                                     |
|  └── Chaos testing                                        |
+----------------------------------------------------------+
```

**Example — Running Tests in CI:**
```yaml
# GitHub Actions example
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Run unit tests
      run: npm test
    - name: Run integration tests
      run: npm run test:integration
    - name: Run security scan
      run: npm audit
```

### Phase 5: RELEASE

```
+----------------------------------------------------------+
|  RELEASE                                                  |
|  ════════                                                 |
|                                                           |
|  Build Artifact ──► Approval ──► Release Tag              |
|                                                           |
|  Strategies:                                              |
|  ├── Manual approval gate (staging → prod)                |
|  ├── Automated release (if all tests pass)                |
|  └── Semantic versioning (v1.2.3)                         |
|                                                           |
|  ┌──────────────────────────────────────┐                 |
|  │  v1.0.0 ──► v1.1.0 ──► v1.2.0       │                 |
|  │  MAJOR.MINOR.PATCH                   │                 |
|  │                                       │                 |
|  │  MAJOR: Breaking changes              │                 |
|  │  MINOR: New features (backward compat)│                 |
|  │  PATCH: Bug fixes                     │                 |
|  └──────────────────────────────────────┘                 |
+----------------------------------------------------------+
```

### Phase 6: DEPLOY

```
+----------------------------------------------------------+
|  DEPLOY                                                   |
|  ═══════                                                  |
|                                                           |
|  Deployment Strategies:                                   |
|                                                           |
|  1. Rolling Update          2. Blue-Green                 |
|  ┌───┬───┬───┬───┐         ┌─────────┐ ┌─────────┐      |
|  │v1 │v1 │v2 │v2 │         │ BLUE    │ │ GREEN   │      |
|  │ ↓ │ ↓ │   │   │         │ (v1)    │ │ (v2)    │      |
|  │v2 │v2 │v2 │v2 │         │ Active  │ │ Standby │      |
|  └───┴───┴───┴───┘         └────┬────┘ └────┬────┘      |
|  Gradual replacement              └──switch──┘            |
|                                                           |
|  3. Canary                  4. Feature Flags              |
|  ┌──────────────────┐      ┌──────────────────┐          |
|  │ 95% ──► v1       │      │ if (flag.enabled) │          |
|  │  5% ──► v2       │      │   showNewUI()     │          |
|  │ Monitor metrics   │      │ else              │          |
|  │ Gradually increase│      │   showOldUI()     │          |
|  └──────────────────┘      └──────────────────┘          |
|                                                           |
|  Tools: ArgoCD, Spinnaker, Kubernetes, AWS CodeDeploy     |
+----------------------------------------------------------+
```

### Phase 7: OPERATE

```
+----------------------------------------------------------+
|  OPERATE                                                  |
|  ════════                                                 |
|                                                           |
|  ┌────────────────────────────────────────┐               |
|  │           Production Environment       │               |
|  │                                        │               |
|  │   ┌──────┐  ┌──────┐  ┌──────┐       │               |
|  │   │ App  │  │ App  │  │ App  │       │               |
|  │   │Pod 1 │  │Pod 2 │  │Pod 3 │       │               |
|  │   └──┬───┘  └──┬───┘  └──┬───┘       │               |
|  │      └─────────┼─────────┘            │               |
|  │           Load Balancer               │               |
|  │                │                      │               |
|  │   ┌────────────┴──────────────┐       │               |
|  │   │  Auto-scaling · Backups   │       │               |
|  │   │  Patching · Security      │       │               |
|  │   └───────────────────────────┘       │               |
|  └────────────────────────────────────────┘               |
|                                                           |
|  Key Activities: Scaling, patching, incident response     |
+----------------------------------------------------------+
```

### Phase 8: MONITOR & LEARN

```
+----------------------------------------------------------+
|  MONITOR & LEARN                                          |
|  ════════════════                                         |
|                                                           |
|  Three Pillars of Observability:                          |
|                                                           |
|  ┌──────────┐  ┌──────────┐  ┌──────────┐               |
|  │  LOGS    │  │ METRICS  │  │ TRACES   │               |
|  │          │  │          │  │          │               |
|  │ What     │  │ How much │  │ Where    │               |
|  │ happened │  │ /how fast│  │ (path)   │               |
|  └──────────┘  └──────────┘  └──────────┘               |
|       │              │              │                     |
|       └──────────────┼──────────────┘                     |
|                      ▼                                    |
|              ┌──────────────┐                             |
|              │  DASHBOARDS  │                             |
|              │  & ALERTS    │                             |
|              └──────┬───────┘                             |
|                     ▼                                     |
|              Feed back to PLAN phase                      |
|                                                           |
|  Tools: Prometheus, Grafana, ELK, Datadog, PagerDuty     |
+----------------------------------------------------------+
```

---

## How the Phases Connect: End-to-End Flow

```
Developer          CI/CD Pipeline              Production
─────────          ──────────────              ──────────

  commit ──►  ┌──────────────────────┐
              │ 1. Build             │
              │ 2. Unit Tests        │
              │ 3. Security Scan     │
              │ 4. Integration Tests │
              │ 5. Package Artifact  │
              │ 6. Deploy to Staging │
              │ 7. Smoke Tests       │
              │ 8. Deploy to Prod    │ ──►  Running in
              └──────────────────────┘      Production
                                                │
  ◄──────── Alert / Dashboard ◄────────── Monitoring
  (feedback)                              (metrics, logs)
```

---

## Real-World Scenario: Full Lifecycle Example

### 🏢 Scenario: Adding a "Forgot Password" Feature

```
PLAN:    Product owner creates user story: "As a user, I want to reset
         my password via email." Added to sprint backlog in Jira.

CODE:    Developer creates branch 'feature/forgot-password',
         implements the feature, opens a Pull Request on GitHub.

BUILD:   GitHub Actions triggers: compiles code, resolves dependencies,
         builds Docker image → pushes to Docker Registry.

TEST:    Pipeline runs 47 unit tests, 12 integration tests, 
         3 E2E tests, and a security scan. All pass.

RELEASE: Version bumped to v2.3.0, release notes auto-generated.

DEPLOY:  Canary deployment to 5% of users. Monitors error rates
         for 15 minutes → no issues → rolls out to 100%.

OPERATE: Auto-scaler handles increased load from password reset
         emails. No manual intervention needed.

MONITOR: Grafana dashboard shows: 150 password resets in first hour,
         0 errors, avg response time 200ms.
         Insight fed back to PLAN: "Add rate limiting for reset emails."
```

---

## Troubleshooting Tips

| Phase | Common Issue | Solution |
|-------|-------------|----------|
| **Plan** | Unclear requirements | Use user story format: "As a [role], I want [feature], so that [benefit]" |
| **Code** | Merge conflicts | Use short-lived branches, merge main frequently |
| **Build** | "Works on my machine" | Use containers (Docker) for consistent environments |
| **Test** | Flaky tests | Isolate test dependencies, use test containers |
| **Release** | Version confusion | Adopt semantic versioning (SemVer) strictly |
| **Deploy** | Deployment failures | Use blue-green or canary deployments; always have rollback |
| **Operate** | Manual scaling | Implement auto-scaling policies (HPA in Kubernetes) |
| **Monitor** | Alert fatigue | Tune alert thresholds, use severity levels |

---

## Summary Table

| Phase | Purpose | Key Tools |
|-------|---------|-----------|
| **Plan** | Define what to build | Jira, Azure Boards, GitHub Issues |
| **Code** | Write and review code | Git, GitHub, VS Code |
| **Build** | Compile and package | Maven, Docker, npm |
| **Test** | Validate quality and security | JUnit, Selenium, SonarQube |
| **Release** | Version and approve | GitHub Releases, Semantic Versioning |
| **Deploy** | Ship to production | ArgoCD, Kubernetes, AWS CodeDeploy |
| **Operate** | Run and maintain | Kubernetes, Terraform, Ansible |
| **Monitor** | Observe and learn | Prometheus, Grafana, ELK, PagerDuty |

---

## Quick Revision Questions

1. **Name the 8 phases of the DevOps lifecycle in order.**
   <details><summary>Answer</summary>Plan → Code → Build → Test → Release → Deploy → Operate → Monitor (& Learn). These form a continuous loop.</details>

2. **Why is the DevOps lifecycle represented as an infinity loop?**
   <details><summary>Answer</summary>Because the process is continuous and iterative. Feedback from monitoring feeds back into planning, creating a never-ending cycle of improvement. There is no "end" — only continuous delivery and learning.</details>

3. **What are the four common deployment strategies? Briefly describe each.**
   <details><summary>Answer</summary>1) Rolling Update: gradually replace old instances with new ones. 2) Blue-Green: maintain two identical environments, switch traffic. 3) Canary: deploy to a small percentage first, monitor, then expand. 4) Feature Flags: deploy code but toggle features on/off without redeployment.</details>

4. **What are the three pillars of observability?**
   <details><summary>Answer</summary>Logs (what happened), Metrics (how much/how fast), and Traces (the path of a request through the system). Together they provide full visibility into system behavior.</details>

5. **How does the Test phase in DevOps differ from traditional QA?**
   <details><summary>Answer</summary>In DevOps, testing is automated, continuous, and runs on every commit (shift-left). In traditional QA, testing is manual, done by a separate team, and occurs late in the cycle. DevOps uses the testing pyramid: many fast unit tests, fewer integration tests, and minimal E2E tests.</details>

6. **In the lifecycle example, why was a canary deployment chosen over a blue-green deployment?**
   <details><summary>Answer</summary>Canary deployments allow you to test with real production traffic on a small percentage of users, gathering real-world metrics before full rollout. This is less risky than blue-green (which switches all traffic at once) and provides gradual confidence in the new release.</details>

---

[← Previous: DevOps vs Traditional IT](02-devops-vs-traditional-it.md) | [Next: DevOps Culture and Mindset →](04-devops-culture-and-mindset.md)
