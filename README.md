# 🌐 Cisco Lab: RIP V2 Multi-Branch Enterprise Network — Project 116

> **Tools:** Cisco Packet Tracer · IOS 12.2 / 15.1 · RIP Version 2 · Router-on-a-Stick · 9 VLANs · DHCP Relay · SSH · Telnet · HTTP Server · Port Security (Sticky MAC)

---

## 📌 Project Overview

This is a **large-scale enterprise simulation** built entirely in Cisco Packet Tracer — featuring **3 branch offices**, a **centralized ISP router**, **dual servers** (DHCP + HTTP), **9 VLANs**, and **RIP Version 2** as the dynamic routing protocol connecting everything together.

Every branch runs its own Router-on-a-Stick, serves multiple VLANs via a centralized DHCP server, and is reachable from any other branch through RIP-learned routes. Remote management is secured via **SSH** on branch routers and **Telnet** on the ISP. Port security protects every access port on the switches.

This lab covers the full stack — from physical topology design down to Layer 2 security and Layer 7 HTTP service verification.

---

## 🗺️ Network Topology Design

```
                        ┌─────────────┐
                        │  ISP Router │  (Router-on-a-Stick)
                        │  10.1.1.1   │
                        │  20.1.1.1   │
                        └──┬──┬──┬───┘
                  Serial   │  │  │  Serial
              ┌────────────┘  │  └────────────┐
              │   1.1.1.0/24  │  3.1.1.0/24   │
              │               │ 2.1.1.0/24    │
         ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
         │BRANCH-1 │     │BRANCH-2 │     │BRANCH-3 │
         │Router   │     │Router   │     │Router   │
         └────┬────┘     └────┬────┘     └────┬────┘
        ROASM │         ROASM │         ROASM │
         ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
         │Switch0  │     │Switch2  │     │Switch1  │
         └─┬──┬──┬─┘     └─┬──┬──┬─┘     └─┬──┬──┬─┘
           │  │  │          │  │  │          │  │  │
         PC0 PC1 PC2      PC3 PC4 PC5      PC6 PC7 PC8
        V10 V20 V30      V40 V50 V60      V70 V80 V90

┌──────────────────────┬──────────────────────────────────────┐
│  DHCP-SERVER         │  HTTP-SERVER                         │
│  10.1.1.2/24         │  20.1.1.2/24                         │
│  Serves VLAN 10–90   │  Hosts company web pages             │
└──────────────────────┴──────────────────────────────────────┘
```

### IP Addressing Table

| Device      | Interface         | IP Address         | Purpose             |
|-------------|-------------------|--------------------|---------------------|
| ISP         | Serial 2/0        | 1.1.1.2/24         | Link to BRANCH-1    |
| ISP         | Serial 3/0        | 2.1.1.2/24         | Link to BRANCH-2    |
| ISP         | Serial 4/0        | 3.1.1.2/24         | Link to BRANCH-3    |
| ISP         | FastEthernet 0/0  | 10.1.1.1/24        | DHCP Server segment |
| ISP         | FastEthernet 1/0  | 20.1.1.1/24        | HTTP Server segment |
| BRANCH-1    | Serial 2/0        | 1.1.1.1/24         | WAN to ISP          |
| BRANCH-1    | Fa0/0.10          | 192.168.1.1/24     | VLAN 10 gateway     |
| BRANCH-1    | Fa0/0.20          | 192.168.2.1/24     | VLAN 20 gateway     |
| BRANCH-1    | Fa0/0.30          | 192.168.3.1/24     | VLAN 30 gateway     |
| BRANCH-2    | Serial 3/0        | 2.1.1.1/24         | WAN to ISP          |
| BRANCH-2    | Fa0/0.40          | 192.168.4.1/24     | VLAN 40 gateway     |
| BRANCH-2    | Fa0/0.50          | 192.168.5.1/24     | VLAN 50 gateway     |
| BRANCH-2    | Fa0/0.60          | 192.168.6.1/24     | VLAN 60 gateway     |
| BRANCH-3    | Serial 2/0        | 3.1.1.1/24         | WAN to ISP          |
| BRANCH-3    | Fa0/0.70          | 192.168.7.1/24     | VLAN 70 gateway     |
| BRANCH-3    | Fa0/0.80          | 192.168.8.1/24     | VLAN 80 gateway     |
| DHCP Server | FastEthernet 0    | 10.1.1.2/24        | Centralized DHCP    |
| HTTP Server | FastEthernet 0    | 20.1.1.2/24        | Web server          |

