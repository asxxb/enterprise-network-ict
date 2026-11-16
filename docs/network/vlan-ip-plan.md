# VLAN and IP Addressing Plan

## 1. Executive Summary & Design Principles

This document defines the VLAN segmentation, IPv4 addressing architecture, default gateways, static allocations, and DHCP configuration for the 4-storey enterprise headquarters building (Ground + 3 floors).

The design satisfies all requirements specified in [`docs/01-requirements-assumptions.md`](../01-requirements-assumptions.md):
- Strict functional separation of traffic (users, voice, management, servers, CCTV, access control, printers, and guests).
- Standardized octet-aligned addressing scheme based on the `10.10.0.0/16` RFC 1918 private address space.
- Deterministic IP allocation rules (.1 for gateway, reserved ranges, static ranges, and dynamic DHCP pools).
- High availability with HSRP/VRRP VIPs and transit interconnects to the Next-Generation Firewall (NGFW).
- Built-in 30% minimum growth capacity across all subnets.

---

## 2. IP Addressing Architecture

### 2.1 Campus Allocation (`10.10.0.0/16`)
The private block `10.10.0.0/16` is dedicated to the Headquarters campus. The addressing follows a hierarchical 4-octet structure:

$$\mathbf{10} \,.\, \mathbf{Site} \,.\, \mathbf{VLAN / Subnet} \,.\, \mathbf{Host}$$

- **Octet 1 (`10`):** Enterprise RFC 1918 root prefix.
- **Octet 2 (`10`):** Campus Site Identifier (`10` = Headquarters Building).
- **Octet 3 (`X`):** Corresponds directly to the **VLAN ID** (e.g., VLAN 10 = `10.10.10.0`, VLAN 30 = `10.10.30.0`). For `/23` subnets spanning two consecutive octets, the lower octet matches the VLAN ID (e.g., VLAN 40 spans `10.10.40.0` – `10.10.41.255`).
- **Octet 4 (`Y`):** Host identifier adhering to standardized functional blocks.

### 2.2 Standardized Host Addressing Convention
Each `/24` broadcast domain follows a deterministic host numbering policy:

| Host Range | Purpose | Description |
|---|---|---|
| **.1** | Default Gateway | Virtual IP (VIP) for HSRP / VRRP on Core Switches (or Firewall) |
| **.2 – .3** | Layer 3 Physical Interfaces | Core-1 (.2) and Core-2 (.3) physical interface IPs |
| **.4 – .9** | Redundancy & Infrastructure VIPs | Secondary gateways, load balancers, appliance HA interfaces |
| **.10 – .49** | Static Devices | Physical servers, NVRs, access controllers, network printers |
| **.50 – .99** | Reserved Static Buffer | Planned expansions, staging, and static infrastructure |
| **.100 – .220** | DHCP Dynamic Pool | End-user workstations, laptops, IP phones, guest clients |
| **.221 – .254** | Expansion & Temporary DHCP | Spares, overflow, VPN pool, lab testing |
| **.255** | Subnet Broadcast | Directed broadcast address (dropped at L3 boundary) |

---

## 3. Master VLAN & IP Allocation Table

