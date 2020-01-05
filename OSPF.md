# OSPF (Open Shortest Path First)

- [OSPF (Open Shortest Path First)](#ospf-open-shortest-path-first)
  - [Distance Vector vs. Link State](#distance-vector-vs-link-state)
  - [History](#history)
  - [Features](#features)
  - [OSPF 3 tables](#ospf-3-tables)
  - [OSPF Area](#ospf-area)
  - [OSPF 支援的網路架構](#ospf-%e6%94%af%e6%8f%b4%e7%9a%84%e7%b6%b2%e8%b7%af%e6%9e%b6%e6%a7%8b)
  - [DR, BDR, DR Other](#dr-bdr-dr-other)
  - [OSPF 與其他路由協議的 AD 值](#ospf-%e8%88%87%e5%85%b6%e4%bb%96%e8%b7%af%e7%94%b1%e5%8d%94%e8%ad%b0%e7%9a%84-ad-%e5%80%bc)
  - [三種方式選擇 Router ID（優先權為由上至下）](#%e4%b8%89%e7%a8%ae%e6%96%b9%e5%bc%8f%e9%81%b8%e6%93%87-router-id%e5%84%aa%e5%85%88%e6%ac%8a%e7%82%ba%e7%94%b1%e4%b8%8a%e8%87%b3%e4%b8%8b)
  - [OSPF Packet Type](#ospf-packet-type)
    - [LSA Types](#lsa-types)
  - [Metrics](#metrics)
  - [實作](#%e5%af%a6%e4%bd%9c)
    - [實驗 2](#%e5%af%a6%e9%a9%97-2)
    - [實驗 3](#%e5%af%a6%e9%a9%97-3)
    - [實驗 4](#%e5%af%a6%e9%a9%97-4)
  - [雜記](#%e9%9b%9c%e8%a8%98)

## Distance Vector vs. Link State

1. Distance Vector, DV
   - 兩兩交換路由得到網路資訊
   - 自己只知道鄰居的資訊（旁邊的 node）
   - 收斂時間長
   - 適合中小型網路
2. Link State, LS
   - 透過 flooding 群播所有 node，因此每個節點都有相同的 LSDB
   - 透過 LSDB，建構 Shortest Path Tree, SPT，以得到自己到不同 Node 的最短距離
   - 僅單純轉發原始的 LSA，並不會編輯 LSA 的內容
   - 收斂時間短
   - 因為沒有 hop 數限制，因此適合大型網路

## History

- OSPFv2: IPv4
- OSPFv3: IPv6

## Features

- Link State
- Dynamic Routing
- IGP, Interior Gateway Protocol
- Support VLSM, CIDR
  - VLSM = Variable-Length Subnet Mask（子網路切割）
  - CIDR = Class Inter-Domain Routing（超網化）
- Maintenance Cost: every 30s update once

## OSPF 3 tables

1. Neighbors Table (LSA: Link State Advertisement)
2. LSDB Table (Link State Database)
   - Every node has the same LSDB
3. Routing Table

## OSPF Area

- 骨幹網路：只有一個
  - 裡面的 Router 稱作 Interior Router
  - area 設定為 0
- 非骨幹區：皆會與骨幹網路相鄰
  - ABR: Area Border Router
  - ASBR: Autonomous System Boundary Router

## OSPF 支援的網路架構

1. Point to Point
2. Point to Multi-Point
3. Multi Access (Broadcast), MA
   - 接下來主要會講的內容
4. NBMA(Not Broadcast Multi Access)

## DR, BDR, DR Other

- DR: Designated Router
- BDR: Backup Designated Router
- DR Other

## OSPF 與其他路由協議的 AD 值

1. OSPF 110
2. RIP 120
3. EIGRP 90

## 三種方式選擇 Router ID（優先權為由上至下）

1. Manual
2. Loopback
3. 'up' state 實體介面 IP
   - Physical Layer 的連結可能會斷掉，因此此選項優先順序為最後

## OSPF Packet Type

1. Hello
2. DBD (Database Description)
3. LSR (Link State Request)
4. LSU (Link State Update)
5. LSA (Link State Ack)

### LSA Types

- Type 1 (Router LSA): DR Other -> DR
- Type 2 (Network LSA): DR -> DR Other
- Type 3 (ABR LSA)
- Type 4, 5, 7: ...

## Metrics

1. OSPF: only bandwidth
2. RIP: hop
3. EIGRP: bandwidth, load, reliability, delay, MTU

## 實作

> ref: [Jan Ho 的網絡世界 - Open Shortest Path First (OSPF) 開放最短路由優先協定](https://www.jannet.hk/zh-Hant/post/open-shortest-path-first-ospf/)

```
# R1
enable
configure terminal
hostname R1
intreface e0/0
ip addr 12.1.1.1 255.255.255.0
no shutdown
```

```
# R2
enable
configure terminal
hostname R2
intreface e0/0
ip addr 12.1.1.2 255.255.255.0
no shutdown
interface e0/1
ip addr 23.1.1.2 255.255.255.0
no shutdown
```

```
# R3
enable
configure terminal
hostname R3
intreface e0/0
ip addr 23.1.1.3 255.255.255.0
no shutdown
```

```
# R1
router ospf 1
network 12.1.1.0 0.0.0.255 area 0
```

```
# R2
router ospf 1
network 12.1.1.0 0.0.0.255 area 0
network 23.1.1.0 0.0.0.255 area 0
```

```
# R3
router ospf 1
network 23.1.1.0 0.0.0.255 area 0
```

```
# R1
ping 12.1.1.2 # 此時應該會 ping 通
router ospf 1
R1(config)#do show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
23.1.1.2          1   FULL/DR         00:00:38    12.1.1.2        Ethernet0/0
```

```
# 查看各種狀態
show ip ospf neighbor
show ip ospf interface
show ip ospf database
show interfaces e0/0
show ip route
```

### 實驗 2

```
# R3
no router ospf 1
```

```
# R1
no router ospf 1
router ospf 1
router-id 1.1.1.1
network 12.1.1.0 0.0.0.255 a 0 # a for area
```

```
# R2
no router ospf 1
router ospf 1
network 12.1.1.0 0.0.0.255 a 0
network 23.1.1.0 0.0.0.255 a 0
```

```
# R1
do show ip ospf neighbor
```

```
# R2
show ip ospf neightbor
```

### 實驗 3

```
# R1
no router ospf 1
int lo0
ip addr 1.1.1.1 255.255.255.0 # 邏輯接口不需 no shutdown
do show ip interface
```

```
# R2
no router ospf 1
int lo0
ip addr 2.2.2.2 255.255.255.0
```

```
# R3
no router ospf 1
int lo0
ip addr 3.3.3.3 255.255.255.0
int e0/1
ip addr 24.1.1.3 255.255.255.0
no shutdown
```

```
# R4
enable
configure terminal
int lo0
ip addr 4.4.4.4 255.255.255.0
int e0/0
ip addr 34.1.1.4 255.255.255.0
no shutdown
```

```
# R1
router ospf 1
network 12.1.1.0 0.0.0.255 a 1
network 1.1.1.0 0.0.0.255 a 1
```

```
# R2
router ospf 1
network 12.1.1.0 0.0.0.255 a 1
network 23.1.1.0 0.0.0.255 a 0
network 2.2.2.0 0.0.0.255 a 0
```

```
# R3
router ospf 1
network 23.1.1.0 0.0.0.255 a 0
network 3.3.3.0 0.0.0.255 a 0
network 34.1.1.0 0.0.0.255 a 2
```

```
# R4
router ospf 1
network 34.1.1.0 0.0.0.255 a 0
network 4.4.4.0 0.0.0.255 a 2
```

```
# R1
show ip route
ping 4.4.4.4 source 1.1.1.1
```

### 實驗 4

```
# R1
router ospf 1
network 172.16.8.0 0.0.0.255 a 1
network 172.16.9.0 0.0.0.255 a 1
network 172.16.10.0 0.0.0.255 a 1
network 172.16.11.0 0.0.0.255 a 1
```

```
# R2
router ospf 1
network 172.16.8.0 0.0.0.255 a 1
network 172.16.9.0 0.0.0.255 a 1
network 172.16.10.0 0.0.0.255 a 1
network 172.16.11.0 0.0.0.255 a 1
area 1 range 172.16.8.0 255.255.252.0 # 需在 ABR 上操作（R2 屬於 ABR），因此不能在 R1 上操作
```

```
# R3
do clear ip route *
do show ip route # 應可看到路由條目合併
```

## 雜記

```
show run | sec ospf # 查看 ospf 區塊
do clear ip route # 清除路由表
```