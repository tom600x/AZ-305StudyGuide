# Identity, Governance & Monitoring — Deep-Dive Study Guide for AZ-305

> **Exam Weight:** ~25% — one of the highest-weighted domains

---

## 1. Microsoft Entra ID (Azure Active Directory)

### Core Concepts
- **Tenant:** A dedicated Entra ID instance for your organization
- **Directory:** Stores users, groups, applications, and devices
- **Subscription ↔ Tenant:** A subscription trusts exactly one tenant; a tenant can manage multiple subscriptions

### Entra ID Identity Types

| Type | Description | Use Case |
|---|---|---|
| **User** | Human identities | Employees, admins |
| **Service Principal** | App registration identity | Applications, automation scripts |
| **Managed Identity** | Auto-managed service principal for Azure resources | Azure services accessing other Azure services |
| **Guest User (B2B)** | External user invited to your tenant | Partner collaboration |

### Managed Identity — System vs User-Assigned

| | System-Assigned | User-Assigned |
|---|---|---|
| Lifecycle | Tied to resource (deleted with resource) | Independent resource |
| Sharing | One resource only | Can be shared across multiple resources |
| Use Case | Single resource needs identity | Multiple resources share same permissions |

> **Exam tip:** Managed Identity is almost always preferred over stored credentials or service principal secrets. No credential management needed.

### External Identities

| | Azure AD B2B | Azure AD B2C |
|---|---|---|
| **Purpose** | Partner/vendor collaboration | Customer identity (external users) |
| **Identity source** | Guest users from any Identity Provider (IdP) | Local accounts + social IdPs (Google, Facebook) |
| **Tenant** | Invited into your tenant | Separate B2C tenant |
| **Use Case** | Share internal apps with partners | Consumer-facing apps |

### Conditional Access
Policies that enforce access controls based on signals:

| Signal | Examples |
|---|---|
| **User/Group** | Specific users, roles, guests |
| **Device** | Compliant, Hybrid Entra ID joined |
| **Location** | Named locations, IP ranges, countries |
| **App** | Specific cloud apps |
| **Risk** | Sign-in risk (Identity Protection) |

**Common Policy Configurations:**
- Require MFA for all users accessing sensitive apps
- Block access from non-compliant devices
- Require compliant device for Intune-managed access
- Block legacy authentication protocols

### Identity Protection
- **User risk:** Probability that identity is compromised (leaked credentials, unusual activity)
- **Sign-in risk:** Probability that specific sign-in is not the owner
- **Risk-based Conditional Access:** Automatically require MFA or block on high risk
- Requires Entra ID P2 license

---

## 2. Hybrid Identity

### Sync Options

| Method | Description | When to Use |
|---|---|---|
| **Password Hash Sync (PHS)** | Hash of password hash synced to Entra ID | Simplest, resilient (works even if on-premises is down) |
| **Pass-Through Authentication (PTA)** | Authentication request forwarded to on-premises AD | Need on-premises password validation, no hash sync |
| **AD FS Federation** | Federated trust with on-premises ADFS | Complex claims, third-party MFA, legacy apps |

### Decision Rule
- No complex requirements → **PHS** (simplest, most resilient)
- Must validate passwords on-premises → **PTA**
- Complex claims-based auth, third-party MFA → **AD FS**

### Azure AD Connect
- Synchronizes on-premises AD objects to Entra ID
- Supports PHS, PTA, AD FS
- Filtering: OU-based, attribute-based, domain-based

### Azure AD Connect Cloud Sync
- Lighter-weight alternative to Azure AD Connect
- Agent-based (no separate server needed)
- Best for disconnected forests, limited on-premises footprint

### Entra Application Proxy
- Publishes **on-premises web apps** through Entra ID without a VPN
- Users authenticate via Entra ID (MFA supported)
- Connector agent installed on-premises network
- **Use case:** Provide SSO access to on-premises internal apps for remote workers

### Entra Private Access
- Zero Trust Network Access (ZTNA) solution
- Replaces VPN for accessing on-premises resources
- Identity-based, per-app access instead of network-level

---

## 3. Azure Role-Based Access Control (RBAC)

### Built-In Roles

| Role | Permissions |
|---|---|
| **Owner** | Full access + manage access (assign roles) |
| **Contributor** | Full access, cannot manage access |
| **Reader** | Read-only |
| **User Access Administrator** | Manage access only (assign roles), no resource access |

