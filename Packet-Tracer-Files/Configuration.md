# 🔥 __PROJECT CONFIGURATION__

## 🎯 1. IP Addressing Plan

* Assign proper subnets (example):

  * VLAN 10 (PC0, PC1) — 10.10.10.0/24
  * VLAN 20 (PC2, PC3) — 10.10.20.0/24
  * VLAN 30 (PC4, PC5) — 10.10.30.0/24
  * VLAN 40 (PC6, PC7) — 10.10.40.0/24

| Device  | Interface | IP Address  | Purpose         |
| :------ | :-------- | :---------- | :-------------- |
| Router8 | Fa0/0     | 192.168.1.1 | Link to Switch1 |
| Router8 | Fa1/0     | 192.168.2.1 | Link to Switch2 |
| Router9 | Fa0/0     | 192.168.3.1 | Link to Switch3 |
| Router9 | Fa1/0     | 192.168.4.1 | Link to Switch0 |

🧠 Each router interfaces into different switches.

---

## 🎯 2. Basic Router Configuration

On **Router8**:

```bash
enable
configure terminal

hostname Router8

interface FastEthernet0/0
ip address 192.168.1.1 255.255.255.0
no shutdown

interface FastEthernet1/0
ip address 192.168.2.1 255.255.255.0
no shutdown
```

On **Router9**:

```bash
enable
configure terminal

hostname Router9

interface FastEthernet0/0
ip address 192.168.3.1 255.255.255.0
no shutdown

interface FastEthernet1/0
ip address 192.168.4.1 255.255.255.0
no shutdown
```

---

## 🎯 3. Switch VLAN Configuration

On all Switches (example for Switch1):

```bash
enable
configure terminal

vlan 10
name VLAN10
exit

interface range fa2/1, fa3/1
switchport mode access
switchport access vlan 10

interface fa0/1
switchport mode trunk

interface fa1/1
switchport mode trunk
```

Repeat similar for other switches adjusting VLANs accordingly!

---

## 🎯 4. Configure STP (Spanning Tree Protocol)

✅ Make one switch (say, Switch1) **Root Bridge**:

On **Switch1**:

```bash
enable
configure terminal

spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
spanning-tree vlan 30 priority 4096
spanning-tree vlan 40 priority 4096
```

**Other switches will have default priorities (32768)** and STP will automatically **block redundant links**.

---

## 🎯 5. Configure MHSRP (Multigroup HSRP)

Now, **MHSRP Configuration**:

On **Router8**:

```bash
interface FastEthernet0/0
standby 10 ip 10.10.10.254
standby 10 priority 110
standby 10 preempt

interface FastEthernet1/0
standby 20 ip 10.10.20.254
standby 20 priority 90
standby 20 preempt
```

On **Router9**:

```bash
interface FastEthernet0/0
standby 10 ip 10.10.10.254
standby 10 priority 90
standby 10 preempt

interface FastEthernet1/0
standby 20 ip 10.10.20.254
standby 20 priority 110
standby 20 preempt
```

| VLAN    | Virtual IP   | Active Router | Standby Router |
| :------ | :----------- | :------------ | :------------- |
| VLAN 10 | 10.10.10.254 | Router8       | Router9        |
| VLAN 20 | 10.10.20.254 | Router9       | Router8        |

✅ So, each router is *Active* for one VLAN and *Standby* for another — **Load Sharing** + **Failover**!

---

## 🎯 6. PC Configuration

Each PC:

* IP address from its VLAN range.
* Default Gateway: **Virtual IP** (for example 10.10.10.254 for VLAN 10).

Example (PC0):

* IP Address: 10.10.10.10
* Subnet: 255.255.255.0
* Default Gateway: 10.10.10.254

---

# 🚀 FINAL SUMMARY

✅ **Routers**: MHSRP configured for redundancy and load sharing.
✅ **Switches**: VLANs created, STP manages redundant links.
✅ **PCs**: Properly assigned to VLANs, pointing to Virtual IPs.
✅ **Redundancy**: If any router or link goes down, PCs will automatically failover to backup path.
✅ **Network**: Stable, Loop-Free, High Availability!

---

# 📸 The Network Will Behave Like:

* Switches manage backup links via STP.
* Routers manage gateway redundancy via MHSRP.
* PCs automatically keep working even if Router8 or Router9 crashes!

---
