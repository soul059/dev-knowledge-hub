# Chapter 4: Common Issues & Solutions

## Overview

This chapter is a practical troubleshooting reference — a collection of real-world networking problems that DevOps engineers encounter regularly, along with step-by-step diagnostic approaches and solutions. Bookmark this chapter as your go-to when things break in production.

---

## 4.1 Troubleshooting Framework

```
┌──────────────────────────────────────────────────────────────┐
│        SYSTEMATIC TROUBLESHOOTING APPROACH                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DEFINE the problem clearly                               │
│     "What exactly is failing? Since when? Who is affected?"  │
│                                                              │
│  2. ISOLATE the layer                                        │
│     DNS → Network → Transport → Application                 │
│     │       │           │           │                        │
│     dig    ping       nc/telnet    curl                      │
│                                                              │
│  3. REPRODUCE consistently                                   │
│     "Can I reproduce it? From where? Every time?"            │
│                                                              │
│  4. HYPOTHESIZE and test                                     │
│     "If it's a DNS issue, then dig should show..."           │
│                                                              │
│  5. FIX and verify                                           │
│     Apply fix → test → monitor → confirm resolution          │
│                                                              │
│  6. DOCUMENT for next time                                   │
│     Runbook, post-mortem, alert rule                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Issue 1: "Can't Connect to Service"

```
┌──────────────────────────────────────────────────────────────┐
│      DECISION TREE: CONNECTION FAILURE                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Can you ping the host?                                      │
│  ├── NO → Is DNS resolving?                                  │
│  │        ├── NO → Fix DNS (check resolv.conf, VPC DNS)      │
│  │        └── YES → Route issue or ICMP blocked              │
│  │                  → Check route tables, NACLs               │
│  │                                                           │
│  └── YES → Can you reach the port?                           │
│            ├── nc -zv host port                               │
│            ├── TIMEOUT → Firewall/SG blocking                │
│            │             → Check security groups              │
│            │             → Check NACLs                        │
│            │             → Check iptables on host             │
│            ├── REFUSED → Service not running                  │
│            │            → Check: systemctl status service     │
│            │            → Check: ss -tlnp | grep port        │
│            └── SUCCESS → Application-layer issue              │
│                         → Check: curl -v host:port            │
│                         → Check application logs              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Diagnostic Commands

```bash
# Step 1: DNS
dig api.example.com +short
# No result? → DNS issue

# Step 2: Reachability
ping -c 3 api.example.com
# Timeout? → Routing or firewall

# Step 3: Port
nc -zv api.example.com 443 -w 3
# Timeout? → Security group or NACL
# Refused? → Service not listening

# Step 4: Application
curl -vI https://api.example.com
# 5xx? → Application error
# SSL error? → Certificate issue

# Step 5: Check from server side
ssh server
ss -tlnp | grep 443          # Is something listening?
systemctl status nginx        # Is the service running?
journalctl -u nginx --tail 50 # Check service logs
```

---

## 4.3 Issue 2: "DNS Not Resolving"

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| `NXDOMAIN` | Domain doesn't exist / wrong name | Check spelling, verify domain registration |
| `SERVFAIL` | DNS server error | Try different DNS server: `dig @8.8.8.8` |
| `REFUSED` | DNS server refusing queries | Check DNS server config, VPC DNS settings |
| Timeout | DNS server unreachable | Check SG allows UDP 53, check resolv.conf |
| Wrong IP returned | Stale cache or wrong record | Check TTL: `dig +ttl`, flush cache |
| Works externally, not internally | Private DNS zone issue | Check Route 53 private hosted zone association |

```bash
# Full DNS debugging sequence
# 1. Check what DNS server is being used
cat /etc/resolv.conf

# 2. Query the configured DNS server
dig api.example.com

# 3. Query a public DNS server (bypass local DNS)
dig @8.8.8.8 api.example.com

# 4. Trace the full resolution
dig +trace api.example.com

# 5. Check if it's a caching issue
# Flush DNS cache
# Linux (systemd-resolved):
sudo systemd-resolve --flush-caches
# macOS:
sudo dscacheutil -flushcache

# 6. In Kubernetes — check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail 50

# Test DNS from inside a pod
kubectl run dnstest --rm -it --image=busybox -- nslookup my-service.default.svc.cluster.local
```

