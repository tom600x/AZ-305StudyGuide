# Azure Networking — Deep-Dive Study Guide for AZ-305

## Why Networking Matters on AZ-305
Networking is embedded in almost every scenario. The exam tests your ability to:
- Design hub-spoke and secure network topologies
- Choose the right connectivity option (VPN vs ExpressRoute vs peering)
- Protect workloads with the right firewall, NSG, and DDoS strategy
- Route traffic correctly for performance, security, and availability
- Enable private connectivity for PaaS services

---

## 1. Virtual Networks (VNets)

### Core Concepts
- **Address space:** Classless Inter-Domain Routing (CIDR) blocks assigned at creation, can add more later
- **Subnets:** Subdivide the VNet; each resource lives in a subnet
- **Region-bound:** A VNet exists in one region — connect across regions via peering or VPN
- **Subscription-bound:** Can peer across subscriptions

### Subnet Planning for AZ-305
```
VNet: 10.0.0.0/16
├── GatewaySubnet        10.0.0.0/27   → Required for VPN/ExpressRoute GW
├── AzureFirewallSubnet  10.0.1.0/26   → Required for Azure Firewall
├── AzureBastionSubnet   10.0.2.0/26   → Required for Bastion (min /26)
├── AppSubnet            10.0.10.0/24  → App tier VMs / App Service
├── DataSubnet           10.0.20.0/24  → SQL, Redis, private endpoints
└── ManagementSubnet     10.0.30.0/24  → Jump boxes, monitoring
```

**Exam tip:** `GatewaySubnet`, `AzureFirewallSubnet`, `AzureBastionSubnet` are **reserved names** — Azure requires these exact names for those services.

---

## 2. Network Security Groups (NSGs)

### What They Do
- Stateful layer-4 filtering (TCP/UDP/ICMP)
- Applied to **subnets** or **individual NICs**
- Rules: Priority (100–4096), lower = higher priority
- Default rules allow VNet-inbound, Azure LB, deny-all-inbound

### NSG vs Azure Firewall

| Feature | NSG | Azure Firewall |
|---|---|---|
| Layer | L4 (port/IP) | L4 + L7 (Fully Qualified Domain Name (FQDN), URL, TLS inspection) |
| Scope | Subnet/NIC | Centralized hub |
| FQDN filtering | No | Yes |
| Threat intelligence | No | Yes |
| Cost | Free | ~$1.25/hr + data |
| Use case | Basic subnet isolation | Enterprise perimeter control |

**Exam rule:** NSGs for **subnet-level segmentation**; Azure Firewall for **centralized, policy-based control with FQDN rules**.

### Application Security Groups (ASGs)
- Group VMs by role (e.g., "web-servers", "db-servers")
- Write NSG rules referencing ASG names instead of IPs
- **Choose when:** Dynamic VM membership; avoid managing IP lists

---

## 3. Azure Firewall

### SKUs

| SKU | Features | Use Case |
|---|---|---|
| **Standard** | FQDN filtering, NAT, network rules, threat intel | Most enterprise workloads |
| **Premium** | + TLS inspection, Intrusion Detection and Prevention System (IDPS), URL categories, web categories | Zero-trust, compliance-heavy |
| **Basic** | Limited policy, no threat intel | SMB / dev environments |

### Key Capabilities
- **Destination Network Address Translation (DNAT) rules:** Inbound internet traffic → internal IP (replaces inbound NAT rules)
- **Network rules:** IP/port/protocol filtering
- **Application rules:** FQDN/URL-based (HTTP/HTTPS)
- **Threat intelligence:** Block known malicious IPs/FQDNs
- **Forced tunneling:** Route all egress through on-premises firewall

### Azure Firewall Policy vs Classic Rules
- **Firewall Policy:** Recommended — hierarchical, reusable across multiple firewalls
- **Classic rules:** Legacy, per-firewall only

---

## 4. VNet Peering

### Types

| Type | Scope | Latency | Transit |
|---|---|---|---|
| **VNet Peering** | Same region | Lowest | No (by default) |
| **Global VNet Peering** | Cross-region | Low (MS backbone) | No (by default) |

