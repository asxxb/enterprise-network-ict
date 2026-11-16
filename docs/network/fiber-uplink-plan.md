# Fiber Optic Uplink Infrastructure & Transceiver Engineering Plan

## 1. Overview & Structured Riser Design

This document details the optical fiber backbone infrastructure connecting the Main Distribution Frame (MDF) in the Ground Floor Server Room to the Intermediate Distribution Frames (IDFs) on Floors 1, 2, and 3, as well as the external Service Provider (ISP) demarcation link.

### Design Objectives:
- Deliver 20 Gbps (2 $\times$ 10G LACP LAG) non-blocking backbone bandwidth from the Collapsed Core switches to each floor access stack.
- Implement physical cable path diversity in the vertical riser shaft.
- Optimize component expenditure by matching optical transceiver types to link distances.
- Establish deterministic link loss budgets conforming to TIA-568-D and ISO/IEC 11801 standards.

---

## 2. Fiber Media Selection: Multimode (OM4) vs. Single-Mode (OS2)

A primary architectural decision in campus engineering is selecting between Multimode Fiber (MMF) and Single-Mode Fiber (SMF). 

```
                                  CAMPUS MEDIA SELECTION
                                             │
                   ┌─────────────────────────┴─────────────────────────┐
                   ▼                                                   ▼
       INTERNAL RISER BACKBONE                                 EXTERNAL WAN DEMARC
        (MDF to IDFs < 70 m)                                  (ISP Metro-E Hand-off)
                   │                                                   │
                   ▼                                                   ▼
       OM4 Multimode Fiber (50/125 μm)                        OS2 Single-Mode Fiber (9/125 μm)
       Transceiver: 10GBASE-SR (850 nm)                       Transceiver: 1000BASE-LX (1310 nm)
       Cost: ~$35 / SFP+ Optic                                Cost: ~$85 / SFP Optic
```

### Detailed Media Comparison & Engineering Justification:

| Parameter | OM4 Multimode Fiber (50/125 $\mu$m) | OS2 Single-Mode Fiber (9/125 $\mu$m) | Engineering Evaluation & Decision |
|---|---|---|---|
| **Core / Cladding Diameter** | $50 / 125\,\mu\text{m}$ (Laser-Optimized) | $9 / 125\,\mu\text{m}$ | OM4's larger core simplifies termination and connector alignment in enterprise patch panels. |
| **Max Reach at 10 Gbps** | **400 meters** (10GBASE-SR) | **10,000 meters** (10GBASE-LR) | The longest vertical cable run in the building is **65 meters** (MDF to Floor 3). OM4 provides over 600% distance headroom. |
| **Max Reach at 25 Gbps** | **100 meters** (25GBASE-SR) | **10,000 meters** (25GBASE-LR) | OM4 provides an immediate, zero-cabling upgrade path to 25 Gbps uplinks without pulling new glass. |
| **Transceiver Unit Cost** | Low ($\approx \$35\text{–}\$50$ per 10G-SR SFP+) | Moderate ($\approx \$120\text{–}\$180$ per 10G-LR SFP+) | Equipping 16 switch uplink ports with 10G-SR optics reduces optical hardware procurement costs by over 70%. |
| **Cable Attenuation @ 850 nm** | $\le 3.0\text{ dB/km}$ | N/A (Operates at 1310 / 1550 nm) | Over a 65m run, fiber attenuation is negligible ($< 0.2\text{ dB}$). |
| **Application in this Design** | **Internal Riser Backbone (MDF $\leftrightarrow$ IDFs)** | **ISP Demarcation / WAN Handoff Link** | **Hybrid Architecture:** OM4 utilized internally for cost efficiency; OS2 utilized for external carrier handoff. |

---

## 3. Optical Link Budget Calculation

To guarantee error-free 10 Gigabit Ethernet transmission (Bit Error Rate $\text{BER} \le 10^{-12}$), the total insertion loss must not exceed the maximum allowable channel attenuation defined by the IEEE 802.3ae standard.

### 3.1 Optical Power Budget Formula

