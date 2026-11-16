# Physical Access Control System (PACS) Architecture

## 1. Overview & System Objectives

This document establishes the technical engineering design for the enterprise Physical Access Control System (PACS) deployed across the 4-storey headquarters building.

The system secures physical entry points, restricts access to sensitive facilities (Server/MDF Room, Executive Suites), tracks ingress/egress events for security compliance, and integrates directly with the building's Life Safety Fire Alarm System to ensure instant emergency egress.

### Architectural Parameters:
- **Total Controlled Portals:** **16 Doors / Portals** distributed across Ground, Floor 1, Floor 2, and Floor 3.
- **Hardware Architecture:** Distributed IP-based intelligent sub-controllers communicating over an isolated network segment (**VLAN 80: `10.10.80.0/24`**) to a central Access Control Server (**`10.10.20.17`**).
- **Credential Technology:** Contactless Smart Card (MIFARE DESFire EV3, 13.56 MHz, AES-128 encrypted) and Mobile BLE (Bluetooth Low Energy / NFC) smartphone digital passes.
- **Reader-to-Controller Protocol:** **OSDP v2 (Open Supervised Device Protocol)** with Secure Channel Protocol (SCP AES-128 encryption) over RS-485, completely replacing vulnerable legacy unencrypted Wiegand signaling.

---

## 2. Subsystem Components & Hardware Architecture

```mermaid
flowchart TD
    subgraph SERVER_ROOM ["MDF: Ground Floor Server Room"]
        ACS_SRV["Access Control Head-End Server\n(IP: 10.10.20.17 - VLAN 20)\nDatabase, Cardholder Management, Audit Logs"]
        CORE_SW["Collapsed Core Switch\n(SVI Vlan80 Gateway: 10.10.80.1)"]
    end

    subgraph ENCLOSURE ["Typical Floor ELV Enclosure (Wall-Mount Cabinet)"]
        direction TB
        FACP_RELAY["Fire Alarm System (FACP)\nEmergency Egress Relay (Dry Contact)"]
        PSU["12V / 24V DC Linear Power Supply\nwith 4-Hour Battery Backup (Dual 12V 18Ah SLA)"]
        CTRL["Intelligent IP Door Controller\n(Mercury / HID / Gallagher)\n[VLAN 80 - Cat6 RJ-45 PoE / IP]"]

        FACP_RELAY ==>|"Cuts Power on Fire Alarm"| PSU
        PSU -->|"Clean DC Lock Power"| CTRL
    end

    subgraph DOOR_HARDWARE ["Secured Door Portal Peripherals"]
        READER["Multi-Tech RFID / BLE Smart Reader\n(Encrypted OSDP v2 RS-485)"]
        LOCK["Electric Lock Hardware\n(Fail-Safe Maglock or Fail-Secure Strike)"]
        REX["Request-to-Exit (REX) Sensor\n(Passive Infrared - PIR)"]
        DPS["Door Position Switch (DPS)\n(Magnetic Reed Contact)"]
        BGU["Emergency Break-Glass Unit (BGU)\n(Green Manual Emergency Override)"]

        CTRL --- READER
        CTRL --- LOCK
        CTRL --- REX
        CTRL --- DPS
        BGU -.->|"Physical Hardwire Power Cut"| LOCK
    end

    ACS_SRV <==|"TLS Encrypted IP (TCP 443 / 3001)"| CORE_SW
    CORE_SW <==|"802.1Q Trunk / Cat6 Uplink"| CTRL
```

### Component Details:
1. **Intelligent IP Controllers:** 2-door and 4-door modular IP controllers mounted in locked ELV enclosures adjacent to IDF rooms. Each controller maintains a full local database of cardholder credentials and access schedules; if the network link or server fails, the controller operates autonomously in local mode.
2. **Readers:** Wall-mounted mullion smart readers supporting dual-frequency 13.56 MHz RFID and 2.4 GHz Bluetooth Low Energy (BLE) mobile credentials.
3. **Locking Devices:**
   - **Electromagnetic Locks (Maglocks):** 1,200 lbs holding force with internal Hall-effect magnetic bond sensor.
   - **Electric Mortise Strikes:** Heavy-duty ANSI grade 1 stainless steel strikes.
4. **Monitoring Sensors:**
   - **Door Position Switch (DPS):** High-security recessed magnetic reed contact monitoring whether the door is open, closed, forced open, or held open past allowed duration (Door-Held-Open alert).
   - **Request-to-Exit (REX):** Overhead passive infrared (PIR) sensor detecting approaching occupants and automatically shunting alarm monitoring to permit free exit.
5. **Emergency Break-Glass Units (BGU):** Green, double-pole, key-resettable manual emergency release boxes wired in series with maglock power leads on the interior escape side of the door.

