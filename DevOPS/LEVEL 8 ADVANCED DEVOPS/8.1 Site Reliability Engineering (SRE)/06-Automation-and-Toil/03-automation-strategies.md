# Chapter 6.3: Automation Strategies

[← Previous: Measuring Toil](02-measuring-toil.md) | [Back to README](../README.md) | [Next: Reducing Operational Burden →](04-reducing-operational-burden.md)

---

## Overview

Automation is the primary weapon against toil. However, not all automation is created equal. Effective automation strategies consider the automation hierarchy (from no automation to fully autonomous systems), choose appropriate tools, implement safely with proper guard rails, and measure impact. The goal is to move from manual processes toward self-service and self-healing systems.

---

## 1. The Automation Hierarchy

```
  ┌──────────────────────────────────────────────────────────┐
  │  AUTOMATION MATURITY LEVELS                              │
  │                                                          │
  │  Level 5: ★ AUTONOMOUS                                   │
  │  │  System detects, decides, and acts without humans     │
  │  │  Example: Auto-scaling, self-healing pods             │
  │  │                                                       │
  │  Level 4: SYSTEM-INITIATED + HUMAN APPROVAL              │
  │  │  System proposes action, human approves               │
  │  │  Example: "Scale up? [Approve] [Deny]"               │
  │  │                                                       │
  │  Level 3: HUMAN-INITIATED + AUTOMATED EXECUTION          │
  │  │  Human triggers, automation does the work             │
  │  │  Example: Click "Deploy" → pipeline runs              │
  │  │                                                       │
  │  Level 2: DOCUMENTED (RUNBOOK)                           │
  │  │  Written steps, human executes manually               │
  │  │  Example: Step-by-step restart procedure              │
  │  │                                                       │
  │  Level 1: TRIBAL KNOWLEDGE                               │
  │     "Ask Bob, he knows how to do it"                     │
  │     Worst state — knowledge locked in one person         │
  │                                                          │
  │  TARGET: Move everything to Level 3+                     │
  │  IDEAL: Level 4-5 for routine operations                 │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Automation Decision Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │               SHOULD YOU AUTOMATE IT?                    │
  │                                                          │
  │  Is it done more than once?                              │
  │  ├─ NO  ──▶ Don't automate. Document it instead.        │
  │  └─ YES                                                  │
  │      Does it take > 15 minutes?                          │
  │      ├─ NO                                               │
  │      │   Is it done > 10× per week?                     │
  │      │   ├─ YES ──▶ AUTOMATE (small but frequent)       │
  │      │   └─ NO  ──▶ Probably not worth it yet.          │
  │      └─ YES                                              │
  │          Is it error-prone when done manually?           │
  │          ├─ YES ──▶ AUTOMATE (high priority ★)          │
  │          └─ NO  ──▶ AUTOMATE (medium priority)          │
  │                                                          │
  │  XKCD Rule of Thumb:                                     │
  │  ┌──────────┬──────────────────────────────────┐         │
  │  │ Frequency│  Justifiable automation time     │         │
  │  ├──────────┼──────────────────────────────────┤         │
  │  │ 5×/day   │  Up to 2 weeks of dev time       │         │
  │  │ Daily    │  Up to 1 week                    │         │
  │  │ Weekly   │  Up to 1 day                     │         │
  │  │ Monthly  │  Up to 4 hours                   │         │
  │  │ Yearly   │  Up to 30 minutes                │         │
  │  └──────────┴──────────────────────────────────┘         │
  │  (Assumes saving 5 minutes, recouped over 5 years)       │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Automation Patterns

```
  ┌──────────────────────────────────────────────────────────┐
  │  PATTERN          │ DESCRIPTION               │ EXAMPLE  │
  ├───────────────────┼───────────────────────────┼──────────┤
  │ Script            │ Simple scripted procedure │ Bash     │
  │ automation        │ triggered manually        │ deploy.sh│
  │                   │                           │          │
  │ Pipeline          │ Multi-stage automated     │ CI/CD,   │
  │ automation        │ workflow with gates       │ Jenkins  │
  │                   │                           │          │
  │ Event-driven      │ Triggers on events        │ Lambda   │
  │ automation        │ (webhooks, alerts)        │ on alert │
  │                   │                           │          │
  │ Policy-based      │ Declarative rules engine  │ OPA,     │
  │ automation        │ enforces desired state    │ Kyverno  │
  │                   │                           │          │
  │ Self-service      │ UI/API for users to       │ Internal │
  │ automation        │ perform operations        │ portal   │
  │                   │                           │          │
  │ Self-healing      │ Auto-detect and fix       │ K8s      │
  │ automation        │ without human input       │ operators│
  └───────────────────┴───────────────────────────┴──────────┘
