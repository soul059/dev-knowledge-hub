# Chapter 6: Diagnostics

## Overview
**Azure Diagnostics** covers the tools and techniques for troubleshooting Azure resources — diagnostic settings, boot diagnostics, App Service diagnostics, serial console access, and resource-specific troubleshooting tools that help identify and resolve issues in production.

---

## 6.1 Diagnostic Tooling Overview

```
┌──────────── AZURE DIAGNOSTICS ────────────────┐
│                                                │
│  ┌────────────────────────────────────────┐    │
│  │  PLATFORM DIAGNOSTICS                 │    │
│  │  ├── Diagnostic Settings (all res.)   │    │
│  │  ├── Activity Log                     │    │
│  │  └── Resource Health                  │    │
│  └────────────────────────────────────────┘    │
│                                                │
│  ┌────────────────────────────────────────┐    │
│  │  VM DIAGNOSTICS                       │    │
│  │  ├── Boot Diagnostics (screenshot)    │    │
│  │  ├── Serial Console (text access)     │    │
│  │  ├── Run Command (remote scripts)     │    │
│  │  ├── Performance Diagnostics ext.     │    │
│  │  └── Azure Monitor Agent (logs)       │    │
│  └────────────────────────────────────────┘    │
│                                                │
│  ┌────────────────────────────────────────┐    │
│  │  APP SERVICE DIAGNOSTICS              │    │
│  │  ├── Diagnose & Solve Problems        │    │
│  │  ├── App Service Logs (stdout/stderr) │    │
│  │  ├── Log Stream (real-time)           │    │
│  │  ├── Advanced Tools (Kudu / SCM)      │    │
│  │  └── Health Check                     │    │
│  └────────────────────────────────────────┘    │
│                                                │
│  ┌────────────────────────────────────────┐    │
│  │  NETWORK DIAGNOSTICS                  │    │
│  │  ├── Network Watcher                  │    │
│  │  ├── Connection Monitor               │    │
│  │  ├── IP Flow Verify                   │    │
│  │  ├── NSG Flow Logs                    │    │
│  │  ├── Next Hop                         │    │
│  │  └── Packet Capture                   │    │
│  └────────────────────────────────────────┘    │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 6.2 VM Diagnostics

### Boot Diagnostics

```
┌────── BOOT DIAGNOSTICS ───────────────────┐
│                                            │
│  When VM won't start or RDP/SSH fails:     │
│                                            │
│  1. Enable Boot Diagnostics               │
│     (captures screenshot + serial log)     │
│                                            │
│  2. View screenshot in Azure Portal        │
│     → See BSOD, boot loop, login screen   │
│                                            │
│  3. View serial log                        │
│     → See Linux boot messages, grub errors │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Enable boot diagnostics
az vm boot-diagnostics enable \
  --resource-group myRG \
  --name myVM

# Get boot diagnostics screenshot URL
az vm boot-diagnostics get-boot-log \
  --resource-group myRG \
  --name myVM

# Serial Console (interactive terminal)
# Available in Azure Portal → VM → Serial Console
# Useful when SSH/RDP is broken

# Run Command (execute scripts remotely)
az vm run-command invoke \
  --resource-group myRG \
  --name myVM \
  --command-id RunShellScript \
  --scripts "cat /var/log/syslog | tail -50"

# Windows Run Command
az vm run-command invoke \
  --resource-group myRG \
  --name myWinVM \
  --command-id RunPowerShellScript \
  --scripts "Get-EventLog -LogName System -Newest 20"
```

---

## 6.3 App Service Diagnostics

```
┌────── APP SERVICE DIAGNOSTICS ────────────┐
│                                            │
│  Diagnose and Solve Problems (Portal):     │
│  ┌──────────────────────────────────────┐  │
│  │  Interactive diagnostic tool:       │  │
│  │  ├── Availability & Performance     │  │
│  │  ├── High CPU / Memory              │  │
│  │  ├── HTTP 5xx Errors                │  │
│  │  ├── Application Crashes            │  │
│  │  ├── SSL/TLS Issues                 │  │
│  │  └── Configuration & Management     │  │
│  │                                     │  │
│  │  AI-powered suggestions ✨          │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Enable App Service logging
az webapp log config \
  --resource-group myRG \
  --name myapp \
  --application-logging filesystem \
  --detailed-error-messages true \
  --failed-request-tracing true \
  --web-server-logging filesystem \
  --level information

# Stream logs in real-time
az webapp log tail \
  --resource-group myRG \
  --name myapp

# Download log files
az webapp log download \
  --resource-group myRG \
  --name myapp \
  --log-file webapp-logs.zip

# Access Kudu console (Advanced Tools)
# URL: https://myapp.scm.azurewebsites.net
# Features: file browser, process explorer, debug console
```

### Health Check

```
┌────── HEALTH CHECK ───────────────────────┐
│                                            │
│  Configure health check path: /health      │
│                                            │
│  App Service sends GET /health every min   │
│       │                                    │
│       ├── 200-299 → Instance is healthy    │
│       │                                    │
│       └── Other → After threshold (e.g.,   │
│           10 consecutive failures):        │
│           Instance removed from load       │
│           balancer and replaced             │
│                                            │
│  Enable in: App Service → Health Check     │
│  Path: /health                             │
│  Threshold: 2-10 failed checks             │
│                                            │
└────────────────────────────────────────────┘
```

---

## 6.4 Network Diagnostics

```bash
# IP Flow Verify — check if traffic is allowed
az network watcher test-ip-flow \
  --resource-group myRG \
  --vm myVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.0.4:80 \
  --remote 203.0.113.5:60000