### Scope Hierarchy
```
Root Management Group
  └── Management Groups
        └── Subscriptions
              └── Resource Groups
                    └── Resources
```
- Roles assigned at higher scope are inherited downward
- Deny assignments override allow assignments and are inherited downward too

### Custom Roles
- Define when built-in roles don't meet least-privilege requirements
- Specify allowed actions, not allowed actions (notActions), data actions
- Assignable scopes define where the role can be assigned
- Limit: 5,000 custom roles per tenant

### Privileged Identity Management (PIM)
- Just-In-Time (JIT) activation of privileged roles
- Requires Entra ID P2
- Features: Time-bound access, approval workflows, MFA on activation, access reviews

| Concept | Description |
|---|---|
| **Eligible assignment** | User can activate the role when needed |
| **Active assignment** | Role is always active for user |
| **Activation** | User requests role, may require MFA + approval |
| **Access review** | Periodic review of who has role |

### RBAC Building Blocks

Every Azure RBAC design question has three parts:

| Building Block | Question to Ask |
|---|---|
| **Security principal** | Who needs access: user, group, service principal, or managed identity? |
| **Role definition** | What permissions are required: read, modify resources, manage access, or data access? |
| **Scope** | Where should access apply: management group, subscription, resource group, or resource? |

> **Exam tip:** AZ-305 usually rewards the most restrictive scope and the least-privileged built-in role that still satisfies the requirement.

### Management Plane vs Data Plane

This is one of the most common RBAC exam traps.

| Plane | What It Controls | Examples |
|---|---|---|
| **Management plane** | Managing Azure resources through Azure Resource Manager | Create VM, delete storage account, change Key Vault settings |
| **Data plane** | Accessing the data inside a resource | Read blob contents, read Key Vault secrets, query queue messages |

**Decision rule:**
- If the question says **manage the resource**, think Azure management-plane roles like Owner, Contributor, or Reader.
- If the question says **read or write the contents inside the resource**, verify whether a **data-plane RBAC role** is required.

### Built-In Role Traps

| Requirement | Best Fit | Common Wrong Answer |
|---|---|---|
| Full control including assigning roles | **Owner** | Contributor |
| Manage resources but not grant access | **Contributor** | Owner |
| Assign roles without changing resources | **User Access Administrator** | Contributor |
| View resources only | **Reader** | Contributor |

> **Exam tip:** `Contributor` cannot grant access. If the scenario includes creating role assignments, you need `Owner` or `User Access Administrator`.

### Data-Plane RBAC Scenarios

| Service | Requirement | Likely Direction |
|---|---|---|
| **Key Vault** | App reads secrets without stored credentials | Managed identity + Key Vault RBAC data access role |
| **Storage** | User reads blobs but should not manage the storage account | Storage data-plane RBAC role, not Contributor on the account |
| **Storage** | External short-term access | SAS, not broad RBAC assignment |
| **Key Vault** | Admin configures purge protection or networking | Management-plane role on the vault |

### AKS Authorization Exam Note

- **Azure RBAC** controls access to the AKS resource in Azure
- **Kubernetes RBAC** controls actions inside the Kubernetes cluster
- Some scenarios require both: Azure RBAC for cluster access path and Kubernetes RBAC for in-cluster permissions

### RBAC Decision Checklist

Before answering an RBAC question, check these in order:

1. Who needs access: human, app, workload, or external user?
2. Is the requirement control-plane or data-plane?
3. What is the smallest scope that works?
4. Is a built-in role sufficient, or is a custom role implied?
5. Should standing access be replaced with PIM eligible access?

### RBAC Scenario Quick Reference

| Scenario | Answer |
|---|---|
| App on App Service needs Key Vault access without secrets | System-assigned managed identity + Key Vault RBAC |
| Several VMs need the same permissions to Storage and Key Vault | User-assigned managed identity |
| Operations team must restart VMs but not assign roles | Contributor at resource group scope |
| Security team must manage role assignments only | User Access Administrator |
| Auditor needs to view resources across one subscription | Reader at subscription scope |
| Team needs read access to blob contents, not account management | Storage Blob Data Reader |
| Admin rights should require approval and MFA when needed | PIM eligible assignment |

---

## 4. Identity Governance

### Entitlement Management
- Manage access to groups, apps, and SharePoint sites through **access packages**
- External users can request access packages (connected organizations)
- Access packages can have expiration policies and approval workflows
- **Use case:** Onboard external partners with governed access

