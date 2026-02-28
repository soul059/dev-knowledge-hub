# Chapter 8.5: GitHub Pages

[← Previous: Wikis and Documentation](04-wikis-and-documentation.md) | [Back to README](../README.md) | [Next: Integration with Other Tools →](06-integration-with-other-tools.md)

---

## Overview

**GitHub Pages** is a free static site hosting service that serves websites directly from a GitHub repository. It's commonly used for project documentation, portfolios, blogs, and landing pages. GitLab offers a similar feature called **GitLab Pages**.

---

## How GitHub Pages Works

```
┌──────────────────────────────────────────────────────────────┐
│  GITHUB PAGES FLOW                                          │
│                                                              │
│  ┌──────────┐     ┌──────────┐     ┌──────────────────────┐ │
│  │ You push │────►│ GitHub   │────►│ Your site is live at │ │
│  │ code     │     │ builds   │     │ username.github.io/  │ │
│  │ to repo  │     │ the site │     │ repo-name            │ │
│  └──────────┘     └──────────┘     └──────────────────────┘ │
│                                                              │
│  Sources:                                                   │
│  1. From a branch (main or gh-pages)                        │
│  2. From /docs folder on a branch                           │
│  3. From GitHub Actions workflow                            │
│                                                              │
│  URL patterns:                                              │
│  • User site:    username.github.io                         │
│  • Project site: username.github.io/repo-name              │
│  • Custom:       www.yourdomain.com                         │
└──────────────────────────────────────────────────────────────┘
```

---

## Setting Up GitHub Pages

### Method 1: Deploy from Branch

```
┌──────────────────────────────────────────────────────────────┐
│  Settings → Pages → Source                                  │
│                                                              │
│  Source: Deploy from a branch                               │
│  Branch: main  /  Folder: / (root) or /docs                │
│                                                              │
│  [Save]                                                     │
│                                                              │
│  Your site is live at:                                      │
│  https://username.github.io/repo-name/                      │
└──────────────────────────────────────────────────────────────┘
```

### Method 2: Deploy with GitHub Actions

```yaml
# .github/workflows/deploy-pages.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

---

## Simple HTML Site

```html
<!-- index.html at repo root or /docs -->
<!DOCTYPE html>
<html>
<head>
    <title>My Project</title>
</head>
<body>
    <h1>Welcome to My Project</h1>
    <p>Documentation and resources.</p>
</body>
</html>
```

---

## Jekyll (GitHub Pages Default)

```
┌──────────────────────────────────────────────────────────────┐
│  JEKYLL — GitHub Pages' built-in static site generator      │
│                                                              │
│  repo/                                                      │
│  ├── _config.yml          ← Jekyll configuration            │
│  ├── index.md             ← Home page                       │
│  ├── about.md             ← About page                      │
│  ├── _posts/              ← Blog posts                      │
│  │   └── 2025-01-15-first-post.md                           │
│  ├── _layouts/            ← Page templates                  │
│  │   └── default.html                                       │
│  └── assets/              ← CSS, images                     │
│      └── css/style.css                                      │
│                                                              │
│  GitHub automatically builds Jekyll sites!                  │
│  No GitHub Actions needed for basic Jekyll sites.           │
└──────────────────────────────────────────────────────────────┘
```

### Minimal `_config.yml`

```yaml
title: My Project Documentation
description: Comprehensive docs for My Project
theme: minima
baseurl: /repo-name
url: https://username.github.io

# Navigation
header_pages:
  - about.md
  - docs.md
```

### Jekyll Markdown Page

```markdown
---
layout: default
title: Getting Started
permalink: /getting-started/
---

# Getting Started

Follow these steps to set up the project...
```

---

## Using Other Static Site Generators

### MkDocs + GitHub Actions

```yaml
# .github/workflows/docs.yml
name: Deploy Docs
on:
  push:
    branches: [main]
    paths: ['docs/**', 'mkdocs.yml']

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

### Docusaurus

```bash
$ npx create-docusaurus@latest my-docs classic
$ cd my-docs
$ npm run build
# Deploy dist/ to GitHub Pages via Actions
```

---

## Custom Domains

```
┌──────────────────────────────────────────────────────────────┐
│  CUSTOM DOMAIN SETUP                                        │
│                                                              │
│  1. Add CNAME file to repo root:                            │
│     docs.mycompany.com                                      │
│                                                              │
│  2. Configure DNS:                                          │
│     ┌────────────────────────────────────────────┐          │
│     │ Type   Name      Value                     │          │
│     │ CNAME  docs      username.github.io        │          │
│     │                                            │          │
│     │ For apex domain (mycompany.com):           │          │
│     │ A      @         185.199.108.153           │          │
│     │ A      @         185.199.109.153           │          │
│     │ A      @         185.199.110.153           │          │
│     │ A      @         185.199.111.153           │          │
│     └────────────────────────────────────────────┘          │
│                                                              │
│  3. Settings → Pages → Custom domain → docs.mycompany.com  │
│  4. Enable "Enforce HTTPS" ✅                                │
└──────────────────────────────────────────────────────────────┘
```

