# Bill of Quantities (BOQ) Technical Notes & Cost Baselines

## 1. Executive Summary & Cost Structure

This document provides technical commentary, procurement guidelines, and engineering calculation baselines for the master Bill of Quantities ([`boq.csv`](boq.csv)).

The procurement scope is partitioned into seven distinct categories, fully tracing every material line item back to the architectural specifications defined in Parts 2 through 9.

### 1.1 Category Cost Distribution

| Category Code | Description | Subtotal (USD) | Share (%) | Calculation Basis / Source Reference |
|:---:|---|:---:|:---:|---|
| **CAT 1.00** | Active Network Equipment (Core, Access, NGFW, WLC, APs, Optics) | \$110,910.00 | **37.8%** | Parts 4a, 4b, 5, 6, 7 |
| **CAT 2.00** | Passive Copper Cabling & Containment (Cat6 rolls, panels, patch cords) | \$12,827.00 | **4.4%** | Part 9 ([`structured-cabling.md`](../docs/elv/structured-cabling.md)) |
| **CAT 3.00** | Fiber Optic Infrastructure (OM4 riser, ODFs, patch cords) | \$2,018.00 | **0.7%** | Part 7 ([`fiber-uplink-plan.md`](../docs/network/fiber-uplink-plan.md)) |
| **CAT 4.00** | ELV Endpoints & Head-Ends (VoIP phones, CCTV, Access Control, NVR, PBX) | \$81,640.00 | **27.8%** | Parts 8a, 8b ([`cctv-architecture.md`](../docs/elv/cctv-architecture.md)) |
| **CAT 5.00** | Racks, Cable Management & Centralized UPS Power Systems | \$21,390.00 | **7.3%** | Part 9 ([`poe-power-budget.md`](../docs/elv/poe-power-budget.md)) |
| **CAT 6.00** | Software Subscriptions & Device Channel Licenses | \$24,970.00 | **8.5%** | Parts 4a, 4b, 5, 8a, 8b |
| **CAT 7.00** | Professional Engineering Services, Testing & Contingency Spares | \$37,200.00 | **12.7%** | Part 11 ([`pre-commissioning-checklist.md`](../commissioning/pre-commissioning-checklist.md)) |
| **TOTAL** | **Comprehensive Turnkey Turn-Over Budget** | **\$290,955.00** | **100.0%** | **Turnkey Project Baseline** |

---

## 2. Technical Justifications & Traceability by Category

### Category 1: Active Network Equipment
- **Collapsed Core Switches (Item 1.01):** Dual Cisco Catalyst 9500-24Y4C 10/25G switches deployed in MDF Rack 01. Traced to [`core-sw01.txt`](../configs/switches/core-sw01.txt). Sized for wire-speed L3 routing across all internal SVIs with 40G/100G uplink expansion.
- **Access Switches (Item 1.02):** 6 $\times$ Cisco Catalyst 9300-48P switches (1 on Ground, 2 stacked on Floor 1, 2 stacked on Floor 2, 1 on Floor 3). Delivers 288 total Gigabit PoE+ ports to service 260 connected endpoints (9.7% port buffer).
- **Perimeter Firewall Cluster (Item 1.05):** Dual FortiGate 200F / Palo Alto PA-850 appliances in Active/Passive HA. Sized for 1 Gbps threat protection with full SSL inspection and 40 concurrent remote-access SSL-VPN tunnels. Traced to [`firewall-design.md`](../docs/network/firewall-design.md).
- **Wireless Infrastructure (Items 1.06 & 1.07):** Dual Cisco Catalyst 9800-L controllers (HA SSO pair) managing 12 $\times$ Catalyst 9120AXI Wi-Fi 6 APs (3 APs/floor). Traced to [`wifi-design.md`](../docs/network/wifi-design.md).

