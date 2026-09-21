# IT Security — Theory Answers

Written answers to the theoretical portion of the IT Security exam. Covers cloud service models, hybrid solutions, Azure access control, encryption, redundancy, backup responsibility, and secure hybrid networking.

---

## Q1 — Explain the difference between IaaS, SaaS, PaaS, and On-Prem. Who is responsible for what?

The core difference between these models is **how much of the stack the cloud provider manages versus the customer**.

| Model | Provider Manages | Customer Manages | Example |
|-------|------------------|------------------|---------|
| **IaaS** | Virtualization, storage, networking | OS, runtime, applications, data | Azure VMs |
| **PaaS** | Above + OS and runtime | Applications, data | Azure App Service |
| **SaaS** | Entire stack (infrastructure → application) | Application configuration, user data | Microsoft 365 |
| **On-Prem** | Nothing | Everything — hardware, software, network, security, OS, apps | Owned local server |

**In short:** As you move from On-Prem → IaaS → PaaS → SaaS, the provider takes on more responsibility, and the customer takes on less.

---

## Q2 — What is a hybrid solution?

A **hybrid solution** combines on-prem infrastructure with public cloud, allowing applications and services to work as a single unified system.

**Common reasons for choosing hybrid:**

- **Performance scaling** — keep the base workload on-prem, burst to the cloud during peak demand
- **Data sensitivity** — keep sensitive or regulated data on-prem while running less sensitive workloads in the cloud
- **Cost control** — avoid migrating everything at once
- **Compliance** — meet data residency or regulatory requirements

**Main benefit:** The organization gets the optimal environment for each workload, rather than forcing everything into one model.

---

## Q3 — Which Azure feature can assign and restrict the use of resources? How do you control who can do what in a cloud environment?

**Azure Role-Based Access Control (RBAC)** — often managed through **IAM (Identity and Access Management)** in the Azure portal.

**How it works:**
- Permissions are assigned to **roles**, not directly to users
- Users (or groups) are assigned to those roles
- Each assignment is scoped — e.g. *Owner* role at the *resource group* level

**Example:**
> Assign the **Owner** role to a user at the scope of a specific resource group. That user can manage everything inside that resource group — but nothing outside it.

**Why this matters:**
- **Scalable** — role changes apply to everyone in that role
- **Secure** — least privilege by default
- **Maintainable** — centralized permission management

---

## Q4 — A company wants to use a FortiGate firewall in the cloud. What do you use in Azure to ensure traffic from a subnet goes through the firewall before reaching the internet?

**User Defined Routes (UDR)** combined with an **Azure Route Table**.

**How to set it up:**
1. Deploy the FortiGate firewall (typically in its own subnet)
2. Create a **Route Table** in Azure
3. Add a route:
   - **Destination:** `0.0.0.0/0` (all internet-bound traffic)
   - **Next hop:** the FortiGate's private IP
4. Associate the route table with the subnet(s) whose traffic must pass through the firewall

**Result:** All outbound traffic from that subnet is forced through the firewall first.

---

## Q5 — If you want to ensure your resource is not dependent on a single Azure data center (i.e. create redundancy), how do you solve this in Azure? What feature do you use?

**Availability Zones.**

An Availability Zone is a **physically separate data center** within the same Azure region. Each zone has:
- Independent power, cooling, and networking
- Its own physical infrastructure
- Low-latency connectivity to the other zones in the region

**How to use them:**
- Deploy resources across **two or more zones** within the same region
- If one zone fails, the other zones keep running
- Azure automatically load-balances and replicates across zones

**Alternative (for larger geographic redundancy):** **Region Pairs** — Azure regions paired within the same geography for disaster recovery and data residency.

---

## Q6 — What is the difference between asymmetric and symmetric encryption?

| Feature | Symmetric | Asymmetric |
|---------|-----------|------------|
| **Keys** | 1 shared key | 2 keys — public + private |
| **Speed** | Fast | Slow |
| **Key exchange** | Difficult (main weakness) | Easy (public key can be shared freely) |
| **Use case** | Database encryption, bulk data | Digital signatures, SSL/TLS, email encryption |
| **Example algorithms** | AES | RSA |

**In short:**
- **Symmetric** is fast but the key must be shared securely.
- **Asymmetric** solves key exchange but is slower.
- In practice, they're combined — asymmetric is used to exchange a symmetric key, which then encrypts the bulk data.

---

## Q7 — Can a Virtual Network exist in multiple regions? If not, how do you connect regions?

**No.** An Azure Virtual Network (VNet) belongs to **one region**.

**To connect VNets across regions:**

- **Global VNet Peering** — connect VNets in different regions using private IP addresses. Traffic stays on Microsoft's backbone, so it's fast and secure.
- **VPN Gateway (Site-to-Site)** — connect over the public internet using IPsec tunnels (slower, but works across any distance)
- **Azure ExpressRoute** — dedicated private connection (most expensive, highest performance)

**Most common for region-to-region connectivity:** **Global VNet Peering**.

---

## Q8 — You have a cloud installation. Who is responsible for backing up your content in the cloud?

**The customer is always responsible** — regardless of whether the service is IaaS, PaaS, or SaaS.

**Why:**
- Cloud providers guarantee **infrastructure availability**, not **data protection**
- The provider ensures the service is *running* — not that your data is *recoverable*
- Backups must meet the organization's own retention, compliance, and recovery requirements

