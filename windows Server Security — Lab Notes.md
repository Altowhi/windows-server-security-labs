# Windows Server Security — Lab Notes

Hands-on notes and walkthroughs from a Windows Server administration and security course. Covers Active Directory, Group Policy, PKI, file server security, firewall configuration, wireless authentication, and hybrid cloud networking.

---

## 1. Active Directory — Delegation & Least Privilege

### Principle
Use the **Domain Administrator** account as little as possible. Delegate specific tasks to specific users or groups instead of granting full admin rights.

### Delegating Control in ADUC
1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. Right-click the target OU → **Delegate Control**
3. Add the user or group you want to delegate to
4. Choose the specific task (e.g. *Reset user passwords*, *Create/delete user objects*)
5. Finish

> ⚠️ **Important:** Delegating a task at the wrong scope can give unintended rights. For example, delegating password reset at the domain root lets the delegate reset the Domain Admin password — full domain takeover.

### Best Practice
Create a dedicated OU containing only Domain Admin accounts. Delegate tasks *below* that OU, never above it.

### RSAT (Remote Server Administration Tools)
RSAT lets administrators manage Windows Server roles from a Windows client — no need to log into the server directly.

**Install on Windows 10/11:**
1. Settings → Apps → Optional Features → Add a feature
2. Search for the desired RSAT tool
3. Install (requires internet connection)

---

## 2. Restricting Local Administrators via Group Policy

### Goal
Only the IT Support group (and optionally Domain Admins) should have local admin rights on domain-joined machines. Remove everyone else.

### Steps

**1. Create the IT Support group**
- In ADUC: New → Group → Name: `IT Support`
- Add your IT support users as members

**2. Create a new GPO**
- Open **Group Policy Management** (`gpmc.msc`)
- Create a GPO named `Local Admin - IT Support Only`
- Link it to the OU containing your computer accounts

**3. Edit the GPO**
- Navigate to: `Computer Configuration → Preferences → Control Panel Settings → Local Users and Groups`
- Right-click → New → Local Group

**4. Configure the Administrators group**
- Action: **Replace** (removes all existing local admins except built-in Administrator)
- Group name: `Administrators (built-in)`
- Members to add:
  - `DOMAIN\IT Support`
  - `DOMAIN\Domain Admins` *(optional, recommended)*
- Click OK

**5. Apply and test**
- Ensure the GPO is linked to the correct OU
- On a test machine: `gpupdate /force`
- Verify:
  ```cmd
  net localgroup administrators
  ```
- Expected output: `Administrator`, `DOMAIN\IT Support`, `DOMAIN\Domain Admins`

**6. (Optional) Delegate password reset**
- In ADUC, right-click the OU containing user accounts → **Delegate Control**
- Add `IT Support` → choose *Reset Passwords* and related permissions

---

## 3. Public Key Infrastructure (PKI) — Enterprise CA

### What a Certificate Contains
- Encryption type
- Identity of the creator
- Time of use
- Hashing technique

### Installing Active Directory Certificate Services (AD CS)

**On the CA server:**
1. Log in as Domain Admin
2. Server Manager → **Manage** → **Add Roles and Features**
3. Next → Next → Next → **Active Directory Certificate Services** → Add Features
4. Next → Next → Next → **Certification Authority Web Enrollment** → Add Features
5. Next → Next → **Restart if required** → Install

**Post-install configuration:**
1. Click the triangle flag in Server Manager → **Configure Active Directory Certificate Services**
2. Check: **Certification Authority** + **Certification Authority Web Enrollment**
3. Next → **Enterprise CA** → Next → **Root CA**
4. Next → **Create new private key** → Next
5. Cryptographic provider: default
6. Key length: **4096** (longer = harder to crack)
7. Hash: **SHA512**
8. Common name: **do not change** (machines must recognize the CA)
9. Validity period: default or admin-set
10. Configure

**Verify:**
- Tools → **Certification Authority**
- Under `Certification Authority (Local) → [CA-Name]` you'll see folders: *Revoked Certificates*, *Issued Certificates*, *Pending Requests*, *Failed Requests*

---

## 4. Creating Certificates

### Domain Certificate (for a web server)