### Key Rules
- Peering is **non-transitive** — VNet A→B and B→C does NOT give A→C
- To enable transit routing: use **Azure Firewall** or **Network Virtual Appliance (NVA)** in hub, enable `Use Remote Gateways` / `Allow Gateway Transit`
- Peering is **bidirectional** but configured independently each direction
- **No overlapping address spaces** allowed

### Hub-Spoke Topology (Exam Favorite)
```
         [On-Premises]
               |
         [VPN / ER GW]
               |
           [HUB VNet]
          /     |     \
    [Spoke1] [Spoke2] [Spoke3]
    (Dev)    (Prod)   (DMZ)
```
- Hub contains: Firewall, VPN/ExpressRoute GW, Bastion, DNS
- Spokes contain: Application workloads
- Spokes peer to hub; spoke-to-spoke traffic goes through hub firewall
- **Choose when:** Centralized security, shared services, multiple workload teams

---

## 5. VPN Gateway

### Gateway SKUs (Performance)

| SKU | Max Aggregate Throughput | Max S2S Tunnels | Use Case |
|---|---|---|---|
| **Basic** | 100 Mbps | 10 | Dev/test only |
| **VpnGw1** | 650 Mbps | 30 | Small production |
| **VpnGw2** | 1 Gbps | 30 | Medium production |
| **VpnGw3** | 1.25 Gbps | 30 | High throughput |
| **VpnGw4/5** | Up to 5 Gbps | 100 | Large enterprise |
| **AZ variants** | Same + zone redundancy | Same | HA zone protection |

### VPN Types
- **Route-based VPN:** Supports Internet Key Exchange version 2 (IKEv2), point-to-site, dynamic routing — **use for new deployments**
- **Policy-based VPN:** Static routing, IKEv1 only, limited compatibility — **legacy only**

### Connection Types
- **Site-to-Site (S2S):** On-premises ↔ Azure over IPsec/IKE
- **Point-to-Site (P2S):** Individual device → Azure (OpenVPN, Secure Socket Tunneling Protocol (SSTP), IKEv2)
- **VNet-to-VNet:** Azure VNet ↔ Azure VNet (use peering if same region is simpler)

### High Availability
- **Active-Active:** Two public IPs, two tunnels — recommended for production
- **Active-Passive:** One active, failover to passive — default

---

## 6. ExpressRoute

### Connection Models

| Model | Description | Latency | Reliability |
|---|---|---|---|
| **CloudExchange Co-location** | Colocation provider's switch | Very low | High |
| **Point-to-Point Ethernet** | Dedicated WAN link | Low | High |
| **Any-to-Any (IPVPN)** | Multiprotocol Label Switching (MPLS)/VPN network integration | Variable | Provider SLA |
| **ExpressRoute Direct** | Direct 10/100 Gbps to Microsoft Enterprise Edge (MSEE) | Lowest | Highest |

### Circuit SKUs

| SKU | Scope | Use Case |
|---|---|---|
| **Local** | Same metro region | Data near a specific Azure region |
| **Standard** | One geopolitical region | Single-region enterprise |
| **Premium** | Global + Microsoft 365 | Multi-region, global enterprise |

### ExpressRoute vs VPN

| Criteria | VPN Gateway | ExpressRoute |
|---|---|---|
| Max bandwidth | 10 Gbps | 100 Gbps (Direct) |
| SLA | 99.9% | 99.95% |
| Path | Public internet (encrypted) | Private MPLS/colocation |
| Setup time | Minutes | Weeks |
| Cost | Low | High |
| **Choose when** | Dev/test, backup, lower traffic | Production, regulated, high throughput |

### ExpressRoute + VPN Coexistence
- Use VPN as **failover** for ExpressRoute — automatic failover if ER circuit goes down
- Requires both gateways in the same VNet; `GatewaySubnet` shared

### FastPath
- Bypasses the gateway for data-plane traffic (gateway still needed for control plane)
- **Use when:** Ultra-low latency required for database or real-time traffic

---

## 7. Azure Load Balancer

### SKU Comparison

