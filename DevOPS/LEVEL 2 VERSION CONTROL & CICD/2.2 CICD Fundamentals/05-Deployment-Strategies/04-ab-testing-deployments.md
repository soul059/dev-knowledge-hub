# Chapter 5.4: A/B Testing Deployments

[← Previous: Canary Deployments](03-canary-deployments.md) | [Back to README](../README.md) | [Next: Feature Flags →](05-feature-flags.md)

---

## Overview

**A/B testing deployments** route specific user segments to different application versions to measure the impact of changes on business metrics. Unlike canary deployments (which focus on technical stability), A/B testing focuses on **business outcomes** — conversion rates, revenue, user engagement.

---

## A/B Testing vs Canary

```
  CANARY DEPLOYMENT              A/B TESTING DEPLOYMENT
  ═════════════════              ══════════════════════

  Goal: Technical stability      Goal: Business impact
  Split: Random % of traffic     Split: Targeted user segments
  Metric: Error rate, latency    Metric: Conversion, revenue
  Duration: Minutes to hours     Duration: Days to weeks
  Decision: Promote or rollback  Decision: Which variant wins

  ┌────────┐                     ┌────────┐
  │  5%    │──► v2 (new code)    │ Group A│──► Version A (blue button)
  │ random │                     │ 50%    │
  │traffic │                     │ users  │
  └────────┘                     └────────┘
  ┌────────┐                     ┌────────┐
  │  95%   │──► v1 (stable)      │ Group B│──► Version B (green button)
  │ random │                     │ 50%    │
  │traffic │                     │ users  │
  └────────┘                     └────────┘
```

---

## How A/B Testing Works

```
  ┌──────────────────────────────────────────────────────┐
  │              A/B TESTING FLOW                        │
  │                                                      │
  │  User Request                                        │
  │       │                                              │
  │       ▼                                              │
  │  ┌────────────┐                                      │
  │  │ Router /   │  Segment by:                         │
  │  │ Feature    │  • User ID hash                      │
  │  │ Flag Svc   │  • Geography                         │
  │  │            │  • Device type                       │
  │  └─────┬──────┘  • User attributes                  │
  │    A   │   B                                         │
  │  ┌─────▼──┐ ┌──▼─────┐                              │
  │  │Version │ │Version │                               │
  │  │   A    │ │   B    │                               │
  │  └───┬────┘ └───┬────┘                               │
  │      │          │                                    │
  │      ▼          ▼                                    │
  │  ┌────────────────────┐                              │
  │  │ Analytics Platform │  Track: conversions,         │
  │  │ (Amplitude, GA,    │  clicks, revenue,            │
  │  │  Mixpanel)         │  engagement per variant      │
  │  └────────────────────┘                              │
  │                                                      │
  │  After sufficient data:                              │
  │  ► Statistical significance reached?                 │
  │  ► Winning variant promoted to 100%                  │
  └──────────────────────────────────────────────────────┘
```

---

## Istio-Based A/B Testing

```yaml
# Route by HTTP header (user segment)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts: ["myapp.example.com"]
  http:
    # Premium users → Version B (new checkout)
    - match:
        - headers:
            x-user-tier:
              exact: "premium"
      route:
        - destination:
            host: myapp
            subset: version-b

    # US users → Version A (original checkout)
    - match:
        - headers:
            x-user-country:
              exact: "US"
      route:
        - destination:
            host: myapp
            subset: version-a

    # Default → Version A
    - route:
        - destination:
            host: myapp
            subset: version-a
```

---

## Feature Flag-Based A/B Testing

```javascript
// Using LaunchDarkly for A/B testing
const LaunchDarkly = require('launchdarkly-node-server-sdk');

const client = LaunchDarkly.init(process.env.LD_SDK_KEY);

app.get('/checkout', async (req, res) => {
  const user = {
    key: req.user.id,
    custom: {
      plan: req.user.plan,        // premium, free
      country: req.user.country,  // US, UK, etc.
      signupDate: req.user.createdAt,
    },
  };

  // Feature flag determines which checkout to show
  const variant = await client.variation('new-checkout-flow', user, 'control');

  if (variant === 'treatment') {
    res.render('checkout-v2', { variant: 'treatment' });
  } else {
    res.render('checkout-v1', { variant: 'control' });
  }

  // Track event for analytics
  client.track('checkout-page-viewed', user, { variant });
});

// Track conversion
app.post('/purchase', async (req, res) => {
  const user = { key: req.user.id };
  client.track('purchase-completed', user, {
    revenue: req.body.total,
  });
  // ... process purchase
});
```

---

## Statistical Significance

