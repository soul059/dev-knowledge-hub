# Chapter 4.4: End-to-End (E2E) Testing in CI/CD

[← Previous: Integration Testing](03-integration-testing.md) | [Back to README](../README.md) | [Next: Performance Testing →](05-performance-testing.md)

---

## Overview

**End-to-End (E2E) tests** simulate real user interactions through the entire application stack — browser, frontend, backend, database, and external services. They verify that complete user journeys work as expected.

---

## E2E Test Architecture

```
  ┌──────────────────────────────────────────────────┐
  │                E2E TEST SETUP                    │
  │                                                  │
  │  ┌──────────┐    ┌──────────┐    ┌──────────┐   │
  │  │ E2E Test │───►│ Browser  │───►│ Frontend │   │
  │  │ Runner   │    │ (Chrome) │    │ (React)  │   │
  │  │ Playwright│   └──────────┘    └────┬─────┘   │
  │  └──────────┘                         │         │
  │                                   API calls     │
  │                                       │         │
  │                                  ┌────▼─────┐   │
  │                                  │ Backend  │   │
  │                                  │ (Node.js)│   │
  │                                  └────┬─────┘   │
  │                                       │         │
  │                                  ┌────▼─────┐   │
  │                                  │ Database │   │
  │                                  │ (Postgres)│  │
  │                                  └──────────┘   │
  └──────────────────────────────────────────────────┘
```

---

## E2E Testing Frameworks

| Framework | Language | Browsers | Speed | Key Feature |
|---|---|---|---|---|
| **Playwright** | JS/TS/Python/Java | Chrome, Firefox, Safari | Fast | Auto-wait, multi-browser |
| **Cypress** | JavaScript | Chrome, Firefox, Edge | Medium | Time travel debugging |
| **Selenium** | Multi-language | All browsers | Slow | Industry standard |
| **Puppeteer** | JavaScript | Chrome only | Fast | Chrome DevTools Protocol |

---

## Playwright Example

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('successful login redirects to dashboard', async ({ page }) => {
    // Navigate to login page
    await page.goto('/login');

    // Fill in credentials
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'Password123!');

    // Click login
    await page.click('[data-testid="login-button"]');

    // Verify redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toContainText('Welcome');
  });

  test('failed login shows error message', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrong');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('.error-message')).toContainText(
      'Invalid credentials'
    );
    await expect(page).toHaveURL('/login');
  });

  test('complete checkout flow', async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'Password123!');
    await page.click('[data-testid="login-button"]');

    // Add item to cart
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart-1"]');

    // Checkout
    await page.click('[data-testid="cart-icon"]');
    await page.click('[data-testid="checkout-button"]');

    // Fill shipping
    await page.fill('[data-testid="address"]', '123 Main St');
    await page.click('[data-testid="place-order"]');

    // Verify order confirmation
    await expect(page.locator('.order-confirmation')).toBeVisible();
    await expect(page.locator('.order-number')).toContainText(/ORD-\d+/);
  });
});
```

---

## Cypress Example

```javascript
// cypress/e2e/signup.cy.js
describe('User Signup', () => {
  beforeEach(() => {
    cy.visit('/signup');
  });

  it('creates a new account', () => {
    cy.get('[data-testid="name"]').type('Alice Smith');
    cy.get('[data-testid="email"]').type('alice@example.com');
    cy.get('[data-testid="password"]').type('SecurePass123!');
    cy.get('[data-testid="confirm-password"]').type('SecurePass123!');

    cy.get('[data-testid="signup-button"]').click();

    // Should redirect and show welcome
    cy.url().should('include', '/dashboard');
    cy.contains('Welcome, Alice').should('be.visible');
  });

  it('shows validation errors for weak password', () => {
    cy.get('[data-testid="password"]').type('123');
    cy.get('[data-testid="signup-button"]').click();

    cy.contains('Password must be at least 8 characters').should('be.visible');
  });
});
```

---

## E2E Tests in CI Pipeline

```yaml
# GitHub Actions — Playwright E2E tests
name: E2E Tests
on:
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      # Install browsers
      - run: npx playwright install --with-deps

      # Start application in background
      - name: Start app
        run: npm run start &
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/testdb
          NODE_ENV: test

      # Wait for app to be ready
      - name: Wait for app
        run: npx wait-on http://localhost:3000 --timeout 30000

      # Run E2E tests
      - run: npx playwright test

      # Upload test artifacts on failure
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7

      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: test-screenshots
          path: test-results/
