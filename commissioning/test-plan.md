# Comprehensive Network & ELV Commissioning Test Plan

## 1. Overview & Test Methodology

This document defines the formal commissioning test cases to validate the end-to-end functionality, high availability, security controls, and performance of the enterprise network and ELV systems prior to operational handover.

Tests are structured systematically across OSI layers and converged subsystems:
- **Layer 1:** Physical Link & Fiber Integrity
- **Layer 2:** VLAN Segmentation, 802.1Q Trunks, Rapid-PVST, and LACP EtherChannels
- **Layer 3:** Gateway Redundancy (HSRPv2), SVI Routing, OSPF, and Upstream Transit
- **Layer 4–7 Security:** Stateful Firewall Inspection, Dynamic PAT, Inter-VLAN ACLs, and L2 Hardening
- **Subsystem Integration:** Power over Ethernet (PoE), Wi-Fi 6, CCTV, Access Control, and VoIP

---

## 2. Commissioning Test Matrix

| Test ID | Subsystem / Layer | Test Case Description | Test Procedure & Execution Commands | Expected Result & Acceptance Criteria | Status (Pass/Fail) | Tester & Date |
|:---:|---|---|---|---|:---:|:---:|
| **TC-L1-01** | Layer 1 Physical | 10G OM4 Multimode Fiber Link State | Check SFP+ interface transceiver optics on Core switches: `show interfaces GigabitEthernet1/0/1 transceiver detail`. | RX optical power between **$-2.0\text{ dBm}$ and $-8.0\text{ dBm}$**; link operational at full 10 Gbps duplex with zero CRC/frame errors. | [  ] Pass | |
| **TC-L1-02** | Layer 1 Physical | WAN Fiber Demarcation Signal | Check SFP 1000BASE-LX optical levels on NGFW-01 `eth1/1`: `get hardware nic eth1/1`. | Optical RX signal $\ge -15.0\text{ dBm}$; link state is UP; carrier negotiates 1000BASE-LX full duplex. | [  ] Pass | |
| **TC-L2-01** | Layer 2 Switching | LACP Port-Channel Aggregation | Execute `show etherchannel summary` on Core switches and floor access switches. | `Po1` through `Po4` and `Po10` display flag **`SU`** (Layer 2, In-Use); all physical member links bundled (**`P`**). | [  ] Pass | |
| **TC-L2-02** | Layer 2 Switching | 802.1Q VLAN Trunking & Pruning | Execute `show interfaces trunk` on access switches. | Native VLAN is **999**; only designated floor VLANs are forwarded; inactive VLANs are pruned. | [  ] Pass | |
| **TC-L2-03** | Layer 2 Switching | Rapid-PVST Root Bridge Election | Execute `show spanning-tree root` on all switches. | `Core-SW-01` is **Root Bridge** for all VLANs (Priority 4096); `Core-SW-02` is **Secondary** (Priority 8192); floor switches are designated roots towards core. | [  ] Pass | |
| **TC-L2-04** | Layer 2 Switching | Spanning Tree BPDU Guard Trigger | Connect an unauthorized standalone switch running STP to user port `Gi1/0/5` on `Acc-SW-F1`. | Port detects incoming BPDU within 1 second; transitions immediately to **`err-disabled`**; generates Syslog alert. | [  ] Pass | |
| **TC-L3-01** | Layer 3 Routing | HSRPv2 Virtual Gateway States | Execute `show standby brief` on `Core-SW-01` and `Core-SW-02`. | `Core-SW-01` is **Active** for all SVIs (`10.10.X.1`, Pri 110); `Core-SW-02` is in **Standby** (Pri 100). | [  ] Pass | |
| **TC-L3-02** | Layer 3 Routing | HSRP Active Gateway Failover | Disconnect uplink Port-Channel 10 on `Core-SW-01` or issue `shutdown` on SVI `Vlan31`. | `Core-SW-02` assumes **Active** gateway state within **$< 3\text{ seconds}$**; workstation continuous ping to `10.10.31.1` drops $\le 1$ packet. | [  ] Pass | |
| **TC-L3-03** | Layer 3 Routing | HSRP Preemption Recovery | Re-enable interface on `Core-SW-01` (`no shutdown`). | `Core-SW-01` preempts standby node and gracefully resumes **Active** state; zero dropped packets on client sessions. | [  ] Pass | |
| **TC-L3-04** | Layer 3 Routing | OSPF Dynamic Route Peering | Execute `show ip ospf neighbor` on Core switches. | Adjacency with Firewall inside interface (`10.10.100.4`) reaches **`FULL/BDR`** state; default route `0.0.0.0/0` present in routing table. | [  ] Pass | |
| **TC-SEC-01**| Network Security | Outbound Dynamic PAT (Internet) | Initiate HTTPS web browsing session from Floor 1 workstation (`10.10.31.105`) to external web destination (`https://www.google.com`). | Session traverses firewall successfully; NAT table translates source IP to public WAN IP `198.51.100.2`; web page renders. | [  ] Pass | |
| **TC-SEC-02**| Network Security | Guest Wireless Air-Gap Isolation | Connect client to `HQ-Guest` Wi-Fi (IP `10.10.60.25`); attempt ping and HTTP connection to internal server `10.10.20.10` and core VIP `10.10.30.1`. | Packets dropped immediately by Firewall Rule 60; zero packets reach internal network; client only accesses external Internet. | [  ] Pass | |
| **TC-SEC-03**| Network Security | CCTV SVI Access Control List | Attempt to ping CCTV camera (`10.10.70.11`) from standard user workstation (`10.10.31.105`). | Connection blocked by `ACL_USERS_SVI_IN` on Core switch; NVR server (`10.10.20.16`) connects to RTSP stream on camera successfully. | [  ] Pass | |
| **TC-SEC-04**| Network Security | Cisco Sticky MAC Anti-Tampering | Disconnect authorized IP camera from `Gi1/0/25` on `Acc-SW-GF`; plug in unauthorized technician laptop. | Switchport detects foreign MAC address; transitions instantly to **`err-disable`** state; cuts link power; generates SNMP security trap. | [  ] Pass | |
| **TC-SEC-05**| Network Security | DHCP Snooping & Rogue DHCP Blocker | Connect a rogue home wireless router running a rogue DHCP server to user wall jack on Floor 2. | Switch drops untrusted DHCP Offer packets on access port; logs security violation; legitimate clients continue receiving IPs from `10.10.20.10`. | [  ] Pass | |
| **TC-POE-01**| PoE Subsystem | Access Switch Full Load Delivery | Power on all connected IP phones (60), APs (3), cameras (8), and access controllers (4) on `Acc-SW-F1`. | Execute `show power inline`: total power draw is $\approx 498\text{ W}$; switch power budget maintains **$66.4\%$ spare headroom**; zero power faults. | [  ] Pass | |
| **TC-WIFI-01**| Wireless LAN | WPA3-Enterprise 802.1X Auth | Connect corporate laptop to SSID `Corp-Secure` using Active Directory credentials (`corp\user01`). | RADIUS EAP-TLS / PEAP-MSCHAPv2 handshake completes within 1.2s; client receives IP in `10.10.50.0/23`; corporate intranet accessible. | [  ] Pass | |
| **TC-WIFI-02**| Wireless LAN | Fast BSS Transition Roaming (802.11r)| Run continuous voice call on softphone while walking from Floor 1 West Wing (`WAP-F1-01`) to East Wing (`WAP-F1-03`). | Roaming handover occurs in **$< 30\text{ ms}$**; zero audio stutter or voice dropout; client maintains same IP address without re-authenticating. | [  ] Pass | |
| **TC-WIFI-03**| Wireless LAN | RF Signal Coverage Heatmap Audit | Measure 5 GHz RSSI signal across all office workstations and meeting rooms using Ekahau / NetSpot survey tool. | Signal strength $\ge \mathbf{-65\text{ dBm}}$ across 100% of floor area; Signal-to-Noise Ratio (SNR) $\ge 25\text{ dB}$. | [  ] Pass | |
| **TC-CCTV-01**| IP CCTV Subsystem| Continuous 32-Channel Ingest | Monitor NVR VMS dashboard for 60 consecutive minutes during active business hours. | All 32 cameras stream continuously at 1080p @ 25 fps; aggregate ingest bandwidth stable at $\approx 128\text{ Mbps}$; zero frame drops or network timeouts. | [  ] Pass | |
| **TC-CCTV-02**| IP CCTV Subsystem| NVR RAID 6 Disk Fault Tolerance | Pull out physical Hard Disk Drive #3 from the NVR chassis during active multi-channel recording. | NVR alarm sounds; RAID controller switches to degraded mode; hot-spare disk activates automatically; **zero video loss or recording interruption**. | [  ] Pass | |
| **TC-ACS-01** | Access Control | Smart Card Badge Authentication | Present valid MIFARE DESFire EV3 access badge to card reader at Ground Floor Reception Speed Gate (`D-GF-01`). | Controller reads encrypted OSDP credential; validates authorization against local DB in $< 250\text{ ms}$; unlocks gate; logs green entry event in server. | [  ] Pass | |
| **TC-ACS-02** | Access Control | Life Safety Fire Alarm Override Cut| Activate test key on building Fire Alarm Control Panel (FACP) manual pull station. | Master emergency relay de-energizes instantly; cuts DC power to all fail-safe magnetic locks; **all egress doors unlock instantaneously**. | [  ] Pass | |
| **TC-VOIP-01**| VoIP Subsystem | Zero-Touch Option 150 Provisioning | Unbox a factory-fresh Cisco 7841 IP phone; connect to user port on Floor 3 (`Acc-SW-F3 Gi1/0/10`). | Phone discovers Voice VLAN 40 via CDP/LLDP-MED; receives IP via DHCP; downloads config via TFTP from `10.10.20.15`; registers extension `4010` in $< 90\text{s}$. | [  ] Pass | |
| **TC-VOIP-02**| VoIP Subsystem | Voice Audio Quality (MOS Score) | Place test calls between Ground Floor Reception and Floor 3 Executive Boardroom while transmitting a large 10GB file transfer across network. | QoS Strict Priority Queue (PQ) protects voice RTP packets; measured jitter $< 10\text{ ms}$; packet loss $= 0.00\%$; **Mean Opinion Score (MOS) $= 4.38$**. | [  ] Pass | |
| **TC-VOIP-03**| VoIP Subsystem | Location-Based E911 Dispatch | Dial emergency test number from Floor 2 office desk phone (`Ext 3010`). | PBX transmits ELIN associated with Floor 2 West; emergency responder console displays exact building address, 2nd floor, and desk zone. | [  ] Pass | |

---

## 3. Formal Commissioning Sign-Off & System Acceptance

By signing below, the commissioning engineering team confirms that all test cases specified above have been thoroughly executed, verified, and found compliant with project technical specifications.

- **Senior Network Infrastructure Engineer:** __________________________  **Signature:** __________________ **Date:** ____________
- **Senior ELV Systems Commissioning Lead:** __________________________  **Signature:** __________________ **Date:** ____________
- **Client Project Director / Technical Representative:** __________________________  **Signature:** __________________ **Date:** ____________

