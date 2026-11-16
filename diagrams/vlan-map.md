# Enterprise VLAN Distribution Map

## 1. Overview & VLAN Allocation Strategy

This document maps every Virtual Local Area Network (VLAN) to its physical distribution across building floors, telecommunications spaces (MDF and IDFs), and connected endpoint device classes.

### Guiding Principles:
1. **Floor Isolation for User Data:** Workstation data subnets are strictly segmented per floor (VLAN 30 for Ground, VLAN 31 for Floor 1, VLAN 32 for Floor 2, VLAN 33 for Floor 3) to contain broadcast storms and limit the blast radius of malware or network loops.
2. **Building-Wide Specialized VLANs:** Systems that require unified management or seamless roaming span all four floors via 802.1Q trunks:
   - **Voice (VLAN 40):** Enables building-wide SIP extension portability and uniform QoS policy.
   - **Corporate Wi-Fi (VLAN 50) & Guest Wi-Fi (VLAN 60):** Enables seamless Layer 2 client roaming between Access Points across all floors without IP re-negotiation.
   - **CCTV (VLAN 70) & Access Control (VLAN 80):** Centralized physical security management.
   - **Printers (VLAN 90):** Uniform printer VLAN accessed through the central print server.
3. **VLAN Pruning on Uplink Trunks:** Trunks between Core and floor access switches carry only the VLANs active on that specific floor, minimizing unnecessary broadcast flooding.
4. **Security Blackhole (VLAN 999):** Defined as the untagged native VLAN on all 802.1Q trunks and assigned to all unused/disabled access switchports.

---

## 2. Visual VLAN Distribution Diagram