---

## ⚙️ Full Configuration

### 🔷 ISP Router — Hub of the Network

```cisco
Router(config)#hostname ISP

! Serial links to all three branches
ISP(config)#interface gigabitEthernet 0/1
ISP(config-if)#no ip address
ISP(config-if)#no shutdown

! Interface toward DHCP Server
ISP(config)#interface fastEthernet 0/0
ISP(config-if)#ip address 10.1.1.1 255.255.255.0
ISP(config-if)#no shutdown

! Interface toward HTTP Server
ISP(config)#interface fastEthernet 1/0
ISP(config-if)#ip address 20.1.1.1 255.255.255.0
ISP(config-if)#no shutdown
```

---

### 🔴 RIP Version 2 — Dynamic Routing Protocol

> **Why RIP V2?** Classless routing, subnet mask support, no auto-summary — perfect for a multi-branch topology with /24 subnets across serial WAN links.

#### ISP Router — RIP Configuration

```cisco
ISP(config)#router rip
ISP(config-router)#version 2
ISP(config-router)#no auto-summary
ISP(config-router)#network 1.1.1.0
ISP(config-router)#network 2.1.1.0
ISP(config-router)#network 3.1.1.0
ISP(config-router)#network 10.1.1.0
ISP(config-router)#network 20.1.1.0
```

#### BRANCH-1 — RIP Configuration

```cisco
BRANCH-1(config)#router rip
BRANCH-1(config-router)#version 2
BRANCH-1(config-router)#no auto-summary
BRANCH-1(config-router)#network 192.168.1.0
BRANCH-1(config-router)#network 192.168.2.0
BRANCH-1(config-router)#network 192.168.3.0
BRANCH-1(config-router)#network 1.1.1.0
```

#### BRANCH-2 — RIP Configuration

```cisco
BRANCH-2(config)#router rip
BRANCH-2(config-router)#version 2
BRANCH-2(config-router)#no auto-summary
BRANCH-2(config-router)#network 192.168.4.0
BRANCH-2(config-router)#network 192.168.5.0
BRANCH-2(config-router)#network 192.168.6.0
BRANCH-2(config-router)#network 2.1.1.0
```

#### BRANCH-3 — RIP Configuration

```cisco
BRANCH-3(config)#router rip
BRANCH-3(config-router)#version 2
BRANCH-3(config-router)#no auto-summary
BRANCH-3(config-router)#network 192.168.7.0
BRANCH-3(config-router)#network 192.168.8.0
BRANCH-3(config-router)#network 3.1.1.0
```

---

### 📊 RIP Routing Tables — Verified

#### BRANCH-1 `show ip route` (key entries)

```
C    1.1.1.0 is directly connected, Serial2/0
R    2.1.1.0 [120/1] via 1.1.1.2, 00:00:24, Serial2/0   ← learned via RIP
R    3.1.1.0 [120/1] via 1.1.1.2, 00:00:24, Serial2/0   ← learned via RIP
R    10.1.1.0 [120/1] via 1.1.1.2, 00:00:24, Serial2/0  ← DHCP server reachable
R    20.1.1.0 [120/1] via 1.1.1.2, 00:00:24, Serial2/0  ← HTTP server reachable
C    192.168.1.0 directly connected, FastEthernet0/0.10
C    192.168.2.0 directly connected, FastEthernet0/0.20
C    192.168.3.0 directly connected, FastEthernet0/0.30
R    192.168.4.0 [120/2] via 1.1.1.2, Serial2/0         ← BRANCH-2 VLANs learned
R    192.168.5.0 [120/2] via 1.1.1.2, Serial2/0
R    192.168.6.0 [120/2] via 1.1.1.2, Serial2/0
R    192.168.7.0 [120/2] via 1.1.1.2, Serial2/0         ← BRANCH-3 VLANs learned
R    192.168.8.0 [120/2] via 1.1.1.2, Serial2/0
```

