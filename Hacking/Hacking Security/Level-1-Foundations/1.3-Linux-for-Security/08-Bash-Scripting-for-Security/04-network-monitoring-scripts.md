# Network Monitoring Scripts

## Unit 8 - Topic 4 | Bash Scripting for Security

---

## Overview

**Network monitoring scripts** detect unauthorized hosts, open ports, suspicious connections, and network anomalies. These bash scripts automate the discovery and alerting process, providing lightweight **network security monitoring** without expensive commercial tools.

---

## 1. Network Discovery Script

```bash
#!/bin/bash
# network_discovery.sh — Discover hosts and services on the network
set -uo pipefail

SUBNET="${1:-10.0.0}"
OUTPUT="discovery_$(date +%Y%m%d_%H%M%S).txt"

echo "=== NETWORK DISCOVERY ==="
echo "Subnet: ${SUBNET}.0/24"
echo "Date: $(date)"
echo ""

# Ping sweep (parallel for speed)
echo "--- LIVE HOSTS ---"
for i in $(seq 1 254); do
    ping -c 1 -W 1 "${SUBNET}.$i" &>/dev/null && \
        echo "[UP] ${SUBNET}.$i" &
done
wait                                  # Wait for all pings to complete
echo ""

# ARP table (more reliable on local network)
echo "--- ARP TABLE ---"
echo "IP Address      | MAC Address       | Interface"
arp -a 2>/dev/null | awk '{print $2, $4, $7}' | tr -d '()'

# Compare with known hosts
echo ""
echo "--- COMPARISON WITH KNOWN HOSTS ---"
KNOWN_HOSTS="/opt/security/known_hosts.txt"
if [[ -f "$KNOWN_HOSTS" ]]; then
    arp -a | awk '{print $2}' | tr -d '()' | while read -r ip; do
        grep -q "$ip" "$KNOWN_HOSTS" || echo "[NEW] Unknown host: $ip"
    done
else
    echo "No known hosts file found. Create: $KNOWN_HOSTS"
fi
```

---

## 2. Connection Monitor

```bash
#!/bin/bash
# connection_monitor.sh — Monitor active network connections
set -uo pipefail

echo "=== ACTIVE CONNECTION MONITOR ==="
echo "Date: $(date)"
echo ""

# 1. Listening services
echo "--- LISTENING SERVICES ---"
printf "%-6s %-25s %-s\n" "PROTO" "ADDRESS:PORT" "PROCESS"
ss -tlnp 2>/dev/null | awk 'NR>1 {print $1, $4, $6}' | \
    while read -r proto addr proc; do
        printf "%-6s %-25s %-s\n" "$proto" "$addr" "$proc"
    done

# 2. Established connections
echo ""
echo "--- ESTABLISHED CONNECTIONS ---"
echo "Count | Remote IP"
ss -tn state established 2>/dev/null | awk 'NR>1 {print $5}' | \
    cut -d: -f1 | sort | uniq -c | sort -rn | head -20

# 3. Connection by state
echo ""
echo "--- CONNECTIONS BY STATE ---"
ss -tan 2>/dev/null | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn

# 4. Suspicious connections
echo ""
echo "--- POTENTIAL CONCERNS ---"
# Connections to unusual ports
ss -tn state established 2>/dev/null | awk 'NR>1 {print $5}' | \
    grep -E ':(4444|5555|6666|1337|31337|8888|9999)$' && \
    echo "⚠️ Connections to known suspicious ports detected!" || \
    echo "✅ No connections to known suspicious ports"
```

---

## 3. Bandwidth Monitor

```bash
#!/bin/bash
# bandwidth_monitor.sh — Monitor network bandwidth per interface
set -uo pipefail

INTERFACE="${1:-eth0}"
INTERVAL="${2:-2}"                    # Seconds between checks

echo "=== BANDWIDTH MONITOR: $INTERFACE ==="
echo "Interval: ${INTERVAL}s | Press Ctrl+C to stop"
echo ""
printf "%-20s %12s %12s %12s %12s\n" "TIME" "RX KB/s" "TX KB/s" "RX Total" "TX Total"

# Read initial values
rx_prev=$(cat /sys/class/net/"$INTERFACE"/statistics/rx_bytes 2>/dev/null || echo 0)
tx_prev=$(cat /sys/class/net/"$INTERFACE"/statistics/tx_bytes 2>/dev/null || echo 0)

while true; do
    sleep "$INTERVAL"
    
    rx_curr=$(cat /sys/class/net/"$INTERFACE"/statistics/rx_bytes)
    tx_curr=$(cat /sys/class/net/"$INTERFACE"/statistics/tx_bytes)
    
    rx_rate=$(( (rx_curr - rx_prev) / INTERVAL / 1024 ))
    tx_rate=$(( (tx_curr - tx_prev) / INTERVAL / 1024 ))
    rx_total=$(( rx_curr / 1024 / 1024 ))
    tx_total=$(( tx_curr / 1024 / 1024 ))
    
    printf "%-20s %10d KB %10d KB %10d MB %10d MB\n" \
        "$(date +%H:%M:%S)" "$rx_rate" "$tx_rate" "$rx_total" "$tx_total"
    
    # Alert on high bandwidth
    if [ "$rx_rate" -gt 10240 ] || [ "$tx_rate" -gt 10240 ]; then
        echo "⚠️  HIGH BANDWIDTH ALERT: RX=${rx_rate}KB/s TX=${tx_rate}KB/s"
    fi
    
    rx_prev=$rx_curr
    tx_prev=$tx_curr
done
```

