# SecureIT Lab — Windows Server Security Walkthrough

A complete lab build for a fictional company, **SecureIT**, covering Domain Controller setup, Enterprise CA, File Server permissions, Group Policy hardening, firewall rules, and auditing.

---

## Lab Overview

### Environment
- **Hypervisor:** Hyper-V (or equivalent)
- **IP Range:** `172.16.50.0/24`
- **No default gateway required** (isolated lab network)

### Machines

| Hostname | Role | IP | DNS |
|----------|------|----|-----|
| `SITDC01` | Domain Controller + DNS | `172.16.50.10` | `172.16.50.10` |
| `SITCA01` | Enterprise CA | `172.16.50.20` | `172.16.50.10` |
| `SITFileServer01` | File Server | `172.16.50.30` | `172.16.50.10` |
| `SITWin10-01` | Windows 10 Client | `172.16.50.100` | `172.16.50.10` |

> **Passwords** are intentionally omitted from this document. Use your own strong passwords when rebuilding the lab.

---

## Task 1 — Build the Lab Environment

Created four VMs in Hyper-V:

1. **SITDC01** — Windows Server, promoted to Domain Controller
2. **SITCA01** — Windows Server, joined to domain, AD CS role
3. **SITFileServer01** — Windows Server, joined to domain, file services
4. **SITWin10-01** — Windows 10 client, joined to domain

All machines use `SITDC01` as their DNS server.

---

## Task 2 — Create a Second Domain Administrator

**Goal:** Ensure at least 2 Domain Admins for redundancy.

### Steps
1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. Navigate to `SecureIT.local → Users`
3. Right-click **Administrator** → **Copy**
4. Fill in:
   - First name: `Admin2`
   - Last name: *(generic)*
   - Full name: `SecureIT Admin2`
   - User logon name: `admin2`
5. Set a strong password
6. Uncheck *User must change password at next logon* (lab convenience)
7. Check *Password never expires* (lab only — not for production)
8. Finish

### Add to Domain Admins
1. Open **Domain Admins** group properties
2. **Members** tab → **Add** → `SecureIT Admin2`
3. OK

### Verify
- Log out and log in as `SECUREIT\admin2`
- Confirm admin rights by opening Server Manager

---

## Task 3 — Create OU Structure

**Goal:** A clean OU layout for SecureIT objects.

### OUs Created
```
SecureIT.local
├── SecureITUsers
├── SecureITComputers
├── SecureITGroups
└── SecureITServers
```

### Steps
1. In ADUC, right-click the domain → **New** → **Organizational Unit**
2. Create each OU
3. Move existing objects into their matching OU:
   - User accounts → `SecureITUsers`
   - Computer accounts → `SecureITComputers`
   - Groups → `SecureITGroups`
   - Servers → `SecureITServers`

---

## Task 4 — Install Enterprise CA

**Goal:** A functioning Enterprise Root CA.

### Steps
1. On `SITCA01`, open **Server Manager** → **Add Roles and Features**
2. Select **Active Directory Certificate Services**
3. Add **Certification Authority** + **Certification Authority Web Enrollment**
4. Install
5. After install: click the flag → **Configure Active Directory Certificate Services**
6. Credentials: Domain Admin
7. Check: **Certification Authority** + **Certification Authority Web Enrollment**
8. Setup type: **Enterprise CA**
9. CA type: **Root CA**
10. Private key: **Create new**
11. Cryptography: **RSA 4096**, **SHA512**
12. Common name: default (do not change)
13. Validity: default
14. Configure

### Verify
- Tools → **Certification Authority**
- CA is listed and running
- `Issued Certificates` folder exists (empty for now)

---

## Task 5 — Auto-Enrollment GPO for Computer Certificates

**Goal:** Domain computers automatically receive a computer certificate from the CA.

### Steps
1. On `SITDC01`, open **Group Policy Management** (`gpmc.msc`)
2. Navigate to `SecureIT.local → Group Policy Objects`
3. Right-click → **New** → name: `Auto Computer Cert`
4. Right-click the GPO → **Edit**
5. Navigate to:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Public Key Policies`
6. Double-click **Certificate Services Client - Auto-Enrollment**
   - Configuration: **Enabled**
   - Check: *Renew expired certificates*
   - Check: *Update certificates that use certificate templates*
7. OK
8. Still in Public Key Policies:
   - Right-click **Automatic Certificate Request Settings** → **New** → **Automatic Certificate Request**
   - Next → **Computer** → Next → Finish
9. Close the GPO editor
10. Link the GPO to `SecureITComputers` and `SecureITServers`

### Verify
1. On `SITWin10-01`: `gpupdate /force`
2. On `SITCA01`: **Certification Authority → Issued Certificates**
3. The client's computer certificate should appear

---

## Task 6 — Restrict Local Administrators via GPO

**Goal:** Only members of the `ITDepartment` group are local admins on client machines.

### Steps

**1. Create the IT Department user + group**
- New user: `Sam Morphy` (generic name in this doc), logon: `samo`
- New group: `ITDepartment`
- Add `Sam Morphy` to `ITDepartment`

**2. Create the GPO**
1. Group Policy Management → Group Policy Objects → New
2. Name: `Restricted Local Admins`
3. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Restricted Groups`
   - Right-click → **Add Group**
   - Browse → **Administrators** (local)
   - Members of this group: add `ITDepartment` (domain group)
   - Members of this group are: **Administrators**
