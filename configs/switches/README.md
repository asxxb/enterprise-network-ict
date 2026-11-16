# Switch Configurations & Deployment Notes

## 1. Overview

This directory contains production-grade Cisco IOS-XE configuration templates for the enterprise campus network, designed for Cisco Catalyst 9500/9300 series hardware and validated for Cisco Packet Tracer (Catalyst 3650/3560/2960 models).

```
configs/switches/
├── core-sw01.txt               # Primary Collapsed Core Switch configuration
├── core-sw02.txt               # Secondary Collapsed Core Switch configuration
├── access-sw-template.txt      # Universal Floor Access Switch template
├── access-port-templates.md    # Standardized port profiles for all endpoint types
├── verification-commands.md    # Operational verification and troubleshooting guide
└── README.md                   # This implementation and deployment guide
```

---

## 2. Core Switching Layer Architecture

### 2.1 High Availability & Default Gateways (HSRPv2)
- **HSRP Version 2:** Enabled across all Switch Virtual Interfaces (SVIs) to support expanded group IDs and millisecond timers.
- **Master Node (`Core-SW-01`):** Configured with priority **110** and `preempt`. Active gateway for all internal subnets (`10.10.X.1`).
- **Standby Node (`Core-SW-02`):** Configured with priority **100** and `preempt`. Takes over gateway responsibilities within 3 seconds if Core-SW-01 fails.
- **DHCP Relay:** All client SVIs (VLANs 30–33, 40, 50, 90) forward DHCP discovery requests via `ip helper-address` to the primary (`10.10.20.10`) and secondary (`10.10.20.11`) Windows Domain Controllers.

### 2.2 Spanning Tree Protocol (STP)
- **Mode:** Rapid Per-VLAN Spanning Tree (`rapid-pvst`).
- **Root Primary:** `Core-SW-01` set to priority **4096** for all VLANs.
- **Root Secondary:** `Core-SW-02` set to priority **8192** for all VLANs.
- **Floor Switches:** Access switches run default priority (32768), ensuring deterministic root bridge placement at the core.

### 2.3 Layer 3 Transit & Routing Boundary
- **Transit Subnet (`VLAN 100`):** `10.10.100.0/29` interconnects the Core HSRP VIP (`10.10.100.1`) with the Next-Generation Firewall Cluster VIP (`10.10.100.4`).
- **Default Route:** `ip route 0.0.0.0 0.0.0.0 10.10.100.4` directs all outbound Internet traffic to the NGFW.
- **OSPF Area 0:** Configured on both Core switches with passive interfaces on all client SVIs, peering with the firewall over VLAN 100 for dynamic route advertisement.

---

## 3. Access Switching Layer Architecture

### 3.1 Template Design
The file [`access-sw-template.txt`](access-sw-template.txt) is a parameterized configuration template. By substituting variables (`{{HOSTNAME}}`, `{{MGMT_IP}}`, `{{DATA_VLAN}}`, `{{UPLINK_PO}}`), one unified script generates configurations for all floors:

| Variable | Ground Floor (MDF) | Floor 1 (IDF-1) | Floor 2 (IDF-2) | Floor 3 (IDF-3) |
|---|---|---|---|---|
| `{{HOSTNAME}}` | `Acc-SW-GF` | `Acc-SW-F1` | `Acc-SW-F2` | `Acc-SW-F3` |
| `{{MGMT_IP}}` | `10.10.10.10` | `10.10.10.11` | `10.10.10.12` | `10.10.10.13` |
| `{{DATA_VLAN}}` | `30` | `31` | `32` | `33` |
| `{{DATA_VLAN_NAME}}` | `DATA_USERS_GF` | `DATA_USERS_F1` | `DATA_USERS_F2` | `DATA_USERS_F3` |
| `{{UPLINK_PO}}` | `1` | `2` | `3` | `4` |

### 3.2 Layer 2 Security Suite
All access switches implement defense-in-depth:
1. **DHCP Snooping:** Drops unauthorized DHCP offers from untrusted access ports. Only uplinks to Core switches are trusted.
2. **Dynamic ARP Inspection (DAI):** Intercepts and validates all ARP requests against the DHCP snooping binding database.
3. **BPDU Guard & Portfast:** Protects against rogue switches or accidental loops on end-user ports.
4. **Storm Control:** Caps broadcast at 5% and multicast at 10% to prevent broadcast radiation.
5. **Port Security:** Restricts MAC address spoofing and rogue hub attachment.

---

## 4. Cisco Packet Tracer Validation Guide

To simulate and test these configurations in Cisco Packet Tracer:

### Recommended Packet Tracer Device Types:
- **Core Switches:** Use **Multilayer Switch 3650-24TS** or **3560-24PS**.
- **Access Switches:** Use **2960-24TT** or **3650-24TS** (in L2 switchport mode).
- **Firewall:** Use **ASA 5506-X** or a router representing the NGFW.
- **Servers:** Generic Server appliances with static IP configurations.

### Configuration Steps in Packet Tracer:
1. Open the CLI tab of `Core-SW-01` and enter `enable` mode.
2. Paste the contents of `core-sw01.txt`.
3. If Packet Tracer flags `algorithm-type scrypt`, change to `username admin privilege 15 secret Cisco@Admin2026!` (standard MD5/Type 5 secret supported by Packet Tracer).
4. Paste `core-sw02.txt` on the second core switch.
5. Apply the parameterized template on the access switches.
6. Verify HSRP and STP states using the commands in [`verification-commands.md`](verification-commands.md).

