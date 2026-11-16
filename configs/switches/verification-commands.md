# Switch Configuration Verification & Operational Guide

## 1. Overview

This document provides a systematic catalog of Cisco IOS-XE operational verification commands (`show` commands) to validate, troubleshoot, and commission every network feature configured in Part 4a and Part 4b across Core and Access switches.

---

## 2. VLANs, Trunks & EtherChannel Verification

### 2.1 Verify VLAN Creation and Port Memberships
```text
Core-SW-01# show vlan brief
```
**Expected Output:**
```text
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
10   MGMT_INFRA                       active    Po1, Po2, Po3, Po4, Po10
20   SERVERS_PROD                     active    Po10, Po20
30   DATA_USERS_GF                    active    Po1, Po10
31   DATA_USERS_F1                    active    Po2, Po10
32   DATA_USERS_F2                    active    Po3, Po10
33   DATA_USERS_F3                    active    Po4, Po10
40   VOICE_VOIP                       active    Po1, Po2, Po3, Po4, Po10
50   WIFI_CORP                        active    Po1, Po2, Po3, Po4, Po10
60   WIFI_GUEST                       active    Gi1/0/20, Po1, Po2, Po3, Po4, Po10
70   ELV_CCTV                         active    Po1, Po2, Po3, Po4, Po10, Po20
80   ELV_ACCESS                       active    Po1, Po2, Po3, Po4, Po10, Po20
90   PRINTERS                         active    Po1, Po2, Po3, Po4, Po10
100  TRANSIT_CORE_FW                  active    Gi1/0/20, Po10
999  NATIVE_BLACKHOLE                 active    Gi1/0/6, Gi1/0/7, Gi1/0/8
```

### 2.2 Verify 802.1Q Trunk Status & Allowed VLANs
```text
Acc-SW-F1# show interfaces trunk
```
**Expected Output:**
```text
Port        Mode             Encapsulation  Status        Native vlan
Po2         on               802.1q         trunking      999
Gi1/0/39    on               802.1q         trunking      10
Gi1/0/49    on               802.1q         trunking      999
Gi1/0/50    on               802.1q         trunking      999

Port        Vlans allowed on trunk
Po2         10,31,40,50,60,70,80,90,999
Gi1/0/39    10,50,60

Port        Vlans in spanning tree forwarding state and not pruned
Po2         10,31,40,50,60,70,80,90
Gi1/0/39    10,50,60
```

### 2.3 Verify LACP Link Aggregation Groups (LAG)
```text
Core-SW-01# show etherchannel summary
```
**Expected Output:**
```text
Number of channel-groups in use: 5
Number of aggregators:           5

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi1/0/1(P) 
2      Po2(SU)         LACP      Gi1/0/2(P) 
3      Po3(SU)         LACP      Gi1/0/3(P) 
4      Po4(SU)         LACP      Gi1/0/4(P) 
10     Po10(SU)        LACP      Gi1/0/23(P) Gi1/0/24(P)
20     Po20(SU)        LACP      Gi1/0/5(P)
```
> [!NOTE]
> Flag **`S`** indicates Layer 2, flag **`U`** indicates in use (operational), and flag **`P`** indicates bundled in port-channel.

---

## 3. Spanning Tree Protocol (Rapid-PVST) Verification

### 3.1 Verify Spanning Tree Mode & Root Placement
```text
Core-SW-01# show spanning-tree summary
```
**Expected Output:**
```text
Switch is in rapid-pvst mode
Root bridge for: VLAN0010, VLAN0020, VLAN0030, VLAN0031, VLAN0032, VLAN0033,
                 VLAN0040, VLAN0050, VLAN0060, VLAN0070, VLAN0080, VLAN0090,
                 VLAN0100, VLAN0999
PortFast Default is disabled
Portfast Edge BPDU Guard Default is disabled
```

### 3.2 Verify Root Bridge on Access Switch
```text
Acc-SW-F1# show spanning-tree vlan 31
```
**Expected Output:**
```text
VLAN0031
  Spanning tree enabled protocol rstp
  Root ID    Priority    4127 (Priority 4096 + sys-id-ext 31)
             Address     0001.42a1.b101 (Core-SW-01)
             Cost        3
             Port        51 (Port-channel2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32800 (Priority 32768 + sys-id-ext 31)
             Address     0009.7c55.3301 (Acc-SW-F1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po2                 Root FWD 3         128.51   P2p
Gi1/0/1             Desg FWD 4         128.1    P2p Edge
Gi1/0/2             Desg FWD 4         128.2    P2p Edge
```