4. OK → Close

**3. Link the GPO**
- Link to `SecureITComputers`

### Verify
1. On `SITWin10-01`: `gpupdate /force`
2. `net localgroup administrators`
3. Should show:
   - `Administrator` (built-in)
   - `SECUREIT\ITDepartment`

---

## Task 7 — Create the Ekonomi Group

**Goal:** A separate group for the Finance department.

### Steps
1. ADUC → `SecureITGroups` → New → Group
2. Name: `Ekonomi`
3. Scope: Global, Type: Security
4. Create a user: `Lila Larson` (generic), logon: `lila`
5. Add `Lila Larson` to the `Ekonomi` group

---

## Task 8 — File Server with Two Shares

**Goal:** Two shares with different permissions:
- **Share 1 (IT):** Full access for IT only
- **Share 2 (Economy):** Full access for Economy, Read access for IT

### Setup

**1. Create the folders**
On `SITFileServer01`:
- `D:\Shares\IT`
- `D:\Shares\Economy`

**2. Configure Share 1 — IT**

*Share permissions:*
1. Right-click `IT` → Properties → **Sharing** → **Advanced Sharing**
2. Check **Share this folder** → name: `IT`
3. Permissions → Remove `Everyone` → Add `ITDepartment` → **Full Control**
4. OK

*NTFS permissions:*
1. **Security** tab → **Advanced**
2. **Disable inheritance** → **Convert inherited permissions to explicit**
3. Remove inherited entries
4. Add `ITDepartment` → **Full Control**
5. Apply

**3. Configure Share 2 — Economy**

*Share permissions:*
1. Right-click `Economy` → Properties → **Sharing** → **Advanced Sharing**
2. Check **Share this folder** → name: `Economy`
3. Permissions → Remove `Everyone` → Add:
   - `Ekonomi` → **Full Control**
   - `ITDepartment` → **Read**
4. OK

*NTFS permissions:*
1. **Security** tab → **Advanced**
2. **Disable inheritance** → convert to explicit
3. Remove inherited entries
4. Add:
   - `Ekonomi` → **Full Control**
   - `ITDepartment` → **Read & Execute**
5. Apply

### Verify
- Log in as `Lila Larson` → access `\\SITFileServer01\Economy` → read/write works
- Log in as `Sam Morphy` (IT) → access `\\SITFileServer01\IT` → full access
- `Sam Morphy` → access `\\SITFileServer01\Economy` → read-only

---

## Task 9 — Cached Logins & Login Screen GPO

**Goal:**
- No cached logins (0)
- Login screen doesn't show username

### Steps
1. Group Policy Management → New GPO → `Security - Login Hardening`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options`
   - **Interactive logon: Number of previous logons to cache** → `0`
   - **Interactive logon: Do not display last user name** → **Enabled**
   - **Interactive logon: Do not display username @ domain** → **Enabled**
   - **Interactive logon: Machine inactivity limit** → `60` seconds
3. Link to `SecureITComputers`

### Verify
- `gpupdate /force` on client
- Lock screen should not show last username
- `secpol.msc` on client confirms settings

---

## Task 10 — Force Firewall On via GPO

**Goal:** Windows Defender Firewall is always on for all profiles.

### Steps
1. New GPO: `Firewall Always On`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Windows Defender Firewall with Advanced Security`
   - Click **Windows Defender Firewall Properties**
   - For **Domain**, **Private**, **Public**:
     - Firewall state: **On**
     - Inbound connections: **Block**
     - Outbound connections: **Allow**
   - OK
3. Link to `SecureITComputers` and `SecureITServers`

### Verify
- `gpupdate /force` on client
- `wf.msc` → all profiles show firewall on
- Local user cannot disable it

---

## Task 11 — File Deletion Auditing

**Goal:** Log every deletion in a shared folder.

### Steps

**1. Create a folder to audit**
- On `SITFileServer01`: `D:\Shares\Audited`
- Share it (Give `Everyone` full share + NTFS access for lab simplicity)