| VLAN ID | VLAN Name | Purpose / Function | Subnet | CIDR | Subnet Mask | Default Gateway | Usable IP Range | DHCP Pool Range | Static / Reserved Range | L3 Routing Gateway Location |
|:---:|:---|:---|:---|:---:|:---|:---|:---|:---|:---|:---|
| **10** | `MGMT_INFRA` | Network In-Band Management (Switches, APs, UPS, PDU) | `10.10.10.0/24` | /24 | `255.255.255.0` | `10.10.10.1` | `10.10.10.1` – `10.10.10.254` | `10.10.10.200` – `10.10.10.220` (Staging) | `10.10.10.2` – `10.10.10.199` | Core Switch (HSRP) |
| **20** | `SERVERS_PROD` | Production On-Prem Servers (AD, DNS, DHCP, PBX, NVR) | `10.10.20.0/24` | /24 | `255.255.255.0` | `10.10.20.1` | `10.10.20.1` – `10.10.20.254` | None (Static / Reservations only) | `10.10.20.2` – `10.10.20.254` | Core Switch (HSRP) / NGFW |
| **30** | `DATA_USERS_GF` | Ground Floor User Workstations (Wired) | `10.10.30.0/24` | /24 | `255.255.255.0` | `10.10.30.1` | `10.10.30.1` – `10.10.30.254` | `10.10.30.100` – `10.10.30.220` | `10.10.30.2` – `10.10.30.99` | Core Switch (HSRP) |
| **31** | `DATA_USERS_F1` | Floor 1 User Workstations (Wired) | `10.10.31.0/24` | /24 | `255.255.255.0` | `10.10.31.1` | `10.10.31.1` – `10.10.31.254` | `10.10.31.100` – `10.10.31.220` | `10.10.31.2` – `10.10.31.99` | Core Switch (HSRP) |
| **32** | `DATA_USERS_F2` | Floor 2 User Workstations (Wired) | `10.10.32.0/24` | /24 | `255.255.255.0` | `10.10.32.1` | `10.10.32.1` – `10.10.32.254` | `10.10.32.100` – `10.10.32.220` | `10.10.32.2` – `10.10.32.99` | Core Switch (HSRP) |
| **33** | `DATA_USERS_F3` | Floor 3 User Workstations (Wired) | `10.10.33.0/24` | /24 | `255.255.255.0` | `10.10.33.1` | `10.10.33.1` – `10.10.33.254` | `10.10.33.100` – `10.10.33.220` | `10.10.33.2` – `10.10.33.99` | Core Switch (HSRP) |
| **40** | `VOICE_VOIP` | IP Phones & Telephony (All Floors) | `10.10.40.0/23` | /23 | `255.255.254.0` | `10.10.40.1` | `10.10.40.1` – `10.10.41.254` | `10.10.40.100` – `10.10.41.220` | `10.10.40.2` – `10.10.40.99` | Core Switch (HSRP) |
| **50** | `WIFI_CORP` | Corporate Wi-Fi (Laptops, Tablets, Mobiles) | `10.10.50.0/23` | /23 | `255.255.254.0` | `10.10.50.1` | `10.10.50.1` – `10.10.51.254` | `10.10.50.50` – `10.10.51.240` | `10.10.50.2` – `10.10.50.49` | Core Switch (HSRP) |
| **60** | `WIFI_GUEST` | Visitor Wi-Fi (Isolated Internet Only) | `10.10.60.0/24` | /24 | `255.255.255.0` | `10.10.60.1` | `10.10.60.1` – `10.10.60.254` | `10.10.60.20` – `10.10.60.250` | `10.10.60.2` – `10.10.60.19` | NGFW (Direct Gateway) |
| **70** | `ELV_CCTV` | IP Cameras & NVR Video Feeds | `10.10.70.0/24` | /24 | `255.255.255.0` | `10.10.70.1` | `10.10.70.1` – `10.10.70.254` | `10.10.70.150` – `10.10.70.200` (Staging) | `10.10.70.2` – `10.10.70.149` (Static) | Core Switch (HSRP) |
| **80** | `ELV_ACCESS` | Access Control Units & Biometric Readers | `10.10.80.0/24` | /24 | `255.255.255.0` | `10.10.80.1` | `10.10.80.1` – `10.10.80.254` | `10.10.80.150` – `10.10.80.200` (Staging) | `10.10.80.2` – `10.10.80.149` (Static) | Core Switch (HSRP) |
| **90** | `PRINTERS` | Network Multi-Function Printers (MFPs) | `10.10.90.0/24` | /24 | `255.255.255.0` | `10.10.90.1` | `10.10.90.1` – `10.10.90.254` | `10.10.90.100` – `10.10.90.150` (Transient) | `10.10.90.2` – `10.10.90.99` (Static) | Core Switch (HSRP) |
| **999** | `NATIVE_BLACKHOLE` | 802.1Q Native Trunk VLAN & Unused Ports | *None* | — | *None* | *None* | *Unrouted* | *None* | *None* | *Unrouted / Blackhole* |

---

## 4. Point-to-Point Links, Loopbacks & Firewall Transit

In addition to user and ELV access subnets, the campus network requires deterministic infrastructure subnets for routing, high availability, and device identification.

### 4.1 Firewall Transit Subnet (`10.10.100.0/29`)
The Core Switch layer connects to the Next-Generation Firewall (NGFW) cluster using a dedicated Layer 3 transit VLAN/subnet (`VLAN 100`). A `/29` subnet provides 6 usable host addresses, accommodating an active/standby Core switch pair and an active/passive or active/active Firewall cluster with dedicated interface IPs and Virtual IPs (VIPs).

