# Samba
* 參考：[鳥哥的 Linux 私房菜 -- SAMBA 伺服器](http://linux.vbird.org/linux_server/0370samba.php)
* 全名：Server Message Block, SMB
* 前置動作
    * 注意安裝前的網路配置是否正常
```shell
yum install -y samba # 安裝 samba
su
cd /
mkdir -p sharedpath/
chmod 777 sharedpath/ # 注意：讓所有帳號都能存取
cd sharedpath
touch {a..d}
```

```shell
vim /etc/samba/smb.conf
```

* 針對 `/etc/samba/smb.conf` 配置：
```shell
[global]
    workgroup = WORKGROUP

[myshare]
    path = /sharedapth
    read only = No
    browsable = Yes
```

* 測試 samba 指令有無問題
```shell
testparm
```

* 啟動 samba
```shell
systemctl start smb # 啟用 Samba Server 的服務
systemctl start nmb # 在 Linux 上啟用 NetBIOS 協定
```

* 建立 Samba 中新使用者 user
```shell
pdbedit -a user # 建立(add) Samba 使用者
# 接著輸入密碼
```

* 在 Windows 開始中輸入：`\\虛擬機 IP 位置`，接著輸入上一步的帳號與密碼
    * 清除 SMB 登入資訊：在終端機輸入 `net use * /delete`

# 新增 mary 使用者，但無法讀取 /sharedpath 資料夾

```shell
useradd mary
passwd mary
mkdir /home/mary
chown mary:mary mary
pdbedit -a mary
# 再輸入 mary 的密碼
```

修改 /etc/samba/smb.conf
```shell
[myshare]
    read only = yes
    valid users = user tom # 以空白間隔使用者名稱
```
* 測試 SMB Server 指令是否有誤
```shell
testparm
```
* 重啟 SMB Server
```shell
systemctl restart smb
```
* 此時若以 mary 登入，無法查看 `/sharedpath` 資料夾的檔案（但 user 跟 tom 可以）

# Siege 壓力測試工具
* 可以用來測試 HTTP Server
* 支援 GET 與 POST 的請求

## 安裝 Siege
```shell
wget http://download.joedog.org/siege/siege-latest.tar.gz
tar -xzvf siege-latest.tar.gz
./configure; make
make instal
```

* 測試老師 E321 教室中的主機
```shell
siege -c 100 -r 10 -b http://192.168.60.133/index.htm
```
* `-c` : concurrent 並發量
* `-r` = `-reps=NUM`：repeat 重複次數
* `-b` = `-benchmark`：跑分

# Squid
* 功能：作為 Proxy Server，用來存放網頁的快取，可以加速已經瀏覽過的網頁
* 參考：[CentOS 7 安裝 squid proxy server 及基本設定 | MIS的背影](https://blog.pmail.idv.tw/?p=13663)
* Proxy Server 預設的埠號為 3128

```shell
yum install squid
cd /var/spool/squid
du -s -h
```

修改 `/etc/squid/squid.conf` 檔案，以封鎖特定網站
```
acl aaa dstdomainu
http_access deny domain
```

* 也可以將要封鎖的網頁寫入檔案中（這邊檔案為 `/etc/squid/urldeny.txt`）
```
google.com
nqu.edu.tw
```

```shell
acl denyurl url_regex "/et/squid/urldeny.txt"
http_access deny
```

* 如果 Squid 在啟動時失敗，可以嘗試：
```shell
yum install -y openssl-devel
```