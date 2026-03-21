# Unit 3: Threat Detection — Topic 6: Detection as Code

## Overview

**Detection as Code (DaC)** applies software engineering practices to security detection content. By treating detection rules as code, security teams gain version control, peer review, automated testing, CI/CD pipelines, and collaboration workflows — dramatically improving detection quality, consistency, and velocity.

---

## 1. Detection as Code Principles

```
DaC PRINCIPLES:

  ┌──────────────────────────────────────────────────┐
  │          DETECTION AS CODE                        │
  │                                                    │
  │  VERSION CONTROL                                  │
  │  → All rules in Git repository                    │
  │  → Full change history                            │
  │  → Branch-based development                       │
  │  → Tag releases                                   │
  │                                                    │
  │  PEER REVIEW                                      │
  │  → Pull request workflow                          │
  │  → Code review for detection logic                │
  │  → Multiple eyes on each rule                     │
  │  → Quality gates                                  │
  │                                                    │
  │  AUTOMATED TESTING                                │
  │  → Unit tests for each rule                       │
  │  → Integration tests with sample data             │
  │  → CI/CD pipeline validation                      │
  │  → Attack simulation validation                   │
  │                                                    │
  │  CONTINUOUS DEPLOYMENT                            │
  │  → Automated deployment to SIEM                   │
  │  → Staging → Production workflow                  │
  │  → Rollback capability                            │
  │  → Monitoring post-deployment                     │
  └──────────────────────────────────────────────────┘

TRADITIONAL vs DaC:
  Aspect        | Traditional       | Detection as Code
  Storage       | SIEM only         | Git repository
  Changes       | GUI edits         | Code changes
  Review        | None/informal     | Pull requests
  Testing       | Manual            | Automated
  Deploy        | Copy/paste        | CI/CD pipeline
  History       | Limited/none      | Full git history
  Collaboration | Difficult         | Standard workflow
  Rollback      | Manual            | Git revert
```

---

## 2. Sigma: Universal Detection Format

```
SIGMA RULE FORMAT:

  Sigma = Universal detection rule format
  → Write once, convert to any SIEM
  → YAML-based syntax
  → Community-maintained rule set
  → 3000+ community rules

  SIGMA RULE STRUCTURE:
  title: Suspicious PowerShell Download Cradle
  id: 403c8b91-2e3c-43c0-98aa-c2f5e0f2c5b1
  status: stable
  description: |
    Detects PowerShell download cradle commands
    commonly used by attackers to download and
    execute payloads.
  author: SOC Team
  date: 2024/01/15
  modified: 2024/06/01
  references:
    - https://attack.mitre.org/techniques/T1059/001/
  tags:
    - attack.execution
    - attack.t1059.001
    - attack.command_and_control
    - attack.t1105
  logsource:
    category: process_creation
    product: windows
  detection:
    selection_process:
      Image|endswith: '\powershell.exe'
    selection_commands:
      CommandLine|contains:
        - 'Invoke-WebRequest'
        - 'IWR'
        - 'Invoke-RestMethod'
        - 'DownloadString'
        - 'DownloadFile'
        - 'Net.WebClient'
        - 'Start-BitsTransfer'
    condition: selection_process and selection_commands
  falsepositives:
    - Legitimate admin scripts
    - Software deployment tools
  level: high

SIGMA CONVERSION:
  # Install sigma CLI
  pip install sigma-cli
  
  # Convert to Splunk
  sigma convert -t splunk -p sysmon rule.yml
  
  # Convert to Elastic/Lucene
  sigma convert -t elasticsearch -p ecs_windows rule.yml
  
  # Convert to Microsoft Sentinel
  sigma convert -t microsoft365defender rule.yml
  
  # Convert to QRadar AQL
  sigma convert -t qradar rule.yml

SIGMA OUTPUT EXAMPLE (Splunk SPL):
  EventCode=1 Image="*\\powershell.exe"
  (CommandLine="*Invoke-WebRequest*" OR
   CommandLine="*IWR*" OR
   CommandLine="*DownloadString*" OR
   CommandLine="*Net.WebClient*")
```

---

## 3. Repository Structure

```
DETECTION REPO STRUCTURE:

detection-rules/
├── README.md
├── .github/
│   └── workflows/
│       ├── validate.yml      # CI validation
│       ├── test.yml           # Testing pipeline
│       └── deploy.yml         # Deployment pipeline
├── rules/
│   ├── windows/
│   │   ├── process_creation/
│   │   │   ├── win_susp_powershell.yml
│   │   │   ├── win_lolbin_execution.yml
│   │   │   └── win_office_shell_spawn.yml
│   │   ├── authentication/
│   │   │   ├── win_brute_force.yml
│   │   │   └── win_account_lockout.yml
│   │   └── persistence/
│   │       ├── win_registry_run_keys.yml
│   │       └── win_scheduled_task.yml
│   ├── linux/
│   │   ├── process_creation/
│   │   └── authentication/
│   ├── network/
│   │   ├── dns_tunneling.yml
│   │   └── beaconing.yml
│   └── cloud/
│       ├── aws/
│       ├── azure/
│       └── gcp/
├── tests/
│   ├── test_data/
│   │   ├── win_susp_powershell_tp.json
│   │   └── win_susp_powershell_fp.json
│   └── test_rules.py
├── config/
│   ├── splunk_config.yml
│   ├── elastic_config.yml
│   └── sentinel_config.yml
└── docs/
    ├── writing_guide.md
    ├── review_checklist.md
    └── deployment_guide.md

NAMING CONVENTION:
  {platform}_{category}_{description}.yml
  Examples:
  → win_proc_powershell_download.yml
  → lin_auth_brute_force_ssh.yml
  → net_dns_tunneling.yml
  → aws_iam_root_login.yml
```