**2. Enable auditing on the folder**
1. Right-click `Audited` → Properties → **Security** → **Advanced** → **Auditing**
2. Add → **Everyone**
3. Type: **All**
4. Applies to: **This folder, subfolders and files**
5. Check: **Delete**, **Delete subfolders and files**
6. OK → Apply

**3. Enable auditing via GPO**
1. DC → new GPO: `Auditing File Server`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies → Object Access`
   - **Audit File System** → **Success** + **Failure**
3. Link to `SecureITServers`
4. On the file server: `gpupdate /force`

**4. Test**
1. Create `test.txt` inside `Audited`
2. Delete it
3. Event Viewer → `Windows Logs → Security`
4. Filter for **4660** (deleted) and **4663** (accessed)
5. Confirm the deletion event appears with user, time, and file details

---

## Task 12 — RDP Restricted to One Client IP

**Goal:** Only `SITWin10-01` can RDP into `SITDC01`.

### Steps

**1. Enable Remote Desktop on the DC**
- Server Manager → Local Server → **Enable Remote Desktop**

**2. Create the firewall rule**
1. Tools → **Windows Defender Firewall with Advanced Security**
2. **Inbound Rules** → New Rule
3. Type: **Custom** → Next
4. All programs → Next
5. Protocol: **TCP** → Local port: **3389** → Next
6. Remote IP: **`172.16.50.100`** (client IP) → Next
7. Action: **Allow the connection** → Next
8. Profiles: **Domain**, **Private**, **Public** → Next
9. Name: `RDP from SITWin10-01` → Finish

**3. (Optional) Tighten further**
- Change the RDP listening port on the DC to a non-standard port (e.g. `33900`)
- Update the NSG/firewall rule to use the new port
- Enable a strong password + MFA on the local admin

### Verify
1. From `SITWin10-01` → RDP to `172.16.50.10` → works
2. From any other machine → RDP blocked

---

## Task 13 — Block Internet Browsing on the DC

**Goal:** Block outbound HTTP/HTTPS on the Domain Controller.

### Steps
1. On `SITDC01`: Windows Defender Firewall → **Outbound Rules**
2. New Rule → **Custom** → Next
3. All programs → Next
4. Protocol: **TCP** → Remote ports: **80, 443** → Next
5. **Block the connection** → Next
6. All profiles → Next
7. Name: `Block Web (TCP)` → Finish

8. Repeat with **UDP** → name: `Block Web (UDP)`

> Blocking UDP port 443 also blocks QUIC (HTTP/3).

### Verify
- Try browsing from the DC → blocked
- Ping still works
- DNS still works (port 53)

---

## Task 14 — Disable LLMNR and NetBIOS on Client

**Goal:** Reduce MITM/poisoning attack surface on `SITWin10-01`.

### Disable NetBIOS
1. Network adapter → Properties → **IPv4** → Properties → **Advanced**
2. **WINS** tab → **Disable NetBIOS over TCP/IP**
3. OK

### Disable LLMNR (via GPO)
1. New GPO: `Disable LLMNR`
2. Edit:
   - `Computer Configuration → Policies → Administrative Templates → Network → DNS Client`
   - **Turn off multicast name resolution** → **Enabled**
3. Link to `SecureITComputers`
4. `gpupdate /force` on the client

### Verify
- `ipconfig /all` → NetBIOS over TCP/IP shows *Disabled*
- Local Group Policy / registry confirms LLMNR off

---

## Summary of GPOs Created

| GPO Name | Purpose | Linked To |
|----------|---------|-----------|
| Auto Computer Cert | Auto-enroll computer certificates | Computers, Servers |
| Restricted Local Admins | Restrict local admin to IT Department | Computers |
| Security - Login Hardening | Cached logins = 0, hide username | Computers |
| Firewall Always On | Enforce firewall state on all profiles | Computers, Servers |
| Auditing File Server | Enable file system auditing | Servers |
| Disable LLMNR | Turn off multicast name resolution | Computers |

---

## Summary of Firewall Rules Created

| Rule | Direction | Purpose |
|------|-----------|---------|
| RDP from SITWin10-01 | Inbound | Restrict RDP to one client IP |
| Block Web (TCP) | Outbound | Block HTTP/HTTPS from DC |
| Block Web (UDP) | Outbound | Block QUIC/HTTP3 from DC |

---

## Key Takeaways

- **Least privilege** — delegate, don't over-grant
- **Layer your defenses** — GPO + firewall + NTFS + auditing
- **Audit everything** — know who did what, when
- **Segment the network** — don't run flat Layer 2 everywhere
- **Harden defaults** — LLMNR, NetBIOS, cached logins, USB, SMB signing
- **Automate with GPO** — one policy, many machines

---

*Coursework — Cloud and Infrastructure Specialist program.*