---

## 4. First-Hop Redundancy Protocol (HSRPv2) Verification

### 4.1 Verify HSRP Gateway States on Core-SW-01 (Master)
```text
Core-SW-01# show standby brief
```
**Expected Output:**
```text
                     P indicates configured to PREEMPT.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   110 P Active  local           10.10.10.3      10.10.10.1
Vl20        20   110 P Active  local           10.10.20.3      10.10.20.1
Vl30        30   110 P Active  local           10.10.30.3      10.10.30.1
Vl31        31   110 P Active  local           10.10.31.3      10.10.31.1
Vl32        32   110 P Active  local           10.10.32.3      10.10.32.1
Vl33        33   110 P Active  local           10.10.33.3      10.10.33.1
Vl40        40   110 P Active  local           10.10.40.3      10.10.40.1
Vl50        50   110 P Active  local           10.10.50.3      10.10.50.1
Vl70        70   110 P Active  local           10.10.70.3      10.10.70.1
Vl80        80   110 P Active  local           10.10.80.3      10.10.80.1
Vl90        90   110 P Active  local           10.10.90.3      10.10.90.1
Vl100       100  110 P Active  local           10.10.100.3     10.10.100.1
```

### 4.2 Verify HSRP Standby States on Core-SW-02 (Backup)
```text
Core-SW-02# show standby brief
```
**Expected Output:**
```text
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   100 P Standby 10.10.10.2      local           10.10.10.1
Vl20        20   100 P Standby 10.10.20.2      local           10.10.20.1
Vl30        30   100 P Standby 10.10.30.2      local           10.10.30.1
Vl31        31   100 P Standby 10.10.31.2      local           10.10.31.1
Vl32        32   100 P Standby 10.10.32.2      local           10.10.32.1
Vl33        33   100 P Standby 10.10.33.2      local           10.10.33.1
Vl40        40   100 P Standby 10.10.40.2      local           10.10.40.1
Vl50        50   100 P Standby 10.10.50.2      local           10.10.50.1
Vl70        70   100 P Standby 10.10.70.2      local           10.10.70.1
Vl80        80   100 P Standby 10.10.80.2      local           10.10.80.1
Vl90        90   100 P Standby 10.10.90.2      local           10.10.90.1
Vl100       100  100 P Standby 10.10.100.2     local           10.10.100.1
```

---

## 5. Routing Table & OSPF Verification

### 5.1 Verify Default Route and Connected SVIs
```text
Core-SW-01# show ip route
```
**Expected Output:**
```text
Gateway of last resort is 10.10.100.4 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.10.100.4
      10.0.0.0/8 is variably subnetted, 14 subnets, 4 masks
C        10.10.10.0/24 is directly connected, Vlan10
C        10.10.20.0/24 is directly connected, Vlan20
C        10.10.30.0/24 is directly connected, Vlan30
C        10.10.31.0/24 is directly connected, Vlan31
C        10.10.32.0/24 is directly connected, Vlan32
C        10.10.33.0/24 is directly connected, Vlan33
C        10.10.40.0/23 is directly connected, Vlan40
C        10.10.50.0/23 is directly connected, Vlan50
C        10.10.70.0/24 is directly connected, Vlan70
C        10.10.80.0/24 is directly connected, Vlan80
C        10.10.90.0/24 is directly connected, Vlan90
C        10.10.100.0/29 is directly connected, Vlan100
C        10.10.255.1/32 is directly connected, Loopback0
```

### 5.2 Verify OSPF Neighbor Adjacency with Firewall
```text
Core-SW-01# show ip ospf neighbor
```
**Expected Output:**
```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.255.4       1   FULL/BDR        00:00:33    10.10.100.4     Vlan100
```

---

## 6. Access Layer Security Verification

### 6.1 Port Security Status
```text
Acc-SW-F1# show port-security
```
**Expected Output:**
```text
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
---------------------------------------------------------------------------
Gi1/0/1                   2            2                  0         Restrict
Gi1/0/2                   2            2                  0         Restrict
Gi1/0/25                  1            1                  0         Shutdown
Gi1/0/33                  1            1                  0         Shutdown
Gi1/0/37                  1            1                  0         Restrict
---------------------------------------------------------------------------
Total Addresses in System (excluding duplicates) : 7
```

