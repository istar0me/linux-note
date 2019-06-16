- [Docker 簡介](#docker-%E7%B0%A1%E4%BB%8B)
  - [除錯：安裝過程出現錯誤](#%E9%99%A4%E9%8C%AF%E5%AE%89%E8%A3%9D%E9%81%8E%E7%A8%8B%E5%87%BA%E7%8F%BE%E9%8C%AF%E8%AA%A4)
- [Docker 指令](#docker-%E6%8C%87%E4%BB%A4)
  - [基礎指令](#%E5%9F%BA%E7%A4%8E%E6%8C%87%E4%BB%A4)
  - [除錯：若指令不能執行⋯⋯](#%E9%99%A4%E9%8C%AF%E8%8B%A5%E6%8C%87%E4%BB%A4%E4%B8%8D%E8%83%BD%E5%9F%B7%E8%A1%8C%E2%8B%AF%E2%8B%AF)
  - [Docker run](#docker-run)
    - [掛載路徑、埠號對應](#%E6%8E%9B%E8%BC%89%E8%B7%AF%E5%BE%91%E5%9F%A0%E8%99%9F%E5%B0%8D%E6%87%89)
    - [設置環境變數](#%E8%A8%AD%E7%BD%AE%E7%92%B0%E5%A2%83%E8%AE%8A%E6%95%B8)
- [其它](#%E5%85%B6%E5%AE%83)
- [Docker File](#docker-file)

---

# Docker 簡介
* 容器運用很多空間隔離、程序隔離等觀念
* 開機後第一個執行的程式為 `systemd`（可透過 `pstree` 查詢
* Docker 最重要的兩項技術：
    1. `namespace`
    2. `CGroup`：用作資源管理
* 製作成鏡像後，只能讀不能寫
* 修改後會形成新的鏡像，但若沒有儲存成新的鏡像，會還原到上個階段
* image（鏡像）啟動後為 Container（容器）
* 另外一個安裝 Docker 的指令：`curl -sSL https://get.docker.com | sh`


## 除錯：安裝過程出現錯誤
* 如果原先透過 `yum install -y docker` 安裝後，再透過 `yum remove -y docker` 移除後，會出現 `Warning: the "docker" command appears to already exist on this system.` 的問題
* 解決方法：移除其他 docker 相關的套件（可透過 `yum list installed | grep docker` 查詢）
* 參照：[Docker的安裝與啟動-生活就是等待戈多-51CTO博客](https://blog.51cto.com/chenx1242/1844932)

# Docker 指令
## 基礎指令
* `docker run` = `docker create` + `docker start`
    * 參照：[docker run 與docker start的區別，為容器命名 | 王民利的個人站點](https://www.wangminli.com/?p=1184)
    * `docker create` : 將鏡像放入容器中
    * `docker start` : 
    * `docker attach` : 
    * Ctrl-p + Ctrl-q : detach the tty without exiting the shell
    * `docker ps -a` : 顯示所有 container (包含未啟動)
    * `docker rm -f containerID/name` : remove container
        * `docker rm -f $(docker ps -a -q)` : 移除所有 containers
        * 以上指令會先執行 `docker ps -a -q`，`-q` 代表 only display numeric IDs
    * `docker rmi iamgeID/name` : remove image
        * 如果不能刪除，可以嘗試加上 `-f` (force)

## 除錯：若指令不能執行⋯⋯
* 由於作業系統在 docker 盡可能輕量化，因此部份指令可能不行執行，若發生此情形可搜尋 `指令 not found yum install`，如 `ifconfig not found yum install`
    * `docker run -t -i` → can be detached with ^P^Qand reattached with docker attach
    * `docker run -i` → cannot be detached with ^P^Q; will disrupt stdin
    * `docker run` → cannot be detached with ^P^Q; can SIGKILL client; can reattach with docker attach
* [Day6：把 Docker Image Push 到 Docker Hub - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天](https://ithelp.ithome.com.tw/articles/10191139)

## Docker run
* `docker run --rm centos:7`
  * 執行完就移除
  * `--rm=true|false` : Automatically remove the container when it exits. The default is false. `--rm`  flag  can  work together with `-d`, and auto-removal will be done on daemon side. Note that it's incompatible
       with any restart policy other than none.
* `docker inspect <container_id|container_name>` : 查看容器的資訊

### 掛載路徑、埠號對應
* `docker run -it --rm -v <Linux路徑>:<docker路徑> <image> /bin/bash`
  * `-v|--volume[=[[HOST-DIR:]CONTAINER-DIR[:OPTIONS]]]` : Create a bind mount. If you specify, -v /HOST-DIR:/CONTAINER-DIR, Docker bind mounts /HOST-DIR in the host to /CONTAINER-DIR in the Docker container. If 'HOST-DIR' is omitted,  Docker automatically creates the new volume on the host.
* `docker run -d -p <image_name> 8080:80 httpd` : 容器內的 Apache 使用 80 埠，對應到主機的 8080 埠
  * `-d` : daemon（在背景執行）
  * `-p` : 將容器內的埠號與主機的埠號對應
* `docker rm -f $(docker ps -a -q)` : 移除所有 Container
* `/usr/local/apache2/htdocs` : docker 中 httpd 的網頁路徑

```
[root@localhost Desktop]# docker run -d -p 8080:80 -v /mydata:/usr/local/apache2/htdocs httpd
6faf7446f036f268da4cb4edf792764f787fe31a9fefdb0533ea5d2155c3370d
[root@localhost Desktop]# cd /mydata
[root@localhost mydata]# echo "hi" > a.html
[root@localhost mydata]# docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                  NAMES
6faf7446f036        httpd               "httpd-foreground"   55 seconds ago      Up 53 seconds       0.0.0.0:8080->80/tcp   laughing_ptolemy
```

* 接著在 CentOS 中瀏覽 `127.0.0.1:8080/a.html`，看能不能瀏覽網頁

### 設置環境變數
* 參照：[docker run | Docker Documentation](https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file)
* `docker run --env VAR1=value1 --env VAR2=value2 ubuntu env | grep VAR`
  * `-e` 或 `-env` : 將系統環境變數傳到 docker 容器中
```
VAR1=value1
VAR2=value2
```

# 其它
* Docker 家目錄：`/var/lib/docker`
  * image 存放在 `/image` 的子目錄
* `docker --help`：查看相關說明

# Docker File
* 能將 docker 指令記錄在一個檔案，之後會再講解