# Compute Solutions — AZ-305 Study Guide

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

#### Availability Options

| Option | Protects Against | SLA | Use Case |
|---|---|---|---|
| **Availability Sets** | Rack / power / switch failure (same datacenter) | 99.95% | Legacy deployments, single-datacenter HA |
| **Availability Zones** | Datacenter failure (3 physical zones per region) | 99.99% | Recommended for all new production VMs |
| **VMSS across Zones** | Node failure + auto-scale across zones | 99.95%–99.99% | Stateless workloads needing elasticity |

#### VM Cost & Licensing Options

| Option | Savings | Commitment | Best For |
|---|---|---|---|
| **Pay-as-you-go** | Baseline | None | Dev/test, unpredictable workloads |
| **Spot VMs** | Up to 90% | None (evictable with 30-sec notice) | Batch, ML training, fault-tolerant tasks |
| **Reserved Instances (RI)** | Up to 72% | 1 or 3 years | Stable, long-running production workloads |
| **Azure Hybrid Benefit** | Up to 49% | None | Existing Windows Server / SQL Server licenses |
| **Dev/Test pricing** | Significant | Active Visual Studio subscription | Non-production environments only |
| **Azure Dedicated Host** | No discount | Per host/hour | Physical isolation for GDPR, HIPAA compliance |

> **Exam tip:** Proximity Placement Groups co-locate VMs in the same datacenter to minimize inter-VM latency — use for tightly coupled HPC or latency-sensitive applications.

#### Managed Disks

| Disk Type | Max IOPS | Max Throughput | Use Case |
|---|---|---|---|
| **Standard HDD** | 2,000 | 500 MB/s | Backup, cold data, dev/test |
| **Standard SSD** | 6,000 | 750 MB/s | Web servers, lightly used apps |
| **Premium SSD** | 20,000 | 900 MB/s | Production databases — required for 99.9% VM SLA |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | High-throughput databases (MySQL, SQL Server) |
| **Ultra Disk** | 160,000 | 4,000 MB/s | Extreme IOPS — SAP HANA, highest-tier workloads |

> **Exam tip:** A single VM qualifies for the **99.9% SLA** only when **all disks** are **Premium SSD or better**. Standard HDD/SSD disqualifies the SLA.

### 2. Azure Virtual Machine Scale Sets (VMSS)

#### Orchestration Modes

| Mode | Description | Use Case |
|---|---|---|
| **Uniform** | All VMs use identical image and config | Stateless, homogeneous workloads (web front-ends) |
| **Flexible** | Mix of VM types; each VM managed individually | Mixed workloads — preferred for new deployments |

#### Scaling Policies

| Policy | Trigger | Notes |
|---|---|---|
| **Metric-based** | CPU %, memory, custom Azure Monitor metric | Scale out when CPU > 70% for 5 min |
| **Schedule-based** | Time of day or week | Pre-scale for known peak periods |
| **Predictive** | ML model from historical patterns (Premium Autoscale) | Pre-scales before anticipated load arrives |

- **Health extension + automatic repairs:** Replaces unhealthy VMs automatically
- **Overprovisioning:** Creates extra VMs, then deletes the slowest — faster scale-out at no extra cost
- Max instances: 1,000 (Uniform) or 600 (Flexible) per scale set

### 3. Azure App Service

PaaS for web apps, REST APIs, and mobile backends. Supported runtimes: .NET, Node.js, Python, Java, PHP, Ruby, custom containers.

#### App Service Plan Tiers

| Tier | SLA | Autoscale | Slots | VNet Integration | Use Case |
|---|---|---|---|---|---|
| **Free / Shared** | No SLA | No | No | No | Dev/test only — shared infrastructure |
| **Basic** | 99.95% | No | No | No | Simple apps, no scale required |
| **Standard** | 99.95% | Yes (10 instances) | 5 | Outbound VNet Integration | Standard production workloads |
| **Premium v3** | 99.95% | Yes (30 instances) | 20 | Outbound + zone redundancy | High-scale production |
| **Isolated v2** | 99.95% | Yes | 20 | Full VNet injection (ASE) | Compliance, fully private, extreme scale |

