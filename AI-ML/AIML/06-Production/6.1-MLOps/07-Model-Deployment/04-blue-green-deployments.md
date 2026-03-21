# 🔵🟢 Blue-Green Deployments for ML Models

> **Unit 7 · Chapter 4** — Zero-downtime deployment using two identical production environments with instant traffic switching.

---

## 📋 Chapter Overview

A **blue-green deployment** maintains two identical production environments. At any time, one ("blue") serves all live traffic while the other ("green") stands by with the new model version. Deployment is an **instant switch** at the load balancer, and rollback is just as fast — switch back.

This chapter covers:

- Blue-green architecture for ML serving
- Zero-downtime switching mechanics
- Health checks and validation before switching
- Comparison with canary deployments
- Cost implications and when to use blue-green

---

## 🏗️ Blue-Green Architecture

```
                         ┌────────────────────────┐
                         │     Load Balancer /     │
                         │     API Gateway         │
                         │                         │
                         │   Active ──► 🔵 BLUE    │
                         │   (switch is instant)    │
                         └────────────┬─────────────┘
                                      │
                     ┌────────────────┼────────────────┐
                     │                                 │
                     ▼                                 ▼
        ┌───────────────────────┐        ┌───────────────────────┐
        │    🔵 BLUE ENV        │        │    🟢 GREEN ENV       │
        │    ─────────────      │        │    ─────────────      │
        │    Model: v1          │        │    Model: v2          │
        │    Status: ACTIVE     │        │    Status: STANDBY    │
        │                       │        │                       │
        │  ┌─────────────────┐  │        │  ┌─────────────────┐  │
        │  │ Model Server    │  │        │  │ Model Server    │  │
        │  │ (3 replicas)    │  │        │  │ (3 replicas)    │  │
        │  └─────────────────┘  │        │  └─────────────────┘  │
        │  ┌─────────────────┐  │        │  ┌─────────────────┐  │
        │  │ Feature Store   │  │        │  │ Feature Store   │  │
        │  │ Cache           │  │        │  │ Cache           │  │
        │  └─────────────────┘  │        │  └─────────────────┘  │
        └───────────────────────┘        └───────────────────────┘
```

---

## 🔄 Deployment Lifecycle

```
  Step 1: INITIAL STATE                Step 2: DEPLOY TO GREEN
  ┌─────┐     ┌─────┐                 ┌─────┐     ┌─────┐
  │ 🔵  │ ◄── │ LB  │                 │ 🔵  │ ◄── │ LB  │
  │ v1  │     └─────┘                 │ v1  │     └─────┘
  │LIVE │                              │LIVE │
  └─────┘     ┌─────┐                 └─────┘     ┌─────┐
              │ 🟢  │                              │ 🟢  │ ◄── Deploy v2
              │idle │                              │ v2  │     + Health check
              └─────┘                              │READY│
                                                   └─────┘

  Step 3: SWITCH TRAFFIC               Step 4: AFTER VALIDATION
  ┌─────┐     ┌─────┐                 ┌─────┐     ┌─────┐
  │ 🔵  │     │ LB  │                 │ 🔵  │     │ LB  │
  │ v1  │     └──┬──┘                 │idle │     └──┬──┘
  │STBY │        │                     │(v1) │        │
  └─────┘        │                     └─────┘        │
              ┌──▼──┐                              ┌──▼──┐
              │ 🟢  │ ◄── All traffic              │ 🟢  │ ◄── All traffic
              │ v2  │                              │ v2  │
              │LIVE │                              │LIVE │
              └─────┘                              └─────┘
                                          Blue is now standby
                                          (ready for next deploy)
```

---

## 🏥 Health Checks Before Switching

### Pre-Switch Validation Checklist

```
  ┌─────────────────────────────────────────────────────────┐
  │                 PRE-SWITCH VALIDATION                   │
  │  ───────────────────────────────────────────────────    │
  │                                                         │
  │  [✓] Model loaded successfully                         │
  │  [✓] Warm-up predictions completed                     │
  │  [✓] Health endpoint returns 200 OK                    │
  │  [✓] Readiness probe passes                            │
  │  [✓] Sample predictions match expected range           │
  │  [✓] Latency within SLA (p99 < 200ms)                 │
  │  [✓] Memory / GPU usage within limits                  │
  │  [✓] Feature store connectivity verified               │
  │  [✓] Smoke test suite passes                           │
  │                                                         │
  │  ALL CHECKS PASSED ──► SWITCH TRAFFIC                  │
  └─────────────────────────────────────────────────────────┘
```

### Python — Blue-Green Controller

