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

---

## 8. Azure Application Gateway

### What It Is
- Layer-7 load balancer (HTTP/HTTPS)
- SSL termination, URL-based routing, cookie affinity, WebSocket
- Integrates with **WAF (Web Application Firewall)**

### SKUs

| SKU | WAF | Autoscaling | Zones | Use Case |
|---|---|---|---|---|
| **Standard v2** | No | Yes | Yes | HTTP/HTTPS LB without WAF |
| **WAF v2** | Yes (OWASP 3.2) | Yes | Yes | Web apps requiring OWASP protection |

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

---

## 10. Azure DNS & Private DNS

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

## 11. Private Endpoints & Service Endpoints

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

## 12. Azure Bastion

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

## 13. Network Watcher & Monitoring

| Tool | Purpose |
|---|---|
| **IP Flow Verify** | Test if NSG allows/denies specific traffic |
| **Next Hop** | Trace routing path from a VM |
| **Connection Monitor** | Continuous connectivity health between endpoints |
| **NSG Flow Logs** | Log all NSG-allowed/denied flows → Storage or Log Analytics |
| **Traffic Analytics** | Visual analysis of NSG flow logs |
| **Packet Capture** | Remote PCAP on Azure VMs |

---

## 14. DDoS Protection

| Plan | Protection | Use Case |
|---|---|---|
| **Network Protection** (formerly Standard) | Adaptive tuning, rapid response team, cost protection | Production public-facing apps |
| **IP Protection** | Per-public-IP protection, no rapid response | Smaller/selective deployments |
| **Basic (Infrastructure)** | Always-on, limited | All Azure resources, free |

**Exam tip:** DDoS Network Protection is applied at the **VNet level** and covers all public IPs in that VNet. Layer 7 protection still requires **WAF** (App Gateway or Front Door).

---

## 15. Routing

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

## 16. Networking Topology Decision Guide

```
Connectivity to on-premises?
├── Dev/test, low cost → VPN Gateway (Route-based, Active-Active for prod)
├── Production, compliance, high throughput → ExpressRoute
└── Both → ER primary + VPN failover

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

## 17. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Isolate web tier from DB tier at L4 | NSG on each subnet |
| Block specific malicious FQDNs at egress | Azure Firewall with threat intel |
| Inspect inbound HTTPS for OWASP attacks | App Gateway WAF v2 |
| Route all traffic through on-premises firewall | UDR `0.0.0.0/0` → NVA + forced tunneling |
| Global HTTP app, lowest latency, CDN, WAF | Azure Front Door Premium |
| Non-HTTP multi-region failover | Traffic Manager (Priority routing) |
| Connect 10 spoke VNets sharing one ExpressRoute | Hub-spoke with gateway transit |
| Prevent spokes from talking to each other | NSG + Azure Firewall in hub, no spoke-to-spoke peering |
| Securely access Azure SQL from VNet, no public IP | Private Endpoint + Private DNS zone |
| Protect all public IPs in a VNet from DDoS L3/L4 | DDoS Network Protection on VNet |
| RDP to VMs without exposing port 3389 | Azure Bastion Standard |
| 10 Gbps dedicated private circuit to Azure | ExpressRoute (Standard or Premium) |
| Dev/test site-to-site VPN, minimize cost | VPN Gateway Basic SKU |
| VM needs to reach internet, no public IP | NAT Gateway on subnet |

---

## 18. Key Limits to Know

| Resource | Limit |
|---|---|
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| VNet peerings per VNet | 500 |
| NSG rules per NSG | 1,000 |
| VPN Gateway max S2S tunnels (VpnGw5) | 100 |
| ExpressRoute Direct bandwidth options | 10 Gbps, 100 Gbps |
| App Gateway v2 max instances (autoscale) | 125 |
| Front Door PoPs | 100+ globally |