| Feature | **Basic** | **Standard** |
|---|---|---|
| Scope | Single availability set/VMSS | Regional or cross-zone |
| Zone redundancy | No | Yes |
| Health probes | HTTP, TCP | HTTP, HTTPS, TCP |
| SLA | None | 99.99% |
| Outbound rules | No | Yes |
| Cost | Free | Per rule + data |
| **Status** | Being retired | Use for all new deployments |

### Load Balancer Types
- **Public LB:** Internet-facing, distributes traffic to VMs
- **Internal LB:** Private IP, distributes traffic within VNet (e.g., SQL tier)

### Key Concepts
- **Health probes:** Detect unhealthy backends — misconfiguration is a common exam trap
- **Session persistence (affinity):** None (5-tuple hash), Client IP, Client IP+Protocol
- **HA Ports rule:** Single rule for all ports/protocols — used with NVAs

### When Load Balancer Is the Right Answer
- Choose **Azure Load Balancer** for **regional Layer-4** distribution of TCP/UDP traffic
- Use it for **IaaS VMs/VMSS** inside a VNet or for internet-facing VM workloads
- It is a **passthrough** load balancer: the client connects directly to the selected backend
- It does **not** provide URL routing, TLS offload, or WAF

> **Exam tip:** If the scenario is not asking for Layer-7 inspection and is centered on VMs, protocols, ultra-low latency, or internal/private load balancing, start with **Load Balancer**.

---

## 8. Azure Application Gateway

### What It Is
- Layer-7 load balancer (HTTP/HTTPS)
- SSL termination, URL-based routing, cookie affinity, WebSocket
- Integrates with **WAF (Web Application Firewall)**

### SKUs

| SKU | WAF | Autoscaling | Zones | Use Case |
|---|---|---|---|---|
| **Standard (v1)** | No | No | No | Legacy regional web load balancing |
| **WAF (v1)** | Yes | No | No | Legacy web load balancing with WAF |
| **Basic (v2 preview)** | No | No | Limited | Lower-traffic apps with lower SLA and no advanced traffic features |
| **Standard v2** | No | Yes | Yes | Current HTTP/HTTPS LB without WAF |
| **WAF v2** | Yes (OWASP 3.2) | Yes | Yes | Current web apps requiring OWASP protection |

> **Exam tip:** `Basic` does exist for Application Gateway, but as a **v2 preview SKU** targeted at lower-traffic scenarios with fewer features and a lower SLA than `Standard v2`.
>
> **Current guidance:** Application Gateway **v1 (Standard and WAF)** is retired as of April 2026. For production design questions, prefer **Standard v2** or **WAF v2** unless a question explicitly calls out the lighter `Basic` tier.

### Basic vs Standard v2

| Feature Area | Basic (v2 preview) | Standard v2 |
|---|---|---|
| SLA | 99.9% | 99.95% |
| Autoscaling | No | Yes |
| Advanced features | No URL rewrite, mTLS, Private Link, Private-only, TCP/TLS proxy, or AKS via AGIC | Yes |
| Scale limits | Very limited | Production scale |
| Best fit | Small or lower-traffic apps | Production workloads |

### Routing Rules
- **Basic:** Route all traffic to one backend pool
- **Path-based:** `/api/*` → API pool, `/images/*` → CDN pool
- **Multi-site:** `app1.contoso.com` → Pool1, `app2.contoso.com` → Pool2

### App Gateway vs Azure Load Balancer

| | App Gateway | Load Balancer |
|---|---|---|
| Layer | L7 (HTTP/HTTPS) | L4 (TCP/UDP) |
| SSL offload | Yes | No |
| URL routing | Yes | No |
| WAF | Yes (WAF v2) | No |
| Protocols | HTTP, HTTPS, WebSocket, HTTP/2 | Any TCP/UDP |

### When Application Gateway Is the Right Answer
- Choose **Application Gateway** for **regional** web applications that need **application-layer processing per request**
- Use it when the question mentions **path-based routing**, **host-based routing**, **cookie affinity**, **TLS termination**, or **WAF**
- It is a **terminating** load balancer: the client connects to the gateway, and the gateway opens a separate connection to the backend
- It can front private backends in a VNet and is a common ingress choice for regional web workloads

