# W4 20191001

- [W4 20191001](#w4-20191001)
  - [Docker save, export](#docker-save-export)
    - [Application: Create a website using Docker](#application-create-a-website-using-docker)
      - [1. Mariadb](#1-mariadb)
      - [2. apache, php](#2-apache-php)
      - [3. write a php page to test](#3-write-a-php-page-to-test)
      - [4.](#4)
    - [雜記](#%e9%9b%9c%e8%a8%98)

## Docker save, export

- ref: [[Docker] 比較 save, export 對於映象檔操作差異](https://blog.hinablue.me/docker-bi-jiao-save-export-dui-yu-ying-xiang-dang-cao-zuo-chai-yi/)
- `docker save`, `docker load`：適用於備份
  - 不只能備份一個 image
  - `docker save --help`：查看說明
  - `docker save -o two.tar <image1> <image2>`：備份多個 images 到 two.tar 檔案中
- `docker import`, `docker export`：適用於資料互通

```
docker images # 先確認有沒有 busybox image
docker pull busybox
docker tag 192.168.56.102:5000/busybox:latest busybox:latest
docker run -it busybox sh

mkdir -p /mydata
cd /mydata
touch {a..d}
ls
Ctrl + P, Q
```

```
docker save busybox:latest > busybox_save.tar
docker export quizzical_almeida > busybox_export.tar
```

- 再將 busybox_save.tar 與 busybox_export.tar
- 利用 `scp` 傳送檔案
- 先確認 `sshd` 服務有開啟

```
systemctl status sshd
```

- 傳送到 192.168.56.104 的 `/home/user` 資料夾

```
scp *.tar user@192.168.56.104:/home/user
```

```
cat busybox_export.tar | docker import - busybox1
docker load < busybox_save.tar
docker run -it busybox:latest sh
docker run -it busybox1:latest sh
ls /mydata
```

[[Docker] 比較 save, export 對於映象檔操作差異](https://blog.hinablue.me/docker-bi-jiao-save-export-dui-yu-ying-xiang-dang-cao-zuo-chai-yi/)

###

- Open two terminals
- 先建立一個基於 busybox 的 Container 稱為 c1

```
[root@localhost user]# docker run -it --name c1 --rm busybox sh
 
/ # ifconfig
# 查到 IP 為 172.17.0.2
```

- 再建立另一個 Container 稱為 c2
  - `--link c1`: 能在 ping 的時候直接 ping c1

```
docker run -it --name c2 --rm --link c1 busybox sh # 172.17.0.3
```

### Application: Create a website using Docker

- ref: [docker安裝apache、mariadb、php | 聰明的生活](https://blog.yslifes.com/archives/2523)

#### 1. Mariadb

- Pull the mariadb image (SQL community verstion)
```
docker pull mariadb
```

- Run docker
  - `-e`: Environment Variable

```
docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=<password> --restart unless-stopped mariadb
```

- Enter the docker

```
docker exec -it mariadb bash
```

- login to mysql

```
mysql -uroot -p
```


```
Enter password: <password>
cretae database test;
show databases; # we should see the 'test' db
```

#### 2. apache, php

- Add a new directory sharing to another docker container

```
mkdir -p /opt/www-data
```

- pull the php image

```
docker pull php:7.2-apache
```

- run the docker container
  - `-d`: daemon
  - `--restart unless-stopped`: Always restart the container regardless of the exit status, including on daemon startup, except if the container was put into a stopped state before the Docker daemon was stopped.
  - `--link mariadb:mariadb`: Add link to another container (`<name or id>:alias` or `<name or id>`)
  - ref: [Docker run reference | Docker Documentation](https://docs.docker.com/engine/reference/run/)

```
docker run -d --name apache --restart unless-stopped -p 80:80 -v /opt/www-data:/var/www/html --link mariadb:mariadb php:7.2-apache
```

```
docker exec -ti apache sh
# docker-php-ext-install mysqli
# docker-php-ext-enable mysqli
# apachectl restart
```

#### 3. write a php page to test

```
vim /opt/www-data/index.php
```

```php
<?php
$db = new mysqli('mariadb', '<user>', '<password>', 'test');
if (mysqli_connect_errno())
{
echo '<p>' . 'Connect DB error';
exit;
}
else
{
echo "Connection success";
}
```

- then open browser to check the result.

#### 4.

```
docker commit <containerID> myphp:<image>
docker commit <containerID> mariadb:<image>
```

### 雜記

- 如果介面卡的 IP 不見的時候，可以呼叫 DHCP 來重新取得：

```
dhclient enp0s8
```