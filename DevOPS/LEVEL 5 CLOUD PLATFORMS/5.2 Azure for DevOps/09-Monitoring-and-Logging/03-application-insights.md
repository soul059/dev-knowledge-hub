# Chapter 3: Application Insights

## Overview
**Application Insights** is an APM (Application Performance Management) feature of Azure Monitor. It provides deep monitoring for live web applications — tracking requests, dependencies, exceptions, traces, and user behaviour with automatic anomaly detection and smart alerts.

---

## 3.1 Application Insights Architecture

```
┌────────────── APPLICATION INSIGHTS ──────────────┐
│                                                    │
│  Your Application                                  │
│  ┌──────────────────────────────────────────────┐  │
│  │  App Insights SDK / Auto-Instrumentation     │  │
│  │                                              │  │
│  │  Telemetry Types:                            │  │
│  │  ├── Requests (incoming HTTP)                │  │
│  │  ├── Dependencies (outbound: SQL, HTTP, etc.)│  │
│  │  ├── Exceptions (unhandled + tracked)        │  │
│  │  ├── Traces (custom log messages)            │  │
│  │  ├── Page Views (browser-side)               │  │
│  │  ├── Custom Events (business events)         │  │
│  │  └── Custom Metrics (business KPIs)          │  │
│  └──────────────────────────────────────────────┘  │
│           │                                        │
│           ▼ (Connection String)                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  APPLICATION INSIGHTS RESOURCE               │  │
│  │  (backed by Log Analytics Workspace)         │  │
│  │                                              │  │
│  │  Features:                                   │  │
│  │  ├── Live Metrics (real-time dashboard)      │  │
│  │  ├── Application Map (dependency graph)      │  │
│  │  ├── Transaction Search (trace individual)   │  │
│  │  ├── Failures (exceptions + failed deps)     │  │
│  │  ├── Performance (response times, bottleneck)│  │
│  │  ├── Availability Tests (synthetic pings)    │  │
│  │  ├── Smart Detection (anomaly alerts)        │  │
│  │  └── Usage (users, sessions, events)         │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 3.2 Instrumentation Methods

| Method | Description | Languages |
|--------|-------------|-----------|
| **Auto-instrumentation** | No code changes; agent-based | .NET, Java, Node.js, Python |
| **SDK integration** | Add NuGet/npm/pip package + code | All supported languages |
| **OpenTelemetry** | Vendor-neutral standard | All major languages |
| **Codeless attach** | App Service auto-enable | .NET, Java, Node.js |

### .NET SDK Example

```csharp
// Program.cs (.NET 8 Minimal API)
var builder = WebApplication.CreateBuilder(args);

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry();

var app = builder.Build();

app.MapGet("/", () => "Hello World");
app.Run();
```

```json
// appsettings.json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=xxx;IngestionEndpoint=https://uksouth-0.in.applicationinsights.azure.com/"
  }
}
```

### Python SDK Example

```python
from azure.monitor.opentelemetry import configure_azure_monitor

# Auto-configure OpenTelemetry for Azure Monitor
configure_azure_monitor(
    connection_string="InstrumentationKey=xxx;IngestionEndpoint=https://..."
)

# Your Flask/Django app will auto-track requests, dependencies, exceptions
```

---

## 3.3 Application Map

```
┌────── APPLICATION MAP ────────────────────┐
│                                            │
│         ┌─────────┐                        │
│         │ Browser │                        │
│         │ (Users) │                        │
│         └────┬────┘                        │
│              │ 1.2k req/min                │
│              ▼                             │
│         ┌─────────┐    200 req/min         │
│         │  Web    │──────────────►┌──────┐ │
│         │  App    │               │ SQL  │ │
│         │  (98%)  │◄──────────────│  DB  │ │
│         └────┬────┘  avg: 45ms   │(99%) │ │
│              │                   └──────┘ │
│              │ 150 req/min                │
│              ▼                             │
│         ┌─────────┐                        │
│         │  Redis  │                        │
│         │  Cache  │                        │
│         │ (100%)  │                        │
│         └─────────┘                        │
│                                            │
│  (%) = Success rate                        │
│  Clicking a node shows details             │
│                                            │
└────────────────────────────────────────────┘
```

---

## 3.4 Key KQL Queries for App Insights

```kql
// Top 10 slowest requests
requests
| where timestamp > ago(24h)
| where success == true
| top 10 by duration desc
| project timestamp, name, duration, resultCode

