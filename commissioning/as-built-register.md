# As-Built Documentation Register & Design Variation Record

## 1. Document Control & Revision History

This As-Built register documents all field deviations, engineering redline modifications, and physical adjustments incorporated during the construction, configuration, and commissioning phases of the enterprise network and ELV infrastructure.

### Document Revision Table:

| Revision | Issue Date | Author / Engineer | Approver | Scope & Summary of Changes |
|:---:|:---:|---|---|---|
| **v0.1** | Nov 01, 2026 | A. Shibin (Lead Architect) | Project Director | Initial Preliminary Engineering Design & Scope Definition |
| **v0.5** | Nov 08, 2026 | Network / ELV Engineering Team | Technical Review Board | Intermediate Detailed Design (VLAN plan, topologies, configs) |
| **v0.9** | Nov 14, 2026 | Commissioning Lead | Operations Manager | Redline markup post pre-commissioning testing and snag remediation |
| **v1.0** | Nov 16, 2026 | A. Shibin (Lead Architect) | Client Representative | **Final Approved As-Built Documentation Package** |

---

## 2. Redline Design Variations (As-Built vs. Initial Design)

During field execution, five (5) formal Engineering Change Requests (ECRs) were authorized, implemented, and verified:

| Change ID | Subsystem / Location | Initial Design Assumption | As-Built Final Implementation | Engineering Justification & Impact |
|:---:|---|---|---|---|
| **ECR-01** | Power / MDF Server Room | Single internal battery string in APC 10 kVA UPS providing 18 minutes runtime. | Added 1 $\times$ External Battery Module (SRT192RMBP2), extending runtime to **47.4 minutes**. | Client safety audit required sufficient runtime to bridge facility generator transfer failures without abrupt server shutdown. |
| **ECR-02** | Structured Cabling / Floor 2 | 60 standard workstation dual-drops planned on Floor 2. | Added 4 $\times$ supplemental Cat6 drops in Floor 2 East R&D Hardware Lab (`2-OUT-09`). | Accommodates dedicated embedded hardware testing rigs and lab telemetry analyzers. |
| **ECR-03** | Riser Fiber / Uplinks | 10GBASE-LR Single-Mode optics initially considered for core-to-floor risers. | Standardized exclusively on **OM4 Multimode Fiber with 10GBASE-SR SFP+ optics**. | Cost-benefit analysis confirmed 65m longest riser run operates at $< 1.1\text{ dB}$ loss, saving over 70% in optical transceiver procurement cost. |
| **ECR-04** | Access Control / Elevator Lobby | Magnetic locks on Floor 1 and Floor 2 elevator lobby doors specified as standard 600 lb holding force. | Upgraded to **1,200 lb heavy-duty shear maglocks** with integrated Hall-effect bond sensors. | High-traffic lobby doors experienced occasional micro-vibrations; 1,200 lb locks eliminate accidental breach alarms. |
| **ECR-05** | Network Security / Core ACLs | Management VLAN (VLAN 10) open to general corporate network via standard username/password. | Enforced **Zero-Trust Jump Host restriction** (`ACL_USERS_SVI_IN` permits only IT Admin Jump Host `10.10.31.50`). | Penetration test recommendation: eliminates lateral network pivoting from compromised employee laptops to switch consoles. |

---

## 3. Final As-Built VLAN & Gateway Register