```python
import time
import logging
import requests
from dataclasses import dataclass
from enum import Enum
from typing import Optional, List, Dict, Any

logger = logging.getLogger("blue_green")

class Environment(Enum):
    BLUE = "blue"
    GREEN = "green"

@dataclass
class HealthCheckConfig:
    endpoint: str = "/health"
    readiness_endpoint: str = "/ready"
    timeout_seconds: int = 5
    retries: int = 3
    warmup_requests: int = 10

class BlueGreenController:
    """Manages blue-green deployment switching for ML model services."""

    def __init__(
        self,
        blue_url: str,
        green_url: str,
        health_config: Optional[HealthCheckConfig] = None,
    ):
        self.urls = {
            Environment.BLUE: blue_url,
            Environment.GREEN: green_url,
        }
        self.active = Environment.BLUE
        self.health_config = health_config or HealthCheckConfig()

    @property
    def standby(self) -> Environment:
        return Environment.GREEN if self.active == Environment.BLUE else Environment.BLUE

    def check_health(self, env: Environment) -> bool:
        """Run health checks against an environment."""
        url = self.urls[env]

        for attempt in range(self.health_config.retries):
            try:
                resp = requests.get(
                    f"{url}{self.health_config.endpoint}",
                    timeout=self.health_config.timeout_seconds,
                )
                if resp.status_code == 200:
                    logger.info(f"{env.value} health check passed")
                    return True
            except requests.RequestException as e:
                logger.warning(f"{env.value} health check attempt {attempt+1} failed: {e}")
                time.sleep(2 ** attempt)

        logger.error(f"{env.value} health check FAILED after {self.health_config.retries} retries")
        return False

    def run_smoke_tests(self, env: Environment, test_inputs: List[Dict[str, Any]]) -> bool:
        """Send sample predictions to validate model behavior."""
        url = self.urls[env]
        failures = 0

        for i, payload in enumerate(test_inputs):
            try:
                resp = requests.post(
                    f"{url}/predict",
                    json=payload,
                    timeout=self.health_config.timeout_seconds,
                )
                if resp.status_code != 200:
                    failures += 1
                    logger.warning(f"Smoke test {i} returned {resp.status_code}")
                else:
                    result = resp.json()
                    if result.get("prediction") is None:
                        failures += 1
                        logger.warning(f"Smoke test {i} returned null prediction")
            except requests.RequestException as e:
                failures += 1
                logger.warning(f"Smoke test {i} failed: {e}")

        success_rate = 1 - (failures / len(test_inputs))
        logger.info(f"Smoke tests: {success_rate:.0%} success rate")
        return success_rate >= 0.95

    def warmup(self, env: Environment, sample_payload: Dict[str, Any]) -> bool:
        """Warm up model by sending initial requests."""
        url = self.urls[env]
        logger.info(f"Warming up {env.value} with {self.health_config.warmup_requests} requests")

        for _ in range(self.health_config.warmup_requests):
            try:
                requests.post(f"{url}/predict", json=sample_payload, timeout=10)
            except requests.RequestException:
                pass
        return True

    def switch(self) -> bool:
        """Switch traffic from active to standby environment."""
        target = self.standby

        # Pre-switch validation
        if not self.check_health(target):
            logger.error(f"Cannot switch — {target.value} is unhealthy")
            return False

        # Switch
        previous = self.active
        self.active = target
        logger.info(f"SWITCHED: {previous.value} → {target.value}")

        # Post-switch validation
        if not self.check_health(self.active):
            logger.error("Post-switch health check failed — rolling back!")
            self.active = previous
            return False

        logger.info(f"Active environment: {self.active.value}")
        return True

    def rollback(self) -> bool:
        """Instant rollback by switching back to the previous environment."""
        logger.warning(f"ROLLBACK: switching from {self.active.value} to {self.standby.value}")
        return self.switch()

    def deploy(
        self,
        test_inputs: List[Dict[str, Any]],
        sample_payload: Dict[str, Any],
    ) -> bool:
        """Full blue-green deployment workflow."""
        target = self.standby
        logger.info(f"Deploying to {target.value} (standby)")

        # Step 1: Health check
        if not self.check_health(target):
            return False

        # Step 2: Warm up
        self.warmup(target, sample_payload)

        # Step 3: Smoke tests
        if not self.run_smoke_tests(target, test_inputs):
            logger.error("Smoke tests failed — aborting deployment")
            return False

        # Step 4: Switch traffic
        return self.switch()

# ─── Usage ───────────────────────────────────────────────────

controller = BlueGreenController(
    blue_url="http://ml-model-blue:8080",
    green_url="http://ml-model-green:8080",
)

test_inputs = [
    {"features": {"age": 25, "income": 50000}},
    {"features": {"age": 60, "income": 120000}},
    {"features": {"age": 35, "income": 75000}},
]

success = controller.deploy(
    test_inputs=test_inputs,
    sample_payload={"features": {"age": 30, "income": 60000}},
)
print(f"Deployment {'succeeded' if success else 'failed'}")
```

