# Chapter 7.4: Feature Flags

[← Previous: Canarying](03-canarying.md) | [Back to README](../README.md) | [Next: Rollback Strategies →](05-rollback-strategies.md)

---

## Overview

Feature flags (also called feature toggles) decouple deployment from release. Code is deployed to production but new features are hidden behind flags that can be enabled or disabled without redeploying. This enables safer releases, A/B testing, gradual rollouts, and instant kill switches for problematic features.

---

## 1. Deploy vs Release

```
  ┌──────────────────────────────────────────────────────────┐
  │  WITHOUT FEATURE FLAGS:                                  │
  │  Deploy = Release (same moment, all-or-nothing)          │
  │                                                          │
  │  ──────────▶ Deploy v2.0 ───▶ ALL users get feature     │
  │                                                          │
  │  ═══════════════════════════════════════════════════════  │
  │                                                          │
  │  WITH FEATURE FLAGS:                                     │
  │  Deploy ≠ Release (decoupled, independent)               │
  │                                                          │
  │  ──▶ Deploy v2.0 (flag OFF) ──▶ No users see feature    │
  │                                                          │
  │  ──▶ Enable flag for 5% ──▶ 5% of users see feature    │
  │                                                          │
  │  ──▶ Enable flag for all ──▶ 100% see feature           │
  │                                                          │
  │  ──▶ Disable flag (instant) ──▶ Feature hidden again    │
  │                                                          │
  │  KEY: The feature can be turned off in seconds          │
  │  without a code deployment or rollback                   │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Types of Feature Flags

```
  ┌──────────────────────────────────────────────────────────┐
  │  TYPE              │ LIFESPAN  │ PURPOSE                  │
  ├────────────────────┼───────────┼──────────────────────────┤
  │ Release toggle     │ Days-weeks│ Control feature rollout   │
  │                    │           │ "New checkout enabled?"  │
  │                    │           │                          │
  │ Experiment toggle  │ Weeks     │ A/B testing              │
  │                    │           │ "Show layout A or B?"    │
  │                    │           │                          │
  │ Ops toggle         │ Permanent │ Operational control      │
  │ (kill switch)      │           │ "Disable recommendations │
  │                    │           │  under high load?"       │
  │                    │           │                          │
  │ Permission toggle  │ Permanent │ Feature access control   │
  │                    │           │ "Premium feature for     │
  │                    │           │  paying users only?"     │
  │                    │           │                          │
  │ Development toggle │ Days      │ Trunk-based development  │
  │                    │           │ "Hide WIP feature from   │
  │                    │           │  prod while developing?" │
  └────────────────────┴───────────┴──────────────────────────┘
```

---

## 3. Feature Flag Implementation

```python
# Simple feature flag implementation
class FeatureFlags:
    """Feature flag service with percentage-based rollout."""
    
    def __init__(self, config_source):
        self.flags = config_source.load()
    
    def is_enabled(self, flag_name, user_id=None, context=None):
        flag = self.flags.get(flag_name)
        if not flag or not flag['enabled']:
            return False
        
        # Check targeting rules
        if flag.get('user_ids') and user_id in flag['user_ids']:
            return True  # Explicitly enabled for this user
        
        # Percentage rollout (deterministic per user)
        if flag.get('percentage'):
            # Hash ensures same user always gets same result
            hash_val = hash(f"{flag_name}:{user_id}") % 100
            return hash_val < flag['percentage']
        
        return flag['enabled']

# Usage in application code
flags = FeatureFlags(config)

def handle_checkout(request):
    if flags.is_enabled("new_checkout_flow", user_id=request.user_id):
        return new_checkout(request)    # New path
    else:
        return legacy_checkout(request)  # Old path
```

```yaml
# Feature flag configuration
feature_flags:
  new_checkout_flow:
    enabled: true
    percentage: 25           # 25% of users
    description: "New streamlined checkout experience"
    owner: "checkout-team"
    created: "2024-11-15"
    expires: "2025-01-15"    # Must be cleaned up by this date
    
  ai_recommendations:
    enabled: true
    percentage: 100          # All users
    kill_switch: true        # Can be disabled instantly
    owner: "ml-team"
    
  dark_mode:
    enabled: true
    user_ids:                # Beta testers only
      - "user-001"
      - "user-042"
      - "user-108"
    percentage: 0            # Not for general population yet
    
  heavy_report_generation:
    enabled: true
    percentage: 100
    ops_toggle: true         # Disable under high load
    disable_conditions:
      - metric: "cpu_usage"
        threshold: 80        # Auto-disable if CPU > 80%
