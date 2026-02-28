# Chapter 3: Synthetic Monitoring

[← Previous: APM Tools](02-apm-tools.md) | [Back to README](../README.md) | [Next: Performance Profiling →](04-performance-profiling.md)

---

## Overview

Synthetic monitoring uses scripted tests that simulate user interactions from external locations to proactively detect issues before real users are affected. Unlike RUM (Real User Monitoring), synthetic tests run continuously even when no real users are active.

---

## 3.1 Synthetic vs Real User Monitoring

```
┌──────────────────────────────────────────────────────────┐
│  SYNTHETIC MONITORING vs REAL USER MONITORING            │
│                                                          │
│  SYNTHETIC (Proactive)          RUM (Reactive)           │
│  ┌─────────────────────┐      ┌─────────────────────┐  │
│  │ Scripted tests       │      │ Real user data       │  │
│  │ Known baseline       │      │ Actual experience    │  │
│  │ 24/7 (no users req.) │      │ Only when users visit│  │
│  │ Controlled environ.  │      │ Diverse environments │  │
│  │ Consistent results   │      │ Noisy data           │  │
│  │ Detect before users  │      │ Detect with users    │  │
│  │ Limited user journeys│      │ All user journeys    │  │
│  └─────────────────────┘      └─────────────────────┘  │
│                                                          │
│  Best practice: Use BOTH together                       │
│  • Synthetic for baseline and availability              │
│  • RUM for actual user experience                       │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Types of Synthetic Tests

```
┌──────────────────────────────────────────────────────────┐
│  SYNTHETIC TEST TYPES                                    │
│                                                          │
│  1. UPTIME/PING CHECK (Simplest)                        │
│     HTTP GET → Status 200? Response < 2s?               │
│     Frequency: Every 1-5 minutes                        │
│     Example: Check if homepage responds                  │
│                                                          │
│  2. API MONITORING (Medium)                              │
│     Send API request → Validate response JSON           │
│     Assert: status, body content, headers, latency      │
│     Example: POST /api/login → check token returned     │
│                                                          │
│  3. BROWSER TESTS (Complex)                              │
│     Full browser simulation (headless Chrome)            │
│     Click buttons, fill forms, navigate pages           │
│     Measure Core Web Vitals, screenshots on failure     │
│     Example: Complete checkout flow                     │
│                                                          │
│  4. MULTI-STEP TRANSACTIONS                             │
│     Chain of API/browser steps simulating a workflow    │
│     Login → Search → Add to cart → Checkout             │
│     Assert each step succeeds                           │
│                                                          │
│  5. SSL/TLS MONITORING                                  │
│     Check certificate expiry, chain validity            │
│     Alert days before expiration                        │
└──────────────────────────────────────────────────────────┘
```

---

## 3.3 Synthetic Monitoring Tools

| Tool | Type | Features | Pricing |
|------|------|----------|---------|
| Grafana Synthetic Monitoring | OSS + Cloud | k6 browser, global probes | Free tier / usage |
| Datadog Synthetics | Commercial | Browser, API, multi-step | Per test run |
| New Relic Synthetics | Commercial | Scripted browser, API | Included in plan |
| Checkly | Dedicated | Playwright-based, CI/CD native | Per check |
| Uptime Robot | Dedicated | Simple HTTP checks | Free tier |
| Pingdom | Commercial | Uptime + page speed | Per plan |
| Better Uptime | Commercial | Status page + monitoring | Per plan |

---

## 3.4 Implementing Synthetic Tests

### Simple HTTP Check (Grafana k6)

```javascript
// k6 HTTP check script
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  thresholds: {
    http_req_duration: ['avg<500', 'p(95)<1000'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  // Check homepage
  const homepage = http.get('https://example.com');
  check(homepage, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'body contains title': (r) => r.body.includes('Welcome'),
  });
  sleep(1);

  // Check API health
  const health = http.get('https://api.example.com/health');
  check(health, {
    'health check passes': (r) => r.status === 200,
    'API is healthy': (r) => JSON.parse(r.body).status === 'ok',
  });
}
```

### Browser Test (Playwright-based Checkly)

```javascript
// Checkly browser check
const { chromium } = require('playwright');

async function run() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Navigate to login page
  await page.goto('https://app.example.com/login');
  
  // Fill login form
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'testpassword');
  await page.click('#login-button');
  
  // Verify dashboard loads
  await page.waitForSelector('#dashboard', { timeout: 10000 });
  
  // Check for critical elements
  const title = await page.textContent('h1');
  if (!title.includes('Dashboard')) {
    throw new Error(`Expected Dashboard, got: ${title}`);
  }
  
  // Take screenshot for reference
  await page.screenshot({ path: 'dashboard.png' });
  
  await browser.close();
}

