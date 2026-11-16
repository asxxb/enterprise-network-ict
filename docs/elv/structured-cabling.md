# Structured Cabling System Architecture & Infrastructure Specification

## 1. Executive Summary & Standards Compliance

This document establishes the physical structured cabling architecture for the 4-storey enterprise headquarters building. The design governs all horizontal copper cabling, telecommunications spaces, cable pathways, patch panels, rack elevations, and field testing methodologies.

### Referenced International Standards:
- **ANSI/TIA-568.2-D:** Balanced Twisted-Pair Telecommunications Cabling and Components.
- **ISO/IEC 11801-1 (Class E):** Generic Cabling for Customer Premises.
- **ANSI/TIA-569-E:** Telecommunications Pathways and Spaces.
- **ANSI/TIA-606-C:** Administration Standard for Commercial Telecommunications Infrastructure.
- **ANSI/TIA-607-D:** Generic Telecommunications Bonding and Grounding (Earthing) for Customer Premises.

---

## 2. Horizontal Media Selection: Category 6 vs. Category 6A

A critical infrastructure engineering choice is selecting the horizontal copper cable grade. 

```
                             HORIZONTAL CABLING SELECTION
                                          │
                ┌─────────────────────────┴─────────────────────────┐
                ▼                                                   ▼
     HORIZONTAL FLOOR DISTRIBUTION                             DATA CENTER / MDF SAN
        (Access to Workstations, APs, ELV)                       (Server-to-Switch Interconnects)
                │                                                   │
                ▼                                                   ▼
     Category 6 UTP (250 MHz)                                  Category 6A F/UTP (500 MHz)
     Diameter: 5.5 mm (LSZH)                                   Diameter: 7.5 mm (Shielded)
     Max Channel: 1G / 2.5G @ 100 m                            Max Channel: 10GBASE-T @ 100 m
     Cost Ratio: 1.0x (Baseline)                               Cost Ratio: 2.3x (Material + Labor)
```

### Detailed Engineering Justification:

| Evaluation Parameter | Category 6 UTP (23 AWG) | Category 6A F/UTP (23 AWG) | Design Decision & Engineering Rationale |
|---|---|---|---|
| **Bandwidth Frequency** | **250 MHz** | **500 MHz** | Cat6 easily fulfills all 1000BASE-T and 2.5GBASE-T (Wi-Fi 6) Nyquist frequency requirements. |
| **Data Rate Capability** | 1 Gbps @ 100 m / 2.5 Gbps @ 100 m / 10 Gbps up to 55 m | 10 Gbps @ 100 m | Edge devices (VoIP, cameras, workstations, APs) require 1 Gbps or 2.5 Gbps. No edge workstation requires 10 Gbps. |
| **Cable Outer Diameter** | **5.5 mm** | **7.5 mm – 8.2 mm** | Cat6's smaller cross-sectional area requires **45% less space in ceiling cable trays and conduit sleeves**, eliminating tray overcrowding. |
| **Bend Radius & Flexibility**| $4 \times \text{OD} \approx 22\text{ mm}$ | $8 \times \text{OD} \approx 65\text{ mm}$ | Thinner Cat6 cables drastically simplify neat dressing in 18U/24U IDF wall cabinets and tight desk furniture pathways. |
| **Termination Density** | Standard 24-port 1U RJ45 Patch Panels | Staggered or Shielded Patch Panels | Cat6 terminates on high-density keystone jacks with standard punch-down tools in half the labor time of shielded 6A foil. |
| **Total Cost Impact** | Baseline ($1.0\times$) | $+130\%\text{–}+160\%$ ($2.3\times$ to $2.6\times$) | Deploying Cat6A for all 350+ horizontal runs would add substantial cost without technical benefit at the access layer. |
| **Design Selection** | **All Horizontal Access Drops (MDF/IDF to Workstations, APs, CCTV, ACS)** | **MDF Server Rack Interconnects (Top-of-Rack to Hypervisors)** | **Optimal Hybrid Strategy:** Cat6 UTP deployed horizontally; 10G SFP+ DAC / Cat6A reserved for server storage. |