> **Metric [120/2]** = AD 120 (RIP), hop count 2 (via ISP then to other branch)

#### ISP `show ip rip database` (selected)

```
1.1.1.0/24     directly connected, Serial2/0
2.1.1.0/24     directly connected, Serial3/0
3.1.1.0/24     directly connected, Serial4/0
192.168.1.0/24 [1] via 1.1.1.1, Serial2/0   ← BRANCH-1 VLANs
192.168.4.0/24 [1] via 2.1.1.1, Serial3/0   ← BRANCH-2 VLANs
192.168.7.0/24 [1] via 3.1.1.1, Serial4/0   ← BRANCH-3 VLANs
```

---

### 🔷 BRANCH-1 — Router-on-a-Stick (3 VLANs)

```cisco
BRANCH-1(config)#interface fastEthernet 0/0
BRANCH-1(config-if)#no ip address
BRANCH-1(config-if)#no shutdown

! VLAN 10 — subinterface
BRANCH-1(config)#interface fastEthernet 0/0.10
BRANCH-1(config-subif)#encapsulation dot1Q 10
BRANCH-1(config-subif)#ip address 192.168.1.1 255.255.255.0
BRANCH-1(config-subif)#ip helper-address 10.1.1.2

! VLAN 20 — subinterface
BRANCH-1(config)#interface fastEthernet 0/0.20
BRANCH-1(config-subif)#encapsulation dot1Q 20
BRANCH-1(config-subif)#ip address 192.168.2.1 255.255.255.0
BRANCH-1(config-subif)#ip helper-address 10.1.1.2

! VLAN 30 — subinterface
BRANCH-1(config)#interface fastEthernet 0/0.30
BRANCH-1(config-subif)#encapsulation dot1Q 30
BRANCH-1(config-subif)#ip address 192.168.3.1 255.255.255.0
BRANCH-1(config-subif)#ip helper-address 10.1.1.2

! WAN link to ISP
BRANCH-1(config)#interface serial 2/0
BRANCH-1(config-if)#clock rate 64000
BRANCH-1(config-if)#ip address 1.1.1.1 255.255.255.0
```

#### BRANCH-2 — Router-on-a-Stick (3 VLANs)

```cisco
BRANCH-2(config)#interface fastEthernet 0/0.40
BRANCH-2(config-subif)#encapsulation dot1Q 40
BRANCH-2(config-subif)#ip address 192.168.4.1 255.255.255.0
BRANCH-2(config-subif)#ip helper-address 10.1.1.2

BRANCH-2(config)#interface fastEthernet 0/0.50
BRANCH-2(config-subif)#encapsulation dot1Q 50
BRANCH-2(config-subif)#ip address 192.168.5.1 255.255.255.0
BRANCH-2(config-subif)#ip helper-address 10.1.1.2

BRANCH-2(config)#interface fastEthernet 0/0.60
BRANCH-2(config-subif)#encapsulation dot1Q 60
BRANCH-2(config-subif)#ip address 192.168.6.1 255.255.255.0
BRANCH-2(config-subif)#ip helper-address 10.1.1.2

BRANCH-2(config)#interface serial 3/0
BRANCH-2(config-if)#clock rate 64000
BRANCH-2(config-if)#ip address 2.1.1.1 255.255.255.0
```

#### BRANCH-3 — Router-on-a-Stick (2 VLANs)

```cisco
BRANCH-3(config)#interface fastEthernet 0/0.70
BRANCH-3(config-subif)#encapsulation dot1Q 70
BRANCH-3(config-subif)#ip address 192.168.7.1 255.255.255.0
BRANCH-3(config-subif)#ip helper-address 10.1.1.2

BRANCH-3(config)#interface fastEthernet 0/0.80
BRANCH-3(config-subif)#encapsulation dot1Q 80
BRANCH-3(config-subif)#ip address 192.168.8.1 255.255.255.0
BRANCH-3(config-subif)#ip helper-address 10.1.1.2

BRANCH-3(config)#interface serial 2/0
BRANCH-3(config-if)#clock rate 64000
BRANCH-3(config-if)#ip address 3.1.1.1 255.255.255.0
```

---

### 🔐 SSH Remote Access — BRANCH-1 (Secure Management)

