# Chapter 4: Dockerignore

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

The **`.dockerignore`** file tells Docker which files and directories to exclude from the build context. It works similarly to `.gitignore` and is critical for build performance, security, and image size.

---

## 4.1 Why .dockerignore Matters

```
  WITHOUT .dockerignore:
  
  my-project/  (1.2 GB total)
  ├── .git/              200MB  ──┐
  ├── node_modules/      400MB  ──┤
  ├── coverage/          100MB  ──┤  ALL sent to
  ├── test-data/         300MB  ──┤  Docker daemon
  ├── .env               1KB   ──┤  (1.2 GB!)
  ├── src/               50MB  ──┤
  ├── package.json       2KB   ──┘
  └── Dockerfile         1KB
  
  WITH .dockerignore:
  
  my-project/  (50MB sent)
  ├── .git/              IGNORED
  ├── node_modules/      IGNORED
  ├── coverage/          IGNORED
  ├── test-data/         IGNORED
  ├── .env               IGNORED
  ├── src/               50MB  ──┐  Only needed files
  ├── package.json       2KB   ──┤  sent to daemon
  └── Dockerfile         1KB   ──┘  (50 MB)
```

---

## 4.2 .dockerignore Syntax

```
  # .dockerignore

  # Comments start with #
  
  # Exact match
  README.md
  
  # Directory (trailing slash optional)
  node_modules
  node_modules/
  
  # Wildcard * (matches any sequence except /)
  *.log
  *.md
  temp*
  
  # Double wildcard ** (matches any path)
  **/*.test.js
  **/temp/
  
  # Single character wildcard ?
  debug?.log         # debug1.log, debugA.log
  
  # Negation ! (re-include previously excluded)
  *.md
  !README.md         # Exclude all .md EXCEPT README.md
  
  # All files in any subdirectory
  **/.git
  **/__pycache__
```

---

## 4.3 Pattern Matching Rules

```
  Pattern          | Matches                      | Does NOT Match
  =================|==============================|==================
  *.log            | error.log, app.log           | logs/error.log
  **/*.log         | error.log, logs/error.log    | (matches all)
  temp?            | temp1, tempA                 | temp, temp12
  temp*            | temp, temp1, temporary       | atemp
  docs/            | docs/ directory              | docs.txt
  !important.log   | Re-includes important.log    |
```

### Order Matters for Negation

```
  # CORRECT: Exclude then re-include
  *.md
  !README.md
  # Result: All .md excluded EXCEPT README.md
  
  # WRONG: Re-include then exclude
  !README.md
  *.md
  # Result: ALL .md excluded (including README.md)
  # Because *.md overrides the earlier !README.md
  
  Rule: Last matching pattern wins
```

---

## 4.4 Comprehensive .dockerignore Examples

### Node.js Project

```
# Dependencies
node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Build output (if building in container)
dist
build

# Test & Coverage
coverage
.nyc_output
*.test.js
*.spec.js
__tests__

# Development
.env
.env.local
.env.*.local

# IDE
.vscode
.idea
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Git
.git
.gitignore

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Documentation
*.md
!README.md
LICENSE
docs/
```

### Python Project

```
# Virtual environments
venv
.venv
env
.env

# Python cache
__pycache__
*.py[cod]
*$py.class
*.pyo

# Distribution
dist
build
*.egg-info
*.egg

# Testing
.pytest_cache
.coverage
htmlcov
.tox

# IDE
.vscode
.idea
*.swp

# OS
.DS_Store
Thumbs.db

# Git
.git
.gitignore

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Docs
*.md
docs/
```

### Go Project

```
# Binary output
/bin
/build

# Test
*_test.go
coverage.out
coverage.html

# IDE
.vscode
.idea

# OS
.DS_Store
Thumbs.db

# Git
.git
.gitignore

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Docs
*.md
docs/

# Vendor (if using modules)
# vendor/  (uncomment if not vendoring)
```

### Java / Spring Boot Project

```
# Build output
target/
build/
*.jar
*.war
*.class

# IDE
.idea/
*.iml
.vscode/
.settings/
.project
.classpath

# Gradle
.gradle/
gradle/

# Maven
.mvn/

# Logs
*.log

# OS
.DS_Store
Thumbs.db

# Git
.git
.gitignore

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Docs
*.md
```

---

## 4.5 Security-Focused .dockerignore

```
# NEVER include these in build context:
.env
.env.*
*.pem
*.key
*.crt
*.p12
*.jks
credentials/
secrets/
*.secret
.aws/
.ssh/
id_rsa*
.npmrc
.pypirc

# CI/CD configs (may contain secrets)
.travis.yml
.circleci/
.github/
```

