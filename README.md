# Windows Server Security Labs

Hands-on Windows Server administration and security labs completed as part of the **Cloud and Infrastructure Specialist program (EC Utbildning)**. Covers Active Directory, Group Policy, PKI, file server security, firewall hardening, wireless authentication, and hybrid cloud networking.

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

`Windows Server` · `Active Directory` · `Group Policy` · `AD CS` · `Hyper-V` · `IIS` · `DNS` · `Windows Defender Firewall` · `NPS / RADIUS` · `802.1X` · `EFS` · `SMB` · `Cisco Packet Tracer` · `Azure` · `IPsec VPN` · `FortiGate` (concepts only) · `VLANs` · `ACLs` · `RSTP` · `LACP`

---

## 💡 What I Learned

- Applying least privilege through delegation instead of over-granting admin rights
- Designing layered security using GPO, firewall rules, NTFS permissions, and auditing together
- Building and managing an Enterprise Certificate Authority with auto-enrollment
- Understanding how share and NTFS permissions interact — and why the most restrictive wins
- Hardening Windows defaults (LLMNR, NetBIOS, cached logins, USB, SMB signing)
- Restricting RDP access to specific source IPs and blocking outbound web traffic on servers
- Setting up WPA2-Enterprise Wi-Fi with RADIUS and NPS
- Designing a site-to-site IPsec VPN between on-prem and Azure
- Segmenting networks with VLANs and controlling traffic with ACLs
- Comparing switching protocols like RSTP and LACP in real topologies

---

## 🔒 Notes on Anonymization

All usernames, hostnames, credentials, real IP addresses, and tenant identifiers in this repository have been **anonymized** for privacy and security. Lab names (`SITDC01`, `SecureIT.local`) are fictional placeholders. No real credentials are included.

---

## 📄 License

This is coursework material. Feel free to use it for learning purposes.

---

*Coursework — Cloud and Infrastructure Specialist program, EC Utbildning.*
