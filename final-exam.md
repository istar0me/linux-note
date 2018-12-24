# 期末考統整
1. 106-2 W3：重新安裝 Linux（佔 10 分）
> 補充：W2(C11)：安裝 Guest Addition
2. W3(C11-2)：遠端主機登入 with SSH、儲存雙方密碼 with scp or `ssh-copy-id` 
3. W6(C13)：將 `NetworkManager` 改為 `network` 的網路服務（佔 10 分，大魔王）
4. W7(C13-4) + C16：架設 FTP Server with ftpd
5. W9(C14)：架設網頁伺服器 With httpd
6. W11(C15)：架設 HTTP Server with Apache、MySQL
    * 建議先關閉防火牆與 SELinux
7. W13：NFS
8. W14：Samba, Siege, Squid
9. W15：DHCP server

# 1. install CentOS
* [照影片操作](https://www.youtube.com/watch?v=TOjsp3RVIB4)
* 映像檔：[CentOS-7-livecd-x86_64.iso](https://buildlogs.centos.org/centos/7/isos/x86_64/CentOS-7-livecd-x86_64.iso)

# 1-1. 安裝 Guest Addition
* [照影片操作](https://www.youtube.com/watch?v=-vHTAlvF0z4)
* 目的：為了複製剪貼簿、動態調整視窗
* 須先安裝 SSH server
* 虛擬機中的終端機輸入
```shell
systemctl start sshd
netstat -anp | grep sshd
ifconfig # 記住虛擬機的 IP
```

* 在主機上輸入
```shell
# 先透過 ssh 軟體（如 putty）連接到虛擬機 IP

rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

yum install -y perl gcc dkms kernel-headers make bzip2
    # 如果卡住，輸入 kill -9 <PID>
    # 並重新輸入：
    # yum install -y perl gcc dkms kernel-headers make bzip2
    # 等待安裝⋯⋯

echo $(uname -r)
    # 輸出：3.10.0-123.el7.x86_64
    # 在 Google 中搜尋「kernel devel 3.10.0-123.el7.x86_64」，並複製對應的 rpm 網址

yum install -y https://buildlogs.centos.org/c7-updates/kernel/3.10.0-123.el7/20140630120647/kernel-devel-3.10.0-123.el7.x86_64.rpm
    # 如果上面輸出並非 3.10.0-123.el7.x86_64，網址需改成其他檔案
```
* VirtualBox 選單 -> 裝置 -> 插入 Guest Additions CD 映像檔
* 掛載完成後，執行並等待安裝完成
* 關機
* VirtualBox -> 設定值 -> 進階 -> 共用剪貼簿改為雙向

# 2. 遠端主機登入 with SSH、儲存雙方密碼
* 第一台機器輸入：
```shell
systemctl start sshd

netstat -anp | grep sshd # 確認是否開啟

ifconfig # 記住第一台的 IP
```

* 再製第二台機器，並輸入：
```shell

```