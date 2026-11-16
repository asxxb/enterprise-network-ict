# IP Video Surveillance (CCTV) System Architecture

## 1. Overview & Surveillance Objectives

This document establishes the technical design for the converged IP Video Surveillance (CCTV) system deployed across the 4-storey enterprise headquarters building. 

The system provides 24/7 continuous visual monitoring, forensic recording, and automated perimeter alerting across all sensitive internal and external zones while maintaining strict physical security network isolation.

### Key Design Parameters:
- **Total Cameras:** **32 PoE IP Cameras** (8 cameras per floor across Ground, 1, 2, 3).
- **Video Compression:** **H.265 (High Efficiency Video Coding - HEVC)** with dynamic Group of Pictures (GOP) and Smart Codec bitrate suppression.
- **Recording Resolution & Frame Rate:** Full HD 1080p (1920 $\times$ 1080) at **25 frames per second (fps)**.
- **Storage Retention Period:** **30 Days continuous 24/7 recording** for all cameras.
- **Network Isolation:** Dedicated non-routable **VLAN 70 (`10.10.70.0/24`)** with zero Internet access.

---

## 2. Camera Hardware Types & Spatial Allocation

Cameras are strategically chosen based on environmental conditions, field of view (FOV), lighting dynamics (Wide Dynamic Range - WDR), and vandal resistance (IK10 impact rating):

| Camera Type | Lens & Optics | Resolution | IR Distance | Environmental Rating | Typical Building Locations | Qty |
|---|---|:---:|:---:|:---:|---|:---:|
| **Type A: Fixed Indoor Dome** | 2.8 mm Fixed ($106^\circ\text{ H-FOV}$), True WDR (120 dB) | 1080p (2 MP) | 30 m Smart IR | Indoor IK10 | Corridors, elevator lobbies, stairwells, pantry | 18 |
| **Type B: Panoramic Turret** | 2.0 mm Ultra-Wide ($120^\circ\text{ H-FOV}$), 3D DNR | 4 MP (1440p) | 30 m Smart IR | Indoor IK10 | Reception lounge, main staff cafeteria, open office | 6 |
| **Type C: Varifocal Bullet** | 2.8–12 mm Motorized Varifocal ($98^\circ\text{–}32^\circ$), WDR | 4 MP (1440p) | 50 m EXIR | Outdoor IP67 / IK10 | Main building exterior entry, rear loading bay | 4 |
| **Type D: High-Security Dome** | 2.8 mm Fixed, Audio Line-In, Low-Light Starlight | 1080p (2 MP) | 20 m Smart IR | Indoor Anti-Tamper | Server room / MDF, cash handling, security office | 4 |
| **Total Cameras** | — | — | — | — | — | **32** |

### Floor-by-Floor Camera Deployment Schedule:

```
┌────────────────────────────────────────────────────────────────────────┐
│ CAMERA ALLOCATION SCHEDULE (32 CAMERAS TOTAL)                         │
├──────────────┬─────┬───────────────────────────────────────────────────┤
│ Ground Floor │  8  │ 2x Main Entry/Reception, 2x Exterior Entrances,   │
│              │     │ 1x Elevator Lobby, 1x Server Room MDF, 2x Passages│
├──────────────┼─────┼───────────────────────────────────────────────────┤
│ Floor 1      │  8  │ 1x Elevator Lobby, 2x Main Corridors,             │
│              │     │ 2x Open Office Wings, 2x Fire Stairwells, 1x Cafe │
├──────────────┼─────┼───────────────────────────────────────────────────┤
│ Floor 2      │  8  │ 1x Elevator Lobby, 2x Main Corridors,             │
│              │     │ 2x Open Office Wings, 2x Fire Stairwells, 1x Quiet│
├──────────────┼─────┼───────────────────────────────────────────────────┤
│ Floor 3      │  8  │ 1x Elevator Lobby, 2x Executive Corridors,        │
│              │     │ 1x Boardroom Entry, 2x Offices, 2x Fire Stairwells│
└──────────────┴─────┴───────────────────────────────────────────────────┘
```

---

## 3. Video Streaming Parameters & Bitrate Calculations

Each camera generates two simultaneous IP video streams:
1. **Primary Stream (Recording):** High-definition 1080p / 4MP @ 25 fps, H.265 compression, encoded at a constant bitrate (CBR) target of **4.0 Mbps** for continuous recording on the NVR.
2. **Secondary Stream (Live Monitoring):** Standard-definition 4CIF / D1 (720 $\times$ 576) @ 15 fps, H.265 compression, encoded at **512 kbps** for live multi-screen split displays (16-split or 32-split matrix) on security control room monitors without overburdening GPU decoding or switch links.

### Network Throughput Impact:
- Single camera recording bitrate: $4.0\text{ Mbps}$
- Aggregate continuous recording bandwidth across building:
  $$\text{Aggregate Throughput} = 32\text{ cameras} \times 4.0\text{ Mbps} = \mathbf{128.0\text{ Mbps}}$$
