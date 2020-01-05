# Computer Network

## DISCLAIMER

- Not all the docs are translated into English, please be patient for the translated version.😓

## Outline

- [Simulator vs. Emulator](simulatorEmulator.md)
  - [Cisco Router](ciscoRouter.md)
  - [Eve-NG: Emulated virtual environment - Next Generation](eveng.md)
- [Basic Concept](basicConcept.md)
- Layer 2 Technique
  - [VLAN: Virtual LAN](VLAN.md)
  - [DTP: Dynamic Trunking Protocol](DTP.md)
  - [VTP: VLAN Trunking Protocol](VTP.md)
  - [STP: Spanning Tree Protocol](STP.md)
  - [EtherChannel](etherChannel.md)
  <!-- - [CDP: Cisco Discovery Protocol](CDP.md) -->
- Layer 3 Technique
  - [Routing Experiment](routingExperiment.md)
  - [Routing Protocol](routingProtocol.md)
    <!-- - [RIP: Routing Information Protocol](RIP.md) -->
    - [EIGRP: Enhanced Interior Gatewaty Protocol](EIGRP.md)
    - [OSPF: Open Shortest Path First](OSPF.md)
  - [Route Redistribute](routeRedistribute.md)
  <!-- - BGP: Border Gateway Protocol -->
- Security
  - [VPN (GRE, IPSec)](VPN.md)
  <!-- - [GRE over IPSec vs. IPSec over GRE](greIpsec.md) -->
  <!-- - Firewall -->
- Network Service
  - [DHCP: Dynamic Host Configuration Protocol](DHCP.md)
  <!-- - NAT: Network Address Translation -->
- Other
  <!-- - Terminology
  - Command Cheat Sheet -->
  - [Network Architecture](networkArchitecture.md)
- Experiment
  - [比較不同的路由協定間的收斂時間](比較不同的路由協定間的收斂時間.md)
  - [Per VLAN Spanning Tree + (PVST+)](PVST+.md)

## Intro.

- The course focus on practicing, operating cisco equipment.
- Having full cisco equipment is expensive, so we'll using the tools below to substitute:
  1. Cisco Packet Tracer (Simulator)
  2. Eve-NG (Emulator)
- It also has to take notes in the class, sharing on the Internet is better.
- After learning this course, you'll be able to pass CCNA(Associate), CCEP(Professional) certification.
  - ref: [Cisco國際認證考試-證照簡介](https://www.pcschool.com.tw/campus/cert/certLink/9/index.htm)

<!-- ## Sort by Week

- W1
  - Intro
  - Cisco Router
  - Eve-NG
- W2
  - set up password(plain text, cipher)
  - Upload image to Eve-NG
- W3
  - Upload image to Eve-NG
  - configure ssh server on Tiny Core
  - Data Center Multi-Tier Model Topology
- W4
  - 內定路由
  - 浮動路由
  - TODO: 未完成
- W5
  - Routing Protocol
  - Static Routing & Dynamic Routing
  - RIP
- W6
  - EIGRP
- W7
  - IGP, EGP
  - ECMP(等價路由)
  - VPN
    - GRE
- W8
  - IPsec, GRE over IPsec, IPsec over GRE
- W9
  - OSPF
- W10
  - 期中考休息
- W11
  - Route Redistribution
  - VLAN
- W12
  - Network Architecture
  - VLAN 實驗：未命名
  - VLAN 實驗：將兩個 vLAN 通道整合成一個 Trunk
  - DTP
  - VTP
    - VTP Pruning
- W13
  - VLAN 實作1, 2
  - VTP 實作
  - VTP Pruning (uncompleted)
  - STP + STP 實作 (uncompleted)
  - TODO: 未完成
- W14
  - 比較不同的路由協定間的收斂時間
    - tag: rouing protocol
  - STP
- W15
  - VLAN 間通訊方式
  - DHCP
- W16
  - DHCP (cont.)
  - Firewall (uncertain)
  - ACL -->