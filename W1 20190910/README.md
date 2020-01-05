# W1 20190910

- [W1 20190910](#w1-20190910)
  - [Introduction](#introduction)
  - [Docker Highlight](#docker-highlight)
  - [Docker Installation](#docker-installation)
  - [Docker Terminology](#docker-terminology)
  - [雜記](#%e9%9b%9c%e8%a8%98)
  - [Reference](#reference)

## Introduction

- The course mainly talks about Docker & Kubernete
- Textbook:
  - [Docker這樣學才有趣：從入門，到玩直播、挖礦](https://24h.pchome.com.tw/books/prod/DJAA2V-A9008IVZQ-000?fq=/S/DJAA4B)
  - [Docker 與 Kubernetes 集群](https://www.tenlong.com.tw/products/9787121339066)

## Docker Highlight

- 彈性化、輕量化
- 避免環境的差異影響執行結果
- 避免工具與版本過於零散
- 簡化移交流程：得益於輕量化，打包與傳輸 container 較簡易
- 效能較 VirtualBox 高，因為少了 N 個 Guest OS
- Host OS 一定是 Linux

## Docker Installation

- [Get Docker Engine - Community for CentOS | Docker Documentation](https://docs.docker.com/install/linux/docker-ce/centos/)

```bash
[root@localhost Desktop]# docker pull chusiang/takaojs1607
Using default tag: latest
// 省略
Status: Downloaded newer image for chusiang/takaojs1607:latest
docker.io/chusiang/takaojs1607:latest
[root@localhost Desktop]# docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
httpd                  latest              5eace252f2f2        5 months ago        132MB
istar0me/centos        0.1                 4b0a69e065d3        5 months ago        286MB
centos                 0.1                 4b0a69e065d3        5 months ago        286MB
centos                 7                   1e1148e4cc2c        9 months ago        202MB
centos                 latest              1e1148e4cc2c        9 months ago        202MB
chusiang/takaojs1607   latest              90ebc637b878        2 years ago         1.31GB
```

- `docker images`: show all the docker images
- `docker pull <dockerRegistry> <account> <imageName>:<tag>`
  - e.g. `docker pull chusiang/takaojs1607`
  - 如果 dockerRegistry 是 dockerhub，可以省略不寫
  - 如果 tag 為 latest 可省略不寫
- `docker run` = `docker create` + `docker start`
  - `docker run -it -d --name <name> <images> <terminal>`
    - e.g. `docker run -it -d --name test1 chusiang/takaojs1607 bash`
    - i: interactive
    - t: terminal
    - d: daemon (run in the background)
    - --name: specify name of the container
- `docker ps`: show all the (docker) running process
  - `docker ps -a`: a for all, including the inactive containers
- `docker stop <containerID>`
- `docker rm <containerID>`
  - `docker rm -f <containerID>`: f for force
  - `docker container prune`: remove all stopped containers.
  - `docker rm -f $(docker ps -q -a)`: remove all the container (even when the container is running)
    - `docker ps -q -a`: show all the container ID
      - q: query
      - a: all

---

- 如果不想直接輸入 `docker run -it chusiang/takaojs1607 bash` 直接啟動並進入終端機，也可分成兩個步驟操作
  1. `docker start <containerID>`: 啟動
  2. `docker attach <containerID>`: 進入終端機中

## Docker Terminology

- ![Docker Commands Diagram](./Docker-Command-Diagram.png)
- Docker Hub: online container images
- Docker Registry: the place where docker image storing, whether in public/private cloud.
- image: class
- container: object

## 雜記

- IPS: Internet Protection System
- WAF: Web Application Firewall
- redis: in-memory key-value DB (perform faster than traditional DB)
  - 可用於建置搶購的網站（瞬間負載高）

## Reference

- [前端工程師一定要知道的 Docker 虛擬化容器技巧 (18:34) - YouTube](https://www.youtube.com/watch?v=k5iwKUZY9tk)