### Access Reviews
- Periodic review of users' access to resources and roles
- Reviewers: resource owners, users themselves, designated reviewers
- Automated decisions on no-response: approve or deny
- **Use case:** Certify that users still need access quarterly

### Lifecycle Workflows
- Automate identity lifecycle tasks (onboard, offboard, change)
- Trigger actions based on HR events (hire, terminate, role change)

### Microsoft Entra Permissions Management (CIEM)
- Cloud Infrastructure Entitlement Management
- Discover, remediate, and monitor permissions across Azure, AWS, GCP
- Detect permissions that are granted but unused

---

## 5. Management Groups and Policy

### Management Group Hierarchy
- Root Management Group (one per tenant, can't move/delete)
- Up to 6 levels of management group depth below root
- Apply policies and RBAC at management group scope — inherited by all subscriptions below

### Azure Policy

| Element | Description |
|---|---|
| **Policy Definition** | Rule that evaluates resource compliance |
| **Initiative (Policy Set)** | Collection of policy definitions grouped together |
| **Assignment** | Attaches a definition/initiative to a scope |
| **Effect** | What happens when rule is triggered |

### Policy Effects

| Effect | Description | Use Case |
|---|---|---|
| **Deny** | Block non-compliant resource creation/update | Enforce standards |
| **Audit** | Log non-compliance but allow | Reporting/assessment mode |
| **Append** | Add fields to resource | Add tags, settings |
| **Modify** | Add/change resource properties | Tag enforcement |
| **AuditIfNotExists** | Audit if related resource doesn't exist | Detect missing diagnostics |
| **DeployIfNotExists** | Deploy related resource if it doesn't exist | Auto-remediate (e.g., install agents) |
| **Disabled** | Policy is inactive | Pause enforcement |

> **Exam tip:** `DeployIfNotExists` is the key effect for **automatic remediation** (e.g., auto-install Log Analytics agent when missing).

### Azure Blueprints
- Package governance artifacts: policies, RBAC, ARM templates, Resource Groups
- **Blueprint assignment** locks down governance artifacts
- **vs Bicep/ARM:** Blueprints = governance packages with compliance tracking; Bicep/ARM = pure IaC deployment

> **Note:** Azure Blueprints (Preview) is deprecated effective **July 11, 2026**. Prefer **Deployment Stacks + Azure Policy + RBAC**.

---

## 6. Azure Key Vault

### Key Vault Object Types

| Type | Description | Use Case |
|---|---|---|
| **Secrets** | Any sensitive string (passwords, connection strings) | App credentials |
| **Keys** | Cryptographic keys (RSA, EC) | Encrypt/decrypt, sign/verify |
| **Certificates** | X.509 certificates with lifecycle management | TLS/SSL, code signing |

### Access Models

| Model | Description | Recommended? |
|---|---|---|
| **Vault Access Policy** | Legacy per-user/app permission model | No — legacy |
| **Azure RBAC** | Role-based control over Key Vault objects | Yes — preferred |

### Important Settings

| Setting | Description |
|---|---|
| **Soft Delete** | Deleted objects retained for 7–90 days (default 90) — on by default for new vaults |
| **Purge Protection** | Prevents permanent deletion during retention period — required for compliance |

### Backup and Restore Notes
- Key Vault backup and restore is for **individual objects**: **keys, secrets, and certificates**.
- Key Vault does **not** back up an entire vault in one operation.
- Backup output is an encrypted blob that **cannot be decrypted outside Azure**.
- A backup can be restored only to a vault in the **same Azure subscription** and **same Azure geography**.
- It does **not** have to be the exact same region, so "same region only" is too strict.
- Backups are **point-in-time snapshots** and do not stay synchronized automatically.
- Backup of an object with more than **500 versions** is not supported.

### Customer-Managed Keys (CMK)
- Key Vault stores the key that encrypts Azure resource data
- Supported services: Storage, Disk Encryption, SQL, Cosmos DB, etc.
- You control key rotation, expiration, and revocation
- Key Vault Managed HSM for highest security requirements (Federal Information Processing Standard (FIPS) 140-2 Level 3)

---

## 7. Microsoft Defender for Cloud

### Key Features

| Feature | Description |
|---|---|
| **Secure Score** | Aggregated security health score across subscriptions |
| **Security Recommendations** | Actionable steps to improve posture |
| **Security Alerts** | Real-time threat detections |
| **Regulatory Compliance** | Compliance against standards (PCI DSS, ISO 27001, CIS) |
| **Workload Protections** | Enhanced threat detection per resource type |

### Defender Plans (Enhanced Protection)
- Defender for Servers, SQL, Storage, Containers, App Service, Key Vault, DNS, etc.
- Each plan adds threat intelligence and detection for that resource type

---

## 8. Monitoring — Azure Monitor

### Azure Monitor Overview
Central monitoring platform collecting metrics, logs, and traces.

### Data Types

| Data | Description | Store |
|---|---|---|
| **Metrics** | Numerical time-series data (CPU %, requests/sec) | Azure Monitor Metrics (93 days) |
| **Logs** | Text-based records with rich query support | Log Analytics Workspace |
| **Traces** | Distributed tracing for application performance | Application Insights |
| **Activity Log** | Subscription-level control plane events | 90 days by default |

### Log Analytics Workspace Design

| Consideration | Recommendation |
|---|---|
| **Centralized** | Single workspace for all teams — simpler, cross-resource queries | Single workspace for most orgs |
| **Decentralized** | Separate workspaces per BU/environment — data isolation, billing control | Regulated industries, strict isolation |
| **RBAC** | Table-level RBAC, resource-scope RBAC — give teams access without seeing all data | Balance centralization with access control |

### Azure Monitor Agents

| Agent | Status | Use Case |
|---|---|---|
| **Azure Monitor Agent (AMA)** | **Current, preferred** | All new deployments — supports DCRs |
| **Log Analytics Agent (MMA/OMS)** | Legacy, retiring Aug 2024 | Legacy only |
| **Diagnostics Extension** | Legacy | Legacy VMs |

**Data Collection Rules (DCR):** Define what data AMA collects and where to send it.

### Alerts

| Alert Type | Triggers On | Use Case |
|---|---|---|
| **Metric alert** | Threshold on metric value | CPU > 90%, requests > 1000/min |
| **Log alert** | Kusto query results | Complex condition from logs |
| **Activity log alert** | Azure control plane events | Service health, resource deletion |
| **Smart detection** | ML-based anomaly detection | Application Insights |

### Application Insights
- Application Performance Monitoring (APM) — part of Azure Monitor
- Tracks: Requests, exceptions, dependencies, page views, custom events
- **Availability tests:** HTTP ping tests from multiple regions
- **Application Map:** Visual distributed system topology
- Connection string (preferred) or instrumentation key

---

## 9. Azure Advisor & Cost Management

### Azure Advisor
- Proactive recommendations in 5 categories:
  - Reliability, Security, Performance, Cost, Operational Excellence
- Integrates with Azure Monitor and Defender for Cloud

### Microsoft Cost Management + Billing
- Track spending by subscription, resource group, tag
- **Budgets:** Set spending limits, trigger alerts or action groups
- **Cost alerts:** Budget alerts, anomaly alerts, credit alerts
- **Reservations:** 1 or 3 year commitments for ~40–72% savings
- **Azure Hybrid Benefit:** Use existing Windows Server / SQL Server licenses in Azure

---

## 10. Microsoft Sentinel

### What It Is
Microsoft Sentinel is a cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automation, and Response)** platform built on Azure Monitor / Log Analytics.