#### App Service Environment (ASE)
- Fully isolated, single-tenant deployment into **your own VNet** — no shared infrastructure
- **Internal (ILB ASE):** No public IP; accessible only within VNet or via private peering
- **External ASE:** Public-facing with dedicated IP
- **Choose when:** Regulatory compliance requires dedicated compute, private inbound/outbound networking, or scale beyond Premium limits

#### Key Features

| Feature | Detail |
|---|---|
| **Deployment slots** | Swap staging → production with pre-warm; sticky settings are not swapped |
| **VNet Integration** | Routes outbound traffic into VNet — Standard tier and above |
| **Private Endpoint** | Inbound private IP for the app — removes public exposure |
| **Auto-heal** | Auto-restart on HTTP errors, memory limits, or slow request threshold |
| **Always On** | Keeps app warm; required for WebJobs — Basic tier and above |
| **Custom domains + TLS** | Bring your own domain and cert; free managed certificates available |

### 4. Azure Kubernetes Service (AKS)

Managed Kubernetes — Microsoft manages the control plane at no cost; you pay for agent (worker) nodes only.

#### AKS Tiers

| Tier | API Server SLA | Features | Use Case |
|---|---|---|---|
| **Free** | No SLA | Basic managed cluster | Dev/test, learning |
| **Standard** | 99.5% | Uptime SLA, cluster autoscaler | Production workloads |
| **Premium** | 99.95% | Long-Term Support (LTS) Kubernetes versions | Enterprise, compliance |

#### Node Pools

| Pool Type | Purpose | OS |
|---|---|---|
| **System** | Critical system pods (CoreDNS, metrics-server, kube-proxy) | Linux only |
| **User** | Application workloads | Linux or Windows |

- Multiple user node pools allow mixing VM sizes (GPU for ML, high-memory for databases, burstable for low-traffic)
- **Spot node pools:** Lower-cost evictable nodes for fault-tolerant batch or ML workloads within AKS

#### Scaling

| Method | Scales | Trigger |
|---|---|---|
| **Horizontal Pod Autoscaler (HPA)** | Pod replicas | CPU / memory or custom metrics |
| **Cluster Autoscaler** | Node count | Unschedulable pods (insufficient cluster capacity) |
| **KEDA** | Pod replicas | External event sources — queues, topics, HTTP, cron |

#### Networking

| Model | Pod IPs | VNet Integration | Best For |
|---|---|---|---|
| **Kubenet** | Overlay (not VNet IPs) | Partial — nodes in VNet | Simple clusters, IP-constrained environments |
| **Azure CNI** | VNet subnet IPs | Full — pods directly addressable in VNet | Private endpoints, NSG rules on pods |
| **Azure CNI Overlay** | Overlay, nodes use VNet IPs | Full for nodes | Saves IP space with full node-level VNet access |

> **Exam tip:** Azure CNI (or Overlay) is required when pods need to reach on-premises resources via ExpressRoute/VPN, or when using Private Endpoints from the pod network.

#### Security & Storage

| Area | Feature |
|---|---|
| **Authentication** | Entra ID integration for kubectl and Azure portal access |
| **Authorization** | Kubernetes RBAC + Azure RBAC (can use both simultaneously) |
| **Workload Identity** | Pods authenticate to Azure services via managed identity (replaces pod identity) |
| **Azure Policy add-on** | Enforce pod security standards — no privileged containers, required labels, etc. |
| **Defender for Containers** | Runtime threat detection, vulnerability scanning for images and running pods |
| **Storage** | Azure Disk (RWO — single pod), Azure Files (RWX — multi-pod share), Azure Container Storage |

- **Private cluster:** API server endpoint only reachable within VNet — no public API server endpoint

