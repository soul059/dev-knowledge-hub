# Chapter 3.1: Build Tools Overview

[← Previous: Parallelization and Caching](../02-Pipeline-Design/06-parallelization-caching.md) | [Back to README](../README.md) | [Next: Compilation and Packaging →](02-compilation-packaging.md)

---

## Overview

A **build tool** automates the process of transforming source code into a deployable artifact. Different languages and ecosystems have their own build tools, but they all share common responsibilities: dependency resolution, compilation, testing, and packaging.

---

## What Build Tools Do

```
┌──────────────────────────────────────────────────────────────────┐
│                  BUILD TOOL RESPONSIBILITIES                      │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ Resolve  │─▶│ Compile  │─▶│   Test   │─▶│ Package  │        │
│  │   Deps   │  │  Source  │  │  (run)   │  │ Artifact │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
│    Download      Convert       Execute       Create JAR,        │
│    libraries     to binary     test suite    Docker image,      │
│                                              bundle, etc.        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Build Tools by Language

| Language | Build Tool | Config File | Command |
|---|---|---|---|
| **Java** | Maven | `pom.xml` | `mvn clean package` |
| **Java** | Gradle | `build.gradle` | `gradle build` |
| **JavaScript** | npm / yarn / pnpm | `package.json` | `npm run build` |
| **Python** | pip / setuptools / poetry | `pyproject.toml` | `python -m build` |
| **Go** | go build (built-in) | `go.mod` | `go build ./...` |
| **.NET** | MSBuild / dotnet CLI | `.csproj` | `dotnet build` |
| **Rust** | Cargo | `Cargo.toml` | `cargo build --release` |
| **C/C++** | Make / CMake | `Makefile` / `CMakeLists.txt` | `make` / `cmake --build .` |
| **Ruby** | Bundler / Rake | `Gemfile` / `Rakefile` | `bundle exec rake build` |
| **Containers** | Docker | `Dockerfile` | `docker build .` |

---

## Build Tool Categories

```
┌──────────────────────────────────────────────────────────────────┐
│                BUILD TOOL CATEGORIES                              │
│                                                                   │
│  ┌────────────────────┐   ┌──────────────────────┐              │
│  │  LANGUAGE-SPECIFIC │   │  GENERAL-PURPOSE     │              │
│  │                    │   │                      │              │
│  │  Maven (Java)      │   │  Make                │              │
│  │  Gradle (Java/     │   │  Bazel (Google)      │              │
│  │     Kotlin)        │   │  Buck2 (Meta)        │              │
│  │  npm (JavaScript)  │   │  Pants (Twitter)     │              │
│  │  Cargo (Rust)      │   │  Nx (Monorepo)       │              │
│  │  pip (Python)      │   │  Turborepo (Vercel)  │              │
│  └────────────────────┘   └──────────────────────┘              │
│                                                                   │
│  ┌────────────────────┐   ┌──────────────────────┐              │
│  │  CONTAINER         │   │  TASK RUNNER         │              │
│  │                    │   │                      │              │
│  │  Docker            │   │  Makefile            │              │
│  │  Buildah           │   │  Just                │              │
│  │  Kaniko (no daemon)│   │  Task (taskfile.dev) │              │
│  │  BuildKit          │   │  npm scripts         │              │
│  └────────────────────┘   └──────────────────────┘              │
└──────────────────────────────────────────────────────────────────┘
```

---

## Common Build Tool Commands

### Java — Maven

```bash
mvn clean              # Clean build outputs
mvn compile            # Compile source code
mvn test               # Run unit tests
mvn package            # Create JAR/WAR
mvn verify             # Run integration tests
mvn install            # Install to local repo
mvn deploy             # Publish to remote repo
mvn clean package -DskipTests   # Build without tests
```

### JavaScript — npm

```bash
npm install            # Install dependencies
npm ci                 # Clean install (deterministic, CI-optimized)
npm run build          # Build project
npm test               # Run tests
npm run lint           # Lint code
npm pack               # Create tarball
npm publish            # Publish to npm registry
```

### Python — pip / poetry

```bash
# pip
pip install -r requirements.txt
python -m pytest
python -m build