### Key Capabilities

| Capability | Description |
|---|---|
| **Data connectors** | Ingest logs from Azure, M365, AWS, Syslog, CEF, and 3rd-party products |
| **Analytics rules** | Detect threats via scheduled KQL queries or ML-based fusion rules |
| **Incidents** | Grouped related alerts into actionable investigations |
| **Workbooks** | Interactive dashboards for security monitoring |
| **Playbooks** | Azure Logic Apps workflows triggered on alerts/incidents (SOAR) |
| **Threat hunting** | Proactive KQL-based queries against raw logs |
| **UEBA** | User and Entity Behavior Analytics — detect anomalous activity |
| **Threat intelligence** | Import indicators of compromise (IoCs) to enrich alerts |

### Sentinel vs Defender for Cloud

| | Microsoft Sentinel | Microsoft Defender for Cloud |
|---|---|---|
| **Primary role** | SIEM/SOAR — aggregate and correlate logs across everything | CSPM/CWPP — secure posture for Azure workloads |
| **Scope** | Cross-platform (Azure, AWS, GCP, on-premises, M365) | Azure resources + hybrid |
| **Alerts** | Cross-source, correlated incidents | Resource-level threat alerts |
| **Response** | Logic Apps playbooks (SOAR) | Recommendations + auto-remediation |
| **Cost model** | Pay-per-GB ingested | Per resource/plan enabled |
| **Choose when** | Enterprise SOC, cross-platform threat detection | Azure security posture and compliance |