---

## GitLab Pages

```yaml
# .gitlab-ci.yml
pages:
  stage: deploy
  image: node:20
  script:
    - npm ci
    - npm run build
    - mv dist/ public/       # GitLab Pages serves from public/
  artifacts:
    paths:
      - public
  only:
    - main

# Site available at: username.gitlab.io/project-name
```

```
┌──────────────────────────────────────────────────────────────┐
│  GITHUB PAGES vs GITLAB PAGES                               │
│                                                              │
│  Feature          GitHub Pages      GitLab Pages            │
│  ───────          ────────────      ────────────            │
│  Config           Settings UI       .gitlab-ci.yml          │
│  Built-in SSG     Jekyll            None (use any)          │
│  Deploy folder    / or /docs        public/                 │
│  Custom domain    Yes + HTTPS       Yes + HTTPS             │
│  Private pages    Enterprise only   Yes (access control)    │
│  Build via CI     Actions           CI/CD pipeline          │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Project Documentation Site

```
1. Write docs in docs/ folder using Markdown
2. Use MkDocs with Material theme
3. GitHub Actions builds on push to main
4. Deploys to username.github.io/project
5. CODEOWNERS routes docs/ changes to tech writers
```

### Scenario 2: Portfolio Website

```
1. Create username.github.io repository
2. Add index.html with portfolio content
3. Enable GitHub Pages from main branch
4. Site live at https://username.github.io
5. Add custom domain for professional URL
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Site shows 404 | Check source branch/folder in Settings → Pages |
| CSS/images not loading | Check `baseurl` in config matches repo name |
| Custom domain not working | Verify DNS records; allow propagation time (24-48h) |
| Build fails on push | Check Actions tab for error logs |
| Jekyll build errors | Test locally: `bundle exec jekyll serve` |

---

## Summary Table

| Feature | Details |
|---------|---------|
| Service | Free static site hosting |
| Sources | Branch, /docs folder, or GitHub Actions |
| URL | `username.github.io/repo-name` |
| Built-in SSG | Jekyll (no config needed) |
| Custom domain | CNAME file + DNS configuration |
| HTTPS | Free, auto-configured |
| Limits | 1 GB repo, 100 GB bandwidth/month |
| Alternatives | GitLab Pages, Netlify, Vercel, Cloudflare Pages |

---

## Quick Revision Questions

1. **What is GitHub Pages?**
   <details><summary>Answer</summary>GitHub Pages is a free static site hosting service that serves websites directly from a GitHub repository. It supports HTML, CSS, JavaScript, and static site generators like Jekyll. Sites are available at `username.github.io/repo-name`.</details>

2. **What are the three ways to configure the source for GitHub Pages?**
   <details><summary>Answer</summary>1) Deploy from a branch (root `/` of the branch). 2) Deploy from the `/docs` folder on a branch. 3) Deploy using a GitHub Actions workflow. The Actions method gives the most flexibility for custom build processes.</details>

3. **What is Jekyll and why is it special for GitHub Pages?**
   <details><summary>Answer</summary>Jekyll is a static site generator written in Ruby. It's special because GitHub Pages has built-in Jekyll support — you can push Markdown and Jekyll config, and GitHub automatically builds and deploys the site without needing a CI/CD workflow.</details>

4. **How do you set up a custom domain for GitHub Pages?**
   <details><summary>Answer</summary>1) Add a CNAME file to the repo root with your domain name. 2) Configure DNS: add a CNAME record pointing to `username.github.io` (for subdomains) or A records pointing to GitHub's IPs (for apex domains). 3) Set the custom domain in Settings → Pages. 4) Enable "Enforce HTTPS".</details>

5. **What is the main difference between GitHub Pages and GitLab Pages?**
   <details><summary>Answer</summary>GitHub Pages has built-in Jekyll support and can be configured via the Settings UI. GitLab Pages requires a `.gitlab-ci.yml` pipeline that outputs to a `public/` directory — no built-in SSG, but you can use any generator. GitLab Pages also supports access control for private pages.</details>

---

[← Previous: Wikis and Documentation](04-wikis-and-documentation.md) | [Back to README](../README.md) | [Next: Integration with Other Tools →](06-integration-with-other-tools.md)