```
  STATISTICAL SIGNIFICANCE
  ════════════════════════

  Variant A (Control):     1,000 visitors → 32 conversions  = 3.2%
  Variant B (Treatment):   1,000 visitors → 45 conversions  = 4.5%

  Is this difference REAL or just noise?

  ┌──────────────────────────────────────────────┐
  │  p-value = 0.03  (< 0.05 threshold)         │
  │  Confidence: 97%                             │
  │  Result: STATISTICALLY SIGNIFICANT           │
  │                                              │
  │  Variant B wins! (+40% relative improvement) │
  │  ► Promote Variant B to 100%                 │
  └──────────────────────────────────────────────┘

  ⚠ Common mistakes:
  ✕ Stopping test too early (peeking)
  ✕ Not enough sample size
  ✕ Testing too many variants at once
  ✓ Pre-calculate required sample size
  ✓ Run until significance OR max duration
```

---

## Nginx-Based A/B Testing

```nginx
# nginx.conf — Split traffic by cookie
upstream version_a {
    server app-v1:8080;
}

upstream version_b {
    server app-v2:8080;
}

split_clients "${remote_addr}" $variant {
    50% version_a;
    50% version_b;
}

server {
    listen 80;

    location / {
        # Set sticky cookie so user stays on same variant
        add_header Set-Cookie "ab_variant=$variant; Path=/; Max-Age=86400";

        # Route based on existing cookie or new assignment
        if ($cookie_ab_variant) {
            set $variant $cookie_ab_variant;
        }

        proxy_pass http://$variant;
    }
}
```

---

## CI/CD Pipeline with A/B Testing

```yaml
# GitHub Actions — Deploy A/B test
name: A/B Test Deploy
on:
  workflow_dispatch:
    inputs:
      experiment_name:
        description: 'Experiment name'
        required: true
      traffic_percentage:
        description: 'Traffic to variant B (%)'
        required: true
        default: '50'

jobs:
  deploy-ab:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build variant B
        run: |
          docker build -t myapp:variant-b-${{ github.sha }} .
          docker push registry.example.com/myapp:variant-b-${{ github.sha }}

      - name: Deploy variant B
        run: |
          kubectl set image deployment/myapp-variant-b \
            myapp=registry.example.com/myapp:variant-b-${{ github.sha }}

      - name: Configure traffic split
        run: |
          kubectl apply -f - <<EOF
          apiVersion: networking.istio.io/v1beta1
          kind: VirtualService
          metadata:
            name: myapp
          spec:
            hosts: ["myapp.example.com"]
            http:
              - route:
                - destination:
                    host: myapp-control
                  weight: $((100 - ${{ github.event.inputs.traffic_percentage }}))
                - destination:
                    host: myapp-variant-b
                  weight: ${{ github.event.inputs.traffic_percentage }}
          EOF

      - name: Register experiment
        run: |
          curl -X POST https://analytics.example.com/experiments \
            -d '{"name": "${{ github.event.inputs.experiment_name }}", "variant_b_traffic": ${{ github.event.inputs.traffic_percentage }}}'
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Results inconclusive | Not enough traffic/time | Calculate sample size upfront; run longer |
| User sees both variants | No sticky session/cookie | Use consistent hashing or cookie-based routing |
| Variant B shows higher errors | New code has bugs | Combine A/B testing with canary monitoring |
| Multiple experiments conflict | Users in overlapping tests | Use mutual exclusion groups in feature flag tool |
| Peeking bias | Checking significance too early | Pre-define stopping rules; use sequential testing |
| Carryover effects | Users remember old experience | Use new-user-only segments for UI experiments |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Measure business impact of changes on user segments |
| **Focus** | Conversion, revenue, engagement (business metrics) |
| **Duration** | Days to weeks (needs statistical significance) |
| **Routing** | By user attributes, cookies, headers, user ID hash |
| **Tools** | LaunchDarkly, Optimizely, Istio, Nginx split_clients |
| **Key concept** | Statistical significance before making decisions |

---

## Quick Revision Questions

1. **How does A/B testing differ from canary deployment?** *A/B testing measures business outcomes (conversion, revenue) across user segments over days/weeks; canary measures technical stability (errors, latency) over minutes/hours.*
2. **What is statistical significance in A/B testing?** *Confidence that the observed difference between variants is real and not due to random chance (typically p-value < 0.05).*
3. **How do you ensure users see consistent variants?** *Use sticky sessions via cookies or consistent hashing based on user ID.*
4. **What is "peeking" in A/B testing and why is it bad?** *Checking results before sufficient data is collected, leading to premature conclusions based on noise.*
5. **Name two tools used for A/B testing.** *LaunchDarkly and Optimizely (feature flag platforms); Istio (infrastructure-level traffic splitting).*
6. **What routing methods can A/B testing use?** *HTTP headers, cookies, user ID hash, geography, device type, or user attributes like subscription tier.*

---

[← Previous: Canary Deployments](03-canary-deployments.md) | [Back to README](../README.md) | [Next: Feature Flags →](05-feature-flags.md)
