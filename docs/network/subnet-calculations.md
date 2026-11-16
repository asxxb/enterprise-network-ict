# Subnet Calculations & Sizing Analysis

## 1. Overview and Design Objectives

A robust enterprise IP addressing architecture requires careful capacity planning. Subnets must be sized with sufficient headroom to prevent IP address exhaustion, while avoiding overly large broadcast domains that degrade network performance.

This document details:
1. Exact host calculations based on the building inventory in [`docs/01-requirements-assumptions.md`](../01-requirements-assumptions.md).
2. Application of the mandatory **30% growth factor**.
3. Theoretical Variable Length Subnet Masking (VLSM) minimum requirements.
4. Comprehensive engineering justifications: **"Why /24 vs /25 vs /27 vs /28 vs /23"** for each specific functional VLAN.
5. Sizing and design of infrastructure interconnects: Point-to-Point links (`/30` vs `/31` RFC 3021), Loopbacks (`/32`), and Firewall Transit (`/29`).

---

## 2. Host Sizing and Growth Modeling

### 2.1 Growth Factor Formula
The enterprise requirement dictates a minimum **30% spare capacity** ($+30\%$) for all device categories:

$$\text{Planned Capacity} = \lceil \text{Initial Device Count} \times 1.30 \rceil$$

Additional overhead is factored in for network infrastructure:
- **Default Gateway (.1):** 1 IP
- **HSRP / VRRP Physical Interfaces & Redundancy:** 2 IPs
- **Static Staging / VIP Buffer:** 2–5 IPs

### 2.2 Subnet Capacity Reference (IPv4)
For an IPv4 prefix of length $n$, the total addresses and usable host capacities are:

$$\text{Total Addresses} = 2^{(32 - n)}, \quad \text{Usable Hosts} = 2^{(32 - n)} - 2$$

| Prefix Length | Subnet Mask | Total Addresses | Usable Hosts | Ideal Use Case |
|:---:|:---|:---:|:---:|:---|
| **/32** | `255.255.255.255` | 1 | 1 | Router ID, Loopback interfaces, Host routes |
| **/31** | `255.255.255.254` | 2 | 2 (RFC 3021) | Point-to-point router/switch links |
| **/30** | `255.255.255.252` | 4 | 2 | Legacy point-to-point links (50% host overhead) |
| **/29** | `255.255.255.248` | 8 | 6 | High-Availability firewall/core transit clusters |
| **/28** | `255.255.255.240` | 16 | 14 | Small isolated device groups (printers, lab rigs) |
| **/27** | `255.255.255.224` | 32 | 30 | Access control, physical security, small server farms |
| **/26** | `255.255.255.192` | 64 | 62 | CCTV subnets, small floor departments |
| **/25** | `255.255.255.128` | 128 | 126 | Medium floor user groups (60–90 users) |
| **/24** | `255.255.255.0` | 256 | 254 | Standard enterprise campus building block |
| **/23** | `255.255.254.0` | 512 | 510 | Unified Voice, high-density Corporate Wi-Fi |

---

## 3. Detailed Worked Subnet Calculations per VLAN

### 3.1 VLAN 10: Management Infrastructure (`MGMT_INFRA`)
- **Initial Devices:**
  - 2 $\times$ Collapsed Core Switches
  - 4 $\times$ Access Switch Stacks (Ground, Floor 1, Floor 2, Floor 3)
  - 12 $\times$ Wi-Fi Access Points (management plane)
  - 2 $\times$ Next-Gen Firewalls (HA pair management interfaces)
  - 4 $\times$ Smart UPS Network Management Cards (NMC)
  - 4 $\times$ Switched Rack PDUs
  - 1 $\times$ Out-of-band Console Terminal Server
  - 1 $\times$ Hardware Wireless LAN Controller (WLC)
  - *Total Initial Devices = 30*
- **Growth Calculation:** $30 \times 1.30 = 39$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) + Staging pool (10 IPs) = 13 IPs.
- **Total Required IPs:** $39 + 13 = 52$ IPs.
- **Minimum Theoretical Prefix:** `/26` (62 usable hosts).
- **Allocated Prefix:** `10.10.10.0/24` (254 usable hosts).
- **Utilization:** Initial $11.8\%$, Growth $20.5\%$.

---