1. On the CA server: Tools → **IIS Manager**
2. Double-click **Server Certificates** → **Create Domain Certificate**
3. **Common Name** = the owner (most important field)
4. Other fields are optional
5. Select the CA → Next
6. Friendly name: e.g. `Web Certificate` → Finish

**Bind to a website:**
1. IIS → Sites → Default Web Site → **Bindings**
2. Add → **https**
3. SSL certificate: select your new certificate → OK
4. Browse to `https://[server-name]` to verify

> ⚠️ The connection may show as "not secure" until the CA root is trusted by the client.

### Certificate Request (advanced)

1. IIS → Server Certificates → **Create Certificate Request**
2. Common Name = certificate name
3. Cryptographic provider: **RSA**, length **4096**
4. Save the request file
5. Browse to `https://[ca-server]/certsrv/` → **Request a certificate** → **Advanced certificate request**
6. Paste the request → select template → **Submit**
7. Download the certificate
8. IIS → Server Certificates → **Complete Certificate Request**
9. Select the downloaded file → friendly name → **Web Hosting** → OK
10. Bind the new certificate to the site (port 443)

### DNS Alias for the CA
1. On the DC: Tools → **DNS**
2. Forward Lookup Zones → `[domain].local`
3. Right-click → **New Alias (CNAME)**
4. Name: `certificate`
5. Target: the CA server
6. Now `https://certificate` resolves to the CA

### User Certificate

1. Log into a client machine as a normal user
2. Browse to `https://[ca-server]/certsrv/`
3. **Request a certificate** → **User certificate** → Submit

### Auto-Enrollment via GPO

**Goal:** Machines get certificates automatically.

1. DC → Group Policy Management → `[domain].local` → Group Policy Objects
2. Right-click → New → name: `Auto Computer Cert`
3. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Public Key Policies → Certificate Services Client - Auto-Enrollment`
   - Configuration: **Enabled**
   - Check: *Renew expired certificates*, *Update certificates that use certificate templates*
4. Same path → **Automatic Certificate Request Settings**
   - Right-click → New → Automatic Certificate Request
   - Next → **Computer** → Next → Finish
5. Link the GPO to the Computers OU and Servers OU
6. On a client: `gpupdate /force`
7. Verify issued certs in **Certification Authority → Issued Certificates**

### Creating New Certificate Templates
1. CA → Tools → **Certificate Authority**
2. **Certificate Templates** → right-click → **Manage**
3. Select a template → right-click → **Duplicate**
4. Modify settings as needed

---

## 5. Firewall Configuration

### Default Behavior
- **Inbound:** blocked by default
- **Outbound:** allowed by default

### Firewall Profiles
Three profiles, depending on network type:
- **Domain** — machine is on the corporate domain
- **Private** — home or trusted network
- **Public** — untrusted public network

### Allowing RDP from a Specific IP

**On the target server:**
1. Server Manager → Local Server → **Enable Remote Desktop**
2. Tools → **Windows Defender Firewall with Advanced Security**
3. **Inbound Rules** → sort by Local Port → find **3389**
4. Right-click → Properties → **Scope** tab
5. Under *Remote IP address*, add the allowed IP(s)
6. Apply

### Blocking Internet Browsing on a Server

**Option A — Direct firewall rule:**
1. Windows Defender Firewall → **Outbound Rules**
2. Right-click → **New Rule** → Custom → Next
3. All programs → Next
4. Protocol: **TCP** → Remote ports: **80, 443** → Next
5. **Block the connection** → Next
6. Apply to all profiles → Name it → Finish
7. Repeat with **UDP** to cover QUIC (HTTP/3)

**Option B — Group Policy:**
1. DC → Group Policy Management → Group Policy Objects
2. New GPO → name it (e.g. `Block Outbound Web`)
3. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Windows Defender Firewall with Advanced Security → Outbound Rules`
   - New Rule → Custom → TCP → Remote port 80, 443 → Block
4. Link to the target OU
5. `gpupdate /force` on the client

