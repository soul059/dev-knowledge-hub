# Chapter 3.5: Multi-Stage Builds

[← Previous: Build Optimization](04-build-optimization.md) | [Back to README](../README.md) | [Next: Build Caching Strategies →](06-build-caching-strategies.md)

---

## Overview

**Multi-stage builds** use multiple `FROM` statements in a Dockerfile. Each stage can use a different base image and only the final stage becomes your production image. This separates build-time tools from runtime, producing **smaller, more secure images**.

---

## The Problem: Bloated Images

```
SINGLE-STAGE BUILD (Bloated)
═════════════════════════════

  ┌──────────────────────────────────────────┐
  │  Production Image (~1.2 GB)              │
  │                                          │
  │  ✅ Application code                     │
  │  ✅ Runtime (Node.js)                    │
  │  ❌ npm (not needed at runtime)          │
  │  ❌ Build tools (webpack, tsc)           │
  │  ❌ Dev dependencies                     │
  │  ❌ Source code (TypeScript)             │
  │  ❌ Test files                           │
  └──────────────────────────────────────────┘

MULTI-STAGE BUILD (Lean)
═════════════════════════

  Stage 1: Builder (discarded)    Stage 2: Production Image
  ┌─────────────────────┐         ┌──────────────────────┐
  │  npm, webpack, tsc  │         │  App code (~150 MB)  │
  │  Dev dependencies   │  COPY   │                      │
  │  Source code        │───────▶│  ✅ Compiled JS       │
  │  Test files         │  only   │  ✅ Runtime deps     │
  │  Build output ✅    │  what's │  ✅ Node.js runtime  │
  └─────────────────────┘  needed └──────────────────────┘
       (thrown away)               (shipped to prod)
```

---

## Multi-Stage Dockerfile Examples

### Node.js Application

```dockerfile
# ─── Stage 1: Build ───
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
RUN npm prune --production   # Remove devDependencies

# ─── Stage 2: Production ───
FROM node:20-alpine AS production
WORKDIR /app
# Copy ONLY production dependencies and build output
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json .

EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

### Java Application

```dockerfile
# ─── Stage 1: Build with Maven ───
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline     # Cache deps layer
COPY src ./src
RUN mvn clean package -DskipTests

# ─── Stage 2: Runtime only ───
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/myapp-*.jar app.jar
EXPOSE 8080
USER 1000
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Go Application (Scratch Image)

```dockerfile
# ─── Stage 1: Build ───
FROM golang:1.22-alpine AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /server ./cmd/server

# ─── Stage 2: Scratch (no OS!) ───
FROM scratch
COPY --from=build /server /server
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 8080
ENTRYPOINT ["/server"]
# Final image: ~10-15 MB!
```

---

## Image Size Comparison

```
Image Sizes: Node.js App
════════════════════════

  Single-stage (node:20)     ████████████████████████████████ 1.1 GB
  Multi-stage (node:20)      ████████████████                 500 MB
  Multi-stage (alpine)       ████████                         150 MB
  Multi-stage (distroless)   ██████                           100 MB

Image Sizes: Go App
════════════════════

  Single-stage (golang:1.22) ████████████████████████████████ 850 MB
  Multi-stage (alpine)       ████                              25 MB
  Multi-stage (scratch)      ██                                12 MB
```

---

## Three-Stage Build Pattern

```dockerfile
# ─── Stage 1: Dependencies ───
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# ─── Stage 2: Build ───
FROM node:20-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# ─── Stage 3: Production ───
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
RUN npm prune --production
USER node
CMD ["node", "dist/server.js"]
```

**Benefits of separate stages:**
- Dependencies cached independently from source code changes
- Build tools never reach production image
- Security: minimal attack surface

---

## Using Targets for Dev vs Prod

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS development
RUN npm install                    # ALL deps including dev
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS production
RUN npm ci --production            # Only prod deps
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

```bash
# Build for development
docker build --target development -t myapp:dev .

# Build for production
docker build --target production -t myapp:prod .
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Runtime error: "module not found" | Production deps missing | Verify COPY includes node_modules with prod deps |
| Final image still large | Too much copied from builder | Be explicit about what you COPY |
| Build cache invalidated every time | COPY . too early | Copy lockfile first, install deps, then COPY source |
| Permission errors at runtime | Running as root | Add `USER node` or `USER 1000` |

---

## Summary Table

| Concept | Description |
|---|---|
| **Multi-stage build** | Multiple FROM stages in one Dockerfile |
| **Builder stage** | Contains build tools and dev deps (discarded) |
| **Production stage** | Minimal image with only runtime requirements |
| **Scratch image** | Empty base image (Go binaries) — smallest possible |
| **Target builds** | Build specific stages with `--target` flag |
| **Size reduction** | 50-95% smaller images than single-stage |
| **Security** | Smaller attack surface — no compilers, no dev tools |

---

## Quick Revision Questions

1. **What is a multi-stage Docker build?** *A Dockerfile with multiple FROM statements where each stage can have a different base image, and only the final stage produces the output image.*
2. **Why are multi-stage builds more secure?** *Build tools, compilers, and dev dependencies are excluded from the final image, reducing the attack surface.*
3. **What is a `scratch` base image?** *An empty image with no OS — only a statically compiled binary can run in it. Produces the smallest possible images.*
4. **How can you build different targets from the same Dockerfile?** *Use `docker build --target <stage-name>` to build a specific stage.*
5. **Why should you copy lockfiles before source code in a Dockerfile?** *So the dependency installation layer is cached and only invalidated when dependencies change, not when source code changes.*
6. **What typical image size reduction can multi-stage builds achieve?** *50-95% reduction depending on the language and base image choice.*

---

[← Previous: Build Optimization](04-build-optimization.md) | [Back to README](../README.md) | [Next: Build Caching Strategies →](06-build-caching-strategies.md)