| IP Address | Device / Interface | Description |
|---|---|---|
| `10.10.100.1` | Core Switch HSRP VIP | Default Route ($0.0.0.0/0$) return path from Firewall |
| `10.10.100.2` | Core Switch 1 (Core-SW-01) | Physical interface `TenGigE 1/0/48` |
| `10.10.100.3` | Core Switch 2 (Core-SW-02) | Physical interface `TenGigE 1/0/48` |
| `10.10.100.4` | NGFW Cluster VIP | Inside Gateway for Core Switch default route (`ip route 0.0.0.0 0.0.0.0 10.10.100.4`) |
| `10.10.100.5` | NGFW Unit 1 (Primary) | Inside Interface `eth1/1` |
| `10.10.100.6` | NGFW Unit 2 (Secondary) | Inside Interface `eth1/1` |
| `10.10.100.0` | Network Identifier | Subnet ID |
| `10.10.100.7` | Subnet Broadcast | Directed broadcast |

> [!NOTE]
> If a simple single-firewall or active/standby setup with pure routed point-to-point links is deployed, a `/30` (`10.10.100.0/30`) or `/31` (`10.10.100.0/31`) can be substituted: Core = `10.10.100.1`, NGFW = `10.10.100.2`. The `/29` provides complete flexibility for dual-switch and dual-firewall HA.

### 4.2 Switch Point-to-Point Interconnects (`10.10.250.0/24` Block)
Dedicated `/30` or `/31` point-to-point subnets connect Layer 3 routed boundaries:

| Link Purpose | Subnet (/30) | Subnet (/31 RFC 3021) | Node A IP | Node B IP | Notes |
|---|:---:|:---:|:---:|:---:|---|
| **Core-01 $\leftrightarrow$ Core-02 L3 Link 1** | `10.10.250.0/30` | `10.10.250.0/31` | `10.10.250.1` (Core-1) | `10.10.250.2` (Core-2) | Primary L3 routed cross-connect |
| **Core-01 $\leftrightarrow$ Core-02 L3 Link 2** | `10.10.250.4/30` | `10.10.250.2/31` | `10.10.250.5` (Core-1) | `10.10.250.6` (Core-2) | Redundant cross-connect |
| **Core $\leftrightarrow$ Future WAN/SD-WAN Router** | `10.10.250.8/30` | `10.10.250.4/31` | `10.10.250.9` (Core VIP) | `10.10.250.10` (WAN Edge) | Reserved for secondary ISP/SD-WAN |
| **Core $\leftrightarrow$ Out-of-Band Staging** | `10.10.250.12/30` | `10.10.250.6/31` | `10.10.250.13` (Core) | `10.10.250.14` (Staging) | Staging router link |

### 4.3 Loopback Interfaces (`10.10.255.0/24` Block)
Loopback `/32` host routes are configured on all manageable Layer 3 and Layer 2 network devices. These interfaces never go down as long as the device is powered, making them ideal for:
- OSPF / BGP Router IDs
- SSH / Telnet management termination
- SNMP poll destination and Syslog source IP
- RADIUS / TACACS+ client authentication source

| Device Hostname | Role / Location | Loopback IP | Subnet Mask | Description |
|---|---|:---:|:---:|---|
| `Core-SW-01` | Collapsed Core Switch 1 (Ground Server Room) | `10.10.255.1` | `255.255.255.255 (/32)` | Primary OSPF Router ID & SSH target |
| `Core-SW-02` | Collapsed Core Switch 2 (Ground Server Room) | `10.10.255.2` | `255.255.255.255 (/32)` | Secondary OSPF Router ID & SSH target |
| `NGFW-01` | Perimeter Firewall Primary Node | `10.10.255.3` | `255.255.255.255 (/32)` | Internal management termination |
| `NGFW-02` | Perimeter Firewall Secondary Node | `10.10.255.4` | `255.255.255.255 (/32)` | Internal management termination |
| `WLC-01` | Wireless LAN Controller Primary | `10.10.255.5` | `255.255.255.255 (/32)` | CAPWAP tunnel termination / Mgmt |
| `Acc-SW-GF` | Ground Floor Access Switch Stack | `10.10.255.10` | `255.255.255.255 (/32)` | In-band management identifier |
| `Acc-SW-F1` | Floor 1 Access Switch Stack | `10.10.255.11` | `255.255.255.255 (/32)` | In-band management identifier |
| `Acc-SW-F2` | Floor 2 Access Switch Stack | `10.10.255.12` | `255.255.255.255 (/32)` | In-band management identifier |
| `Acc-SW-F3` | Floor 3 Access Switch Stack | `10.10.255.13` | `255.255.255.255 (/32)` | In-band management identifier |

