# Enterprise Office Network & ELV/ICT Infrastructure Design

[![Release](https://img.shields.io/badge/release-v1.0-blue.svg)](https://github.com/)
[![Documentation](https://img.shields.io/badge/docs-complete-brightgreen.svg)](docs/)
[![Architecture](https://img.shields.io/badge/architecture-collapsed--core-orange.svg)](diagrams/)

A comprehensive, simulated engineering design of a converged 4-storey commercial office network (Ground + 3 floors) servicing ~200 corporate staff and unified Extra Low Voltage (ELV/ICT) subsystems including IP CCTV, Physical Access Control, and VoIP telephony.

All parameters, device counts, and floor layouts are baseline assumptions documented in [`docs/01-requirements-assumptions.md`](docs/01-requirements-assumptions.md).

---

## Architecture Overview

```
                      [ 1 Gbps ISP Fiber Handoff (OS2 SMF) ]
                                       ▲
                                       │ 1000BASE-LX
                          ┌────────────┴────────────┐
                          │   High-Availability     │ (Direct Gateway for
                          │ Perimeter NGFW Cluster  │  VLAN 60 Guest Wi-Fi)
                          └────────────┬────────────┘
                                       │ 10G L3 Transit (VLAN 100: 10.10.100.0/29)
                          ┌────────────┴────────────┐
                          │  Collapsed Core Switch  │ (HSRPv2 Active/Standby SVIs
                          │  Pair (Catalyst 9500)   │  for VLANs 10, 20, 30-50, 70-90)
                          └──────┬───────────┬──────┘
                                 │           │
       ┌─────────────────────────┘           └─────────────────────────┐
       ▼ (2x 10G OM4 LACP LAG)                                         ▼ (2x 10G OM4 LACP LAG)
 ┌─────────────┐                                                 ┌─────────────┐
 │ Floor 1 IDF │ Acc-SW-F1 Stack (2x 48P)                        │ Floor 2 IDF │ Acc-SW-F2 Stack (2x 48P)
 └──────┬──────┘                                                 └──────┬──────┘
        ├─ 60x Workstations (VLAN 31)                                   ├─ 60x Workstations (VLAN 32)
        ├─ 60x IP Phones (VLAN 40)                                      ├─ 60x IP Phones (VLAN 40)
        ├─ 3x Wi-Fi 6 APs (VLAN 10/50/60)                               ├─ 3x Wi-Fi 6 APs (VLAN 10/50/60)
        ├─ 8x CCTV Cameras (VLAN 70)                                    ├─ 8x CCTV Cameras (VLAN 70)
        └─ 4x Access Controllers (VLAN 80)                              └─ 4x Access Controllers (VLAN 80)
```

### Key Engineering Specifications:
- **Core Switching:** Cisco Catalyst 9500-24Y4C pair with HSRPv2 `.1` virtual default gateways and Rapid-PVST root primary/secondary roles.
- **Access Switching:** Cisco Catalyst 9300-48P PoE+ stacks providing 802.3af/at inline power, sticky MAC port security, and DHCP snooping.
- **Riser Backbone:** Dual OM4 multimode fiber runs ($< 70\text{ m}$) configured as 20 Gbps (2 $\times$ 10G) LACP Port-Channels.
- **Horizontal Edge:** 350 Cat6 UTP permanent links ($< 90\text{ m}$, ANSI/TIA-568-D) terminating at 1U 24-port patch panels.
- **Perimeter Edge:** Active/Passive NGFW cluster enforcing 5 security zones, dynamic PAT, SSL-VPN with MFA, and SVI micro-segmentation.
- **Wireless Infrastructure:** On-premises Cisco Catalyst 9800-L WLC managing 12 $\times$ Wi-Fi 6 APs with sub-30ms 802.11k/v/r fast roaming.
- **ELV Subsystems:** 32 H.265 CCTV cameras (30-day RAID 6 storage), 16 OSDP v2 access-controlled portals with Fire Alarm emergency release, and 208 VoIP endpoints using wideband G.722 audio.

---

## Repository Structure & Navigation Guide

```
enterprise-network-ict-design/
├── README.md                           # Master project summary & navigation index
├── LICENSE                             # Permissive open documentation license
├── .gitignore                          # Standard exclusion rules
├── boq/                                # Part 10: Bill of Quantities & Procurement
│   ├── boq.csv                         # Itemized 50-line turnkey cost schedule
│   └── boq-notes.md                    # Pricing models, baselines & spares strategy
├── commissioning/                      # Part 11: QA, Testing, Snagging & Handover
│   ├── pre-commissioning-checklist.md  # Physical, electrical & cabling checks
│   ├── test-plan.md                    # Layer 1–7 test cases with pass/fail criteria
│   ├── snag-report.md                  # Defect log with root cause & resolution
│   ├── as-built-register.md            # Redlines, variations & final As-Built tables
│   └── handover-checklist.md           # Client credentials, warranties & training
├── configs/
│   ├── firewall/                       # Part 5: Firewall Policies & NAT
│   │   ├── policy-matrix.csv           # 12-rule security zone matrix
│   │   └── nat-and-rules.txt           # NGFW CLI rules, PAT/DNAT & SVI ACLs
│   └── switches/                       # Part 4: Switch Configurations
│       ├── core-sw01.txt               # Primary Core switch (Active HSRP / STP Root)
│       ├── core-sw02.txt               # Secondary Core switch (Standby HSRP / Root 2)
│       ├── access-sw-template.txt      # Universal floor access switch template
│       ├── access-port-templates.md    # 6 standardized switchport profiles
│       ├── verification-commands.md    # Operational verification show commands
│       └── README.md                   # Switch deployment & Packet Tracer guide
├── diagrams/                           # Part 3: Topology Diagrams (GitHub Mermaid)
│   ├── physical-topology.md            # Physical layout, media & cable color codes
│   ├── logical-l3-topology.md          # L3 SVIs, routing paths & security zones
│   ├── vlan-map.md                     # Floor-by-floor VLAN distribution map
│   └── elv-overlay.md                  # CCTV, access control & VoIP convergence
└── docs/
    ├── 00-project-scope.md             # Project boundaries & deliverables checklist
    ├── 01-requirements-assumptions.md  # Building inventory, user counts & constraints
    ├── resume-mapping.md               # Resume competency proof matrix
    ├── network/                        # Parts 2, 5, 6, 7: Core Network Engineering
    │   ├── vlan-ip-plan.md             # Master VLAN, IP & DHCP architecture
    │   ├── vlan-ip-plan.csv            # Machine-readable IPAM table
    │   ├── subnet-calculations.md      # Worked sizing & /24 vs /23 justification
    │   ├── firewall-design.md          # Security zones, PAT & VPN architecture
    │   ├── wifi-design.md              # Wi-Fi 6 RF channels, roaming & capacity
    │   ├── wifi-ap-schedule.csv        # 12 AP hardware & switchport schedule
    │   ├── fiber-uplink-plan.md        # OM4/OS2 link budget & TIA-606-B labelling
    │   └── sfp-selection.csv           # SFP/SFP+ transceiver schedule
    └── elv/                            # Parts 8, 9: ELV / ICT Infrastructure
        ├── cctv-architecture.md        # 32 cameras, H.265 bitrate & NVR RAID 6 math
        ├── access-control-architecture.md # 16 doors, fail-safe/secure & fire interface
        ├── voip-architecture.md        # Dial plan, G.722 codecs & Option 150
        ├── extension-plan.csv          # 4-digit telephone extension schedule
        ├── poe-power-budget.md         # Per-switch PoE headroom & UPS runtimes
        ├── poe-budget.csv              # Switchport PoE allocation table
        ├── structured-cabling.md       # Cat6 permanent link rules & rack elevations
        └── outlet-schedule.csv         # 350-drop horizontal outlet schedule
```

---

## Deliverables Status & Progress

- [x] **Part 1:** Scope and requirements definition ([`docs/01-requirements.md`](docs/01-requirements-assumptions.md))
- [x] **Part 2:** VLAN and IP addressing plan ([`docs/network/vlan-ip-plan.md`](docs/network/vlan-ip-plan.md))
- [x] **Part 3:** Topology diagrams ([`diagrams/physical-topology.md`](diagrams/physical-topology.md))
- [x] **Part 4a:** Core and distribution switch configurations ([`configs/switches/core-sw01.txt`](configs/switches/core-sw01.txt))
- [x] **Part 4b:** Access switch configuration and templates ([`configs/switches/access-sw-template.txt`](configs/switches/access-sw-template.txt))
- [x] **Part 5:** Firewall, NAT, and ACL design ([`docs/network/firewall-design.md`](docs/network/firewall-design.md))
- [x] **Part 6:** Enterprise Wi-Fi 6 design ([`docs/network/wifi-design.md`](docs/network/wifi-design.md))
- [x] **Part 7:** Fiber uplinks and optical transceiver selection ([`docs/network/fiber-uplink-plan.md`](docs/network/fiber-uplink-plan.md))
- [x] **Part 8a:** IP CCTV and Physical Access Control architecture ([`docs/elv/cctv-architecture.md`](docs/elv/cctv-architecture.md))
- [x] **Part 8b:** VoIP telephony architecture and extension plan ([`docs/elv/voip-architecture.md`](docs/elv/voip-architecture.md))
- [x] **Part 9:** PoE power budget and structured cabling ([`docs/elv/poe-power-budget.md`](docs/elv/poe-power-budget.md))
- [x] **Part 10:** Bill of Quantities ([`boq/boq.csv`](boq/boq.csv))
- [x] **Part 11a:** Pre-commissioning checklist and test plan ([`commissioning/test-plan.md`](commissioning/test-plan.md))
- [x] **Part 11b:** Snag report and As-Built documentation ([`commissioning/as-built-register.md`](commissioning/as-built-register.md))
- [x] **Part 12:** Release v1.0, resume proof mapping and documentation polish ([`docs/resume-mapping.md`](docs/resume-mapping.md))

---

## Resume Proof Mapping

For hiring managers and technical recruiters evaluating networking, cybersecurity, and ELV skills, refer to [`docs/resume-mapping.md`](docs/resume-mapping.md) for a line-by-line mapping between standard senior engineer resume bullets and the corresponding design artifacts in this repository.

---

## Standards Referenced
- **IEEE:** 802.1Q (VLAN), 802.3ad (LACP), 802.3af/at (PoE/PoE+), 802.11ax (Wi-Fi 6), 802.11k/v/r (Fast Roaming), 802.1X (EAP-TLS).
- **TIA / ISO:** ANSI/TIA-568.2-D (Balanced Twisted-Pair), ANSI/TIA-569-E (Pathways/Spaces), ANSI/TIA-606-C (Labelling), ANSI/TIA-607-D (Grounding/Bonding), ISO/IEC 11801 Class E.
- **Life Safety & Security:** NFPA 72 (National Fire Alarm Code), NFPA 101 (Life Safety Code), OSDP v2 (SIA Encrypted Protocol).
- **IETF RFCs:** RFC 1918 (Private IP), RFC 3021 (/31 P2P Links), RFC 4594 (DiffServ QoS), RFC 8446 (TLS 1.3).
