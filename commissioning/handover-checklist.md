# Project Handover Documentation & Operational Transition Checklist

## 1. Overview & Handover Scope

This document details the formal handover package, operational transfer protocols, credential vaults, manufacturer warranty certificates, and administrator training records transferred from the implementation engineering team to the client's Network Operations Center (NOC) and Facility IT staff.

---

## 2. Handover Deliverables Inventory

| Deliverable Item | File / Artifact Path | Verification Standard | Handover Status | Receiving Team Sign-Off |
|---|---|---|:---:|:---:|
| **Scope & Requirements** | [`00-project-scope.md`](../docs/00-project-scope.md), [`01-requirements.md`](../docs/01-requirements-assumptions.md) | Baseline requirements verified against as-built state | ✅ Received | [  ] Approved |
| **VLAN & IP Addressing Plan** | [`vlan-ip-plan.md`](../docs/network/vlan-ip-plan.md), [`vlan-ip-plan.csv`](../docs/network/vlan-ip-plan.csv) | Full IPAM subnets, gateways, and DHCP scope allocations | ✅ Received | [  ] Approved |
| **Subnet Sizing Calculations** | [`subnet-calculations.md`](../docs/network/subnet-calculations.md) | 30% growth headroom calculations verified | ✅ Received | [  ] Approved |
| **Network Topology Diagrams** | [`physical-topology.md`](../diagrams/physical-topology.md), [`logical-l3.md`](../diagrams/logical-l3-topology.md) | GitHub-native Mermaid diagrams validated | ✅ Received | [  ] Approved |
| **VLAN & ELV Overlay Maps** | [`vlan-map.md`](../diagrams/vlan-map.md), [`elv-overlay.md`](../diagrams/elv-overlay.md) | Port-level mappings and converged backbone overlay | ✅ Received | [  ] Approved |
| **Switch Configurations** | [`core-sw01.txt`](../configs/switches/core-sw01.txt), [`core-sw02.txt`](../configs/switches/core-sw02.txt) | Validated Cisco IOS-XE / Packet Tracer configs | ✅ Received | [  ] Approved |
| **Access Switch Templates** | [`access-sw-template.txt`](../configs/switches/access-sw-template.txt), [`templates.md`](../configs/switches/access-port-templates.md)| Parameterized scripts with 6 port profiles | ✅ Received | [  ] Approved |
| **Switch Verification Guide** | [`verification-commands.md`](../configs/switches/verification-commands.md) | Operational `show` commands and expected outputs | ✅ Received | [  ] Approved |
| **Firewall Design & Rules** | [`firewall-design.md`](../docs/network/firewall-design.md), [`nat-and-rules.txt`](../configs/firewall/nat-and-rules.txt)| Zone matrix, NAT/PAT, SSL-VPN, and SVI ACLs | ✅ Received | [  ] Approved |
| **Wi-Fi 6 Architecture & Plan**| [`wifi-design.md`](../docs/network/wifi-design.md), [`wifi-ap-schedule.csv`](../docs/network/wifi-ap-schedule.csv) | RF channel plan, 802.11k/v/r roaming, 12 AP schedule | ✅ Received | [  ] Approved |
| **Fiber & Transceiver Specs** | [`fiber-uplink-plan.md`](../docs/network/fiber-uplink-plan.md), [`sfp-selection.csv`](../docs/network/sfp-selection.csv) | OM4/OS2 link budgets and TIA-606-B labelling | ✅ Received | [  ] Approved |
| **CCTV System Architecture** | [`cctv-architecture.md`](../docs/elv/cctv-architecture.md) | 32 cameras, H.265, 30-day RAID 6 storage math | ✅ Received | [  ] Approved |
| **Access Control Architecture**| [`access-control-architecture.md`](../docs/elv/access-control-architecture.md) | 16 doors, fail-safe/secure matrix, fire alarm cut | ✅ Received | [  ] Approved |
| **VoIP Telephony Plan** | [`voip-architecture.md`](../docs/elv/voip-architecture.md), [`extension-plan.csv`](../docs/elv/extension-plan.csv)| 4-digit dial plan, G.722 HD voice, DHCP Option 150 | ✅ Received | [  ] Approved |
| **PoE Budget & Cabling Plan** | [`poe-power-budget.md`](../docs/elv/poe-power-budget.md), [`structured-cabling.md`](../docs/elv/structured-cabling.md)| Per-switch PoE calculations, Cat6 rules, UPS sizing | ✅ Received | [  ] Approved |
| **Master Bill of Quantities** | [`boq.csv`](../boq/boq.csv), [`boq-notes.md`](../boq/boq-notes.md) | Itemized material quantities and cost baselines | ✅ Received | [  ] Approved |
| **Commissioning & Test Records**| [`test-plan.md`](test-plan.md), [`snag-report.md`](snag-report.md), [`as-built.md`](as-built-register.md)| 100% passed test cases, 8 snags closed, redlines | ✅ Received | [  ] Approved |