### 3.2 VLAN 20: Production Servers (`SERVERS_PROD`)
- **Initial Devices:**
  - 2 $\times$ Active Directory Domain Controllers / DNS / DHCP (`DC01`, `DC02`)
  - 1 $\times$ File & Storage Server (`FILE01`)
  - 1 $\times$ Print Server (`PRINT01`)
  - 1 $\times$ VoIP PBX Master Server (`VOIP-PBX`)
  - 1 $\times$ CCTV NVR / VMS Head-End (`NVR01`)
  - 1 $\times$ Access Control Server (`ACS01`)
  - 1 $\times$ Backup Appliance / NAS (`BKP01`)
  - 3 $\times$ Virtualization Hypervisor Hosts (ESXi 01–03)
  - 6 $\times$ Server Out-of-band IPMI / iLO Interfaces
  - *Total Initial Devices = 18*
- **Growth Calculation:** $18 \times 1.30 = 23.4 \to 24$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) + Cluster VIPs (.4–.9) = 9 IPs.
- **Total Required IPs:** $24 + 9 = 33$ IPs.
- **Minimum Theoretical Prefix:** `/26` (62 usable hosts) — note that `/27` (30 usable hosts) is insufficient because $33 > 30$.
- **Allocated Prefix:** `10.10.20.0/24` (254 usable hosts).
- **Utilization:** Initial $10.6\%$, Growth $16.9\%$.

---

### 3.3 VLAN 30: Ground Floor Wired Users (`DATA_USERS_GF`)
- **Initial Devices:**
  - 40 $\times$ Staff workstations / desktop docks
  - 4 $\times$ Meeting room wired table ports (2 meeting rooms $\times$ 2 ports)
  - 4 $\times$ Reception and visitor desk wired ports
  - *Total Initial Devices = 48*
- **Growth Calculation:** $48 \times 1.30 = 62.4 \to 63$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $63 + 3 = 66$ IPs.
- **Minimum Theoretical Prefix:** `/25` (126 usable hosts) — note that `/26` (62 usable hosts) is mathematically insufficient for 66 total IPs.
- **Allocated Prefix:** `10.10.30.0/24` (254 usable hosts).
- **Utilization:** Initial $20.1\%$, Growth $26.0\%$.

---

### 3.4 VLAN 31: Floor 1 Wired Users (`DATA_USERS_F1`)
- **Initial Devices:**
  - 60 $\times$ Staff workstations / docking stations
  - 4 $\times$ Meeting room wired table ports (2 meeting rooms $\times$ 2 ports)
  - 6 $\times$ Hot-desk and collaborative bench wired ports
  - *Total Initial Devices = 70*
- **Growth Calculation:** $70 \times 1.30 = 91$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $91 + 3 = 94$ IPs.
- **Minimum Theoretical Prefix:** `/25` (126 usable hosts).
- **Allocated Prefix:** `10.10.31.0/24` (254 usable hosts).
- **Utilization:** Initial $28.7\%$, Growth $37.0\%$.

---

### 3.5 VLAN 32: Floor 2 Wired Users (`DATA_USERS_F2`)
- **Initial Devices:**
  - 60 $\times$ Staff workstations / docking stations
  - 4 $\times$ Meeting room wired table ports (2 meeting rooms $\times$ 2 ports)
  - 6 $\times$ Hot-desk and collaborative bench wired ports
  - *Total Initial Devices = 70*
- **Growth Calculation:** $70 \times 1.30 = 91$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $91 + 3 = 94$ IPs.
- **Minimum Theoretical Prefix:** `/25` (126 usable hosts).
- **Allocated Prefix:** `10.10.32.0/24` (254 usable hosts).
- **Utilization:** Initial $28.7\%$, Growth $37.0\%$.

---

### 3.6 VLAN 33: Floor 3 Wired Users (`DATA_USERS_F3`)
- **Initial Devices:**
  - 40 $\times$ Staff workstations
  - 8 $\times$ Executive boardroom and meeting room ports
  - *Total Initial Devices = 48*
- **Growth Calculation:** $48 \times 1.30 = 62.4 \to 63$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $63 + 3 = 66$ IPs.
- **Minimum Theoretical Prefix:** `/25` (126 usable hosts).
- **Allocated Prefix:** `10.10.33.0/24` (254 usable hosts).
- **Utilization:** Initial $20.1\%$, Growth $26.0\%$.

---