```

---

## 4. Feature Flag Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  FEATURE FLAG SYSTEM ARCHITECTURE                        │
  │                                                          │
  │  ┌─────────────────────────────┐                         │
  │  │   Flag Management UI       │  (dashboard for PMs,    │
  │  │   • Create/edit flags      │   engineers, ops)       │
  │  │   • Set targeting rules    │                         │
  │  │   • View flag status       │                         │
  │  └──────────┬──────────────────┘                         │
  │             │                                            │
  │             ▼                                            │
  │  ┌─────────────────────────────┐                         │
  │  │   Flag Configuration Store │  (database/config)      │
  │  │   • Flag definitions       │                         │
  │  │   • Targeting rules        │                         │
  │  │   • Audit log              │                         │
  │  └──────────┬──────────────────┘                         │
  │             │ push/poll                                  │
  │             ▼                                            │
  │  ┌─────────────────────────────┐                         │
  │  │   Application SDK          │  (embedded in each      │
  │  │   • Local cache of flags   │   service)              │
  │  │   • Evaluation engine      │                         │
  │  │   • Usage tracking         │                         │
  │  └─────────────────────────────┘                         │
  │                                                          │
  │  Tools: LaunchDarkly, Unleash, Flagsmith, Split,        │
  │         OpenFeature, DIY (config files + reload)         │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Feature Flag Best Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │  DO                           │ DON'T                    │
  ├───────────────────────────────┼──────────────────────────┤
  │ ✓ Set expiration dates        │ ✗ Leave flags forever    │
  │   (clean up after rollout)    │   (tech debt accumulates)│
  │                               │                          │
  │ ✓ Test BOTH paths             │ ✗ Only test flag-on      │
  │   (flag on AND flag off)      │   (flag-off path rots)   │
  │                               │                          │
  │ ✓ Use deterministic hashing   │ ✗ Use random()           │
  │   (consistent per user)       │   (inconsistent UX)      │
  │                               │                          │
  │ ✓ Log flag evaluations        │ ✗ No audit trail         │
  │   (who saw what)              │   (can't debug issues)   │
  │                               │                          │
  │ ✓ Keep flags simple           │ ✗ Nest flags in flags    │
  │   (one flag = one feature)    │   (combinatorial         │
  │                               │    explosion)            │
  │                               │                          │
  │ ✓ Have a flag naming          │ ✗ Inconsistent naming    │
  │   convention                  │   (team.feature.variant) │
  │                               │                          │
  │ ✓ Default to OFF for new      │ ✗ Default to ON          │
  │   flags (safe by default)     │   (surprise features)    │
  └───────────────────────────────┴──────────────────────────┘
  
  FLAG LIFECYCLE:
  Create → Test (off) → Enable (%) → Roll out (100%) → 
  Remove flag + dead code (CRITICAL — clean up!)
```

---

## 6. Ops Kill Switches

