# Per VLAN Spanning Tree + (PVST+)

- 參考：[Types of Spanning Tree Protocol (STP) - GeeksforGeeks](https://www.geeksforgeeks.org/types-of-spanning-tree-protocol-stp/) 中的 PVST+ 區塊

```
// SW1, SW2, SW3
Switch(config)#do show spanning-tree summary

interface e0/0
switchport trunk encapsulation dot1q
switchport mode trunk
interface e0/1
switchport trunk encapsulation dot1q
switchport mode trunk



/* 讓 SW1 成為 vlan 10 的主根、vlan 20 的次根 */
// SW1
vlan 10
vlan 20
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root sceondary



/* 讓 SW2 成為 vlan 20 的主根、vlan 10 的次根 */
// SW2
vlan 10
vlan 20
spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary
```

- 在 SW1, SW2, SW3 外接兩個機器，分別設定 vlan 10 與 vlan 20
  - 因為是第二層的設備，因此不能使用 `trace` 指令，只能採用擷取封包的方式