> **Exam tip:** If the app is public HTTP/HTTPS but only needs to operate in one region, **Application Gateway** is usually the first service to evaluate.

---

## 9. Azure Front Door

### What It Is
- Global L7 load balancer + CDN + WAF on Microsoft's edge network
- Anycast routing to nearest edge PoP
- Automatic failover between regions

### SKUs

| SKU | CDN | WAF | Custom Rules | Use Case |
|---|---|---|---|---|
| **Standard** | Yes | Basic | Limited | Web acceleration, simple routing |
| **Premium** | Yes | Advanced (Bot protection, DDoS) | Full | Enterprise, PCI-DSS, high security |

### Front Door vs Traffic Manager

| | Front Door | Traffic Manager |
|---|---|---|
| Layer | L7 (HTTP/HTTPS) | DNS (L3) |
| Protocol | HTTP/HTTPS only | Any (DNS-based) |
| SSL offload | Yes | No |
| CDN/caching | Yes | No |
| Failover speed | Seconds (anycast) | Minutes (DNS TTL) |
| **Choose when** | Web/HTTP apps, global CDN | Non-HTTP, DNS-based routing |

### When Front Door Is the Right Answer
- Choose **Azure Front Door** for **global HTTP/HTTPS** applications with multiple regions or distributed backends
- Use it when the scenario mentions **performance acceleration**, **edge POP ingress**, **global failover**, **caching**, or **WAF at the edge**
- Front Door is the global Layer-7 entry point; it often sits **in front of** regional services like Application Gateway or Load Balancer

> **Exam tip:** Front Door is often the right global answer for internet-facing web apps, but the architecture can still need a **regional** load balancer behind it.

---

## 10. Azure Traffic Manager

### What It Is
- DNS-based global load balancer — returns the IP of the best endpoint via DNS
- Works with **any internet-facing endpoint** (Azure, on-premises, other clouds)
- No proxy — traffic flows directly from client to endpoint (not through Traffic Manager)
- Health probes check endpoint availability before returning DNS responses

### Routing Methods

| Method | How It Works | Use Case |
|---|---|---|
| **Priority** | Always route to primary; failover to secondary if primary is unhealthy | Active-passive disaster recovery |
| **Weighted** | Distribute traffic by assigned weight (e.g., 90/10) | Gradual rollout, A/B testing, canary deployment |
| **Performance** | Route to endpoint with lowest latency from client's location | Global app, minimize latency for users |
| **Geographic** | Route based on client's DNS source geography | Data sovereignty, regional content, GDPR compliance |
| **Multivalue** | Return all healthy endpoints (IPv4/IPv6 only) | Client-side load balancing, return multiple endpoints |
| **Subnet** | Map specific IP ranges to specific endpoints | Controlled routing for office/ISP ranges |

> **Exam tip:** Geographic routing is used for **data residency and compliance** — users in the EU always go to the EU endpoint regardless of performance. Performance routing is for **lowest latency**, not compliance.

### Key Characteristics
- DNS TTL controls how quickly clients pick up changes after a failover
- Nested profiles: combine routing methods (e.g., Performance outer + Priority inner per region)
- **Endpoint types:** Azure endpoints, external endpoints (any public IP/FQDN), nested Traffic Manager profiles

### When Traffic Manager Is the Right Answer
- Choose **Traffic Manager** for **global DNS-based** distribution when the endpoints can be Azure, on-premises, or other clouds
- Use it when the scenario is **non-HTTP**, or when DNS-level routing methods like **Priority**, **Performance**, or **Geographic** best match the requirement
- Traffic Manager does **not** proxy traffic and cannot inspect or transform requests

> **Exam tip:** Traffic Manager is commonly a distractor in web-app questions when the requirement actually needs WAF, TLS offload, or fast application failover. In those cases, prefer **Front Door**.

---

## 11. Load Balancing Decision Framework

Microsoft's architecture guidance groups Azure load balancing choices across two dimensions: **global vs regional** and **HTTP(S) vs non-HTTP(S)**.

### Global vs Regional

| Scope | Meaning | Primary Services |
|---|---|---|
| **Global** | Route users across regions, clouds, or hybrid endpoints | Front Door, Traffic Manager |
| **Regional** | Distribute traffic within one region or VNet | Load Balancer, Application Gateway |