> **SSH vs Telnet:** SSH encrypts the session. Telnet sends credentials in plaintext. This lab uses SSH on all branch routers for secure admin access.

```cisco
BRANCH-1(config)#enable secret maaz
BRANCH-1(config)#ip domain-name khan.com
BRANCH-1(config)#crypto key generate rsa
! → Key size: 1024 bits

BRANCH-1(config)#username user1 privilege 15 secret user1
BRANCH-1(config)#line vty 0 1
BRANCH-1(config-line)#login local
BRANCH-1(config-line)#transport input ssh
```

#### ✅ SSH Verification (from PC1 in VLAN 20)

```
C:\>ssh -l user1 192.168.1.1

Password:
BRANCH-1#show running-config
Building configuration...
Current configuration : 1383 bytes
hostname BRANCH-1
...
```

SSH session established — `BRANCH-1#` prompt reached. Running config pulled remotely. ✅

---

### 📡 Telnet — ISP Router Remote Access

```cisco
ISP(config)#enable secret maaz
ISP(config)#ip domain-name khan.com
ISP(config)#crypto key generate rsa
ISP(config)#username user1 privilege 15 secret user1
ISP(config)#username user2 privilege 15 secret user2
ISP(config)#line vty 0 1
ISP(config-line)#login local
ISP(config-line)#transport input telnet
```

---

### 🌐 HTTP Server — Web Services on 20.1.1.2

The HTTP server (20.1.1.2) is reachable from all 9 VLANs across all 3 branches via RIP-learned routes. Any PC in the network can open a browser and access the company web portal.

**Route path from PC0 (VLAN 10) to HTTP Server:**
```
PC0 → Switch0 → BRANCH-1 (Fa0/0.10) → Serial 2/0 → ISP → FastEthernet 1/0 → HTTP Server
```

Packets traverse 4 hops — RIP handles all routing automatically. No static routes needed.

---

### 🛡️ Centralized DHCP Server — All 9 VLANs

The DHCP server at `10.1.1.2` serves IP addresses to all 9 VLANs via `ip helper-address` configured on every branch router subinterface.

| VLAN | Pool Name | Default Gateway | Start IP     | Subnet Mask   |
|------|-----------|-----------------|--------------|---------------|
| 10   | Vlan 10   | 192.168.1.1     | 192.168.1.1  | 255.255.255.0 |
| 20   | Vlan 20   | 192.168.2.1     | 192.168.2.1  | 255.255.255.0 |
| 30   | Vlan 30   | 192.168.3.1     | 192.168.3.1  | 255.255.255.0 |
| 40   | Vlan 40   | 192.168.4.1     | 192.168.4.1  | 255.255.255.0 |
| 50   | Vlan 50   | 192.168.5.1     | 192.168.5.1  | 255.255.255.0 |
| 60   | Vlan 60   | 192.168.6.1     | 192.168.6.1  | 255.255.255.0 |
| 70   | Vlan 70   | 192.168.7.1     | 192.168.7.1  | 255.255.255.0 |
| 80   | Vlan 80   | 192.168.8.1     | 192.168.8.1  | 255.255.255.0 |
| 90   | Vlan 90   | 192.168.9.1     | 192.168.9.1  | 255.255.255.0 |

---

### 🔒 Port Security — All Switches (Sticky MAC + Shutdown)

Applied on all access ports of Switch0 (BRANCH-1), Switch2 (BRANCH-2), and Switch1 (BRANCH-3).

```cisco
! BRANCH-1 Switch0 — VLAN 10, 20, 30 access ports
Switch(config)#vlan 10
Switch(config-vlan)#name 10
Switch(config)#vlan 20
Switch(config-vlan)#name 20
Switch(config)#vlan 30
Switch(config-vlan)#name 30

Switch(config)#interface fastEthernet 0/24
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan 10,20,30

Switch(config)#interface fastEthernet 0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10

Switch(config)#interface fastEthernet 0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20

Switch(config)#interface fastEthernet 0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 30

! Port Security on all access ports
Switch(config)#interface range fastEthernet 0/1-3
Switch(config-if-range)#switchport port-security
Switch(config-if-range)#switchport port-security mac-address sticky
Switch(config-if-range)#switchport port-security maximum 1
Switch(config-if-range)#switchport port-security violation shutdown
```

