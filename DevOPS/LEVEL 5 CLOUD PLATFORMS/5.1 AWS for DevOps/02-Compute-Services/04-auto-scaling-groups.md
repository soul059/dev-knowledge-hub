# Chapter 2.4: Auto Scaling Groups (ASG)

## Overview

Auto Scaling Groups automatically adjust the number of EC2 instances based on demand, schedules, or health checks. ASGs are foundational for **high availability**, **fault tolerance**, and **cost optimization** in AWS.

---

## ASG Architecture

```
┌────────────────────────── VPC ──────────────────────────────┐
│                                                              │
│                  ┌─────────────────────┐                    │
│                  │  Application Load   │                    │
│                  │    Balancer (ALB)    │                    │
│                  └──────────┬──────────┘                    │
│                             │                                │
│            ┌────────────────┼────────────────┐              │
│            ▼                ▼                ▼              │
│  ┌──── AZ-1a ────┐  ┌──── AZ-1b ────┐  ┌──── AZ-1c ────┐ │
│  │  ┌──────────┐ │  │  ┌──────────┐ │  │  ┌──────────┐ │ │
│  │  │   EC2    │ │  │  │   EC2    │ │  │  │   EC2    │ │ │
│  │  │ (from LT)│ │  │  │ (from LT)│ │  │  │ (from LT)│ │ │
│  │  └──────────┘ │  │  └──────────┘ │  │  └──────────┘ │ │
│  └───────────────┘  └───────────────┘  └───────────────┘ │
│                                                              │
│  ┌──────────────── Auto Scaling Group ─────────────────┐   │
│  │  Min: 2    Desired: 3    Max: 6                     │   │
│  │  Launch Template: webapp-template v3                 │   │
│  │  Health Check: ELB (recommended)                    │   │
│  │  Cooldown: 300 seconds                              │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## ASG Capacity Settings

```
┌─────────────────────────────────────────────────────┐
│            ASG CAPACITY SETTINGS                    │
│                                                     │
│  Instance Count                                     │
│       │                                             │
│  Max ─┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ 6 ─ ─ ─ ─ ─ ─   │
│       │                                             │
│       │      ┌─────┐   ┌─────────┐                 │
│       │      │Scale│   │ Scale   │                 │
│       │      │ Up  │   │  Down   │                 │
│  Des ─┤──────┤     │───┤         │─── 3 ───────── │
│       │      └─────┘   └─────────┘                 │
│  Min ─┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ 2 ─ ─ ─ ─ ─ ─   │
│       │                                             │
│       └─────────────────────────────────► Time      │
│                                                     │
│  Min = Minimum instances (always running)           │
│  Desired = Target (ASG maintains this count)        │
│  Max = Upper limit (won't exceed this)              │
└─────────────────────────────────────────────────────┘
```

---

## Creating an Auto Scaling Group

```bash
# Create ASG with Launch Template
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name webapp-asg \
  --launch-template LaunchTemplateName=webapp-template,Version='$Latest' \
  --min-size 2 \
  --max-size 6 \
  --desired-capacity 3 \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb,subnet-ccc" \
  --target-group-arns "arn:aws:elasticloadbalancing:...:targetgroup/webapp/xxx" \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  --tags '[
    {"Key":"Name","Value":"WebApp","PropagateAtLaunch":true},
    {"Key":"Environment","Value":"Production","PropagateAtLaunch":true}
  ]'
```

---

## Scaling Policies

### Types of Scaling

```
┌────────────────────────────────────────────────────────────────┐
│                  ASG SCALING TYPES                             │
│                                                                │
│  1. MANUAL SCALING                                             │
│     └── Change desired capacity directly                      │
│                                                                │
│  2. DYNAMIC SCALING                                            │
│     ├── Target Tracking ─── "Keep CPU at 50%"                 │
│     ├── Step Scaling ────── "Add 2 when CPU > 80%"            │
│     └── Simple Scaling ──── "Add 1 when alarm fires"          │
│                                                                │
│  3. SCHEDULED SCALING                                          │
│     └── "Scale to 10 at 9 AM on weekdays"                    │
│                                                                │
│  4. PREDICTIVE SCALING                                         │
│     └── ML-based: learns patterns, scales proactively         │
└────────────────────────────────────────────────────────────────┘
```

### Target Tracking (Most Common)

```bash
# Keep average CPU at 50%
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name webapp-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

### Predefined Metrics for Target Tracking

| Metric | Description |
|--------|-------------|
| `ASGAverageCPUUtilization` | Average CPU across all instances |
| `ASGAverageNetworkIn` | Average network bytes received |
| `ASGAverageNetworkOut` | Average network bytes sent |
| `ALBRequestCountPerTarget` | Requests per target from ALB |

