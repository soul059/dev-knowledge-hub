# Chapter 3: DNS Resolution Process

## Overview

Understanding the step-by-step DNS resolution process — from a user typing a URL to a TCP connection being established — helps DevOps engineers diagnose latency issues, configure caching correctly, design resilient systems, and perform zero-downtime DNS migrations. This chapter explores recursive vs iterative queries, DNS caching, TTL strategies, and resolution in containerized environments.

---

## 3.1 Recursive vs Iterative Resolution

```
┌──────────────────────────────────────────────────────────────┐
│        RECURSIVE vs ITERATIVE DNS QUERIES                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  RECURSIVE QUERY (Client → Resolver):                        │
│  ┌────────┐    "Give me the final     ┌───────────┐         │
│  │ Client │ ──────answer for ──────▶  │ Recursive │         │
│  │        │    app.example.com"       │ Resolver  │         │
│  │        │ ◀──── 93.184.216.34 ──── │ (8.8.8.8) │         │
│  └────────┘    (resolver does         └───────────┘         │
│                 all the work)                                │
│                                                              │
│  ITERATIVE QUERIES (Resolver → NS servers):                  │
│  ┌───────────┐  "Where's .com?"   ┌──────┐                  │
│  │ Recursive │ ────────────────▶  │ Root │                  │
│  │ Resolver  │ ◀── "Ask a.gtld"── │  .   │                  │
│  │           │                    └──────┘                  │
│  │           │  "Where's          ┌──────┐                  │
│  │           │   example.com?"    │ .com │                  │
│  │           │ ────────────────▶  │ TLD  │                  │
│  │           │ ◀── "Ask ns1" ──── │      │                  │
│  │           │                    └──────┘                  │
│  │           │  "IP for app.      ┌──────┐                  │
│  │           │   example.com?"    │ Auth │                  │
│  │           │ ────────────────▶  │ NS   │                  │
│  │           │ ◀── "93.184.x" ── │      │                  │
│  └───────────┘                    └──────┘                  │
│                                                              │
│  Summary: Client makes 1 recursive query                     │
│           Resolver makes multiple iterative queries          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 TTL (Time to Live) Strategies

```
┌──────────────────────────────────────────────────────────────┐
│              TTL STRATEGY GUIDE                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TTL Value     │ Seconds │ Use Case                          │
│  ──────────────┼─────────┼──────────────────────────────     │
│  Very Low      │ 30-60   │ During DNS migration / failover   │
│  Low           │ 300     │ Dynamic IPs, auto-scaling          │
│  Standard      │ 3600    │ Most production records            │
│  High          │ 86400   │ Stable records (MX, NS, TXT)       │
│                                                              │
│  MIGRATION TTL STRATEGY:                                     │
│                                                              │
│  1. Before migration (24-48 hrs ahead):                      │
│     Lower TTL from 3600 → 60 seconds                         │
│                                                              │
│  2. During migration:                                        │
│     Change DNS record to new IP                              │
│     All resolvers pick up new IP within 60 seconds           │
│                                                              │
│  3. After migration (verified stable):                       │
│     Raise TTL back to 3600 seconds                           │
│                                                              │
│  ⚠ Some resolvers ignore low TTLs (enforce minimum)          │
│  ⚠ Browser caches may not respect TTL                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.3 DNS Resolution in Containers

