# Routing Experiment

- [Routing Experiment](#routing-experiment)
  - [使用內定路由](#%e4%bd%bf%e7%94%a8%e5%85%a7%e5%ae%9a%e8%b7%af%e7%94%b1)
  - [使用內定路由 (cont.)](#%e4%bd%bf%e7%94%a8%e5%85%a7%e5%ae%9a%e8%b7%af%e7%94%b1-cont)
  - [使用浮動路由](#%e4%bd%bf%e7%94%a8%e6%b5%ae%e5%8b%95%e8%b7%af%e7%94%b1)
  - [How to see the router table?](#how-to-see-the-router-table)

## 使用內定路由

R2
```
R2#configure terminal
R2(config)#interface loopback 2
R2(config-if)#ip addr 8.8.8.8 255.255.255.255
R2(config-if)#do show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                12.1.1.2        YES manual up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Loopback1                  2.2.2.2         YES manual up                    up
Loopback2                  8.8.8.8         YES manual up                    up
R2(config-if)#ip route 0.0.0.0 255.255.255.255 12.1.1.1
R2(config)#do show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

S     0.0.0.0/32 [1/0] via 12.1.1.1
      1.0.0.0/32 is subnetted, 1 subnets
S        1.1.1.1 [1/0] via 12.1.1.1
      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback1
      8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback2
      12.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        12.1.1.0/24 is directly connected, Ethernet0/0
L        12.1.1.2/32 is directly connected, Ethernet0/0
```

---

R1
```
en
conf t
hostname R1
int e0/0
ip addr 12.1.1.1 255.255.255.0
no shutdown
int lo 1
ip addr 1.1.1.1 255.255.255.255
do show ip int brief # 確認是否 up
```

R2
```
en
conf t
hostname R2
int e0/0
ip addr 12.1.1.2 255.255.255.0
no shutdown
int lo 1
ip addr 2.2.2.2 255.255.255.255
do show ip int brief # 確認是否 up
```

R1
```
ping 12.1.1.2 #.代表失敗, !代表成功
.!!!! # 回應，第一個封包傳送失敗代表 ARP 解析尚未完成，因此會失敗
show ip route # 之所以會 Ping 成功，是因為有路由表
ping 12.1.1.2 repeat 3 # 重複三次（預設五次），只有在 eve-NG 可用；cisco packet tracer 不可用
ping 12.1.1.2 source 1.1.1.1 # 封包出得去、回不來
```

R2
```
ip route 1.1.1.1 255.255.255.255 12.1.1.1 # 12.1.1.1 為中間經過的路線
do show ip route
# 會出現 [1/0]
# 前面是 Administrative Distance, AD（管理距離）：數值越小、權限越大
# 後面是 Metric（度量值）：起點到終點的行走成本，數值越小、
```

R1
```
ping 2.2.2.2
ip route 2.2.2.2 255.255.255.255 12.1.1.2
```

## 使用內定路由 (cont.)

R2
```
enable
configure terminal
interface loopback 2
ip addr 8.8.8.8 255.255.255.255
do show ip interface brief
ip route 0.0.0.0 255.255.255.255 12.1.1.1
do show ip route
```

R1
```
enable
ping 8.8.8.8
# ..... 代表五次失敗
ip route 0.0.0.0 255.255.255.255 12.1.1.2
do show ip route
# 出現 S*，代表以內定路由控制（candidate default）
do ping 8.8.8.8 # 成功
do ping 8.8.8.8 source 1.1.1.1 # 成功
```

## 使用浮動路由

- 再新增一台機器
- Nums of Nodes:1, Image: L3_ADVEN...
- add text
- Equal Cost Path: 等價路由

R1
```
enable
configure terminal
interface 0/1
ip addr 12.1.1.1 255.255.255.0
no shutdown
do show ip route
ip route 0.0.0.0 0.0.0.0.0 13.1.1.3 ?
no ip route 0.0.0.0 0.0.0.0.0 13.1.1.3
ip route 0.0.0.0 0.0.0.0.0 13.1.1.3 10 # 將 AD 設定為 10
do show ip route # 不會顯示剛剛新增的 13.1.1.3 路徑
do show ip static route 0.0.0.0/0 # 顯示到某個位置的所有路徑
# 會看到兩筆，一筆是 A(Active)，另一筆是 N(Non-Active)
```

R3
```
enabele
configure terminal
hostname R3
interface 30/0
ip addr 13.1.1.3 255.255.255.0
no shutdown
interface loopback 13.1.1.3
ip addr 8.8.8.8 255.255.255.255
do show ip route
exit
ip route 0.0.0.0 0.0.0.0 13.1.1.1
```

R2&R3
- 除了可用 WireShark 觀察封包外，也可用以下指令查看：

```
debug ip icmp
```

R1
```
interface e0/0
shutdown
ping 8.8.8.8 source 1.1.1.1
```

## How to see the router table?

```
do show arp # router
show mac-address-table # switch
```