**Best practice:**
- Define a backup strategy (frequency, retention, locations)
- Test restores periodically
- Follow the **3-2-1 rule** (3 copies, 2 media types, 1 off-site)

---

## Q9 — Can you store keys, passwords, and certificates in the cloud? If yes, what is the feature called?

**Yes — Azure Key Vault.**

**What it stores:**
- **Secrets** — passwords, API keys, connection strings
- **Keys** — encryption keys
- **Certificates** — SSL/TLS certificates

**Key features:**
- Encrypted at rest and in transit
- **Integration** with Azure services and applications
- **Access control** via RBAC and access policies
- **Auditing** — every access is logged
- **Hardware Security Modules (HSM)** support for extra protection

---

## Q10 — Locally you can use ACL and VLAN to separate and filter traffic between subnets. What feature do you use in Azure?

**Network Security Groups (NSGs).**

**How NSGs work:**
- Applied to a **subnet** or a **network interface (NIC)**
- Contain **inbound** and **outbound** rules
- Each rule specifies:
  - Source (IP or range)
  - Destination (IP or range)
  - Port
  - Protocol
  - Allow or Deny
- Rules are evaluated by **priority** — lower number = higher priority

**Use case:** Deny RDP from the internet, allow it only from a specific subnet; allow HTTP/HTTPS in but block everything else.

---

## Q11 — Why is it recommended to have more than one Domain Administrator?

Having only one Domain Admin creates a **single point of failure** and a serious security risk.

**Reasons for multiple admins:**

- **Availability** — if one admin is unavailable, others can still manage the domain
- **Redundancy** — no single account locks out the organization
- **Continuity** — business operations continue during incidents
- **Compromise resilience** — if one admin account is compromised, others remain secure
- **Separation of duties** — different admins can hold different responsibilities

**Best practices:**
- Each admin has their **own account** (no shared accounts)
- All admin accounts require **MFA**
- Use **dedicated admin workstations** for admin tasks
- Apply **just-in-time (JIT)** access where possible

---

## Q12 (Advanced) — You created a VM for a customer who wants to access it via RDP. How do you secure it so only that customer can connect from their office — and nobody else in the world?

**Layered approach:**

1. **Create an NSG** for the VM's subnet or NIC
2. **Remove the default RDP rule** (which allows `3389` from any source)
3. **Create a new inbound rule** with:
   - Source: the customer's office **public IP** (or IP range)
   - Source port: `*`
   - Destination: `*`
   - Destination port: `3389`
   - Protocol: `TCP`
   - Action: **Allow**
4. **Add a deny-all rule** for RDP from anywhere else
5. **Change the RDP port** in the VM's registry/firewall to a non-standard port (e.g. `33900`) — then use that port in the NSG
6. **Strong local admin password** (or better: disable local admin, use a domain account with MFA)
7. **Open the port only when needed** — use Just-In-Time VM access
8. **Require VPN first** — allow RDP only from the VPN subnet

**Result:** RDP is only reachable from the customer's office IP, over a non-standard port, with strong authentication and minimal exposure window.

---

## Q13 (Advanced) — A customer has an on-prem installation and wants to extend to the cloud for some virtual tools. They want secure, unobstructed communication between on-prem servers and cloud servers. What is needed?

**A hybrid network connection — typically a Site-to-Site IPsec VPN.**

**Requirements:**

**On the Azure side:**
- A **Virtual Network (VNet)** with an address space
- A **Gateway Subnet** (dedicated subnet for the gateway)
- A **Virtual Network Gateway** (VPN type, Route-based)
- A **Local Network Gateway** representing the on-prem router (public IP + on-prem subnet)

**On the on-prem side:**
- A **VPN-capable firewall/router** (e.g. FortiGate)
- A **public IP address**
- The on-prem LAN subnet info

**On both sides:**
- A matching **pre-shared key (PSK)**
- Matching **encryption settings** (IKE version, AES, SHA, DH group)
- Matching **Phase 2 selectors** (local and remote subnets)

**Critical requirement:**
> The Azure VNet address space and the on-prem subnet **must not overlap**. If they do, routing breaks.

**Result:** Encrypted, private communication between on-prem and Azure over the public internet — with no need to expose internal services.

**Alternative for higher performance:** **Azure ExpressRoute** — a dedicated private connection that bypasses the public internet entirely.

---

## Summary

| # | Topic | Key Answer |
|---|-------|-----------|
| 1 | Cloud service models | IaaS, PaaS, SaaS, On-Prem — responsibility split |
| 2 | Hybrid solution | On-prem + cloud working as one system |
| 3 | Azure access control | RBAC / IAM |
| 4 | Force traffic through firewall | User Defined Routes (UDR) |
| 5 | Cross-datacenter redundancy | Availability Zones |
| 6 | Encryption types | Symmetric (1 key, fast) vs Asymmetric (2 keys, slow) |
| 7 | Cross-region VNet | Global VNet Peering |
| 8 | Cloud backup responsibility | Customer — always |
| 9 | Secret storage | Azure Key Vault |
| 10 | Subnet filtering | Network Security Groups (NSGs) |
| 11 | Multiple Domain Admins | Redundancy, resilience, separation of duties |
| 12 | Secure RDP | NSG + source IP restriction + non-standard port + JIT |
| 13 | Secure hybrid connection | Site-to-site IPsec VPN (or ExpressRoute) |

---

*Coursework — Cloud and Infrastructure Specialist program.*