### HTTP(S) vs Non-HTTP(S)

| Traffic Type | Typical Services | Why |
|---|---|---|
| **HTTP(S)** | Application Gateway, Front Door | Need Layer-7 routing, TLS offload, WAF, path/host decisions |
| **Non-HTTP(S)** | Load Balancer, Traffic Manager | Need TCP/UDP or DNS-based routing without Layer-7 processing |

### Passthrough vs Terminating

| Model | Service Examples | Meaning |
|---|---|---|
| **Passthrough** | Load Balancer | Client connects directly to backend chosen by the load balancer |
| **Terminating / Proxy** | Application Gateway, Front Door | Client connects to the load balancer, which creates a new backend connection |

### Fast Selection Rules

1. **Regional VM or TCP/UDP workload** -> Load Balancer
2. **Regional web app needing WAF, TLS offload, or URL routing** -> Application Gateway
3. **Global web app needing acceleration or fast failover** -> Front Door
4. **Global DNS-based routing for any endpoint type** -> Traffic Manager
5. **API-only backend already using APIM** -> APIM can load balance API backends, but do not choose it solely as a general-purpose load balancer

### Layered Architectures Are Common

AZ-305 often expects a combination, not a single product:

- **Front Door + Application Gateway**: global web routing at the edge plus regional WAF and routing
- **Front Door + Load Balancer**: global HTTP entry plus regional Layer-4 distribution to VMs
- **Traffic Manager + Application Gateway**: DNS-based regional failover with regional Layer-7 inspection

> **Exam tip:** When the workload has multiple tiers or multiple regions, evaluate each traffic hop separately. One architecture can legitimately need both a global and a regional load balancing service.


## 12. Azure DNS & Private DNS

### Azure DNS (Public)
- Host public DNS zones in Azure
- Integrated with Role-Based Access Control (RBAC), Azure Monitor
- No DNSSEC support (as of 2026)

### Azure Private DNS
- Resolve names within a VNet without custom DNS servers
- **Auto-registration:** VMs auto-register A records when linked with auto-registration enabled
- Link VNets to zones for resolution

### Private DNS Zones for Private Endpoints (Critical)
Each PaaS service needs a specific private DNS zone:

| Service | Private DNS Zone |
|---|---|
| Blob Storage | `privatelink.blob.core.windows.net` |
| File Storage | `privatelink.file.core.windows.net` |
| SQL Database | `privatelink.database.windows.net` |
| Key Vault | `privatelink.vaultcore.azure.net` |
| App Service | `privatelink.azurewebsites.net` |
| Cosmos DB | `privatelink.documents.azure.com` |

**Exam tip:** In a hub-spoke topology, host private DNS zones in the **hub** and link all spoke VNets to them.

---

## 13. Private Endpoints & Service Endpoints

### Service Endpoints
- Extends VNet identity to the PaaS service
- Traffic stays on Microsoft backbone but **PaaS service still has public IP**
- Simple, no extra IP allocation, low cost
- **Choose when:** Simple PaaS protection without private IP requirement

### Private Endpoints
- Injects PaaS service into VNet with a **private IP**
- Disables public internet access to the service
- Requires Private DNS zone configuration
- **Choose when:** Zero-trust, compliance, PaaS fully private

### Comparison

| | Service Endpoint | Private Endpoint |
|---|---|---|
| Private IP in VNet | No | Yes |
| Public IP removed | No | Yes (optional) |
| DNS change needed | No | Yes |
| Cross-region | No | Yes |
| Cost | Free | Per hour + data |
| **Choose when** | Simple, same-region PaaS | Full private, compliance |

---

## 14. Azure Bastion

### What It Is
- Managed jump box — RDP/SSH via browser over TLS 443
- No public IP on VMs required
- Eliminates exposure of management ports (22, 3389) to internet

### SKUs

| SKU | Features | Use Case |
|---|---|---|
| **Basic** | RDP/SSH via portal only | Simple use |
| **Standard** | Native client, tunneling, IP-based, host scaling | Enterprise, DevOps tooling |
| **Premium** | + Private-only deployment, session recording | High security |