// Failed requests by endpoint
requests
| where timestamp > ago(1h)
| where success == false
| summarize FailCount = count() by name, resultCode
| order by FailCount desc

// Exception trends
exceptions
| where timestamp > ago(7d)
| summarize Count = count() by bin(timestamp, 1h), type
| render timechart

// Dependency performance (external calls)
dependencies
| where timestamp > ago(1h)
| summarize AvgDuration = avg(duration), FailRate = countif(success == false) * 100.0 / count() by target, type
| order by AvgDuration desc

// Custom events (business tracking)
customEvents
| where timestamp > ago(24h)
| where name == "OrderPlaced"
| summarize OrderCount = count() by bin(timestamp, 1h)
| render barchart
```

---

## 3.5 Availability Tests

```
┌────── AVAILABILITY TESTS ─────────────────┐
│                                            │
│  URL Ping Test (Standard Test):            │
│  ┌──────────────────────────────────────┐  │
│  │  • HTTP GET/POST to your endpoint   │  │
│  │  • Check status code (200 OK)       │  │
│  │  • Check response body for text     │  │
│  │  • Check SSL certificate validity   │  │
│  │  • Run from 5+ Azure locations      │  │
│  │  • Frequency: every 5-15 minutes    │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Multi-step Web Test (Classic):            │
│  ┌──────────────────────────────────────┐  │
│  │  • Login → Navigate → Assert        │  │
│  │  • Recorded via Visual Studio       │  │
│  │  • Being replaced by TrackAvail API │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Custom (TrackAvailability):               │
│  ┌──────────────────────────────────────┐  │
│  │  • Azure Function runs custom test  │  │
│  │  • Tests complex scenarios          │  │
│  │  • Reports via SDK                  │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

---

## 3.6 CLI Commands

```bash
# Create Application Insights resource
az monitor app-insights component create \
  --app myapp-insights \
  --resource-group myRG \
  --location uksouth \
  --kind web \
  --application-type web \
  --workspace "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace"

# Get connection string
az monitor app-insights component show \
  --app myapp-insights \
  --resource-group myRG \
  --query "connectionString" -o tsv

# Enable App Insights on App Service (codeless)
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="<connection-string>" \
             ApplicationInsightsAgent_EXTENSION_VERSION="~3"

# Query App Insights from CLI
az monitor app-insights query \
  --app myapp-insights \
  --resource-group myRG \
  --analytics-query "requests | where success == false | summarize count() by name"
```

---

## 3.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| No telemetry appearing | Wrong connection string | Verify `ConnectionString` in app config |
| Missing dependencies | SDK not tracking outbound calls | Ensure SDK or auto-instrumentation is configured |
| High data volume / cost | Verbose trace logging | Use sampling or adjust `SetMinimumLevel` |
| Live Metrics not working | Firewall blocking outbound HTTPS | Allow App Insights endpoints in firewall |
| Duplicate telemetry | SDK + auto-instrumentation both enabled | Use only one instrumentation method |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Application Insights** | APM for live apps: requests, deps, exceptions, traces |
| **Instrumentation** | SDK, auto-instrumentation, or OpenTelemetry |
| **Application Map** | Visual dependency graph with success rates |
| **Live Metrics** | Real-time telemetry dashboard |
| **Availability Tests** | Synthetic monitoring from Azure locations |
| **Smart Detection** | Automatic anomaly detection and alerts |

---

## Quick Revision Questions

1. **What types of telemetry does Application Insights collect?**
   > Requests, Dependencies, Exceptions, Traces, Page Views, Custom Events, and Custom Metrics.

2. **What is the Application Map?**
   > A visual diagram showing your app's components and their dependencies, with success rates and average response times.

3. **What are availability tests?**
   > Synthetic tests that ping your endpoints from multiple Azure locations to verify uptime and response time.

4. **What is the difference between SDK and auto-instrumentation?**
   > SDK requires adding packages and code to your app. Auto-instrumentation requires no code changes — it's agent/platform-based.

5. **How do you reduce Application Insights costs?**
   > Enable sampling (process a percentage of telemetry), filter verbose traces, set daily caps, or use ingestion-time sampling.

---

[⬅ Previous: Log Analytics](02-log-analytics.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Alerts ➡](04-alerts.md)
