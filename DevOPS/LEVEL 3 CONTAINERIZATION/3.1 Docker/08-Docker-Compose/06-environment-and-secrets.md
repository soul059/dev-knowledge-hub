# Chapter 6: Environment and Secrets

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

Managing configuration and sensitive data is critical in containerized applications. Docker Compose provides multiple mechanisms for injecting environment variables and handling secrets, following the 12-factor app methodology of storing config in the environment.

---

## 6.1 Environment Variables

```yaml
services:
  api:
    image: myapi

    # Method 1: Inline key-value (most common)
    environment:
      NODE_ENV: production
      DB_HOST: db
      DB_PORT: "5432"
      DEBUG: "false"

    # Method 2: List format
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - DB_PORT=5432

    # Method 3: From .env file
    env_file:
      - .env
      - .env.production

    # Method 4: Pass from host (value from shell)
    environment:
      - API_KEY              # Takes value from host env
      - SECRET_TOKEN         # undefined if not set on host
```

```
  Environment Variable Precedence (highest → lowest):
  
  1. docker compose run -e VAR=val    (CLI override)
  2. environment: in compose.yaml      (inline)
  3. env_file: in compose.yaml         (.env files)
  4. Dockerfile ENV instruction        (image default)
  
  Higher precedence values OVERRIDE lower ones
```

---

## 6.2 The .env File

```bash
# .env (in same directory as compose.yaml)
# Automatically loaded by Compose for variable substitution

# Database
POSTGRES_USER=myapp
POSTGRES_PASSWORD=supersecret
POSTGRES_DB=myapp_production

# App settings
APP_PORT=3000
LOG_LEVEL=info
```

```yaml
# compose.yaml — using .env variables
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${DB_PORT:-5432}:5432"    # Default if not set

  api:
    build: ./api
    ports:
      - "${APP_PORT}:3000"
    environment:
      DATABASE_URL: postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
```

```
  .env File Processing:
  
  .env file                   compose.yaml
  +------------------+        +---------------------------+
  | APP_PORT=3000    | -----> | ports:                    |
  | DB_PASSWORD=xyz  |        |   - "${APP_PORT}:3000"    |
  +------------------+        | environment:              |
                              |   DB_PASS: ${DB_PASSWORD} |
                              +---------------------------+
  
  The .env file is for COMPOSE variable substitution
  NOT automatically passed to containers!
  
  To pass to containers, use:
  - environment: section
  - env_file: section
```

### Variable Substitution Syntax

```yaml
services:
  api:
    image: myapi:${TAG}                # Required (error if unset)
    image: myapi:${TAG:-latest}        # Default value
    image: myapi:${TAG-latest}         # Default if unset (not empty)
    image: myapi:${TAG:?Error message} # Error if unset/empty
    image: myapi:${TAG?Error message}  # Error if unset
```

---

## 6.3 Multiple Environment Files

```yaml
services:
  api:
    image: myapi
    env_file:
      - .env                    # Common settings
      - .env.${ENV:-development}  # Environment-specific
      - path: .env.local        # Optional local overrides
        required: false         # Don't fail if missing
```

```
  Environment File Layering:
  
  .env.defaults          .env.production        .env.local
  +---------------+      +---------------+      +---------------+
  | LOG_LEVEL=debug|     | LOG_LEVEL=warn |     | LOG_LEVEL=info|
  | DB_HOST=local  |     | DB_HOST=rds.aws|     |               |
  | PORT=3000      |     | PORT=8080      |     | DEBUG=true    |
  +---------------+      +---------------+      +---------------+
        |                       |                      |
        +-------+---------------+----------+-----------+
                                           |
                              Final values (last wins):
                              LOG_LEVEL=info  (from .env.local)
                              DB_HOST=rds.aws (from .env.production)
                              PORT=8080       (from .env.production)
                              DEBUG=true      (from .env.local)
```

---

## 6.4 Docker Compose Secrets

```yaml
services:
  db:
    image: postgres:16
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

  api:
    build: ./api
    secrets:
      - db_password
      - api_key
      - source: ssl_cert
        target: /etc/ssl/cert.pem
        uid: "1000"
        gid: "1000"
        mode: 0400

secrets:
  db_password:
    file: ./secrets/db_password.txt    # From file
  api_key:
    environment: API_KEY               # From env var
  ssl_cert:
    file: ./certs/server.crt
```

