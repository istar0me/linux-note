# W9

- [W9](#w9)
  - [Video Streaming Service](#video-streaming-service)
    - [Machines](#machines)
    - [Prerequisite](#prerequisite)
      - [2 VMs](#2-vms)
      - [Host Machine](#host-machine)
    - [Steps](#steps)
      - [1. VM1](#1-vm1)
      - [2. VM2](#2-vm2)
  - [雜記](#%e9%9b%9c%e8%a8%98)

## Video Streaming Service

- ![]()
- RTMP: Real-Time Messagin Protocol
- ffmpeg: image compression, streaming software (mainly for enconding)

### Machines

1. CentOS 7 (NAT+Host only)
   - IP: 172.16.45.128
2. CentOS 7 (NAT+Host only)
   - IP: 172.16.45.130

### Prerequisite

#### 2 VMs

1. Disable firewall service

```
systemctl disable firewalld
systemctl stop firewalld
```

2. Disable SELinux (After editing the file, don't forget to reboot)

- file: `/etc/selinux/config`
```
SELINUX=disabled # don't forget the last d
```

#### Host Machine

1. On the host machine, [install the vlc player](https://www.videolan.org/vlc/index.zh-TW.html)

### Steps

> *All the steps below are executed in the root*

#### 1. VM1

> ref: [在CentOS 7上設定Nginx-RTMP - IT閱讀](https://www.itread01.com/content/1544615310.html)

1. Make sure the connection between host machine and virtual machine

```
ping <IP>
```

2. Install ssh server

```
yum install -y openssh-server
systemctl start sshd
systemctl status sshd # check whether the ssh service is start
```

3. install the compile software (e.g. gcc) to

```
yum groupinstall -y "Development Tools"
```

4. create a temp directory

```
mkdir ~/temp
cd ~/temp
```

5. Download the nginx & nginx-rtmp binary file

```
wget http://nginx.org/download/nginx-1.9.9.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
```

6. install the unzip software

```
yum install -y unzip
```

7. unzip the files

```
tar -xvf nginx-1.9.9.tar.gz
unzip master.zip
cd nginx-1.9.9
```

8. Install

```
./configure --add-module=../nginx-rtmp-module-master/
make
make install
```

9. ...

```
mkdir /etc/nginx -p
cd conf
cp nginx.conf mime.types /etc/nginx/
```

10. Modify the `/etc/nginx/nginx.conf` file

```
# Append the text at the end

rtmp {
  server {
    listen 1935;
    chunk_size 4096;

    application live {
      live on;
      record off;
    }
  }
}
```

11. configure

```
/usr/local/nginx/sbin/nginx -c /etc/nginx/nginx.conf # -c: configuration
```

12. check whether the port is listened

```
netstat -tunlp | grep 80
netstat -tunlp | grep 1935
```

#### 2. VM2

1. Bring the app up to date

```
yum install -y epel-release
yum update -y # It may take a while to complete ...
shutdown -r now
```

2. Install Nux library

```
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
```

3. Install the ffmpeg dependencies

```
yum install -y ffmpeg ffmpeg-devel
```

4. Use winscp or FileZille to Transfer the mp4 file to the VM 2

5. Playing test, make sure the file isn't broken

```
ffplay sample.mp4
```

6. Start the streaming service

```
ffmpeg -re -i sample.mp4 -c copy -f flv rtmp://<VM1_IP>/live/demo
```

7. On the host, open the vlcplay(or other player) enter the address below to test.

```
rtmp://<VM1_IP>/live/demo
```

## 雜記

- 下下週要上 Docker swarm 與 K8s
- Docker swarm
  - 2 CentOS VMs
- Kubernetes
  - 3 Ubuntu VMs (recommended version: 16.04)
    - master: 2 cores, 2GB RAM
    - slave1, slave2: not mentioned