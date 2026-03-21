# Unit 6: Cloud Network Security — Topic 5: DDoS Protection

## Overview

**Distributed Denial of Service (DDoS) protection** in cloud environments safeguards applications and infrastructure from volumetric, protocol, and application-layer attacks. Cloud providers offer native DDoS protection services that absorb massive attack volumes leveraging their global infrastructure. Understanding DDoS protection is essential for ensuring availability and resilience.

---

## 1. DDoS Attack Types

```
DDoS ATTACK CATEGORIES:

VOLUMETRIC ATTACKS (Layer 3/4):
  → Flood bandwidth with massive traffic
  → UDP floods, ICMP floods, amplification
  → Measured in Gbps/Tbps
  → Goal: Saturate network capacity
  
  Examples:
  → DNS amplification (54x amplification)
  → NTP amplification (556x amplification)
  → SSDP amplification
  → Memcached amplification (51,200x!)

PROTOCOL ATTACKS (Layer 3/4):
  → Exploit protocol weaknesses
  → SYN floods, Ping of Death, Smurf
  → Measured in packets per second (PPS)
  → Goal: Exhaust server/firewall resources
  
  Examples:
  → SYN flood (half-open connections)
  → ACK flood
  → TCP state exhaustion
  → Fragmented packet attacks

APPLICATION LAYER ATTACKS (Layer 7):
  → Target application logic
  → HTTP floods, Slowloris, API abuse
  → Measured in requests per second (RPS)
  → Goal: Exhaust application resources
  → Hardest to detect (looks like legitimate traffic)
  
  Examples:
  → HTTP GET/POST flood
  → Slowloris (slow connections)
  → DNS query flood
  → API endpoint abuse
  → Login page brute force
```

---

## 2. Cloud DDoS Protection

```
AWS SHIELD:

  SHIELD STANDARD (Free):
  → Automatic protection for all AWS resources
  → Layer 3/4 protection
  → Always-on detection
  → Inline attack mitigation
  → No configuration needed

  SHIELD ADVANCED ($3,000/month):
  → Enhanced Layer 3/4/7 protection
  → Real-time attack visibility
  → DDoS Response Team (DRT) access
  → Cost protection (refund for scaling costs)
  → Health-based detection
  → Global threat environment dashboard
  → WAF integration (free WAF rules)
  → Proactive engagement
  → Applied to: CloudFront, ALB, NLB, EIP, Global Accelerator

AZURE DDOS PROTECTION:

  BASIC (Free):
  → Always-on traffic monitoring
  → Real-time mitigation
  → Same protection as Azure infrastructure
  → Automatic, no configuration

  STANDARD ($2,944/month):
  → Adaptive tuning (learns your traffic)
  → Attack analytics and reporting
  → DDoS Rapid Response (DRR) team
  → Cost guarantee
  → Integration with Azure Monitor
  → Applied to: VNets (protects all public IPs)

GCP CLOUD ARMOR:

  → DDoS protection built into Google's edge
  → Layer 3/4 always-on (free with GCP)
  → Layer 7 via Cloud Armor policies
  → Adaptive Protection (ML-based)
  → No separate DDoS service needed
  → Applied to: External HTTP(S) Load Balancer

COMPARISON:
  Feature          | AWS Shield Adv | Azure DDoS Std | GCP Cloud Armor
  Layer 3/4        | Yes            | Yes            | Yes (built-in)
  Layer 7          | With WAF       | Partial        | Yes (policies)
  ML Detection     | Yes            | Yes            | Adaptive Protect
  DDoS Team        | DRT            | DRR            | Support
  Cost Protection  | Yes            | Yes            | No
  Monthly Cost     | $3,000         | $2,944         | Per policy
```

---

## 3. DDoS Mitigation Strategies

