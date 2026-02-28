# Chapter 7.2: Actions & Marketplace

[← Previous: Workflow Syntax](01-workflow-syntax.md) | [Back to README](../README.md) | [Next: Runners →](03-runners.md)

---

## Overview

**Actions** are reusable units of code that perform specific tasks in a workflow. The **GitHub Marketplace** hosts thousands of community and official actions. You can also create custom actions.

---

## Action Types

```
  ACTION TYPES
  ════════════

  1. JAVASCRIPT / TYPESCRIPT ACTION
     ├── Runs directly on runner (fast startup)
     ├── Node.js runtime
     └── Example: actions/checkout, actions/cache

  2. DOCKER CONTAINER ACTION
     ├── Runs in a Docker container
     ├── Any language / tools
     └── Slower startup (image pull)

  3. COMPOSITE ACTION
     ├── Combines multiple steps
     ├── Written in YAML (action.yml)
     ├── No separate runtime needed
     └── Great for bundling common workflows
```

---

## Essential Actions

```
  TOP ACTIONS BY CATEGORY
  ═══════════════════════

  SETUP:
  ├── actions/checkout@v4          → Clone repository
  ├── actions/setup-node@v4        → Install Node.js
  ├── actions/setup-python@v5      → Install Python
  ├── actions/setup-java@v4        → Install Java
  └── actions/setup-go@v5          → Install Go

  CACHING & ARTIFACTS:
  ├── actions/cache@v4             → Cache dependencies
  └── actions/upload-artifact@v4   → Upload build outputs
  └── actions/download-artifact@v4 → Download artifacts

  DOCKER:
  ├── docker/login-action@v3       → Registry authentication
  ├── docker/build-push-action@v5  → Build & push images
  └── docker/setup-buildx-action@v3→ Multi-platform builds

  DEPLOYMENT:
  ├── aws-actions/configure-aws-credentials@v4
  ├── azure/login@v2
  └── google-github-actions/auth@v2

  CODE QUALITY:
  ├── github/codeql-action@v3      → Security scanning
  ├── actions/dependency-review-action@v4
  └── codecov/codecov-action@v4    → Coverage reports
```

---

## Using Actions

```yaml
steps:
  # 1. Official action with version tag
  - uses: actions/checkout@v4

  # 2. Action with inputs
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
      registry-url: 'https://npm.pkg.github.com'

  # 3. Docker container action
  - uses: docker://alpine:3.19
    with:
      args: echo "Running in Alpine container"

  # 4. Action from another repository
  - uses: my-org/my-action@v1

  # 5. Local action (from same repo)
  - uses: ./.github/actions/my-custom-action

  # 6. Action at specific commit SHA (most secure)
  - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
```

---

## Creating a Composite Action

```yaml
# .github/actions/node-ci/action.yml
name: 'Node.js CI Setup'
description: 'Setup Node.js, install deps, and run common checks'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '20'
  working-directory:
    description: 'Working directory'
    required: false
    default: '.'

outputs:
  test-result:
    description: 'Test execution result'
    value: ${{ steps.test.outcome }}

runs:
  using: 'composite'
  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json

    - name: Install dependencies
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm ci

    - name: Lint
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm run lint

    - name: Test
      id: test
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm test -- --ci --coverage
```

### Using the Composite Action

```yaml
# .github/workflows/ci.yml
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: ./.github/actions/node-ci
        with:
          node-version: '20'
          working-directory: './backend'
```

---

## Creating a JavaScript Action

```
  JAVASCRIPT ACTION STRUCTURE
  ═══════════════════════════

  my-action/
  ├── action.yml          ← Action metadata
  ├── index.js            ← Entry point
  ├── package.json
  └── node_modules/       ← Bundled (committed or ncc compiled)
```

```yaml
# action.yml
name: 'Deployment Notifier'
description: 'Send deployment notifications to Slack'
inputs:
  webhook-url:
    description: 'Slack webhook URL'
    required: true
  status:
    description: 'Deployment status'
    required: true
outputs:
  message-id:
    description: 'Slack message timestamp'
runs:
  using: 'node20'
  main: 'dist/index.js'
```