---

## 🌐 Blue-Green with Cloud Infrastructure

### AWS Example (Application Load Balancer)

```python
import boto3

def switch_target_group(
    alb_arn: str,
    listener_arn: str,
    new_target_group_arn: str,
):
    """Switch ALB traffic to a different target group (blue ↔ green)."""
    client = boto3.client("elbv2")

    client.modify_listener(
        ListenerArn=listener_arn,
        DefaultActions=[
            {
                "Type": "forward",
                "TargetGroupArn": new_target_group_arn,
            }
        ],
    )
    print(f"Switched listener to target group: {new_target_group_arn}")

# Switch from blue to green
switch_target_group(
    alb_arn="arn:aws:elasticloadbalancing:...:loadbalancer/app/ml-model/...",
    listener_arn="arn:aws:elasticloadbalancing:...:listener/app/ml-model/.../...",
    new_target_group_arn="arn:aws:elasticloadbalancing:...:targetgroup/ml-green/...",
)
```

---

## 📊 Blue-Green vs Canary Comparison

| Aspect | Blue-Green | Canary |
|--------|-----------|--------|
| **Traffic migration** | All-at-once switch | Gradual (1% → 100%) |
| **Rollback speed** | ⚡ Instant (switch back) | ⚡ Fast (route to 0%) |
| **Risk during deploy** | 🟡 Medium (all users at once) | 🟢 Low (small % first) |
| **Infrastructure cost** | 🔴 2× full environments | 🟡 1 extra instance |
| **Downtime** | Zero | Zero |
| **Complexity** | 🟡 Medium | 🟡 Medium |
| **Monitoring need** | Post-switch only | Continuous per phase |
| **Best for** | Fast, confident releases | Cautious, gradual releases |

### When to Choose Which

```
  Choose BLUE-GREEN when:                Choose CANARY when:
  ─────────────────────                  ───────────────────
  • Zero-downtime is mandatory           • You want gradual validation
  • You need instant rollback            • Budget is limited (no 2× infra)
  • Model has been tested offline        • Model behavior is uncertain
  • Compliance requires full env parity  • You want to monitor incrementally
  • Small number of deployments/month    • Frequent deployments (CI/CD)
```

---

## 🔧 Kubernetes Blue-Green with Services

```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-model-blue
  labels:
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-model
      version: blue
  template:
    metadata:
      labels:
        app: ml-model
        version: blue
    spec:
      containers:
        - name: model-server
          image: ml-model:v1
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
---
# Service pointing to BLUE (switch by changing selector)
apiVersion: v1
kind: Service
metadata:
  name: ml-model-service
spec:
  selector:
    app: ml-model
    version: blue    # ← Change to "green" to switch
  ports:
    - port: 80
      targetPort: 8080
```

**Switching traffic** is a single `kubectl patch`:

```bash
# Switch from blue to green
kubectl patch service ml-model-service \
  -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback to blue
kubectl patch service ml-model-service \
  -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Blue-green gives **zero-downtime** deployment and **instant rollback** — just switch the load balancer. |
| 2 | Always run **health checks + smoke tests** on the standby environment before switching. |
| 3 | The main cost is **double infrastructure** — you maintain two full production environments. |
| 4 | Keep the standby environment **warm** — cold-start latency after switching can degrade UX. |
| 5 | Combine with **canary** for the best of both: blue-green for the switch, canary for gradual validation. |

---

## ❓ Revision Questions

1. **What makes blue-green deployment achieve zero downtime?**
   > Traffic switches happen at the load balancer level — the old environment continues serving requests until the exact moment the LB is reconfigured, so there's no gap in service.

2. **Why is it important to warm up the standby environment before switching?**
   > ML models often have cold-start latency (loading weights into GPU memory, JIT compilation, filling caches). Without warmup, the first users after the switch experience high latency.

3. **What is the primary disadvantage of blue-green deployments?**
   > Cost — you must maintain two identical production environments, effectively doubling infrastructure expenses for the model serving layer.

4. **How would you combine blue-green with canary deployment?**
   > Deploy the new model to the green environment, then use canary-style weighted routing (1% → 10% → 100%) to gradually shift traffic from blue to green, instead of an all-at-once switch.

5. **In Kubernetes, how do you perform a blue-green switch?**
   > Patch the Service selector to point to the new deployment's labels (e.g., change `version: blue` to `version: green`), which instantly routes all traffic to the new pods.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Canary Deployments](./03-canary-deployments.md) | [📂 Model Deployment](./README.md) | [Feature Flags →](./05-feature-flags.md) |

---

*Unit 7 · Chapter 4 · Blue-Green Deployments — MLOps Course Notes*
