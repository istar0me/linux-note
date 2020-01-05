# Docker Swarm (cont.)

- [Docker Swarm (cont.)](#docker-swarm-cont)
  - [1. Local Data Persistent](#1-local-data-persistent)
  - [2. share the same files](#2-share-the-same-files)
  - [Service Discovery](#service-discovery)

vm1, vm2, vm3
```
docker pull pingbo/whoami
```

vm1
```
docker service create --name web --replicas 2 -p 8000:8000 pingbo/whoami
```

## 1. Local Data Persistent

> ref: [Docker集群管理Swarm數據持久化 - Bigberg - 博客園](https://www.cnblogs.com/bigberg/p/8795265.html)

vm1
```
docker service create --replicas 3 --name web --mount type=volume,src=local_data,dst=/usr/local/apache2/htdocs httpd:latest

docker volume ls
cd <volume_path>
echo "I am vm1" >> hi.html

docker service update --publish-add 80:80 web
```

## 2. share the same files

- NFS (今天介紹)
- Ceph (需自學，bilibili 網站內有介紹)

vm1, vm2, vm3
```
yum install -y nfs-utils
```

vm1
```
systemctl start nfs
systemctl start rpcbind
mkdir -p /data/nfs_vol # 待會要共享的資料夾
```

vm1
```
vim /etc/exports

# file content:
/data/nfs_vol 172.16.45.128(rw,sync,no_root_squash)

systemctl restart nfs
```

vm2, vm3
```
mkdir -p /tmp
mount -t nfs 172.16.45.128:/data/nfs_vol /tmp # 測試是否能掛載
```

vm2
```
cd /tmp
touch vm2
```

vm3
```
cd /tmp
touch vm3
```

vm1
```
cd /tmp
touch vm1
ls # 若有掛載順利，應會看到 vm1, vm2, vm3
```

vm1
```
docker service create --replicas 3 --name web -p 80:80 --mount 'type=volume,src=share_data,dst=/usr/local/apache2/htdocs,volume-driver=local,volume-nocopy=true,volume-opt=type=nfs,volume-opt=device=172.16.45.128:/data/nfs_vol,"volume-opt=o=addr-172.16.45.128,vers=4,soft,timeo=180,bg,top,rw"' httpd

cd /data/nfs_vol/
ls # 應會見到 vm1, vm2, vm3
echo "hi1" > hi1.html
curl 172.16.45.128/hi.html
```

## Service Discovery

- 以往需手動設定 IP 對應，此時若要縮編或擴展其他裝置不清楚
- 因此可以透過 Service Discovery Service(它本身也是 service)來解決
- Docker Swarm 中的 overlay 網路能用來處理 Service Discovery，但預設的 overlay 並沒有提供此功能，**需手動新增一個 overlay 網路來處理**

vm1
```
docker service ls
docker service rm web # 先將舊的 service 移除

docker network create --driver overlay mynet
docker network ls # 應會見到新的名為 mynet 的 overlay 型態的網路

docker service create --name s1 --replicas 2 --network mynet httpd

docker service create --name s2 --repliacs 2 --network mynet busybox
```

vm1, vm2, vm3
```
docker ps
docker inspect <busybox_service_id> # 查看 IP

nslookup s2 # 可以得到 service ip
```