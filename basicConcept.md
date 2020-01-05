# Basic Concept

## Data Center Multi-Tier Model Topology

> ref: [Cisco Data Center Infrastructure 2.5 Design Guide - Data Center Multi-Tier Model Design [Design Zone for Data Center Networking] - Cisco](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Data_Center/DC_Infra2_5/DCI_SRND_2_5a_book/DCInfra_2a.html)

![](./img/143311.png)

1. Core
   - e.g. 電算中心
2. Aggregation/Distribution
   - Layer 3
3. Access
   - Layer 2
   - it can setup policy

- If the scale of enterprise/organization isn't large, we may combine core and aggregation layer.
- Aggregation 連接 Core 的 Port 為 Layer3 (有IP)；連接 Access 的 Port 為 Layer2
- Loop
  - We use SPT(Spanning Tree) to avoid loop. (block logically)
  - We only care about loop in layer2, not in layer3.
- Ether Channel
- L3 Switch can have virtual port.
- 這種情況下最怕單一節點錯誤，因此會在 Aggregation 區塊設置多台 Switch