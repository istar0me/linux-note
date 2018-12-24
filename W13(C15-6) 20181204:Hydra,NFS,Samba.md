# C15 網頁伺服器架設
## C15-6 存取控制

### 透過 Hydra 破解登入視窗
* 這邊所破解的為 HTTP GET 模式

# 課堂補充
## NFS
* [RHEL / CentOS 7 安裝 NFS Server](https://www.phpini.com/linux/rhel-centos-7-install-nfs-server)

* TODO：詳細步驟

## Samba
* 本週沒講，會在下週補上

# 雜記
* 防護可以實現在不同的層級上
* 暫定 12/25 期末考
    * 安裝 CentOS
    * 網路設定：network + NetworkManager
    * 測試：SSH + FTP + WWW + Apache + PHP + MySQL Server
    * 期末可以 Open Book
    * 如果考得不是很理想，可能會在下週再補考

# 期末考
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