```
  Security Flow:
  
  +------------------+     .dockerignore     +------------------+
  |  Build Context   |  ------------------>  |  Filtered        |
  |                  |    removes secrets    |  Context          |
  |  .env (secrets!) |                       |                  |
  |  id_rsa (key!)   |                       |  src/            |
  |  src/            |                       |  package.json    |
  |  package.json    |                       |  (SAFE)          |
  +------------------+                       +------------------+
```

---

## 4.6 .dockerignore vs .gitignore

```
  +---------------------+-----------------------------+
  | .gitignore          | .dockerignore               |
  +---------------------+-----------------------------+
  | Excludes from Git   | Excludes from build context |
  | Tracked by Git      | Used by Docker daemon       |
  | ** matches paths    | ** matches paths            |
  | ! for negation      | ! for negation              |
  | Per-directory        | One file at context root    |
  | Inherits from parent| No inheritance              |
  +---------------------+-----------------------------+
  
  KEY DIFFERENCE:
  .gitignore: node_modules is excluded from Git
  .dockerignore: node_modules is excluded from Docker build
  
  You often need BOTH for the same patterns!
```

---

## 4.7 Debugging .dockerignore

```bash
# Check what's being sent to the daemon
docker build --no-cache . 2>&1 | head -1
# "Sending build context to Docker daemon  4.5MB"

# Compare with and without .dockerignore
mv .dockerignore .dockerignore.bak
docker build --no-cache . 2>&1 | head -1
# "Sending build context to Docker daemon  450MB"  <-- much larger!
mv .dockerignore.bak .dockerignore

# List files that WOULD be in context
# (manual check — Docker doesn't have a built-in tool)
rsync -n -av --exclude-from=.dockerignore . /dev/null
```

### Common Mistakes

```
  # MISTAKE 1: Wrong placement
  project/
  ├── docker/
  │   └── .dockerignore    <-- WRONG! Must be at context root
  ├── src/
  └── .dockerignore        <-- CORRECT

  # MISTAKE 2: Ignoring Dockerfile
  # If you exclude the Dockerfile itself, Docker still finds it
  # because -f flag path is resolved separately from context

  # MISTAKE 3: Negation without prior exclusion
  !important.txt           <-- Does nothing if not excluded first
  
  # CORRECT:
  *
  !important.txt
  !src/
```

---

## 4.8 Advanced: Whitelist Approach

```
# Ignore EVERYTHING
*

# Then whitelist only what you need
!src/
!package.json
!package-lock.json
!tsconfig.json
!.env.example

# This is the most secure approach:
# Only explicitly allowed files enter the build context
```

```
  Blacklist Approach:          Whitelist Approach:
  
  Include everything           Exclude everything
  Exclude known bad            Include known good
  
  Risk: Forgot to              Risk: Forgot to
  exclude something            include something
  (security risk)              (build fails — obvious!)
  
  RECOMMENDED: Whitelist for production images
```

---

## Summary Table

| Feature | Description | Example |
|---------|-------------|---------|
| **Purpose** | Exclude files from build context | Faster builds, security |
| **Location** | Root of build context | `.dockerignore` |
| **Wildcard** | Match any sequence | `*.log` |
| **Recursive** | Match in any subdirectory | `**/*.test.js` |
| **Negation** | Re-include excluded file | `!README.md` |
| **Whitelist** | Exclude all, include specific | `*` then `!src/` |
| **Security** | Never include secrets | `.env`, `*.key`, `*.pem` |
| **Order** | Last matching pattern wins | Put negations after exclusions |

---

## Quick Revision Questions

1. **What is the purpose of .dockerignore?**
   - It excludes files and directories from the build context, reducing context size, speeding up builds, and preventing sensitive files from being accessible during the build

2. **Where should .dockerignore be placed?**
   - At the root of the build context directory (the path specified in `docker build` command)

3. **What does the `!` prefix do in .dockerignore?**
   - It negates (re-includes) a previously excluded pattern. For example, after excluding `*.md`, `!README.md` re-includes README.md

4. **What is the whitelist approach and why is it recommended?**
   - Start with `*` to exclude everything, then use `!` to include only the files you need. It's more secure because forgotten files are excluded by default rather than included

5. **How is .dockerignore different from .gitignore?**
   - .gitignore controls what Git tracks, while .dockerignore controls what's sent to the Docker daemon during builds. They serve different purposes but often cover similar patterns

6. **Why should .env files always be in .dockerignore?**
   - .env files typically contain secrets (API keys, passwords, tokens) that should never be part of the build context where they could accidentally end up in image layers

---

[← Previous: Multi-Stage Builds](03-multi-stage-builds.md) | [Next: Build Args and ENV →](05-build-args-and-env.md)
