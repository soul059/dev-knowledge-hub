# Chapter 9.2: CircleCI

[← Previous: Azure Pipelines](01-azure-pipelines.md) | [Back to README](../README.md) | [Next: ArgoCD →](03-argocd.md)

---

## Overview

**CircleCI** is a cloud-native CI/CD platform known for speed, Docker-first workflows, and powerful caching. It uses `.circleci/config.yml` for pipeline configuration and offers both cloud-hosted and self-hosted runners.

---

## Architecture

```
  CIRCLECI ARCHITECTURE
  ═════════════════════

  ┌──────────────┐    ┌──────────────────────────────┐
  │   GitHub /   │    │  CircleCI Cloud              │
  │   Bitbucket  │───▶│                              │
  │   GitLab     │    │  ┌────────────┐              │
  └──────────────┘    │  │  Pipeline  │              │
                      │  └──────┬─────┘              │
                      │         │                    │
                      │  ┌──────▼─────┐              │
                      │  │ Workflows  │              │
                      │  └──────┬─────┘              │
                      │         │                    │
                      │  ┌──────▼─────┐              │
                      │  │   Jobs     │              │
                      │  └──────┬─────┘              │
                      │         │                    │
                      │  ┌──────▼─────┐              │
                      │  │ Executors  │              │
                      │  │ docker/    │              │
                      │  │ machine/   │              │
                      │  │ macos      │              │
                      │  └────────────┘              │
                      └──────────────────────────────┘
```

---

## Configuration Basics

```yaml
# .circleci/config.yml
version: 2.1

# Reusable executor definitions
executors:
  node-executor:
    docker:
      - image: cimg/node:20.11
    working_directory: ~/project
    resource_class: medium

# Reusable commands
commands:
  install-deps:
    description: "Install and cache dependencies"
    steps:
      - restore_cache:
          keys:
            - deps-v1-{{ checksum "package-lock.json" }}
            - deps-v1-
      - run: npm ci
      - save_cache:
          key: deps-v1-{{ checksum "package-lock.json" }}
          paths:
            - ~/.npm

# Job definitions
jobs:
  build:
    executor: node-executor
    steps:
      - checkout
      - install-deps
      - run: npm run build
      - persist_to_workspace:
          root: .
          paths:
            - dist/

  test:
    executor: node-executor
    parallelism: 4                    # Split tests across 4 containers
    steps:
      - checkout
      - install-deps
      - run:
          name: Run tests
          command: |
            TESTS=$(circleci tests glob "tests/**/*.test.js" | circleci tests split)
            npx jest $TESTS

  deploy:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - run: ./deploy.sh

# Workflow orchestration
workflows:
  build-test-deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - deploy:
          requires:
            - test
          filters:
            branches:
              only: main
```

---

## Key Concepts

### Executors

```
  ┌───────────────┬───────────────────────────────────┐
  │ Executor      │ Use Case                          │
  ├───────────────┼───────────────────────────────────┤
  │ docker        │ Default. Fast. cimg/* images      │
  │               │ Can add service containers        │
  ├───────────────┼───────────────────────────────────┤
  │ machine       │ Full VM, Docker-in-Docker         │
  │               │ Linux/Windows/Arm                 │
  ├───────────────┼───────────────────────────────────┤
  │ macos         │ macOS VMs for iOS/Swift builds    │
  └───────────────┴───────────────────────────────────┘
```

```yaml
# Docker executor with services
jobs:
  integration-test:
    docker:
      - image: cimg/node:20.11            # Primary
      - image: cimg/postgres:16.0         # Service
        environment:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
      - image: cimg/redis:7.2             # Another service
    steps:
      - checkout
      - run: npm test

# Machine executor
  build-docker:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - run: docker build -t myapp .
      - run: docker push myapp
```

### Orbs (Reusable Packages)

```yaml
version: 2.1

# Import orbs
orbs:
  node: circleci/node@5.2
  aws-cli: circleci/aws-cli@4.1
  slack: circleci/slack@4.13

workflows:
  main:
    jobs:
      # Use orb-provided jobs directly
      - node/test:
          pkg-manager: npm
      - aws-cli/setup:
          requires:
            - node/test
      - slack/on-hold:
          requires:
            - aws-cli/setup
```

```
  ORBS ECOSYSTEM
  ══════════════

  Popular orbs:
  ├── circleci/node       → Node.js build/test
  ├── circleci/python     → Python build/test
  ├── circleci/docker     → Docker build/push
  ├── circleci/aws-cli    → AWS authentication
  ├── circleci/aws-ecr    → Push to ECR
  ├── circleci/aws-ecs    → Deploy to ECS
  ├── circleci/kubernetes → K8s deployments
  ├── circleci/slack      → Slack notifications
  └── circleci/terraform  → Terraform operations

  Create your own:
  circleci orb create myorg/myorb
  circleci orb publish myorg/myorb@1.0.0
```

---

## Caching & Workspaces

```
  CACHE vs WORKSPACE vs ARTIFACTS
  ════════════════════════════════

  Cache:
  ├── Persists across pipelines
  ├── Key-based (checksum of lock file)
  └── Used for: dependencies (node_modules)

  Workspace:
  ├── Persists within a single workflow
  ├── Pass data between jobs
  └── Used for: build outputs

  Artifacts:
  ├── Persists after pipeline completes
  ├── Downloadable from UI
  └── Used for: reports, binaries
```

