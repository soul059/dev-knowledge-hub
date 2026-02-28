# Chapter 5.5: Feature Flags

[← Previous: A/B Testing Deployments](04-ab-testing-deployments.md) | [Back to README](../README.md) | [Next: Rollback Strategies →](06-rollback-strategies.md)

---

## Overview

**Feature flags** (also called feature toggles) decouple deployment from release. Code is deployed to production but new features are hidden behind toggles that can be turned on/off without redeploying. This enables trunk-based development, gradual rollouts, and instant kill switches.

---

## Deploy vs Release

```
  WITHOUT FEATURE FLAGS           WITH FEATURE FLAGS
  ═════════════════════           ═══════════════════

  Deploy = Release                Deploy ≠ Release
  ┌──────────┐                    ┌──────────┐ ┌──────────┐
  │ Deploy   │──► Feature live    │ Deploy   │ │ Release  │
  │ v2 code  │    to all users    │ v2 code  │ │ (toggle) │
  └──────────┘                    └──────────┘ └──────────┘
                                       │             │
  Risk: All users get                  │        Enable flag
  new feature at once         Code is in prod    for 5% users
                              but OFF by default
                                                     │
                                              Gradual rollout
                                              10% → 50% → 100%
```

---

## Types of Feature Flags

```
  ┌──────────────────────────────────────────────────────┐
  │           FEATURE FLAG CATEGORIES                    │
  │                                                      │
  │  RELEASE TOGGLES (short-lived)                       │
  │  ├── Enable new feature for % of users               │
  │  ├── Kill switch if something goes wrong             │
  │  └── Remove after feature is fully rolled out        │
  │                                                      │
  │  EXPERIMENT TOGGLES (medium-lived)                   │
  │  ├── A/B testing different UI variants               │
  │  ├── Measure business impact                         │
  │  └── Remove after experiment concludes               │
  │                                                      │
  │  OPS TOGGLES (long-lived)                            │
  │  ├── Circuit breakers for degraded mode              │
  │  ├── Maintenance mode switches                       │
  │  └── May live permanently                            │
  │                                                      │
  │  PERMISSION TOGGLES (long-lived)                     │
  │  ├── Premium features for paying users               │
  │  ├── Beta features for early adopters                │
  │  └── Part of the business model                      │
  └──────────────────────────────────────────────────────┘
```

---

## Implementation Patterns

### Simple Boolean Flag

```javascript
// Basic feature flag
const features = {
  newCheckout: process.env.FEATURE_NEW_CHECKOUT === 'true',
  darkMode: process.env.FEATURE_DARK_MODE === 'true',
};

app.get('/checkout', (req, res) => {
  if (features.newCheckout) {
    res.render('checkout-v2');
  } else {
    res.render('checkout-v1');
  }
});
```

### Feature Flag Service (LaunchDarkly)

```javascript
const LaunchDarkly = require('launchdarkly-node-server-sdk');
const client = LaunchDarkly.init(process.env.LD_SDK_KEY);

app.get('/dashboard', async (req, res) => {
  const user = {
    key: req.user.id,
    email: req.user.email,
    custom: {
      plan: req.user.plan,
      country: req.user.country,
    },
  };

  const showNewDashboard = await client.variation(
    'new-dashboard',    // Flag key
    user,               // User context
    false               // Default value
  );

  if (showNewDashboard) {
    res.render('dashboard-v2');
  } else {
    res.render('dashboard-v1');
  }
});
```

### Python (Feature Flags with Unleash)

```python
from UnleashClient import UnleashClient

client = UnleashClient(
    url="https://unleash.example.com/api",
    app_name="myapp",
    custom_headers={"Authorization": UNLEASH_API_KEY}
)
client.initialize_client()

@app.route('/api/search')
def search():
    if client.is_enabled("new-search-algorithm", {"userId": current_user.id}):
        return new_search(request.args.get('q'))
    else:
        return legacy_search(request.args.get('q'))
```

---

## Feature Flag Platforms

| Platform | Type | Free Tier | Best For |
|---|---|---|---|
| **LaunchDarkly** | SaaS | No | Enterprise, full featured |
| **Unleash** | Open Source + SaaS | Yes (OSS) | Self-hosted flexibility |
| **Flagsmith** | Open Source + SaaS | Yes | OSS + remote config |
| **Split.io** | SaaS | Free tier | Experimentation focus |
| **ConfigCat** | SaaS | Free tier | Simple flag management |
| **Environment vars** | DIY | Free | Simple on/off toggles |

---

## Gradual Rollout Strategy

```
  GRADUAL ROLLOUT WITH FEATURE FLAGS
  ═══════════════════════════════════

  Day 1:  Internal team (1%)
          ┌█░░░░░░░░░░░░░░░░░░┐

  Day 3:  Beta users (5%)
          ┌██░░░░░░░░░░░░░░░░░┐

  Day 5:  10% of all users
          ┌████░░░░░░░░░░░░░░░┐

  Day 7:  25% of all users
          ┌████████░░░░░░░░░░░┐

  Day 10: 50% of all users
          ┌███████████░░░░░░░░┐

  Day 14: 100% — fully launched!
          ┌████████████████████┐

  At any point: set to 0% = instant kill switch
```

