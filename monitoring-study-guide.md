# Monitoring & Management — AZ-305 Study Guide

> **Exam Weight:** Part of ~25% Identity/Governance domain

---

## Key Topics to Study

### 1. Azure Monitor
- Central platform for **metrics, logs, alerts, dashboards**
- Data sources: Azure resources, OS (agents), applications, custom

#### Metrics
- Numerical time-series data, stored 93 days
- Near-real-time, low latency
- Used for autoscale rules and alert conditions

#### Logs (Azure Monitor Logs / Log Analytics)
- Stored in **Log Analytics workspace** (Azure Data Explorer under the hood)
- Query with **Kusto Query Language (KQL)** — exam may show basic KQL
- Retention: 30 days default, up to 730 days (longer via archive)

### 2. Log Analytics Workspace

Central repository for logs from multiple sources. Backed by Azure Data Explorer.

- One workspace can aggregate: VMs, AKS, App Service, Activity Log, Security events, Network Watcher
- **Workspace design:** Centralized (one workspace) vs decentralized (per team/region)
  - **Exam tip:** Centralized is recommended unless compliance requires data isolation by region or team

#### KQL Quick Reference

```kql
-- Count errors in the last hour
AzureDiagnostics
| where TimeGenerated > ago(1h)
| where Level == "Error"
| summarize count() by Resource

-- Top 10 VMs by CPU usage
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer
| top 10 by avg_CounterValue desc

-- Find failed sign-ins in Entra ID
SigninLogs
| where ResultType != 0
| summarize FailedAttempts = count() by UserPrincipalName
| where FailedAttempts > 10
```

> **Exam tip:** KQL uses `where`, `summarize`, `project`, `top`, `join`, `extend`. Exam questions may show a snippet and ask what it returns — know the basics.

### 3. Azure Monitor Agents

| Agent | Platform | Data Collected | Status |
|---|---|---|---|
| **Azure Monitor Agent (AMA)** | Windows / Linux | Metrics + Logs via Data Collection Rules | Current — preferred |
| **MMA / OMS Agent** | Windows / Linux | Logs only | Legacy — being retired |
| **Diagnostics Extension (WAD/LAD)** | Windows / Linux | Metrics → Storage Account / Event Hubs | Legacy |

- **Data Collection Rules (DCR):** AMA uses DCRs to define what data to collect and where to send it — centrally managed, reusable

### 4. Diagnostic Settings

Diagnostic Settings route **resource-level metrics and logs** to a destination. Every Azure resource (VMs, App Service, SQL, AKS, etc.) can have diagnostic settings configured.

#### Sources → Destinations

| Source | What It Captures |
|---|---|
| **Resource diagnostic logs** | Operations on the resource (query logs, access logs, request traces) |
| **Resource metrics** | Numeric performance data (CPU, DTU, requests/sec) |
| **Activity Log** (subscription-level) | Control-plane operations (who created/deleted/changed what) |
| **Entra ID logs** | Sign-in logs, audit logs |

| Destination | Use Case |
|---|---|
| **Log Analytics Workspace** | Query with KQL, alert, retain up to 730 days |
| **Storage Account** | Long-term archival, compliance (cheap cold storage) |
| **Event Hubs** | Stream to external SIEM (Splunk, Datadog), real-time pipeline |
| **Partner solution** | Directly to Datadog, Elastic, etc. |

> **Exam tip:** Diagnostic settings are the primary mechanism for getting **resource logs and metrics into Log Analytics**. Without a diagnostic setting, resource logs don't flow to the workspace automatically — except for Activity Logs which can be connected via a single subscription-level setting.

> **Exam tip:** To send data to an **external SIEM**, the answer is Diagnostic Settings → **Event Hubs**.

### 5. Alerts

- **Metric alerts:** Fast, near-real-time, stateless or stateful; fire when metric crosses threshold
- **Log alerts:** Based on KQL query result; minimum 1-minute frequency; can alert on count, measure, or custom columns
- **Activity log alerts:** Triggered by control-plane events (VM deleted, policy assigned, role changed)
- **Smart detection alerts:** Application Insights anomaly detection — auto-configured, no threshold needed