### 6.2 View Sticky MAC Addresses
```text
Acc-SW-F1# show port-security address
```
**Expected Output:**
```text
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
31      0050.56a1.1001    SecureConfigured (dynamic)    Gi1/0/1      -
40      0004.f211.2201    SecureConfigured (dynamic)    Gi1/0/1      -
70      a414.37b2.9901    SecureSticky                  Gi1/0/25     -
80      c038.9610.4401    SecureSticky                  Gi1/0/33     -
90      001e.8f33.7701    SecureSticky                  Gi1/0/37     -
-----------------------------------------------------------------------------
```

### 6.3 DHCP Snooping Binding Database
```text
Acc-SW-F1# show ip dhcp snooping binding
```
**Expected Output:**
```text
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:50:56:A1:10:01   10.10.31.105     691200      dhcp-snooping  31    GigabitEthernet1/0/1
00:04:F2:11:22:01   10.10.40.105     2592000     dhcp-snooping  40    GigabitEthernet1/0/1
```

### 6.4 Dynamic ARP Inspection (DAI) Statistics
```text
Acc-SW-F1# show ip arp inspection statistics
```
**Expected Output:**
```text
 Vlan      Permitted    Dropped    DHCP Drops    ACL Drops
 ----      ---------    -------    ----------    ---------
   10              0          0             0            0
   31            854          0             0            0
   40           1240          0             0            0
   70            312          0             0            0
```

### 6.5 Storm Control Suppression
```text
Acc-SW-F1# show storm-control broadcast
```
**Expected Output:**
```text
Interface  Filter State   Upper        Lower        Current    Action
---------  -------------  -----------  -----------  ---------  ------
Gi1/0/1    Forwarding       5.00%        5.00%        0.00%    Trap
Gi1/0/25   Forwarding       3.00%        3.00%        0.00%    Trap
Gi1/0/39   Forwarding      10.00%       10.00%        0.00%    Trap
```

### 6.6 Recovery from BPDU Guard / Port Security Err-Disable
If a port is disabled due to an accidental loop or security violation:
```text
Acc-SW-F1# show interfaces status err-disabled
```
To restore the interface:
```text
Acc-SW-F1# configure terminal
Acc-SW-F1(config)# interface GigabitEthernet1/0/25
Acc-SW-F1(config-if)# shutdown
Acc-SW-F1(config-if)# no shutdown
Acc-SW-F1(config-if)# end
```

---

## 7. Power over Ethernet (PoE) Verification

```text
Acc-SW-F1# show power inline
```
**Expected Output:**
```text
Available: 740.0(w)  Used: 498.0(w)  Remaining: 242.0(w)

Interface Admin  Oper       Power   Device              Class Max
                            (Watts)
--------- ------ ---------- ------- ------------------- ----- ----
Gi1/0/1   auto   on         5.5     IP Phone 8845       2     15.4
Gi1/0/2   auto   on         5.5     IP Phone 8845       2     15.4
Gi1/0/25  auto   on         8.5     IP Camera Dome      3     15.4
Gi1/0/33  auto   on         10.0    Door Controller     3     15.4
Gi1/0/39  auto   on         20.0    Catalyst 9120 AP    4     30.0
```

---

## 8. Management, Time & Logging Services Verification

### 8.1 SSH Status
```text
Core-SW-01# show ip ssh
```
**Expected Output:**
```text
SSH Enabled - version 2.0
Authentication timeout: 60 secs; Authentication retries: 3
```

### 8.2 NTP Synchronization
```text
Core-SW-01# show ntp status
```
**Expected Output:**
```text
Clock is synchronized, stratum 2, reference is 10.10.20.10
nominal freq is 250.0000 Hz, actual freq is 249.9998 Hz, precision is 2**18
reference time is E72F41A0.3129841B (17:28:32.194 UTC Mon Nov 16 2026)
clock offset is 0.812 msec, root delay is 1.45 msec
```

### 8.3 Syslog Message Buffer
```text
Core-SW-01# show logging | include %HSRP
```
**Expected Output:**
```text
%HSRP-5-STATECHANGE: Vlan31 Grp 31 state Standby -> Active
%HSRP-5-STATECHANGE: Vlan40 Grp 40 state Standby -> Active
```

