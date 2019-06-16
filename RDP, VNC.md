# 遠端桌面（RDP, VNC）
* 建議同學用 putty, secureCRT, XShell 連線到客機
* Windows 上能透過 `netstat -an` 查看有哪些 Port 被佔用
    * `netstat -an | findstr 3389` 能過濾 3389 號 Port
    * `findstr` 類似於 Linux 中的 `grep`
    * 3306 號 Port 通常用於 MySQL
    * 3389 號 Port 通常用於遠端桌面(VNC)
* RDP：Windows 有內建 Mstsc 連線軟體
    * 遠端桌面除了遠端操作外，也可以用於教學用途
    > mstsc 只有支援 RDP 協定，並不支援 VNC 協定，需另外下載第三方的軟體
    * TODO : 在 Windows 主機上測試 RDP
* `0.0.0.0` 所代表的意思
    * TODO : 研究 0.0.0.0
    * 機器未開機好（尚未申請伺服器），一開始的位址
    * 機器任一介面的位址
    * 如果路由表其他的規則都找不到時，則會去使用 `0.0.0.0` 的路由
* Server 通常會綁定一組特定 IP，如果在本機上 `127.0.0.1` 能連線，但在其他電腦上這個 IP 就連不上