---

## 4. CI/CD Pipeline

```
DETECTION CI/CD PIPELINE:

  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Develop  │───▶│  Review  │───▶│  Test    │
  │ (Branch) │    │  (PR)    │    │  (CI)    │
  └──────────┘    └──────────┘    └──────────┘
                                       │
  ┌──────────┐    ┌──────────┐    ┌────▼─────┐
  │ Monitor  │◀───│ Deploy   │◀───│ Approve  │
  │ (Prod)   │    │ (CD)     │    │ (Gate)   │
  └──────────┘    └──────────┘    └──────────┘

GITHUB ACTIONS EXAMPLE:

  # .github/workflows/validate.yml
  name: Validate Detection Rules
  on: [pull_request]
  jobs:
    validate:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Install sigma-cli
          run: pip install sigma-cli pySigma
        - name: Validate Sigma rules
          run: |
            sigma check rules/**/*.yml
        - name: Convert to Splunk
          run: |
            sigma convert -t splunk rules/**/*.yml
        - name: Run tests
          run: python -m pytest tests/

TESTING FRAMEWORK:
  # tests/test_rules.py
  import yaml
  import pytest
  
  def test_rule_has_required_fields(rule_file):
      with open(rule_file) as f:
          rule = yaml.safe_load(f)
      assert 'title' in rule
      assert 'description' in rule
      assert 'detection' in rule
      assert 'level' in rule
      assert 'tags' in rule
      # Verify ATT&CK mapping
      attack_tags = [t for t in rule['tags']
                     if t.startswith('attack.t')]
      assert len(attack_tags) > 0
  
  def test_rule_against_true_positive(rule, test_data):
      """Verify rule detects known-bad data"""
      result = evaluate_rule(rule, test_data)
      assert result.matched == True

DEPLOYMENT AUTOMATION:
  # Deploy to Splunk
  splunk-sdk create-saved-search \
    --name "$RULE_TITLE" \
    --search "$SPL_QUERY" \
    --alert-type "number of results" \
    --alert-threshold 0
  
  # Deploy to Elastic
  curl -X POST "elastic:9200/_security/api/rules" \
    -H "Content-Type: application/json" \
    -d @converted_rule.json
```

---

## 5. Best Practices

```
DaC BEST PRACTICES:

DEVELOPMENT:
  [ ] All rules in Git repository
  [ ] Sigma format for portability
  [ ] One rule per file
  [ ] Consistent naming convention
  [ ] ATT&CK tags on every rule
  [ ] False positive documentation
  [ ] Analyst guidance in description
  [ ] Version tags for releases

CODE REVIEW CHECKLIST:
  [ ] Rule detects intended technique
  [ ] Logic is correct (no AND/OR errors)
  [ ] ATT&CK mapping is accurate
  [ ] Severity is appropriate
  [ ] False positives are documented
  [ ] Test data provided
  [ ] No duplicate detection exists
  [ ] Description is clear for analysts
  [ ] Performance is acceptable
  [ ] Peer approved

CONTINUOUS IMPROVEMENT:
  → Track metrics per rule in production
  → Analyst feedback integration
  → Regular rule review cycles
  → Retire stale/ineffective rules
  → Update for new attack variants
  → Benchmark against community rules
  → Share improvements back to community

COMMUNITY RESOURCES:
  → Sigma Rules: github.com/SigmaHQ/sigma
  → Elastic Rules: github.com/elastic/detection-rules
  → Splunk Detections: github.com/splunk/security_content
  → Azure Sentinel: github.com/Azure/Azure-Sentinel
  → Atomic Red Team: github.com/redcanaryco/atomic-red-team
```

---

## Summary Table

| DaC Component | Tool/Practice | Benefit |
|--------------|--------------|---------|
| Format | Sigma (YAML) | SIEM-agnostic portability |
| Version Control | Git | Change history, collaboration |
| Review | Pull Requests | Quality assurance |
| Testing | CI pipeline | Automated validation |
| Deployment | CD pipeline | Consistent rollout |
| Community | SigmaHQ | 3000+ shared rules |

---

## Revision Questions

1. What is Detection as Code and how does it improve security operations?
2. How does Sigma enable SIEM-agnostic detection rules?
3. What should a detection rules repository structure look like?
4. How does CI/CD apply to detection rule management?
5. What should be included in a detection rule code review?

---

*Previous: [05-mitre-attack-for-detection.md](05-mitre-attack-for-detection.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