---

## 5. DHCP Scope and Exclusion Plan

Dynamic Host Configuration Protocol (DHCP) services will be hosted centrally on the Windows Server / Active Directory Domain Controller pair (`10.10.20.10` and `10.10.20.11`) with DHCP failover (split-scope 80/20 or active/standby).

The Core Switch SVI interfaces forward DHCP discovery broadcasts to the servers via `ip helper-address 10.10.20.10` and `ip helper-address 10.10.20.11`.
Guest Wi-Fi (VLAN 60) DHCP is handled directly by the NGFW to avoid opening DHCP traffic to the internal server farm.

### 5.1 DHCP Scopes Master Table

| Scope Name | VLAN ID | Network ID / CIDR | Excluded Range (Statics & Spares) | Active DHCP Pool Range | Default Lease Time | Gateway (Option 3) | DNS Servers (Option 6) | Special Options |
|---|:---:|:---:|:---|:---|:---:|:---:|:---|:---|
| `SCOPE_DATA_GF` | 30 | `10.10.30.0/24` | `10.10.30.1` – `10.10.30.99`<br>`10.10.30.221` – `10.10.30.254` | `10.10.30.100` – `10.10.30.220` (121 IPs) | 8 Days | `10.10.30.1` | `10.10.20.10`<br>`10.10.20.11` | Option 15: `corp.office.local` |
| `SCOPE_DATA_F1` | 31 | `10.10.31.0/24` | `10.10.31.1` – `10.10.31.99`<br>`10.10.31.221` – `10.10.31.254` | `10.10.31.100` – `10.10.31.220` (121 IPs) | 8 Days | `10.10.31.1` | `10.10.20.10`<br>`10.10.20.11` | Option 15: `corp.office.local` |
| `SCOPE_DATA_F2` | 32 | `10.10.32.0/24` | `10.10.32.1` – `10.10.32.99`<br>`10.10.32.221` – `10.10.32.254` | `10.10.32.100` – `10.10.32.220` (121 IPs) | 8 Days | `10.10.32.1` | `10.10.20.10`<br>`10.10.20.11` | Option 15: `corp.office.local` |
| `SCOPE_DATA_F3` | 33 | `10.10.33.0/24` | `10.10.33.1` – `10.10.33.99`<br>`10.10.33.221` – `10.10.33.254` | `10.10.33.100` – `10.10.33.220` (121 IPs) | 8 Days | `10.10.33.1` | `10.10.20.10`<br>`10.10.20.11` | Option 15: `corp.office.local` |
| `SCOPE_VOICE` | 40 | `10.10.40.0/23` | `10.10.40.1` – `10.10.40.99`<br>`10.10.41.221` – `10.10.41.254` | `10.10.40.100` – `10.10.41.220` (377 IPs) | 30 Days | `10.10.40.1` | `10.10.20.10`<br>`10.10.20.11` | **Option 150:** `10.10.20.15` (PBX TFTP)<br>**Option 66:** `pbx.corp.office.local` |
| `SCOPE_WIFI_CORP` | 50 | `10.10.50.0/23` | `10.10.50.1` – `10.10.50.49`<br>`10.10.51.241` – `10.10.51.254` | `10.10.50.50` – `10.10.51.240` (447 IPs) | 8 Hours | `10.10.50.1` | `10.10.20.10`<br>`10.10.20.11` | Option 15: `corp.office.local` |
| `SCOPE_WIFI_GUEST` | 60 | `10.10.60.0/24` | `10.10.60.1` – `10.10.60.19`<br>`10.10.60.251` – `10.10.60.254` | `10.10.60.20` – `10.10.60.250` (231 IPs) | 2 Hours | `10.10.60.1` | `1.1.1.1`<br>`8.8.8.8` (Public DNS) | **No Internal DNS**; Captive Portal redirection |
| `SCOPE_MGMT_STAGING` | 10 | `10.10.10.0/24` | `10.10.10.1` – `10.10.10.199`<br>`10.10.10.221` – `10.10.10.254` | `10.10.10.200` – `10.10.10.220` (21 IPs) | 12 Hours | `10.10.10.1` | `10.10.20.10`<br>`10.10.20.11` | **Option 43:** WLC IP (`10.10.255.5`) |