```

---

## 4. Infrastructure as Code (IaC)

```yaml
# Terraform — declarative infrastructure automation
# Eliminates manual cloud resource provisioning

resource "aws_instance" "api_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  
  tags = {
    Name        = "api-server"
    Team        = "platform"
    Environment = "production"
    ManagedBy   = "terraform"
  }
  
  lifecycle {
    create_before_destroy = true  # Zero-downtime updates
  }
}

resource "aws_autoscaling_group" "api" {
  min_size         = 3
  max_size         = 20
  desired_capacity = 5
  
  launch_template {
    id      = aws_launch_template.api.id
    version = "$Latest"
  }
  
  tag {
    key                 = "AutoScaled"
    value               = "true"
    propagate_at_launch = true
  }
}

# Ansible — procedural configuration automation
# Eliminates manual server configuration
---
- name: Configure API servers
  hosts: api_servers
  become: true
  tasks:
    - name: Install required packages
      apt:
        name:
          - nginx
          - python3
          - prometheus-node-exporter
        state: present
        
    - name: Deploy application
      copy:
        src: /builds/api-server/latest/
        dest: /opt/api-server/
      notify: restart api-server
        
    - name: Ensure service is running
      systemd:
        name: api-server
        state: started
        enabled: true
```

---

## 5. CI/CD Pipeline Automation

```
  ┌──────────────────────────────────────────────────────────┐
  │  BEFORE AUTOMATION (manual deploy):                      │
  │                                                          │
  │  Developer → SSH to server → git pull → run build →      │
  │  copy artifacts → update config → restart service →      │
  │  check logs → update wiki → notify team                  │
  │  Time: 45-90 minutes    Error rate: ~15%                 │
  │                                                          │
  │  ═══════════════════════════════════════════════════════  │
  │                                                          │
  │  AFTER AUTOMATION (CI/CD pipeline):                      │
  │                                                          │
  │  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐  │
  │  │ Push │──▶│Build │──▶│ Test │──▶│Deploy│──▶│Verify│  │
  │  │ Code │   │      │   │      │   │Canary│   │Health│  │
  │  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘  │
  │    auto      auto       auto       auto       auto      │
  │                                                          │
  │  Time: 12-15 minutes   Error rate: ~1%                   │
  │  Human involvement: Approve PR + monitor canary          │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# GitHub Actions — CI/CD pipeline example
name: Deploy API Service
on:
  push:
    branches: [main]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run unit tests
        run: |
          go test ./... -v -race -coverprofile=coverage.out
          
      - name: Build container
        run: |
          docker build -t api-service:${{ github.sha }} .
          docker push $REGISTRY/api-service:${{ github.sha }}
          
      - name: Deploy canary (5%)
        run: |
          kubectl set image deploy/api-service-canary \
            api=api-service:${{ github.sha }}
          kubectl rollout status deploy/api-service-canary
          
      - name: Validate canary (10 min)
        run: |
          sleep 600
          ERROR_RATE=$(curl -s $PROMETHEUS/api/v1/query \
            --data-urlencode 'query=rate(http_errors{deploy="canary"}[10m])' \
            | jq '.data.result[0].value[1]')
          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "Canary failed! Rolling back."
            kubectl rollout undo deploy/api-service-canary
            exit 1
          fi
          
      - name: Full rollout
        run: |
          kubectl set image deploy/api-service \
            api=api-service:${{ github.sha }}
          kubectl rollout status deploy/api-service
