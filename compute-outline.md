# Compute Solutions — AZ-305 Study Outline

> **Status:** Outline — expand as needed
> **Exam Weight:** Part of ~35% Infrastructure domain

---

## Key Topics to Study

### 1. Azure Virtual Machines

#### VM Series

| Series | Type | Use Case |
|---|---|---|
| **B** | Burstable | Dev/test, low-traffic web, small DBs — earns CPU credits when idle, spends on bursts |
| **D** | General purpose | Balanced CPU/memory — most common production workloads |
| **E** | Memory optimized | Large in-memory DBs, SAP HANA, caching |
| **F** | Compute optimized | High CPU-to-memory — batch processing, game servers |
| **L** | Storage optimized | High disk throughput/IOPS — Cassandra, Elasticsearch, NoSQL |
| **M** | Large memory | Largest in-memory workloads — SAP HANA scale-up |
| **N** | GPU | ML training/inference, graphics rendering |

**B-Series (Burstable) — Exam Detail:**
- Accumulates **CPU credits** when running below baseline CPU usage
- Spends credits to burst above baseline when needed
- Baseline varies by size (e.g., B2s = 40% baseline, can burst to 100%)
- **Credit model:** If credits are exhausted, CPU is throttled to baseline
- **Choose when:** Workload has low average CPU but occasional spikes (dev/test, CI runners, small web apps, microservices)
- **Avoid when:** Sustained high CPU — credits will drain and performance degrades

#### Other VM Options
- Availability: Availability Sets vs Availability Zones vs VMSS
- **Spot VMs:** Up to 90% savings, evictable — dev/test and batch only
- **Reserved Instances:** 1–3 year commitment, up to 72% savings
- **Azure Dedicated Host:** Physical server isolation for compliance (GDPR, HIPAA)
- Proximity Placement Groups — minimize latency between VMs

### 2. Azure Virtual Machine Scale Sets (VMSS)
- Auto-scale based on metrics or schedules
- Uniform (same image) vs Flexible (mix of VMs) orchestration
- **Flexible is preferred** for new deployments
- Health extension + automatic repairs for self-healing

### 3. Azure App Service
- PaaS for web apps, REST APIs, mobile backends
- **Plans:** Free/Shared (dev), Basic (no autoscale), Standard/Premium (autoscale, slots), Isolated (VNet injection)
- Deployment slots: Blue-green deployments, slot swap with zero downtime
- Networking: VNet Integration (outbound), Private Endpoint (inbound)
- App Service Environment (ASE): Fully isolated, VNet-injected, highest scale

### 4. Azure Kubernetes Service (AKS)
- Managed Kubernetes control plane
- Node pools: System pool (critical pods) + User pools (workloads)
- **Scaling:** Horizontal Pod Autoscaler (HPA), Cluster Autoscaler, Kubernetes Event-Driven Autoscaling (KEDA)
- Networking: Kubenet vs Azure CNI (Azure CNI for VNet integration)
- **Azure CNI Overlay:** Pod IPs from overlay, nodes use VNet IPs — saves IP space
- Security: Azure AD integration, Role-Based Access Control (RBAC), Azure Policy for AKS, workload identity
- Storage: Azure Disk (RWO), Azure Files (RWX), Azure Container Storage

### 5. Azure Container Instances (ACI)
- Serverless containers, per-second billing
- No orchestration — single container or container groups
- **Choose when:** Short-lived tasks, CI/CD runners, burst workloads (can burst from AKS)

### 6. Azure Functions (Serverless)
- Event-driven, auto-scale to zero
- **Hosting plans:**
  - **Consumption:** Auto-scale, pay-per-execution, 5-min timeout (10 configurable)
  - **Flex Consumption:** New, faster cold start, VNet support, concurrency control
  - **Premium:** Pre-warmed instances, VNet, unlimited timeout
  - **Dedicated (App Service):** Run on existing App Service Plan
- Durable Functions: Stateful orchestrations (fan-out, chaining, human interaction)

### 7. Choosing the Right Compute

```
Full control over OS?
  YES → Azure VMs
  NO  → Continue...

Containers?
  YES → Kubernetes needed? → AKS
      → Short-lived/simple? → ACI
  NO  → Continue...

Web app / API?
  YES → App Service (Standard+ for production)

Event-driven / short tasks?
  YES → Azure Functions (Consumption or Premium)
```

---

## 8. Azure Arc

### What It Is
Azure Arc **projects non-Azure resources into Azure Resource Manager**, enabling you to manage on-premises servers, Kubernetes clusters, and databases using Azure tools, policies, and services — regardless of where they run.

### What Azure Arc Can Manage

| Resource Type | Description |
|---|---|
| **Arc-enabled Servers** | Windows/Linux machines on-prem, AWS, GCP — manage with Azure Policy, Defender, Monitor, Update Manager |
| **Arc-enabled Kubernetes** | Any Kubernetes cluster (on-prem, EKS, GKE) — deploy workloads via GitOps, apply Azure Policy |
| **Arc-enabled Data Services** | Run Azure SQL Managed Instance or PostgreSQL on any infrastructure |
| **Arc-enabled VMware/SCVMM** | Manage VMware vSphere or Hyper-V VMs from Azure portal |

### Key Capabilities Enabled by Arc

| Capability | What It Does |
|---|---|
| **Azure Policy (Guest Config)** | Apply and audit compliance rules on non-Azure servers |
| **Microsoft Defender for Cloud** | Threat protection for on-premises and multi-cloud servers |
| **Azure Monitor** | Collect metrics and logs from Arc servers into Log Analytics |
| **Update Manager** | Centralize OS patching across Azure + non-Azure |
| **Role-Based Access Control (RBAC)** | Use Azure RBAC to control access to on-premises resources |
| **GitOps (Kubernetes)** | Declarative Kubernetes config via Git repo using Flux |

### Exam Decision Points

| Scenario | Answer |
|---|---|
| Enforce Azure Policy on on-premises Linux servers | Azure Arc-enabled Servers |
| Use Azure Monitor to collect logs from AWS EC2 instances | Azure Arc-enabled Servers |
| Deploy workloads to on-premises Kubernetes using Azure tools | Azure Arc-enabled Kubernetes |
| Run Azure SQL Managed Instance in on-premises datacenter | Azure Arc-enabled Data Services |
| Centrally manage VMware VMs from Azure portal | Azure Arc-enabled VMware vSphere |

> **Exam tip:** Azure Arc doesn't move workloads to Azure — it brings Azure management **to** the workload wherever it runs. When a scenario asks "manage on-premises or multi-cloud servers with Azure Policy / Defender / Monitor" — the answer is **Azure Arc**.

---

## Exam Tips
- **Lift-and-shift** scenarios = VMs
- **Microservices at scale** = AKS
- **Event-driven, serverless** = Azure Functions
- **Web apps with slots** = App Service
- Know the **App Service plan tiers** — Isolated is the answer for "fully isolated network"
- AKS with Azure CNI is required for VNet-integrated pods
- Durable Functions for **long-running, stateful workflows** in serverless
- **On-premises / multi-cloud management via Azure** = Azure Arc (policy, monitoring, Defender)