| VLAN ID | VLAN Name | Subnet & CIDR | Default Gateway (HSRP VIP) | Core-01 Physical IP | Core-02 Physical IP | DHCP Service Location | Primary Device Types |
|:---:|---|---|:---:|:---:|:---:|---|---|
| **10** | `MGMT_INFRA` | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.2` | `10.10.10.3` | Static / Staging Pool | Switches, Firewalls, WLC, PDUs, UPS |
| **20** | `SERVERS_PROD` | `10.10.20.0/24` | `10.10.20.1` | `10.10.20.2` | `10.10.20.3` | Static Assignments | Active Directory, DNS, DHCP, PBX, NVR |
| **30** | `DATA_USERS_GF` | `10.10.30.0/24` | `10.10.30.1` | `10.10.30.2` | `10.10.30.3` | Central DC (`10.10.20.10`) | Ground Floor Workstations & Reception |
| **31** | `DATA_USERS_F1` | `10.10.31.0/24` | `10.10.31.1` | `10.10.31.2` | `10.10.31.3` | Central DC (`10.10.20.10`) | Floor 1 Workstations & Meeting Rooms |
| **32** | `DATA_USERS_F2` | `10.10.32.0/24` | `10.10.32.1` | `10.10.32.2` | `10.10.32.3` | Central DC (`10.10.20.10`) | Floor 2 Workstations & R&D Lab |
| **33** | `DATA_USERS_F3` | `10.10.33.0/24` | `10.10.33.1` | `10.10.33.2` | `10.10.33.3` | Central DC (`10.10.20.10`) | Floor 3 Workstations & Executive Suites |
| **40** | `VOICE_VOIP` | `10.10.40.0/23` | `10.10.40.1` | `10.10.40.2` | `10.10.40.3` | Central DC (`10.10.20.10`) | 208 IP Desk Phones & Conference Units |
| **50** | `WIFI_CORP` | `10.10.50.0/23` | `10.10.50.1` | `10.10.50.2` | `10.10.50.3` | Central DC (`10.10.20.10`) | Corporate Laptops, Mobiles, Tablets |
| **60** | `WIFI_GUEST` | `10.10.60.0/24` | `10.10.60.1` | *N/A (L2 on Core)*| *N/A (L2 on Core)*| NGFW Gateway (`10.10.60.1`) | Visitor Laptops & Mobiles (Captive Portal) |
| **70** | `ELV_CCTV` | `10.10.70.0/24` | `10.10.70.1` | `10.10.70.2` | `10.10.70.3` | Static Reservations | 32 IP Security Cameras & NVR Ingest NIC |
| **80** | `ELV_ACCESS` | `10.10.80.0/24` | `10.10.80.1` | `10.10.80.2` | `10.10.80.3` | Static Reservations | 16 Door Controllers & ACS Server |
| **90** | `PRINTERS` | `10.10.90.0/24` | `10.10.90.1` | `10.10.90.2` | `10.10.90.3` | Central DC (`10.10.20.10`) | 8 Network Multi-Function Printers |
| **100**| `TRANSIT_CORE_FW`| `10.10.100.0/29`| `10.10.100.1` | `10.10.100.2` | `10.10.100.3` | Static Routed | Core Switch to NGFW Cluster Interconnect |
| **999**| `NATIVE_BLACKHOLE`| *Unrouted* | *None* | *None* | *None* | *None* | Untagged Native Trunk & Unused Ports |

---

## 4. Final As-Built Switchport Allocation Architecture

Each 48-port floor access switch stack operates according to the standardized port allocation schema below:

```
┌────────────────────────────────────────────────────────────────────────┐
│ AS-BUILT 48-PORT ACCESS SWITCH ALLOCATION MAP                         │
├─────────┬──────────────────────────────────────────────────────────────┤
│ 01 – 24 │ User Workstations + IP Phones (Data VLAN 30-33 + Voice 40)   │
│ 25 – 32 │ IP CCTV Surveillance Cameras (VLAN 70 - Sticky MAC)         │
│ 33 – 36 │ Physical Access Control Door Controllers (VLAN 80)           │
│ 37 – 38 │ Network Multi-Function Printers (VLAN 90)                   │
│ 39 – 42 │ Wi-Fi 6 Access Points (Trunk: Native 10, Tagged 50 & 60)     │
│ 43 – 48 │ Unused / Spare Ports (VLAN 999 Blackhole - Admin Shutdown)   │
│ 49 – 50 │ Dual 10G SFP+ Uplinks (Port-Channel to Core-SW01 & Core-SW02)│
└─────────┴──────────────────────────────────────────────────────────────┘
```

---

## 5. Formal As-Built Engineering Sign-Off

The signatures below confirm that this As-Built Register represents an accurate, true, and complete physical and logical record of the installed, tested, and commissioned network infrastructure.

- **Lead Network Architect & Engineer:** __________________________  **Signature:** __________________ **Date:** ____________
- **Project Operations & Handover Manager:** __________________________  **Signature:** __________________ **Date:** ____________
- **Client Technical Authority:** __________________________  **Signature:** __________________ **Date:** ____________
