# Access Layer Switchport Configuration Templates

## 1. Overview & Standardized Port Profiles

In a modern enterprise network, edge access switchports must be strictly standardized into functional profiles based on the connected endpoint. This ensures consistent security, deterministic QoS marking, and uniform STP protection across all floors.

This document details the configuration, security mechanics, and rationale for all five standard access port profiles deployed across the Ground, 1st, 2nd, and 3rd-floor access switch stacks.

---

## 2. Port Profiles Specification Summary

| Profile | Target Endpoint | Port Mode | Untagged VLAN | Tagged VLAN | PoE Class | Security Controls | QoS Trust |
|---|---|---|:---:|:---:|:---:|---|:---:|
| **Profile 1** | PC + IP Phone | Multi-VLAN Access | Respective Floor Data VLAN (30–33) | Voice (VLAN 40) | 802.3af (15.4W) | Port Security (Max 2), BPDU Guard, DHCP Snooping, DAI | Trust CoS / AutoQoS |
| **Profile 2** | IP CCTV Camera | Dedicated Access | VLAN 70 (ELV_CCTV) | None | 802.3af/at | Sticky MAC (Max 1, Violation Shutdown), BPDU Guard | AF41 (Video) |
| **Profile 3** | Access Controller | Dedicated Access | VLAN 80 (ELV_ACCESS) | None | 802.3af (15.4W) | Sticky MAC (Max 1, Violation Shutdown), BPDU Guard | CS3 (Control) |
| **Profile 4** | Network Printer | Dedicated Access | VLAN 90 (PRINTERS) | None | None | Sticky MAC (Max 1, Violation Restrict), BPDU Guard | Best Effort (0) |
| **Profile 5** | Wi-Fi 6 AP | 802.1Q Trunk | VLAN 10 (MGMT_INFRA) | VLAN 50, 60 | 802.3at (30W) | BPDU Guard, Storm Control, DHCP Snooping | Trust DSCP |
| **Profile 6** | Unused / Spare | Access | VLAN 999 (BLACKHOLE) | None | Disabled | Administratively `shutdown` | N/A |

---

## 3. Detailed Profile Configurations & Command Rationales

### 3.1 Profile 1: User Workstation & IP Phone (Daisy-Chained)

#### Cisco CLI Configuration:
```text
interface GigabitEthernet1/0/1
 description USER_PC_AND_IP_PHONE_DROP
 switchport mode access
 switchport access vlan 31
 switchport voice vlan 40
 auto qos voip cisco-phone
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 storm-control broadcast level 5.0
 storm-control multicast level 10.0
 storm-control action trap
 no shutdown
```

#### Line-by-Line Technical Rationale:
1. `switchport access vlan 31`: Assigns the untagged native access frame to the floor-specific data VLAN (e.g. VLAN 31 for Floor 1). The workstation PC connected to the phone's internal switch sends untagged frames here.
2. `switchport voice vlan 40`: Instructs the Cisco IP phone via Cisco Discovery Protocol (CDP) or LLDP-MED to tag its own RTP audio and SIP signaling packets with 802.1Q tag `VLAN 40` and CoS 5.
3. `auto qos voip cisco-phone`: Automatically classifies voice media (DSCP EF), signaling (DSCP CS3), and places voice packets into the switchport's hardware Strict Priority Queue (PQ) while remarking untrusted PC frames to Best Effort (DSCP 0).
4. `spanning-tree portfast` & `bpduguard enable`: Enables immediate transition from Blocking to Forwarding state (bypassing the 30-second listening/learning delay) so DHCP does not time out. BPDU Guard immediately disables the port if a rogue switch is plugged in.
5. `switchport port-security maximum 2`: Permits exactly two MAC addresses on the physical port: one for the IP phone and one for the connected laptop/desktop.
6. `switchport port-security violation restrict`: If a third device (e.g. unauthorized mini-switch or hub) is attached, packets from the third MAC are dropped, a security violation counter increments, and an SNMP trap is logged to the Syslog server.
7. `storm-control broadcast level 5.0`: Suppresses broadcast radiation if broadcast frames exceed 5% of total line rate.

---

### 3.2 Profile 2: IP CCTV Camera (Physical Security)

#### Cisco CLI Configuration:
```text
interface GigabitEthernet1/0/25
 description ELV_CCTV_CAMERA_DROP
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 storm-control broadcast level 3.0
 storm-control multicast level 5.0
 storm-control action trap
 no shutdown
```