---

## 4.4 Issue 3: "Intermittent Timeouts"

```
┌──────────────────────────────────────────────────────────────┐
│      INTERMITTENT TIMEOUT CAUSES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. CONNTRACK TABLE FULL (Linux)                             │
│     ┌──────────────────────────────────────────────────┐     │
│     │ dmesg | grep "nf_conntrack: table full"          │     │
│     │ sysctl net.netfilter.nf_conntrack_count           │     │
│     │ sysctl net.netfilter.nf_conntrack_max             │     │
│     │                                                  │     │
│     │ Fix: sysctl -w net.netfilter.nf_conntrack_max=   │     │
│     │      262144                                      │     │
│     └──────────────────────────────────────────────────┘     │
│                                                              │
│  2. EPHEMERAL PORT EXHAUSTION                                │
│     ┌──────────────────────────────────────────────────┐     │
│     │ ss -s  (check TIME_WAIT count)                    │     │
│     │ ss -tnp state time-wait | wc -l                   │     │
│     │                                                  │     │
│     │ Fix: sysctl -w net.ipv4.tcp_tw_reuse=1           │     │
│     │      or use connection pooling                    │     │
│     └──────────────────────────────────────────────────┘     │
│                                                              │
│  3. NAT GATEWAY THROTTLING                                   │
│     ┌──────────────────────────────────────────────────┐     │
│     │ CloudWatch: ErrorPortAllocation > 0               │     │
│     │ CloudWatch: PacketsDropCount > 0                  │     │
│     │                                                  │     │
│     │ Fix: Add more NAT gateways across AZs            │     │
│     │      or reduce outbound connections              │     │
│     └──────────────────────────────────────────────────┘     │
│                                                              │
│  4. DNS TIMEOUT                                              │
│     ┌──────────────────────────────────────────────────┐     │
│     │ Some requests slow because DNS cache expired      │     │
│     │                                                  │     │
│     │ Fix: Use DNS caching (dnsmasq, CoreDNS cache)    │     │
│     │      Set appropriate TTLs                        │     │
│     └──────────────────────────────────────────────────┘     │
│                                                              │
│  5. LOAD BALANCER HEALTH CHECK FAILURES                      │
│     ┌──────────────────────────────────────────────────┐     │
│     │ Targets flapping healthy/unhealthy               │     │
│     │                                                  │     │
│     │ Fix: Tune health check interval/threshold        │     │
│     │      Fix app health endpoint                     │     │
│     └──────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.5 Issue 4: "SSL/TLS Errors"

| Error | Cause | Solution |
|-------|-------|----------|
| `SSL: CERTIFICATE_VERIFY_FAILED` | Untrusted CA or self-signed cert | Install CA cert in trust store |
| `SSL: WRONG_VERSION_NUMBER` | Hitting HTTPS port with HTTP (or vice versa) | Check protocol and port |
| `ERR_CERT_DATE_INVALID` | Certificate expired | Renew cert (certbot renew, ACM auto-renew) |
| `ERR_CERT_COMMON_NAME_INVALID` | Cert doesn't match domain | Issue cert for correct domain/SAN |
| `SSL routines:ssl3_get_record:wrong version` | TLS 1.0/1.1 disabled on server | Update client to support TLS 1.2+ |
| `unable to get local issuer certificate` | Missing intermediate certificate | Use fullchain.pem, not just cert |
| `handshake failure` | No common cipher suite | Update cipher configuration |

```bash
# Debug SSL/TLS issues
# Check certificate details
openssl s_client -connect example.com:443 -servername example.com

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# Check certificate chain
openssl s_client -connect example.com:443 -showcerts