**Subnet requirement:** `/26` minimum, named `AzureBastionSubnet`

---

## 15. Network Watcher & Monitoring

| Tool | Purpose |
|---|---|
| **IP Flow Verify** | Test if NSG allows/denies specific traffic |
| **Next Hop** | Trace routing path from a VM |
| **Connection Monitor** | Continuous connectivity health between endpoints |
| **NSG Flow Logs** | Log all NSG-allowed/denied flows → Storage or Log Analytics |
| **Traffic Analytics** | Visual analysis of NSG flow logs |
| **Packet Capture** | Remote PCAP on Azure VMs |

---

## 16. DDoS Protection

| Plan | Protection | Use Case |
|---|---|---|
| **Network Protection** (formerly Standard) | Adaptive tuning, rapid response team, cost protection | Production public-facing apps |
| **IP Protection** | Per-public-IP protection, no rapid response | Smaller/selective deployments |
| **Basic (Infrastructure)** | Always-on, limited | All Azure resources, free |

**Exam tip:** DDoS Network Protection is applied at the **VNet level** and covers all public IPs in that VNet. Layer 7 protection still requires **WAF** (App Gateway or Front Door).

---

## 17. Routing

### Route Priority Order
1. System routes (auto-created)
2. User-defined routes (UDRs)
3. Border Gateway Protocol (BGP) routes (from on-premises via VPN/ER)

### User-Defined Routes (UDRs)
- Override system routes
- Force traffic through NVA or Azure Firewall
- **Forced tunneling:** UDR with `0.0.0.0/0` → NVA/Firewall → on-premises

### Border Gateway Protocol (BGP)
- Dynamic routing protocol used with VPN Gateway (Route-based) and ExpressRoute
- Enables automatic route propagation from on-premises
- Required for ExpressRoute

---

## 18. Azure Virtual WAN (vWAN)

### What It Is
Azure Virtual WAN is a **networking service that provides any-to-any connectivity at scale** — branches, VNets, and remote users connected through a Microsoft-managed hub. Replaces manual hub-spoke topologies for large enterprises.

### SKUs

| SKU | S2S VPN | P2S VPN | ExpressRoute | VNet-to-VNet | NVA in Hub | Use Case |
|---|---|---|---|---|---|---|
| **Basic** | Yes | No | No | No | No | S2S VPN only, simple branch connectivity |
| **Standard** | Yes | Yes | Yes | Yes | Yes | Full enterprise hub, all connectivity types |

### Key Concepts

| Concept | Description |
|---|---|
| **Virtual Hub** | Microsoft-managed regional hub — replaces your own hub VNet |
| **Hub VNet connection** | Connect spoke VNets to the hub |
| **Branch connectivity** | S2S VPN, ExpressRoute, or SD-WAN partner devices into the hub |
| **Any-to-any routing** | All connected branches and VNets can communicate through the hub |
| **Secured Virtual Hub** | Virtual WAN hub with Azure Firewall deployed inside |

### Virtual WAN vs Manual Hub-Spoke

| | Manual Hub-Spoke | Azure Virtual WAN |
|---|---|---|
| Management | You manage all peerings, GWs, routing tables | Microsoft manages the hub infrastructure |
| Scale | Limited by VNet peering limits and manual config | Thousands of branches and VNets |
| Any-to-any | Manual UDRs required for spoke-to-spoke | Built-in |
| SD-WAN partners | Not integrated | Native partner integrations (Barracuda, Cisco, etc.) |
| **Choose when** | Small/medium, fine-grained control | Large enterprise, many branches, SD-WAN |

> **Exam tip:** When the scenario describes **many branch offices, SD-WAN, or simplifying large-scale hub-spoke**, the answer is Azure Virtual WAN Standard. Manual hub-spoke is appropriate when you need full control over routing or have a small number of VNets.

---

## 19. Networking Topology Decision Guide