#### Action Groups

Reusable set of notification + remediation actions — attach one action group to many alerts.

| Action Type | Example |
|---|---|
| Email / SMS / Push | Notify on-call engineer |
| Webhook | Call external API or ticketing system |
| Logic App | Automated workflow (create ITSM ticket) |
| Azure Automation Runbook | Run remediation script |
| Azure Function | Custom event-driven action |
| ITSM connector | Integrate with ServiceNow, Remedy |

> **Exam tip:** Action groups are defined once and reused — if the question asks about "notifying multiple people when any of 5 alerts fire," create one action group and attach it to all 5 alerts.

### 6. Application Insights
- Application Performance Monitoring (APM) for web apps
- Tracks: requests, dependencies, exceptions, custom events, traces
- **Smart detection:** Auto-detect anomalies in failure rate, performance
- **Application Map:** Visual dependency topology
- **Live Metrics:** Real-time stream (useful for deployments)
- **Sampling:** Adaptive (default) or fixed-rate — reduces data volume

### 7. Azure Workbooks
- Interactive reports combining metrics, logs, parameters
- Use for custom dashboards, operational reports, DR runbooks

### 8. Azure Advisor
- Personalized best-practice recommendations
- Categories: Cost, Security, Reliability, Performance, Operational Excellence
- Integrates with Defender for Cloud for security recommendations

### 9. Cost Management
- **Cost Analysis:** Visualize spend by resource, tag, subscription
- **Budgets:** Alert at threshold % of spending
- **Azure Reservations:** Commit to 1–3 years for significant discounts
- **Azure Hybrid Benefit:** Use existing Windows Server / SQL Server licenses
- **Tags:** Required for cost allocation — enforce via Azure Policy

### 10. Azure Service Health
- **Azure Status:** Public global outage page (status.azure.com)
- **Service Health:** Personalized view of incidents affecting your subscription, region, and services
- **Resource Health:** Per-resource health status and historical availability
- Configure **activity log alerts** for planned maintenance events and service incidents

---

### 11. Azure Automation

#### What It Is
Azure Automation provides process automation, configuration management, and update management for Azure and on-premises resources.

#### Key Features

| Feature | Description |
|---|---|
| **Runbooks** | PowerShell, Python, or graphical scripts — triggered by schedule, webhook, alert action group, or manually |
| **Update Management** | Centralized OS patch compliance and deployment across Azure VMs and Arc-enabled servers |
| **Change Tracking & Inventory** | Detect changes to software, files, registry, services on VMs |
| **State Configuration (DSC)** | Enforce desired state on VMs using PowerShell DSC — VM configuration drift detection |
| **Hybrid Runbook Worker** | Run runbooks on on-premises machines or Arc-enabled servers (not just Azure VMs) |

#### Runbook Types

| Type | Language | Use Case |
|---|---|---|
| **PowerShell** | PowerShell | Most common — manage Azure resources, Windows tasks |
| **Python** | Python 2/3 | Cross-platform scripting, Linux workloads |
| **Graphical** | Visual drag-and-drop | Non-developer authors |
| **PowerShell Workflow** | PowerShell + checkpoints | Long-running, resumable jobs |

> **Exam tip:** Azure Automation is the answer when a scenario asks for **scheduled VM start/stop**, **automated patch deployment**, or **remediation runbooks triggered by alerts**. Action groups in Azure Monitor can call an Automation runbook directly.

---

## SLA Quick Reference — Tier Edge Cases

Exam questions frequently test whether you know which tier gains or loses SLA.