# Verify certificate matches key
openssl x509 -noout -modulus -in cert.pem | md5sum
openssl rsa -noout -modulus -in key.pem | md5sum
# Both should match!

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_3
openssl s_client -connect example.com:443 -tls1_2
```

---

## 4.6 Issue 5: "Kubernetes Pod Networking"

```
┌──────────────────────────────────────────────────────────────┐
│      K8S POD CONNECTIVITY DEBUGGING                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Pod → Service fails?                                       │
│  ├── Check service exists: kubectl get svc                   │
│  ├── Check endpoints: kubectl get endpoints <service>        │
│  │   (Empty endpoints = no healthy pods!)                    │
│  ├── Check pod labels match service selector                 │
│  ├── Check NetworkPolicy blocking traffic                    │
│  └── Test DNS: nslookup <svc>.<ns>.svc.cluster.local        │
│                                                              │
│  Pod → External fails?                                      │
│  ├── Check CoreDNS: nslookup google.com                     │
│  ├── Check egress NetworkPolicy                              │
│  ├── Check node NAT/internet gateway                         │
│  └── Check pod security context (UID restrictions)           │
│                                                              │
│  External → Pod fails?                                      │
│  ├── Check ingress controller: kubectl get ingress           │
│  ├── Check Service type (LoadBalancer/NodePort)              │
│  ├── Check cloud LB health checks                            │
│  └── Check node security group allows traffic                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Full Kubernetes networking debug session

# 1. Check pod is running
kubectl get pods -o wide

# 2. Check pod logs
kubectl logs <pod-name> --tail 50

# 3. Check service and endpoints
kubectl get svc <service-name>
kubectl get endpoints <service-name>
# If endpoints empty → pods not matching label selector

# 4. Check label selectors match
kubectl get pods --show-labels
kubectl describe svc <service-name> | grep Selector

# 5. DNS test from inside cluster
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
# Inside pod:
nslookup my-service.default.svc.cluster.local
curl -v http://my-service:8080/health

# 6. Check NetworkPolicy
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy-name>

# 7. Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# 8. Check node connectivity
kubectl get nodes -o wide
# SSH to node and check iptables rules
```

---

## 4.7 Issue 6: "Docker Container Networking"

| Symptom | Cause | Solution |
|---------|-------|----------|
| Container can't reach internet | Missing DNS or no default route | Check `--dns` flag or Docker daemon DNS config |
| Container can't reach other container | Different Docker networks | Put on same network or use `docker network connect` |
| Port not accessible from host | Port not published | Use `-p 8080:80` to publish |
| "Address already in use" | Host port conflict | Change host port or stop conflicting service |
| Slow DNS in container | Docker's embedded DNS issues | Set `--dns 8.8.8.8` or configure daemon DNS |
| Container can ping IP but not hostname | DNS resolution failing | Check `/etc/resolv.conf` inside container |

```bash
# Docker networking debug commands

# Check container networks
docker inspect <container> | jq '.[0].NetworkSettings.Networks'

# Check container can reach internet
docker exec -it <container> ping -c 3 8.8.8.8          # IP works?
docker exec -it <container> ping -c 3 google.com       # DNS works?

# Check container DNS config
docker exec -it <container> cat /etc/resolv.conf

# Check Docker networks
docker network ls
docker network inspect bridge

# Test inter-container connectivity
docker exec -it container1 ping container2

# Check published ports
docker port <container>

# Check for port conflicts on host
ss -tlnp | grep :8080
```

---

## 4.8 Issue 7: "Performance Problems"

```
┌──────────────────────────────────────────────────────────────┐
│      NETWORK PERFORMANCE CHECKLIST                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. MEASURE LATENCY BREAKDOWN                                │
│     curl -w "DNS: %{time_namelookup}s                        │
│              Connect: %{time_connect}s                       │
│              TLS: %{time_appconnect}s                        │
│              TTFB: %{time_starttransfer}s                    │
│              Total: %{time_total}s\n"                        │
│       -o /dev/null -s https://api.example.com                │
│                                                              │
│     If DNS > 100ms  → DNS caching issue                      │
│     If Connect high → Network latency / routing              │
│     If TLS high     → TLS config (disable old ciphers)       │
│     If TTFB high    → Application processing slow            │
│                                                              │
│  2. CHECK MTU ISSUES                                         │
│     ping -s 1472 -M do destination  (should work for 1500)   │
│     If fails → MTU mismatch (common in VPNs, tunnels)        │
│     Fix: Adjust MTU or enable PMTUD                          │
│                                                              │
│  3. CHECK TCP WINDOW SCALING                                 │
│     sysctl net.ipv4.tcp_window_scaling                       │
│     Should be 1 (enabled) for high-bandwidth links           │
│                                                              │
│  4. CHECK KERNEL BUFFER SIZES                                │
│     sysctl net.core.rmem_max                                 │
│     sysctl net.core.wmem_max                                 │
│     Increase for high-throughput workloads                    │
│                                                              │
│  5. CHECK CONNECTIONS / BACKLOG                              │
│     sysctl net.core.somaxconn           (listen backlog)     │
│     sysctl net.ipv4.tcp_max_syn_backlog (SYN queue)          │
│     Increase if connections are being dropped                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Linux Network Tuning