### 5. Azure Container Instances (ACI)
- Serverless containers — no VM or cluster management required
- Per-second billing (CPU + memory)
- No persistent orchestration — single containers or **container groups** (multi-container pods)
- Supports both Linux and Windows containers
- **Virtual nodes (AKS + ACI):** Burst AKS pods into ACI during traffic spikes without adding cluster nodes

**Choose ACI when:** Short-lived tasks, CI/CD build runners, event-driven jobs, or burst capacity alongside AKS

### 6. Azure Functions (Serverless)

Event-driven compute that scales to zero. Ideal for short-lived tasks triggered by events, timers, queues, or HTTP.

#### Hosting Plans

| Plan | Cold Start | VNet Support | Timeout | Use Case |
|---|---|---|---|---|
| **Consumption** | Yes | No (outbound only via VNet integration) | 5 min (10 configurable) | True serverless, pay-per-execution |
| **Flex Consumption** | Faster cold start | Yes | Configurable | Serverless with VNet + concurrency control |
| **Premium** | No (pre-warmed) | Yes (full) | Unlimited | Low-latency, VNet-required, always-warm |
| **Dedicated (App Service)** | No | Yes (if plan has it) | Unlimited | Run on existing App Service plan |

#### Durable Functions
Stateful orchestration patterns built on top of Azure Functions:

| Pattern | Description | Example |
|---|---|---|
| **Function chaining** | Sequential execution, output of one feeds next | Multi-step order processing |
| **Fan-out / Fan-in** | Parallel execution, aggregate results | Parallel file processing |
| **Async HTTP** | Long-running operation with status polling | Document generation |
| **Human interaction** | Wait for external approval/input | Multi-step approval workflow |
| **Monitor** | Periodic polling until condition met | Job status checker |

### 7. Choosing the Right Compute

```
Need full OS control / custom kernel / VM licenses?
  YES → Azure VMs (Spot for batch, Reserved for steady-state)

Running containers?
  YES → Need orchestration / auto-scaling / service mesh?
          YES → AKS (Standard or Premium tier for production)
          NO  → Short-lived or burst?
                  YES → ACI (or ACI via AKS virtual nodes)
                  NO  → AKS still preferred (predictable workload)

Web app / REST API / mobile backend?
  YES → App Service
        Need VNet injection / compliance isolation? → Isolated v2 (ASE)
        Need high scale + slots? → Premium v3
        Standard production → Standard plan

Event-driven / queue-triggered / short tasks (< 10 min)?
  YES → Azure Functions (Consumption or Flex Consumption)
        Need always-warm + VNet? → Premium plan

Batch / HPC / parallel large-scale processing?
  YES → Azure Batch

On-premises or multi-cloud servers to manage via Azure?
  YES → Azure Arc
```

---

## 8. Azure Batch

### What It Is
Azure Batch runs **large-scale parallel and high-performance computing (HPC) workloads** without managing infrastructure. You define jobs and tasks; Batch automatically provisions, scales, and tears down compute nodes.

### Key Concepts

| Concept | Description |
|---|---|
| **Batch Account** | Top-level resource — linked to a storage account for task I/O |
| **Pool** | Set of compute nodes (VMs) — configured with OS, VM size, autoscale |
| **Job** | Container for a collection of tasks |
| **Task** | Unit of work — runs a command or executable on a node |
| **Job Manager task** | Controls job lifecycle, can submit child tasks dynamically |

### Pool Configuration

| Setting | Options / Notes |
|---|---|
| **Node type** | Dedicated (reserved) or Spot/Low-priority (up to 80% cheaper, evictable) |
| **Autoscale** | Formula-based (custom or built-in) — scale based on pending tasks |
| **Start task** | Runs on each node when it joins the pool (install software, mount storage) |
| **Application packages** | Deploy zipped applications to nodes automatically |
| **VNet integration** | Nodes can join a VNet for private access to data sources |

### Exam Decision Points