| Service | Tier / Config | SLA | Key Note |
|---|---|---|---|
| **APIM** | Consumption | 99.95% | Serverless, no dedicated infra |
| **APIM** | Developer | **No SLA** | Never use in production |
| **APIM** | Basic / Standard / Premium | 99.95% | Premium adds multi-region |
| **Azure Cache for Redis** | Basic | **No SLA** | Dev/test only |
| **Azure Cache for Redis** | Standard | 99.9% | Primary + replica |
| **Azure Cache for Redis** | Premium | 99.9% + zone redundancy option | Adds persistence, clustering |
| **App Service** | Free / Shared | **No SLA** | Not for production |
| **App Service** | Basic and above | 99.95% | Basic has no autoscale |
| **Azure SQL Database** | General Purpose (single) | 99.99% | |
| **Azure SQL Database** | Business Critical (single) | 99.99% | Adds readable secondary |
| **Azure SQL Database** | Hyperscale | 99.99% | |
| **Azure Functions** | Consumption | 99.95% | |
| **Azure Functions** | Premium | 99.95% | Pre-warmed, VNet |
| **VMs** | Single VM with Premium SSD | 99.9% | |
| **VMs** | Availability Set | 99.95% | Same datacenter, different fault domains |
| **VMs** | Availability Zones | 99.99% | Different datacenters in region |
| **Storage Account** | LRS | 99.9% read / 99.9% write | |
| **Storage Account** | ZRS | 99.99% read / 99.9% write | |
| **Storage Account** | GRS/RA-GRS | 99.9% / 99.99% read | RA-GRS adds secondary read |
| **AKS** | Free tier | **No SLA** | Dev/test clusters |
| **AKS** | Standard tier | 99.5% API server | Paid tier with SLA |
| **AKS** | Premium tier | 99.95% API server | Adds longer support |
| **Azure Kubernetes Service** | + Availability Zones | 99.9% nodes | Node pool spread |
| **Event Hubs** | Basic | 99.95% | |
| **Event Hubs** | Standard | 99.95% | Adds consumer groups |
| **Event Hubs** | Premium / Dedicated | 99.95% | Adds zone redundancy |
| **Service Bus** | Basic | 99.9% | No topics |
| **Service Bus** | Standard / Premium | 99.9% | Premium adds VNet, zone redundancy |

> **Exam tip:** Any **Developer** or **Free/Shared/Basic** tier (APIM Developer, Redis Basic, App Service Free/Shared, AKS Free) typically means **no production SLA**. Upgrading to the next tier is the fix.

---

## Exam Scenario Cheat Sheet

| Scenario | Answer | Key Detail |
|---|---|---|
| Collect VM CPU/memory metrics from Azure and on-premises | Azure Monitor Agent (AMA) + Log Analytics Workspace | AMA is the current agent; DCR defines collection |
| Route SQL Database query logs to Log Analytics | Diagnostic Settings → Log Analytics Workspace | Must be explicitly configured |
| Stream resource logs to Splunk in real time | Diagnostic Settings → Event Hubs | Event Hubs is the stream gateway |
| Archive logs cheaply for 2 years for compliance | Diagnostic Settings → Storage Account | Cold archival |
| Get notified when VM CPU > 90% for 5 minutes | Metric alert + Action Group | Near-real-time metric alert |
| Alert when a specific user deletes a VM | Activity Log alert | Control-plane event detection |
| Alert when error rate increases by 30% (no threshold) | Application Insights Smart Detection | Anomaly-based, no manual threshold |
| Monitor application request duration and dependency failures | Application Insights | APM — not Azure Monitor metrics |
| Real-time traffic view during deployment | Application Insights Live Metrics | Live stream of requests/failures |
| Custom operational dashboard combining metrics and logs | Azure Workbooks | Interactive parameterized report |
| Get cost recommendations without manual analysis | Azure Advisor | Personalized best-practice recommendations |
| Set a spending limit and get alert at 80% | Azure Cost Management Budgets | Threshold-based cost alert |
| Enforce cost tagging across all resources | Azure Policy + Required Tags | Deny untagged resources at creation |
| Detect planned maintenance for your region | Azure Service Health alert | Subscription + region scoped |
| Auto-start VMs at 8 AM and stop at 6 PM | Azure Automation Runbook + Schedule | PowerShell runbook on schedule |
| Deploy OS patches to 500 VMs monthly | Azure Automation Update Management | Centralized patching across Azure + Arc |
| Enforce VM configuration settings, detect drift | Azure Automation State Configuration (DSC) | Desired State Configuration |
| Run remediation runbook when alert fires | Alert Action Group → Automation Runbook | Action groups support runbook actions |
| Centralized Log Analytics or one per team? | Centralized (unless data sovereignty required) | Centralized reduces cost and simplifies queries |

