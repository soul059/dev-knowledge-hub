# Chapter 4.5: Performance Testing in CI/CD

[← Previous: End-to-End Testing](04-end-to-end-testing.md) | [Back to README](../README.md) | [Next: Security Testing →](06-security-testing.md)

---

## Overview

**Performance testing** validates that your application meets speed, scalability, and stability requirements under expected and peak loads. Integrating performance tests into CI/CD catches regressions before they reach production.

---

## Types of Performance Tests

```
  ┌──────────────────────────────────────────────────────┐
  │            PERFORMANCE TEST TYPES                    │
  │                                                      │
  │  LOAD TEST          STRESS TEST        SPIKE TEST    │
  │  ──────────         ───────────        ──────────    │
  │  Users ▲            Users ▲            Users ▲       │
  │   100  │ ████████    500  │    ████      500 │  █    │
  │        │ ████████         │   █████          │ ███   │
  │        │ ████████         │  ██████          │█████  │
  │        └─────────►        └────────►         └────►  │
  │  Normal load        Beyond capacity    Sudden burst  │
  │  for extended        to find break      instant load │
  │  period              point                           │
  │                                                      │
  │  SOAK TEST          CAPACITY TEST                    │
  │  ─────────          ─────────────                    │
  │  Users ▲            Users ▲                          │
  │   100  │████████████  ?   │  ░░░████                 │
  │        │████████████      │  ░░████                  │
  │        │████████████      │  ░████                   │
  │        └────────────►     └────────►                 │
  │  Sustained load      Find max users                  │
  │  for hours/days      system handles                  │
  └──────────────────────────────────────────────────────┘
```

---

## Performance Testing Tools

| Tool | Type | Protocol | Language | Best For |
|---|---|---|---|---|
| **k6** | Load testing | HTTP, WebSocket, gRPC | JavaScript | Modern, developer-friendly |
| **JMeter** | Load testing | HTTP, JDBC, LDAP | GUI / XML | Java ecosystem, enterprise |
| **Gatling** | Load testing | HTTP, WebSocket | Scala / Java | High-perf simulations |
| **Locust** | Load testing | HTTP | Python | Python teams, distributed |
| **Artillery** | Load testing | HTTP, WebSocket | YAML / JS | Serverless, quick setup |
| **Lighthouse** | Frontend perf | Chrome | CLI / Node | Web Core Vitals |

---

## k6 Load Test Example

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

// Test configuration
export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up to 50 users
    { duration: '3m', target: 50 },   // Stay at 50 users
    { duration: '1m', target: 100 },  // Ramp up to 100
    { duration: '3m', target: 100 },  // Stay at 100
    { duration: '1m', target: 0 },    // Ramp down to 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
    http_reqs: ['rate>100'],           // Throughput > 100 req/s
  },
};

export default function () {
  // Test API endpoint
  const res = http.get('http://localhost:3000/api/products');

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'body has products': (r) => JSON.parse(r.body).length > 0,
  });

  sleep(1); // Think time between requests
}
```

---

## Performance Testing in CI

```yaml
# GitHub Actions — k6 performance tests
name: Performance Tests
on:
  pull_request:
    branches: [main]

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Start application
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm start &
      - run: npx wait-on http://localhost:3000

      # Install k6
      - name: Install k6
        run: |
          sudo gpg -k
          sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg \
            --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D68
          echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" \
            | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install k6

      # Run load test
      - name: Run k6 load test
        run: k6 run --out json=results.json tests/performance/load-test.js

      # Upload results
      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: k6-results
          path: results.json

      # Comment PR with results
      - name: Post results to PR
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = fs.readFileSync('results.json', 'utf8');
            // Parse and format results...
```

---

## Key Performance Metrics

```
  ┌──────────────────────────────────────────────────┐
  │          KEY PERFORMANCE METRICS                 │
  │                                                  │
  │  LATENCY (Response Time)                         │
  │  ├── p50  = 120ms  (median)                      │
  │  ├── p90  = 350ms  (90th percentile)             │
  │  ├── p95  = 480ms  (95th percentile)   ← target  │
  │  └── p99  = 890ms  (99th percentile)             │
  │                                                  │
  │  THROUGHPUT                                      │
  │  ├── Requests/sec  = 1,250 req/s                 │
  │  └── Bytes/sec     = 15 MB/s                     │
  │                                                  │
  │  ERROR RATE                                      │
  │  ├── HTTP errors   = 0.3%                        │
  │  └── Timeouts      = 0.1%                        │
  │                                                  │
  │  RESOURCE USAGE                                  │
  │  ├── CPU           = 65%                         │
  │  ├── Memory        = 2.1 GB / 4 GB               │
  │  └── Connections   = 450 active                  │
  └──────────────────────────────────────────────────┘
