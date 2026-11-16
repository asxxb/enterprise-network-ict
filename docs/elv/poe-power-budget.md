# Power over Ethernet (PoE) & Uninterruptible Power Supply (UPS) Engineering Calculations

## 1. Executive Summary & Design Scope

This document establishes the electrical Power over Ethernet (PoE) capacity planning for all network access switches and the Uninterruptible Power Supply (UPS) battery backup sizing for the central Main Distribution Frame (MDF) and floor Intermediate Distribution Frames (IDFs).

The design ensures:
- 100% centralized power delivery for 260 connected PoE devices (IP phones, Wi-Fi 6 APs, CCTV cameras, and access control units).
- Minimum **30% power supply headroom** on every access switch stack to prevent thermal throttling or port shutdowns under peak surge.
- Minimum **30 minutes of autonomous battery runtime** in floor IDFs and **40 minutes in the central MDF** during complete utility grid blackout.

---

## 2. PoE Classification & Endpoint Power Profiles

Power is distributed via IEEE 802.3 standards over structured Category 6 4-pair balanced copper cabling:

| Endpoint Device Category | IEEE Standard | PoE Class | Max Switchport Allocation ($P_{\text{Port}}$) | Typical Device Consumption ($P_{\text{Typ}}$) | Worst-Case Peak Consumption ($P_{\text{Peak}}$) | Cable Resistance Loss Factor |
|---|---|:---:|:---:|:---:|:---:|:---:|
| **IP Desk Phones (Cisco 7841/8845)** | **802.3af (PoE)** | Class 2 / 3 | **15.4 W** | **4.2 W – 5.5 W** | **6.5 W** | ~10% (0.6W loss @ 90m) |
| **IP Conference Units (Cisco 8832)** | **802.3af (PoE)** | Class 3 | **15.4 W** | **7.5 W – 9.0 W** | **11.5 W** | ~10% (1.0W loss @ 70m) |
| **Indoor Fixed Dome Cameras** | **802.3af (PoE)** | Class 3 | **15.4 W** | **5.0 W (Day) / 8.5 W (IR On)** | **10.5 W (IR Peak)** | ~10% (0.9W loss @ 90m) |
| **Outdoor Varifocal Bullet Cameras** | **802.3at (PoE+)**| Class 4 | **30.0 W** | **9.0 W (Day) / 14.0 W (IR)** | **18.0 W (Heater/IR)** | ~12% (1.8W loss @ 80m) |
| **Access Control Door Controllers** | **802.3af (PoE)** | Class 3 | **15.4 W** | **6.0 W (Idle) / 10.0 W (Active)**| **12.5 W (Reader+Lock)**| ~10% (1.1W loss @ 90m) |
| **Wi-Fi 6 Access Points (C9120AXI)** | **802.3at (PoE+)**| Class 4 | **30.0 W** | **16.5 W (Avg) / 19.8 W (Peak)** | **24.5 W (Max 4x4)** | ~15% (3.2W loss @ 80m) |

---

## 3. Floor-by-Floor Access Switch PoE Power Sizing

### 3.1 Hardware Power Supply Architecture
- **Switch Model:** Cisco Catalyst 9300-48P (48 $\times$ 10/100/1000 PoE+ ports).
- **Power Supply Configuration:** Dual redundant, hot-swappable **715W AC Power Supplies (PWR-C1-715WAC-P)** per switch.
- **Dedicated PoE Output:** **740 Watts** per 48-port chassis (the system reserves 690W for PoE output with the remainder dedicated to CPU, ASIC logic, and fans).
- **Stacked Resilience:** Multi-switch stacks (Floors 1 and 2) employ **Cisco StackPower**, sharing DC current across a common power bus. If an individual power supply fails, zero PoE ports are dropped.

### 3.2 Detailed Power Budget Breakdown

```
┌────────────────────────────────────────────────────────────────────────┐
│ POE CAPACITY VS DEMAND SUMMARY                                          │
│                                                                        │
│ Ground Floor (Acc-SW-GF):   [█████░░░░░] 388W / 740W  (47.6% Spare)     │
│ Floor 1 (Acc-SW-F1 Stack):  [████░░░░░░] 498W / 1480W (66.4% Spare)    │
│ Floor 2 (Acc-SW-F2 Stack):  [████░░░░░░] 498W / 1480W (66.4% Spare)    │
│ Floor 3 (Acc-SW-F3):        [█████░░░░░] 388W / 740W  (47.6% Spare)     │
│                                                                        │
│ CAMPUS TOTAL:               [████░░░░░░] 1,772W / 4440W (60.1% Buffer) │
└────────────────────────────────────────────────────────────────────────┘
```

#### Detailed Calculations Table:

| Switch Stack ID | Floor Location | Chassis Stack Qty | Total Switchports | Total Available PoE Budget | Connected PoE Devices | Typical Real-Time Draw | Max Worst-Case Draw | Remaining PoE Headroom | Utilization % | Safety Status |
|---|:---:|:---:|:---:|:---:|---|:---:|:---:|:---:|:---:|:---:|
| **`Acc-SW-GF`** | Ground Floor (MDF) | 1 $\times$ 48P | 48 | **740 W** | 40 Phones, 3 APs, 8 Cameras, 4 Door Ctrls | **388.0 W** | **456.0 W** | **284.0 W** | **52.4%** | ✅ Exceeds 30% Headroom |
| **`Acc-SW-F1`** | Floor 1 (IDF-1) | 2 $\times$ 48P Stack | 96 | **1,480 W** | 60 Phones, 3 APs, 8 Cameras, 4 Door Ctrls | **498.0 W** | **586.0 W** | **894.0 W** | **33.6%** | ✅ Exceeds 60% Headroom |
| **`Acc-SW-F2`** | Floor 2 (IDF-2) | 2 $\times$ 48P Stack | 96 | **1,480 W** | 60 Phones, 3 APs, 8 Cameras, 4 Door Ctrls | **498.0 W** | **586.0 W** | **894.0 W** | **33.6%** | ✅ Exceeds 60% Headroom |
| **`Acc-SW-F3`** | Floor 3 (IDF-3) | 1 $\times$ 48P | 48 | **740 W** | 40 Phones, 3 APs, 8 Cameras, 4 Door Ctrls | **388.0 W** | **456.0 W** | **284.0 W** | **52.4%** | ✅ Exceeds 30% Headroom |
| **CAMPUS TOTAL**| **All Floors** | **6 Chassis** | **288** | **4,440 W** | **260 Connected PoE Devices** | **1,772.0 W** | **2,084.0 W** | **2,356.0 W** | **39.9%** | ✅ **60.1% Total Spare** |

---

## 4. Uninterruptible Power Supply (UPS) Sizing & Runtime Calculations

To ensure business continuity during utility power anomalies, brownouts, and blackouts, centralized on-line double-conversion UPS systems are deployed in the MDF and each floor IDF.

### 4.1 Electrical Formula for Battery Autonomy Runtime

$$\text{Battery Energy Available (Watt-Hours)} = V_{\text{string}} \times C_{\text{Ah}} \times \eta_{\text{inv}} \times D_{\text{od}}$$

$$\text{Estimated Runtime (Minutes)} = \frac{V_{\text{string}} \times C_{\text{Ah}} \times \eta_{\text{inv}} \times D_{\text{od}}}{P_{\text{load}}(\text{Watts})} \times 60$$

Where:
- $V_{\text{string}}$: DC Battery bus voltage.
- $C_{\text{Ah}}$: Total battery string ampere-hour capacity.
- $\eta_{\text{inv}}$: Inverter operating efficiency ($\approx 92\% = 0.92$).
- $D_{\text{od}}$: Depth of discharge limit ($\approx 85\% = 0.85$).
- $P_{\text{load}}$: Continuous connected electrical load in Watts.

---

### 4.2 MDF Server Room UPS: 10 kVA / 9 kW Online System
- **Selected System:** APC Smart-UPS SRT 10000VA / 9000W On-Line Double Conversion ($3\text{U}$ electronics $+ 3\text{U}$ battery chassis) with one External Battery Pack (SRT192RMBP2).
- **Connected Equipment in MDF:**
  - 2 $\times$ Cisco Catalyst 9500 Core Switches ($2 \times 350\text{ W} = 700\text{ W}$)
  - 1 $\times$ Cisco Catalyst 9300-48P Ground Access Stack + PoE ($550\text{ W}$)
  - 2 $\times$ FortiGate / Palo Alto NGFW Firewalls ($2 \times 120\text{ W} = 240\text{ W}$)
  - 2 $\times$ Cisco Catalyst 9800-L Wireless LAN Controllers ($2 \times 85\text{ W} = 170\text{ W}$)
  - 4 $\times$ Production Hypervisor & AD Servers ($4 \times 300\text{ W} = 1,200\text{ W}$)
  - 1 $\times$ CCTV NVR Storage Array with 10 HDDs ($380\text{ W}$)
  - 1 $\times$ SAN Storage Array ($450\text{ W}$)
  - Rack airflow fans & environmental sensors ($110\text{ W}$)
  - **Total Continuous Load ($P_{\text{MDF}}$):** **3,800 Watts** (Operating at $42.2\%$ of 9 kW rating).
- **Runtime Sizing:**
  - At $3.8\text{ kW}$ continuous load, standard battery string ($192\text{V DC}, 10\text{ Ah}$) provides **18 minutes**.
  - With the addition of **one External Battery Pack** (total $20\text{ Ah}$ string):
    $$\text{Runtime} \approx \frac{192\text{ V} \times 20\text{ Ah} \times 0.92 \times 0.85}{3,800\text{ W}} \times 60 \approx \mathbf{47.4\text{ Minutes}}$$
  - **Conclusion:** Delivers **47 minutes of full autonomous power**, easily bridging the 30-second start window of the facility emergency diesel generator and allowing graceful virtual machine shutdown if fuel exhausts.

---

### 4.3 Floor IDF Telecommunications Rooms: 3 kVA / 2.7 kW Online Systems
- **Deployment:** One unit installed in IDF-1 (Floor 1), IDF-2 (Floor 2), and IDF-3 (Floor 3).
- **Selected System:** APC Smart-UPS SRT 3000VA / 2700W 230V 2U Rackmount ($96\text{V DC}$ battery bus, $9\text{ Ah}$).
- **Connected Load per IDF:**
  - IDF-1 (Floor 1): 2-switch stack + 75 PoE endpoints = **850 Watts** (31.5% capacity).
  - IDF-2 (Floor 2): 2-switch stack + 75 PoE endpoints = **850 Watts** (31.5% capacity).
  - IDF-3 (Floor 3): 1-switch chassis + 55 PoE endpoints = **620 Watts** (23.0% capacity).
- **Runtime Calculation (Worst-Case 850W Load on Floor 1 & 2):**
  $$\text{Runtime} = \frac{96\text{ V} \times 9\text{ Ah} \times 0.92 \times 0.85}{850\text{ W}} \times 60 \approx \mathbf{47.7\text{ Minutes}}$$
- **Conclusion:** Exceeds the 30-minute target by over 50%, guaranteeing uninterrupted VoIP telephony, emergency access control egress, and CCTV recording during localized power breaker trips.