```yaml
jobs:
  build:
    steps:
      - checkout
      # Cache: dependencies
      - restore_cache:
          keys:
            - v2-deps-{{ checksum "package-lock.json" }}
      - run: npm ci
      - save_cache:
          key: v2-deps-{{ checksum "package-lock.json" }}
          paths:
            - node_modules

      # Workspace: pass build output to next job
      - run: npm run build
      - persist_to_workspace:
          root: .
          paths: [dist/]

  test:
    steps:
      - checkout
      - restore_cache:
          keys:
            - v2-deps-{{ checksum "package-lock.json" }}
      - attach_workspace:
          at: .
      - run: npm test

      # Artifact: save test report
      - store_test_results:
          path: test-results/
      - store_artifacts:
          path: coverage/
          destination: coverage-report
```

---

## Test Splitting & Parallelism

```yaml
# Automatic test splitting across containers
test:
  parallelism: 4
  steps:
    - checkout
    - run:
        name: Run tests
        command: |
          # Split by timing data (fastest method)
          TESTS=$(circleci tests glob "spec/**/*_spec.rb" \
            | circleci tests split --split-by=timings)
          bundle exec rspec $TESTS

# CircleCI collects timing data from store_test_results
# and optimally distributes tests across containers
```

```
  PARALLELISM = 4
  ═══════════════

  Container 1: test_a.rb, test_d.rb  (3 min)
  Container 2: test_b.rb, test_e.rb  (3 min)
  Container 3: test_c.rb             (3 min)
  Container 4: test_f.rb, test_g.rb  (3 min)
                                 Total: 3 min
  vs. Sequential:                Total: 12 min
```

---

## Contexts & Secrets

```
  CONTEXTS
  ════════

  Organization Settings → Contexts

  ┌────────────────────────┐
  │ Context: aws-prod      │
  │ ├── AWS_ACCESS_KEY_ID  │
  │ ├── AWS_SECRET_KEY     │
  │ └── AWS_REGION         │
  │                        │
  │ Security:              │
  │ ├── Restricted to:     │
  │ │   team-ops group     │
  │ └── Expression:        │
  │     branch == main     │
  └────────────────────────┘
```

```yaml
workflows:
  deploy:
    jobs:
      - deploy-prod:
          context:
            - aws-prod             # Injects context variables
            - docker-hub
          filters:
            branches:
              only: main
```

---

## Dynamic Configuration

```yaml
# Use setup workflows to generate config dynamically
version: 2.1
setup: true

orbs:
  continuation: circleci/continuation@1.0

jobs:
  generate-config:
    docker:
      - image: cimg/base:current
    steps:
      - checkout
      - run:
          name: Generate pipeline config
          command: |
            # Detect changed services in monorepo
            python generate-config.py > generated-config.yml
      - continuation/continue:
          configuration_path: generated-config.yml
          parameters: '{"deploy": true}'

workflows:
  setup:
    jobs:
      - generate-config
```

---

## Real-World Scenario

> **Scenario**: A Ruby on Rails team has a test suite that takes 45 minutes to run sequentially. They need faster feedback.
>
> **Solution**: Use CircleCI's `parallelism: 8` with `circleci tests split --split-by=timings`. After the first run, CircleCI collects timing data via `store_test_results`. Subsequent runs distribute tests optimally across 8 containers, reducing test time to ~6 minutes. Use caching for `vendor/bundle` with a checksum key on `Gemfile.lock`. Total pipeline drops from 50 minutes to 10 minutes.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Config validation error | Invalid YAML/syntax | Run `circleci config validate` locally |
| Cache not restoring | Key mismatch | Use checksum + fallback keys |
| Orb not found | Not imported or wrong version | Check orb registry, pin version |
| SSH into debug | Need to debug failed job | Use "Rerun with SSH" in UI |
| Resource class error | Plan doesn't support size | Check plan limits or downgrade class |
| Docker layer caching miss | DLC not enabled | Enable DLC feature (paid) |

---

## Summary Table

| Feature | CircleCI |
|---|---|
| Config file | `.circleci/config.yml` |
| Reusable packages | Orbs (versioned, published) |
| Executors | Docker, Machine (VM), macOS |
| Test splitting | Built-in `circleci tests split` |
| Caching | Key-based with checksum templates |
| Secrets | Contexts (org-level, restricted) |
| Dynamic config | Setup workflows + continuation |
| Best for | Fast feedback, test-heavy projects |

---

## Quick Revision Questions

1. **What are CircleCI orbs?** *Reusable, versioned packages of config (commands, executors, jobs) that can be shared across projects. Similar to GitHub Actions or Jenkins shared libraries.*
2. **How does test splitting work?** *Set `parallelism: N` on a job, then use `circleci tests split` to distribute test files across N containers. `--split-by=timings` uses historical data for optimal distribution.*
3. **What's the difference between cache, workspace, and artifacts?** *Cache persists across pipelines (for dependencies), workspace passes data between jobs in one workflow, artifacts persist after completion (for reports/downloads).*
4. **What executor types does CircleCI support?** *Docker (fast, layered), Machine (full VM with Docker-in-Docker), and macOS (for iOS/Swift builds).*
5. **What are contexts?** *Organization-level groups of environment variables that can be restricted by team/group and branch — injected into jobs via `context:` in workflow config.*
6. **What is dynamic configuration?** *Using `setup: true` workflows to generate pipeline config at runtime — useful for monorepos that need different pipelines based on changed files.*

---

[← Previous: Azure Pipelines](01-azure-pipelines.md) | [Back to README](../README.md) | [Next: ArgoCD →](03-argocd.md)