---

## 3. Horizontal Cabling Topologies & Length Rules

All horizontal cabling adheres strictly to the **Hierarchical Star Topology** terminating at the floor IDF:

```
[Workstation / Phone]
         │
         │ Cat6 Solid Copper Horizontal Run (Max 90 m Permanent Link)
         ▼
[Floor Patch Panel in IDF]
         │
         │ Cat6 Flexible Stranded Patch Cord (Max 5 m in Rack)
         ▼
[Floor Access Switch Port]
```

### The 100-Meter Channel Rule (ANSI/TIA-568-D):

$$\text{Total Channel Length} = L_{\text{Permanent Link}} + L_{\text{Workstation Cord}} + L_{\text{Patch Cord}} \le \mathbf{100\text{ meters}}$$

- **Permanent Link Limit:** $\le \mathbf{90\text{ meters}}$ (Solid 23 AWG horizontal cable from IDF patch panel to workstation faceplate).
- **Work Area Patch Cord:** $\le \mathbf{5\text{ meters}}$ (Stranded Cat6 factory-molded cord).
- **IDF Comms Rack Patch Cord:** $\le \mathbf{5\text{ meters}}$ (Stranded Cat6 patch cord).
- **Longest Measured Horizontal Run:** **68 meters** (Floor 1 IDF to far Southeast corner camera), maintaining a **22-meter safety margin** beneath the 90-meter limit.

---

## 4. Cable Pathway & Containment Architecture

To protect cables from mechanical tension, electromagnetic interference (EMI), and ambient heat:
1. **Vertical Riser Shaft:** 
   - 2 $\times$ 100 mm heavy-duty hot-dip galvanized steel conduit sleeves penetrate floor slabs between MDF and Floors 1, 2, and 3.
   - Firestop pillows and intumescent elastomeric sealant (UL 1479 / ASTM E814 compliant) seal all riser penetrations to maintain 2-hour building fire ratings.
2. **Ceiling Horizontal Trays:**
   - 300 mm wide $\times$ 50 mm deep wire mesh cable basket tray (Cablofil / Legrand) suspended above the false ceiling along main corridors.
   - Trapeze-style threaded rod hangers spaced at 1.5-meter intervals.
3. **Perimeter Wall Trunking:**
   - Dual-compartment PVC dado trunking (50 $\times$ 100 mm) runs along office perimeter baseboards, providing complete physical metal/barrier segregation between 230V AC power wiring and low-voltage Cat6 data cabling.
4. **Separation from High-Voltage Interference:**
   - Minimum **300 mm separation** maintained between Cat6 data cables and fluorescent light fixtures, transformers, and 415V power distribution runs.

---

## 5. Telecommunications Room Rack Elevations

### 5.1 MDF / Server Room (Ground Floor): 2 $\times$ 42U Server Cabinets (800 $\times$ 1000 mm)

```
┌─────────────────────────────────────────────────────────┐
│ RACK 01: NETWORK & PERIMETER INFRASTRUCTURE (MDF - 42U) │
├───────┬─────────────────────────────────────────────────┤
│ 41-42U│ Fiber Patch Panel ODF (24-Port LC OM4 / OS2)    │
│ 39-40U│ 2x 24-Port Cat6 Patch Panels (Ground Access)   │
│ 38U   │ Horizontal Cable Management Duct (1U)           │
│ 36-37U│ 1x Cisco Catalyst 9300-48P Access Switch (GF)  │
│ 34-35U│ 2x Cisco Catalyst 9500-24Y4C Core Switches     │
│ 32-33U│ 2x FortiGate / Palo Alto NGFW Firewalls (HA)    │
│ 30-31U│ 2x Cisco Catalyst 9800-L WLC Controllers (HA)  │
│ 28-29U│ Out-of-band Terminal Server & Master PDU        │
│ 10-27U│ Cable Management / Expansion Buffer             │
│ 04-09U│ APC Smart-UPS SRT 10000VA (Electronics + Batt) │
│ 01-03U│ APC External Battery Pack (SRT192RMBP2)         │
└───────┴─────────────────────────────────────────────────┘
```