#### Line-by-Line Technical Rationale:
1. `switchport access vlan 70`: Confines the camera into the isolated ELV CCTV broadcast domain.
2. `switchport port-security maximum 1`: Enforces strict 1-to-1 binding between the physical port and the camera.
3. `switchport port-security mac-address sticky`: The switch automatically discovers the camera's hardware MAC address upon first boot and writes it directly into the running configuration.
4. `switchport port-security violation shutdown`: **Zero-Trust Anti-Tamper Protection.** If an attacker unplugs the camera and plugs in a laptop to intercept video feeds or hack the internal network, the switchport transitions immediately into the `err-disable` state, cutting power and link signal.
5. `storm-control broadcast level 3.0`: Cameras communicate via continuous unicast/multicast UDP RTSP streams; strict 3% broadcast capping prevents malfunctioning cameras from flooding the network.

---

### 3.3 Profile 3: Physical Access Control Reader / Door Controller

#### Cisco CLI Configuration:
```text
interface GigabitEthernet1/0/33
 description ELV_ACCESS_CONTROL_CONTROLLER_DROP
 switchport mode access
 switchport access vlan 80
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 storm-control broadcast level 3.0
 storm-control multicast level 5.0
 storm-control action trap
 no shutdown
```

#### Technical Rationale:
- Mirrors the anti-tamper security of CCTV cameras. Access controllers control magnetic door strikes, turnstiles, and server room biometric entry. The sticky MAC violation shutdown rule ensures an intruder cannot detach an exterior door reader to inject forged Wiegand/OSDP door-open commands into the network.

---

### 3.4 Profile 4: Network Multi-Function Printer (MFP)

#### Cisco CLI Configuration:
```text
interface GigabitEthernet1/0/37
 description NETWORK_MFP_PRINTER_DROP
 switchport mode access
 switchport access vlan 90
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 storm-control broadcast level 5.0
 storm-control multicast level 10.0
 storm-control action trap
 no shutdown
```

#### Technical Rationale:
- Placed on the dedicated `VLAN 90` print VLAN.
- Uses `violation restrict` rather than `shutdown` so that accidental laptop docking at a printer maintenance port does not trigger an on-site technician dispatch to un-shut the interface.

---

### 3.5 Profile 5: Wi-Fi 6 Access Point (WAP Trunk)

#### Cisco CLI Configuration:
```text
interface GigabitEthernet1/0/39
 description WIFI6_ACCESS_POINT_TRUNK
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10,50,60
 spanning-tree portfast trunk
 spanning-tree bpduguard enable
 storm-control broadcast level 10.0
 storm-control multicast level 20.0
 storm-control action trap
 no shutdown
```

#### Line-by-Line Technical Rationale:
1. `switchport mode trunk`: The AP is an infrastructure node that simultaneously services multiple wireless networks (SSIDs) mapped to different VLANs.
2. `switchport trunk native vlan 10`: The AP's internal Linux management operating system operates untagged on VLAN 10 (Management), where it obtains a management IP address via DHCP and discovers the Wireless LAN Controller (WLC).
3. `switchport trunk allowed vlan 10,50,60`: Prunes all other VLANs, passing only:
   - `VLAN 10`: AP in-band management and CAPWAP tunnel.
   - `VLAN 50`: Corporate 802.1X wireless clients.
   - `VLAN 60`: Visitor Guest Wi-Fi captive portal clients.
4. `spanning-tree portfast trunk`: Allows the trunk port to enter forwarding state immediately without triggering Spanning Tree Topology Change Notifications (TCNs) when the AP reboots.

---

### 3.6 Profile 6: Unused / Spare Switchport (Security Blackhole)

#### Cisco CLI Configuration:
```text
interface range GigabitEthernet1/0/43 - 48
 description UNUSED_SECURITY_BLACKHOLE
 switchport mode access
 switchport access vlan 999
 shutdown
```

#### Technical Rationale:
- Adheres to the Cisco Enterprise Security Hardening Guide: Any inactive port is isolated from all operational VLANs by assigning it to the unrouted dummy `VLAN 999` and administratively shut down. If an unauthorized user patches an ethernet cable into an open wall jack, the link will not power up or obtain IP transit.

---

## 4. Per-Floor Variable Substitution Reference

When generating configurations from [`access-sw-template.txt`](access-sw-template.txt), use the parameter mappings below:

| Building Floor | Hostname | In-Band Mgmt IP (VLAN 10) | User Data VLAN ID | User Data VLAN Name | Core Uplink Port-Channel |
|---|---|---|:---:|---|:---:|
| **Ground Floor (MDF)** | `Acc-SW-GF` | `10.10.10.10/24` | **30** | `DATA_USERS_GF` | `Port-channel1` |
| **Floor 1 (IDF-1)** | `Acc-SW-F1` | `10.10.10.11/24` | **31** | `DATA_USERS_F1` | `Port-channel2` |
| **Floor 2 (IDF-2)** | `Acc-SW-F2` | `10.10.10.12/24` | **32** | `DATA_USERS_F2` | `Port-channel3` |
| **Floor 3 (IDF-3)** | `Acc-SW-F3` | `10.10.10.13/24` | **33** | `DATA_USERS_F3` | `Port-channel4` |

