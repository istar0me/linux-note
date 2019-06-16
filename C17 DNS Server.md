- [C17 DNS Server（名稱解析伺服器）](#c17-dns-server%E5%90%8D%E7%A8%B1%E8%A7%A3%E6%9E%90%E4%BC%BA%E6%9C%8D%E5%99%A8)
  - [內部 vs. 外部 DNS 的差異](#%E5%85%A7%E9%83%A8-vs-%E5%A4%96%E9%83%A8-dns-%E7%9A%84%E5%B7%AE%E7%95%B0)
- [架設 DNS Server 補充](#%E6%9E%B6%E8%A8%AD-dns-server-%E8%A3%9C%E5%85%85)
- [反向解析伺服器](#%E5%8F%8D%E5%90%91%E8%A7%A3%E6%9E%90%E4%BC%BA%E6%9C%8D%E5%99%A8)
- [作業：設置 parent DNS Server](#%E4%BD%9C%E6%A5%AD%E8%A8%AD%E7%BD%AE-parent-dns-server)

---

# C17 DNS Server（名稱解析伺服器）
* UDP, 53 port, 分為 client 與 server
    * UDP 為非連線導向式的傳輸協定
* **[攻擊1]** DNS Spoofing : 傳輸過程中封包被竄改，導向錯誤的 IP/Domain name
* **[攻擊2]** DDoS
    * 駭客將 DNS Server 的 Destination 導向為 victim(受害者)
    * ![](/media/W3_dnsServerDDoS.jpg)
* 有兩種方向
    * 正向解析 : Domain name -> IP
    * 反向解析 : IP -> Domain name
* 常見 DNS Server
    * `1.1.1.1` (CloudFlare)
    * `4.4.4.4` (Google 2rd)
    * `8.8.8.8` (Google 1th)
    * `9.9.9.9` (Quad9 by IBM)
    * `180.76.76.76` (百度)
* 還有另外一項功用：負載均衡（Load Balancer）
    * 假設有 3 位使用者，都要連上 `google.com.tw`
        * 使用者 1 會連線到主機 1（如 192.168.0.1）
        * 使用者 2 會連線到主機 2（如 192.168.0.2）
        * 使用者 3 會連線到主機 3（如 192.168.0.3）
    * 困境：目前多為動態網頁，難以確保各台主機間的資料都同步，因此適合早期的靜態網頁，近期已較少見
    * 參照：[Linux Bind負載均衡-慕課網](https://www.imooc.com/learn/723)
* 應用：阻擋外國的 IP，不提供相對應的服務
* 老師推薦課程：[Linux 智能DNS-慕課網](https://www.imooc.com/learn/768)

## 內部 vs. 外部 DNS 的差異
* 透過校內的 DNS `10.10.10.3` 會回傳 `10.10.10.20` 的 IP，而透過 Google Public DNS `8.8.8.8` 會回傳 `203.72.226.40` 的 IP
* 會有這樣的差異是因為前者採用該網域的權限管轄伺服器，後者採用外部公開的 DNS，因此產生不同的結果
* 註 : 前者的 IP `10.10.10.20` 只有在內部網路才能連線上
```
minidino-macbook:testFolder minidino$ host nqu.edu.tw 10.10.10.3
Using domain server:
Name: 10.10.10.3
Address: 10.10.10.3#53
Aliases: 

nqu.edu.tw has address 10.10.10.20
nqu.edu.tw mail is handled by 1 mail2.nqu.edu.tw.
minidino-macbook:testFolder minidino$ host nqu.edu.tw 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases: 

nqu.edu.tw has address 203.72.226.40
nqu.edu.tw mail is handled by 5 mfilter0.nqu.edu.tw.
```

* TODO : 補上描述
```
minidino-macbook:~ minidino$ dig nqu.edu.tw ns

; <<>> DiG 9.10.6 <<>> nqu.edu.tw ns
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62641
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;nqu.edu.tw.			IN	NS

;; ANSWER SECTION:
nqu.edu.tw.		259200	IN	NS	ns1.nqu.edu.tw.

;; Query time: 233 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Mar 06 11:00:20 CST 2019
;; MSG SIZE  rcvd: 57

minidino-macbook:~ minidino$ dig ns1.nqu.edu.tw

; <<>> DiG 9.10.6 <<>> ns1.nqu.edu.tw
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32891
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;ns1.nqu.edu.tw.			IN	A

;; ANSWER SECTION:
ns1.nqu.edu.tw.		259200	IN	A	10.10.10.3
ns1.nqu.edu.tw.		259200	IN	A	203.72.226.1

;; Query time: 199 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Mar 06 11:00:38 CST 2019
;; MSG SIZE  rcvd: 75
```

* `netstat tunlp | grep 53` : 確認 Port 53 是否打開

# 架設 DNS Server 補充
* 參照：[在CentOS 7架設DNS服務及設定Bind chroot環境 | Kevin Linul 網路日記本](http://blog.kevinlinul.idv.tw/?p=188)

# 反向解析伺服器
* ![](/media/W4_named.jpg)
* 建置成功後，輸入 `dig -x 要查詢的IP @DNSServer` 測試是否成功
```
[root@localhost named]# dig -x 192.168.56.100 @127.0.0.1

; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> -x 192.168.56.100 @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61536
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;100.56.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
100.56.168.192.in-addr.arpa. 86400 IN	PTR	ftp.test.com.

;; AUTHORITY SECTION:
56.168.192.in-addr.arpa. 86400	IN	NS	dns.test.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Mar 13 11:14:55 CST 2019
;; MSG SIZE  rcvd: 100

[root@localhost named]# dig -x 192.168.56.1 @127.0.0.1

; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> -x 192.168.56.1 @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35642
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;1.56.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
1.56.168.192.in-addr.arpa. 86400 IN	PTR	www.test.com.

;; AUTHORITY SECTION:
56.168.192.in-addr.arpa. 86400	IN	NS	dns.test.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Mar 13 11:15:02 CST 2019
;; MSG SIZE  rcvd: 98
```

# 作業：設置 parent DNS Server
* ![](/media/W4_parent_DNS_server.jpg)
* 虛擬環境中開三台機器
* 不能使用 DHCP，因此需要手動設定
    * 指令：`ip addr add 192.168.10.254/24 brd + dev enp0s8`