### Enforcing Firewall State via GPO
1. Create/edit a GPO
2. Navigate to:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Windows Defender Firewall with Advanced Security`
3. Click **Windows Defender Firewall Properties**
4. For **Domain**, **Private**, and **Public** profiles:
   - Firewall state: **On**
   - Inbound: **Block**
   - Outbound: **Allow**
5. Link the GPO to the target computers OU
6. `gpupdate /force`
7. Verify with `gpresult /r`

---

## 6. Encrypting File System (EFS)

EFS encrypts files **locally** using a certificate tied to the user account.

### Steps
1. On a client machine, log in as the user who will own the encrypted file
2. `mmc` → File → Add/Remove Snap-in → **Certificates** → Add → OK
3. Navigate to: `Certificates → Personal`
4. Right-click → All Tasks → **Request New Certificate**
5. Next → Next → check **Basic** and **User** → Enroll → Finish
6. Create a test file (e.g. `secret.txt`)
7. Right-click the file → **Properties** → **Advanced** → check **Encrypt contents to secure data** → OK
8. The file icon now shows a lock/encryption indicator

### Access Control
- Only the user who encrypted the file can open it
- You can add another user — but they must have a certificate on that machine
- Recovery options are available at the bottom of the Advanced Attributes window

---

## 7. File Server Permissions & Auditing

### Share Permissions vs NTFS Permissions
**The most restrictive wins.**

Examples:

| Scenario | Share Permission | NTFS Permission | Result |
|----------|-----------------|-----------------|--------|
| Same group | Read | Full | **Read** (AND) |
| Same group | Full | Write | **Write** (AND) |
| Same group | Read | Write | **No access** (AND of conflicting) |
| Different groups | Write (Group A) | Full (Group B) | **Full** (OR) |
| Explicit Deny | Any | Deny | **Deny always wins** |

**Rule of thumb:**
- Multiple groups → **OR** (most permissive wins)
- Explicit deny → **always wins**
- Same group with conflicting share + NTFS → **AND** (most restrictive wins)

### Setting Up a Secure Share

1. Create a folder on the file server (e.g. `D:\Shares\Economy`)
2. Right-click → **Properties** → **Sharing** → **Advanced Sharing**
3. Check **Share this folder** → give it a share name
4. **Permissions** → remove `Everyone` → add `Authenticated Users` → Full Control → OK

**Now configure NTFS:**
1. **Security** tab → **Advanced**
2. **Disable inheritance** → **Convert inherited permissions to explicit**
3. Remove inherited users
4. Add the group that should have access (e.g. `Economy`)
5. Set the appropriate level (Full Control, Modify, Read)
6. Apply

### Denying Access
Add the user/group → check **Deny** for the permission you want to block.
> ⚠️ Deny overrides all other permissions. Use sparingly.

### Auditing File Access

**1. Enable auditing on the folder:**
1. Folder → Properties → **Security** → **Advanced** → **Auditing**
2. Add → select the user/group (or `Everyone`)
3. Type: **All** or **Success/Failure**
4. Applies to: **This folder, subfolders and files**
5. Check: **Delete**, **Delete subfolders and files** (or Full Control)
6. OK

**2. Enable auditing via GPO:**
1. DC → Group Policy Management → new GPO
2. Name: `Auditing File Server`
3. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies → Object Access`
   - Enable **Audit File System**
   - Check **Success** and **Failure**
4. Link the GPO to the Servers OU
5. `gpupdate /force` on the file server

**3. View the logs:**
- File Server → Tools → **Event Viewer**
- `Windows Logs → Security`
- Filter for Event IDs **4660** (file deleted) and **4663** (file accessed)
- You can see *who* deleted/accessed *what* and *when*

### Auditing Who's Connected
- File Server → right-click Windows Start → **Computer Management**
- **Shared Folders** → **Sessions**
- Shows which users have open connections to which shares

### Hidden Shares
Add a `$` at the end of the share name (e.g. `Economy$`).
- The share is hidden from network browsing
- Only users who know the exact path can access it

### Encrypted Files Copied to File Server
If a user shares an EFS-encrypted file to a network share:
- The file is **copied without encryption**
- The destination NTFS permissions apply instead
- This is why auditing matters — you can trace who copied, deleted, or accessed the file

---

## 8. Wireless Network Security (802.1X + RADIUS)

### Goal
Force all domain devices to connect only to an authenticated Wi-Fi network (WPA2-Enterprise) using RADIUS.