### Step Scaling

```bash
# Scale up: Add instances based on CPU severity
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name webapp-asg \
  --policy-name cpu-step-scaling \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments '[
    {"MetricIntervalLowerBound": 0, "MetricIntervalUpperBound": 20, "ScalingAdjustment": 1},
    {"MetricIntervalLowerBound": 20, "MetricIntervalUpperBound": 40, "ScalingAdjustment": 2},
    {"MetricIntervalLowerBound": 40, "ScalingAdjustment": 3}
  ]'

# Requires a CloudWatch alarm to trigger
aws cloudwatch put-metric-alarm \
  --alarm-name cpu-high \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 60 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=webapp-asg \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:autoscaling:...:scalingPolicy:xxx
```

### Scheduled Scaling

```bash
# Scale up for business hours (Mon-Fri, 9 AM EST)
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name webapp-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 9 * * 1-5" \
  --desired-capacity 6 \
  --min-size 4

# Scale down for nights (Mon-Fri, 7 PM EST)
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name webapp-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 19 * * 1-5" \
  --desired-capacity 2 \
  --min-size 2
```

---

## Health Checks

```
┌───────────────────────────────────────────────────────┐
│              ASG HEALTH CHECK TYPES                   │
│                                                       │
│  EC2 Health Check (Default):                         │
│  ┌──────────┐                                        │
│  │ Instance │── Running? ── YES ──► ✓ Healthy       │
│  │          │              NO  ──► ✗ Unhealthy      │
│  └──────────┘  (only checks if VM is up)             │
│                                                       │
│  ELB Health Check (Recommended):                     │
│  ┌──────────┐                                        │
│  │ Instance │── HTTP 200 on /health? ── YES ──► ✓   │
│  │          │                          NO  ──► ✗    │
│  └──────────┘  (checks if APP is responding)         │
│                                                       │
│  When unhealthy:                                     │
│  1. ASG terminates the unhealthy instance            │
│  2. ASG launches a replacement                       │
│  3. New instance registered with ELB                 │
│                                                       │
│  Grace Period: Time to wait before first check       │
│  (allow app to start up — typically 300s)            │
└───────────────────────────────────────────────────────┘
```

---

## Instance Refresh (Rolling Updates)

Update all instances in an ASG without downtime.

```bash
# Start instance refresh (replace instances with new Launch Template version)
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name webapp-asg \
  --preferences '{
    "MinHealthyPercentage": 90,
    "InstanceWarmup": 120
  }'

# Check refresh status
aws autoscaling describe-instance-refreshes \
  --auto-scaling-group-name webapp-asg
```

```
┌─────────────────────────────────────────────────────────────┐
│            INSTANCE REFRESH (Rolling Update)                │
│                                                             │
│  Before:  [v1] [v1] [v1] [v1]     ← Old AMI              │
│                                                             │
│  Step 1:  [v1] [v1] [v1] [  ] [v2]  ← Launch new         │
│  Step 2:  [v1] [v1] [  ] [v2] [v2]  ← Replace one by one │
│  Step 3:  [v1] [  ] [v2] [v2] [v2]                        │
│  Step 4:  [  ] [v2] [v2] [v2] [v2]  ← Remove last old    │
│                                                             │
│  After:   [v2] [v2] [v2] [v2]     ← All on new AMI       │
│                                                             │
│  MinHealthyPercentage = 90%  (at least 90% always healthy) │
│  InstanceWarmup = 120s (wait before checking health)       │
└─────────────────────────────────────────────────────────────┘
```

---

## Mixed Instance Types (Cost Optimization)

```bash
# ASG with mixed On-Demand + Spot instances
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name webapp-mixed-asg \
  --mixed-instances-policy '{
    "LaunchTemplate": {
      "LaunchTemplateSpecification": {
        "LaunchTemplateName": "webapp-template",
        "Version": "$Latest"
      },
      "Overrides": [
        {"InstanceType": "t3.medium"},
        {"InstanceType": "t3a.medium"},
        {"InstanceType": "t3.large"},
        {"InstanceType": "m5.large"}
      ]
    },
    "InstancesDistribution": {
      "OnDemandBaseCapacity": 2,
      "OnDemandPercentageAboveBaseCapacity": 25,
      "SpotAllocationStrategy": "capacity-optimized"
    }
  }' \
  --min-size 2 --max-size 10 --desired-capacity 4 \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb"
```