---

## Exam Tips
- **Log Analytics workspace** is the backbone of most monitoring scenarios
- Know **AMA** is the current agent; MMA is legacy
- **Application Insights** is the answer for application-level monitoring (not VM metrics)
- **Action groups** are reusable — one group can be attached to many alerts
- Tags + Azure Policy = cost governance at scale
- Understand the difference between **Metrics** (fast, numeric) and **Logs** (rich, queryable)

---

## Gap-Closing Review Priorities

This guide already covers the core services. For AZ-305, spend extra time on the architecture decisions behind them.

### Areas to Strengthen

| Topic | What to be ready to explain |
|---|---|
| **Diagnostic Settings destinations** | When to send logs to Log Analytics, Event Hubs, or Storage based on query, integration, or retention needs |
| **Centralized vs segmented workspaces** | Why centralization is simpler, and when sovereignty or isolation requirements force multiple workspaces |
| **Budgets vs Advisor vs Policy** | Budgets alert on spend, Advisor recommends optimization, Policy enforces governance |
| **Application Insights vs Azure Monitor metrics** | App behavior and dependencies vs platform metrics |
| **Operational design** | How monitoring choices affect cost, retention, alert noise, and investigation speed |

### Cost and Operations Recommendations

1. Add budgets and tagging reviews to your study routine, not just technical monitoring.
2. Practice choosing a log destination based on the requirement instead of defaulting everything to Log Analytics.
3. Review scenarios where the cheapest storage or retention option is acceptable because query speed is not required.
4. Pair every alerting topic with the action mechanism: Action Group, Logic App, Automation Runbook, or ITSM integration.

### Practice Prompts

1. A company needs to retain logs for two years for audits but rarely query them. Why is Storage a better destination than Log Analytics?
2. A team needs to stream platform logs to a SIEM in near real time. Why is Event Hubs the right destination?
3. Finance wants alerting before monthly spend exceeds budget. Why is Cost Management Budget the answer instead of Advisor?

---

## Monitoring Exam Traps

### 1. Sending every log to Log Analytics by default
- **Trap:** Log Analytics is central and queryable, so it feels universally correct
- **Better default:** Use Storage for cheap long retention and Event Hubs for streaming integration when query is not the main requirement

### 2. Choosing Azure Monitor metrics for application diagnostics
- **Trap:** Metrics are faster and simpler
- **Better default:** Application Insights for request flows, dependencies, failures, and app-level performance

### 3. Confusing budgets, Advisor, and Policy
- **Trap:** All three touch cost or governance
- **Better default:** Budgets alert, Advisor recommends, Policy enforces

### 4. Choosing Activity Log alerts for resource performance thresholds
- **Trap:** Activity Log sounds like the central place for Azure events
- **Better default:** Metric alerts for numeric thresholds and Activity Log alerts for control-plane events

### 5. Ignoring the action path after the alert
- **Trap:** The alert itself looks like the final design step
- **Better default:** Match the action to the need: Action Group, Logic App, Automation Runbook, ITSM, or notification channel

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Rarely queried long-term audit retention | Log Analytics-only answers |
| Stream logs to external SIEM | Storage-only answers |
| Track request latency and dependency calls | VM metrics-only answers |
| Alert on VM CPU over 90 percent | Activity Log alert answers |
| Enforce tagging or governance | Advisor-only answers |