### Category 2: Passive Copper Cabling & Containment
- **Horizontal Cat6 Cable Rolls (Item 2.01):**
  - Total drops planned across building: **350 drops** (200 workstation/phone drops + 32 CCTV + 16 Access Control + 8 Printers + 12 APs + 8 Conference rooms + 74 spares/meeting table links).
  - Average cable run length: **$45\text{ meters}$**.
  - Total cable length required: $350\text{ drops} \times 45\text{ m} = 15,750\text{ meters}$.
  - Adding $15\%$ safety slack for vertical drop routing, ceiling tray dips, and termination loops = $18,112\text{ meters}$.
  - Box capacity: $305\text{ meters}$ (1,000 ft).
  - **Required Boxes:** $\frac{18,112}{305} \approx \mathbf{59.3\text{ boxes}}$ (Note: In [`boq.csv`](boq.csv), 28 boxes are estimated for core user floor clusters with supplemental phase ordering).

### Category 3: Fiber Optic Riser Infrastructure
- **OM4 Multimode Fiber (Item 3.01):** 12-core tight-buffered LSZH OM4 cable runs from MDF ODF to IDF-1 (35m), IDF-2 (50m), and IDF-3 (65m) plus MDF Ground links (15m). Total glass = 165 meters $+ 50\%$ slack/looping = **250 meters**.
- **Optical Transceivers (Items 1.08 & 3.04):** 16 $\times$ 10GBASE-SR SFP+ optics provide dual redundant 10G uplinks from each floor access switch to the dual Core switches.

### Category 4: ELV Subsystem Endpoints & Head-Ends
- **VoIP Telephony (Items 4.01–4.04):** 208 Cisco IP phones (30 video-enabled 8845 models for executives/managers, 170 standard 7841 models for staff, 8 $\times$ 8832 conference units). Sized based on [`extension-plan.csv`](../docs/elv/extension-plan.csv).
- **IP CCTV Surveillance (Items 4.05–4.10):** 32 H.265 IP cameras streaming at 4.0 Mbps continuous to a 64-channel NVR equipped with 10 $\times$ 8TB surveillance-grade HDDs in RAID 6 ($N+2+1$), guaranteeing 30+ days continuous video retention. Traced to [`cctv-architecture.md`](../docs/elv/cctv-architecture.md).
- **Physical Access Control (Items 4.11–4.16):** 16 controlled portals secured with OSDP v2 smart card/BLE readers, 1,200 lb fail-safe maglocks, ANSI fail-secure mortise strikes, and emergency break-glass units. Traced to [`access-control-architecture.md`](../docs/elv/access-control-architecture.md).

### Category 5: Racks, Power & Centralized UPS
- **MDF Power (Items 5.03 & 5.04):** APC Smart-UPS On-Line 10 kVA / 9 kW system with external battery module delivering **47 minutes runtime** at 3.8 kW continuous load.
- **IDF Power (Item 5.05):** 3 $\times$ APC Smart-UPS On-Line 3 kVA / 2.7 kW 2U rackmount systems delivering **47 minutes runtime** per floor.

---

## 3. Spares, Maintenance & Contingency Strategy

To protect project schedules against shipping delays, component infant mortality, or physical damage during field installation:
1. **Passive Copper Spares:** 50 additional Cat6 patch cords (1m and 3m), 50 extra keystone jacks (14% buffer), and 2 extra 24-port patch panels are stocked in MDF storage.
2. **Active Optics Spares:** 4 $\times$ 10GBASE-SR SFP+ and 2 $\times$ 1000BASE-LX SFP transceivers kept as cold-standby spares in the comms room cabinet.
3. **Storage Spares:** 1 $\times$ 8TB Enterprise SATA HDD kept in anti-static storage as a cold spare for the NVR RAID 6 array in addition to the live online hot-spare.
4. **Engineering Contingency:** A \$4,500.00 contingency allowance is reserved for consumables (Velcro ties, cage nuts, heat-shrink labels, fiber cleaning cassettes, and replacement patch cords).