| Scenario | Answer |
|---|---|
| Render thousands of frames in parallel, minimize cost | Azure Batch with Spot nodes |
| Run genome sequencing on 10,000 samples simultaneously | Azure Batch |
| Need managed containers without VMs for short tasks | ACI or Azure Functions |
| HPC workload requiring MPI communication between nodes | Azure Batch (supports InfiniBand, HPC VM sizes) |

> **Exam tip:** Azure Batch is the answer when the scenario involves **massively parallel, embarrassingly parallel, or HPC workloads** — especially rendering, simulation, transcoding, or scientific computing.

---

## 9. Azure Arc

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

## Exam Scenario Cheat Sheet

| Scenario | Best Compute Choice | Key Reason |
|---|---|---|
| Lift-and-shift on-premises Windows Server | Azure VMs | Full OS control, existing licenses (Hybrid Benefit) |
| Web app needing blue-green deployments and autoscale | App Service Standard/Premium | Deployment slots + autoscale |
| Web app requiring fully private network, compliance isolation | App Service Isolated v2 (ASE) | VNet-injected, no shared infra |
| Microservices with 10+ teams, mixed languages, auto-scaling | AKS | Kubernetes orchestration at scale |
| AKS pods need to reach on-prem via ExpressRoute | AKS with Azure CNI | Pods need VNet-routable IPs |
| Burst extra capacity on top of AKS during traffic spikes | ACI with virtual nodes | No new cluster nodes required |
| Process queue messages, scale to zero, run < 10 min | Azure Functions (Consumption) | Pay-per-execution, event-driven |
| Functions need VNet connectivity and no cold starts | Azure Functions Premium | Pre-warmed + VNet support |
| Long-running stateful workflow (human approval step) | Durable Functions | Stateful orchestration pattern |
| Render 100,000 video frames in parallel, minimize cost | Azure Batch with Spot nodes | HPC parallel workloads |
| Manage on-premises Linux servers with Azure Policy | Azure Arc-enabled Servers | Projects servers into ARM |
| Run Azure SQL MI on-premises | Azure Arc-enabled Data Services | Azure data services anywhere |
| Single VM needing highest availability SLA | VMs + Availability Zones | 99.99% with zone redundancy |
| Dev/test VM with low average CPU but occasional bursts | B-Series (Burstable) VMs | CPU credit model |
| Deploy container groups without managing clusters | ACI | Serverless containers |

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

---

## 10. Compute Exam Traps

### 1. Choosing VMs for every migrated app
- **Trap:** Lift-and-shift is familiar and seems safest
- **Better default:** Use PaaS or serverless when the scenario values less management, faster scaling, or built-in deployment features

### 2. Choosing AKS when orchestration requirements are light
- **Trap:** AKS is the most capable compute platform in the list
- **Better default:** Container Apps, App Service, or ACI when cluster-level control is unnecessary

### 3. Choosing Functions for long-running or VNet-heavy workloads without checking the plan
- **Trap:** Functions sound right for all event-driven scenarios
- **Better default:** Premium or Durable Functions when you need VNet support, no cold starts, or orchestration

### 4. Ignoring App Service plan tier limitations
- **Trap:** App Service is correct, but the selected tier lacks slots, autoscale, or isolation
- **Better default:** Match the feature to the tier, especially for deployment slots and ASE isolation

### 5. Forgetting Azure Arc for hybrid management questions
- **Trap:** The question is about servers or Kubernetes running outside Azure, so Azure-native compute looks wrong
- **Better default:** Azure Arc when the real requirement is governance, monitoring, or Defender across on-premises and multi-cloud

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Minimal ops for a web app | VM-first answers |
| Full Kubernetes control not required | AKS-first answers |
| Scale to zero and event-driven execution | Always-on VM or AKS answers |
| On-premises servers managed with Azure Policy | Native Azure-only compute answers |
| Blue-green or deployment slots | App Service Basic answers |
