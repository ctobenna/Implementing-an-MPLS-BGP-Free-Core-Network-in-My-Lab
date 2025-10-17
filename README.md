# 🧪 MPLS BGP-Free Core Lab
---
### 🧠 Lab Overview

In this lab, I implemented a MPLS BGP-Free Core topology to demonstrate how service providers forward customer traffic efficiently without running BGP in the core routers.

The Provider Edge (PE) routers maintain BGP sessions with customer routers, while the core (P) routers use OSPF for routing and LDP for label distribution. This allows traffic to be label-switched across the core using MPLS, keeping the backbone BGP-free and highly scalable.

<p align=center> <img src="https://imgur.com/k3LVsOY.png" height="80%" width="80%" alt="MPLS BGP-Free Core Topology"> </p>

---

## 🔍 Key Lab Concepts

### 🧩 Why “BGP-Free Core”?

In traditional service provider networks, all routers—including the core—must run BGP to exchange customer routes.
This can quickly lead to scalability issues due to large BGP tables and high CPU utilization.

The MPLS BGP-Free Core solves this by separating control and forwarding planes:

BGP runs only on PE routers, which connect to customers.

MPLS labels carry packets across the core (P routers), which forward traffic based on labels, not IP lookups.

The P routers use:

OSPF to exchange internal routes (loopbacks, interfaces).

LDP to assign labels to those routes.
Thus, the P routers never need to learn customer BGP routes — they only forward based on MPLS labels.

### ⚙️ Lab Configuration Summary

Device	Role	IP/Loopback	AS	Key Protocols
* R1	CE1	1.1.1.1/32	100	eBGP to PE1
* R2	PE1	2.2.2.2/32	200	iBGP, MPLS, OSPF, LDP
* R3	P2	3.3.3.3/32	200	OSPF, LDP
* R4	P1	4.4.4.4/32	200	OSPF, LDP
* R5	PE2	5.5.5.5/32	200	iBGP, MPLS, OSPF, LDP
* R6	CE2	6.6.6.6/32	300	eBGP to PE2
  
### 🔗 Connectivity Overview

PE1 ↔ CE1: eBGP (AS 100 ↔ AS 200)

PE2 ↔ CE2: eBGP (AS 200 ↔ AS 300)

PE1 ↔ P2 ↔ P1 ↔ PE2: OSPF + LDP + MPLS core

PE1 ↔ PE2: iBGP using loopbacks (2.2.2.2 ↔ 5.5.5.5)


### 🧠 Logical Flow of Traffic

CE1 advertises its local route (1.1.1.1/32) to PE1 via eBGP.

PE1 advertises this route to PE2 using iBGP, with a BGP label.

PE1 also assigns an LDP label for the path through the core to PE2’s loopback.

P routers (R3, R4) only perform label swapping, not IP lookups.

PE2 receives the labeled packet, pops the labels, and forwards it to CE2.

The result: BGP routes are exchanged only between PE routers, but packets are label-switched across the core, keeping it BGP-free.

---

### ✅ Final Outcome

<p align=center> <img src="https://i.imgur.com/Cui0MKP.png" height="70%" width="70%" alt="PE1 BGP Table"> </p>

* CE1 (AS 100) successfully pings CE2 (AS 300) through the MPLS BGP-free core.

* Core routers (R3 & R4) never learn customer routes — they only forward based on MPLS labels.

* End-to-end connectivity achieved using a scalable, efficient provider backbone.

** 🧭 BGP Route Table – PE1 (R2)
<p align=center> <img src="https://imgur.com/AFBQfsb.png" height="70%" width="70%" alt="PE1 BGP Table"> </p>
** 🧭 BGP Route Table – PE2 (R5)
<p align=center> <img src="https://imgur.com/Gq67dLE.png" height="70%" width="70%" alt="PE2 BGP Table"> </p>
** 🔖 MPLS Label Table – P Router (R3)
<p align=center> <img src="https://imgur.com/s54uMmE.png" height="70%" width="70%" alt="MPLS Labels"> </p>

---
### 💡 Lessons Learned

* BGP-Free Core reduces routing overhead in large-scale networks.

* MPLS labels replace IP lookups, improving forwarding speed and scalability.

* PE routers handle all BGP sessions and route advertisements.

* OSPF + LDP is sufficient for internal label switching within the core.

* The architecture forms the foundation for MPLS Layer 3 VPNs and Traffic Engineering.

## ⚙️ Configuration Summary

Below is a simplified sample (partial) configuration outline used in this lab:

### 🧩 PE1 (R2)
            hostname PE1
            interface Loopback0
             ip address 2.2.2.2 255.255.255.255
            !
            interface Ethernet0/0
             ip address 192.168.12.2 255.255.255.0
             mpls ip
            !
            interface Ethernet0/1
             ip address 192.168.23.2 255.255.255.0
             mpls ip
            !
            router ospf 10
             network 2.2.2.2 0.0.0.0 area 0
             network 192.168.23.0 0.0.0.255 area 0
            !
            mpls ldp router-id Loopback0 force
            !
            router bgp 200
             bgp log-neighbor-changes
             neighbor 5.5.5.5 remote-as 200
             neighbor 5.5.5.5 update-source Loopback0
              neighbor 5.5.5.5 next-hop-self
              neighbor 192.168.12.1 remote-as 100
            !

### 🧩 Core Router P2 (R3)
            hostname P2
            interface Loopback0
             ip address 3.3.3.3 255.255.255.255
            !
            interface Ethernet0/0
             ip address 192.168.34.3 255.255.255.0
             mpls ip
            !
            interface Ethernet0/1
             ip address 192.168.23.3 255.255.255.0
             mpls ip
            !
            router ospf 10
             network 3.3.3.3 0.0.0.0 area 0
             network 192.168.23.0 0.0.0.255 area 0
             network 192.168.34.0 0.0.0.255 area 0
            !

### 🧩 PE2 (R5)
            hostname PE2
            interface Loopback0
             ip address 5.5.5.5 255.255.255.255
            !
            interface Ethernet0/0
             ip address 192.168.56.5 255.255.255.0
             mpls ip
            !
            interface Ethernet0/1
             ip address 192.168.45.5 255.255.255.0
             mpls ip
            !
            router ospf 10
             network 5.5.5.5 0.0.0.0 area 0
             network 192.168.45.0 0.0.0.255 area 0
            !

            router bgp 200
             bgp log-neighbor-changes
             neighbor 2.2.2.2 remote-as 200
             neighbor 2.2.2.2 update-source Loopback0
              neighbor 2.2.2.2 next-hop-self
              neighbor 192.168.56.6 remote-as 300
            !

### 🧩 CE Routers (R1 and R6)
            ! CE1 (AS 100)
            router bgp 100
             neighbor 192.168.12.2 remote-as 200
             network 1.1.1.1 mask 255.255.255.255
            
            ! CE2 (AS 300)
            router bgp 300
             neighbor 192.168.56.5 remote-as 200
             network 6.6.6.6 mask 255.255.255.255

## 🎯 Final Verification

✅ End-to-end ping successful: CE1 ➜ CE2 using MPLS Labeled Path

✅ Core routers (R3, R4) remain BGP-free

✅ PE routers exchange routes via iBGP

✅ Labels distributed correctly through LDP

## 🧩 Conclusion

This lab demonstrates how service providers build scalable and efficient MPLS backbones by separating the control plane (BGP) from the data plane (MPLS).
The BGP-Free Core design keeps routing simple, while MPLS ensures fast, label-based packet forwarding — forming the backbone of modern ISP and enterprise WAN architectures.