---

## 4. Port Change Detector

```bash
#!/bin/bash
# port_change_detector.sh — Detect new or closed ports
set -euo pipefail

BASELINE="/opt/security/port_baseline.txt"
CURRENT=$(mktemp)
trap "rm -f $CURRENT" EXIT

# Capture current listening ports
ss -tlnp | awk 'NR>1 {print $4}' | sort > "$CURRENT"

if [[ ! -f "$BASELINE" ]]; then
    echo "Creating initial baseline..."
    cp "$CURRENT" "$BASELINE"
    echo "Baseline saved: $BASELINE"
    echo "Current listening ports:"
    cat "$BASELINE"
    exit 0
fi

echo "=== PORT CHANGE DETECTION ==="
echo "Date: $(date)"
echo ""

# Find new ports (in current but not baseline)
new_ports=$(comm -23 "$CURRENT" "$BASELINE")
if [[ -n "$new_ports" ]]; then
    echo "⚠️  NEW PORTS DETECTED:"
    echo "$new_ports" | while read -r port; do
        proc=$(ss -tlnp | grep "$port" | awk '{print $6}')
        echo "  [NEW] $port → $proc"
    done
else
    echo "✅ No new ports"
fi

echo ""

# Find closed ports (in baseline but not current)
closed_ports=$(comm -13 "$CURRENT" "$BASELINE")
if [[ -n "$closed_ports" ]]; then
    echo "ℹ️  CLOSED PORTS (were in baseline):"
    echo "$closed_ports" | while read -r port; do
        echo "  [CLOSED] $port"
    done
else
    echo "✅ No closed ports"
fi

# Update baseline option
echo ""
read -p "Update baseline? (y/n): " update
[[ "$update" == "y" ]] && cp "$CURRENT" "$BASELINE" && echo "Baseline updated"
```

---

## 5. DNS Monitor

```bash
#!/bin/bash
# dns_monitor.sh — Monitor DNS queries for suspicious domains
set -uo pipefail

SUSPICIOUS_DOMAINS="malware.com evil.net c2server.org darknet.io"
LOG="/var/log/dns_monitor.log"

echo "=== DNS QUERY MONITOR ==="
echo "Watching for suspicious domains..."
echo "Press Ctrl+C to stop"

# Monitor DNS traffic with tcpdump
sudo tcpdump -i any -l port 53 2>/dev/null | while read -r line; do
    for domain in $SUSPICIOUS_DOMAINS; do
        if echo "$line" | grep -qi "$domain"; then
            alert="[ALERT $(date '+%H:%M:%S')] Suspicious DNS query: $domain"
            echo "$alert" | tee -a "$LOG"
        fi
    done
    
    # Detect DNS tunneling (long domain names)
    query=$(echo "$line" | grep -oP 'A\? \K\S+' 2>/dev/null)
    if [[ -n "$query" ]] && [[ ${#query} -gt 50 ]]; then
        echo "[ALERT] Possible DNS tunneling: ${query:0:60}..." | tee -a "$LOG"
    fi
done
```

---

## Summary Table

| Script | Purpose |
|--------|---------|
| **Network discovery** | Find live hosts, detect unknown devices |
| **Connection monitor** | Track active connections, find suspicious ports |
| **Bandwidth monitor** | Real-time bandwidth tracking with alerts |
| **Port change detector** | Detect new/closed listening ports |
| **DNS monitor** | Watch for suspicious DNS queries |

---

## Quick Revision Questions

1. **How do you perform a parallel ping sweep in bash?**
2. **What command shows only established TCP connections?**
3. **How do you read network interface statistics in Linux?**
4. **What is a port baseline and why is it useful?**
5. **How would you detect DNS tunneling with a bash script?**

---

[← Previous: Log Parsing Scripts](03-log-parsing-scripts.md) | [Next: Incident Response Scripts →](05-incident-response-scripts.md)

---

*Unit 8 - Topic 4 of 5 | [Back to README](../README.md)*