```

---

## Visual Regression Testing

```
  ┌──────────────────────────────────────────────────┐
  │          VISUAL REGRESSION TESTING               │
  │                                                  │
  │  Baseline Screenshot    Current Screenshot       │
  │  ┌────────────────┐    ┌────────────────┐       │
  │  │  ┌──────────┐  │    │  ┌──────────┐  │       │
  │  │  │  Button   │  │    │  │  Button   │  │       │
  │  │  │  [Blue]   │  │    │  │  [RED]    │  │  ← !  │
  │  │  └──────────┘  │    │  └──────────┘  │       │
  │  │  Welcome text  │    │  Welcome text  │       │
  │  └────────────────┘    └────────────────┘       │
  │                                                  │
  │  Diff: Button color changed!                     │
  │  Action: FAIL (unexpected visual change)         │
  └──────────────────────────────────────────────────┘
```

```typescript
// Playwright visual comparison
test('homepage looks correct', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100,
  });
});
```

---

## Managing Test Data

```
  ┌──────────────────────────────────────────────────┐
  │         E2E TEST DATA STRATEGIES                 │
  │                                                  │
  │  1. SEED BEFORE TESTS                            │
  │     ► Load known dataset before each test        │
  │     ► Predictable, repeatable                    │
  │                                                  │
  │  2. API-DRIVEN SETUP                             │
  │     ► Create data via API before test            │
  │     ► Tests are independent & self-contained     │
  │                                                  │
  │  3. DATABASE SNAPSHOTS                           │
  │     ► Restore DB snapshot before each suite      │
  │     ► Fast reset, consistent state               │
  │                                                  │
  │  ⚠ AVOID:                                       │
  │     ✕ Sharing data between tests                 │
  │     ✕ Depending on test execution order          │
  │     ✕ Using production data                      │
  └──────────────────────────────────────────────────┘
```

---

## Dealing with Flaky E2E Tests

```
  FLAKY TEST STRATEGIES
  ═════════════════════

  1. Auto-Wait (Playwright)
     ✓ page.click() waits for element to be actionable
     ✕ Don't use: await sleep(5000)

  2. Retry on Failure
     ✓ playwright.config.ts: retries: 2
     ✓ Cypress: retries: { runMode: 2 }

  3. Test Isolation
     ✓ Each test starts with known state
     ✕ Don't rely on test A to set up for test B

  4. Quarantine Flaky Tests
     ✓ Tag flaky tests, track them, fix or remove
     ✕ Don't ignore them indefinitely

  5. Network Stubbing
     ✓ Mock external API calls in E2E
     ✕ Don't test third-party APIs in your E2E
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Tests timeout in CI | App not ready when tests start | Use `wait-on` or health-check polling |
| Different results per browser | Browser-specific rendering | Run tests across all target browsers |
| Screenshots differ in CI vs local | Font rendering, viewport | Set fixed viewport; use Docker for consistency |
| Element not found | Race condition / async loading | Use auto-wait; avoid `sleep()`; use proper selectors |
| Tests take 30+ minutes | Too many E2E tests | Parallelize; reduce count; move some to integration |
| Auth issues in CI | Cookies/tokens not persisted | Use `storageState` to save/restore auth |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Verify complete user journeys end-to-end |
| **Scope** | Full stack: browser + frontend + backend + database |
| **Speed** | Seconds to minutes per test |
| **Tools** | Playwright, Cypress, Selenium |
| **Runs** | Before deployment; on PR to main |
| **Key challenge** | Flakiness (manage with auto-wait, retries, isolation) |

---

## Quick Revision Questions

1. **What is an E2E test?** *A test that simulates real user interaction through the entire application stack — browser, frontend, backend, and database.*
2. **Why are E2E tests at the top of the test pyramid?** *They are the slowest, most expensive, and most prone to flakiness, so you should have the fewest.*
3. **What is Playwright?** *A modern E2E testing framework by Microsoft that supports Chrome, Firefox, and Safari with auto-wait and multi-browser testing.*
4. **How do you handle flaky E2E tests?** *Use auto-wait instead of sleep, retry on failure, ensure test isolation, quarantine persistently flaky tests.*
5. **What is visual regression testing?** *Comparing screenshots of the current UI against known-good baseline images to detect unexpected visual changes.*
6. **What artifacts should you save when E2E tests fail in CI?** *Screenshots, videos, trace files, and HTML reports for debugging after the CI run completes.*

---

[← Previous: Integration Testing](03-integration-testing.md) | [Back to README](../README.md) | [Next: Performance Testing →](05-performance-testing.md)
