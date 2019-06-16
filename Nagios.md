- [監控軟體](#%E7%9B%A3%E6%8E%A7%E8%BB%9F%E9%AB%94)
- [系統監控](#%E7%B3%BB%E7%B5%B1%E7%9B%A3%E6%8E%A7)
  - [目的](#%E7%9B%AE%E7%9A%84)
  - [配置圖與環境設置](#%E9%85%8D%E7%BD%AE%E5%9C%96%E8%88%87%E7%92%B0%E5%A2%83%E8%A8%AD%E7%BD%AE)
  - [常見工具](#%E5%B8%B8%E8%A6%8B%E5%B7%A5%E5%85%B7)
- [Nagios](#nagios)
  - [直接監控 安裝與測試](#%E7%9B%B4%E6%8E%A5%E7%9B%A3%E6%8E%A7-%E5%AE%89%E8%A3%9D%E8%88%87%E6%B8%AC%E8%A9%A6)
    - [排錯](#%E6%8E%92%E9%8C%AF)
  - [nagios-plugins](#nagios-plugins)
  - [間接監控 安裝與測試](#%E9%96%93%E6%8E%A5%E7%9B%A3%E6%8E%A7-%E5%AE%89%E8%A3%9D%E8%88%87%E6%B8%AC%E8%A9%A6)
- [參考資料](#%E5%8F%83%E8%80%83%E8%B3%87%E6%96%99)
- [Nagios Plugins](#nagios-plugins)
    - [被控端(192.168.56.104)](#%E8%A2%AB%E6%8E%A7%E7%AB%AF19216856104)
    - [主控端(192.168.56.102)](#%E4%B8%BB%E6%8E%A7%E7%AB%AF19216856102)
      - [排錯：主控端出現 connect to address 錯誤⋯⋯](#%E6%8E%92%E9%8C%AF%E4%B8%BB%E6%8E%A7%E7%AB%AF%E5%87%BA%E7%8F%BE-connect-to-address-%E9%8C%AF%E8%AA%A4%E2%8B%AF%E2%8B%AF)
  - [手動設置監控內容](#%E6%89%8B%E5%8B%95%E8%A8%AD%E7%BD%AE%E7%9B%A3%E6%8E%A7%E5%85%A7%E5%AE%B9)
    - [備註](#%E5%82%99%E8%A8%BB)
- [NSClient++](#nsclient)
  - [Windows 上安裝 NSClient++ 過程](#windows-%E4%B8%8A%E5%AE%89%E8%A3%9D-nsclient-%E9%81%8E%E7%A8%8B)

---

# 監控軟體
* zabbix : 功能較強大
    * 如服務壞掉，傳統上只能透過 E-mail 通知。但透過其中的二次開發功能，能寫出透過第三方通訊軟體（如 Line）發送通知的功能。
* nagios : 較簡單，本學期所要學的
    * 有分作直接監控、間接監控
        * 直接監控：監視器與被控端直接連接（能透過 ping, curl 等指令確認）
        * 間接監控：有些系統資源沒辦法直接量測，因此派一個 agent 在被控端偵測，再回傳給監視端

* 老師電腦環境配置：
  * 主控端機器：7.0-1；IP：192.168.56.10**3**
  * 被控端機器：7.0-2；IP：192.168.56.10**2**

# 系統監控
## 目的
* 管理大量的電腦

## 配置圖與環境設置
![](media/W11_nagios_environment.jpg)
* 若主控端與被控端都是 Linux（圖上半部），雙方都需安裝  `nrpe`
* 若主控端為 Linux, 被控端為 Windows（圖下半部）， 被控端要安裝 `NSClock++`
* 兩台機器，皆設置為 NAT + Host-only
* 筆者電腦環境配置：
  * 主控端機器：centos test2 (RS1)；IP：192.168.56.10**2**
  * 被控端機器：centos test2 (RS2)；IP：192.168.56.10**4**

## 常見工具
* nagios（今天的主題）
  * 有分成伺服器與客戶端
  * 伺服器：一般都是在 Linux 上
  * 客戶端：可在 Linux 與 Windows 上查看
* zabbix

# Nagios
* 有分成兩種監控模式：
  1. [直接監控](#%E7%9B%B4%E6%8E%A5%E7%9B%A3%E6%8E%A7-%E5%AE%89%E8%A3%9D%E9%81%8E%E7%A8%8B%E8%88%87%E6%B8%AC%E8%A9%A6)
     * 不需要客戶端的軟體，用一些簡單的方式去判斷，但也只能掌握基本的資訊
     * 可透果主機是否 alive 或 server 是否有成功執行判斷（例如透過 ping 來測試主機是否上線中）
  2. [間接監控](#%E9%96%93%E6%8E%A5%E7%9B%A3%E6%8E%A7-%E5%AE%89%E8%A3%9D%E8%88%87%E6%B8%AC%E8%A9%A6)
     * 週期性（固定一段時間）去抓取，能掌握 CPU、硬碟空間、記憶體等較詳細的資訊
     * 如 NRPE(Nagios Remote Plugin Executor)
     * 可參考：[nagios + nrpe 監控遠端 Linux 主機的CPU或硬碟空間 | SSORC.tw](https://ssorc.tw/1122)

## 直接監控 安裝與測試
![](media/W11_nagios_command.jpg)

1. Server 上輸入以下指令安裝相關套件：
```
yum install epel-release # 安裝第三方軟體資料庫
yum install httpd nagios* nrpe
systemctl start httpd
systemctl start nagios
htpasswd -c /etc/nagios/passwd nagiosadmin
systemctl restart httpd
systemctl restart nagios
```

2. 接著進入管理端介面查看是否建置成功
   * 管理端 IP：`主控端IP/nagios`，如 `192.168.56.102/nagios`
   * 帳號：nagiosadmin
   * 密碼：輸入 `htpasswd -c /etc/nagios/passwd nagiosadmin` 所設定的密碼（這邊筆者設定為 `centos`）

3. 再編輯配置檔
   * 註：nagios 設定檔位置：`/etc/nagios`
```
cd /etc/nagios/objects
vim localhost.cfg
```

4. 複製定義當作範本
```
# Define a host for the local machine

define host {

    use                     linux-server            ; Name of host template to use
                                                    ; This host definition will inherit all variables that are defined
                                                    ; in (or inherited by) the linux-server host template definition.
    host_name               localhost
    alias                   localhost
    address                 127.0.0.1
}
```

5. 貼上一份並修改 host_name, alias, address
```
# Define a host for the local machine

define host {

    use                     linux-server            ; Name of host template to use
                                                    ; This host definition will inherit all variables that are defined
                                                    ; in (or inherited by) the linux-server host template definition.
    host_name               web1
    alias                   web1
    address                 192.168.56.104
}
```

6. 重啟 Nagios
```
systemctl restart nagios
```

7. 查看 nagios 管理端介面是否多了 web1

### 排錯
* 如果在 Host 頁面下連 localhost 都不能 ping 的話，且出現以下問題：
```
(No output on stdout) stderr: execvp(/usr/lib64/nagios/plugins/check_ping, ...) failed. errno is 2: No such file or directory
```
* 原因：安裝 nagios 套件時，未加上後方的 `*`
  * `yum install httpd nagios* nrpe`
* 可透過安裝 Nagios 相關套件來排除：
```
yum install -y nagios-plugins*
```
* 參考：[Missing Nagios plugins in CentOS 7 – The Accidental Developer](https://osric.com/chris/accidental-developer/2016/12/missing-nagios-plugins-in-centos-7/)

## nagios-plugins
* `rpm -qa | grep nagios-plugins`：查看 nagios plugins 列表
* `rpm -ql nagios-plugins-http`：查看 nagios-plugins-http 檔案位置
  * 註：會印出`/usr/lib64/nagios/plugins/check_http`
* `./check_http -I 192.168.56.104 -p 80`：測試是否能在 80 port 連接到 192.168.56.104 (主控端)
* `./check_ssh 192.168.56.104`：測試是否能透過 ssh 連接到


## 間接監控 安裝與測試
* 在被控端上輸入：
```sh
yum install epel-release nrpe nagios-plugins*
vim /etc/nagios/nrpe.cfg # 新增 allow-hosts 127.0.0.1, 192.168.56.102 (主控端 IP)
systemctl start nrpe
```
* `cat /etc/nagios/nrpe.cfg`：可查看 __
* `gedit /etc/nagios/objects/commands.cfg`
```
define command {
    command_name    check_nrpe
    command_line    $USER1/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```
* 說明：
  * `$USER1`：對應到 __

* `gedit /etc/nagios/objects/localhost.cfg`
```
define service {
    use                     local-service
    host_name               web1
    service_description     check users
    check_command           check_nrpe!check_users
    notifications_enabled   0

}
```

# 參考資料
* [系統監看工具Nagios簡易安裝教學 - 行政院國家資通安全會報技術服務中心](http://www.nccst.nat.gov.tw/ArticlesDetail?lang=zh&seq=1106)

# Nagios Plugins

### 被控端(192.168.56.104)
* 先確認參數是否對應到相對應的 plugins 路徑
```sh
vim /etc/nrpe.d/lcgdm-common.cfg
```
> * 至少需要這兩行：
> ```
> command[check_cpu]=/usr/lib64/nagios/plugins/lcgdm/check_cpu
> command[check_network]=/usr/lib64/nagios/plugins/lcgdm/check_network
> ```

* 先在被控端上測試 plugins 是否能執行
```sh
/usr/lib64/nagios/plugins/lcgdm/check_cpu # 先在被控端上測試 check_cpu
/usr/lib64/nagios/plugins/lcgdm/check_network # 先在被控端上測試 check_network
systemctl start nrpe
```

### 主控端(192.168.56.102)
* 再到主控端上測試，能否透過 nrpe 執行擷取被控端上執行後的訊息
```sh
cd /usr/lib64/nagios/plugins # 切換到 plugins 資料夾
./check_nrpe -H 192.168.56.104 -c check_cpu # 透過 nrpe 代理人在被控端執行 check_cpu，再將資訊回傳給主控端
./check_nrpe -H 192.168.56.104 -c check_network
```

* 新增指令：`gedit /etc/nagios/objects/commands.cfg`
```
define command {
    command_name    check_nrpe
    command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```

* 新增服務：`gedit /etc/nagios/objects/localhost.cfg`
```
define service {
    use                     local-service
    host_name               web1
    service_description     check cpu
    check_command           check_nrpe!check_cpu
    notifications_enabled   0

}

define service {
    use                     local-service
    host_name               web1
    service_description     check networks
    check_command           check_nrpe!check_network
    notifications_enabled   0

}
```

* 接著再重開：`systemctl restart nagios`
* 再去 web 端介面查看 host:web1 是否新增 `check_cpu` 與 `check_network` service

#### 排錯：主控端出現 connect to address 錯誤⋯⋯
* Q: 主控端出現 connect to address 192.168.56.102 port 5666 錯誤
* A:
  1. 先在被控端輸入 `netstat -tunlp | grep 5666` 確認是否有開啟 5666 port
  2. 如果沒出現，在被控端上輸入 `systemctl start nrpe`

## 手動設置監控內容
* 有時候要監控的資訊預設可能沒有提供，因此可以手動撰寫來達成
* 在**被控端**輸入：`gedit /usr/lib64/nagios/plugins/check_waiting_connect`
```sh
# !/bin/bash
STATE_OK=0
STATE_CRITICAL=2
W=`netstat -an | wc -l` # 統計有幾行(等同於有多少機器連線此電腦)，並將值賦予給 W
if [ $W -le 1000 ]; then
    echo "OK, waiting connections < 1000 low"
    exit $STATE_OK;
else
    echo "WARNING, waiting connections > 1000 high"
    exit $STATE_CRITICAL
fi
```

### 備註
> * 若回傳不同的 state，會在 Nagios 的 web 介面端顯示不同的狀態
> 0:OK
>   * 1: Warning
>   * 2: Critical
>   * 3: Unknown
> 
> * 以下兩行相等，都會先執行裡面的內容
> ```
> echo `netstate -an | wc -l`
> echo $(netstate -an | wc -l)
> ```

* 被控端新增參數：`vim /etc/nrpe.d/lcgdm-common.cfg`
```sh
command[check_waiting_connect]=/usr/lib64/nagios/plugins/check_waiting_connect
```

* 在被控端中測試（記得要設定執行的權限）
```sh
chmod +x /usr/lib64/nagios/plugins/check_waiting_connect
/usr/lib64/nagios/plugins/check_waiting_connect
```

* 在主控端中測試
```sh
cd /usr/lib64/nagios/plugins # 切換到 plugins 資料夾
./check_nrpe -H 192.168.56.104 -c check_waiting_connect
```

* 新增服務：`gedit /etc/nagios/objects/localhost.cfg`
```
define service {
    use                     local-service
    host_name               web1
    service_description     check waiting connect
    check_command           check_nrpe!check_waiting_connect
    notifications_enabled   0

}
```

* 重啓 nagios：`systemctl restart nagios`
* 在 nagios web 端介面查看是否有多出一個 check_waiting_connect 的 service

# NSClient++
* 若主控端為 Linux, 被控端為 Windows， 被控端要安裝 `NSClock++`

## Windows 上安裝 NSClient++ 過程
* Configuration 全部打勾，Allowed Host 輸入主控端 IP（如筆者環境為 `192.168.56.102`
* 在監控端中編輯 `/etc/nagios/nagios.cfg`，取消以下此行的註解，開放讓 Windows 使用
```
cfg_file=/etc/nagios/objects/windows.cfg
```

* `vim /etc/nagios/objects/windows.cfg`
  * 將 define host 的 address 改成 `192.168.56.1`