# Output: Access=Allow or Access=Deny (and which NSG rule)

# Next Hop — check routing
az network watcher show-next-hop \
  --resource-group myRG \
  --vm myVM \
  --source-ip 10.0.0.4 \
  --dest-ip 8.8.8.8
# Output: NextHopType=Internet, NextHopIp=null

# Connection Monitor — continuous connectivity testing
az network watcher connection-monitor create \
  --resource-group myRG \
  --name "web-to-db-monitor" \
  --location uksouth \
  --test-group-name "WebToSQL" \
  --endpoint-source-name "WebVM" \
  --endpoint-source-resource-id "/subscriptions/.../virtualMachines/webVM" \
  --endpoint-dest-name "SQLDB" \
  --endpoint-dest-address "mydb.database.windows.net" \
  --test-config-name "TCP443" \
  --protocol Tcp \
  --tcp-port 1433

# NSG Flow Logs — traffic analysis
az network watcher flow-log create \
  --resource-group myRG \
  --name "nsg-flow-log" \
  --nsg "web-nsg" \
  --storage-account "flowlogsstorage" \
  --workspace "myWorkspace" \
  --enabled true \
  --retention 30
```

---

## 6.5 Common Diagnostic Scenarios

```
┌────── TROUBLESHOOTING PLAYBOOK ───────────┐
│                                            │
│  "VM won't start":                         │
│  1. Check Boot Diagnostics screenshot      │
│  2. Review Serial Console output           │
│  3. Check Activity Log for stop events     │
│  4. Verify disk and NIC are attached       │
│                                            │
│  "App returns 500 errors":                 │
│  1. Check App Service Logs (log stream)    │
│  2. Review Application Insights exceptions │
│  3. Use Diagnose & Solve Problems          │
│  4. Check configuration (connection strings│
│  5. Review deployment logs in Kudu         │
│                                            │
│  "Can't connect to VM/resource":           │
│  1. IP Flow Verify (is traffic allowed?)   │
│  2. Check NSG rules                        │
│  3. Check Next Hop (routing)               │
│  4. Test connectivity with Connection Mon. │
│  5. Verify firewall and NVA config         │
│                                            │
│  "High latency / slow response":           │
│  1. Application Insights performance tab   │
│  2. Check dependency call durations        │
│  3. Review VM CPU/Memory metrics           │
│  4. Check database DTU/vCore usage         │
│  5. Network Watcher connection diagnostics │
│                                            │
└────────────────────────────────────────────┘
```

---

## 6.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Boot diagnostics unavailable | Feature not enabled on VM | Enable boot diagnostics in VM settings |
| Serial Console not working | Not available on Basic-tier VMs | Use Run Command as alternative |
| Log Stream shows nothing | App logging not enabled | Enable application logging first |
| NSG Flow Logs not capturing | Network Watcher not registered | Register `Microsoft.Network` provider |
| Kudu inaccessible | SCM site blocked by network restrictions | Allow access to `.scm.azurewebsites.net` |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Diagnostic Settings** | Route platform logs/metrics to workspace/storage/hub |
| **Boot Diagnostics** | Screenshot + serial log for VM boot issues |
| **Serial Console** | Text-based terminal access to VM (when RDP/SSH fails) |
| **App Service Diagnostics** | Diagnose & Solve, Log Stream, Kudu, Health Check |
| **Network Watcher** | IP Flow Verify, Next Hop, Connection Monitor, Flow Logs |
| **Run Command** | Execute scripts on VMs remotely via Azure |

---

## Quick Revision Questions

1. **What is boot diagnostics?**
   > A VM feature that captures a screenshot and serial log during boot, helping diagnose startup failures when RDP/SSH is unavailable.

2. **How do you stream App Service logs in real-time?**
   > Use `az webapp log tail` or view the Log Stream blade in the Azure Portal after enabling application logging.

3. **What is IP Flow Verify?**
   > A Network Watcher tool that checks whether traffic to/from a VM is allowed or denied by NSG rules, identifying which rule applies.

4. **What is Kudu (Advanced Tools)?**
   > A management site (`.scm.azurewebsites.net`) for App Service providing file browser, process explorer, debug console, and deployment logs.

5. **When would you use the Serial Console?**
   > When a VM's RDP/SSH is broken (e.g., firewall misconfiguration, OS crash) and you need text-based terminal access to fix it.

6. **What are NSG Flow Logs?**
   > Logs showing network traffic (allowed/denied) through NSG rules, stored in a storage account and analysable with Traffic Analytics.

---

[⬅ Previous: Dashboards](05-dashboards.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Key Vault ➡](../10-Security/01-azure-key-vault.md)