$$\text{Power Budget } (P_B) = P_{\text{TX, Min}} - P_{\text{RX, Sensitivity}}$$

$$\text{Total Channel Insertion Loss } (L_{\text{Total}}) = (\alpha \times L) + (N_{\text{conn}} \times L_{\text{conn}}) + (N_{\text{splice}} \times L_{\text{splice}}) + M$$

Where:
- $\alpha$: Fiber attenuation coefficient at 850 nm ($3.0\text{ dB/km} = 0.003\text{ dB/m}$).
- $L$: Cable length in meters.
- $N_{\text{conn}}$: Number of mated connector pairs ($2\text{ pairs}$: MDF patch panel + IDF patch panel).
- $L_{\text{conn}}$: Maximum insertion loss per connector pair ($0.35\text{ dB}$ per TIA-568-D).
- $N_{\text{splice}}$: Number of fusion splices ($2\text{ splices}$ per pigtail termination).
- $L_{\text{splice}}$: Maximum insertion loss per fusion splice ($0.10\text{ dB}$).
- $M$: Operating safety margin / aging buffer ($3.0\text{ dB}$).

### 3.2 Worked Calculation for Longest Run (MDF to Floor 3 IDF: 65 meters)

1. **Optical Power Budget of 10GBASE-SR SFP+:**
   - Minimum Transmit Power ($P_{\text{TX, Min}}$): $-7.3\text{ dBm}$
   - Receiver Sensitivity ($P_{\text{RX, Sensitivity}}$): $-11.1\text{ dBm}$
   - **Allowable Power Budget ($P_B$):** $-7.3\text{ dBm} - (-11.1\text{ dBm}) = \mathbf{3.8\text{ dB}}$ (Standard IEEE 802.3ae channel loss limit is **$2.6\text{ dB}$** at 10 Gbps over OM4).
   *Note: High-grade commercial SFP+ modules typically provide up to $7.3\text{ dB}$ total power budget.*

2. **Calculated Channel Losses:**
   - **Fiber Cable Loss:** $0.003\text{ dB/m} \times 65\text{ m} = \mathbf{0.195\text{ dB}}$
   - **Connector Loss (2 LC-Duplex pairs):** $2 \times 0.35\text{ dB} = \mathbf{0.700\text{ dB}}$
   - **Fusion Splice Loss (2 splices):** $2 \times 0.10\text{ dB} = \mathbf{0.200\text{ dB}}$
   - **Total Channel Insertion Loss ($L_{\text{Channel}}$):** $0.195 + 0.700 + 0.200 = \mathbf{1.095\text{ dB}}$

3. **Margin Calculation:**
   $$\text{Operating Margin} = \text{Max Allowable Loss } (2.600\text{ dB}) - L_{\text{Channel }} (1.095\text{ dB}) = \mathbf{+1.505\text{ dB}}$$
   *Conclusion:* The calculated channel loss of **$1.10\text{ dB}$** is well within the IEEE limit of **$2.60\text{ dB}$**, leaving an excellent operating safety margin of **$1.51\text{ dB}$** for cable aging and future patching.

---

## 4. Optical Transceiver & SFP+ Selection Table

| Transceiver Type | Form Factor | Data Rate | Wavelength | Fiber Medium | Connector | Max Reach | Target Link Deployment |
|---|:---:|:---:|:---:|:---:|:---:|:---:|---|
| **10GBASE-SR** | SFP+ | 10 Gbps | 850 nm | OM4 MMF (50/125 $\mu$m) | LC Duplex | 400 m | MDF-to-Floor Riser Uplinks (Core to Access) |
| **10GBASE-CU (DAC)**| SFP+ Passive | 10 Gbps | Copper | Twinaxial Copper | SFP+ Direct | 3 m | In-Rack Core-to-Core & Core-to-NGFW Links |
| **1000BASE-LX** | SFP | 1 Gbps | 1310 nm | OS2 SMF (9/125 $\mu$m) | LC Duplex | 10 km | ISP WAN Demarcation Handoff to NGFW |
| **1000BASE-SX** | SFP | 1 Gbps | 850 nm | OM4 MMF (50/125 $\mu$m) | LC Duplex | 550 m | Out-of-band management console & spare optics |