- With secondary live streams included ($32 \times 0.512\text{ Mbps} = 16.4\text{ Mbps}$), total network streaming bandwidth is $\approx \mathbf{144.4\text{ Mbps}}$.
- Because floor switch stacks connect to the Core via **20 Gbps (2 $\times$ 10G) LACP fiber Port-Channels**, the 144 Mbps surveillance stream utilizes **less than 0.75%** of available riser bandwidth.

---

## 4. NVR Sizing & Storage Retention Calculations

Enterprise security policy mandates continuous 24/7 video recording retention for a minimum of **30 calendar days**.

### 4.1 Storage Sizing Formula

$$\text{Daily Storage per Camera (GB)} = \frac{\text{Bitrate (kbps)} \times 3600\,\text{s/hr} \times 24\,\text{hr/day}}{8\text{ bits/byte} \times 1,024 \times 1,024} \approx \frac{\text{Bitrate (Mbps)} \times 86,400}{8 \times 1,000}$$

$$\text{Daily Storage (GB)} = \frac{4.0\text{ Mbps} \times 86,400\text{ seconds}}{8 \times 1,000} \approx \mathbf{43.2\text{ GB per camera / day}}$$

### 4.2 30-Day Usable Storage Calculation (32 Cameras)

$$\text{Total 30-Day Usable Storage} = 32\text{ cameras} \times 43.2\text{ GB/day} \times 30\text{ days} = 41,472\text{ GB} \approx \mathbf{41.5\text{ Terabytes (TB)}}$$

Adding a **20% system overhead buffer** (filesystem metadata, database indexing, motion event tagging, snapshot caching):

$$\text{Target Usable Capacity} = 41.5\text{ TB} \times 1.20 = \mathbf{49.8\text{ TB Usable}}$$

### 4.3 Hardware RAID & Hard Disk Sizing
To prevent data loss from physical disk failure and maintain 100% video recording continuity, the central Network Video Recorder (NVR) is equipped with a hardware **RAID 6 array (dual parity)** plus one global Hot Spare ($N+2+1$):

- **Drive Selection:** Enterprise Surveillance SATA 6 Gb/s 3.5" HDDs (Seagate SkyHawk AI or WD Purple Pro, 7200 RPM, 256MB cache, 2.5M hours MTBF).
- **Drive Capacity:** **8 Terabyte (TB)** drives ($7.28\text{ TB}$ usable formatted per disk).
- **Array Configuration:** **8 $\times$ 8TB Drives in RAID 6** (6 data disks + 2 parity disks) + **1 Hot-Spare**:
  $$\text{RAID 6 Usable Storage} = (8 - 2) \times 7.28\text{ TB} = \mathbf{43.68\text{ TB}}$$
  *Note:* To achieve the full 49.8 TB target with 30-45 day growth margin, an **optional 12-bay chassis with 10 $\times$ 8TB disks in RAID 6** provides $(10 - 2) \times 7.28\text{ TB} = \mathbf{58.24\text{ TB Usable}}$, providing **42 days of actual retention**.

---

## 5. Video Management Software (VMS) & Network Isolation

### 5.1 Dual-Homed NVR Architecture
The Network Video Recorder operates as a secure hardware bridge between the isolated CCTV network and the corporate user network:

```
┌────────────────────────────────────────────────────────────────────────┐
│ ISOLATED CCTV NETWORK (VLAN 70: 10.10.70.0/24)                        │
│  [32x IP Cameras] ──(Cat6 PoE)──> [Floor Switches]                     │
│                                           │ (VLAN 70 802.1Q Trunk)    │
│                                           ▼                           │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ NVR NIC 1: IP 10.10.70.5 (Recording Storage Ingest Interface)     │  │
│  │ - PULLS RTSP STREAMS FROM CAMERAS                                 │  │
│  │ - ZERO INTERNET / NO WORKSTATION ROUTING                         │  │
│  ├──────────────────────────────────────────────────────────────────┤  │
│  │ NVR NIC 2: IP 10.10.20.16 (VMS Management / Server LAN Interface) │  │
│  │ - SERVES WEB / THIN-CLIENT VMS PORTAL TO OPERATORS                │  │
│  │ - AUTHENTICATES AGAINST ACTIVE DIRECTORY VIA RADIUS/LDAP          │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                           ▲                           │
│                                           │ (HTTPS TCP 443 / 8443)    │
│ [Authorized Security Workstations] ───────┴                           │
│ (In Corporate VLANs 30-33 / VPN Pool)                                  │
└────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Zero-Trust Security Enforcement Rules:
1. **Air-Gapped Camera Network:** Cameras on `VLAN 70` have no default gateway to the Internet and cannot initiate sessions to any IP outside their subnet.
2. **No Direct Camera Access:** Standard users, visitors, and compromised workstations cannot ping, browse, or connect to camera IP addresses directly.
3. **Encrypted Operator Sessions:** Security guards and facility managers access camera feeds solely through the VMS client software (Milestone XProtect / Genetec Security Center / HikCentral) connecting to NVR NIC 2 over TLS-encrypted HTTPS (TCP 443/8443).
4. **Physical Port Tamper Protection:** Cisco Sticky MAC port security is active on all camera switchports. Disconnecting a camera instantly drops port link state, preventing unauthorized laptop insertion into the CCTV VLAN.