```

---

## 6. Automation Safety Principles

```
  ┌──────────────────────────────────────────────────────────┐
  │  SAFE AUTOMATION PRINCIPLES                              │
  │                                                          │
  │  1. DRY RUN FIRST                                        │
  │     Always support --dry-run or --plan mode              │
  │     Show what WILL change before changing it             │
  │                                                          │
  │  2. BLAST RADIUS LIMITING                                │
  │     Don't automate "update all servers at once"          │
  │     Use rolling updates, canaries, percentage-based      │
  │                                                          │
  │  3. HUMAN GATES FOR RISKY OPERATIONS                     │
  │     Automation proposes, human approves                  │
  │     "Delete 500 VMs? [Confirm] [Cancel]"                 │
  │                                                          │
  │  4. ROLLBACK CAPABILITY                                  │
  │     Every automated change must be reversible            │
  │     Especially for: config changes, deployments, DNS     │
  │                                                          │
  │  5. AUDIT TRAIL                                          │
  │     Log WHO triggered WHAT, WHEN, and WHAT changed       │
  │     Immutable audit logs for compliance                  │
  │                                                          │
  │  6. RATE LIMITING                                        │
  │     Prevent automation from acting too fast              │
  │     "Max 3 rollbacks per hour" prevents cascading fixes  │
  │                                                          │
  │  7. DEAD MAN'S SWITCH                                    │
  │     If automation fails silently, alert on ABSENCE       │
  │     "No deploy in 7 days? Something may be broken."     │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Automating Certificate Renewal

```
  BEFORE: Manual SSL/TLS certificate renewal
  ├─ Frequency: 47 domains, certs expire every 90 days
  ├─ Process: Generate CSR → Submit to CA → Download cert 
  │           → Upload to servers → Reload nginx → Verify
  ├─ Time per cert: 30 minutes
  ├─ Total toil: 47 × 30min × 4/year = 94 hours/year
  ├─ Risk: 3 outages last year due to expired certs
  
  AFTER: cert-manager on Kubernetes
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  cert-manager watches Ingress resources                  │
  │       │                                                  │
  │       ▼                                                  │
  │  Detects cert nearing expiry (30 days before)            │
  │       │                                                  │
  │       ▼                                                  │
  │  Automatically requests cert from Let's Encrypt          │
  │       │                                                  │
  │       ▼                                                  │
  │  Stores in Kubernetes Secret                             │
  │       │                                                  │
  │       ▼                                                  │
  │  Ingress controller picks up new cert (zero downtime)    │
  │       │                                                  │
  │       ▼                                                  │
  │  Prometheus alert if renewal fails                       │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
  
  Implementation time: 2 days
  Toil eliminated: 94 hours/year
  Cert expiry incidents: 0 since deployment
  ROI: Paid for itself in 2 weeks
```

```yaml
# cert-manager ClusterIssuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: sre@company.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx

# Certificate auto-creation via Ingress annotation
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Automation caused an outage" | Add guardrails: dry-run mode, blast radius limits, mandatory review for destructive operations. |
| "Too many things to automate, where to start?" | Use the frequency × duration matrix. Start with highest-frequency, longest-duration toil. |
| "Automation scripts are unreliable" | Treat automation as production code: tests, code review, CI/CD, monitoring. |
| "Team resists automation (job security fears)" | Reframe: automation frees engineers for interesting work. Toil is what they should fear, not automation. |

---

## Summary Table

| Strategy | Level | Effort | Impact |
|----------|-------|--------|--------|
| **Runbooks** | L2 (documented) | Low | Reduces errors, enables any team member |
| **Scripts** | L3 (human-initiated) | Low-Med | Eliminates manual steps |
| **CI/CD** | L3-4 (pipeline) | Medium | Deployment toil → near zero |
| **IaC** | L3-4 (declarative) | Medium | Infra provisioning → code review |
| **Event-driven** | L4 (system-initiated) | Med-High | Reactive automation |
| **Self-healing** | L5 (autonomous) | High | Human-free remediation |

---

## Quick Revision Questions

1. **What are the 5 levels of the automation hierarchy?**
   > L1: Tribal knowledge, L2: Documented/runbook, L3: Human-initiated automated execution, L4: System-initiated with human approval, L5: Fully autonomous.

2. **When should you NOT automate something?**
   > When it's a truly one-time task, when the cost of automation exceeds the toil savings, or when the task genuinely requires human judgment every time.

3. **What are the key safety principles for automation?**
   > Dry-run first, blast radius limiting, human gates for risky operations, rollback capability, audit trails, rate limiting, and dead-man's-switch alerting.

4. **How does IaC reduce toil?**
   > Infrastructure as Code makes provisioning repeatable, version-controlled, and reviewable. It replaces manual clicking in cloud consoles with declarative code that can be automated via CI/CD.

5. **What's the difference between Level 3 and Level 4 automation?**
   > L3: Human initiates the action (clicks "deploy"). L4: System detects the need and proposes the action, human approves. L4 is more proactive — the system identifies the problem.

---

[← Previous: Measuring Toil](02-measuring-toil.md) | [Back to README](../README.md) | [Next: Reducing Operational Burden →](04-reducing-operational-burden.md)
