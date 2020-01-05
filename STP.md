# STP: Spanning Tree Protocol

## STP 端口狀態

- Blocking（阻塞狀態）：執行阻塞端口
- Listening（偵聽端口）：執行端口選舉
- Learning（學習狀態）：執行地址學習
- Forwarding（轉發狀態）：執行數據轉發

### STP 鏈路收斂

- 網路發生拓樸變更 ~ 鏈路重新回到穩定狀態的整個過程
- 由於 STP 存在端口狀態機，所以當鏈路發生變動，鏈路收斂需要經過一段時間。直鏈鏈路收斂需要經過 30 秒，間接鏈路收斂需經過 50 秒

### STP 增強特性

- 由於 STP 預設收斂速度緩慢，cisco 提出了以下三種：
  1. Portfast：端口加速，用於加速用戶接入時間，能加速 30 秒(blocking 直接切換到 forwarding)
  2. Uplinkfast：上聯加速，用於加速直連鏈路收斂時間，能加速 30 秒
  3. Backbonefast：骨幹加速，用於加速間接鏈路收斂時間，能加速 20 秒

### STP 實驗

```
// VPC1
ip 192.168.1.1 255.255.255.0



// VPC2
ip 192.168.1.2 255.255.255.0



// VPC3
ip 192.168.1.3 255.255.255.0



// SW1, SW2, SW3
do show spanning-tree // 能查看 STP 狀態（如 BLK）
```

```
// SW3 e0/2 使用 cisco portfast
interface e0/2
spanning-tree portfast
exit

spanning-tree uplinkfast



// SW1
spanning-tree backbonefast



// SW3
debug spanning-tree events // 查閱 spanning-tree 的狀態
int e0/2
shutdown
```