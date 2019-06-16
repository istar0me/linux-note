- [萬用字元 vs. 正規表達式](#%E8%90%AC%E7%94%A8%E5%AD%97%E5%85%83-vs-%E6%AD%A3%E8%A6%8F%E8%A1%A8%E9%81%94%E5%BC%8F)
  - [使用時機](#%E4%BD%BF%E7%94%A8%E6%99%82%E6%A9%9F)
- [萬用字元（萬用字元）](#%E8%90%AC%E7%94%A8%E5%AD%97%E5%85%83%E8%90%AC%E7%94%A8%E5%AD%97%E5%85%83)
  - [常見萬用字元](#%E5%B8%B8%E8%A6%8B%E8%90%AC%E7%94%A8%E5%AD%97%E5%85%83)
    - [範例](#%E7%AF%84%E4%BE%8B)
    - [shell meta（元字符）](#shell-meta%E5%85%83%E5%AD%97%E7%AC%A6)
- [正規表達式](#%E6%AD%A3%E8%A6%8F%E8%A1%A8%E9%81%94%E5%BC%8F)
  - [常見語法](#%E5%B8%B8%E8%A6%8B%E8%AA%9E%E6%B3%95)
  - [重複](#%E9%87%8D%E8%A4%87)
  - [任意字符串：`.*`](#%E4%BB%BB%E6%84%8F%E5%AD%97%E7%AC%A6%E4%B8%B2)
  - [範例](#%E7%AF%84%E4%BE%8B-1)
  - [應用 1 : 篩選 QQ 號](#%E6%87%89%E7%94%A8-1--%E7%AF%A9%E9%81%B8-qq-%E8%99%9F)
  - [應用 2 : 篩選中國身分證號](#%E6%87%89%E7%94%A8-2--%E7%AF%A9%E9%81%B8%E4%B8%AD%E5%9C%8B%E8%BA%AB%E5%88%86%E8%AD%89%E8%99%9F)
  - [結論](#%E7%B5%90%E8%AB%96)
- [雜記](#%E9%9B%9C%E8%A8%98)
- [參考資料](#%E5%8F%83%E8%80%83%E8%B3%87%E6%96%99)

---

# 萬用字元 vs. 正規表達式

## 使用時機
* 萬用字元：搜尋檔案的名稱
* 正則表達式：搜尋檔案的內文或標準輸入

# 萬用字元（萬用字元）
* 英文 : wildcard character
* 可與 `find`, `ls`, `cp`, `mv` 指令搭配使用

## 常見萬用字元
萬用字元|含義|輸入|輸入介紹|輸出
-|-|-|-|-
`*`|匹配零或多個字元|`a*b`|代表 a 跟 b 之間內含有任意長度的字元，a 開頭且 b 結尾|`a012b` `aabxcb` `ab` ~~`ab012`~~(*1)
`?`|匹配任意一個字元|`a?b`|代表 a 跟 b 之間內含有任意一個字元|`aab` `abb` `acb` `a0b`
`[list]`|匹配 list 中的任意單一字元|`a[xy]b`|代表 x 或 y 選一個|`axb`, `ayb`
`[c1-c2]`|匹配 c1-c2 中的任意單一字元 如：[0-9] [a-z]|`a[0-9]b`|0與9之間必須也只能有一個字元|`a0b`, `a1b`... `a9b`
擴展：`{string1, string2}`|匹配 sring1 或 string2 (或更多)其一字串|`a{abc,xyz,123}b`|-|`aabcb`, `axyzb`, `a123b`

* *1：ab012 由於結尾不是 b，因此匹配不到

### 範例
* `*` : 匹配 0 個或多個字元
    * 如 `a*b` 代表 a 跟 b 之間內含有任意長度的字元，a 開頭且 b 結尾
```sh
minidino-macbook:Desktop minidino$ touch ab a012b aabxcb
minidino-macbook:Desktop minidino$ ls a*b
a012b	aabxcb	ab
minidino-macbook:Desktop minidino$ touch ab012
minidino-macbook:Desktop minidino$ ls a*b
a012b	aabxcb	ab # ab012 由於結尾不是 b，因此匹配不到
```

* `?` : 匹配任意一個字元
```sh
minidino-macbook:testFolder minidino$ ls
a012b	aabxcb	ab	ab012
minidino-macbook:testFolder minidino$ ls ??
ab
minidino-macbook:testFolder minidino$ ls ???
ls: ???: No such file or directory
minidino-macbook:testFolder minidino$ ls ?????
a012b	ab012
```

* `a[xy]b` 代表 x 或 y 選一個，有 `axb`, `ayb` 兩種選擇
* `a[^xy]b` 代表中間排除 x 與 y
```sh
minidino-macbook:testFolder minidino$ touch axb ayb acb
minidino-macbook:testFolder minidino$ ls a[xy]b
axb	ayb
minidino-macbook:testFolder minidino$ ls a[^xy]b
acb
# 大小寫有別
minidino-macbook:testFolder minidino$ touch aZb
minidino-macbook:testFolder minidino$ ls a[A-Z]b
aZb
minidino-macbook:testFolder minidino$ ls a[a-z]b
acb	axb	ayb
```

* 擴展 `{}`
    * 如 `{str1, str2}` 為匹派 str1 或 str2 的任意字串
```sh
minidino-macbook:testFolder minidino$ ls
a012b	aZb	aabxcb	ab	ab012	acb	axb	ayb
minidino-macbook:testFolder minidino$ ls a{012,Z,c,x,y}b
a012b	aZb	acb	axb	ayb
```

### shell meta（元字符）
* `IFS` : 由 <space> 或 <tab> 或 <enter> 三者之一組成
* `CR` : 由 `<enter>` 產生

# 正規表達式
* 針對檔案內容、標準輸入進行匹配
* 如果善用正規表達式，能對自動化的工作很有幫助
* 常用在爬蟲
* 有分為基本型正規表達式與擴充型正規表達式
  * 基本型正規表達式（Basic Regular Expression, BRE, 指令為 `grep`）
  * 擴充型正規表達式（**Extended** Regular Expression, ERE, 指令為 `egrep`）
* 正規表達式為飢餓模式的匹配，只要有符合皆會呈現（如下面第一個例子）
  1. `grep '[0-9][0-9]' passwd` : 會盡可能匹配所有可能的結果
  2. `grep '\b[0-9][0-9]\b' passwd` : 只會匹配所有 2 + 2 個數字的結果
    * `\b` : boundary

---

* 待會會針對 `etc/passed` 來做查找
* `cat /etc/passwd`：存放使用者的資訊，除了密碼
    * 真正的密碼存放在 `etc/shadow` 中
* 有兩種方式進行篩選：
    1. 從標準輸出：`cat passwd | grep 'docker'`
    2. 從檔案：`grep 'docker' passwd`
        * `grep -n` 能顯示匹配的行號

## 常見語法
字元|含義
-|-
`.`|任一字元
`*`|任意次數
`^`|以...為開頭
`$`|以...為結尾

```
root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6 | awk '{print $2}'
192.168.56.103
[root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6 | sed -r 's#(.*inet)##'
 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255
[root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6 | sed -r 's#(.*inet)##' | sed -r "s#(netmask.*$)##"
 192.168.56.103
[root@localhost security]# ifconfig enp0s8 | grep inet | grep -v inet6 | sed -r 's#(.*inet)(.*)(netmask.*$)#\2#'
 192.168.56.103 
```

* 透過 `cat >> 檔案名稱` 插入內容到檔案中
> 萬用字元與正規表達式有一些共用的符號，但作用不同
* `g*` 代表前面的字元 g 重複的次數，能重複 0 或多次
```sh
minidino-macbook:Desktop minidino$ cat a.txt
cat: a.txt: No such file or directory
minidino-macbook:Desktop minidino$ cat >> a.txt
google
goole
ggle
^C
minidino-macbook:Desktop minidino$ cat a.txt
google
goole
ggle
minidino-macbook:Desktop minidino$ cat >> a.txt
123
456
^C
minidino-macbook:Desktop minidino$ cat a.txt
google
goole
ggle
123
456
minidino-macbook:Desktop minidino$ grep "g*" a.txt
google
goole
ggle
123
456
```

## 重複
> 注意跟萬用字元之間的區別

* `?` : 零或一次
* `*` : 零或多次
* `+` : 一或多次
* `{,}`
* `|`

* 重複特定次數 `{n,m}`
  * n : 最小次數, m : 最大次數
  * `grep '[0-9]\{2,3\}' passwd`
    * 註：因為沒有設定邊界，可能會出現超過 3 個數字，如要修正，需要改寫成 `grep '[0-9]\{2,3\}' passwd`

* `grep 'se*' test.txt`
* `grep 'se+' test.txt`
* `grep 'se\+' test.txt`
  * 需使用 `\` （跳脫符號）
  * 也可寫成 `grep -E 'se+'` 或 `egrep 'se+'`
  * `-E` : Extended (GREP) = egrep
  * 一般比較建議透過 egrep 的方法
* `egrep -o 'se+' text.txt` : 只會顯示 se
  * `-o` 只顯示匹配的結果，不匹配的不顯示
  * `-o`, `--only-matching` : Prints only the matching part of the lines.
* `grep '\(se\)*' test.txt`
  * 建議改寫成 `egrep '(se)*' test.txt`
* `grep '\(se\)\+' test.txt`
* `grep '\(se\)\?' test.txt`

## 任意字符串：`.*`
* 如 `^r.*` 以 r 開頭直到結尾
*  `grep '^r.*' passwd`
*  `grep 'm.*c' passwd` : 結尾並不是第一個 c, 而是每行中的最後一個 c
*  `grep 'm..c. passwd`
*  `grep '\bm.*c\b' passwd`
*  `grep '\bm[a-z]

## 範例
* test.txt 檔案：
```
sesesese

se

seeeeeeeese

eeeeeeeeeeee

soooooooo

+se+se+
```

## 應用 1 : 篩選 QQ 號
```
[root@localhost Desktop]# cat qq.txt
fggsgd
23434646
234028309840
2342
0234568872090822X
980321197908032220
32080619800308999X
420204199005096
sdfs_sdfWEE23342
ksdf^*&(
sdfjlsf&&**(
sdfsf7899HIUT_8
[root@localhost Desktop]# grep '^[0-9]\{4,10\}' qq.txt
23434646
234028309840
2342
0234568872090822X
980321197908032220
32080619800308999X
420204199005096
[root@localhost Desktop]# grep -o '^[0-9]\{4,10\}' qq.txt
23434646
2340283098
2342
0234568872
9803211979
3208061980
4202041990
```

## 應用 2 : 篩選中國身分證號
* `grep '[1-9]\([0-9]\{13\}\|[0-9xX]\{16\}\)' qq.txt`
* `grep '^[1-9]\([0-9]\{13\}\|[0-9xX]\{16\}\)$' qq.txt`
* `grep '^\w\+$' qq.txt`

## 結論
![](/media/W6_regexSummary.jpg)
* 正規很無情
* 貪婪式匹配（盡可能地找出最多的結果）
* `grep` 別忘記 `\`（或者可以改用 `egrep`）


```
[root@localhost Desktop]# cat a.txt
Taipei 1
Kinmen
Tainan
2 Tainan
Taipei
Kinmen
a Taipei 34 343
[root@localhost Desktop]# cat a.txt | egrep -o "(Taipei|Kinmen|Tainan)"
Taipei
Kinmen
Tainan
Tainan
Taipei
Kinmen
Taipei
[root@localhost Desktop]# cat a.txt | egrep -o "(Taipei|Kinmen|Tainan)" | sort
Kinmen
Kinmen
Tainan
Tainan
Taipei
Taipei
Taipei[root@localhost Desktop]# cat a.txt | egrep -o "(Taipei|Kinmen|Tainan)" | sort | uniq
Kinmen
Tainan
Taipei
[root@localhost Desktop]# cat a.txt | egrep -o "(Taipei|Kinmen|Tainan)" | sort | uniq -c
      2 Kinmen
      2 Tainan
      3 Taipei
```
* `uniq -c` : 會顯示次數

# 雜記
* `cat -A <fileName>` 能分辨是否為空白行（行尾會加上`$`）

* `cd -` 代表上次造訪的目錄
```
[root@localhost ~]# cd /home/user/
[root@localhost user]# cd Desktop/
[root@localhost Desktop]# cd -
/home/user
```

* `cat !$`
  * `!$` : 前一個指令的最後一個參數
```
[root@localhost user]# ls /home/user/Desktop
a.txt                        qq.txt      test.txt
google-authenticator-libpam  testFolder
[root@localhost user]# cat !$
cat /home/user/Desktop
cat: /home/user/Desktop: 是個目錄
[root@localhost user]# cat !$/test.txt
cat /home/user/Desktop/test.txt
sesesese

se

seeeeeeeese

eeeeeeeeeeee

soooooooo

+se+se+
```

# 參考資料
* 參照：[linux通配符和正則表達式-Mykey-51CTO博客](https://blog.51cto.com/qibingtuan/1970593)