```
┌──────────────────────────────────────────────────────────────┐
│        DNS IN DOCKER & KUBERNETES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DOCKER DNS:                                                 │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ Container resolv.conf:                                │    │
│  │ nameserver 127.0.0.11  ← Docker's embedded DNS       │    │
│  │                                                       │    │
│  │ Docker DNS resolves:                                  │    │
│  │ - Container names → container IPs                     │    │
│  │ - Service names → service VIPs (in Swarm)             │    │
│  │ - External names → forwarded to host DNS              │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  KUBERNETES DNS (CoreDNS):                                   │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ Pod resolv.conf:                                      │    │
│  │ nameserver 10.96.0.10  ← CoreDNS ClusterIP           │    │
│  │ search default.svc.cluster.local svc.cluster.local    │    │
│  │        cluster.local                                  │    │
│  │                                                       │    │
│  │ Resolution:                                           │    │
│  │ "my-service"                                          │    │
│  │  → my-service.default.svc.cluster.local               │    │
│  │  → 10.96.1.50 (ClusterIP)                            │    │
│  │                                                       │    │
│  │ "my-service.prod"                                     │    │
│  │  → my-service.prod.svc.cluster.local                  │    │
│  │  → 10.96.2.100                                        │    │
│  │                                                       │    │
│  │ External names: forwarded to upstream DNS              │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  K8s Service DNS Format:                                     │
│  <service>.<namespace>.svc.cluster.local                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 DNS Failover and Health Checks

```
┌──────────────────────────────────────────────────────────────┐
│         DNS-BASED FAILOVER (Route 53)                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  app.example.com                                             │
│       │                                                      │
│       ▼                                                      │
│  ┌─────────────────────────────────────┐                     │
│  │ Route 53 Health Check               │                     │
│  │                                     │                     │
│  │  Primary: 10.0.1.50 (us-east-1)     │  ← Health check   │
│  │  ┌───────┐                          │    every 10s       │
│  │  │ ✓ UP  │ → Route traffic here     │                     │
│  │  └───────┘                          │                     │
│  │                                     │                     │
│  │  Secondary: 10.1.1.50 (us-west-2)   │  ← Standby        │
│  │  ┌───────┐                          │                     │
│  │  │ Standby│ → Only if primary fails │                     │
│  │  └───────┘                          │                     │
│  └─────────────────────────────────────┘                     │
│                                                              │
│  If primary health check fails:                              │
│  Route 53 automatically responds with secondary IP           │
│  Failover happens within 60-90 seconds                       │
│                                                              │
│  Routing Policies:                                           │
│  - Simple: Single resource                                   │
│  - Weighted: Split traffic (e.g., 90/10 canary)             │
│  - Latency: Route to nearest region                         │
│  - Failover: Active-passive with health checks              │
│  - Geolocation: Route by user's location                    │
│  - Multi-value: Multiple healthy IPs                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.5 DNS-Based Blue/Green Deployment

```bash
# Step 1: Both environments running
# Blue (current):  blue-alb-123.elb.amazonaws.com
# Green (new):     green-alb-456.elb.amazonaws.com

# Step 2: Lower TTL 24 hours before
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "blue-alb-123.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        },
        "SetIdentifier": "blue",
        "Weight": 100
      }
    }]
  }'

# Step 3: Gradually shift traffic
# 90% Blue / 10% Green (canary test)
# Then 50/50, then 0/100

# Step 4: After verification, point fully to green
# Step 5: Raise TTL back to normal
```

---

## 3.6 DNS Debugging Workflow

```bash
# Step 1: Check if DNS resolves at all
dig app.example.com +short
# No output? → NXDOMAIN or no record

# Step 2: Check with specific resolver
dig @8.8.8.8 app.example.com +short
dig @1.1.1.1 app.example.com +short
# Different results? → Propagation issue

# Step 3: Check authoritative answer
dig app.example.com +norecurse @ns1.example.com
# This bypasses all caches

# Step 4: Trace the full resolution
dig +trace app.example.com
# Shows each hop: Root → TLD → Auth → Answer

# Step 5: Check TTL remaining
dig app.example.com | grep -A1 "ANSWER SECTION"
# Shows current TTL countdown

# Step 6: Verify NS delegation
dig NS example.com +short
# Should show your nameservers

# Step 7: Check from multiple locations
# Use online tools: dnschecker.org, whatsmydns.net
```

---

## 3.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| DNS propagation slow | High TTL on old record | Lower TTL before changes |
| Different IPs globally | Propagation in progress | Wait or check with `dig @<ns>` |
| NXDOMAIN for new subdomain | Record not created | Add record in DNS provider |
| K8s service not resolving | Wrong namespace or typo | Use FQDN: `svc.ns.svc.cluster.local` |
| Docker container can't resolve | Network not connected | Check docker network and DNS settings |
| Intermittent resolution failures | Resolver overloaded | Use multiple upstream resolvers |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Recursive Query | Client asks resolver for final answer |
| Iterative Query | Resolver asks NS servers, gets referrals |
| TTL | Cache duration; lower for migrations |
| Docker DNS | 127.0.0.11, resolves container/service names |
| K8s DNS | CoreDNS at 10.96.0.10, `svc.ns.svc.cluster.local` |
| Failover DNS | Health-checked routing (Route 53) |
| Blue/Green DNS | Weighted routing for gradual cutover |
| dig +trace | Debug entire resolution path |

---

## Quick Revision Questions

1. **Explain the difference between a recursive and iterative DNS query.**
2. **Describe the TTL strategy for a zero-downtime DNS migration.**
3. **How does DNS resolution work inside a Kubernetes pod?**
4. **What Route 53 routing policy would you use for a canary deployment?**
5. **You changed a DNS record but users still see the old IP. What do you check?**
6. **Walk through the commands you'd use to debug a DNS resolution failure.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DNS Record Types](02-dns-record-types.md) | [README](../README.md) | [Public vs Private DNS →](04-public-vs-private-dns.md) |