# poetry
poetry install
poetry run pytest
poetry build
poetry publish
```

### Go

```bash
go mod download        # Download dependencies
go build ./...         # Build all packages
go test ./...          # Run all tests
go vet ./...           # Static analysis
```

### Docker

```bash
docker build -t myapp:1.0 .                          # Build image
docker build -t myapp:1.0 --target production .       # Multi-stage
docker build --cache-from myapp:latest -t myapp:1.0 . # With cache
```

---

## Makefile as Build Orchestrator

A `Makefile` provides a **language-agnostic** layer for common tasks:

```makefile
# Makefile — Universal build interface
.PHONY: build test lint deploy clean

build:
	npm ci
	npm run build

test:
	npm test -- --coverage

lint:
	npm run lint

security:
	npm audit --audit-level=high

docker:
	docker build -t myapp:$(shell git rev-parse --short HEAD) .

deploy-staging: docker
	docker push registry.example.com/myapp:$(shell git rev-parse --short HEAD)
	kubectl set image deployment/myapp myapp=registry.example.com/myapp:$(shell git rev-parse --short HEAD)

clean:
	rm -rf dist/ node_modules/

ci: lint test build docker    # Full CI pipeline in one command
```

```bash
# Usage in CI (any CI tool — same commands)
make ci               # Run full CI process
make deploy-staging   # Deploy to staging
```

---

## Troubleshooting Build Tools

| Issue | Cause | Solution |
|---|---|---|
| "Dependency not found" | Network issue or missing repo | Check proxy settings, verify repo URL |
| Non-deterministic builds | Unpinned versions | Use lockfiles, pin dependency versions |
| Build works locally, fails in CI | Different tool version | Pin tool version in CI (`.tool-versions`, Dockerfile) |
| Out of memory during build | Large project, small runner | Increase runner memory, optimize build |
| Slow builds | No caching, full rebuild | Enable build cache, use incremental builds |

---

## Summary Table

| Tool | Language | Strengths | Use Case |
|---|---|---|---|
| **Maven** | Java | Convention over config, ecosystem | Enterprise Java |
| **Gradle** | Java/Kotlin | Flexible, fast (incremental) | Android, large projects |
| **npm/yarn/pnpm** | JavaScript | Huge ecosystem, simple config | Web apps, Node.js |
| **pip/poetry** | Python | Simple, good dependency resolution | Data science, web |
| **Go build** | Go | Built-in, fast, static binaries | Microservices, CLIs |
| **Docker** | Any | Consistent environments | Containerized apps |
| **Make** | Any | Universal, simple | Cross-language orchestration |
| **Bazel** | Any | Hermetic, scalable | Large monorepos |

---

## Quick Revision Questions

1. **What are the four main responsibilities of a build tool?**  
   *Dependency resolution, compilation, testing, and packaging.*

2. **Why use `npm ci` instead of `npm install` in CI pipelines?**  
   *`npm ci` does a clean install from the lockfile, ensuring deterministic builds. It's faster in CI because it deletes node_modules first.*

3. **What advantage does a Makefile provide in CI/CD?**  
   *It provides a language-agnostic, universal interface — the same `make build` command works regardless of which CI tool you use.*

4. **Name two general-purpose build tools for monorepos.**  
   *Bazel (Google), Buck2 (Meta), Nx (Vercel), Turborepo, or Pants.*

5. **Why should you pin build tool versions in CI?**  
   *To prevent "works locally, fails in CI" issues caused by different tool versions producing different outputs.*

6. **What is the difference between Maven and Gradle?**  
   *Maven uses XML config with convention over configuration; Gradle uses Groovy/Kotlin DSL with more flexibility and supports incremental builds.*

---

[← Previous: Parallelization and Caching](../02-Pipeline-Design/06-parallelization-caching.md) | [Back to README](../README.md) | [Next: Compilation and Packaging →](02-compilation-packaging.md)