### 3.7 VLAN 40: IP Telephony (`VOICE_VOIP`)
- **Initial Devices:**
  - 200 $\times$ Desk IP phones (Ground: 40, Floor 1: 60, Floor 2: 60, Floor 3: 40)
  - 8 $\times$ SIP Conference room speakerphones (2 per floor $\times$ 4 floors)
  - 4 $\times$ Reception, security desk, and comms room wall phones
  - *Total Initial Devices = 212*
- **Growth Calculation:** $212 \times 1.30 = 275.6 \to 276$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) + PBX TFTP buffer = 5 IPs.
- **Total Required IPs:** $276 + 5 = 281$ IPs.
- **Subnet Sizing Analysis:**
  - A standard `/24` subnet provides **254 usable IP addresses**.
  - **$281 > 254$:** A single `/24` **cannot accommodate** the required 276 phones plus infrastructure.
- **Minimum Theoretical Prefix:** `/23` (510 usable hosts).
- **Allocated Prefix:** `10.10.40.0/23` (Spanning `10.10.40.0` to `10.10.41.255`, 510 usable hosts).
- **Utilization:** Initial $42.5\%$, Growth $55.1\%$.

---

### 3.8 VLAN 50: Corporate Wi-Fi (`WIFI_CORP`)
- **Initial Devices:**
  - 200 $\times$ Corporate staff with laptops, smartphones, and tablets.
  - Multi-device concurrency factor: $1.5$ devices/user active during work hours.
  - $200 \times 1.5 = 300$ active wireless clients.
- **Growth Calculation:** $300 \times 1.30 = 390$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $390 + 3 = 393$ IPs.
- **Subnet Sizing Analysis:**
  - **$393 \gg 254$:** A `/24` subnet would experience immediate DHCP scope exhaustion during core business hours.
- **Minimum Theoretical Prefix:** `/23` (510 usable hosts).
- **Allocated Prefix:** `10.10.50.0/23` (Spanning `10.10.50.0` to `10.10.51.255`, 510 usable hosts).
- **Utilization:** Initial $59.4\%$, Growth $77.1\%$.

---

### 3.9 VLAN 60: Visitor Guest Wi-Fi (`WIFI_GUEST`)
- **Initial Devices:**
  - Estimated concurrent visitors in reception, waiting lounges, and meeting rooms = 60 visitors.
- **Growth Calculation:** $60 \times 1.30 = 78$ devices. Peak seminar/event surge capacity = 150 devices.
- **Overhead:** Gateway (.1) on Firewall = 1 IP.
- **Total Required IPs:** $78 + 1 = 79$ IPs (Peak 151 IPs).
- **Minimum Theoretical Prefix:** `/25` (126 usable hosts) for normal growth; `/24` (254 usable hosts) for event surge.
- **Allocated Prefix:** `10.10.60.0/24` (254 usable hosts).
- **Lease Time Optimization:** Set to **2 hours** to rapidly reclaim IP leases from departed visitors.
- **Utilization:** Normal Growth $31.1\%$, Peak Event $59.4\%$.

---

### 3.10 VLAN 70: IP CCTV Surveillance (`ELV_CCTV`)
- **Initial Devices:**
  - 32 $\times$ PoE IP Cameras (8 per floor across Ground, 1, 2, 3)
  - 2 $\times$ NVR recording video stream interfaces
  - *Total Initial Devices = 34*
- **Growth Calculation:** $34 \times 1.30 = 44.2 \to 45$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $45 + 3 = 48$ IPs.
- **Minimum Theoretical Prefix:** `/26` (62 usable hosts).
- **Allocated Prefix:** `10.10.70.0/24` (254 usable hosts).
- **Utilization:** Initial $14.6\%$, Growth $18.9\%$.

---

### 3.11 VLAN 80: Physical Access Control (`ELV_ACCESS`)
- **Initial Devices:**
  - 16 $\times$ Access control door controllers, biometric readers, and turnstile gateways
  - *Total Initial Devices = 16*
- **Growth Calculation:** $16 \times 1.30 = 20.8 \to 21$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $21 + 3 = 24$ IPs.
- **Minimum Theoretical Prefix:** `/27` (30 usable hosts).
- **Allocated Prefix:** `10.10.80.0/24` (254 usable hosts).
- **Utilization:** Initial $7.5\%$, Growth $9.4\%$.

