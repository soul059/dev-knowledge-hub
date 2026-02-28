# Chapter 3.2: Compilation and Packaging

[← Previous: Build Tools Overview](01-build-tools-overview.md) | [Back to README](../README.md) | [Next: Dependency Management →](03-dependency-management.md)

---

## Overview

**Compilation** transforms human-readable source code into machine-executable form. **Packaging** bundles the compiled output with its dependencies and metadata into a deployable artifact. In CI/CD pipelines, these steps must be automated, reproducible, and fast.

---

## Compilation Flow

```
┌──────────────────────────────────────────────────────────────────┐
│              SOURCE → COMPILATION → ARTIFACT                      │
│                                                                   │
│  Compiled Languages (Java, Go, C++, Rust):                       │
│  ┌────────┐    ┌──────────┐    ┌──────────┐                     │
│  │ Source │───▶│ Compiler │───▶│ Binary   │                     │
│  │ .java  │    │ javac    │    │ .class   │                     │
│  │ .go    │    │ go build │    │ binary   │                     │
│  │ .rs    │    │ rustc    │    │ .exe     │                     │
│  └────────┘    └──────────┘    └──────────┘                     │
│                                                                   │
│  Interpreted/Transpiled (JS, Python, Ruby):                      │
│  ┌────────┐    ┌──────────┐    ┌──────────┐                     │
│  │ Source │───▶│Transpiler│───▶│ Bundle   │                     │
│  │ .ts    │    │ tsc      │    │ .js      │                     │
│  │ .jsx   │    │ webpack  │    │ bundle   │                     │
│  │ .scss  │    │ babel    │    │ .css     │                     │
│  └────────┘    └──────────┘    └──────────┘                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Packaging by Ecosystem

### Java (JAR / WAR)
```bash
# Maven: compile and package into JAR
mvn clean package
# Output: target/myapp-1.0.0.jar

# Run the JAR
java -jar target/myapp-1.0.0.jar

# Fat JAR (includes all dependencies)
mvn clean package -Pfat-jar
# Includes all libs inside the JAR — single file deployment
```

### JavaScript (Bundle / Docker)
```bash
# Build (transpile + bundle)
npm run build
# Output: dist/
#   ├── index.html
#   ├── main.a1b2c3.js      (hashed for cache busting)
#   ├── styles.d4e5f6.css
#   └── assets/

# Package as npm module
npm pack
# Output: myapp-1.0.0.tgz
```

### Python (Wheel / sdist)
```bash
# Build wheel and source distribution
python -m build
# Output: dist/
#   ├── myapp-1.0.0-py3-none-any.whl   (wheel)
#   └── myapp-1.0.0.tar.gz              (source)

# Install from wheel
pip install dist/myapp-1.0.0-py3-none-any.whl
```

### Go (Static Binary)
```bash
# Build static binary (no external dependencies)
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o myapp ./cmd/server
# Output: myapp (single binary, ~10-20MB)

# Cross-compilation
GOOS=windows GOARCH=amd64 go build -o myapp.exe ./cmd/server
GOOS=darwin  GOARCH=arm64 go build -o myapp-mac ./cmd/server
```

### Docker (Container Image)
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

```bash
docker build -t myapp:1.0.0 .
docker push registry.example.com/myapp:1.0.0
```

---

## Packaging Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│           PACKAGING BEST PRACTICES                            │
│                                                               │
│  1. REPRODUCIBLE                                              │
│     Same source → same artifact every time                   │
│     └── Pin versions, use lockfiles, set timestamps          │
│                                                               │
│  2. MINIMAL                                                   │
│     Include only what's needed to run                        │
│     └── No dev tools, test files, or docs in artifact        │
│                                                               │
│  3. VERSIONED                                                 │
│     Every artifact has a unique identifier                   │
│     └── SemVer + Git SHA                                     │
│                                                               │
│  4. IMMUTABLE                                                 │
│     Never overwrite a published artifact                     │
│     └── v1.0.0 is final; make v1.0.1 for fixes              │
│                                                               │
│  5. SELF-CONTAINED                                            │
│     Include all runtime dependencies                         │
│     └── Fat JARs, Docker images, static binaries             │
└──────────────────────────────────────────────────────────────┘
```

---

## CI/CD Pipeline: Build & Package Stage

```yaml
# GitHub Actions — Build and Package
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Compile TypeScript
        run: npm run build
      
      - name: Build Docker image
        run: |
          docker build \
            -t myapp:${{ github.sha }} \
            --label "git.sha=${{ github.sha }}" \
            --label "git.ref=${{ github.ref }}" .
      
      - name: Push to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker push registry.example.com/myapp:${{ github.sha }}
```

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Compilation error in CI only | Different compiler version | Pin compiler version in CI config |
| Large artifact size | Dev dependencies included | Use `--production` flag, multi-stage Docker |
| Non-reproducible builds | Floating dependency versions | Use lockfiles, pin versions |
| Missing runtime dependency | Not included in package | Check `dependencies` vs `devDependencies` |
| Slow packaging | Large context or no cache | Use `.dockerignore`, enable layer caching |

---

## Summary Table

| Concept | Description |
|---|---|
| **Compilation** | Converting source code to executable form |
| **Packaging** | Bundling compiled output into deployable artifact |
| **Artifact Types** | JAR, Docker image, wheel, binary, bundle |
| **Reproducibility** | Same input always produces same output |
| **Immutability** | Published artifacts are never overwritten |
| **Minimalism** | Only include what's needed at runtime |

---

## Quick Revision Questions

1. **What is the difference between compilation and packaging?**  
   *Compilation converts source code to executable form; packaging bundles the compiled output with dependencies into a deployable artifact.*

2. **Why should artifacts be immutable?**  
   *To ensure traceability — you can always reproduce the exact version running in production. Changes require a new version.*

3. **What is a "fat JAR" and why is it useful?**  
   *A JAR that includes all dependencies inside it, making deployment simple — just one file to deploy, no external dependency management.*

4. **Why use multi-stage Docker builds?**  
   *To keep final images small — build tools and dev dependencies stay in the build stage and don't end up in the production image.*

5. **What does `npm ci` do differently from `npm install`?**  
   *`npm ci` removes node_modules, installs exactly from the lockfile (deterministic), and is faster in CI environments.*

6. **How can you achieve reproducible builds?**  
   *Pin all dependency versions, use lockfiles, set consistent timestamps, and use the same compiler/runtime versions across environments.*

---

[← Previous: Build Tools Overview](01-build-tools-overview.md) | [Back to README](../README.md) | [Next: Dependency Management →](03-dependency-management.md)
