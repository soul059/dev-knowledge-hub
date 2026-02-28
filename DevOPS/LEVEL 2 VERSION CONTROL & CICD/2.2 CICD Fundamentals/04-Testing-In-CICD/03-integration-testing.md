# Chapter 4.3: Integration Testing in CI/CD

[← Previous: Unit Testing](02-unit-testing.md) | [Back to README](../README.md) | [Next: End-to-End Testing →](04-end-to-end-testing.md)

---

## Overview

**Integration tests** verify that multiple components work together correctly — your code talking to databases, APIs, message queues, caches, and file systems. They sit in the middle of the test pyramid: more realistic than unit tests, faster than E2E tests.

---

## Integration vs Unit Tests

```
  UNIT TEST                        INTEGRATION TEST
  ══════════                       ════════════════

  ┌──────────┐                     ┌──────────┐
  │  Code    │──mock──►[DB Mock]   │  Code    │──real──►┌──────┐
  │  Under   │                     │  Under   │         │  DB  │
  │  Test    │──mock──►[API Mock]  │  Test    │──real──►├──────┤
  └──────────┘                     └──────────┘         │ API  │
                                                        └──────┘
  Speed: ~5ms                      Speed: ~500ms
  Deps:  All mocked                Deps:  Real (containerized)
  Scope: Single function           Scope: Multiple components
```

---

## Common Integration Test Scenarios

| Scenario | Tests That... | Dependencies |
|---|---|---|
| **Database** | Queries, migrations, transactions work | PostgreSQL, MySQL, MongoDB |
| **REST API** | Endpoints return correct responses | App server, maybe DB |
| **Message Queue** | Messages are published/consumed | RabbitMQ, Kafka |
| **Cache** | Cache reads/writes/invalidation work | Redis, Memcached |
| **File Storage** | Uploads/downloads work | S3, MinIO |
| **Auth** | OAuth/JWT flows work end-to-end | Auth service |

---

## Service Containers in CI

```yaml
# GitHub Actions — Integration tests with services
name: Integration Tests
on: [pull_request]

jobs:
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports: ['6379:6379']
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
```

### GitLab CI with Services

```yaml
# .gitlab-ci.yml
integration-tests:
  stage: test
  image: node:20
  services:
    - name: postgres:16
      alias: db
      variables:
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
        POSTGRES_DB: testdb
    - name: redis:7
      alias: cache
  variables:
    DATABASE_URL: "postgres://test:test@db:5432/testdb"
    REDIS_URL: "redis://cache:6379"
  script:
    - npm ci
    - npm run test:integration
```

---

## Testcontainers (Programmatic Containers)

```
  ┌─────────────────────────────────────────────────┐
  │               TESTCONTAINERS FLOW               │
  │                                                 │
  │  Test Start ──► Spin up Docker container        │
  │                 (Postgres, Redis, Kafka, etc.)   │
  │             ──► Wait for container to be ready   │
  │             ──► Run integration tests            │
  │             ──► Tear down container              │
  │  Test End                                       │
  │                                                 │
  │  Benefits:                                      │
  │  ✓ Real dependency, not a mock                  │
  │  ✓ Same test runs locally & in CI               │
  │  ✓ No shared test databases                     │
  │  ✓ Each test suite gets a fresh container       │
  └─────────────────────────────────────────────────┘
```

### Java (Testcontainers)

```java
// build.gradle
// testImplementation 'org.testcontainers:postgresql:1.19.3'

import org.testcontainers.containers.PostgreSQLContainer;
import org.junit.jupiter.api.*;
import java.sql.*;

class UserRepositoryIntegrationTest {

    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @BeforeAll
    static void startContainer() { postgres.start(); }

    @AfterAll
    static void stopContainer() { postgres.stop(); }

    @Test
    void shouldSaveAndFindUser() throws Exception {
        String url = postgres.getJdbcUrl();
        try (Connection conn = DriverManager.getConnection(url, "test", "test")) {
            conn.createStatement().execute(
                "CREATE TABLE users (id SERIAL, name TEXT)"
            );
            conn.createStatement().execute(
                "INSERT INTO users (name) VALUES ('Alice')"
            );

            ResultSet rs = conn.createStatement()
                .executeQuery("SELECT name FROM users WHERE id=1");
            rs.next();
            assertEquals("Alice", rs.getString("name"));
        }
    }
}
```

### Node.js (Testcontainers)

