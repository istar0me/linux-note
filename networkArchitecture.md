# Network Architecture

- [Network Architecture](#network-architecture)
  - [網路架構](#%e7%b6%b2%e8%b7%af%e6%9e%b6%e6%a7%8b)
    - [交換機在不同 Vlan 進行封包交換](#%e4%ba%a4%e6%8f%9b%e6%a9%9f%e5%9c%a8%e4%b8%8d%e5%90%8c-vlan-%e9%80%b2%e8%a1%8c%e5%b0%81%e5%8c%85%e4%ba%a4%e6%8f%9b)
    - [為什麼對外的接口一定要採用 Router，而不是 Layer 3 Switch？](#%e7%82%ba%e4%bb%80%e9%ba%bc%e5%b0%8d%e5%a4%96%e7%9a%84%e6%8e%a5%e5%8f%a3%e4%b8%80%e5%ae%9a%e8%a6%81%e6%8e%a1%e7%94%a8-router%e8%80%8c%e4%b8%8d%e6%98%af-layer-3-switch)

## 網路架構

![](img/143311.png)

1. Core（核心層）：快速轉發
   - 機房放置位置
2. Aggregate（匯聚層）：路由策略
   - 在此會有 SVI(Switch Vlan Interface) 接口
3. Access（接入層）：提供接入
   - 提供存取網路的服務、監控使用者流量

- 為了節省預算，有時 Core 與 Aggregate 會合併
- 如果經費允許的話，核心路由器會建議至少有兩台 Router，因為能確保 HA(High Availbility)、Error tolerance 與 Load balance
- 此外，若有兩台 Router，一台能專門處理負載較大的 VPN，另一台就能專注處理 Internet 的封包
- EtherChannel：同一層之間的連線，最多能將 8 個 Port 合併在一起
- Loop 存在優點：代表有備援路線；缺點：廣播風暴
  - 能透過 STP(Spanning Tree Protocol) 來解決此問題

### 交換機在不同 Vlan 進行封包交換

1. 創造 SDI 接口（推薦做法）
2. 單臂路由（邏輯分割）

### 為什麼對外的接口一定要採用 Router，而不是 Layer 3 Switch？

- A：因為 Router 才有 NAT(Network Address Translation) 功能，因為 Internet 需要有 Public IP
