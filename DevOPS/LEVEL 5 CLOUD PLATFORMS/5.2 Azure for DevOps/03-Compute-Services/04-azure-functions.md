# Chapter 4: Azure Functions

## Overview
**Azure Functions** is a serverless compute service that runs event-driven code without managing infrastructure. You pay only for execution time, making it ideal for microservices, event processing, and automation tasks in DevOps workflows.

---

## 4.1 Azure Functions Architecture

```
┌──────────────── AZURE FUNCTIONS ──────────────────┐
│                                                    │
│   TRIGGERS              FUNCTION          BINDINGS │
│   (what starts it)      (your code)       (I/O)   │
│                                                    │
│   ┌──────────┐    ┌──────────────┐    ┌─────────┐ │
│   │ HTTP     │───▶│              │───▶│ Blob    │ │
│   │ Timer    │    │   Function   │    │ Queue   │ │
│   │ Queue    │    │     Code     │    │ CosmosDB│ │
│   │ Blob     │    │              │    │ SQL     │ │
│   │ Event Hub│    │  (C#, JS,    │    │ HTTP    │ │
│   │ CosmosDB │    │   Python,    │    │ Event   │ │
│   │ Service  │    │   Java,      │    │  Hub    │ │
│   │   Bus    │    │   PowerShell)│    │ SendGrid│ │
│   └──────────┘    └──────────────┘    └─────────┘ │
│                                                    │
│   Hosting Plans:                                   │
│   • Consumption: Pay per execution, auto-scale     │
│   • Premium: Pre-warmed, VNet, no cold start      │
│   • Dedicated: Runs on App Service Plan            │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Hosting Plan Comparison

| Feature | Consumption | Premium | Dedicated |
|---------|------------|---------|-----------|
| **Scaling** | 0 to 200 instances | 1 to 100 | Manual/auto |
| **Cold Start** | Yes | No (pre-warmed) | No |
| **Max Timeout** | 5 min (default), 10 min (max) | 60 min | Unlimited |
| **VNet** | No | Yes | Yes (Isolated) |
| **Cost** | Pay per execution | Per second + always-on | App Service Plan cost |

---

## 4.2 Creating Functions

```bash
# Create Function App (Consumption)
az functionapp create \
  --resource-group myRG \
  --name myfuncapp-unique \
  --storage-account mystorageacct \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 18 \
  --functions-version 4

# Create Function App (Premium)
az functionapp plan create \
  --resource-group myRG \
  --name myPremiumPlan \
  --location eastus \
  --sku EP1

az functionapp create \
  --resource-group myRG \
  --name myfuncapp-premium \
  --storage-account mystorageacct \
  --plan myPremiumPlan \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4
```

### Example: HTTP Trigger Function (Python)

```python
# function_app.py
import azure.functions as func
import json

app = func.FunctionApp()

@app.route(route="hello", methods=["GET"])
def hello(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name', 'World')
    return func.HttpResponse(
        json.dumps({"message": f"Hello, {name}!"}),
        mimetype="application/json",
        status_code=200
    )

@app.timer_trigger(schedule="0 */5 * * * *", arg_name="timer")
def cleanup_job(timer: func.TimerRequest) -> None:
    """Runs every 5 minutes"""
    if timer.past_due:
        logging.info('Timer is past due!')
    logging.info('Cleanup job executed')
```

---

## 4.3 DevOps Use Case: Automated Infrastructure Alerts

```python
# Azure Function triggered by Azure Monitor alert
@app.route(route="alert-handler", methods=["POST"])
def handle_alert(req: func.HttpRequest) -> func.HttpResponse:
    alert_data = req.get_json()
    severity = alert_data.get('data', {}).get('essentials', {}).get('severity')
    
    if severity in ['Sev0', 'Sev1']:
        # Send to PagerDuty / Slack / Teams
        notify_oncall_team(alert_data)
    
    return func.HttpResponse("Alert processed", status_code=200)
```

---

## 4.4 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cold start latency | Consumption plan scales to 0 | Use Premium plan or keep-alive pings |
| Function timeout | Exceeds plan limit | Increase timeout or use Durable Functions |
| Cannot connect to VNet | Consumption plan doesn't support VNet | Upgrade to Premium plan |
| Deployment fails | Missing dependencies | Include `requirements.txt` or `package.json` |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Functions** | Serverless, event-driven compute; pay per execution |
| **Triggers** | HTTP, Timer, Queue, Blob, Event Hub, CosmosDB |
| **Bindings** | Declarative I/O connections (input/output) |
| **Plans** | Consumption (cheapest), Premium (no cold start), Dedicated |
| **Languages** | C#, JavaScript, Python, Java, PowerShell |

---

## Quick Revision Questions

1. **What triggers an Azure Function to execute?**
   > Triggers like HTTP requests, timers (CRON), queue messages, blob uploads, Event Hub events, or Cosmos DB changes.

2. **What is the difference between a trigger and a binding?**
   > A trigger causes the function to run (one per function). Bindings are declarative connections for input/output data (multiple per function).

3. **When should you choose Premium plan over Consumption?**
   > When you need no cold starts, VNet integration, longer execution times, or predictable performance.

4. **What is the default timeout for a Consumption plan function?**
   > 5 minutes (can be extended to 10 minutes maximum).

5. **How are Azure Functions billed in the Consumption plan?**
   > Based on number of executions and execution time (GB-seconds).

---

[⬅ Previous: Azure App Service](03-azure-app-service.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Container Instances ➡](05-azure-container-instances.md)