```javascript
// index.js
const core = require('@actions/core');
const github = require('@actions/github');
const fetch = require('node-fetch');

async function run() {
  try {
    const webhookUrl = core.getInput('webhook-url', { required: true });
    const status = core.getInput('status');
    const { context } = github;

    const payload = {
      text: `Deployment ${status}: ${context.repo.repo}`,
      blocks: [{
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: [
            `*${context.repo.repo}* deployment *${status}*`,
            `Branch: \`${context.ref}\``,
            `Commit: \`${context.sha.substring(0, 7)}\``,
            `Actor: ${context.actor}`
          ].join('\n')
        }
      }]
    };

    const response = await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    core.setOutput('message-id', response.headers.get('x-slack-req-id'));
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

---

## Action Versioning & Security

```
  VERSION PINNING
  ═══════════════

  LEAST SECURE → MOST SECURE

  uses: actions/checkout@main      ← Branch (mutable, risky)
  uses: actions/checkout@v4        ← Major tag (auto-updates)
  uses: actions/checkout@v4.1.7    ← Exact tag (better)
  uses: actions/checkout@b4ffd...  ← SHA (immutable, most secure)

  RECOMMENDATION:
  • Use SHA pinning for production workflows
  • Use Dependabot to auto-update action versions
  • Review action source before using
```

### Dependabot for Actions

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      actions:
        patterns: ["*"]
```

---

## Real-World Scenario

> **Scenario**: A monorepo team wants a reusable action that runs lint, test, and build for any Node.js service with configurable parameters.
>
> **Solution**: Create a composite action in `.github/actions/node-ci/action.yml` with inputs for `node-version` and `working-directory`. Each service's workflow calls this action with its specific directory. Changes to the shared action are tested via PR checks before merging.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Action not found | Wrong path or version tag | Verify `uses:` path and tag exist |
| Composite action fails | Missing `shell:` in run steps | Composite actions require explicit `shell: bash` on every `run:` step |
| Input not received | Input name mismatch | Ensure `with:` keys match action's `inputs:` names |
| "Not allowed to access" | Action from private repo | Use PAT or make action repo public; check `actions:` permissions in org settings |
| Slow Docker action | Large image pull | Use slim base images; consider JavaScript action instead |
| Output not available | Missing `id:` on step | Add `id:` to the step and reference via `${{ steps.id.outputs.key }}` |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **JavaScript Actions** | Fast, run on Node.js, `@actions/core` toolkit |
| **Docker Actions** | Any language, isolated container, slower startup |
| **Composite Actions** | YAML-based, combine steps, no runtime needed |
| **Marketplace** | Thousands of community actions, searchable |
| **Versioning** | Tags, branches, or SHA; SHA is most secure |
| **Local Actions** | `uses: ./.github/actions/name` for repo-local actions |

---

## Quick Revision Questions

1. **What are the three types of GitHub Actions?** *JavaScript (Node.js), Docker container, and Composite (YAML combining multiple steps).*
2. **How do you use a local action from the same repository?** *`uses: ./.github/actions/action-name` — referencing the directory containing `action.yml`.*
3. **What is the most secure way to pin an action version?** *Using the full commit SHA: `uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11`.*
4. **What file defines an action's metadata?** *`action.yml` in the action's root directory — it declares inputs, outputs, and the runtime.*
5. **How do composite actions differ from reusable workflows?** *Composite actions are step-level reuse (called within a job's steps), while reusable workflows are job-level reuse (called as an entire job).*
6. **How do you keep action versions updated automatically?** *Configure Dependabot with `package-ecosystem: "github-actions"` in `.github/dependabot.yml`.*

---

[← Previous: Workflow Syntax](01-workflow-syntax.md) | [Back to README](../README.md) | [Next: Runners →](03-runners.md)
