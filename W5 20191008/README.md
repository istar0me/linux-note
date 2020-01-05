# W5 20191008

- [W5 20191008](#w5-20191008)
  - [Docker Network](#docker-network)
    - [Create containers](#create-containers)
    - [See the docker network status](#see-the-docker-network-status)
      - [1. brctl show](#1-brctl-show)
      - [2. docker network inspect bridge](#2-docker-network-inspect-bridge)
    - [Configure Network](#configure-network)

## Docker Network

> ref: [[Docker] Bridge Network 簡介 | 小信豬的原始部落](https://godleon.github.io/blog/Docker/docker-network-bridge/)

- Docker supports 3 types network
  1. bridge (default)
  2. host
  3. none

```
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
75a6b2ca700e        bridge              bridge              local
e6adac9216a3        host                host                local
d157cace1642        none                null                local
```

### Create containers

- create the busybox container as c1, c2

```
[root@localhost ~]# docker run -itd --name c1 busybox sh
dba56b9f45cd55ab42781cb2041ff0ee2078122cafa0304681f57af522c411f3
[root@localhost ~]# docker run -itd --name c2 busybox sh
932842672a72f69cc6d6eced7890d78f68e5caa6973e1366a39a09a3fa9dbfcf
```

### See the docker network status

#### 1. brctl show

- Using `brctl show` to show a list of bridges (software bridge status)
   - `veth30588fd`, `veth7957bc3` are the interfaces of c1, c2

- **brctl**: bridge Control
  - *if it's not installed, type `yum install bridge-utils`*
  - `brctl addbr mybr`
  - `brctl addif eth0`
  - `brctl addif eth1`
  - TODO: add description

```
[root@localhost ~]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242ae4e7478	no		veth30588fd
							            veth7957bc3
virbr0		8000.5254005ae1b8	yes		virbr0-nic
```

- show the IP address for c1 & c2
  - c1: `172.17.0.2`
  - c2: `172.17.0.3`

```
[root@localhost ~]# docker exec -it c1 ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@localhost ~]# docker exec -it c2 ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

#### 2. docker network inspect bridge

- see the bridge network status (with detail infos)
- we can conclude the network status as the table below:

| name   | c1          | c2          |
| --- | ----------- | ----------- |
| IP  | 172.17.0.2  | 172.17.0.3  |
| Interface  | veth30588fd | veth7957bc3 |

```
[root@localhost ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "75a6b2ca700e81724b21d10da96fbcc84b7a86aca9b46558ca3158ec7988975e",
        "Created": "2019-10-08T03:48:28.228940265-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "932842672a72f69cc6d6eced7890d78f68e5caa6973e1366a39a09a3fa9dbfcf": {
                "Name": "c2",
                "EndpointID": "f7f95f26dca9f0638ad98ab0b4083b56d72e81092f0b33646c2ea0c43a789034",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "dba56b9f45cd55ab42781cb2041ff0ee2078122cafa0304681f57af522c411f3": {
                "Name": "c1",
                "EndpointID": "84e36f86d744696b8fe036aeb4c45ce4b272efe59e968cabda2e2910162e790e",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

### Configure Network

- if we use the default network setting, containers can only use IP address for communication.

```
# explanation
docker exec -it c1 ping -c 3 <c2IP>     # work
docker exec -it c1 ping -c 3 c2 # doesn't work
```

```
# practice
[root@localhost ~]# docker exec -it c1 ping -c 3 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.224 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.172 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.140 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.140/0.178/0.224 ms
[root@localhost ~]# docker exec -it c1 ping -c 3 c2

^C[root@localhost ~]# 
```

- create a new bridge network named mynet

```
[root@localhost ~]# docker network create --driver bridge mynet
28ebbf105b758df6f5a3c792fc0ca75fd39b7d5a51e3384bcf2323e940ebb298
[root@localhost ~]# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "28ebbf105b758df6f5a3c792fc0ca75fd39b7d5a51e3384bcf2323e940ebb298",
        "Created": "2019-10-08T04:26:20.111923757-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

- TODO: Add topology image

```
docker run -itd --name c3 --network mynet busybox sh
docker run -itd --name c4 --network mynet busybox sh
```

- remove all the containers

```
docker rm -f $(docker ps -q -a)
```

- create conatiners

```
docker run -itd --name c1 --network mynet busybox sh
docker run -itd --name c2 --network mynet busybox sh
docker run -itd --name c3 busybox sh
docker run -itd --name c4 --network mynet busybox sh
docker network connect bridge c4
```

```
docker network ls
```

- list all the IP address

```
docker exec -it c1 ip addr show
docker exec -it c2 ip addr show
docker exec -it c3 ip addr show
docker exec -it c4 ip addr show
```

```
docker exe -it c1 ping -c2 c2             # work
docker exe -it c1 ping -c2 c3             # work
docker exe -it c1 ping -c2 c4     # doesn't work
docker exe -it c1 ping -c2 <c3IP> # doesn't work
```

- see the command that docker network support, just add `?` at the end
- or run 'docker network COMMAND --help' for more information on a command.

```
docker network ?
```

- run c5 using host network
  - host network often using in testing, not recommended for bussiness/formal usage

```
docker run -itd --name c5 --network host busybox sh
```

- check the c5 IP address
```
docker exec -it c5 ipaddr
```

- run c6 using `none` network, meaning it doesn't have any network connected(only lo interface)

```
docker run -itd --name c6 --network none busybox sh
```