---

## 5. Riser Pathway, Patch Panels & TIA-606-B Labelling

### 5.1 Fiber Patch Panel Allocation
- **MDF (Ground Server Room):** 1 $\times$ 24-Port 1U 19" Optical Distribution Frame (ODF) fitted with LC-Duplex aqua couplers.
- **Each IDF (Floors 1, 2, 3):** 1 $\times$ 12-Port 1U 19" ODF fitted with LC-Duplex couplers.

### 5.2 Structured TIA-606-B Port Labelling Convention
All optical ports and patch cables follow a deterministic alphanumeric code:

$$\mathbf{[Origin\ Rack]}-\mathbf{[Panel\ ID]}-\mathbf{[Port]} \;\longleftrightarrow\; \mathbf{[Destination\ Rack]}-\mathbf{[Panel\ ID]}-\mathbf{[Port]}$$

| Link Description | Origin ODF Port (MDF) | Destination ODF Port (Floor IDF) | Cable Tag / Core ID | SFP+ Port Core SW | SFP+ Port Floor SW |
|---|---|---|---|---|---|
| **MDF $\leftrightarrow$ GF Access Link 1** | `MDF-ODF01-P01/02` | `IDF0-ODF01-P01/02` | `FO-MDF-IDF0-C01/02` | Core-01 `Te1/0/1` | Acc-SW-GF `Te1/0/49` |
| **MDF $\leftrightarrow$ GF Access Link 2** | `MDF-ODF01-P03/04` | `IDF0-ODF01-P03/04` | `FO-MDF-IDF0-C03/04` | Core-02 `Te1/0/1` | Acc-SW-GF `Te1/0/50` |
| **MDF $\leftrightarrow$ F1 Access Link 1** | `MDF-ODF01-P05/06` | `IDF1-ODF01-P01/02` | `FO-MDF-IDF1-C01/02` | Core-01 `Te1/0/2` | Acc-SW-F1 `Te1/0/49` |
| **MDF $\leftrightarrow$ F1 Access Link 2** | `MDF-ODF01-P07/08` | `IDF1-ODF01-P03/04` | `FO-MDF-IDF1-C03/04` | Core-02 `Te1/0/2` | Acc-SW-F1 `Te1/0/50` |
| **MDF $\leftrightarrow$ F2 Access Link 1** | `MDF-ODF01-P09/10` | `IDF2-ODF01-P01/02` | `FO-MDF-IDF2-C01/02` | Core-01 `Te1/0/3` | Acc-SW-F2 `Te1/0/49` |
| **MDF $\leftrightarrow$ F2 Access Link 2** | `MDF-ODF01-P11/12` | `IDF2-ODF01-P03/04` | `FO-MDF-IDF2-C03/04` | Core-02 `Te1/0/3` | Acc-SW-F2 `Te1/0/50` |
| **MDF $\leftrightarrow$ F3 Access Link 1** | `MDF-ODF01-P13/14` | `IDF3-ODF01-P01/02` | `FO-MDF-IDF3-C01/02` | Core-01 `Te1/0/4` | Acc-SW-F3 `Te1/0/49` |
| **MDF $\leftrightarrow$ F3 Access Link 2** | `MDF-ODF01-P15/16` | `IDF3-ODF01-P03/04` | `FO-MDF-IDF3-C03/04` | Core-02 `Te1/0/4` | Acc-SW-F3 `Te1/0/50` |
| **ISP WAN Demarc Primary** | `MDF-ODF01-P21/22` | ISP Demarcation Wall Box | `FO-ISP-WAN-01` | NGFW-01 `eth1/1` | ISP NTU Port 1 |
| **ISP WAN Demarc Secondary** | `MDF-ODF01-P23/24` | ISP Demarcation Wall Box | `FO-ISP-WAN-02` | NGFW-02 `eth1/1` | ISP NTU Port 2 |