### Setup Overview
1. Create a RADIUS profile (IP address + shared secret)
2. Create a WPA2-Enterprise Wi-Fi profile pointing to the RADIUS server
3. Create a `WIFI Computers` group in AD and add authorized computers
4. Create a GPO for certificate auto-enrollment (both computers and RADIUS server need certs)
5. Install **Network Policy and Access Services** role on the DC/NPS server
6. Configure NPS: register in AD, add RADIUS client (the access point), create a network policy
7. Open firewall ports **1812** and **1813** (RADIUS authentication and accounting)
8. Create a GPO forcing clients to use the secure Wi-Fi

### NPS Configuration
1. Tools → **Network Policy Server**
2. Right-click NPS → **Register server in Active Directory**
3. **RADIUS server for 802.1X** → **Configure 802.1X**
4. **Secure Wireless Connection** → add the access point (name + IP + shared secret)
5. Verify → Next → **Microsoft Protected EAP (PEAP)** → Configure
6. `gpupdate /force` to pull the certificate auto-enrollment GPO
7. Add the `WIFI Computers` group
8. Finish

### GPO to Force Secure Wi-Fi
1. New GPO: `Secure WiFi`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Wireless Network (IEEE 802.11) Policies`
   - New → Wireless Network Policy
   - Name: `Wireless Network Policy`
   - Add → Infrastructure
   - Profile name: e.g. `CorpWiFi` | SSID: `CorpWiFi`
   - Security tab → Authentication: **WPA2-Enterprise**
   - Network authentication method → Properties → check *Connect to these servers* → enter `radius.[domain].local`
   - Authentication mode: **User authentication**
3. Link to Computers OU

---

## 9. LLMNR and NetBIOS Security

### Why These Are Risks
- **LLMNR** (Link-Local Multicast Name Resolution) and **NetBIOS** broadcast name resolution requests on the local network
- A **man-in-the-middle** can respond to these broadcasts, impersonating the target host
- This can capture hashed credentials (NTLM relay attacks)

### Disabling NetBIOS on a Client
1. Network adapter → Properties → **IPv4** → Properties → **Advanced**
2. **WINS** tab → **Disable NetBIOS over TCP/IP**
3. OK

### Disabling LLMNR via GPO
1. New GPO: `Disable LLMNR`
2. Edit:
   - `Computer Configuration → Policies → Administrative Templates → Network → DNS Client`
   - **Turn off multicast name resolution** → **Enabled**
3. Link to the target OU

---

## 10. SMB Signing

### Why It Matters
SMB (Server Message Block) transfers data between machines. Without signing, a MITM can impersonate either endpoint.

**SMB signing** ensures both machines prove their identity during every transfer.

### Enable via GPO
1. New GPO: `Enable SMB Signing`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options`
   - **Microsoft network client: Digitally sign communications (always)** → Enabled
   - **Microsoft network server: Digitally sign communications (always)** → Enabled
3. Link to Servers and Computers OU

---

## 11. Cached Logins & Login Screen Hardening

### Why Limit Cached Logins
Windows caches credentials locally so users can log in when the DC is unreachable. If a machine is stolen, those cached hashes can be extracted.

### GPO
1. New GPO: `Limit Cached Logins`
2. Edit:
   - `Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options`
   - **Interactive logon: Number of previous logons to cache** → `0`
   - **Interactive logon: Do not display last user name** → Enabled
   - **Interactive logon: Do not display username @ domain** → Enabled
   - **Interactive logon: Machine inactivity limit** → `60` seconds
3. Link to Computers and Servers OU

---

## 12. USB Storage Blocking

### GPO
1. New GPO: `Block USB`
2. Edit:
   - `Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access`
   - **All Removable Storage classes: Deny all access** → Enabled
3. Link to Computers OU

---

## 13. Network Segmentation & ACLs

### Why Segment
If all machines are on one flat Layer 2 network, any device can broadcast (LLMNR/NetBIOS) and reach any other device. Segmenting via routers/VLANs gives you:
- **ACLs** to control who can reach what
- **Reduced broadcast domain**
- **Better troubleshooting**
- **Improved security posture**

### Example Design
| Subnet | Purpose |
|--------|---------|
| `192.168.96.0/24` | Domain Controllers |
| `192.168.97.0/24` | Certificate Servers |
| `192.168.98.0/24` | Windows 11 Clients |
| `192.168.99.0/24` | File Servers |

- Each subnet on its own VLAN
- Trunk between switch and router
- Router performs inter-VLAN routing
- ACLs restrict traffic between subnets