---

## Feature Flags in CI/CD Pipeline

```yaml
# GitHub Actions — Deploy with feature flag
name: Deploy with Feature Flag
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Deploy code (with new feature behind flag)
      - name: Build and deploy
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker push registry.example.com/myapp:${{ github.sha }}
          kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}

      # Feature is deployed but OFF by default
      # No need to change feature flag here

  # Separate workflow to enable feature
  enable-feature:
    runs-on: ubuntu-latest
    needs: deploy
    if: github.event.inputs.enable_feature == 'true'
    steps:
      - name: Enable feature flag (5%)
        run: |
          curl -X PATCH https://app.launchdarkly.com/api/v2/flags/myproject/new-checkout \
            -H "Authorization: ${{ secrets.LD_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '[{
              "op": "replace",
              "path": "/environments/production/on",
              "value": true
            }]'
```

---

## Best Practices

```
  ┌──────────────────────────────────────────────────────┐
  │          FEATURE FLAG BEST PRACTICES                 │
  │                                                      │
  │  1. SHORT LIFECYCLE                                  │
  │     Remove flags after full rollout (< 30 days)      │
  │     Track flag age; alert on stale flags              │
  │                                                      │
  │  2. NAMING CONVENTION                                │
  │     ✓ enable-new-checkout-flow                       │
  │     ✓ experiment-search-algorithm-v2                 │
  │     ✕ flag1, test, myFeature                         │
  │                                                      │
  │  3. DEFAULT TO OFF                                   │
  │     New features start disabled in production        │
  │     Explicit activation required                     │
  │                                                      │
  │  4. TEST BOTH PATHS                                  │
  │     Unit test with flag ON and OFF                   │
  │     CI tests run both variants                       │
  │                                                      │
  │  5. DOCUMENTATION                                    │
  │     Each flag has: owner, purpose, expected removal  │
  │     Track in feature flag dashboard                  │
  │                                                      │
  │  6. FLAG DEBT CLEANUP                                │
  │     Schedule regular cleanup sprints                 │
  │     Remove dead code paths after rollout             │
  └──────────────────────────────────────────────────────┘
```

---

## Testing with Feature Flags

```javascript
// Test both code paths
describe('Checkout', () => {
  test('renders v1 checkout when flag is off', async () => {
    // Mock flag as OFF
    jest.spyOn(featureFlags, 'isEnabled')
      .mockResolvedValue(false);

    const { getByText } = render(<Checkout />);
    expect(getByText('Classic Checkout')).toBeTruthy();
  });

  test('renders v2 checkout when flag is on', async () => {
    // Mock flag as ON
    jest.spyOn(featureFlags, 'isEnabled')
      .mockResolvedValue(true);

    const { getByText } = render(<Checkout />);
    expect(getByText('New Checkout Experience')).toBeTruthy();
  });
});
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Flag evaluated before client init | SDK not ready | Use `await client.waitForInitialization()` |
| Too many stale flags | No cleanup process | Track flag age; alert at 30 days; schedule cleanup |
| Flag state inconsistent | Caching issues | Use streaming updates; set appropriate TTL |
| Performance impact | Too many flag evaluations | Cache flag decisions per request; use local evaluation |
| Testing complexity | Both paths need testing | Create test helpers for flag mocking |
| Flag spaghetti | Flags referencing other flags | Keep flags independent; avoid nested conditionals |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Decouple deployment from release; gradual rollouts |
| **Types** | Release, Experiment, Ops, Permission |
| **Lifecycle** | Release flags: remove in < 30 days |
| **Tools** | LaunchDarkly, Unleash, Flagsmith, env vars |
| **Testing** | Test both ON and OFF paths |
| **Key benefit** | Deploy anytime, release when ready, instant kill switch |

---

## Quick Revision Questions

1. **What is a feature flag?** *A mechanism to enable/disable features in production without redeploying code — decoupling deploy from release.*
2. **What are the four types of feature flags?** *Release toggles (short-lived), experiment toggles (A/B tests), ops toggles (circuit breakers), and permission toggles (premium features).*
3. **Why should release flags be removed quickly?** *To prevent "flag debt" — accumulating dead code paths, testing complexity, and maintenance burden.*
4. **How do feature flags enable trunk-based development?** *Developers commit to main behind flags; incomplete features are merged but hidden from users.*
5. **Name two feature flag platforms.** *LaunchDarkly (commercial SaaS) and Unleash (open-source + SaaS).*
6. **What is a kill switch?** *A feature flag that can instantly disable a feature in production if problems are detected, without requiring a redeployment.*

---

[← Previous: A/B Testing Deployments](04-ab-testing-deployments.md) | [Back to README](../README.md) | [Next: Rollback Strategies →](06-rollback-strategies.md)