---

## 3. Fail-Safe vs. Fail-Secure Engineering Matrix

The selection of locking hardware mode is governed by strict Life Safety Codes (NFPA 101 Life Safety Code, International Building Code - IBC, and local civil defense regulations):

| Mode | Definition & Operational Behavior | Power Status | Safety Impact | Security Impact | Assigned Door Categories |
|---|---|:---:|---|---|---|
| **FAIL-SAFE** | **Power to Lock.** When electrical power is applied, the door is locked. When power is removed or cut, the door **instantly unlocks**. | Power Loss $\to$ **UNLOCKED** | **Prioritizes Life Safety.** Guarantees immediate, unhindered evacuation during power outages, fire emergencies, or building collapse. | Reduced physical security during total blackout (mitigated by emergency battery backups). | Emergency fire exit doors, stairwell entry/exit, main building glass egress turnstiles, elevator lobby partition doors. |
| **FAIL-SECURE** | **Power to Unlock.** When electrical power is applied, the door unlocks. When power is cut, the door **remains locked**. | Power Loss $\to$ **LOCKED** | **Prioritizes Asset Protection.** Mechanical exit latch or handle on the inside *always* permits mechanical egress regardless of power. | High security maintained at all times; outsiders cannot push door open during a blackout. | Server / Comms Room (MDF), Telecom IDFs, Security Control Room, Executive Vault / Archive. |

> [!IMPORTANT]
> **Code Compliance Mandatory Rule:** All fail-safe magnetic locks on egress doors must be hardwired directly through a dedicated fire alarm emergency release relay and an emergency break-glass unit (BGU) to ensure mechanical/electrical release completely independent of software, firmware, or IP network functionality.

---

## 4. Master Door Schedule & Hardware Specification (16 Doors)

| Door ID | Floor | Location / Room Name | Lock Type | Lock Mode | Reader Tech | REX Device | Break Glass (BGU) | Access Group Restriction | Controller Host & IP |
|:---:|:---:|---|:---:|:---:|:---:|:---:|:---:|:---:|---|
| **D-GF-01** | Ground | Main Reception Glass Speed Gate (Entry) | Motorized Turnstile | Fail-Safe | RFID + BLE | Optical IR | Yes | All Employees & Badged Visitors | `ACC-CTRL-GF-01` (`10.10.80.11`) |
| **D-GF-02** | Ground | Main Reception Glass Speed Gate (Exit) | Motorized Turnstile | Fail-Safe | RFID + BLE | Optical IR | Yes | Free Egress / Badge Required | `ACC-CTRL-GF-01` (`10.10.80.11`) |
| **D-GF-03** | Ground | Ground Floor Elevator Lobby Fire Door | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR + Push | Yes | All Staff (24/7 Schedule) | `ACC-CTRL-GF-01` (`10.10.80.11`) |
| **D-GF-04** | Ground | **Server / MDF Comms Room (High Security)**| Electric Mortise Lock | **Fail-Secure**| **Biometric + RFID**| Push-to-Exit | No | **IT System Admins Only (MFA)** | `ACC-CTRL-GF-02` (`10.10.80.12`) |
| **D-F1-01** | Floor 1 | Floor 1 Elevator Lobby Main Entry | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR REX | Yes | Floor 1 Staff & Badged Guests | `ACC-CTRL-F1-01` (`10.10.80.15`) |
| **D-F1-02** | Floor 1 | Floor 1 West Open Office Suite | Electric Mortise Strike | Fail-Secure | RFID + BLE | Lever Handle | No | Floor 1 Department Staff | `ACC-CTRL-F1-01` (`10.10.80.15`) |
| **D-F1-03** | Floor 1 | Floor 1 East Open Office Suite | Electric Mortise Strike | Fail-Secure | RFID + BLE | Lever Handle | No | Floor 1 Department Staff | `ACC-CTRL-F1-02` (`10.10.80.16`) |
| **D-F1-04** | Floor 1 | Floor 1 West Fire Stairwell Egress | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR + Touch | Yes | Emergency Egress / All Staff | `ACC-CTRL-F1-02` (`10.10.80.16`) |
| **D-F2-01** | Floor 2 | Floor 2 Elevator Lobby Main Entry | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR REX | Yes | Floor 2 Staff & Badged Guests | `ACC-CTRL-F2-01` (`10.10.80.19`) |
| **D-F2-02** | Floor 2 | Floor 2 West Training & Seminar Rm | Electric Mortise Strike | Fail-Secure | RFID + BLE | Lever Handle | No | General Staff | `ACC-CTRL-F2-01` (`10.10.80.19`) |
| **D-F2-03** | Floor 2 | Floor 2 East R&D / Engineering Lab | Electric Mortise Lock | **Fail-Secure**| Biometric + RFID | Lever Handle | No | **R&D Personnel Only** | `ACC-CTRL-F2-02` (`10.10.80.20`) |
| **D-F2-04** | Floor 2 | Floor 2 East Fire Stairwell Egress | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR + Touch | Yes | Emergency Egress / All Staff | `ACC-CTRL-F2-02` (`10.10.80.20`) |
| **D-F3-01** | Floor 3 | Floor 3 Elevator Lobby Executive Suite | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR REX | Yes | Executive Suite Whitelist Only | `ACC-CTRL-F3-01` (`10.10.80.23`) |
| **D-F3-02** | Floor 3 | Executive Boardroom Portal | Electric Mortise Strike | Fail-Secure | RFID + BLE | Lever Handle | No | Board Members & Management | `ACC-CTRL-F3-01` (`10.10.80.23`) |
| **D-F3-03** | Floor 3 | Legal & Corporate Archives Room | Electric Mortise Lock | **Fail-Secure**| RFID + PIN | Lever Handle | No | Legal Department Only | `ACC-CTRL-F3-02` (`10.10.80.24`) |
| **D-F3-04** | Floor 3 | Floor 3 East Fire Stairwell Egress | Maglock (1200 lb) | **Fail-Safe** | RFID + BLE | PIR + Touch | Yes | Emergency Egress / All Staff | `ACC-CTRL-F3-02` (`10.10.80.24`) |