```bash
# /etc/sysctl.d/99-network-tuning.conf

# Increase connection tracking table (high traffic)
net.netfilter.nf_conntrack_max = 262144

# Increase TCP backlog
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Enable TCP reuse (reduce TIME_WAIT)
net.ipv4.tcp_tw_reuse = 1

# Increase buffer sizes (high bandwidth)
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Enable TCP window scaling
net.ipv4.tcp_window_scaling = 1

# Increase local port range
net.ipv4.ip_local_port_range = 1024 65535

# Apply changes
sudo sysctl -p /etc/sysctl.d/99-network-tuning.conf
```

---

## 4.9 Quick Reference — Issue → Tool → Command

| Issue | Tool | Command |
|-------|------|---------|
| Service unreachable | ping | `ping -c 3 host` |
| Route broken | traceroute | `traceroute -T host` |
| Port filtered/closed | nc | `nc -zv -w 3 host port` |
| DNS not resolving | dig | `dig @8.8.8.8 domain` |
| Service not listening | ss | `ss -tlnp \| grep port` |
| HTTP errors | curl | `curl -vI https://url` |
| SSL issues | openssl | `openssl s_client -connect host:443` |
| Packet loss | mtr | `mtr -r host` |
| Connection resets | tcpdump | `tcpdump 'tcp[tcpflags] & tcp-rst != 0'` |
| Slow response | curl | `curl -w "%{time_total}" -s -o /dev/null url` |
| K8s service issue | kubectl | `kubectl get endpoints svc` |
| Docker networking | docker | `docker network inspect bridge` |
| Firewall blocking | VPC Flow Logs | Check REJECT entries |
| Port exhaustion | ss | `ss -s` (check TIME_WAIT count) |

---

## 4.10 Troubleshooting Runbook Template

```markdown
## Runbook: [Issue Name]

### Symptoms
- What the user/monitoring reports

### Impact
- Who/what is affected
- Severity level

### Diagnostics
1. Check [X] with `command`
2. Check [Y] with `command`
3. Check [Z] with `command`

### Resolution Steps
1. Step 1
2. Step 2
3. Verify: `command to verify fix`

### Prevention
- Alert rule to add
- Configuration change to prevent recurrence

### Escalation
- If not resolved in 15 min → escalate to [team]
```

---

## Summary Table

| Issue Category | Common Causes | Key Tools |
|---------------|--------------|-----------|
| Connectivity | SG/NACL, routing, DNS | ping, nc, dig, traceroute |
| DNS | Wrong records, cache, CoreDNS | dig, nslookup, kubectl |
| SSL/TLS | Expired cert, wrong chain, cipher | openssl, curl |
| Performance | Latency, MTU, port exhaustion | curl -w, mtr, sysctl |
| Kubernetes | Labels, endpoints, NetworkPolicy | kubectl, netshoot |
| Docker | Networks, DNS, port mapping | docker inspect, exec |
| Intermittent | Conntrack, NAT limits, health checks | ss, dmesg, CloudWatch |

---

## Quick Revision Questions

1. **A web application returns "connection timed out." Walk through a systematic troubleshooting process from DNS to application.**
2. **You see many TIME_WAIT connections. What causes this and how do you fix it?**
3. **A Kubernetes service endpoint list is empty. What are three things to check?**
4. **Your curl to an HTTPS endpoint shows `SSL: CERTIFICATE_VERIFY_FAILED`. List three possible causes.**
5. **How do you break down HTTP response time into DNS, TCP connect, TLS handshake, and first byte?**
6. **A NAT Gateway is dropping packets. What CloudWatch metric shows this, and what's the fix?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Network Monitoring](03-network-monitoring.md) | [README](../README.md) | [Course Complete! 🎉](../README.md) |
