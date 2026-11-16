# Enterprise VoIP & IP Telephony Architecture

## 1. Overview & Voice Communications Strategy

This document details the engineering design for the unified Voice over IP (VoIP) telephony architecture serving 200 staff and shared facility spaces across the 4-storey enterprise headquarters building.

### Key Objectives:
- Deliver high-fidelity, low-latency enterprise voice communications utilizing standard SIP (Session Initiation Protocol).
- Centralize call processing on an on-premises IP PBX / Unified Communications Manager with redundant SIP trunking.
- Implement strict end-to-end Quality of Service (QoS) guaranteeing high Mean Opinion Scores (MOS $\ge 4.2$).
- Enable automated zero-touch phone provisioning using DHCP Option 150 and LLDP-MED voice VLAN discovery.
- Comply with E911 / emergency calling life-safety standards with precise floor-level caller identification.

---

## 2. IP PBX Call Control & External SIP Trunking

```mermaid
flowchart TD
    subgraph WAN ["External Public Network / ITSP Carrier"]
        CARRIER["ITSP Telecom Carrier\nSIP Softswitch / SBC\nIP: 203.0.113.50"]
    end

    subgraph PERIMETER ["Perimeter Demarcation"]
        NGFW["Next-Generation Firewall\nSIP Inspection / Static 1:1 NAT (198.51.100.11)\nStrict IP Whitelist (No SIP-ALG)"]
    end

    subgraph SERVER_FARM ["MDF: Ground Floor Server Room - Server LAN (VLAN 20)"]
        PBX_PRI["Primary Enterprise IP PBX\n(Cisco CUCM / FreePBX Appliance)\nIP: 10.10.20.15\nSIP Call Control & TFTP Provisioning"]
        PBX_SEC["Secondary Standby PBX\nIP: 10.10.20.18\nHigh-Availability Heartbeat Sync"]
    end

    subgraph CORE_NETWORK ["Collapsed Core Switching (HSRP Gateway)"]
        CORE["Core Switch Pair\nSVI Vlan40 VIP: 10.10.40.1/23\nHardware Strict Priority Queue (PQ) DSCP EF"]
    end

    subgraph ENDPOINTS ["Voice Endpoints (VLAN 40: 10.10.40.0/23)"]
        PH_GF["40x IP Desk Phones (Ground)"]
        PH_F1["60x IP Desk Phones (Floor 1)"]
        PH_F2["60x IP Desk Phones (Floor 2)"]
        PH_F3["40x IP Desk Phones (Floor 3)"]
        CONF["8x SIP Conference Units\n(Meeting Rooms)"]
        RECEPT["4x Reception Attendant Consoles"]
    end

    CARRIER <==|"Dedicated G.711u / TLS SIP Trunk"| NGFW
    NGFW <==|"L3 Transit (VLAN 100)"| CORE
    CORE <==|"LACP Server Link (VLAN 20)"| PBX_PRI
    CORE <==|"LACP Server Link (VLAN 20)"| PBX_SEC
    PBX_PRI <-->|"HA Heartbeat / DB Replication"| PBX_SEC
    CORE ===|"Voice VLAN 40 (OM4 Riser Fiber)"| ENDPOINTS
```

### 2.1 SIP Trunk Sizing & Concurrency Calculation
- Total connected extensions: **212 phones** (200 staff + 8 conference units + 4 reception/security).
- Erlang B Traffic Engineering model:
  - Busy-Hour Calling: Average staff member consumes $0.15\text{ Erlangs}$ of external call traffic during peak hours.
  - Total offered traffic: $200 \times 0.15 = \mathbf{30\text{ Erlangs}}$.
  - Grade of Service (GoS): Target $P \le 0.01$ (less than 1% call blocking probability).
  - Required concurrent SIP trunk channels: **40 Concurrent Call Sessions (SIP Channels)**.
- Bandwidth required on Internet WAN: $40\text{ calls} \times 87.2\text{ kbps (G.711u)} \approx \mathbf{3.49\text{ Mbps}}$ committed information rate (CIR).

---

## 3. Extension Numbering Plan (4-Digit Dial Plan)

The dial plan uses structured 4-digit internal extensions designed around building floor geography:

$$\mathbf{Floor\ Prefix} \;+\; \mathbf{Department\ Code} \;+\; \mathbf{User\ Number}$$

| Extension Range | Location / Category | Subsystem / Role | Direct Inward Dialing (DID) Mapping |
|:---:|---|---|---|
| **`1000 – 1099`** | **Ground Floor** | Reception, Security, Facilities, Comms Room, Ground Users | `+1 (555) 010-1000` to `1099` |
| **`2000 – 2099`** | **Floor 1** | Sales, Marketing, Customer Support, Floor 1 Users | `+1 (555) 010-2000` to `2099` |
| **`3000 – 3099`** | **Floor 2** | Engineering, Software R&D, Operations, Floor 2 Users | `+1 (555) 010-3000` to `3099` |
| **`4000 – 4099`** | **Floor 3** | Executive Suite, Legal, Finance, HR, Boardroom | `+1 (555) 010-4000` to `4099` |
| **`5000 – 5099`** | **Meeting Rooms** | Dedicated SIP Conference Speakerphones (2 per floor) | Internal & Outbound calling |
| **`6000 – 6099`** | **Paging & Intercom** | Emergency Overhead Paging, Door Intercoms | Internal Only |
| **`7000 – 7099`** | **Hunt Groups** | Reception Ring Group (7001), IT Helpdesk (7002), Facilities (7003) | Queued Distribution |
| **`8000 – 8099`** | **System Services** | Automated Attendant / IVR (8000), Voicemail Pilot (8500) | System Automated |
| **`911 / 112 / 999`**| **Emergency Services** | Life Safety Emergency Calling | Immediate PSTN Trunk Override |

