# Chapter 10.5: CI/CD Integration

[← Previous: Testing Strategies](04-testing-strategies.md) | [Next: Scaling Ansible →](06-scaling-ansible.md)

---

## Overview

Integrating Ansible into **CI/CD pipelines** ensures that every change to your automation code is validated, tested, and deployed reliably. This chapter covers pipeline design with GitHub Actions, GitLab CI, and Jenkins, plus patterns for automated deployment workflows.

---

## CI/CD Pipeline Architecture

```
  ══════════════════════════════════════════════════════════

  ┌──────┐   Push    ┌──────────────────────────────────┐
  │ Git  │─────────▶│         CI PIPELINE               │
  │ Repo │          │                                    │
  └──────┘          │  ┌────────┐  ┌────────┐  ┌─────┐ │
                    │  │ Lint   │─▶│Molecule│─▶│Build│ │
                    │  │        │  │ Test   │  │     │ │
                    │  └────────┘  └────────┘  └──┬──┘ │
                    └─────────────────────────────┼────┘
                                                  │
                    ┌─────────────────────────────┼────┐
                    │         CD PIPELINE          │    │
                    │                              ▼    │
                    │  ┌─────────┐  ┌──────┐  ┌──────┐│
                    │  │ Deploy  │─▶│Smoke │─▶│Deploy││
                    │  │ Staging │  │ Test │  │ Prod ││
                    │  └─────────┘  └──────┘  └──────┘│
                    └──────────────────────────────────┘
```

---

## GitHub Actions

```yaml
# .github/workflows/ansible-ci.yml
---
name: Ansible CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install ansible ansible-lint yamllint

      - name: Run yamllint
        run: yamllint .

      - name: Run ansible-lint
        run: ansible-lint playbooks/

      - name: Syntax check
        run: ansible-playbook playbooks/site.yml --syntax-check

  molecule:
    name: Molecule Tests
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        role:
          - common
          - webserver
          - database
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install ansible molecule molecule-docker docker

      - name: Run Molecule
        run: molecule test
        working-directory: roles/${{ matrix.role }}

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: molecule
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Install Ansible
        run: pip install ansible

      - name: Install Galaxy requirements
        run: ansible-galaxy install -r requirements.yml

      - name: Deploy to staging
        env:
          ANSIBLE_VAULT_PASSWORD: ${{ secrets.VAULT_PASSWORD }}
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > /tmp/key
          chmod 600 /tmp/key
          echo "$ANSIBLE_VAULT_PASSWORD" > /tmp/vault_pass
          ansible-playbook \
            -i inventories/staging/hosts.yml \
            --private-key /tmp/key \
            --vault-password-file /tmp/vault_pass \
            playbooks/site.yml
```

---

## GitLab CI

```yaml
# .gitlab-ci.yml
---
stages:
  - lint
  - test
  - deploy-staging
  - deploy-production

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"
  ANSIBLE_FORCE_COLOR: "true"

cache:
  paths:
    - .cache/pip

lint:
  stage: lint
  image: python:3.11-slim
  script:
    - pip install ansible ansible-lint yamllint
    - yamllint .
    - ansible-lint playbooks/
    - ansible-playbook playbooks/site.yml --syntax-check

molecule:
  stage: test
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375
  before_script:
    - apk add --no-cache python3 py3-pip
    - pip install ansible molecule molecule-docker docker
  script:
    - cd roles/webserver && molecule test
  parallel:
    matrix:
      - ROLE:
          - common
          - webserver
          - database

deploy-staging:
  stage: deploy-staging
  image: python:3.11-slim
  only:
    - main
  environment:
    name: staging
  before_script:
    - pip install ansible
    - ansible-galaxy install -r requirements.yml
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
  script:
    - echo "$VAULT_PASSWORD" > /tmp/vault_pass
    - ansible-playbook
        -i inventories/staging/hosts.yml
        --vault-password-file /tmp/vault_pass
        playbooks/site.yml

deploy-production:
  stage: deploy-production
  image: python:3.11-slim
  only:
    - main
  when: manual                    # Manual approval gate
  environment:
    name: production
  before_script:
    - pip install ansible
    - ansible-galaxy install -r requirements.yml
  script:
    - echo "$VAULT_PASSWORD_PROD" > /tmp/vault_pass
    - ansible-playbook
        -i inventories/production/hosts.yml
        --vault-password-file /tmp/vault_pass
        playbooks/site.yml
```

---

## Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        ANSIBLE_FORCE_COLOR = 'true'
    }

    stages {
        stage('Lint') {
            steps {
                sh 'pip install ansible ansible-lint yamllint'
                sh 'yamllint .'
                sh 'ansible-lint playbooks/'
                sh 'ansible-playbook playbooks/site.yml --syntax-check'
            }
        }

        stage('Molecule Test') {
            steps {
                sh 'pip install molecule molecule-docker'
                dir('roles/webserver') {
                    sh 'molecule test'
                }
            }
        }

        stage('Deploy Staging') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ansible-ssh',
                                     keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'vault-pass',
                           variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > /tmp/vault_pass
                        ansible-playbook \
                          -i inventories/staging/hosts.yml \
                          --private-key $SSH_KEY \
                          --vault-password-file /tmp/vault_pass \
                          playbooks/site.yml
                    '''
                }
            }
        }

        stage('Approve Production') {
            when { branch 'main' }
            steps {
                input message: 'Deploy to production?'
            }
        }

        stage('Deploy Production') {
            when { branch 'main' }
            steps {
                withCredentials([...]) {
                    sh 'ansible-playbook -i inventories/production ...'
                }
            }
        }
    }

    post {
        failure {
            slackSend channel: '#deployments',
                      message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        success {
            slackSend channel: '#deployments',
                      message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

---

## Deployment Strategies

```
  DEPLOYMENT PATTERNS
  ══════════════════════════════════════════════════════════

  1. ROLLING UPDATE
  ─────────────────
  serial: 2
  Batch 1: [web01, web02]  ──▶  update ──▶ verify
  Batch 2: [web03, web04]  ──▶  update ──▶ verify

  2. BLUE-GREEN
  ─────────────
  Blue (current) ────────────── serving traffic
  Green (new)    ──▶ deploy ──▶ test ──▶ switch LB ──▶ serving
  Blue           ──▶ standby (rollback target)

  3. CANARY
  ─────────
  1 host: deploy ──▶ monitor 10min ──▶ OK? ──▶ deploy rest
                                       FAIL? ──▶ rollback
```

### Rolling Update Playbook

```yaml
- name: Rolling deployment
  hosts: webservers
  serial: "25%"              # 25% of hosts at a time
  max_fail_percentage: 10     # Stop if >10% fail
  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://lb.example.com/api/drain/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost

  roles:
    - webserver

  post_tasks:
    - name: Smoke test
      uri:
        url: "http://{{ inventory_hostname }}/health"
        status_code: 200

    - name: Add back to load balancer
      uri:
        url: "http://lb.example.com/api/enable/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost
```

---

## Secrets in CI/CD

```
  NEVER store secrets in code or CI config files!
  ══════════════════════════════════════════════════════════

  ┌───────────────────┐     ┌──────────────────────────┐
  │  CI/CD Platform   │     │  Secrets Storage         │
  │                   │     │                          │
  │  GitHub:          │────▶│  Settings > Secrets      │
  │  GitLab:          │────▶│  Settings > CI/CD > Vars │
  │  Jenkins:         │────▶│  Credentials Manager     │
  └───────────────────┘     └──────────────────────────┘

  Inject at runtime as environment variables
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| SSH key permissions in CI | Ensure `chmod 600` on key file |
| Docker-in-Docker fails | Use `services: [docker:dind]` or install Docker in image |
| Vault password not found in CI | Store as CI secret, write to temp file at runtime |
| Molecule tests flaky | Use deterministic Docker images; increase timeouts |
| Deployment fails on one host | Use `serial` and `max_fail_percentage` to limit blast radius |
| CI takes too long | Parallelise lint and molecule; cache pip packages |

---

## Summary Table

| CI/CD Platform | Config File | Key Feature |
|---------------|-------------|-------------|
| GitHub Actions | `.github/workflows/*.yml` | Matrix strategy, environments |
| GitLab CI | `.gitlab-ci.yml` | Built-in environments, manual gates |
| Jenkins | `Jenkinsfile` | withCredentials, input approval |
| All | — | Lint → Test → Stage → Approve → Prod |

---

## Quick Revision Questions

1. **What are the typical stages in an Ansible CI/CD pipeline?**
   > Lint (yamllint, ansible-lint, syntax-check) → Test (Molecule) → Deploy Staging → Approve → Deploy Production.

2. **How do you handle Ansible Vault passwords in CI/CD?**
   > Store the vault password as a CI/CD secret (e.g., GitHub Secret), write it to a temporary file at runtime, and pass it with `--vault-password-file`.

3. **What is a rolling update and how is it implemented in Ansible?**
   > A rolling update deploys to a subset of hosts at a time using `serial`. It can include pre/post tasks to drain and re-enable hosts in a load balancer.

4. **Why should you use `max_fail_percentage`?**
   > To stop the deployment if too many hosts fail, preventing a bad change from affecting the entire fleet.

5. **How can you parallelise Molecule tests in CI?**
   > Use matrix strategies (GitHub Actions) or parallel jobs (GitLab CI) to test multiple roles simultaneously.

---

[← Previous: Testing Strategies](04-testing-strategies.md) | [Next: Scaling Ansible →](06-scaling-ansible.md)
