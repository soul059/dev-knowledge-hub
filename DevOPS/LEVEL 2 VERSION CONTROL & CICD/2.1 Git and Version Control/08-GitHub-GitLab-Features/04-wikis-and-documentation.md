# Chapter 8.4: Wikis and Documentation

[← Previous: GitLab CI/CD Basics](03-gitlab-ci-cd-basics.md) | [Back to README](../README.md) | [Next: GitHub Pages →](05-github-pages.md)

---

## Overview

Good documentation is as important as good code. GitHub and GitLab provide **built-in wikis**, support for **README files**, and tools for generating **documentation sites** — all version-controlled alongside your code.

---

## README.md — The Front Door

```
┌──────────────────────────────────────────────────────────────┐
│  README.md — ESSENTIAL SECTIONS                             │
│                                                              │
│  1. Project Title & Badges                                  │
│  2. Description — what the project does                     │
│  3. Installation — how to set it up                         │
│  4. Usage — how to use it (with examples)                   │
│  5. Configuration — environment variables, settings         │
│  6. API Reference — if applicable                           │
│  7. Contributing — how to contribute                        │
│  8. License — legal terms                                   │
│                                                              │
│  A good README answers:                                     │
│  "What is this? How do I use it? How do I contribute?"     │
└──────────────────────────────────────────────────────────────┘
```

### README Template

```markdown
# Project Name

![CI](https://github.com/user/repo/actions/workflows/ci.yml/badge.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

Brief description of what the project does.

## Installation

 ```bash
 npm install my-project
 ```

## Usage

 ```javascript
 const myProject = require('my-project');
 myProject.doSomething();
 ```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Server port | `3000` |
| `DB_URL` | Database connection | Required |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT — see [LICENSE](LICENSE) for details.
```

---

## GitHub/GitLab Wikis

```
┌──────────────────────────────────────────────────────────────┐
│  WIKI STRUCTURE                                             │
│                                                              │
│  Repo: github.com/user/myproject                            │
│  Wiki: github.com/user/myproject/wiki                       │
│                                                              │
│  Pages:                                                     │
│  ├── Home                 (landing page)                    │
│  ├── Getting Started      (quickstart guide)                │
│  ├── Architecture         (system design)                   │
│  ├── API Reference        (endpoints documentation)         │
│  ├── Deployment Guide     (how to deploy)                   │
│  ├── FAQ                  (common questions)                │
│  └── Troubleshooting      (common issues)                   │
│                                                              │
│  KEY FEATURES:                                              │
│  • Written in Markdown (or other formats)                   │
│  • Version controlled (it's a separate Git repo!)           │
│  • Sidebar navigation with _Sidebar.md                      │
│  • Footer with _Footer.md                                   │
│  • Searchable                                               │
└──────────────────────────────────────────────────────────────┘
```

### Cloning and Editing Wiki Locally

```bash
# Wiki is a separate Git repo!
$ git clone https://github.com/user/myproject.wiki.git
$ cd myproject.wiki

# Edit pages as Markdown files
$ vim Getting-Started.md

# Commit and push
$ git add -A
$ git commit -m "docs: update getting started guide"
$ git push
```

### Wiki Sidebar (`_Sidebar.md`)

```markdown
## Navigation

- [Home](Home)
- **Getting Started**
  - [Installation](Installation)
  - [Quick Start](Quick-Start)
- **Guides**
  - [Architecture](Architecture)
  - [API Reference](API-Reference)
  - [Deployment](Deployment)
- **Help**
  - [FAQ](FAQ)
  - [Troubleshooting](Troubleshooting)
```

---

## Documentation in the Repository

