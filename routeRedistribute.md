# Route Redistribute

- [Route Redistribute](#route-redistribute)
  - [基礎 IP 設置](#%e5%9f%ba%e7%a4%8e-ip-%e8%a8%ad%e7%bd%ae)
  - [設置 R1, R2 的 EIGRP 協議](#%e8%a8%ad%e7%bd%ae-r1-r2-%e7%9a%84-eigrp-%e5%8d%94%e8%ad%b0)
  - [設置 R1, R3 的 OSPF 協議](#%e8%a8%ad%e7%bd%ae-r1-r3-%e7%9a%84-ospf-%e5%8d%94%e8%ad%b0)
  - [設置 R1, R4 的 RIP 協議](#%e8%a8%ad%e7%bd%ae-r1-r4-%e7%9a%84-rip-%e5%8d%94%e8%ad%b0)
  - [不同的分配方式](#%e4%b8%8d%e5%90%8c%e7%9a%84%e5%88%86%e9%85%8d%e6%96%b9%e5%bc%8f)
    - [直連](#%e7%9b%b4%e9%80%a3)
    - [靜態路由](#%e9%9d%9c%e6%85%8b%e8%b7%af%e7%94%b1)
      - [R1, R4 RIP](#r1-r4-rip)
      - [R1, R2 EIGRP](#r1-r2-eigrp)
      - [R1, R3 OSPF](#r1-r3-ospf)
    - [動態路由](#%e5%8b%95%e6%85%8b%e8%b7%af%e7%94%b1)
      - [R1, R4](#r1-r4)
      - [R1, R2](#r1-r2)
      - [R1, R3](#r1-r3)

## 基礎 IP 設置

R1
```
enable
configure terminal
hostname R1
interface lo1
ip addr 11.1.1.1 255.255.255.0
int e0/0
ip addr 12.1.1.1 255.255.255.0
no shutdown
int e0/1
ip addr 12.1.1.1 255.255.255.0
no shutdown
int e0/2
ip addr 13.1.1.1 255.255.255.0
no shutdown
```

R2
```
enable
configure terminal
hostname R2
int lo0
ip addr 22.1.1.1 255.255.255.0
int e0/0
ip addr 12.1.1.2 255.255.255.0
no shutdown
do ping 12.1.1.1
```

R3
```
enable
configure terminal
hostname R3
interface lo0
ip addr 33.1.1.1 255.255.255.0
int e0/0
ip addr 13.1.1.3 255.255.255.0
no shutdown
do show ip interface brief
```

R4
```
enable
configure terminal
interface lo0
ip addr 44.1.1.1 255.255.255.0
int e0/0
ip addr 14.1.1.4 255.255.255.0
no shutdown

do ping 14.1.1.1
```

## 設置 R1, R2 的 EIGRP 協議

- EIGRP 的編號一定要相同

R2
```
enable
configure terminal
router eigrp 90
no auto-summmary # 取消網路合併
network 12.1.1.0 0.0.0.255
network 22.1.1.1 0.0.0.0 # ?
```

R1
```
enable
configure terminal
router eigrp 90
no auto-summary
network 12.1.1.0 0.0.0.255
network 22.1.1.1 0.0.0.0 # ?
do show ip route eigrp
do ping 22.1.1.1
```

## 設置 R1, R3 的 OSPF 協議

- OSPF 的編號不一定要相同，因為後方的是 Process ID

R3
```
enable
configure terminal
router ospf 1
network 13.1.1.0 0.0.0.255 area 0
# network 33.1.1.1 0.0.0.0 area 0 (X)
network 33.1.1.0 0.0.0.255 area 0
```

R1
```
enable
configure terminal
router ospf 1
network 13.1.1.0 0.0.0.255 area 0
network 11.1.1.1 0.0.0.0 area 0
```

## 設置 R1, R4 的 RIP 協議

R4
```
router rip
version 2
no auto-summary
network 14.1.1.4
network 44.1.1.1 # ?
```

R1
```
router rip
version 2
no auto-summary
network 14.1.1.1
network 44.1.1.1 # ?
```

## 不同的分配方式

- 重分配要設置在中央的位置

1. 直連
   - rip/eigrp: `redistribute connected`
   - ospf: `redistribute connected subnets`
2. 靜態路由：分為 default 與 static
   - rip/eigrp: `redistribute static`
   - ospf: `redistribute static subnets`, `default-information originate` (default route)
3. 動態路由
   - 一定要加 metric
   - rip: `redistribute eigrp 90 metric 2`
   - eigrp: `redistribute rip metric 1000 100 255 100 1000`
   - ospf `redistribute rip subnets`, `redistribute eigrp 90`

### 直連

R4
```
do show ip route # 都沒有直連的部分
```

R1
```
enable
configure terminal
router rip
redistribute connected # 設定直連（後方的 metric 可加可不加，若加的話最高設置到 15，因為 16 不可達）

router eigrp 90
redistribute connected

router ospf 1
redistribute connected subnets # ospf 後方需要加上 subnets
```

R2
```
ping 13.1.1.1 # 成功
ping 13.1.1.3 # 不會成功
ping 14.1.1.4 # 成功
```

R3
```
ping 12.1.1.2 # 成功
```

### 靜態路由

#### R1, R4 RIP

R1
```
ip route 111.1.1.0 255.255.255.0 null 0 # static 的路由；null 0 代表路由器如果接到此封包，會將它丟棄

ip route 0.0.0.0 0.0.0.0 null 0 # default 的路由

do show ip route # static 顯示 S*、default 顯示 S

router rip
redistribute static
```

R4
```
do show ip route # 學習到的 static 路由顯示 R*、default 顯示 R
```

#### R1, R2 EIGRP

R1
```
router eigrp 90
redistribute static
```

R2
```
do show ip route
```

#### R1, R3 OSPF

R1
```
router ospf 1
redistribute static subnets
```

R3
```
do show ip route # default route 消失
```

```
# do show ip route 資訊
      11.0.0.0/32 is subnetted, 1 subnets
O        11.1.1.1 [110/11] via 13.1.1.1, 00:26:56, Ethernet0/0
      12.0.0.0/24 is subnetted, 1 subnets
O E2     12.1.1.0 [110/20] via 13.1.1.1, 00:15:47, Ethernet0/0
      13.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        13.1.1.0/24 is directly connected, Ethernet0/0
L        13.1.1.3/32 is directly connected, Ethernet0/0
      14.0.0.0/24 is subnetted, 1 subnets
O E2     14.1.1.0 [110/20] via 13.1.1.1, 00:15:47, Ethernet0/0
      33.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
```

R1
```
default-information originate
```

```
# do show ip route 資訊，可看見多出第一行
O*E2  0.0.0.0/0 [110/1] via 13.1.1.1, 00:00:03, Ethernet0/0
      11.0.0.0/32 is subnetted, 1 subnets
O        11.1.1.1 [110/11] via 13.1.1.1, 00:27:16, Ethernet0/0
      12.0.0.0/24 is subnetted, 1 subnets
O E2     12.1.1.0 [110/20] via 13.1.1.1, 00:16:07, Ethernet0/0
      13.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        13.1.1.0/24 is directly connected, Ethernet0/0
L        13.1.1.3/32 is directly connected, Ethernet0/0
      14.0.0.0/24 is subnetted, 1 subnets
O E2     14.1.1.0 [110/20] via 13.1.1.1, 00:16:07, Ethernet0/0
```

### 動態路由

- 必須要加上 metric，否則數值預設為 infinite

#### R1, R4

R4
```
do show ip route # 尚未學到 R2 的 22.1.1.1
```

R1
```
enable
configure terminal
router rip
redistribute eigrp 90 # 錯誤示範，因為預設為 infinite，此時仍尚未學到
redistribute eigrp 90 metric 2 # 正確示範，需加上 metric
```

R4
```
do show ip route # 已經學到 R2 的 22.1.1.1
```

#### R1, R2

R1
```
router egirp 90
redistribute rip metric 1000 100 255 100 1500
# metric <bandwidth> <delay> <reliability> <loading> <MTU>
```

#### R1, R3

R1
```
router ospf 1
redistribute rip subnets # 不用加 metric
```

R3
```
do show ip route # metric 預設為 20
```

R1
```
redistribute eigrp 90 subnets
```

R3
```
do show ip route # 又多一筆，metric 預設也為 20
do ping 44.1.1.1 source 33.1.1.1 # 成功
```