```
Connectivity to on-premises?
├── Dev/test, low cost → VPN Gateway (Route-based, Active-Active for prod)
├── Production, compliance, high throughput → ExpressRoute
└── Both → ER primary + VPN failover

Hub-spoke at enterprise scale?
├── Many branches, SD-WAN, any-to-any → Azure Virtual WAN Standard
└── Small/medium, fine-grained control → Manual hub-spoke + Azure Firewall

Multi-region web app?
├── HTTP/HTTPS + CDN + global failover → Azure Front Door
└── DNS-based, non-HTTP → Traffic Manager

Load balancing within a region?
├── HTTP/HTTPS, URL routing, WAF → Application Gateway WAF v2
└── TCP/UDP, non-HTTP, internal → Standard Load Balancer

PaaS connectivity?
├── Simple, same-region, budget → Service Endpoint
└── Private IP, compliance, cross-region → Private Endpoint

Network security?
├── Subnet/NIC isolation → NSG
├── Centralized enterprise policy + FQDN → Azure Firewall (Standard)
└── TLS inspection, IDPS, zero-trust → Azure Firewall Premium

Remote management?
└── Bastion (Standard SKU for native client support)
```

---

## 20. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Isolate web tier from DB tier at L4 | NSG on each subnet |
| Block specific malicious FQDNs at egress | Azure Firewall with threat intel |
| Inspect inbound HTTPS for OWASP attacks | App Gateway WAF v2 |
| Route all traffic through on-premises firewall | UDR `0.0.0.0/0` → NVA + forced tunneling |
| Global HTTP app, lowest latency, CDN, WAF | Azure Front Door Premium |
| Non-HTTP multi-region failover | Traffic Manager (Priority routing) |
| EU users must stay on EU endpoint (data residency) | Traffic Manager Geographic routing |
| Gradual rollout, send 10% traffic to new region | Traffic Manager Weighted routing |
| Global app, route users to lowest-latency endpoint | Traffic Manager Performance routing |
| Connect 10 spoke VNets sharing one ExpressRoute | Hub-spoke with gateway transit |
| 50 branch offices, SD-WAN, any-to-any connectivity | Azure Virtual WAN Standard |
| Prevent spokes from talking to each other | NSG + Azure Firewall in hub, no spoke-to-spoke peering |
| Securely access Azure SQL from VNet, no public IP | Private Endpoint + Private DNS zone |
| Protect all public IPs in a VNet from DDoS L3/L4 | DDoS Network Protection on VNet |
| RDP to VMs without exposing port 3389 | Azure Bastion Standard |
| 10 Gbps dedicated private circuit to Azure | ExpressRoute (Standard or Premium) |
| Dev/test site-to-site VPN, minimize cost | VPN Gateway Basic SKU |
| VM needs to reach internet, no public IP | NAT Gateway on subnet |

---

## 21. Key Limits to Know

| Resource | Limit |
|---|---|
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| VNet peerings per VNet | 500 |
| NSG rules per NSG | 1,000 |
| VPN Gateway max S2S tunnels (VpnGw5) | 100 |
| ExpressRoute Direct bandwidth options | 10 Gbps, 100 Gbps |
| Virtual WAN hubs per vWAN | 1 per region (multiple regions supported) |
| App Gateway v2 max instances (autoscale) | 125 |
| Front Door PoPs | 100+ globally |

---

## 22. Likely Exam Gaps to Review

These are the networking decisions that most often separate correct answers from plausible distractors.

### High-Yield Decision Pairs

| Decision | Choose This When | Re-check This Distractor |
|---|---|---|
| **Private Endpoint** | Private IP is required for a PaaS service | Service Endpoint only keeps the service public |
| **Service Endpoint** | You need simpler VNet-based restriction without private IP requirements | Private Endpoint when data exfiltration or DNS control matters |
| **ExpressRoute** | Dedicated private connectivity, predictable performance, enterprise hybrid design | VPN Gateway for lower-cost or faster setup |
| **VPN Gateway** | Cost-sensitive or smaller hybrid connectivity requirements | ExpressRoute when SLA and throughput requirements are strict |
| **Application Gateway** | Layer 7 regional load balancing, WAF, path-based routing | Load Balancer for non-HTTP traffic |
| **Front Door** | Global HTTP/HTTPS entry point, WAF, acceleration, CDN-like edge presence | Traffic Manager when only DNS routing is needed |
| **Traffic Manager** | DNS-based global routing for HTTP or non-HTTP endpoints | Front Door when you need TLS termination or WAF at the edge |