### 5.2 Typical IDF (Floors 1, 2, 3): 18U Wall-Mount Swing-Out Cabinets (600 $\times$ 600 mm)

```
┌─────────────────────────────────────────────────────────┐
│ TYPICAL FLOOR IDF CABINET (Floors 1, 2, 3 - 18U)        │
├───────┬─────────────────────────────────────────────────┤
│ 17-18U│ 12-Port Fiber Patch Panel ODF (LC Duplex OM4)   │
│ 15-16U│ 2x 24-Port Cat6 RJ-45 Patch Panels (Data/Voice) │
│ 14U   │ 1U Brush Strip Horizontal Cable Organizer       │
│ 12-13U│ 1x 24-Port Cat6 RJ-45 Patch Panel (ELV/AP)      │
│ 10-11U│ 2x Cisco Catalyst 9300-48P Switches (Stacked)   │
│ 08-09U│ 1U Cable Management Duct                        │
│ 06-07U│ Access Control & Alarm Relay Interface Unit     │
│ 03-05U│ APC Smart-UPS SRT 3000VA (2U Rackmount)         │
│ 01-02U│ 1U Rack-Mount Switched PDU (C13 Outlets)        │
└───────┴─────────────────────────────────────────────────┘
```

---

## 6. Administration & TIA-606-C Labelling Scheme

To ensure instant operational traceability from user faceplates back to switchports, every outlet, cable, patch panel port, and rack follows the ANSI/TIA-606-C labelling standard:

$$\mathbf{[Floor]}\,.\,\mathbf{[Room]}\,-\mathbf{[Rack]}\,.\,\mathbf{[Panel]}\,-\mathbf{[Port]}$$

### Labelling Examples:
- **Workstation Drop on Floor 1 Desk 12:** `1.IDF1-PP01-012`
- **Ceiling Wi-Fi 6 AP on Floor 2:** `2.IDF2-PP03-001`
- **CCTV Camera at Ground Reception:** `G.MDF-PP02-005`
- **Core-to-Floor Riser Fiber Pair:** `RSR-MDF/IDF1-FO01`

---

## 7. Field Testing & Certification Standards

Following installation, **100% of horizontal copper links** must undergo rigorous certification testing using an enterprise Class E / Level 2G cable analyzer (e.g. **Fluke Networks DSX-8000 CableAnalyzer**):

| Test Parameter | Acceptance Criteria (ANSI/TIA-568.2-D Class E) | Impact of Test Failure |
|---|---|---|
| **Wiremap** | Pin-to-pin continuity on all 8 conductors (T568B sequence); zero split pairs or shorts. | Open circuit, split pair cross-talk, failure to link. |
| **Length** | Permanent link $\le \mathbf{90.0\text{ meters}}$. | Signal attenuation, propagation delay exceeding CSMA/CD collision window. |
| **Insertion Loss (Attenuation)**| $\le \mathbf{21.3\text{ dB}}$ @ 250 MHz. | High signal dampening; bit errors under traffic. |
| **Near-End Crosstalk (NEXT)** | $\ge \mathbf{39.3\text{ dB}}$ @ 250 MHz. | Signal bleed between adjacent pairs; caused by untwisting conductors $> 13\text{ mm}$ during termination. |
| **Power Sum NEXT (PS-NEXT)** | $\ge \mathbf{36.3\text{ dB}}$ @ 250 MHz. | Multi-pair interference; packet retransmissions. |
| **Return Loss (RL)** | $\ge \mathbf{12.0\text{ dB}}$ @ 250 MHz. | Signal reflections caused by cable crushing, kinks, or improper punch-down tension. |
| **Propagation Delay & Skew** | Delay $\le 498\text{ ns}$; Skew $\le \mathbf{35\text{ ns}}$. | Jitter in Gigabit Ethernet transmission. |

> [!NOTE]
> All test results must be exported directly from the Fluke LinkWare software in tamper-proof `.flw` format and archived in the project handover records.