```mermaid
flowchart TD
    subgraph CORE_LAYER ["MDF: Ground Floor Server Room - Collapsed Core"]
        CORE["Core Switch Pair (Core-SW-01 & 02)\nHosts All SVIs & L3 Routing"]
        SRV_SW["MDF Server Rack Top-of-Rack Switch\nVLAN 10, 20, 70, 80"]
    end

    subgraph FLOOR_GF ["GROUND FLOOR - MDF & IDF-0"]
        direction TB
        SW_GF["Acc-SW-GF Stack (Catalyst 9300-48P)"]
        DEV_GF_PC["Workstations: VLAN 30 (Untagged)"]
        DEV_GF_VOIP["IP Phones: VLAN 40 (Tagged Voice)"]
        DEV_GF_AP["Wi-Fi APs: VLAN 10 (Native) + 50, 60 (Trunk)"]
        DEV_GF_CAM["CCTV Cameras: VLAN 70 (Untagged)"]
        DEV_GF_ACS["Access Controllers: VLAN 80 (Untagged)"]
        DEV_GF_PRN["Printers: VLAN 90 (Untagged)"]

        SW_GF --> DEV_GF_PC
        SW_GF --> DEV_GF_VOIP
        SW_GF --> DEV_GF_AP
        SW_GF --> DEV_GF_CAM
        SW_GF --> DEV_GF_ACS
        SW_GF --> DEV_GF_PRN
    end

    subgraph FLOOR_1 ["FLOOR 1 - IDF-1 Comms Cabinet"]
        direction TB
        SW_F1["Acc-SW-F1 Stack (Catalyst 9300-48P)"]
        DEV_F1_PC["Workstations: VLAN 31 (Untagged)"]
        DEV_F1_VOIP["IP Phones: VLAN 40 (Tagged Voice)"]
        DEV_F1_AP["Wi-Fi APs: VLAN 10 (Native) + 50, 60 (Trunk)"]
        DEV_F1_CAM["CCTV Cameras: VLAN 70 (Untagged)"]
        DEV_F1_ACS["Access Controllers: VLAN 80 (Untagged)"]
        DEV_F1_PRN["Printers: VLAN 90 (Untagged)"]

        SW_F1 --> DEV_F1_PC
        SW_F1 --> DEV_F1_VOIP
        SW_F1 --> DEV_F1_AP
        SW_F1 --> DEV_F1_CAM
        SW_F1 --> DEV_F1_ACS
        SW_F1 --> DEV_F1_PRN
    end

    subgraph FLOOR_2 ["FLOOR 2 - IDF-2 Comms Cabinet"]
        direction TB
        SW_F2["Acc-SW-F2 Stack (Catalyst 9300-48P)"]
        DEV_F2_PC["Workstations: VLAN 32 (Untagged)"]
        DEV_F2_VOIP["IP Phones: VLAN 40 (Tagged Voice)"]
        DEV_F2_AP["Wi-Fi APs: VLAN 10 (Native) + 50, 60 (Trunk)"]
        DEV_F2_CAM["CCTV Cameras: VLAN 70 (Untagged)"]
        DEV_F2_ACS["Access Controllers: VLAN 80 (Untagged)"]
        DEV_F2_PRN["Printers: VLAN 90 (Untagged)"]

        SW_F2 --> DEV_F2_PC
        SW_F2 --> DEV_F2_VOIP
        SW_F2 --> DEV_F2_AP
        SW_F2 --> DEV_F2_CAM
        SW_F2 --> DEV_F2_ACS
        SW_F2 --> DEV_F2_PRN
    end

    subgraph FLOOR_3 ["FLOOR 3 - IDF-3 Comms Cabinet"]
        direction TB
        SW_F3["Acc-SW-F3 Stack (Catalyst 9300-48P)"]
        DEV_F3_PC["Workstations: VLAN 33 (Untagged)"]
        DEV_F3_VOIP["IP Phones: VLAN 40 (Tagged Voice)"]
        DEV_F3_AP["Wi-Fi APs: VLAN 10 (Native) + 50, 60 (Trunk)"]
        DEV_F3_CAM["CCTV Cameras: VLAN 70 (Untagged)"]
        DEV_F3_ACS["Access Controllers: VLAN 80 (Untagged)"]
        DEV_F3_PRN["Printers: VLAN 90 (Untagged)"]

        SW_F3 --> DEV_F3_PC
        SW_F3 --> DEV_F3_VOIP
        SW_F3 --> DEV_F3_AP
        SW_F3 --> DEV_F3_CAM
        SW_F3 --> DEV_F3_ACS
        SW_F3 --> DEV_F3_PRN
    end

    %% Uplink Trunks from Core to Floor Switches
    CORE ==="802.1Q Trunk (OM4)\nVLANs: 10, 30, 40, 50, 60, 70, 80, 90, 999"=== SW_GF
    CORE ==="802.1Q Trunk (OM4)\nVLANs: 10, 31, 40, 50, 60, 70, 80, 90, 999"=== SW_F1
    CORE ==="802.1Q Trunk (OM4)\nVLANs: 10, 32, 40, 50, 60, 70, 80, 90, 999"=== SW_F2
    CORE ==="802.1Q Trunk (OM4)\nVLANs: 10, 33, 40, 50, 60, 70, 80, 90, 999"=== SW_F3
    CORE ==="10G DAC Bond\nVLANs: 10, 20, 70, 80, 100"=== SRV_SW
```

---

## 3. Floor-by-Floor VLAN Distribution Matrix

