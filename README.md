# Windows Server Security Labs
Hands-on Windows Server administration and security coursework covering Active Directory, Group Policy, PKI, file server security, firewall hardening, NPS/RADIUS, network segmentation, and hybrid Azure/on-premises networking in an isolated lab environment.

---

## 📖 Overview

This repository contains lab notes and walkthroughs for a fictional company environment, **SecureIT**, built in an isolated Hyper-V lab. The work spans:

1. **Windows Server Security Notes** — conceptual and hands-on notes on AD, GPO, PKI, file permissions, firewall rules, Wi-Fi security, and hybrid cloud VPNs.
2. **SecureIT Lab Walkthrough** — a complete build of a domain environment with a Domain Controller, Enterprise CA, File Server, and client machine.

---

## 📚 Topics Covered

### Active Directory
- Delegation of control
- Least privilege principles
- OU design and structure
- Multiple Domain Administrator accounts for redundancy
- RSAT for remote administration

### Group Policy (GPO)
- Restricting local administrators to a specific group
- Certificate auto-enrollment
- Firewall state enforcement
- File system auditing
- Cached login restrictions
- Login screen hardening
- USB storage blocking
- LLMNR disable
- SMB signing enforcement

### Public Key Infrastructure (PKI)
- Enterprise Root CA installation
- Domain certificates (web server)
- Certificate requests and completion
- User certificates
- Auto-enrollment via GPO
- Custom certificate templates

### File Server Security
- Share permissions vs NTFS permissions
- "Most restrictive wins" logic
- Explicit deny behavior
- Auditing file access and deletion
- Hidden shares (`$`)
- EFS encryption and behavior when files are copied to shares

### Firewall Configuration
- Inbound vs outbound rules
- RDP restriction to a single source IP
- Blocking HTTP/HTTPS on servers
- Enforcing firewall state via GPO
- Domain / Private / Public profiles

### Wireless Security
- WPA2-Enterprise with RADIUS
- Network Policy Server (NPS) configuration
- 802.1X authentication
- Group Policy for secure Wi-Fi enforcement
- Certificate auto-enrollment for RADIUS

### Network Segmentation
- VLANs, trunks, and router-on-a-stick
- ACLs for inter-subnet traffic control
- Cisco Packet Tracer configuration examples
- LLMNR / NetBIOS attack surface reduction

### Hybrid Cloud & VPN
- Site-to-site IPsec VPN between Azure and on-prem firewall
- User Defined Routes (UDR) for traffic steering
- Azure Firewall vs Network Security Groups (NSG)
- IaaS / PaaS / SaaS responsibility model
- CGNAT explained
- VPN vs Tor comparison

### Switching Concepts
- RSTP vs LACP comparison
- Loop prevention vs link aggregation

---

## 📂 Repository Structure

```
windows-server-security-labs/
├── README.md
├── windows-server-security.md
├── secureit-lab-walkthrough.md
└── screenshots/
    ├── ad-delegation/
    ├── group-policy/
    ├── pki/
    ├── file-server/
    ├── firewall/
    └── vpn/
```

---

## 🧪 SecureIT Lab Environment

| Hostname | Role | IP |
|----------|------|----|
| `SITDC01` | Domain Controller + DNS | `172.16.50.10` |
| `SITCA01` | Enterprise CA | `172.16.50.20` |
| `SITFileServer01` | File Server | `172.16.50.30` |
| `SITWin10-01` | Windows 10 Client | `172.16.50.100` |

- IP Range: `172.16.50.0/24`
- No default gateway (isolated lab)
- Domain: `SecureIT.local` (fictional)

### GPOs Created

| GPO Name | Purpose | Linked To |
|----------|---------|-----------|
| Auto Computer Cert | Auto-enroll computer certificates | Computers, Servers |
| Restricted Local Admins | Restrict local admin to IT Department | Computers |
| Security - Login Hardening | Cached logins = 0, hide username | Computers |
| Firewall Always On | Enforce firewall state on all profiles | Computers, Servers |
| Auditing File Server | Enable file system auditing | Servers |
| Disable LLMNR | Turn off multicast name resolution | Computers |

### Firewall Rules Created

| Rule | Direction | Purpose |
|------|-----------|---------|
| RDP from SITWin10-01 | Inbound | Restrict RDP to one client IP |
| Block Web (TCP) | Outbound | Block HTTP/HTTPS from DC |
| Block Web (UDP) | Outbound | Block QUIC/HTTP3 from DC |

---
## 🧰 Tools & Technologies

**Windows & Infrastructure**
- Windows Server
- Active Directory
- Group Policy
- AD CS / PKI
- Hyper-V
- IIS
- DNS
- Windows Defender Firewall
- NPS / RADIUS
- 802.1X

**Security**
- EFS
- SMB
- File system auditing
- Certificate-based authentication

**Networking**
- Cisco Packet Tracer
- VLANs
- ACLs
- RSTP
- LACP
- IPsec VPN

**Azure / Cloud**
- Microsoft Azure
- User Defined Routes (UDR)
- Azure Firewall
- Network Security Groups (NSG)
- FortiGate — concepts only

---

## 💡 Key Skills Demonstrated

- Administering Windows Server environments with Active Directory and DNS
- Applying least-privilege principles through delegation and Group Policy
- Implementing security hardening with GPO, Windows Firewall, auditing, and access controls
- Deploying an Enterprise Certificate Authority and configuring certificate auto-enrollment
- Managing file and folder security using share permissions, NTFS permissions, auditing, and EFS
- Configuring NPS/RADIUS and 802.1X for enterprise wireless authentication
- Designing network segmentation using VLANs, trunks, and ACLs
- Configuring and troubleshooting site-to-site IPsec VPN concepts between on-premises and Azure
- Understanding Azure networking components including UDRs, Azure Firewall, and NSGs
- Comparing RSTP and LACP and understanding their roles in enterprise switching

---

## 🔒 Notes on Anonymization

All usernames, hostnames, credentials, real IP addresses, and tenant identifiers in this repository have been **anonymized** for privacy and security. Lab names (`SITDC01`, `SecureIT.local`) are fictional placeholders. No real credentials are included.

---

## 📄 License

This is coursework material. Feel free to use it for learning purposes.

---

*Coursework — Cloud and Infrastructure Specialist program, EC Utbildning.*
