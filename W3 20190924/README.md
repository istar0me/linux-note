# W3 20190924

> 特別感謝：蘇川民提供上課影片，讓我返臺面試期間還有畫面可以跟著操作

- [W3 20190924](#w3-20190924)
  - [Environment](#environment)
  - [Export](#export)
  - [Import](#import)
  - [Make Connection between master and slave](#make-connection-between-master-and-slave)
    - [Pro Tip: Ping custom name instead of IP](#pro-tip-ping-custom-name-instead-of-ip)

## Environment

- Both VMs are based on Ubuntu 16.04

| VM     | IP         |
| ------ | ---------- |
| Master | 10.140.0.7 |
| Slave  | 10.140.0.9 |

## Export

Master:

```
# docker pull busybox
# docker run -it busybox sh

// inside busybox
# mkdir -p /mydata
# cd /mydata
# touch helloWorld
# ls
helloWorld

// press Ctrl + P + Q to exit busybox container

# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
c29fde2826d9        busybox                "sh"                     4 minutes ago       Up 4 minutes                            vigorous_tharp

# docker save busybox:latest > busybox_save.tar
# docker export vigorous_tharp > busybox_export.tar // vigorous_tharp is the name of busybox container

# ls *.tar // show the tar file that we save and export
busybox_export.tar  busybox_save.tar
```

## Import

Slave:
```
# ssh-copy-id 10.140.0.7
```

Master:
```
# scp *.tar user@10.140.0.9:/home/user
```

Slave:
```
# cd /home/user
# ls
busybox_export.tar  busybox_save.tar
# docker rmi busybox:latest

# cat busybox_export.tar | docker import - busybox1 // import from busybox_export.tar file, and name it as busybox1
# docker images // there's new image called busybox1

# docker load < busybox_save.tar
# docker images // there's new image called busybox


# docker run -it busybox:latest sh 
# ls
// nothing

# docker run -it busybox1:latest sh
# ls /mydata
helloWorld
```

## Make Connection between master and slave

Master:
```
# docker run -it --name c1 --rm busybox sh

// inside busybox
ifconfig // IP = 172.17.0.3
```

Slave:
```
# docker run -it --name c2 --rm busybox sh

// inside busybox
ifconfig // IP = 172.17.0.4
```

Master(inside busybox):
```
# ping 172.17.0.4 // successful
```

Slave(inside busybox):
```
# ping 172.17.0.3 // successful
```

### Pro Tip: Ping custom name instead of IP

Slave:
```
# docker run -it --name c2 --rm --link c1 busybox sh

// inside busybox
# ping c1 // successful
```