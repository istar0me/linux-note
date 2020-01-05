# Outline

- [Outline](#outline)
  - [docker](#docker)
    - [Docker run](#docker-run)
    - [Docker Volume: 掛載 Docker 中的資料夾到本機端](#docker-volume-%e6%8e%9b%e8%bc%89-docker-%e4%b8%ad%e7%9a%84%e8%b3%87%e6%96%99%e5%a4%be%e5%88%b0%e6%9c%ac%e6%a9%9f%e7%ab%af)
    - [Docker exec](#docker-exec)
    - [執行缺失的指令](#%e5%9f%b7%e8%a1%8c%e7%bc%ba%e5%a4%b1%e7%9a%84%e6%8c%87%e4%bb%a4)
    - [Docker Commit](#docker-commit)
    - [實戰：實際架設一個 Apache Server](#%e5%af%a6%e6%88%b0%e5%af%a6%e9%9a%9b%e6%9e%b6%e8%a8%ad%e4%b8%80%e5%80%8b-apache-server)

## docker

### Docker run

- `docker run -it <imagesName> -d /bin/bash -c <script>`
  - e.g. `docker run -it chusiang/takaojs1607 -d /bin/bash -c "while true; do echo 'hi'; sleep 1; done"`
  - description: echo 'hi' every 1 second
  - `-d` : daemon (run in background)
- `docker logs <dockerID/dockerName>`: 可以查看 Docker 印出的 Log 檔
- `docker run -it -v /mydata:/data <imagesName> /bin/bash`：將本機端的 `/mydata` 映射到 docker 中的 `/data/`
  - e.g. `docker run -it -v /mydata:/data chusiang/takaojs1607 /bin/bash`
  - `-v`: volume
  - 在本地端即可修改 docker 內的檔案
  - 持久化，能確保資料能永久保存
  - 類似於 `mount` 的指令

### Docker Volume: 掛載 Docker 中的資料夾到本機端

- Docker:

```
[root@localhost /]# mkdir /data
[root@localhost /]# cd /data
[root@localhost data]# echo "hello world" > hello.txt
[root@localhost data]# cat hello.txt
hello world
```

- Host:

```
[root@localhost /]# mkdir /mydata
[root@localhost /]# cd /mydata
[root@localhost mydata]# docker run -it -v /mydata:/data chusiang/takaojs1607 /bin/bash
```

### Docker exec

- `Docker exec -it <containerID> bash`: add a new terminal
- `docker run -it --name test1 --rm chusiang/takaojs1607 /bin/bash`
  - `--rm`: when exit the container, it will stop automatically. If I don't add `--rm`, It will keep running in background.
- `docker attach <containerID>`
- Press Ctrl + P + Q : exit docker temporary

### 執行缺失的指令

- docker 缺少很多基本功能，如 `ping` 與 `ifconfig`
- 找不到相對的指令時，可以直接搜尋錯誤訊息，然後再去安裝相對應的套件

```
root@2e19fee8afeb:/# apt-get update
root@2e19fee8afeb:/# apt-get install iputils-ping // 安裝 ping
root@2e19fee8afeb:/# apt-get install net-tools // 安裝 ifconfig
```

- 按下 Ctrl + P + Q 暫時離開 Container
- 再透過 `docker commit 2e19 ubuntu/2.0` 進行 Commit

### Docker Commit

- 將特定版本保存起來
- `docker commit <containerID> <REPOSITORY>:<TAG>`
  - e.g. `docker commit cce ubuntu/1.0`


### 實戰：實際架設一個 Apache Server

- ref: [httpd - Docker Hub](https://hub.docker.com/_/httpd)

```
[root@localhost mydata]# docker pull httpd
[root@localhost mydata]# docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
ubuntu                 latest              a2a15febcdf3        4 weeks ago         64.2MB
httpd                  latest              5eace252f2f2        5 months ago        132MB
istar0me/centos        0.1                 4b0a69e065d3        6 months ago        286MB
centos                 0.1                 4b0a69e065d3        6 months ago        286MB
centos                 7                   1e1148e4cc2c        9 months ago        202MB
centos                 latest              1e1148e4cc2c        9 months ago        202MB
chusiang/takaojs1607   latest              90ebc637b878        2 years ago         1.31GB
[root@localhost mydata]# echo "this is a index webpage" > index.html
[root@localhost mydata]# echo "hi" > hi.html
[root@localhost mydata]# ls
a  b  c  d  hello.txt  hi.html  index.html
[root@localhost mydata]# docker run -itd --name myWebServer -p 8080:80 -v /mydata:/usr/local/apache2/htdocs httpd
```

- httpd 的家目錄在 `/usr/local/apache2/htdocs/`
- `docker run -itd --name myWebServer -p 8080:80 -v /mydata:/usr/local/apache2/htdocs httpd`
  - `-p, --publish`: Publish a container's port(s) to the host
    - `-p 8080:80`: host 8080 <-> container 80 port
  - `-P, --publish-all`: Publish all exposed ports to random ports (**capitalized P**)