```
  Secrets in Compose:
  
  Host:                          Container:
  ./secrets/                     /run/secrets/
    db_password.txt ──────────>    db_password
                                   (mounted as file)
  
  secrets:
    db_password:
      file: ./secrets/db_password.txt
  
  Inside container:
  $ cat /run/secrets/db_password
  my-secret-password
  
  Advantages over environment variables:
  ✓ Not visible in docker inspect
  ✓ Not in process listing  
  ✓ File-based (many apps support _FILE pattern)
  ✓ Mounted as tmpfs (in-memory)
```

### Using _FILE Convention

```bash
# Many official images support _FILE suffix
# Instead of: POSTGRES_PASSWORD=secret
# Use:        POSTGRES_PASSWORD_FILE=/run/secrets/db_password

# Supported images:
# postgres, mysql, mariadb, mongo, wordpress, etc.
```

---

## 6.5 Security Best Practices

```
  Environment Variable Security:
  
  ✗ BAD: Secrets in compose.yaml
  environment:
    DB_PASSWORD: supersecret123     # Committed to git!
  
  ✗ BAD: Secrets in .env (if committed)
  DB_PASSWORD=supersecret123        # In version control!
  
  ✓ GOOD: .env in .gitignore
  # .gitignore
  .env
  .env.local
  .env.production
  
  ✓ GOOD: Use secrets
  secrets:
    db_password:
      file: ./secrets/db_password.txt
  
  ✓ BETTER: External secret management
  # HashiCorp Vault, AWS Secrets Manager, etc.
```

```bash
# .gitignore entries for secrets
.env
.env.*
!.env.example         # Keep example file
secrets/
*.key
*.pem
```

```yaml
# .env.example (committed to git — no real values)
# Copy to .env and fill in real values
POSTGRES_USER=myapp
POSTGRES_PASSWORD=CHANGE_ME
API_KEY=CHANGE_ME
JWT_SECRET=CHANGE_ME
```

---

## 6.6 Configuration Patterns

```yaml
# Pattern 1: Dev/Prod with override files
# compose.yaml (base)
services:
  api:
    build: ./api
    environment:
      NODE_ENV: production

# compose.override.yaml (auto-loaded for dev)
services:
  api:
    environment:
      NODE_ENV: development
      DEBUG: "true"
    env_file:
      - .env.development
```

```yaml
# Pattern 2: Configs for files (non-secret config)
services:
  web:
    image: nginx
    configs:
      - source: nginx_config
        target: /etc/nginx/conf.d/default.conf

configs:
  nginx_config:
    file: ./nginx/default.conf
```

---

## Summary Table

| Method | Use Case | Security | Visibility |
|--------|----------|----------|------------|
| **environment:** | Non-sensitive config | Low | `docker inspect` |
| **env_file:** | Bulk env vars | Medium | In container env |
| **.env** | Compose substitution | Medium | Not in container |
| **secrets:** | Passwords, keys | High | File at /run/secrets |
| **configs:** | Config files | Medium | Mounted file |
| **External vault** | Production secrets | Highest | Runtime only |

| Syntax | Meaning |
|--------|---------|
| `${VAR}` | Substitute, error if unset |
| `${VAR:-default}` | Use default if unset or empty |
| `${VAR-default}` | Use default only if unset |
| `${VAR:?error}` | Error message if unset or empty |

---

## Quick Revision Questions

1. **What is the difference between the `.env` file and `env_file:`?**
   - `.env` is used for Compose variable substitution (in the YAML file itself). `env_file:` passes variables directly to the container's environment

2. **Why are secrets better than environment variables for passwords?**
   - Secrets are mounted as files in `/run/secrets/`, not visible in `docker inspect` or process listings, and stored in-memory. Environment variables are exposed in multiple places

3. **What does `${DB_PORT:-5432}` mean?**
   - Use the value of `DB_PORT` if it's set and non-empty, otherwise default to `5432`

4. **How do official database images use secrets?**
   - They support the `_FILE` suffix convention: `POSTGRES_PASSWORD_FILE=/run/secrets/db_password` reads the password from the secret file instead of the environment variable

5. **What should be in `.gitignore` for a Compose project?**
   - `.env`, `.env.*` (except `.env.example`), `secrets/` directory, private keys, and certificates. Commit `.env.example` with placeholder values as documentation

---

[← Previous: Volumes in Compose](05-volumes-in-compose.md) | [Next: Compose in Production →](07-compose-in-production.md)
