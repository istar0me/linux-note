# Eve-NG

- [Eve-NG](#eve-ng)
  - [Eve-NG 預設帳密](#eve-ng-%e9%a0%90%e8%a8%ad%e5%b8%b3%e5%af%86)
    - [R1](#r1)
    - [R2](#r2)
    - [Wireshark](#wireshark)
  - [Configuration](#configuration)
    - [將設定檔儲存到 Running Config](#%e5%b0%87%e8%a8%ad%e5%ae%9a%e6%aa%94%e5%84%b2%e5%ad%98%e5%88%b0-running-config)
    - [將全部的 running config 清除](#%e5%b0%87%e5%85%a8%e9%83%a8%e7%9a%84-running-config-%e6%b8%85%e9%99%a4)
    - [重新開機、逾時登出](#%e9%87%8d%e6%96%b0%e9%96%8b%e6%a9%9f%e9%80%be%e6%99%82%e7%99%bb%e5%87%ba)
  - [雜記](#%e9%9b%9c%e8%a8%98)
    - [Upload image to EVE-ng](#upload-image-to-eve-ng)
    - [configure ssh server on Tiny Core](#configure-ssh-server-on-tiny-core)

## Eve-NG 預設帳密

- terminal:
  - username: root
  - password: eve
- web client
  - username: admin
  - password: eve

- add new lab
  - typing name `test1`
- add node
  - select template as `Cisco IOL`
  - select L3_ADVENTERPRISEK9_M_15.4_2T.bin
  - select Number of nodes to add as 2
- more actions
  - start
- double click to enable putty(ssh)

### R1

```
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#inter e0/0
R1(config-if)#ip addr 12.1.1.1 255.255.255.0
R1(config-if)#no shutdown

R1(config-if)#enable secret cisco # 設定 privileged mode 密碼

# 設定 telent
R1(config)#line vty 0 4
R1(config-line)#password ccna # 設定 ssh 的密碼
R1(config-line)#transport input telnet # 允許 Telnet 的傳輸（這在 Emulator 中才會出現這個問題）
R1(config-line)#login
R1(config)#exit
```

### R2

```
Router>enable
Router#configure terminal
Router(config)#hostname R2
R2(config)#inter e0/0
R2(config-if)#ip addr 12.1.1.2 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#enable secret cisco # 設定 privileged mode 密碼

Router(config-if)#do telnet 12.1.1.1 # telnet 進行連線
# 再輸入密碼 ccna
```

### Wireshark

- telnet 在輸入文字時，一開始會先回傳到 server，再由 server 傳回終端機中顯示，因此封包會出現兩次
- telnet 不安全，因為是明文的傳輸，所以能看到密碼

## Configuration

| Name          | Stored in | Purpose      | Notes                                      |
| ------------- | --------- | ------------ | ------------------------------------------ |
| Running Conf. | DRAM      | Testing      | Data will disappear when the power is off. |
| Startup Conf. | NVRAM     | Stablization |

### 將設定檔儲存到 Running Config

1. `copy run`
2. `copy running-config sta`
3. `copy running-startup-config`
4. `write` # 不是全部平台都能使用

### 將全部的 running config 清除

1. `write er`
2. `write erase`

### 重新開機、逾時登出

- `reload`：重新開機
- `line console 0`：連線到第 0 個接口
- `exec-timeout 0 5`：如果畫面 5 秒沒有進行操作，會自動進行登出
  - 搭配 privileged mode 的密碼，安全性會更佳
  - 取消此功能只需要在前面加上 `no`，如 `no exec-timeout`
  - 如果需要同時管理多台電腦，可考慮將此功能取消
- `logging synchronous`：
  - 有時設定完並不會立即生效，可能要過幾秒鐘才會有輸出；若此時在輸入指令，可能會被中斷。此指令能避免此情況發生。
  - 可縮寫成 `logging sync`

## 雜記

- 不小心輸入錯指令時，會跳出 `Translating "<command>"...domain server`，此時不能輸入其他指令，因此可以按下 Ctrl + Shift + 6 中斷
  - 當系統找不到指令，會去 domain server 找，因此能透過 `no ip domain-lookup` 避免去 DNS 找

---

- [HowTo create own Linux host image](https://www.eve-ng.net/documentation/howto-s/106-howto-create-own-linux-image)
- 下載 `linux-tinycore-6.4.tar.gz` 壓縮的映像檔
- 利用 ssh 上傳這個檔案
  - `service sshd start`：啟動 sshd 服務
  - `service sshd status`：查訊 sshd 狀態
- 下載 Winscp 並安裝
- 連接進去，並切換到 `/opt/unetlab/addons/qemu` 中，應該會看到 vios 兩個資料夾，此時把檔案 copy 到這個資料夾
- 解壓縮 `tar xzvf linux-ubuntu...`

- add new lab
  - typing name `test1`
- add node
  - select template as `Cisco IOL`
  - select L3_ADVENTERPRISEK9_M_15.4_2T.bin
  - select Number of nodes to add as 2
- more actions
  - start
- double click to enable putty(ssh)

- `tcpdump -nni eth0 icmp`
  - `-i`: interface
  - `-n`: 不解析
- `ettercap -G`
  - `-G`: graphical (interface)
- sniff remote connection
- target1: 1.1
- target2: 1.3

- 能夠在模擬的環境中先進行測試，之後只要在現場重複同樣的動作即可按圖施工

### Upload image to EVE-ng

- [HowTo create own Linux host image](https://www.eve-ng.net/documentation/howto-s/106-howto-create-own-linux-image)

### configure ssh server on Tiny Core

- [Configure SSH Server on Tiny Core Linux using openSSH – IoT Bytes](https://iotbytes.wordpress.com/configure-ssh-server-on-microcore-tiny-linux/)

```
netstat -tunlp | grep 22
sudo adduser user
```