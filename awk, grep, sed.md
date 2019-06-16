- [Linux 三劍客](#linux-%E4%B8%89%E5%8A%8D%E5%AE%A2)
  - [grep, awk](#grep-awk)
  - [sed (string editor)](#sed-string-editor)
  - [awk](#awk)
  - [匹配過程](#%E5%8C%B9%E9%85%8D%E9%81%8E%E7%A8%8B)

---

# Linux 三劍客
> `cat >> fileName` 能新增一個名為 `fileName` 的檔案，之後輸入檔案內容，再按 Ctrl + C 停止輸入

* linux 三劍客：`grep`, `sed`, `awk`
    * `grep`：查找
    * `sed`：行編輯器
    * `awk`：文字處理工具

## grep, awk
* `grep -i` : 不分大小寫
* `grep -v` : 排除特定字串
* `grep -o` : 只顯示特定字串（only-matching）
* `awk '{print $2}'` : 取出第二欄的 IP 位址

```
[root@localhost security]# ifconfig enp0s8
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::a00:27ff:fe6d:b9a3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6d:b9:a3  txqueuelen 1000  (Ethernet)
        RX packets 570  bytes 84856 (82.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 311  bytes 51628 (50.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@localhost security]# ifconfig enp0s8 | grep inet
        inet 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::a00:27ff:fe6d:b9a3  prefixlen 64  scopeid 0x20<link>
[root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6
        inet 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255
[root@localhost security]# ifconfig enp0s8 | grep -o inet
        inet
[root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6 | awk '{print $2}'
192.168.56.103
```

## sed (string editor)
* ![](/media/W2_sed.jpg)
* 通常會搭配正規表達式

## awk
* awk 預設的分隔符號為空白鍵，若要更改需要加入 `-F` 參數
    * `-F` fs option defines the input field separator
    * 假設今天要取出 c : `echo 'a:b_c:d' | awk -F '[:_]' '{print $3}'`

## 匹配過程
* 字符 -> 字串 -> 表達式
* 字符
    * 特定字符：某個具體的字符
        * 如 `'a'`, `'1'`
        * 範例：`grep '1' passwd`
    * 範圍內字符：單個字符[]，其中一個符合即可
        * 如數字字符 `[0-9]`, `[259]`
            * `[0-9]` : 0~9 其中一個
            * `[259]` : 2 或 5 或 9
            * 範例：`grep '[0-9]' passwd`, `grep '[259]' passwd`
        * 如英文字母 `[a-z]`, `[A-Z]`, `[a-zA-Z]`
            * `[a-z]` : 所有小寫的字母
            * `[A-Z]` : 所有大寫的字母
            * `[a-zA-Z]` : 匹配全部字母
        * 反向字符 : `[^]`
            * 如 `[^0-9]`, `[^0]`
            * `[^0-9]` : 找到 0~9 的其中一個字元
            * `[^0]` : 找到 0 以外的一個字元
    * 任意字符
        * `.` : 代表任意一個字元
            * `[.]` 與 `\.` 代表相同的功用
            * `[.]` : 只搜尋 `.` 的字元
            * `\.` : 搭配跳脫符號，只搜尋 `.` 的字元
        * 邊界字符
            * 如 `^root`, `false$`
            * `^` : head
                * 注意與 `[^]` 的區別
                * 範例：`grep ^root passwd`
            * `$` : end
                * 注意 `$` 要放在結尾
                * 範例：`grep false$ passwd`
            * `^$` : 空白行（因為頭尾之間沒有內容）
                * 注意：空白行代表完全沒有內容，但如果中間有看不見的空白鍵，就會匹配不到
                * 若要匹配所有空白行（無論是否包含空白鍵）：`grep "^[]*$" passwd`
        * 元字符
            * 參照：[正規表示式 Regular Expression | 就是愛程式](https://atedev.wordpress.com/2007/11/23/%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%A4%BA%E5%BC%8F-regular-expression/)
            
            | 正規表示式的特定字元 | 說明             | 等效的正規表示式 |
            | -------------------- | ---------------- | ---------------- |
            | `\d`                 | 數字             | `[0-9]`          |
            | `\D`                 | 非數字           | `[^0-9]`         |
            | `\w`                 | 數字、字母、底線 | `[a-zA-Z0-9_]`   |
            | `\W`                 | 非 `\w`          | `[^a-zA-Z0-9_]`  |
            | `\s`                 | 空白字元         | `[ \r\t\n\f]`    |
            | `\S`                 | 非空白字元       | `[^ \r\t\n\f]`   |
            
            | 字元 | 說明                       | 簡單範例                                                                                                      |
            | ---- | -------------------------- | ------------------------------------------------------------------------------------------------------------- |
            | \b   | 比對英文字的邊界，例如空格 | 例如 `/\bn\w/` 可以比對 “noonday” 中的 ‘no’ ;`/\wy\b/` 可比對 “possibly yesterday.” 中的 ‘ly’         |
            | \B   | 比對非「英文字的邊界」     | 例如, `/\w\Bn/` 可以比對 “noonday” 中的 ‘on’ ,另外 `/y\B\w/` 可以比對 “possibly yesterday.” 中的 ‘ye’ |

* 字串
* 表達式