# Network & ELV Pre-Commissioning Verification Checklist

## 1. Overview & Purpose

This pre-commissioning checklist defines the mandatory physical, electrical, environmental, and passive infrastructure inspections that must be successfully executed and approved prior to energizing active network hardware or commencing Layer 2/Layer 3 system commissioning.

### Facility Details:
- **Project Name:** Enterprise Office Network & ELV/ICT Infrastructure
- **Building Location:** Headquarters 4-Storey Commercial Building (Ground + 3 Floors)
- **Inspection Date:** ___________________
- **Lead Commissioning Engineer:** ___________________
- **Lead Electrical & Cabling Inspector:** ___________________

---

## 2. Telecommunications Room Physical & Environmental Audit

| Item No | Inspection Check Description | Verification Method | Acceptance Standard / Target | Status (Pass / Fail / N/A) | Notes / Observations |
|:---:|---|---|---|:---:|---|
| **ENV-01** | MDF & IDF Architectural Readiness | Visual inspection | Rooms clean, dry, free of construction debris/dust; painted with anti-static paint; sealed doors with functional locks. | [  ] Pass | |
| **ENV-02** | Room Ambient Operating Temperature | Calibrated Thermometer | Maintained continuously between **$18^\circ\text{C}$ and $22^\circ\text{C}$ ($64^\circ\text{F}\text{–}72^\circ\text{F}$)**. | [  ] Pass | |
| **ENV-03** | Dedicated HVAC Airflow & Redundancy | Physical inspection | Dedicated split AC unit operational 24/7; auto-restart enabled upon power restoration; positive air pressure maintained. | [  ] Pass | |
| **ENV-04** | Relative Humidity Levels | Digital Hygrometer | Relative humidity non-condensing between **$30\%\text{ and }55\%$**. | [  ] Pass | |
| **ENV-05** | Fire Suppression & Detection System | Smoke / Clean Agent Test | Clean agent fire suppression (FM-200 / Novec 1230) or pre-action dry pipe installed; VESDA aspirating smoke detector armed. | [  ] Pass | |
| **ENV-06** | Room Lighting & Emergency Lamps | Photometer measurement | Minimum **500 Lux** illumination at 1 meter above finished floor (AFF); emergency battery backup lights functional. | [  ] Pass | |

---

## 3. Electrical Power, UPS & Earthing (ANSI/TIA-607-D)

| Item No | Inspection Check Description | Verification Method | Acceptance Standard / Target | Status (Pass / Fail / N/A) | Notes / Observations |
|:---:|---|---|---|:---:|---|
| **PWR-01** | Primary Utility Feed & Breaker Sizing | Electrical Multimeter | Dedicated 230V AC 32A single-phase feed to MDF; 16A feeds to IDFs; verified voltage $230\text{V} \pm 5\%$ at 50 Hz. | [  ] Pass | |
| **PWR-02** | Telecommunications Main Grounding Busbar (TMGB) | Ground Resistance Clamp Meter | MDF equipped with standard copper TMGB ($50 \times 6\text{ mm}$); resistance to building earth electrode **$< 5.0\text{ Ohms}$**. | [  ] Pass | |
| **PWR-03** | Telecommunications Grounding Busbars (TGB) | Visual & Continuity Test | Each IDF fitted with copper TGB; bonded to TMGB via 16 $\text{mm}^2$ green/yellow insulated copper grounding backbone cable. | [  ] Pass | |
| **PWR-04** | Rack Cabinet Frame Bonding | Ohm-meter continuity check | All 42U and 18U rack frames, doors, side panels, and PDU housings bonded to TGB using 6 $\text{mm}^2$ copper ground straps. | [  ] Pass | |
| **PWR-05** | MDF UPS Installation & Battery Status | UPS Self-Test & Diagnostic | APC 10 kVA UPS powered on; zero active faults; external battery module detected; verified float charge voltage $218\text{V DC}$. | [  ] Pass | |
| **PWR-06** | Floor IDF UPS Systems (3x 3kVA) | UPS Diagnostics & Status | APC 3 kVA UPS units operational in IDF-1, IDF-2, IDF-3; internal batteries fully charged; LCD shows 100% battery capacity. | [  ] Pass | |
| **PWR-07** | Rack-Mount Metered PDUs | Socket Voltage & Polarity Tester | Verified correct Line, Neutral, Earth polarity on all IEC C13 outlets; master PDU input cords plugged into UPS output bus. | [  ] Pass | |

---

## 4. Passive Structured Cabling & Fiber Optic Certification Audit

| Item No | Inspection Check Description | Verification Method | Acceptance Standard / Target | Status (Pass / Fail / N/A) | Notes / Observations |
|:---:|---|---|---|:---:|---|
| **CAB-01** | Horizontal Cat6 Fluke DSX-8000 Reports | LinkWare Test Record Review | **100% of 350 copper links** certified to ANSI/TIA-568.2-D Class E; zero failed links; verified NEXT, Return Loss, and Length $< 90\text{m}$. | [  ] Pass | |
| **CAB-02** | TIA-606-C Physical Labelling Audit | Random 20% Sampling | Faceplates, patch panel ports, and cable jackets labelled with deterministic alphanumeric code matching [`outlet-schedule.csv`](../docs/elv/outlet-schedule.csv). | [  ] Pass | |
| **CAB-03** | Vertical Riser Fiber Tier 1 Optical Loss | Optical Loss Test Set (OLTS) | End-to-end optical insertion loss at 850 nm for all 12 OM4 fiber cores **$\le 1.50\text{ dB}$** (Target budget: $2.60\text{ dB}$). | [  ] Pass | |
| **CAB-04** | Vertical Riser Fiber Tier 2 Reflectometry | Optical Time-Domain Reflectometer (OTDR) | OTDR traces verify smooth attenuation slope; connector reflectance **$< -35\text{ dB}$**; zero micro-bends or macro-bends. | [  ] Pass | |
| **CAB-05** | Fiber Optic Connector End-Face Cleanliness | 400x Fiber Inspection Microscope | All LC-Duplex connector end-faces inspected and cleaned per IEC 61300-3-35 standard; zero dust, scratches, or oil contamination. | [  ] Pass | |
| **CAB-06** | Cable Containment & Bend Radius Audit | Visual inspection | Cable basket trays and conduit sleeves filled to $< 50\%$ capacity; minimum $4\times$ cable OD bend radius maintained; Velcro ties only (no zip-ties). | [  ] Pass | |
| **CAB-07** | Firestop Penetration Seal Integrity | Physical audit of riser sleeves | All floor slab penetrations sealed airtight with UL-classified intumescent firestop pillows and elastomeric fire caulk. | [  ] Pass | |

---

## 5. Formal Pre-Commissioning Approval & Sign-Off

I hereby certify that the physical environment, electrical supply, grounding, and passive structured cabling systems have been thoroughly inspected, tested, and approved for active equipment energization and system commissioning.

- **Lead Network Commissioning Engineer:** __________________________  **Signature:** __________________ **Date:** ____________
- **Lead Electrical & ELV Site Inspector:** __________________________  **Signature:** __________________ **Date:** ____________
- **Project General Contractor Representative:** __________________________  **Signature:** __________________ **Date:** ____________