```
┌─────────────────────────────────────────────────────┐
│        MIXED INSTANCES STRATEGY                     │
│                                                     │
│  Base: 2 On-Demand (always running, guaranteed)    │
│  Above Base: 25% On-Demand, 75% Spot              │
│                                                     │
│  With desired=8:                                    │
│  [OD] [OD] [OD] [Spot] [Spot] [Spot] [Spot] [Spot]│
│   ▲    ▲    ▲                                      │
│   Base  Above                                      │
│   (2)   base                                       │
│          (1)                                       │
│                                                     │
│  Savings: 50-70% vs all On-Demand!                │
└─────────────────────────────────────────────────────┘
```

---

## Lifecycle Hooks

Run custom actions during instance launch or termination.

```
┌──────────────────────────────────────────────────────────────┐
│               ASG LIFECYCLE HOOKS                            │
│                                                              │
│  LAUNCH:                                                     │
│  Pending ──► Pending:Wait ──► [Your Action] ──► InService  │
│                    │                                         │
│                    ├── Pull configs from S3                  │
│                    ├── Register with service discovery       │
│                    ├── Run smoke tests                       │
│                    └── Send notification                     │
│                                                              │
│  TERMINATION:                                                │
│  InService ──► Terminating:Wait ──► [Your Action] ──► Term │
│                       │                                      │
│                       ├── Drain connections                  │
│                       ├── Deregister from service discovery  │
│                       ├── Upload logs to S3                  │
│                       └── Send notification                  │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Create a launch lifecycle hook
aws autoscaling put-lifecycle-hook \
  --auto-scaling-group-name webapp-asg \
  --lifecycle-hook-name launch-hook \
  --lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING \
  --heartbeat-timeout 300 \
  --default-result CONTINUE \
  --notification-target-arn arn:aws:sns:us-east-1:123456789012:asg-events

# Complete the hook from your automation
aws autoscaling complete-lifecycle-action \
  --auto-scaling-group-name webapp-asg \
  --lifecycle-hook-name launch-hook \
  --instance-id i-xxx \
  --lifecycle-action-result CONTINUE
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Instances keep launching/terminating | Health check failing but ASG replaces | Check app health endpoint; increase grace period |
| Scaling too slow | Cooldown too long | Reduce cooldown; use step scaling with warm-up |
| Stuck at desired count | Launch failures (AMI, SG, subnet) | Check ASG activity history for error messages |
| Instances in wrong AZ | Subnet misconfiguration | Ensure all AZ subnets listed in ASG |
| Spot interruptions | Spot capacity reclaimed | Use capacity-optimized strategy; diversify types |

```bash
# Debug ASG issues — check activity history
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name webapp-asg \
  --max-items 10
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **ASG** | Maintains desired instance count across AZs. Auto-replaces unhealthy instances. |
| **Capacity** | Min (floor), Desired (target), Max (ceiling). ASG adjusts desired between min and max. |
| **Target Tracking** | "Keep metric at X" — simplest scaling. Use ASGAverageCPUUtilization at 50%. |
| **Step Scaling** | Graduated response — add more instances as metric increases beyond thresholds. |
| **Scheduled** | CRON-based scaling for predictable patterns (business hours, weekends). |
| **Health Checks** | ELB > EC2. Use ELB for app-level health. Set appropriate grace period. |
| **Instance Refresh** | Rolling update of all instances to new Launch Template version. |
| **Mixed Instances** | Combine On-Demand + Spot for 50-70% cost savings. |
| **Lifecycle Hooks** | Custom actions at launch/termination (drain, configure, notify). |

---

## Quick Revision Questions

1. **What are the three capacity settings in an ASG?**
   > Minimum (floor), Desired (target maintained), Maximum (ceiling). ASG always keeps running instances between Min and Max.

2. **Why is ELB health check preferred over EC2 health check?**
   > EC2 only checks if the VM is running. ELB checks if the application is actually responding to HTTP requests.

3. **What is the difference between Target Tracking and Step Scaling?**
   > Target Tracking: "keep metric at X" (simple). Step Scaling: "add N instances when metric crosses specific thresholds" (graduated response).

4. **How does Instance Refresh work?**
   > It replaces instances one at a time (or in batches) with new instances from the updated Launch Template, maintaining MinHealthyPercentage during the process.

5. **What is a Lifecycle Hook and when would you use it?**
   > A hook that pauses instance launch/termination so you can run custom actions (drain connections, pull config, run tests) before completing.

6. **How do mixed instance policies save money?**
   > By running a base of On-Demand instances for reliability and using Spot Instances (up to 90% cheaper) for the rest.

---

[← Previous: 2.3 AMIs and Launch Templates](03-amis-and-launch-templates.md) | [Next: 2.5 Elastic Load Balancing →](05-elastic-load-balancing.md)

[← Back to README](../README.md)