### 5.2 Key DHCP Options Explained
1. **Option 3 (Default Router):** Specifies the default gateway for the subnet (Core SVI VIP `.1` or NGFW `.1`).
2. **Option 6 (Domain Name Server):** Directs corporate clients to internal Active Directory integrated DNS servers (`10.10.20.10`, `10.10.20.11`). Guest clients are handed external public DNS servers (`1.1.1.1`, `8.8.8.8`) to ensure complete isolation.
3. **Option 15 (Domain Name):** Configures DNS suffix (`corp.office.local`) for seamless Kerberos and internal hostname resolution.
4. **Option 150 / Option 66 (TFTP Server / PBX Boot Server):** Supplies IP phones with the IP address (`10.10.20.15`) or FQDN of the VoIP PBX server to download firmware, XML configuration, and SIP registration settings automatically upon boot.
5. **Option 43 (Vendor-Specific Option):** Supplies lightweight Access Points (APs) in VLAN 10 with the IP address of the Wireless LAN Controller (`10.10.255.5`), enabling zero-touch CAPWAP tunnel establishment.

---

## 6. Static Allocation Guidelines for Servers & ELV Systems

Fixed network devices must never rely on dynamic unreserved DHCP. Below is the static assignment mapping:

### 6.1 Server Farm (`VLAN 20: 10.10.20.0/24`)
- `10.10.20.1`: Core Switch Gateway SVI
- `10.10.20.10`: Primary Domain Controller / DNS / DHCP (`DC01`)
- `10.10.20.11`: Secondary Domain Controller / DNS / DHCP (`DC02`)
- `10.10.20.12`: Enterprise File & Storage Server (`FILE01`)
- `10.10.20.13`: Central Print Server (`PRINT01`)
- `10.10.20.15`: Enterprise IP PBX / VoIP Call Manager (`VOIP-PBX`)
- `10.10.20.16`: CCTV Network Video Recorder / VMS Master Server (`NVR-SRV01`)
- `10.10.20.17`: Physical Access Control Head-End Management Server (`ACS-SRV01`)
- `10.10.20.20`: Backup Repository Appliance / NAS (`BKP01`)
- `10.10.20.31` – `10.10.20.33`: Virtualization Hypervisors (ESXi 01-03)
- `10.10.20.41` – `10.10.20.46`: Server Out-of-band IPMI / iLO Management

### 6.2 CCTV Surveillance Systems (`VLAN 70: 10.10.70.0/24`)
- `10.10.70.1`: Default Gateway
- `10.10.70.5`: NVR Interface 1 (Recording Storage Interface)
- `10.10.70.6`: NVR Interface 2 (Redundant Stream Interface)
- `10.10.70.11` – `10.10.70.18`: Ground Floor Cameras (8 IP cameras: GF-CAM-01 to GF-CAM-08)
- `10.10.70.21` – `10.10.70.28`: Floor 1 Cameras (8 IP cameras: F1-CAM-01 to F1-CAM-08)
- `10.10.70.31` – `10.10.70.38`: Floor 2 Cameras (8 IP cameras: F2-CAM-01 to F2-CAM-08)
- `10.10.70.41` – `10.10.70.48`: Floor 3 Cameras (8 IP cameras: F3-CAM-01 to F3-CAM-08)
- `10.10.70.49` – `10.10.70.60`: Reserved for future outdoor / perimeter camera expansion

### 6.3 Access Control Infrastructure (`VLAN 80: 10.10.80.0/24`)
- `10.10.80.1`: Default Gateway
- `10.10.80.5`: Master Access Control Head-End Interface
- `10.10.80.11` – `10.10.80.14`: Ground Floor Door Controllers (Reception, Server Room, Turnstile, Exit)
- `10.10.80.15` – `10.10.80.18`: Floor 1 Door Controllers (Lift Lobby, Executive suite, North/South Doors)
- `10.10.80.19` – `10.10.80.22`: Floor 2 Door Controllers (Lift Lobby, Meeting Suites, R&D Lab)
- `10.10.80.23` – `10.10.80.26`: Floor 3 Door Controllers (Lift Lobby, Boardroom, Executive Offices)
- `10.10.80.27` – `10.10.80.35`: Reserved for Biometric Enroll Stations & Visitor Badging Kiosks