---

## 4. Voice Bandwidth & Audio Codec Analysis

| Audio Codec | Sample Rate | Algorithmic Bitrate | Packetization Time | Voice Payload Size | IP/UDP/RTP Overhead | Ethernet Layer 2 Overhead | Total Bandwidth per Call | Audio Quality / Fidelity |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|---|
| **G.711u (PCM)** | 8 kHz | **64.0 kbps** | 20 ms | 160 Bytes | 40 Bytes | 18 Bytes | **87.2 kbps** | Standard PSTN Toll Quality (MOS 4.1) |
| **G.722 (HD Voice)**| 16 kHz | **64.0 kbps** | 20 ms | 160 Bytes | 40 Bytes | 18 Bytes | **87.2 kbps** | **High Definition Wideband (MOS 4.4)** |
| **G.729a (CS-ACELP)**| 8 kHz | **8.0 kbps** | 20 ms | 20 Bytes | 40 Bytes | 18 Bytes | **31.2 kbps** | Compressed WAN / Remote Teleworker (MOS 3.9) |

### Engineering Codec Selection:
1. **Internal Calls (LAN):** **G.722 Wideband HD Audio** is prioritized across all internal desk phones and conference units, delivering crystal-clear speech without consuming additional bandwidth over G.711.
2. **External PSTN Calls (WAN):** **G.711u** is used for standard telco interconnects; fallback to **G.729a** is dynamically triggered for remote teleworkers operating over bandwidth-constrained home broadband VPNs.

---

## 5. Quality of Service (QoS) & Queue Scheduling

To prevent voice packet jitter, latency, and packet loss when large file transfers occur on the network, strict Differentiated Services (DiffServ / RFC 4594) markings are enforced:

```
┌────────────────────────────────────────────────────────────────────────┐
│ VOICE QOS FLOW SPECIFICATION                                           │
│                                                                        │
│  [IP Desk Phone] ──> Emits Voice Media (RTP): Tagged CoS 5 / DSCP EF   │
│                  ──> Emits Signaling (SIP):   Tagged CoS 3 / DSCP CS3  │
│                                                                        │
│  [Access Switch Port] ──> auto qos voip cisco-phone                    │
│                           - Trusts Phone CoS/DSCP                      │
│                           - Untrusted PC port remarked to DSCP 0 (DF)  │
│                                                                        │
│  [Core Switch Uplink] ──> Ingress to Hardware Queues:                  │
│                           - Strict Priority Queue (PQ): DSCP EF        │
│                           - Bandwidth Guaranteed Queue (20%): DSCP CS3 │
│                           - Weighted Round Robin (WDRR 80%): Data      │
└────────────────────────────────────────────────────────────────────────┘
```

- **Voice Latency Budget:** One-way end-to-end delay $\le \mathbf{150\text{ ms}}$ (ITU-T G.114 standard).
- **Jitter Budget:** $\le \mathbf{30\text{ ms}}$.
- **Packet Loss Budget:** $\le \mathbf{1.0\%}$ (target $< 0.25\%$).

---

## 6. Network Integration: Zero-Touch Provisioning & LLDP-MED

When a field technician unboxes an IP phone and patches it into any user wall outlet, configuration occurs automatically without manual static IP assignment:

### 6.1 Voice VLAN Discovery via CDP & LLDP-MED
1. The switchport is configured with:
   ```text
   switchport access vlan 31
   switchport voice vlan 40
   ```
2. The phone boots initially in untagged mode and listens for **Cisco Discovery Protocol (CDP)** and **Link Layer Discovery Protocol - Media Endpoint Discovery (LLDP-MED)** packets from the switch.
3. The switch advertises `Voice VLAN ID: 40`.
4. The phone immediately re-tags its internal network interface to **802.1Q VLAN 40** while keeping its secondary PC pass-through port untagged on **VLAN 31**.

### 6.2 DHCP Option 150 & 66 Provisioning
1. The phone broadcasts a DHCP Request on VLAN 40.
2. The Windows DHCP Server (`10.10.20.10`) provides:
   - IP Address from pool `10.10.40.100` – `10.10.41.220` (Subnet `10.10.40.0/23`).
   - Default Gateway: `10.10.40.1` (Core Switch VIP).
   - **DHCP Option 150 (Cisco TFTP Server):** `10.10.20.15` (PBX IP).
   - **DHCP Option 66 (TFTP Server Name):** `pbx.corp.office.local`.
3. The phone connects to TFTP server `10.10.20.15`, downloads its XML configuration file (`SEP<MAC>.cnf.xml`), applies its extension, upgrades firmware if needed, and registers via SIP.

---

## 7. Emergency Calling (E911) & Branch Survivability

### 7.1 Location-Based Emergency Calling (RAY BAUM'S Act / Kari's Law Compliance)
- When a user dials `911`, `112`, or `999`, the PBX automatically identifies the calling phone's physical location based on its registered switch IP and port.
- The PBX maps the call to a specific **Emergency Location Identification Number (ELIN)** with a dispatchable location:
  - *Example:* Phone on Floor 2, Port 12 transmits ELIN `+1 (555) 010-3001` $\to$ Emergency responders receive: *"HQ Office Building, 2nd Floor, West Wing Desk 12"*.
- The PBX immediately sends an audible alert and email notification to on-site reception and security staff.

### 7.2 PBX High Availability & Local Survivability
- The primary PBX (`10.10.20.15`) replicates database state and active SIP registrations continuously to a secondary PBX node (`10.10.20.18`).
- If the primary PBX fails, phones fail over to the secondary PBX within 10 seconds.
- In the event of an external Internet/WAN link failure, internal extension-to-extension calling across all four floors remains 100% functional through the Core switch.