```javascript
const { GenericContainer } = require('testcontainers');
const { Client } = require('pg');

describe('User Repository', () => {
  let container, client;

  beforeAll(async () => {
    container = await new GenericContainer('postgres:16')
      .withEnvironment({
        POSTGRES_USER: 'test',
        POSTGRES_PASSWORD: 'test',
        POSTGRES_DB: 'testdb',
      })
      .withExposedPorts(5432)
      .start();

    client = new Client({
      host: container.getHost(),
      port: container.getMappedPort(5432),
      user: 'test', password: 'test', database: 'testdb',
    });
    await client.connect();
  });

  afterAll(async () => {
    await client.end();
    await container.stop();
  });

  test('saves and retrieves a user', async () => {
    await client.query('CREATE TABLE users (id SERIAL, name TEXT)');
    await client.query("INSERT INTO users (name) VALUES ('Alice')");
    const res = await client.query('SELECT name FROM users WHERE id=1');
    expect(res.rows[0].name).toBe('Alice');
  });
});
```

---

## Database Migration Testing

```
  ┌──────────────────────────────────────────────────┐
  │          MIGRATION TESTING STRATEGY              │
  │                                                  │
  │  1. Start empty DB container                     │
  │  2. Run ALL migrations from scratch              │
  │  3. Verify schema matches expectations           │
  │  4. Run seed data scripts                        │
  │  5. Run integration tests against seeded DB      │
  │  6. Test rollback migrations                     │
  │  7. Tear down container                          │
  │                                                  │
  │  This catches:                                   │
  │  ✓ Migration ordering issues                     │
  │  ✓ Incompatible schema changes                   │
  │  ✓ Missing rollback scripts                      │
  │  ✓ Data migration bugs                           │
  └──────────────────────────────────────────────────┘
```

---

## API Integration Testing

```python
# Python — Testing REST API endpoints
import pytest
import requests

BASE_URL = "http://localhost:8080"

@pytest.fixture(scope="session")
def api_server():
    """Start server before tests, stop after."""
    import subprocess
    proc = subprocess.Popen(["python", "app.py"])
    # Wait for server to be ready
    import time
    time.sleep(2)
    yield
    proc.terminate()

class TestUserAPI:
    def test_create_user(self, api_server):
        resp = requests.post(f"{BASE_URL}/users", json={
            "name": "Alice", "email": "alice@example.com"
        })
        assert resp.status_code == 201
        assert resp.json()["name"] == "Alice"
        self.user_id = resp.json()["id"]

    def test_get_user(self, api_server):
        resp = requests.get(f"{BASE_URL}/users/1")
        assert resp.status_code == 200
        assert resp.json()["email"] == "alice@example.com"

    def test_get_nonexistent_user(self, api_server):
        resp = requests.get(f"{BASE_URL}/users/9999")
        assert resp.status_code == 404
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Service container not ready | Health check misconfigured | Add proper health checks with retries |
| Connection refused in CI | Port mapping issue | Use service hostname (not localhost) in GitLab CI |
| Tests pass locally, fail in CI | Docker-in-Docker issues | Use service containers or Testcontainers w/ DinD |
| Slow integration tests | Container startup overhead | Reuse containers across tests; use `beforeAll` |
| Flaky database tests | Shared state between tests | Use transactions with rollback, or truncate tables |
| Out of disk in CI | Container images accumulate | Prune images; use smaller base images |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Verify components work together correctly |
| **Dependencies** | Real (containerized) databases, caches, APIs |
| **Speed** | Seconds per test (container startup + teardown) |
| **Tools** | Service containers, Testcontainers, Docker Compose |
| **Runs** | On PRs and before deployment |
| **Key patterns** | Fresh container per suite, transaction rollback |

---

## Quick Revision Questions

1. **What does an integration test verify?** *That multiple components (code + database, API, cache, etc.) work together correctly.*
2. **What are service containers in CI?** *Docker containers spun up by the CI platform alongside the test runner, providing real dependencies like PostgreSQL or Redis.*
3. **What is Testcontainers?** *A library that programmatically manages Docker containers from within test code, giving each test suite fresh dependencies.*
4. **How do you avoid flaky database tests?** *Use transaction rollback after each test, truncate tables between tests, or start fresh containers per suite.*
5. **Why use real containers instead of mocks for integration tests?** *To test actual behavior of dependencies — real SQL queries, real connection handling — not simulated behavior.*
6. **How do integration tests differ from E2E tests?** *Integration tests verify specific component interactions; E2E tests verify complete user workflows through the full application stack.*

---

[← Previous: Unit Testing](02-unit-testing.md) | [Back to README](../README.md) | [Next: End-to-End Testing →](04-end-to-end-testing.md)