| VLAN ID | VLAN Name | Subnet | Ground Floor (MDF) | Floor 1 (IDF-1) | Floor 2 (IDF-2) | Floor 3 (IDF-3) | Primary Connected Endpoints |
|:---:|---|---|:---:|:---:|:---:|:---:|---|
| **10** | `MGMT_INFRA` | `10.10.10.0/24` | ✅ | ✅ | ✅ | ✅ | Switch management SVIs, AP native VLAN, UPS/PDU cards |
| **20** | `SERVERS_PROD` | `10.10.20.0/24` | ✅ | ❌ | ❌ | ❌ | Hypervisors, AD DCs, File, Print, PBX, NVR (MDF rack only) |
| **30** | `DATA_USERS_GF` | `10.10.30.0/24` | ✅ | ❌ | ❌ | ❌ | Ground Floor desktop PCs, docking stations, reception |
| **31** | `DATA_USERS_F1` | `10.10.31.0/24` | ❌ | ✅ | ❌ | ❌ | Floor 1 desktop PCs, docking stations, meeting rooms |
| **32** | `DATA_USERS_F2` | `10.10.32.0/24` | ❌ | ❌ | ✅ | ❌ | Floor 2 desktop PCs, docking stations, collaborative desks |
| **33** | `DATA_USERS_F3` | `10.10.33.0/24` | ❌ | ❌ | ❌ | ✅ | Floor 3 desktop PCs, executive desks, boardroom table |
| **40** | `VOICE_VOIP` | `10.10.40.0/23` | ✅ | ✅ | ✅ | ✅ | Desk IP phones (all floors), SIP conference room phones |
| **50** | `WIFI_CORP` | `10.10.50.0/23` | ✅ | ✅ | ✅ | ✅ | Corporate laptops, company tablets, mobile devices (802.1X) |
| **60** | `WIFI_GUEST` | `10.10.60.0/24` | ✅ | ✅ | ✅ | ✅ | Visitor laptops and phones (Captive portal, Internet only) |
| **70** | `ELV_CCTV` | `10.10.70.0/24` | ✅ | ✅ | ✅ | ✅ | IP dome cameras, bullet cameras, PTZ cameras, NVR ports |
| **80** | `ELV_ACCESS` | `10.10.80.0/24` | ✅ | ✅ | ✅ | ✅ | Biometric door controllers, RFID readers, turnstiles, maglocks |
| **90** | `PRINTERS` | `10.10.90.0/24` | ✅ | ✅ | ✅ | ✅ | Network Multi-Function Printers / Copiers (2 per floor) |
| **100**| `TRANSIT_CORE_FW` | `10.10.100.0/29` | ✅ | ❌ | ❌ | ❌ | MDF Point-to-point transit link between Core switches & NGFW |
| **999**| `NATIVE_BLACKHOLE` | *Unrouted* | ✅ | ✅ | ✅ | ✅ | Untagged native trunk VLAN and all unused switchports |

---

## 4. Switchport Encapsulation & Tagging Standards

To ensure security and operational consistency, access switchports must strictly adhere to the standardized profiles below:

### 4.1 Profile A: Standard User Workstation with IP Phone Daisy-Chain
- **Description:** Port connects to a Cisco IP Phone, which provides an internal 3-port switch passing untagged traffic through to the user workstation PC.
- **Port Mode:** Multi-VLAN Access Port.
- **Access VLAN (Untagged):** Respective Floor Data VLAN (e.g. VLAN 31 on Floor 1).
- **Voice VLAN (Tagged 802.1Q):** VLAN 40 (signaled to phone via CDP / LLDP-MED).
- **Security Features:** Portfast, BPDU Guard, DHCP Snooping, Port Security (max 2 MAC addresses).
```text
interface GigabitEthernet1/0/1
 description USER_DESK_F1_PORT_01
 switchport mode access
 switchport access vlan 31
 switchport voice vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
```

### 4.2 Profile B: Wireless Access Point (Wi-Fi 6 AP)
- **Description:** Ceiling-mounted AP broadcasting corporate and guest SSIDs.
- **Port Mode:** 802.1Q Trunk.
- **Native VLAN (Untagged):** VLAN 10 (Management Plane for AP CAPWAP discovery).
- **Allowed VLANs (Tagged):** VLAN 10, 50 (Corporate Wi-Fi), 60 (Guest Wi-Fi).
```text
interface GigabitEthernet1/0/45
 description WAP_F1_CEILING_01
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10,50,60
 spanning-tree portfast trunk
```

### 4.3 Profile C: Dedicated ELV Device (CCTV Camera / Access Controller)
- **Description:** Dedicated PoE security device with zero user interaction.
- **Port Mode:** Pure Access Port.
- **Access VLAN:** VLAN 70 (CCTV) or VLAN 80 (Access Control).
- **Security Features:** Portfast, BPDU Guard, Strict Port Security (Sticky MAC = 1).
```text
interface GigabitEthernet1/0/35
 description CCTV_F1_LOBBY_CAM_01
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

### 4.4 Profile D: Unused / Spare Switchport (Security Blackhole)
- **Description:** Any unpatched or unassigned copper switchport.
- **Port Mode:** Access.
- **Access VLAN:** VLAN 999 (Blackhole).
- **Admin State:** Administratively Shut Down (`shutdown`).
```text
interface range GigabitEthernet1/0/40 - 44
 description UNUSED_BLACKHOLE
 switchport mode access
 switchport access vlan 999
 shutdown
```