```
  ┌──────────────────────────────────────────────────────────┐
  │  KILL SWITCHES FOR RELIABILITY                           │
  │                                                          │
  │  Scenario: System under extreme load                     │
  │                                                          │
  │  ┌─────────────────────────────┐                         │
  │  │  Normal Operation          │                         │
  │  │  ├─ Search: full results   │                         │
  │  │  ├─ Recommendations: ML    │                         │
  │  │  ├─ Analytics: real-time   │                         │
  │  │  └─ Images: high-res      │                         │
  │  └─────────────────────────────┘                         │
  │                                                          │
  │  ┌─────────────────────────────┐                         │
  │  │  Degraded Mode (flags OFF) │ ← instant toggle!       │
  │  │  ├─ Search: basic results  │ (no deploy needed)      │
  │  │  ├─ Recommendations: OFF   │                         │
  │  │  ├─ Analytics: queued      │                         │
  │  │  └─ Images: compressed    │                         │
  │  └─────────────────────────────┘                         │
  │                                                          │
  │  Kill switches let you shed load in seconds              │
  │  by disabling non-essential features.                    │
  │  Core user journey remains functional.                   │
  │                                                          │
  │  Example kill switch config:                             │
  │  graceful_degradation:                                   │
  │    level_1: (CPU > 70%)                                  │
  │      disable: [analytics, recommendations]               │
  │    level_2: (CPU > 85%)                                  │
  │      disable: [search_suggestions, image_processing]     │
  │    level_3: (CPU > 95%)                                  │
  │      disable: [everything except core checkout]           │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Feature Flag Saves Black Friday

```
  SITUATION: E-commerce company, Black Friday 2024
  
  10:00 — New "AI-powered product search" launched 2 days ago
           via feature flag (100% rollout)
  
  10:30 — Traffic hits 8× normal levels
           AI search service response time: 200ms → 1,500ms
           ML inference pod CPU: 95%
           Overall site latency increasing
  
  10:35 — SRE notices cascade risk
           AI search is consuming 40% of compute resources
           Core checkout starting to slow down
  
  10:36 — ACTION: Disable "ai_search" feature flag
           One click in LaunchDarkly dashboard
           
  10:36 — (30 seconds later)
           All users fall back to traditional keyword search
           ML inference pods stop receiving requests
           CPU drops from 95% → 20%
           Site latency returns to normal
  
  10:37 — Core shopping experience fully functional
           No deployment. No rollback. No downtime.
           Revenue protected: estimated $2M saved
  
  POST-EVENT:
  ├─ AI search re-enabled at 10% after Black Friday
  ├─ ML team scaled inference pods 5× for holiday traffic
  ├─ Gradually re-enabled to 100% over 3 days
  └─ Kill switch saved ~$2M in potential lost sales
  
  WITHOUT feature flag: Would need emergency deployment
  rollback (~15 minutes) or manual code change (~30+ min).
  With flag: 30 seconds, one click.
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Too many feature flags, can't manage them" | Enforce expiration dates. Run quarterly cleanup sprints. Track flag count as a team metric. |
| "Flag-off path doesn't work anymore" | Test both paths in CI. Include flag-off in integration tests. Run periodic flag-off validation. |
| "Feature flags add latency" | Use local SDK caching (evaluate locally, not API call per request). Sync flags every 30-60 seconds. |
| "Can't debug issues — don't know which flags were active" | Log flag evaluations with request context. Include active flags in error reports and traces. |

---

## Summary Table

| Flag Type | Lifespan | Owner | Usage |
|-----------|----------|-------|-------|
| **Release** | Days-weeks | Dev team | Gradual feature rollout |
| **Experiment** | Weeks | Product/PM | A/B testing |
| **Ops (kill switch)** | Permanent | SRE | Emergency feature disable |
| **Permission** | Permanent | Product | Entitlement-based features |
| **Development** | Days | Dev team | Hide WIP from production |

---

## Quick Revision Questions

1. **How do feature flags decouple deployment from release?**
   > Code is deployed with the feature behind a flag (defaulting to OFF). The feature is "released" by enabling the flag for users — independently of code deployment, and reversible without a rollback.

2. **What is an ops kill switch?**
   > A permanent feature flag that allows SRE to instantly disable non-essential features during incidents or high load, preserving the core user experience without requiring a deployment.

3. **Why is deterministic hashing important in feature flags?**
   > Using `hash(flag_name + user_id) % 100` ensures the same user always sees the same variant. Using random() would give inconsistent experiences — a user might see the new feature on one request and not the next.

4. **What's the biggest risk of feature flags?**
   > Technical debt from unremoved flags. Over time, dead code accumulates behind old flags, making the codebase harder to understand and test. Always set expiration dates and clean up after full rollout.

5. **How do feature flags improve incident response?**
   > Problematic features can be disabled in seconds via flag toggle, without rolling back the entire deployment. This is faster and less risky than a full rollback.

6. **Name three feature flag tools.**
   > LaunchDarkly (SaaS, enterprise), Unleash (open-source), Flagsmith (open-source), Split.io (SaaS), and OpenFeature (vendor-neutral standard).

---

[← Previous: Canarying](03-canarying.md) | [Back to README](../README.md) | [Next: Rollback Strategies →](05-rollback-strategies.md)