---

### 3.12 VLAN 90: Network Printers (`PRINTERS`)
- **Initial Devices:**
  - 8 $\times$ Multi-Function Printers / Copiers (2 per floor across Ground, 1, 2, 3)
  - *Total Initial Devices = 8*
- **Growth Calculation:** $8 \times 1.30 = 10.4 \to 11$ devices.
- **Overhead:** Gateway (.1) + HSRP physicals (.2, .3) = 3 IPs.
- **Total Required IPs:** $11 + 3 = 14$ IPs.
- **Minimum Theoretical Prefix:** `/28` (14 usable hosts) or `/27` (30 usable hosts).
- **Allocated Prefix:** `10.10.90.0/24` (254 usable hosts).
- **Utilization:** Initial $4.3\%$, Growth $5.5\%$.

---

## 4. Comprehensive Architectural Justifications: Why /24 vs /25 vs /27 vs /28 vs /23

In academic exercises, engineers often apply "pure VLSM" by assigning tight CIDR blocks (e.g. `/28` to printers, `/27` to access control, `/26` to CCTV, `/25` to floors). However, in professional enterprise campus engineering, addressing strategy is governed by operational reality, troubleshooting efficiency, and route summarization.

Below is the structured analysis comparing both approaches:

### 4.1 Summary Sizing Matrix: Mathematical Need vs. Practical Allocation

| VLAN | Initial Hosts | +30% Growth | Minimum CIDR | Usable Hosts (Min) | Enterprise Allocated CIDR | Usable Hosts (Allocated) | Design Justification |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| **10** (MGMT) | 30 | 39 | **/26** | 62 | **/24** | 254 | Room for 100+ IoT/monitoring sensors & staging |
| **20** (SERVERS) | 18 | 24 | **/26** | 62 | **/24** | 254 | Virtual machines, container pods, SAN/NAS VIPs |
| **30** (USERS_GF) | 48 | 63 | **/25** | 126 | **/24** | 254 | Octet-VLAN mapping; surge desk/visitor events |
| **31** (USERS_F1) | 70 | 91 | **/25** | 126 | **/24** | 254 | High floor density; avoids renumbering |
| **32** (USERS_F2) | 70 | 91 | **/25** | 126 | **/24** | 254 | High floor density; room for team restructures |
| **33** (USERS_F3) | 48 | 63 | **/25** | 126 | **/24** | 254 | Executive suite; conference staging buffer |
| **40** (VOICE) | 212 | 276 | **/23** | 510 | **/23** | 510 | **Mandatory:** Exceeds /24 capacity (276 > 254) |
| **50** (WIFI_CORP) | 300 | 390 | **/23** | 510 | **/23** | 510 | **Mandatory:** Exceeds /24 capacity (390 > 254) |
| **60** (WIFI_GUEST)| 60 | 78 | **/25** | 126 | **/24** | 254 | High visitor turnover; large corporate events |
| **70** (CCTV) | 34 | 45 | **/26** | 62 | **/24** | 254 | Architectural consistency; high camera expansion |
| **80** (ACCESS) | 16 | 21 | **/27** | 30 | **/24** | 254 | Security zone isolation; avoids micro-subnets |
| **90** (PRINTERS) | 8 | 11 | **/28** | 14 | **/24** | 254 | Dedicated print server queue isolation |

---

### 4.2 The "Octet-Aligned" Design Philosophy (`10.10.<VLAN>.0/24`)

In private IPv4 space ($10.0.0.0/8$), the enterprise has **16,777,216 addresses available**. Conserving private IP addresses by carving up irregular bit boundaries (e.g., using `/27` or `/28` on an internal campus LAN) introduces severe operational friction with zero financial or technical benefit:

1. **Instant Human Comprehension:**
   When an engineer or SOC analyst views an IP address such as `10.10.70.25`:
   - Octet 2 (`10`) = Head Office.
   - Octet 3 (`70`) = **VLAN 70 (CCTV)**.
   - Octet 4 (`25`) = Fixed camera 25.
   There is zero cognitive load or lookup required in IPAM.
2. **Simplified ACLs and Firewall Rules:**
   Access lists can use clean wildcard masks (`0.0.0.255`) rather than complex inverse masks (`0.0.0.31` for `/27` or `0.0.0.15` for `/28`). This virtually eliminates human configuration errors in security policies.