---

## 5. Fire Alarm System (FAS) Interface & Life Safety Integration

The integration between the Access Control System and the building's Fire Alarm Control Panel (FACP) is implemented at the **physical hardware layer**, adhering to Life Safety NFPA 72 and NFPA 101 standards:

```
┌────────────────────────────────────────────────────────────────────────┐
│ LIFE SAFETY FIRE ALARM EMERGENCY SHUTOFF CIRCUIT                       │
│                                                                        │
│  [230V Mains AC] ──> [Access Control Power Supply Unit (PSU)]          │
│                                  │                                     │
│                                  ▼ 12V / 24V DC Raw Lock Power Bus     │
│                 ┌────────────────────────────────┐                     │
│                 │ FACP Emergency Egress Relay    │                     │
│                 │ (Normally Closed Dry Contact)  │                     │
│                 └────────────────┬───────────────┘                     │
│                                  │                                     │
│      FIRE ALARM ACTIVATION ──────┼───> RELAY OPENS INSTANTLY           │
│                                  │                                     │
│                                  ▼ DC Power Disconnected               │
│                 ┌────────────────────────────────┐                     │
│                 │ Emergency Break-Glass Unit     │                     │
│                 │ (Manual Secondary Cut)         │                     │
│                 └────────────────┬───────────────┘                     │
│                                  │                                     │
│                                  ▼                                     │
│                 [Fail-Safe Electromagnetic Locks]                      │
│                 *DOORS RELEASE INSTANTLY ACROSS ALL FLOORS*            │
└────────────────────────────────────────────────────────────────────────┘
```

### Safety Features:
1. **FACP Dry Contact:** The Fire Alarm panel supplies a supervisory Normally Closed (NC) dry contact to the power distribution board of each access control power supply. Upon any smoke detector, manual pull station, or sprinkler flow switch activation, the contact opens, dropping power to all fail-safe lock circuits.
2. **True Failsafe Operation:** The release occurs at the power level; even if every access controller, switch, server, or software application crashes or locks up, the doors unlock unconditionally.
3. **Audit Logging:** Secondary auxiliary contacts inform the Access Control Server (`10.10.20.17`) of the emergency release event, triggering an automated priority security alarm on the security workstation console.

---

## 6. Power Design & Battery Backup Autonomous Capacity

Each floor's access control enclosure houses a specialized power supply unit:
- **Model:** Altronix eFlow104NX or LifeSafety Power FPO250 (12V / 24V DC, 10 Amp continuous output).
- **Battery Backup:** Dual **12V 18Ah Sealed Lead-Acid (SLA)** maintenance-free batteries wired in series.
- **Power Calculation:**
  - 4 doors per floor $\times$ (1.0A Maglock/Strike + 0.15A Reader + 0.3A Controller) = **5.8 Amperes maximum draw**.
  - Standby autonomous battery operation:
    $$\text{Backup Autonomy} = \frac{\text{Battery Capacity (18 Ah)} \times 0.85\text{ efficiency}}{5.8\text{ A load}} \approx \mathbf{2.6\text{ to }4.0\text{ Hours}}$$
  - In a catastrophic building power failure, the access control system remains 100% operational for up to 4 hours, maintaining security and logging until emergency generator restoration.