### ACL Rules
- Specify **source** and **destination** IP addresses
- Use `permit` or `deny`
- Use `established` to allow return traffic only for existing sessions
- Apply the ACL in the correct direction (inbound or outbound on the interface)

### Cisco Packet Tracer Example
**Switch configuration:**
```
enable
configure terminal
interface f0/1
 switchport mode access
 switchport access vlan 10
interface f0/2
 switchport mode access
 switchport access vlan 10
interface f0/3
 switchport mode access
 switchport access vlan 20
interface g0/1
 switchport mode trunk
do write mem
```

**Router configuration:**
```
enable
configure terminal
interface g0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface g0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface g0/0/0
 no shutdown
do write mem
```

---

## 14. Hybrid Cloud & VPN

### Site-to-Site IPsec VPN (On-Prem ↔ Azure)

**On the Azure side:**
1. Create a **Virtual Network (VNet)** with an address space (e.g. `10.1.0.0/16`)
2. Create a **Virtual Network Gateway** (VPN type, Route-based, SKU VpnGw1)
3. Create a **Local Network Gateway** representing the on-prem router (public IP + on-prem subnet)
4. Create a **Connection** (Site-to-site IPsec) with a strong pre-shared key

**On the on-prem firewall (e.g. FortiGate):**
1. **Phase 1 (IKE):** Remote gateway = Azure VPN public IP; IKEv2; AES256; SHA256; DH Group 14; PSK
2. **Phase 2 (IPsec):** Local subnet = on-prem LAN; Remote subnet = Azure VNet; ESP; AES256; SHA256
3. **Static Route:** Destination = Azure VNet → gateway = tunnel interface
4. **Firewall Policies:** Allow LAN → IPsec VPN and IPsec VPN → LAN

**Verify:**
- Azure: Connection status = **Connected**
- FortiGate: `get vpn ipsec tunnel summary` → tunnel = **Up**
- Ping between on-prem and Azure VMs

> ⚠️ Ensure the on-prem and Azure address spaces do **not** overlap.

### Azure Firewall vs NSG

| Feature | NSG | Azure Firewall |
|---------|-----|----------------|
| Purpose | Basic network-level filtering | Full enterprise firewall |
| OSI Layer | 3 & 4 | 3, 4, and 7 |
| Cost | Free | Paid |
| Stateful | Yes | Yes |
| URL filtering | No | Yes |
| Threat protection | No | Yes (IDPS) |
| Logging | Basic | Advanced |
| Application rules | No | Yes |
| Central policy | No | Yes |
| TLS inspection | No | Yes (Premium) |

### IaaS / PaaS / SaaS Responsibility

| Model | Provider Manages | Customer Manages |
|-------|------------------|------------------|
| **IaaS** | Network, storage, virtualization | OS, runtime, apps, data |
| **PaaS** | Above + OS and runtime | Apps, data |
| **SaaS** | Entire stack | App config, user data |
| **On-Prem** | Nothing | Everything |

### CGNAT (Carrier-Grade NAT)
ISPs use CGNAT to share a single public IP among many customers by mapping port numbers.

**Pros:** IPv4 conservation, cost-effective
**Cons:** Breaks port forwarding, harder to trace users, can impact P2P/gaming/VoIP

### VPN vs Tor

| Feature | VPN | Tor |
|---------|-----|-----|
| Hops | 1 server | 3+ relays |
| Speed | Fast | Slow |
| Trust model | Trust the provider | Trust-minimized |
| Best for | Privacy, geo-unblocking | Maximum anonymity |

---

## 15. Switching — RSTP vs LACP

| Feature | RSTP | LACP |
|---------|------|------|
| Purpose | Prevent loops | Aggregate links |
| Layer | Layer 2 | Layer 2 |
| Standard | 802.1w | 802.1AX |
| Behavior | Blocks redundant paths | Uses all links in bundle |
| Convergence | Fast | Immediate failover |
| Multiple links | One active, others blocked | All active |

**Example:** Two switches connected by two cables —
- **Without LACP:** A loop forms; RSTP blocks one link
- **With LACP:** Both links bundle and carry traffic together

**RSTP prevents loops. LACP provides redundancy + performance.**

---

*Coursework — Cloud and Infrastructure Specialist program.*