```
MITIGATION ARCHITECTURE:

  ┌─────────────────────────────────────┐
  │        DDoS PROTECTION LAYERS       │
  │                                     │
  │  ┌──────────────────────────────┐   │
  │  │ 1. CDN / Edge Protection     │   │
  │  │ CloudFront, Azure CDN, etc.  │   │
  │  └──────────────┬───────────────┘   │
  │                 │                   │
  │  ┌──────────────▼───────────────┐   │
  │  │ 2. DDoS Protection Service   │   │
  │  │ Shield, DDoS Standard, Armor │   │
  │  └──────────────┬───────────────┘   │
  │                 │                   │
  │  ┌──────────────▼───────────────┐   │
  │  │ 3. WAF (Application Layer)   │   │
  │  │ Rate limiting, bot detection  │   │
  │  └──────────────┬───────────────┘   │
  │                 │                   │
  │  ┌──────────────▼───────────────┐   │
  │  │ 4. Load Balancer             │   │
  │  │ Auto-scaling, health checks   │   │
  │  └──────────────┬───────────────┘   │
  │                 │                   │
  │  ┌──────────────▼───────────────┐   │
  │  │ 5. Application               │   │
  │  │ Rate limiting, caching        │   │
  │  └──────────────────────────────┘   │
  └─────────────────────────────────────┘

BEST PRACTICES:
  [ ] Use CDN for static content (absorbs volume)
  [ ] Enable cloud-native DDoS protection
  [ ] Deploy WAF for Layer 7 protection
  [ ] Auto-scaling for capacity
  [ ] Rate limiting at multiple layers
  [ ] Geoblocking if not serving global users
  [ ] Bot detection and CAPTCHA
  [ ] Anycast routing (DNS spread)
  [ ] Minimize attack surface (only expose needed)
  [ ] DDoS response plan documented and tested

RATE LIMITING:
  AWS WAF:
  → Rate-based rules
  → Block IPs exceeding threshold
  → 5-minute evaluation window
  
  Azure WAF:
  → Custom rate limiting rules
  → Bot protection
  → Geo-filtering
  
  GCP Cloud Armor:
  → Rate limiting policies
  → Ban after threshold exceeded
  → Configurable ban duration
```

---

## 4. DDoS Response

```
DDoS INCIDENT RESPONSE:

DETECTION:
  → Monitor traffic baselines
  → Alert on traffic spikes
  → Health check failures
  → Increased latency/errors
  → CPU/memory spikes on servers

RESPONSE STEPS:
  1. CONFIRM it's a DDoS (not legitimate spike)
  2. ENGAGE DDoS response team (DRT/DRR)
  3. ACTIVATE enhanced protection
  4. IMPLEMENT emergency WAF rules
  5. SCALE resources if needed
  6. BLOCK known bad sources
  7. COMMUNICATE with stakeholders
  8. DOCUMENT timeline and actions

EMERGENCY WAF RULES:
  → Rate limit suspicious IPs
  → Block attacking countries (if applicable)
  → Challenge suspicious requests (CAPTCHA)
  → Block known attack patterns
  → Restrict HTTP methods
  → Block user-agent strings

POST-INCIDENT:
  → Analyze attack vectors
  → Review protection effectiveness
  → Update WAF rules
  → Improve detection thresholds
  → Update response plan
  → Consider Shield Advanced / DDoS Standard
  → Document lessons learned
```

---

## 5. Assessment

```
DDoS PROTECTION ASSESSMENT:

CHECKLIST:
  [ ] DDoS protection service enabled
  [ ] Public-facing resources identified
  [ ] WAF deployed on all web applications
  [ ] Rate limiting configured
  [ ] Auto-scaling enabled
  [ ] CDN in front of web applications
  [ ] DDoS response plan documented
  [ ] Response team contacts identified
  [ ] Regular DDoS drills conducted
  [ ] Monitoring and alerting configured

COMMON GAPS:
  Gap                              | Risk     | Fix
  No DDoS protection service       | High     | Enable Shield/DDoS Standard
  No WAF on web apps               | High     | Deploy WAF
  No rate limiting                  | Medium   | Configure rate rules
  No auto-scaling                   | Medium   | Enable scaling policies
  No DDoS response plan            | High     | Create and test plan
  Direct-to-origin access possible | Medium   | Restrict to CDN/LB only
  DNS not protected                | Medium   | Use managed DNS

TESTING:
  → Use legitimate load testing tools
  → Simulate traffic patterns
  → Verify DDoS protection activation
  → Test auto-scaling triggers
  → Validate WAF rule effectiveness
  → Test alerting and notification
  → Tabletop exercises for response
  ⚠ NEVER conduct real DDoS against production
     without authorization and coordination
```

---

## Summary Table

| Protection | Free Tier | Paid Tier | Layer 7 |
|------------|-----------|-----------|---------|
| AWS Shield | Standard (L3/4) | Advanced ($3K/mo) | With WAF |
| Azure DDoS | Basic (L3/4) | Standard ($2.9K/mo) | Partial |
| GCP Armor | Built-in (L3/4) | Per policy | Yes |
| Cloudflare | Free tier | Pro/Business/Ent | Yes |

---

## Revision Questions

1. What are the three categories of DDoS attacks?
2. How do AWS Shield, Azure DDoS Protection, and GCP Cloud Armor differ?
3. What is a layered DDoS mitigation architecture?
4. What steps should be taken during a DDoS incident?
5. How should DDoS protection be assessed during a security review?

---

*Previous: [04-private-connectivity.md](04-private-connectivity.md) | Next: [06-waf-implementation.md](06-waf-implementation.md)*

---

*[Back to README](../README.md)*
