# Commissioning Snag Report & Defect Rectification Log

## 1. Overview & Snagging Process

During the phased testing and site commissioning of the enterprise network, structured cabling, and ELV subsystems, field discrepancies, installation defects, and configuration anomalies were logged into this formal Snag Register.

### Severity Classification Matrix:
- 🔴 **Critical (Cat 1):** Complete system outage, life-safety hazard, or security vulnerability. Must be resolved within 4 hours prior to live operation.
- 🟡 **Major (Cat 2):** Feature degradation, redundancy failure, or out-of-specification performance. Must be resolved within 24 hours.
- 🟢 **Minor (Cat 3):** Cosmetic defects, labelling errors, or documentation discrepancies. Resolved prior to final commercial handover.

---

## 2. Master Snag & Defect Log

| Snag ID | Date Logged | Subsystem / Location | Defect Description & Observed Symptom | Severity | Root Cause Analysis | Remediation Action Taken | Owner / Assignee | Status | Closure Date | Verified By |
|:---:|:---:|---|---|:---:|---|---|:---:|:---:|:---:|:---:|
| **SNG-01** | Nov 12, 2026 | Access Control / Door D-GF-03 | Magnetic lock on Ground Floor elevator lobby door failed to unlock during Fire Alarm test. | 🔴 **Critical** | Fire Alarm relay dry contact was incorrectly wired to the Normally Open (NO) terminal on the power supply board instead of Normally Closed (NC). | Re-terminated fire alarm trigger leads to the auxiliary NC emergency shutdown bus on Altronix PSU. Retested pull station: lock drops power instantly. | ELV Lead (R. Sharma) | **Closed** | Nov 12, 2026 | Fire Inspector |
| **SNG-02** | Nov 13, 2026 | Switching / `Core-SW-02` | HSRP SVI `Vlan40` on `Core-SW-02` remained in Active state simultaneously with `Core-SW-01` (Dual-Active Split-Brain). | 🔴 **Critical** | Port-Channel 10 trunk between Core-01 and Core-02 was missing `VLAN 40` in the `switchport trunk allowed vlan` command, blocking HSRP hellos. | Added `40` to allowed trunk VLAN list on Port-Channel 10: `switchport trunk allowed vlan add 40`. HSRP state immediately converged to Active / Standby. | Network Lead (A. Shibin) | **Closed** | Nov 13, 2026 | QA Lead |
| **SNG-03** | Nov 13, 2026 | Wireless / `WAP-F3-02` | Wi-Fi 6 AP in Floor 3 Executive Suite reported reduced 2x2 radio operational mode and disabled USB port. | 🟡 **Major** | Access switchport `Gi1/0/40` was negotiating legacy 802.3af (15.4W) PoE instead of 802.3at (30W) PoE+. | Enabled CDP/LLDP power negotiation and explicitly configured `power inline port 2-event` on switchport. AP drew 20.0W and resumed full 4x4:4 MU-MIMO operation. | Network Lead (A. Shibin) | **Closed** | Nov 14, 2026 | Wireless Eng |
| **SNG-04** | Nov 14, 2026 | VoIP / Floor 1 IP Phones | New Cisco 7841 IP phones on Floor 1 failed to download configuration and displayed "Configuring IP... TFTP Timeout". | 🟡 **Major** | DHCP Scope `SCOPE_VOICE` on the secondary Domain Controller (`10.10.20.11`) was missing DHCP Option 150 (TFTP Server IP). | Added Option 150 pointing to PBX IP `10.10.20.15` in Windows DHCP scope configuration. Re-booted phones; phones registered within 45 seconds. | Systems Admin (M. Vance) | **Closed** | Nov 14, 2026 | VoIP Tech |
| **SNG-05** | Nov 14, 2026 | CCTV / Camera `F2-CAM-05` | Video stream on Floor 2 East Corridor experienced intermittent pixelation, frame drops, and RTSP socket timeouts. | 🟡 **Major** | Fluke test showed high Return Loss on drop `2.IDF2-PP03-018` caused by an excessively tight cable zip-tie behind the corridor ceiling tray. | Cut zip-tie, re-dressed cable bundle using 20mm Velcro strap, and re-terminated keystone jack. Fluke DSX-8000 re-test passed with +4.2 dB Return Loss headroom. | Cabling Lead (D. Kalu) | **Closed** | Nov 15, 2026 | Cabling Inspector |
| **SNG-06** | Nov 15, 2026 | Security / Visitor Guest Wi-Fi | Guest Wi-Fi clients on `HQ-Guest` were unable to resolve external hostnames prior to captive portal acceptance. | 🟡 **Major** | Firewall ACL on `ZONE_GUEST` interface was blocking DNS (UDP 53) outbound requests to public DNS servers (`1.1.1.1` and `8.8.8.8`). | Modified Firewall Rule 50 to permit outbound DNS to `1.1.1.1` and `8.8.8.8` pre-authentication. Captive portal redirection script resolved instantly. | Security Eng (K. Tariq) | **Closed** | Nov 15, 2026 | QA Lead |
| **SNG-07** | Nov 15, 2026 | Structured Cabling / Floor 1 IDF | Patch panel labelling on `IDF1-PP02` ports 25–48 was offset by two positions relative to the architectural outlet drawings. | 🟢 **Minor** | Field technician misaligned label sheet during installation of second patch panel tier. | Re-printed standardized TIA-606-C vinyl labels using Brady thermal printer; verified port-to-port continuity; updated As-Built register. | Cabling Tech (J. Cole) | **Closed** | Nov 16, 2026 | Site Foreman |
| **SNG-08** | Nov 16, 2026 | Management / Access Switches | In-band SSH sessions to `Acc-SW-F2` and `Acc-SW-F3` experienced high keyboard latency during peak afternoon file transfers. | 🟢 **Minor** | Management SVI `Vlan10` ICMP and SSH packets were competing in the default Best Effort egress queue without priority tagging. | Applied CoS 6 / DSCP CS6 classification policy to network management traffic across all trunk interfaces. SSH latency dropped from 140ms to $< 2\text{ ms}$. | Network Lead (A. Shibin) | **Closed** | Nov 16, 2026 | Network Lead |

---

## 3. Snag Clearance Certification

All eight (8) recorded snags and defects have been thoroughly rectified, re-tested under operational load, and formally verified by the engineering leads. **Zero open defects remain on the system.**

- **Lead Network Commissioning Engineer:** __________________________  **Signature:** __________________ **Date:** ____________
- **Lead ELV Commissioning Specialist:** __________________________  **Signature:** __________________ **Date:** ____________
- **Client Project Technical Director:** __________________________  **Signature:** __________________ **Date:** ____________