#### 📊 Port Security Status — Before Traffic

```
Switch#show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
               (Count)        (Count)         (Count)
----------------------------------------------------------------------
    Fa0/1          1             0               0            Shutdown
    Fa0/2          1             0               0            Shutdown
    Fa0/3          1             0               0            Shutdown
```

#### 📊 Port Security Status — After Traffic (MACs Learned)

```
Switch#show port-security address
            Secure Mac Address Table
----------------------------------------------------------------------
Vlan    Mac Address       Type          Ports    Remaining Age (mins)
----    -----------       ----          -----    --------------------
  10    0004.9A1C.5269   SecureSticky   Fa0/1         -
  30    000C.CF42.E91D   SecureSticky   Fa0/3         -
----------------------------------------------------------------------
```

#### 🔍 Detailed Interface Status — Fa0/1

```
Switch#show port-security interface fastEthernet 0/1
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 000C.CF42.E91D:10
Security Violation Count   : 1
```

When an unauthorized MAC tried to connect on Fa0/1 — the port immediately entered **secure-shutdown** state. Security Violation Count = 1. ✅

---

## ✅ End-to-End Verification

### Cross-Branch Pings from PC0 (VLAN 10, BRANCH-1)

```
C:\>ping 20.1.1.2     (HTTP Server across ISP)
Reply from 20.1.1.2: bytes=32 time=10ms TTL=126 ✅

C:\>ping 192.168.6.2  (PC in VLAN 60, BRANCH-2)
Reply from 192.168.6.2: bytes=32 time=28ms TTL=125 ✅

C:\>ping 192.168.8.2  (PC in VLAN 80, BRANCH-3)
Reply from 192.168.8.2: bytes=32 time=14ms TTL=125 ✅
```

All cross-branch traffic routed successfully via RIP V2. 25% packet loss on first ping is normal (ARP resolution delay in simulation).

### SSH from PC1 → BRANCH-1 Router

```
C:\>ssh -l user1 192.168.1.1
Password: ****
BRANCH-1#  ← Secure CLI access confirmed ✅
```

---

## 🧠 Key Concepts Demonstrated

| Feature | Implementation Detail |
|---|---|
| RIP Version 2 | `no auto-summary`, classless, hop-count metric, 30-sec update timer |
| Serial WAN Links | `clock rate 64000` on DCE side, point-to-point between ISP and branches |
| Router-on-a-Stick | dot1Q encapsulation on subinterfaces, 3 VLANs per branch router |
| DHCP Relay | `ip helper-address 10.1.1.2` on every subinterface — one server, 9 VLANs |
| SSH (Secure Shell) | `crypto key generate rsa`, `transport input ssh`, privilege 15 local users |
| Telnet | VTY lines with `login local`, used on ISP for management |
| HTTP Server | Layer 7 reachability across 4 hops verified via browser |
| Port Security | Sticky MAC, max 1, violation shutdown — applied to all branch access ports |
| VLAN Segmentation | 9 VLANs across 3 switches — IT, HR, and departmental isolation |
| RIP Hop Count | [120/1] = 1 hop (direct branch), [120/2] = 2 hops (via ISP to other branch) |

---

## 📁 Repository Structure

```
📦 Project-116-RIPv2-MultiSite/
├── 📄 README.md
├── 📁 screenshots/
│   ├── topology.png
│   ├── branch1-show-ip-route.png
│   ├── branch2-show-ip-route.png
│   ├── branch3-show-ip-route.png
│   ├── branch1-rip-routes-only.png
│   ├── isp-show-ip-route.png
│   ├── isp-rip-database.png
│   ├── dhcp-server-pools.png
│   ├── cross-branch-ping.png
│   ├── ssh-login-branch1.png
│   ├── port-security-before.png
│   └── port-security-after.png
└── 📄 Project116.pkt
```

---

## 👤 Author

**Maaz Khan**
CCNA Certified · Network Engineer · NOC Engineer
📍 Lower Dir, KPK, Pakistan
🔗 [LinkedIn](https://linkedin.com/in/maazkhanms) 

---

> *"The network is the computer." — John Gage, Sun Microsystems*
