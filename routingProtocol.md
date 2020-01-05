# Routing Protocol

- [Routing Protocol](#routing-protocol)
  - [Static Routing &amp; Dynamic Routing](#static-routing-amp-dynamic-routing)
    - [Static Routing](#static-routing)
    - [Dynamic Routing](#dynamic-routing)
      - [Distance Vector (距離向量)](#distance-vector-%e8%b7%9d%e9%9b%a2%e5%90%91%e9%87%8f)
      - [Link State (連線狀態)](#link-state-%e9%80%a3%e7%b7%9a%e7%8b%80%e6%85%8b)
  - [Common Routing Protocols](#common-routing-protocols)
    - [Classification](#classification)
  - [Reference](#reference)

## Static Routing & Dynamic Routing

### Static Routing

- Pros：simple
- Cons：updating route frequently, occupy transmission bandwidth.

### Dynamic Routing

- Pros：If path in the network is broken, it can find an alternative path
- Cons: maintenance cost
- There're two consideration factors: Distance Vector & Link State

#### Distance Vector (距離向量)

- Exchange the routing info to the neighbor router routinely.
  - Distance: metrics
  - Vector: next node
- Suited for small/medium-sized network
- *The shortest distance doesn't mean the fastest path.*
- e.g. RIP, EIGRP

#### Link State (連線狀態)

- Evaluation steps:
  1. By **flood**(broadcast), understanding the full network topology.
  2. Calculate the node to node distance by algorithm.
- every router has full network topology.
- if the network topology doesn't change, it won't need to calculate new routing.
- e.g. OSPF

## Common Routing Protocols

### Classification

| -               | IGP(同個自治區內)   | EGP(跨自治區) |
| --------------- | ------------------- | ------------- |
| Distance Vector | RIP(120), EIGRP(90) |
| Link State      | OSPF(110)           |
| -               | -                   | BGP           |

- RIP is a standard protocol, declard in RCF
- EIGRP only availabe in Cisco Router

---

1. Number of nodes:2, images: L3_ADVEN..

R1
```
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#interface e0/0
R1(config-if)#ip addr 12.1.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#int loopback 1
R1(config-if)#ip addr 1.1.1.1 255.255.255.255
R1(config-if)#do show ip interface brief // 確認狀態是否為 up
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                12.1.1.1        YES manual up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Loopback1                  1.1.1.1         YES manual up                    up
```

R2
```
Router>enable
Router#configure terminal
Router(config)#hostname R2
R2(config)#interface e0/0
R2(config-if)#ip addr 12.1.1.2 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#interface loopback 1
R2(config-if)#ip addr 2.2.2.2 255.255.255.255

R2(config-if)#do show ip interface brief // 確認狀態是否為 up
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                12.1.1.2        YES manual up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Loopback1                  2.2.2.2         YES manual up                    up
```

R1
```
R1#ping 12.1.1.2 // . means success, ! means fail
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.1.1.2, timeout is 2 seconds:
.!!!! // ARP 解析尚未完成，因此第一個封包傳送失敗
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/2 ms
R1#ping 12.1.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.1.1.2, timeout is 2 seconds:
!!!!! // ARP 解析已完成，因此全部封包傳送成功
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R1#show ip route // 因為有路由表，所以才能 ping 成功
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

      1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback1
      12.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        12.1.1.0/24 is directly connected, Ethernet0/0
L        12.1.1.1/32 is directly connected, Ethernet0/0
R1#ping 12.1.1.2 repeat 3 // 重複三次（預設五次）
Type escape sequence to abort.
Sending 3, 100-byte ICMP Echos to 12.1.1.2, timeout is 2 seconds:
!!!
Success rate is 100 percent (3/3), round-trip min/avg/max = 1/1/1 ms
R1#ping 12.1.1.2 source 1.1.1.1 // 指定來源為 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.1.1.2, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1
..... // 封包出得去、回不來，因為 R1 路由表中沒有 1.1.1.1 的位址
Success rate is 0 percent (0/5)
```

R2
```
R2(config)#ip route 1.1.1.1 255.255.255.255 12.1.1.1 // dest: 1.1.1.1, mask: 255.255.255.255, via: 12.1.1.1
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

      1.0.0.0/32 is subnetted, 1 subnets
S        1.1.1.1 [1/0] via 12.1.1.1
      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback1
      12.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        12.1.1.0/24 is directly connected, Ethernet0/0
L        12.1.1.2/32 is directly connected, Ethernet0/0
```

* [1/0]
  * 1: Administrative Distance, AD (管理距離): 數值越小、權限越大
  * 0: Metric (度量值): 起點到終點的行走成本
```
S        1.1.1.1 [1/0] via 12.1.1.1
```

R1
```
R1#ping 2.2.2.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R1#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ip route 2.2.2.2 255.255.255.255 12.1.1.2
R1(config)#do ping 2.2.2.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

## Reference

- [[筆記]Cisco基本指令-路由 @ David Liao's Blog :: 痞客邦 ::](https://david50.pixnet.net/blog/post/45224331-%5B%E7%AD%86%E8%A8%98%5Dcisco%E5%9F%BA%E6%9C%AC%E6%8C%87%E4%BB%A4-%E8%B7%AF%E7%94%B1)
- [What is 0.0.0.0 IP address and what are its uses - Threat Brief](https://threatbrief.com/0-0-0-0-ip-address-uses/)