run();
```

### API Multi-Step Check

```javascript
// Multi-step API synthetic test
import http from 'k6/http';
import { check, group } from 'k6';

export default function () {
  let token;

  group('Step 1: Authenticate', () => {
    const loginRes = http.post('https://api.example.com/auth/login', 
      JSON.stringify({ email: 'test@example.com', password: 'test' }),
      { headers: { 'Content-Type': 'application/json' } }
    );
    check(loginRes, { 'login successful': (r) => r.status === 200 });
    token = JSON.parse(loginRes.body).token;
  });

  group('Step 2: Search Products', () => {
    const searchRes = http.get('https://api.example.com/products?q=laptop', {
      headers: { Authorization: `Bearer ${token}` },
    });
    check(searchRes, {
      'search returns results': (r) => JSON.parse(r.body).items.length > 0,
      'search response < 1s': (r) => r.timings.duration < 1000,
    });
  });

  group('Step 3: Add to Cart', () => {
    const cartRes = http.post('https://api.example.com/cart/add',
      JSON.stringify({ productId: 'laptop-001', quantity: 1 }),
      { headers: { Authorization: `Bearer ${token}`, 'Content-Type': 'application/json' } }
    );
    check(cartRes, { 'item added to cart': (r) => r.status === 201 });
  });

  group('Step 4: Verify Cart', () => {
    const cartRes = http.get('https://api.example.com/cart', {
      headers: { Authorization: `Bearer ${token}` },
    });
    check(cartRes, {
      'cart has items': (r) => JSON.parse(r.body).items.length === 1,
    });
  });
}
```

---

## 3.5 Testing from Multiple Locations

```
┌──────────────────────────────────────────────────────────┐
│  MULTI-REGION SYNTHETIC TESTING                          │
│                                                          │
│       ┌─────────┐                                       │
│       │ US-East │──── 150ms ────┐                      │
│       └─────────┘                │                      │
│       ┌─────────┐                ▼                      │
│       │ US-West │──── 120ms ──►┌──────────┐           │
│       └─────────┘              │   YOUR   │           │
│       ┌─────────┐              │ WEBSITE  │           │
│       │ EU-West │──── 200ms ──►│          │           │
│       └─────────┘              └──────────┘           │
│       ┌─────────┐                ▲                      │
│       │ AP-South│──── 350ms ────┘                      │
│       └─────────┘                                       │
│                                                          │
│  Benefits:                                               │
│  • Detect regional outages (CDN, DNS issues)            │
│  • Measure latency from user perspective                │
│  • Verify geo-routing works correctly                   │
│  • Meet data residency requirements                     │
└──────────────────────────────────────────────────────────┘
```

---

## 3.6 Synthetic Monitoring in CI/CD

```yaml
# GitHub Actions - Run synthetic tests before deploy
name: Deploy with Synthetic Validation
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        run: ./deploy.sh staging
      
      - name: Run synthetic tests against staging
        run: |
          npx checkly test \
            --env ENVIRONMENT=staging \
            --reporter=github
      
      - name: Deploy to production
        if: success()
        run: ./deploy.sh production
      
      - name: Run smoke tests against production
        run: |
          npx checkly test \
            --env ENVIRONMENT=production \
            --reporter=github \
            --tags smoke
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Flaky tests | Timing issues, animations | Add explicit waits, increase timeouts |
| False positives | Network blip from test location | Add retry logic, alert after 2 consecutive failures |
| Test blocked by WAF | Test IP not allowlisted | Allowlist synthetic monitoring IPs |
| SSL test false alarm | Intermediate cert issue | Check full certificate chain |
| Test passes but users report issues | Test path differs from real user | Align tests with actual user journeys |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Purpose | Proactive detection before users are affected |
| Test Types | Ping, API, Browser, Multi-step, SSL |
| Key Metric | Availability %, response time, test pass rate |
| Multi-region | Tests from global locations for regional issues |
| CI/CD | Run synthetic tests as deployment gates |
| Tools | Grafana k6, Checkly, Datadog, Uptime Robot |

---

## Quick Revision Questions

1. **How does synthetic monitoring differ from Real User Monitoring (RUM)?**
2. **What are the five types of synthetic tests?**
3. **Why is multi-location testing important?**
4. **How can synthetic tests be integrated into CI/CD pipelines?**
5. **How do you handle false positives in synthetic monitoring?**

---

[← Previous: APM Tools](02-apm-tools.md) | [Back to README](../README.md) | [Next: Performance Profiling →](04-performance-profiling.md)