### 6.4 Network Printers (`VLAN 90: 10.10.90.0/24`)
- `10.10.90.1`: Default Gateway
- `10.10.90.11` – `10.10.90.12`: Ground Floor MFPs (`GF-PRN-01`, `GF-PRN-02`)
- `10.10.90.21` – `10.10.90.22`: Floor 1 MFPs (`F1-PRN-01`, `F1-PRN-02`)
- `10.10.90.31` – `10.10.90.32`: Floor 2 MFPs (`F2-PRN-01`, `F2-PRN-02`)
- `10.10.90.41` – `10.10.90.42`: Floor 3 MFPs (`F3-PRN-01`, `F3-PRN-02`)

---

## 7. Inter-VLAN Routing & Security Policy Matrix

Inter-VLAN routing is performed at Layer 3 on the Collapsed Core switch pair for internal high-throughput subnets, with Access Control Lists (ACLs) or Firewall policy enforcing segregation.

```
       [ Internet / WAN ]
               ▲
               │ Public IP / NAT
       ┌───────┴───────┐
       │   Perimeter   │  (VLAN 60 Guest Terminates Here)
       │     NGFW      │
       └───────┬───────┘
               │ Transit VLAN 100 (10.10.100.0/29)
       ┌───────┴───────┐
       │ Collapsed Core│  (L3 SVIs for Corporate VLANs 10-50, 70-90)
       │ Switch Cluster│
       └───┬───┬───┬───┘
           │   │   │
  ┌────────┘   │   └────────┐
  ▼            ▼            ▼
[Data Users] [Voice] [ELV / Servers]
```

### 7.1 Inter-VLAN Traffic Rules Matrix

| Source VLAN | Destination: MGMT (10) | Destination: SERVERS (20) | Destination: USERS (30-33) | Destination: VOICE (40) | Destination: WIFI_CORP (50) | Destination: GUEST (60) | Destination: CCTV (70) | Destination: ACCESS (80) | Destination: PRINTERS (90) | Destination: Internet |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **MGMT (10)** | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ❌ Deny | ✅ Full | ✅ Full | ✅ Full | ⚠️ NTP/Updates only |
| **SERVERS (20)** | ⚠️ Syslog/SNMP | ✅ Full | ⚠️ Established | ⚠️ SIP/RTP | ⚠️ Established | ❌ Deny | ⚠️ VMS API | ⚠️ ACS API | ⚠️ LPD/RAW 9100 | ⚠️ Cloud Backup/Patch |
| **USERS (30-33)** | ❌ Deny (IT jump host only) | ⚠️ Standard Apps (AD, File, Web) | ❌ Deny inter-floor (or restricted) | ⚠️ Softphone signaling only | ❌ Deny | ❌ Deny | ❌ Deny (Web VMS via Server only) | ❌ Deny | ⚠️ Print ports (9100, 631) | ✅ Filtered Internet |
| **VOICE (40)** | ❌ Deny | ⚠️ PBX only (SIP/TFTP) | ❌ Deny | ✅ Intra-VoIP RTP/SIP | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ⚠️ SIP Trunk (via FW) |
| **WIFI_CORP (50)** | ❌ Deny | ⚠️ Standard Apps (AD, File, Web) | ❌ Deny | ❌ Deny | ❌ Client Isolation | ❌ Deny | ❌ Deny | ❌ Deny | ⚠️ Print ports (9100, 631) | ✅ Filtered Internet |
| **GUEST (60)** | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ❌ Client Isolation | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ❌ **DENY ALL INTERNAL** | ✅ Internet Only (HTTP/S) |
| **CCTV (70)** | ❌ Deny | ⚠️ NVR storage stream only | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ✅ Intra-CCTV | ❌ Deny | ❌ Deny | ❌ **NO INTERNET** |
| **ACCESS (80)** | ❌ Deny | ⚠️ ACS DB stream only | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ✅ Intra-Access | ❌ Deny | ❌ **NO INTERNET** |
| **PRINTERS (90)** | ❌ Deny | ⚠️ Print Server only | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ Deny | ❌ **NO INTERNET** |

> [!CAUTION]
> **Zero Trust for ELV Systems:** Both CCTV (VLAN 70) and Access Control (VLAN 80) are strictly isolated with default-deny ACLs blocking outbound Internet connectivity. This completely protects against IoT botnets, unpatched camera firmware exploits, and lateral pivoting.