3. **Troubleshooting & Packet Analysis:**
   In Wireshark captures, NetFlow records, and SIEM logs, subnets matching `/24` octet boundaries are instantly readable without parsing non-byte-aligned CIDRs.
4. **Broadcast Domain Containment:**
   A `/24` contains at most 254 hosts, which is well below the IEEE 802.1D threshold of ~500–1000 hosts where broadcast traffic begins to impact CPU performance.

---

### 4.3 Why `/27` or `/28` is Inadequate for Long-Term Enterprise Growth

Consider the ELV subnets:
- **Network Printers (`VLAN 90`):**
  - A `/28` mask gives only 14 usable hosts. With 8 initial printers and 30% growth (11 printers), only **3 spare IP addresses remain**. Adding 4 new plotters or desktop label printers would completely exhaust the subnet, forcing the network team to renumber the entire printer fleet, update print queues on all user PCs, and update DNS records.
- **Physical Access Control (`VLAN 80`):**
  - A `/27` mask provides 30 usable hosts. With 21 planned controllers, only **9 spare IPs remain**. Installing automated speed gates in the lobby, smart key cabinets, and server rack electronic locks would instantly exhaust a `/27`.
- **CCTV Surveillance (`VLAN 70`):**
  - A `/26` provides 62 usable hosts. A planned 4K perimeter camera expansion or smart parking camera project would exhaust 62 hosts within 18 months.

**Conclusion:** Allocating `/24` from the start costs nothing in private space, but saves hundreds of hours of operational overhead in re-IPing.

---

### 4.4 Why `/24` Fails and `/23` is Mandatory for Voice and Wi-Fi

A frequent enterprise design flaw is attempting to force every VLAN into a `/24`. This fails catastrophically for Voice and Wireless:

#### 1. IP Voice (`VLAN 40`):
- Total initial phones = 212.
- 30% growth = 276 devices.
- Because a `/24` has **254 usable IPs**, 22 phones would fail to receive a DHCP address if assigned a `/24`.
- **Why `/23` over per-floor Voice VLANs?**
  - If we split voice into 4 per-floor VLANs (GF Voice, F1 Voice, etc.), we would need 4 separate DHCP scopes, 4 separate `switchport voice vlan` definitions, and 4 separate QoS trust boundaries.
  - A single building-wide `/23` (`10.10.40.0/23`, 510 usable hosts) allows a **single, uniform voice VLAN ID** across every switchport in the building, centralized DHCP Option 150/66 configuration, and flawless voice roaming between floors without SIP re-registration delays.

#### 2. Corporate Wi-Fi (`VLAN 50`):
- Modern enterprise staff carry a laptop + smartphone + tablet/smartwatch ($1.5$ devices/user average).
- 200 staff = 300 active devices $\to$ 390 devices with growth.
- Corporate Wi-Fi must also handle transient connection states (devices that enter sleep mode but hold DHCP leases).
- Sizing at `/23` (510 usable IPs) with an 8-hour lease time ensures 100% lease availability without exhausting the scope.

---

## 5. Point-to-Point Links: `/30` vs `/31` (RFC 3021)

For point-to-point Layer 3 links (e.g. Core Switch cross-connects, router links), standard subnetting uses `/30`, but modern networking supports `/31`.

### 5.1 Comparison

| Attribute | `/30` Subnet | `/31` Subnet (RFC 3021) |
|---|---|---|
| **Subnet Mask** | `255.255.255.252` | `255.255.255.254` |
| **Total Addresses** | 4 addresses | 2 addresses |
| **Usable Host Addresses** | **2 usable hosts** | **2 usable hosts** |
| **Address Utilization** | **50% efficiency** (2 lost to Network & Broadcast) | **100% efficiency** (0 lost) |
| **Hardware Compatibility** | Universal (all legacy & modern devices) | Supported by Cisco IOS, IOS-XE, NX-OS, Junos |
| **Configuration Example** | `ip address 10.10.250.1 255.255.255.252` | `ip address 10.10.250.0 255.255.255.254` |

### 5.2 Recommendation
- In this design, **`/31` (RFC 3021)** is the preferred standard on all modern Cisco Catalyst 9000 switches to maximize address efficiency.
- For maximum compatibility with external appliances or packet simulators (such as standard Cisco Packet Tracer versions that do not support RFC 3021), the alternative `/30` addressing plan is provided side-by-side in Section 4.2 of [`vlan-ip-plan.md`](vlan-ip-plan.md).