### Private Connectivity Checklist

Before answering a networking question, confirm whether the scenario requires:

1. No public endpoint at all.
2. Name resolution through Private DNS.
3. Inspection of outbound or inbound traffic.
4. Hybrid connectivity from branches or datacenters.
5. Regional vs global failover.

If the question mentions secure access to Azure SQL, Storage, Key Vault, or App Service from inside a VNet, start by testing **Private Endpoint + Private DNS** as the leading answer.

### Practice Prompts

1. A global web app needs WAF, TLS termination, and low-latency routing. Why is Front Door a better fit than Traffic Manager?
2. A company needs private access from VNets to Azure SQL with no public exposure. Why is Private Endpoint better than Service Endpoint?
3. A hub-spoke network must support many branches and simplify routing at scale. Why might Virtual WAN be better than manually built hub-spoke?
4. A workload needs inspection of HTTP traffic plus path-based routing in one region. Why is Application Gateway a better fit than Azure Load Balancer?

---

## 23. Load Balancing Exam Traps

These are common AZ-305 distractors that show up in practice questions.

### 1. Choosing Traffic Manager for a web app that needs WAF or TLS offload
- **Wrong because:** Traffic Manager is DNS-based and does not inspect or terminate HTTP(S)
- **Usually right answer:** Front Door for global web apps, or Application Gateway for regional web apps

### 2. Choosing Load Balancer for HTTP path-based routing
- **Wrong because:** Load Balancer is Layer 4 only
- **Usually right answer:** Application Gateway if the requirement mentions URL path, host-based routing, cookies, or WAF

### 3. Choosing Application Gateway for non-HTTP protocols by default
- **Wrong because:** Most App Gateway questions on AZ-305 are really about Layer-7 web delivery, not generic TCP/UDP balancing
- **Usually right answer:** Load Balancer for general TCP/UDP scenarios unless the question explicitly needs App Gateway TCP/TLS proxy behavior

### 4. Forgetting that global and regional load balancing can both be required
- **Wrong because:** Many architectures need one service at the edge and another inside the region
- **Usually right answer:** Front Door plus Application Gateway or Front Door plus Load Balancer

### 5. Confusing fastest failover with DNS-based failover
- **Wrong because:** DNS caching slows Traffic Manager failover
- **Usually right answer:** Front Door when the scenario emphasizes fast failover or better user experience during regional issues

### 6. Picking Front Door for non-HTTP workloads
- **Wrong because:** Front Door is HTTP/HTTPS only
- **Usually right answer:** Traffic Manager for global DNS-based routing or Load Balancer for regional TCP/UDP workloads

### 7. Treating API Management as a general-purpose load balancer
- **Wrong because:** APIM is primarily an API gateway
- **Usually right answer:** Use APIM load balancing only when the workload is already API-centric and APIM is already justified for gateway features

### 8. Ignoring hosting model in the answer
- **Wrong because:** IaaS VM workloads often point toward Load Balancer, while web app delivery requirements point toward Application Gateway or Front Door
- **Usually right answer:** Match the load balancing choice to whether the backend is IaaS, PaaS, AKS, or API-only

### 9. Choosing the most powerful service instead of the least-complex service that fits
- **Wrong because:** AZ-305 often rewards the design that meets requirements with lower operational complexity
- **Usually right answer:** Prefer the simplest option that still satisfies scope, protocol, security, and failover requirements

### 10. Solving the whole workload with one product
- **Wrong because:** Different hops in the same architecture can have different requirements
- **Usually right answer:** Evaluate internet ingress, regional distribution, and private backend connectivity separately

### Rapid Elimination Rules

If you see this requirement, eliminate these distractors first:

| Requirement | Eliminate First |
|---|---|
| WAF, TLS offload, URL path routing | Load Balancer, Traffic Manager |
| Non-HTTP protocol | Front Door |
| Global web acceleration | Application Gateway alone |
| Regional internal VM balancing | Front Door, Traffic Manager |
| Private PaaS connectivity | Service Endpoint if private IP is mandatory |