```

---

## Lighthouse CI (Frontend Performance)

```yaml
# GitHub Actions — Lighthouse CI
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v11
  with:
    urls: |
      http://localhost:3000
      http://localhost:3000/products
    budgetPath: ./budget.json
    uploadArtifacts: true

# budget.json — Performance budgets
# {
#   "ci": {
#     "assert": {
#       "assertions": {
#         "categories:performance": ["error", { "minScore": 0.9 }],
#         "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
#         "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
#         "total-blocking-time": ["error", { "maxNumericValue": 300 }],
#         "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }]
#       }
#     }
#   }
# }
```

---

## Performance Budgets

```
  PERFORMANCE BUDGET
  ══════════════════

  Metric                    Budget      Actual     Status
  ─────────────────────────────────────────────────────
  p95 Response Time         < 500ms     480ms      ✓ PASS
  p99 Response Time         < 1000ms    890ms      ✓ PASS
  Error Rate                < 1%        0.3%       ✓ PASS
  Throughput                > 100 rps   1,250 rps  ✓ PASS
  First Contentful Paint    < 2.0s      1.8s       ✓ PASS
  Largest Contentful Paint  < 2.5s      2.7s       ✕ FAIL
  Total Blocking Time       < 300ms     250ms      ✓ PASS
  Bundle Size (JS)          < 250 KB    230 KB     ✓ PASS
  ─────────────────────────────────────────────────────
  Result: FAIL — LCP exceeds budget
```

---

## Locust Test Example (Python)

```python
# locustfile.py
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 3)  # Think time 1-3 seconds

    @task(3)  # Weight: 3x more likely
    def view_products(self):
        self.client.get("/api/products")

    @task(1)
    def view_product_detail(self):
        self.client.get("/api/products/1")

    @task(1)
    def search_products(self):
        self.client.get("/api/products?search=laptop")

    def on_start(self):
        """Login before testing."""
        self.client.post("/api/login", json={
            "email": "test@example.com",
            "password": "password123"
        })
```

```bash
# Run locally
locust -f locustfile.py --host=http://localhost:3000 --users 100 --spawn-rate 10

# Run headless in CI
locust -f locustfile.py --host=http://localhost:3000 \
  --users 100 --spawn-rate 10 --run-time 5m --headless \
  --csv=results
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Inconsistent results in CI | Shared CI runners, variable resources | Use dedicated runners; run multiple iterations |
| Results differ from production | CI environment too small | Match CI resources to production sizing |
| Tests always pass | Thresholds too lenient | Set budgets based on real SLAs; tighten p95 |
| Tests always fail | Unrealistic on CI hardware | Adjust thresholds for CI environment; compareToPR |
| Can't find performance regressions | No baseline comparison | Store and compare against previous run metrics |
| High variance between runs | Not enough warm-up | Add ramp-up stage; discard first minute of data |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Validate speed, scalability, and stability under load |
| **Types** | Load, stress, spike, soak, capacity |
| **Key metrics** | p95 latency, throughput, error rate, resource usage |
| **Tools** | k6, JMeter, Gatling, Locust, Lighthouse |
| **Runs** | On PR to main; nightly; before production deploy |
| **Budgets** | Set thresholds; fail CI when exceeded |

---

## Quick Revision Questions

1. **What is load testing?** *Testing the application under expected normal load to ensure it meets performance requirements.*
2. **What is the difference between load and stress testing?** *Load testing uses expected traffic levels; stress testing pushes beyond capacity to find breaking points.*
3. **What is p95 latency?** *The response time below which 95% of all requests complete — used as a performance threshold.*
4. **What is k6?** *A modern, developer-friendly load testing tool that uses JavaScript for test scripts and supports HTTP, WebSocket, and gRPC.*
5. **What are performance budgets?** *Defined thresholds for metrics (response time, bundle size, LCP) that fail CI if exceeded.*
6. **Why should performance tests run in CI?** *To catch performance regressions early, before they reach production and impact users.*

---

[← Previous: End-to-End Testing](04-end-to-end-testing.md) | [Back to README](../README.md) | [Next: Security Testing →](06-security-testing.md)