```
┌──────────────────────────────────────────────────────────────┐
│  DOCS-AS-CODE APPROACH                                      │
│                                                              │
│  project/                                                   │
│  ├── docs/                    ← Documentation folder        │
│  │   ├── getting-started.md                                 │
│  │   ├── architecture.md                                    │
│  │   ├── api/                                               │
│  │   │   ├── endpoints.md                                   │
│  │   │   └── authentication.md                              │
│  │   ├── guides/                                            │
│  │   │   ├── deployment.md                                  │
│  │   │   └── contributing.md                                │
│  │   └── images/                                            │
│  │       └── architecture-diagram.png                       │
│  ├── README.md                                              │
│  ├── CONTRIBUTING.md                                        │
│  ├── CHANGELOG.md                                           │
│  ├── CODE_OF_CONDUCT.md                                     │
│  └── LICENSE                                                │
│                                                              │
│  Advantages:                                                │
│  • Reviewed via PRs (same as code)                          │
│  • Versioned with the code it documents                     │
│  • Searchable in the repository                             │
│  • Can be used to generate static sites                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Standard Documentation Files

| File | Purpose |
|------|---------|
| `README.md` | Project overview, quickstart |
| `CONTRIBUTING.md` | How to contribute (PR process, style guide) |
| `CHANGELOG.md` | Version history and changes |
| `CODE_OF_CONDUCT.md` | Community behavior expectations |
| `LICENSE` | Legal terms for using the code |
| `SECURITY.md` | How to report security vulnerabilities |
| `SUPPORT.md` | Where to get help |
| `.github/PULL_REQUEST_TEMPLATE.md` | Default PR description template |
| `.github/ISSUE_TEMPLATE/` | Issue templates directory |

---

## Documentation Site Generators

```
┌──────────────────────────────────────────────────────────────┐
│  POPULAR DOC SITE GENERATORS                                │
│                                                              │
│  Tool             Language    Best For                      │
│  ────             ────────    ────────                      │
│  MkDocs           Python      Clean, simple docs sites      │
│  Docusaurus       React       Feature-rich, versioned docs  │
│  VitePress        Vue         Fast, Vue-based docs          │
│  Sphinx           Python      Python API documentation      │
│  Jekyll           Ruby        GitHub Pages native support    │
│  GitBook          Web         Book-style documentation      │
│  Hugo             Go          Fast static site generation   │
│                                                              │
│  Write Markdown → Generate HTML → Deploy to GitHub Pages    │
└──────────────────────────────────────────────────────────────┘
```

### MkDocs Example

```yaml
# mkdocs.yml
site_name: My Project Docs
theme:
  name: material
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api/endpoints.md
  - Deployment: guides/deployment.md
```

```bash
# Install and serve locally
$ pip install mkdocs-material
$ mkdocs serve              # Local preview at localhost:8000
$ mkdocs build              # Generate static site in site/
$ mkdocs gh-deploy          # Deploy to GitHub Pages
```

---

## Real-World Scenarios

### Scenario 1: Documentation Review Process

```
1. Developer updates code AND docs in same PR
2. CODEOWNERS assigns docs/ changes to @tech-writer
3. Tech writer reviews documentation for clarity
4. PR merged → docs site auto-deploys via CI/CD
```

### Scenario 2: API Documentation

```
Option A: OpenAPI/Swagger (auto-generated from code)
Option B: Markdown in docs/api/ (manual, reviewed via PR)
Option C: JSDoc / Docstring → generated reference docs

Best practice: Combine auto-generated API reference with
hand-written guides and tutorials.
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Wiki changes not showing | Push to the wiki repo; changes need to be committed |
| Docs out of date | Add docs updates to PR checklist; enforce via review |
| Can't find documentation | Organize with clear navigation; use search-friendly names |
| Images not rendering | Use relative paths; check file case sensitivity |
| Wiki access restricted | Check repo visibility; wiki inherits repo permissions |

---

## Summary Table

| Platform | Wiki | Docs in Repo | Site Generator |
|----------|------|-------------|----------------|
| GitHub | Built-in Wiki tab | `docs/` folder + GitHub Pages | Jekyll, MkDocs, etc. |
| GitLab | Built-in Wiki | `docs/` folder + GitLab Pages | Any static generator |
| Both | Separate Git repo | PR-reviewed, versioned | Auto-deploy via CI |

---

## Quick Revision Questions

1. **What are the essential sections of a good README?**
   <details><summary>Answer</summary>Title, description, installation instructions, usage examples, configuration, API reference (if applicable), contributing guidelines, and license. A good README answers: "What is this? How do I use it? How do I contribute?"</details>

2. **How is a GitHub Wiki different from docs in the repository?**
   <details><summary>Answer</summary>Wiki is a separate Git repository with its own history, edited via the web UI or by cloning. Docs in the repo (`docs/` folder) are part of the main codebase, reviewed via PRs, and versioned with the code. Docs-in-repo is preferred for code-related documentation that should be reviewed.</details>

3. **What is the "docs-as-code" approach?**
   <details><summary>Answer</summary>Treating documentation like code: writing in Markdown, storing in the repository, reviewing via PRs, testing with linters, versioning alongside code, and deploying automatically via CI/CD. This ensures docs stay current and are reviewed by the team.</details>

4. **Name three documentation site generators.**
   <details><summary>Answer</summary>MkDocs (Python, Material theme), Docusaurus (React-based), VitePress (Vue-based), Sphinx (Python docs), Jekyll (Ruby, GitHub Pages native), Hugo (Go), or GitBook. They convert Markdown files into professional documentation websites.</details>

5. **What standard documentation files should every project have?**
   <details><summary>Answer</summary>At minimum: README.md (overview), LICENSE (legal terms), CONTRIBUTING.md (how to contribute). Additionally recommended: CHANGELOG.md (version history), CODE_OF_CONDUCT.md (community rules), SECURITY.md (vulnerability reporting).</details>

---

[← Previous: GitLab CI/CD Basics](03-gitlab-ci-cd-basics.md) | [Back to README](../README.md) | [Next: GitHub Pages →](05-github-pages.md)