---

## 6. Loopback Addresses (`/32`)

Loopback interfaces are virtual, software-only interfaces that never experience link-down events unless the host device completely powers off.

- **Mask:** `/32` (`255.255.255.255`).
- **Addresses per Subnet:** 1 total address, 1 usable host (0% overhead).
- **Engineering Purpose:**
  1. **OSPF Router ID:** Provides a stable, highest-priority 32-bit identifier for OSPF Area 0 neighbor adjacencies.
  2. **BGP Peering:** Loopback-to-loopback peering allows BGP sessions to stay up even if an individual physical link fails (via alternate physical paths).
  3. **Management Reachability:** Core-1 (`10.10.255.1`) and Core-2 (`10.10.255.2`) can always be managed regardless of physical port states.

---

## 7. Firewall Transit Subnet (`10.10.100.0/29`)

A common mistake is using a `/30` between Core Switches and a Firewall. While a `/30` suffices for a single switch connected to a single firewall, enterprise resilience demands High Availability (HA) on both sides:

```
[Core-SW-01] (.2) ───────┐         ┌─────── (.5) [NGFW-01]
       │                 ▼         ▼                 │
   (HSRP .1)        [L3 Transit VLAN 100]        (VRRP .4)
       │                 ▲         ▲                 │
[Core-SW-02] (.3) ───────┘         └─────── (.6) [NGFW-02]
```

### Why `/29` is Required for Dual-HA:
1. **Network ID:** `10.10.100.0`
2. **Core Switch HSRP VIP:** `10.10.100.1` (Default route destination for NGFW)
3. **Core-SW-01 Physical Interface:** `10.10.100.2`
4. **Core-SW-02 Physical Interface:** `10.10.100.3`
5. **NGFW Cluster Virtual IP (VIP):** `10.10.100.4` (Default route next-hop for Core switches)
6. **NGFW Primary Node Interface:** `10.10.100.5`
7. **NGFW Secondary Node Interface:** `10.10.100.6`
8. **Subnet Broadcast:** `10.10.100.7`

**Total Addresses Needed = 6 usable host addresses.**
Since a `/30` provides only 2 hosts, a **`/29` (6 usable hosts, mask `255.255.255.248`) is the exact mathematical fit** for redundant dual-chassis Core + dual-appliance Firewall high availability.

---

## 8. Hierarchical Route Summarization Table

The addressing plan aggregates cleanly up to the perimeter firewall and external WAN edge:

| Summarized Block | Mask | Subnets Covered | Purpose | Advertising Boundary |
|---|:---:|---|---|---|
| `10.10.10.0/23` | /23 | VLAN 10 (MGMT) + VLAN 20 (SERVERS) | Infrastructure & Core Services | Core Switch $\to$ NGFW |
| `10.10.30.0/22` | /22 | VLAN 30, 31, 32, 33 (User Floors G, 1, 2, 3) | Wired User Access Summary | Core Switch $\to$ NGFW |
| `10.10.40.0/23` | /23 | VLAN 40 (IP Telephony / Voice) | VoIP Phone Subnet | Core Switch $\to$ NGFW / PBX |
| `10.10.50.0/23` | /23 | VLAN 50 (Corporate Wi-Fi) | Wireless Corporate Summary | Core Switch $\to$ NGFW |
| `10.10.60.0/24` | /24 | VLAN 60 (Guest Wi-Fi) | Guest Wi-Fi (Direct FW interface) | Direct on NGFW |
| `10.10.70.0/23` | /23 | VLAN 70 (CCTV) + VLAN 80 (Access Control) | ELV Physical Security Summary | Core Switch $\to$ NGFW |
| `10.10.90.0/24` | /24 | VLAN 90 (Printers) | Corporate Printers | Core Switch $\to$ NGFW |
| `10.10.250.0/23`| /23 | `10.10.250.0/24` (P2P) + `10.10.255.0/24` (Loopbacks) | Core Routing & Device Loopbacks | Internal OSPF Area 0 |
| **`10.10.0.0/16`** | **/16** | **Entire Headquarters Campus Network** | **Campus Summary Route** | **NGFW $\to$ ISP / SD-WAN / VPN** |