---

## 3. Administrative Credentials & Password Policy Handover

To maintain zero-trust security post-handover, all temporary installation passwords have been rotated into a secure enterprise password vault (Bitwarden / CyberArk / HashiCorp Vault):

### Password Policy Standards:
- Minimum length: **16 characters**.
- Complexity: Upper, lower, numeric, and special characters required.
- Passwords encrypted at rest using Type 9 / Scrypt hashes on all network devices.
- Console access restricted via local AAA with Multi-Factor Authentication (MFA) on jump hosts.

| Subsystem Device | Management URL / Access Method | Administrative User Account | Credential Vault Location | Status |
|---|---|---|---|:---:|
| **Core Switches (01 & 02)** | SSH `10.10.255.1` / `10.10.255.2` | `admin` | Vault: `Network/Cisco_Core_Cluster` | Rotated & Vaulted |
| **Floor Access Switches (GF-F3)**| SSH `10.10.10.10` to `.13` | `admin` | Vault: `Network/Cisco_Access_Stacks` | Rotated & Vaulted |
| **Perimeter Firewalls (HA)** | HTTPS / SSH `10.10.100.4` | `fwadmin` | Vault: `Security/Perimeter_NGFW_HA` | Rotated & Vaulted |
| **Wireless LAN Controllers**| HTTPS `10.10.255.5` | `wlcadmin` | Vault: `Wireless/Catalyst_WLC_HA` | Rotated & Vaulted |
| **CCTV NVR Server** | HTTPS `10.10.20.16:8443` | `nvr_admin` | Vault: `ELV/CCTV_VMS_Master` | Rotated & Vaulted |
| **Access Control Server** | HTTPS `10.10.20.17:443` | `acs_admin` | Vault: `ELV/PACS_HeadEnd_Server` | Rotated & Vaulted |
| **VoIP IP PBX Server** | HTTPS `10.10.20.15:443` | `pbxadmin` | Vault: `VoIP/IP_PBX_CallManager` | Rotated & Vaulted |
| **APC Centralized UPS Units** | Web GUI `10.10.10.20` to `.23` | `apc_admin` | Vault: `Facilities/APC_UPS_NMCs` | Rotated & Vaulted |

---

## 4. Manufacturer Warranties & Support Service Level Agreements (SLAs)

| Hardware Category | Manufacturer / Vendor | Support SLA Package | Coverage Period | Support Contact / Portal |
|---|---|---|:---:|---|
| **Core & Access Switches** | Cisco Systems | **Cisco Smart Net Total Care (8x5xNBD)** | 3 Years (Nov 2026 – Nov 2029) | Cisco TAC (1-800-553-2447 / support.cisco.com) |
| **Perimeter Firewalls** | Fortinet / Palo Alto | **24x7 Enterprise Threat Protection & RMA** | 1 Year (Renewable) | Global Support Portal (support.fortinet.com) |
| **Wireless Controllers & APs**| Cisco Systems | **Cisco Solution Support (24x7x4hr)** | 3 Years (Nov 2026 – Nov 2029) | Cisco TAC Priority Queue |
| **Structured Copper & Fiber**| Leviton / CommScope | **25-Year System Performance Warranty** | 25 Years (2026 – 2051) | Certified Installer Warranty Certificate #CW-2026-991 |
| **CCTV Cameras & NVR** | Hikvision / Axis | **3-Year Advanced Hardware Replacement** | 3 Years (Nov 2026 – Nov 2029) | Authorized Regional Distributor Support |
| **Access Control Hardware** | HID Global / Mercury | **Lifetime Reader / 3-Year Controller Warranty**| 3 Years (Nov 2026 – Nov 2029) | HID Technical Support Desk |
| **APC Smart-UPS Systems** | Schneider Electric / APC | **3-Year Factory Warranty (Electronics + Batt)**| 3 Years (Nov 2026 – Nov 2029) | APC Commercial Support (apc.com/support) |