> **Exam tip:** "Centralized security monitoring across Azure, AWS, and on-premises" → **Microsoft Sentinel**. "Improve Azure security score and compliance" → **Defender for Cloud**. Both can be used together.

### Playbooks (SOAR)
- Playbooks are **Azure Logic Apps** triggered by Sentinel alerts or incidents
- Common automations: Block IP in firewall, disable compromised user, create ITSM ticket, notify Teams channel
- Require a **Logic App Standard or Consumption** workflow connected to Sentinel

### Log Analytics Integration
- Sentinel workspace **is** a Log Analytics workspace
- All data stored in standard Log Analytics tables (SecurityEvent, SigninLogs, AzureActivity, etc.)
- Query with KQL — same skills as Log Analytics

---

## 11. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| App needs to access Key Vault, no credentials in code | Managed Identity (System or User-assigned) |
| Multiple VMs all need the same Key Vault permissions | User-assigned Managed Identity |
| Partner users need access to internal SharePoint, no tenant join | Azure AD B2B Guest Users |
| Consumer-facing app with social logins (Google, Facebook) | Azure AD B2C |
| Employees must use MFA when signing in from outside corporate network | Conditional Access — location-based MFA |
| Admin should only have Global Admin when needed, with MFA + approval | PIM eligible assignment |
| Quarterly review of who has Owner role in production | PIM Access Reviews |
| External contractors need access for 90 days to specific app | Entitlement Management access packages |
| Automatically install Log Analytics agent on all new VMs | Azure Policy — DeployIfNotExists effect |
| Block creation of public IP addresses in production subscription | Azure Policy — Deny effect |
| Enforce tagging on all new resources, with auto-tag on creation | Azure Policy — Modify effect |
| On-premises app accessible to remote users via SSO, no VPN | Entra Application Proxy |
| Synchronize on-premises AD, no complex claims needed | Password Hash Sync (PHS) |
| Password sync not allowed, must validate on-premises | Pass-Through Authentication (PTA) |
| Key Vault secrets must be recoverable 90 days after deletion | Soft Delete + Purge Protection |
| Monitor application performance, track dependencies | Application Insights |
| Get alerts when monthly Azure spend exceeds $5,000 | Cost Management Budget alert |
| Centralized security monitoring across Azure, AWS, on-premises | Microsoft Sentinel |
| Automatically block a compromised account when alert fires | Sentinel Playbook (Logic Apps) |
| Detect anomalous user behavior across all sign-in logs | Sentinel UEBA |
| Improve Azure secure score, fix compliance gaps | Microsoft Defender for Cloud |

---

## 12. Identity and Governance Exam Traps

### 1. Choosing service principals with secrets when managed identity is available
- **Trap:** Service principals are familiar and broadly applicable
- **Better default:** Managed identity when the workload is an Azure resource accessing another Azure service

### 2. Confusing Azure AD B2B with Azure AD B2C
- **Trap:** Both handle external identities
- **Better default:** B2B for partners and guest access to your tenant; B2C for customer-facing identity

### 3. Choosing RBAC when governance enforcement is required
- **Trap:** RBAC controls access, so it feels like the whole answer
- **Better default:** Azure Policy for compliance and enforcement; RBAC for authorization

### 4. Choosing Key Vault without checking the access model or recovery settings
- **Trap:** Key Vault is correct, but the design omits RBAC, soft delete, or purge protection
- **Better default:** Match the access model and data protection settings to the requirement

### 5. Assuming Key Vault backup restore requires the exact same region
- **Trap:** "Restore must be in the same region" sounds safe, but it is not the documented rule.
- **Better default:** Key Vault backup restore requires the **same subscription** and **same Azure geography**, not necessarily the exact same region.

### 6. Using Sentinel when the requirement is posture management
- **Trap:** Sentinel sounds like the broadest security answer
- **Better default:** Defender for Cloud for posture, recommendations, and secure score; Sentinel for SIEM/SOAR and investigation

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Azure resource needs secretless access to Key Vault | Secret-based service principal answers |
| Partner access to internal apps | B2C answers |
| Customer sign-in with social identity providers | B2B answers |
| Enforce tags or block public IPs | RBAC-only answers |
| Improve security posture and recommendations | Sentinel-only answers |