---

## 5. Operations & Knowledge Transfer Training Summary

Comprehensive technical training sessions were conducted for client network engineers, systems administrators, and facility technicians:

| Training Module | Target Trainees | Duration | Key Topics Covered | Completion Date |
|---|---|:---:|---|:---:|
| **Module 1: Core Switching & Routing** | Network Engineers | 4 Hours | HSRP gateway failover, STP troubleshooting, LACP aggregation, SVI routing, VLAN trunk management. | Nov 15, 2026 |
| **Module 2: Perimeter Security & VPN** | Security Admins | 4 Hours | NAT policy changes, Zone rules modification, SSL-VPN client onboarding, MFA troubleshooting, Syslog analysis. | Nov 15, 2026 |
| **Module 3: Wi-Fi 6 Operations** | Network / Helpdesk | 2 Hours | WLC dashboard, 802.1X client troubleshooting, Guest captive portal user account provisioning, RF interference. | Nov 16, 2026 |
| **Module 4: CCTV & VMS Administration**| Security Team | 3 Hours | Live video matrix layouts, PTZ presets, forensic video export, RAID 6 drive replacement procedure. | Nov 16, 2026 |
| **Module 5: PACS & Life Safety Systems**| Facilities / Security | 3 Hours | Cardholder enrollment, access group assignment, Fire Alarm emergency override testing, battery maintenance. | Nov 16, 2026 |
| **Module 6: VoIP PBX & Phone Provisioning**| Helpdesk Staff | 2 Hours | Adding extensions, ring groups, DHCP Option 150 troubleshooting, phone firmware pushes, E911 location updates. | Nov 16, 2026 |

---

## 6. Spare Parts Inventory Handover

The following emergency spare hardware and installation materials were physically audited and transferred to the client's comms room spare parts cabinet:

| Item Description | Manufacturer Part Number | Quantity Handed Over | Storage Location | Verified By |
|---|---|:---:|---|:---:|
| 10GBASE-SR SFP+ 850nm Multimode Optic | `SFP-10G-SR` | **4 Units** | MDF Rack 01 Storage Drawer | [  ] Verified |
| 1000BASE-LX SFP 1310nm Single-Mode Optic | `GLC-LH-SMD` | **2 Units** | MDF Rack 01 Storage Drawer | [  ] Verified |
| Cat6 RJ-45 Toolless Keystone Jacks (Blue) | `KJ-CAT6-UTP` | **50 Units** | MDF Spare Parts Bin | [  ] Verified |
| Cat6 UTP 3m Molded Patch Cords (Blue) | `PC-CAT6-3M-BLU` | **50 Units** | MDF Spare Parts Shelf | [  ] Verified |
| Cat6 UTP 1m Slimline Patch Cords | `PC-CAT6-1M-SLM` | **50 Units** | MDF Spare Parts Shelf | [  ] Verified |
| LC-LC Duplex OM4 Multimode Patch Cord 2m | `PC-FO-OM4-LC-2M` | **6 Units** | MDF Fiber Cleaning Kit Box | [  ] Verified |
| Enterprise Surveillance 8TB SATA HDD | `HDD-SURV-8TB` | **1 Unit** | NVR Anti-Static Safe Box | [  ] Verified |
| Cisco 7841 4-Line Gigabit IP Phone | `CP-7841-K9` | **4 Units** | IT Storage Room Shelf | [  ] Verified |
| Green Emergency Break-Glass Reset Keys | `AC-BGU-KEY` | **10 Units** | Facilities Key Box | [  ] Verified |

---

## 7. Formal Project Acceptance & Handover Sign-Off

The signatures below signify formal project acceptance, certifying that all deliverables, training, credentials, warranties, and physical equipment have been completely and satisfactorily handed over to the client.

- **Lead Implementation Architect:** __________________________  **Signature:** __________________ **Date:** ____________
- **Client Project Technical Director:** __________________________  **Signature:** __________________ **Date:** ____________
- **Client Head of IT Infrastructure:** __________________________  **Signature:** __________________ **Date:** ____________

