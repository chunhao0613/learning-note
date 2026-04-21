# Linux Process Part1
---
# 陳老師 課程整體架構

Docker 碼頭工人 - 用來裝程序的
- 既是公司名也是軟體名
- 核心技術 :
  - Application Container 軟體貨櫃 ( 一台電腦 )=>建立獨立空間給程序，方便管理

Kubernetes - k8s
- 由 Google 公司攥寫，穩定度不容小覷
- 維運一堆 Application Container

目前對 Docker 的討論已轉向 kubernetes，
可以說整個資訊界系統革命由 Docker 和 Kubernetes 創造的

### 1. SRE 需要這些工具 (核心：可觀測性)

SRE 的工作是維護系統穩定，面對「海量日誌 (Logs)」與「監控數據 (Metrics)」，傳統工具會崩潰，因此必須使用大數據組件：
- **Hadoop**：作為 **「數據黑盒子」**。存放長期的歷史監控資料、存取紀錄（Access Logs），供事後稽核或分析。
- **Spark**：作為 **「診斷引擎」**。當系統發生故障時，利用 Spark 的高速運算能力，從數億條 Log 中瞬間抓出導致錯誤的異常請求。
- **Hive**：作為 **「維運數據庫」**。讓 SRE 能量化穩定性指標，用 SQL 語法產出 SLO（服務等級目標）達成狀況報告。



重要網站
- cncf.io (系統工程的 Open Source)
- apache.org (資料科技的 Open Source )

## 企業為什麼買電腦？

- 20 幾年前公司為了買電腦是因為電子化，不需要再使用紙本，但是電腦在很多公司還是被當成電子打字機，
- 但時間往後走，資料庫出來了，打破原本電腦的使用方式，很多的企業會去買工作站（Server），讓企業建置出很多應用系統：會計系統、物料系統...等，
- 時間往後再走，老闆要求可不可以快速給分析報表，買更強的Server，做出更強的應用系統，讓企業做出決策的能力。
- 現在台灣流行的是做出預測的能力，譬如銀行會做呆帳的預測，呆帳比例越低獲利越高，一路走來企業為的就是：<font color=red>**要建置出各自需要的應用系統與軟體，來達到應用的需求。**</font>
- 應用軟體、應用系統，要經過開發、寫程式才會出來，才能執行應用。
- <font color=red>一個正在跑的應用系統由 "**多個應用程序**" + "**系統背景服務程序**" 組成</font>
- 寫程式要選擇電腦的語言，來開發企業所需要的應用系統，但為了寫程式同時硬體也要到位。
- 接下來要教的 <font color=red>**Container 和 Kubernetes 就是讓以上的應用系統能跑得安安穩穩長長久久**。</font>

---


## 建置 Docker 虛擬主機

1. 在 Windows 系統, 啟動 Browser, 下載 CNT.2022.v4.2.zip

2. 下載後, 請解壓縮至 使用者家目錄, 例如 /user/rbean

3. 在 Windows 系統的 cmd 終端機,執行以下命令

```bash
# 切換工作目錄
C:\Users\rbean>cd CNT.2022.v4.2

# 檢視 docker 命令可用那些參數
C:\Users\rbean\CNT.2022.v4.2> docker
"docker [create|start|stop|delete]"

# 建立 alp.docker VM
C:\Users\rbean\CNT.2022.v4.2> docker create
"DK\alp.docker\ created"

# 啟動 alp.docker VM
C:\Users\ybean\CNT.2022.v4.2> docker start

# ssh 遠端連線到 alp.docker 這台 VM
# 回答 yes ，表示要接收這台 VM 的 OpenSSH Server 的公鑰
C:\Users\ybean\CNT.2022.v4.2>ssh bigred@192.168.61.144
The authenticity of host '120.96.143.25 (120.96.143.25)' cant be established.
ECDSA key fingerprint is SHA256:haOU24boysVt3pRy0TLgE7rIYx6p4rkLfjOXrBewEU0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

:::spoiler alp docker VM 啟動後的畫面 ( 點我展開 )

![](https://i.imgur.com/dboQLRI.png)

:::



## Alpine Linux 的組成

### musl libc


```bash!
$ /lib/libc.musl-x86_64.so.1
musl libc (x86_64)
Version 1.2.2
Dynamic Program Loader
Usage: /lib/libc.musl-x86_64.so.1 [options] [--] pathname [args]
```

- C 語言的程式庫，裡面有一堆系統的底層運作程式。我們寫的程式會呼叫 System Call 來跟 linux 的 kernel 做溝通
- musl libc = Linux 系統的 API = System call
  > API 可以想成 bash Script 的 Function
- 系統底層的 API 可以讓程式用簡單的指令對 Linux 系統進行操作，而不用實際了解其底層運作及架構 例如: 在檔案系統上做產生目錄區、記憶體配置、指定網卡等動作 
	> so = system object
- 用 C 語言要創造一個目錄區，需不需要透過 System API ( musl libc )？
yes，因為 Linux 的目錄區和檔案，會經過 file system (檔案系統)來做的，linux 的檔案系統叫做 ext4，這個檔案系統他怎麼處理目錄，怎麼處理檔案，檔案名稱怎麼儲存，檔案的內容要放哪裡?一個檔案系統在存資料要不要有所謂的 Meta Data ?要不要有資料儲存的 location ?這個檔案系統怎麼樣在運作，內部是很複雜的，而 linux 裡面有人寫好了一些程式，讓我們可以很方便的使用這個 API (Appication Programming Interface) ，只要跟它說我要產生的資料夾名稱和資料夾的權限，它就幫我去產生了。
- 我們的 Alpine Linux 為甚麼需要 `musl libc (x86_64)` ?
    因為有了它，我們寫的 C 語言的一些高階程式，可以很方便的在檔案系統來產生目錄，可以很方便的在電腦的記憶體裡面挖一塊出來用，可以很方便的指定哪一片網卡幫我送資料，底層的真正的操作運作可以不需要了解。
- **程式執行一定會呼叫系統 API 和應用 API，這兩個 API 都是用電腦語言寫的**
- 不使用 System API 就必須熟悉系統的所有功能，並且用程式語言完全實現，這樣耗費時間、效率低落
	>  以應用開發工程師來說，不使用 API 就必須熟悉系統的所有功能，並且用程式語言完全實現，這樣開發應用相對耗費時間、效率低落，但如果是系統開發工程師，不可避免需要去理解抵成並用程式實現功能、包裝成 API
- 高階程式設計師可透過現有函數很方便的操作系統 ( CPU、網路、記憶體 )，不需知道系統底層運作原理即可操作。

### busybox

```
bigred@alp:~$ which busybox
/bin/busybox

bigred@alp:~$ ls -lh /bin/busybox
-rwxr-xr-x 1 root root 806K Apr  4 18:19 /bin/busybox
```
busybox
- 中文 : 瑞士刀
- 檔案大小 : 806 kb ，是用 C 語言寫的，可以模擬很多 Linux 常用指令（cat、cp...）  
- 屬於高階應用的命令，也會呼叫 System API


### apk

```
$ sudo apk update ; sudo apk upgrade
```

- apk ，全名 : `alpine package`
- apk 套件管理系統，讓我們可以在 Alpine linux 裡面，加裝一些套件，譬如: openssh server 、 jdk-8 ...等
- `update` + `upgrade` 等同於 microsoft 的系統更新，可以修正作業系統的漏洞

## 小總結

**企業買電腦就是為了做出他們需要的應用系統，而應用系統是用程式寫出來的，而我們寫的程式大多都屬於高階的，會去呼叫和使用底層的系統 API，以 bash Script 來說就是 Function，人家寫好的 system function，讓我們不需要接觸最底層的程式設計，在應用這層不需要耗費這些時間。**

> 應用系統最常見的就是會計系統、銀行的呆帳系統...等


## 下載 Alpine Linux


[下載 Alpine Linux 連結](https://alpinelinux.org/downloads/)
- STANDARD ，根據 CPU 的規格來區分，主要裝在實體電腦， Intel 的 CPU 主要是下載 `x86_64` 這個項目，下載下來的檔案是一個光碟檔 (`*.iso`) 
- EXTENDED ，下載下來的 Alpine 會在記憶體裡面跑，檔案會寫入 USB
- NETBOOT ，透過網路就可以把 Alpine 裝起來
- MINI ROOT FILESYSTEM ，可以裝在 Container 上
- RASPBERRY PI ，裝在樹梅派上
- VIRTUAL ，裝在 VMware 的 VM 上
- XEN ，裝在 Xen Hypervisor 啟動的 VM 上


## Alpine Linux 系統更新

```bash!
Alpine Linux 系統更新
$ sudo apk update; sudo apk upgrade
```

- `;` ，表示不管前面的命令有無執行成功，只要結束，就會跑後面的命令。


## 80 年代大型主機實體架構

![](https://i.imgur.com/RnSTxTz.png)

- `RS-232` ，一條連接線，讓終端機與 Linux 主機連接
- 終端機的組成 : 螢幕 + 鍵盤
- **在終端機上面一定會跑一支貝殼程式** ( 例如， sh, ash, zsh, bash ...等 )
- **貝殼程式主要負責接收我們下達的命令**，命令會藉由 RS-232 這條線傳給 Linux 主機，命令的結果會再藉由 RS-232 這條線傳給終端機上面的貝殼程式，貝殼程式收到後會幫我們把命令結果顯示在終端機的螢幕上。
- **Linux 系統支援協同作業 (多人同時共用一台主機)**


## 建立 Alpine Linux 使用者帳號


```bash!
$ echo -e "rbean\nrbean" | sudo adduser -s /bin/bash rbean
Changing password for rbean
New password:
Bad password: too short
Retype password:
passwd: password for rbean changed by root
```
- `adduser` ，Alpine Linux 新增使用者帳號的命令
- `-s` ，設定貝殼程式
- `echo -e` ，讓反斜線可以轉義
- `\n` ，換行符號

```bash!
# 檢視 Linux 系統放帳號資訊的檔案
$ tail -n 2 /etc/passwd
utmp:x:102:406:utmp:/home/utmp:/bin/false
rbean:x:1002:1002:Linux User,,,:/home/rbean:/bin/bash

# 檢查家目錄，有無建立成功
$ ls -al /home
total 20
drwxr-xr-x  5 root   root   4096 Jun  8 19:54 .
drwxr-xr-x 23 root   root   4096 May 26 00:42 ..
drwxr-sr-x  5 bigred bigred 4096 Feb 17 10:09 bigred
drwxr-sr-x  2 qemu   kvm    4096 Sep  5  2020 qemu
drwxr-sr-x  2 rbean  rbean  4096 Jun  8 19:54 rbean
```

- 每個 Linux 的使用者一定會有一個家目錄 ( `/home/$USER` )


## rbean 登入 Alpine Linux 虛擬主機

```bash!
# 在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh rbean@<alp.docker IP>
Welcome to Alpine Linux : 3.16.0
IP : 192.168.61.146

# 虛擬終端機
$ tty
/dev/pts/1

# 檢查現在有動態產生幾個虛擬終端機
$ ls -al /dev/pts
total 0
drwxr-xr-x  2 root   root      0 Aug  8 10:13 .
drwxr-xr-x 13 root   root   2780 Aug  8 10:14 ..
crw--w----  1 bigred tty  136, 0 Aug  8 11:26 0
crw--w----  1 rbean  tty  136, 1 Aug  8 11:27 1
c---------  1 root   root   5, 2 Aug  8 10:13 ptmx

# 顯示當前使用者用的貝殼程式
$ echo $SHELL
/bin/bash

# 顯示當前使用者家目錄的路徑
$ echo $HOME
/home/rbean

$ exit
```

- `/dev/pts/1` ，`dev` ，裝置的意思，`pts` ，ssh 遠端連接進來的虛擬終端機，`/1`，1 號虛擬終端機
- 每個 Linux 使用者透過 ssh 遠端連線，一定會有一個虛擬終端機可以使用，在這台虛擬終端機上還會跑一支貝殼程式
- `/dev/tty` ，本機終端設備的統稱
- `/dev/pts` ，遠端虛擬終端
    - `pts` ，pseudo terminal slave
- The difference between TTY and PTS is the type of connection to the computer. TTY ports are direct connections to the computer such as a keyboard/mouse or a serial connection to the device. PTS connections are SSH connections or telnet connections. All of these connections can connect to a shell which will allow you to issue commands to the computer.

:::spoiler 練習 : 在 Alpine Linux 上建一個 gbean 使用者，並按照上面的步驟做一次

```bash!
# 在 bigred 使用者建立 gbean 使用者帳號
$ echo -e "gbean\ngbean" | sudo adduser -s /bin/bash gbean
Changing password for gbean
New password:
Bad password: too short
Retype password:
passwd: password for gbean changed by root

# 檢視 gbean 帳號資訊
$ tail -n 2 /etc/passwd
rbean:x:1002:1002:Linux User,,,:/home/rbean:/bin/bash
gbean:x:1003:1003:Linux User,,,:/home/gbean:/bin/bash

# 檢查 gbean 帳號家目錄
$ ls -l /home
total 16
drwxr-sr-x 5 bigred bigred 4096 Jun 27 21:12 bigred
drwxr-sr-x 2 gbean  gbean  4096 Aug  8 11:29 gbean
drwxr-sr-x 2 qemu   kvm    4096 Sep  5  2020 qemu
drwxr-sr-x 2 rbean  rbean  4096 Aug  8 11:22 rbean

# gbean 登入 Alpine Linux 虛擬主機
$ ssh bigred@120.96.143.25

# 檢查自己用幾號虛擬終端機
$ tty
/dev/pts/2

# 看目前總共有幾台虛擬終端機
$ ls -al /dev/pts/
total 0
drwxr-xr-x  2 root   root      0 Aug  8 10:13 .
drwxr-xr-x 13 root   root   2780 Aug  8 10:14 ..
crw--w----  1 bigred tty  136, 0 Aug  8 11:30 0
crw--w----  1 rbean  tty  136, 1 Aug  8 11:27 1
crw--w----  1 gbean  tty  136, 2 Aug  8 11:31 2
c---------  1 root   root   5, 2 Aug  8 10:13 ptmx

# 看自己用哪個貝殼程式
gbean@alp:~$ echo $SHELL
/bin/bash

# 顯示自己的家目錄路徑
gbean@alp:~$ echo $HOME
/home/gbean
```

:::

在本機看終端機

```bash!
$ tty
/dev/tty1
```

- `/dev/tty1` ，1 號本機終端機


---


# 電腦程式語言的運作模式


## **Interpreter**


![](https://i.imgur.com/a6gfLLc.png)

- Source Code 的意思，以 sheel 程式舉例，就是我們在寫的 bash Script 
- 系統會一行一行的執行我們所寫的程式，當執行完第一行時，會有解析器，把我們寫的程式變成電腦看得懂的 2 進位的機械碼(machine code)，然後再由電腦來做執行
- 提問: Java Script 是不是直譯式 ?
Java Script 事實上是在瀏覽器上作執行，它的執行方法就是 Interpreter 這種運作方式，所以前端連覽器開了很多分頁，會把硬體資源吃光光，因為每一個分頁裡面都在跑 Java Script。
- **直譯式語言，耗費的電腦資源比較多**
- 目前的瀏覽器都在比，誰設計的 Interpreter 引擎越精簡，代表瀏覽器的表現越穩定、速度越快。
- Python 語言就是 Interpreter


## Compiler


![](https://i.imgur.com/sOySud4.png)

- 將人看得懂的程式，經過編譯器，直接變成電腦看得懂的執行碼
- 執行效能最快


---


### 撰寫 程式碼 (Coding) 

- Coding ，意思就是寫程式

```bash=
$ cd; mkdir mygo; cd mygo

# 開發 golang 網站
# golang 語言是 Compiler
# 寫人看得懂的 go 程式
$ echo 'package main
import (
	"fmt"                
	"log"                
	"net/http"          
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, 世界")
}
func main() {
	http.HandleFunc("/", handler)         
	log.Fatal(http.ListenAndServe(":8888", nil))   
}' > main.go
```

- `import ( )` ，引用人家寫好的 API
- `fmt` ，顯示資料格式的 lib，可以顯示各式各樣的資料 
- `log` ，log 的 API，資料量多大 ? 總共可以寫多少個 log ? 超過這個數量，最前面的檔案是否要刪掉 ?
- `net/http` ，網路的 API，IP 位址、MAC address、ARP...等
- `http.HandleFunc("/", handler)` ，"`/`"，宣告網站的根目錄，只要其他人連到我的網址最後的`/`後面加 `handler`，就會顯示 `Hello, 世界`
- `log.Fatal(http.ListenAndServe(":8888", nil))` ，告訴系統，我們的 web Server 等下要鎖在 8888 port


```bash!
# 將上面的程式經過編譯變成電腦看得懂的語言
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a main.go
```

- `CGO_ENABLED` 這個參數，主要是讓 Go 執行檔能夠直接連結到系統上的函式庫 (libraries)，而不必另外打包進執行檔。
- `CGO_ENABLED=0` ，代表剛剛上面宣告的 3 個 API 的 machine code 也加進最後的機械碼執行檔
- `GOOS=linux GOARCH=amd64` ，產生 Linux 64位元作業系統的執行檔
- `go build` ，編譯我們寫好的 golang 語言的程式
- `-a` ，force rebuilding of packages that are already up-to-date
- 產生的機械碼檔，老師稱之為自走砲，意思就是說將翻譯好的那個機械碼檔拷貝到任意的 Linux 系統就可以跑，不需要在靠別的套件，因為它本身帶著程式庫走。


```bash!
# 檢查
$ dir
total 5.9M
drwxr-sr-x 2 bigred bigred 4.0K May 17 15:29 .
drwxr-sr-x 7 bigred bigred 4.0K May 17 15:29 ..
-rwxr-xr-x 1 bigred bigred 5.8M May 17 15:29 main
-rw-r--r-- 1 bigred bigred  228 May 17 15:29 main.go
```


---


### 看編譯過後的機械碼檔

安裝 binary vi 的編輯器 ( 2 進位檔的編輯器 )

```
$ sudo apk add bvi
(1/1) Installing bvi (1.4.1-r0)
Executing busybox-1.34.1-r5.trigger
OK: 1249 MiB in 258 packages
```

使用編輯器

```
$ bvi main
```

output : 

```
00000000  7F 45 4C 46 02 01 01 00 00 00 00 00 00 00 00 00 02 00 3E 00 01 00 00 00 .ELF..............>.....
00000018  A0 31 46 00 00 00 00 00 40 00 00 00 00 00 00 00 C8 01 00 00 00 00 00 00 .1F.....@...............
00000030  00 00 00 00 40 00 38 00 07 00 40 00 17 00 03 00 06 00 00 00 04 00 00 00 ....@.8...@.............
00000048  40 00 00 00 00 00 00 00 40 00 40 00 00 00 00 00 40 00 40 00 00 00 00 00 @.......@.@.....@.@.....
00000060  88 01 00 00 00 00 00 00 88 01 00 00 00 00 00 00 00 10 00 00 00 00 00 00 ........................
00000078  04 00 00 00 04 00 00 00 9C 0F 00 00 00 00 00 00 9C 0F 40 00 00 00 00 00 ..................@.....
00000090  9C 0F 40 00 00 00 00 00 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00 ..@.....d.......d.......
000000A8  04 00 00 00 00 00 00 00 01 00 00 00 05 00 00 00 00 00 00 00 00 00 00 00 ........................
000000C0  00 00 40 00 00 00 00 00 00 00 40 00 00 00 00 00 2A 7C 1E 00 00 00 00 00 ..@.......@.....*|......
000000D8  2A 7C 1E 00 00 00 00 00 00 10 00 00 00 00 00 00 01 00 00 00 04 00 00 00 *|......................
000000F0  00 80 1E 00 00 00 00 00 00 80 5E 00 00 00 00 00 00 80 5E 00 00 00 00 00 ..........^.......^.....
00000108  40 A2 1D 00 00 00 00 00 40 A2 1D 00 00 00 00 00 00 10 00 00 00 00 00 00 @.......@...............
00000120  01 00 00 00 06 00 00 00 00 30 3C 00 00 00 00 00 00 30 7C 00 00 00 00 00 .........0<......0|.....
00000138  00 30 7C 00 00 00 00 00 20 20 05 00 00 00 00 00 80 AE 08 00 00 00 00 00 .0|.....  ..............
00000150  00 10 00 00 00 00 00 00 51 E5 74 64 06 00 00 00 00 00 00 00 00 00 00 00 ........Q.td............
00000168  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ........................
00000180  00 00 00 00 00 00 00 00 08 00 00 00 00 00 00 00 80 15 04 65 00 2A 00 00 ...................e.*..
00000198  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ........................
000001B0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 08 00 00 00 00 00 00 00 ........................
000001C8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ........................
000001E0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ........................
000001F8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 01 00 00 00 01 00 00 00 ........................
00000210  06 00 00 00 00 00 00 00 00 10 40 00 00 00 00 00 00 10 00 00 00 00 00 00 ..........@.............
00000228  2A 6C 1E 00 00 00 00 00 00 00 00 00 00 00 00 00 20 00 00 00 00 00 00 00 *l.............. .......
00000240  00 00 00 00 00 00 00 00 6A 00 00 00 01 00 00 00 02 00 00 00 00 00 00 00 ........j...............
00000258  00 80 5E 00 00 00 00 00 00 80 1E 00 00 00 00 00 49 8A 0B 00 00 00 00 00 ..^.............I.......
00000270  00 00 00 00 00 00 00 00 20 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ........ ...............
"main" 6071722 bytes
```

- 只要是 Linux 可以執行的程式碼，前面 4 個 byte，一定是 `7F 45 4C 46` ( 16 進位 )，以文字來表示的話就是 `.ELF`
- 如果要做駭客，看懂機械碼是基本功
- 游標可以上下移動，是因為可以直接修改機械碼
- 機械碼要看懂還有一個條件，要會寫組合語言
- 如果要突破軟體搭配硬體才可以跑，但軟體很貴，可以找到 `jump` 的機械碼，把它加進去，讓檢查的步驟跳過，就可以避開硬體的檢查
- [執行檔格式 - ELF 介紹連結](https://ithelp.ithome.com.tw/articles/10222650)
- [Bvi Quick Tutorial 連結](http://bvi.sourceforge.net/quick.html)

離開bvi編輯器
```
:q
```

---

### 練習

1. 那些使用者可以執行 `./main` 程式 ? (請實作証明)
```
bigred@alp:~$ dir
total 40K
drwxr-sr-x 7 bigred bigred 4.0K May 17 15:29 .
drwxr-xr-x 4 root   root   4.0K Jul 10  2021 ..
-rw------- 1 bigred bigred 1.8K May 17 14:17 .bash_history
drwxr-sr-x 3 bigred bigred 4.0K May 17 15:29 .cache
-rw------- 1 bigred bigred    7 Feb 17 10:09 .python_history
drwx--S--- 2 bigred bigred 4.0K Jan  9 18:00 .ssh
drwxr-sr-x 2 bigred bigred 4.0K Oct  6  2021 bin
drwxr-sr-x 3 bigred bigred 4.0K May 17 15:29 go
drwxr-sr-x 2 bigred bigred 4.0K May 17 15:29 mygo
-rwxr-xr-x 1 bigred bigred   63 May 17 15:21 mysleep.sh

bigred@alp:~/mygo$ dir
total 5.9M
drwxr-sr-x 2 bigred bigred 4.0K May 17 15:29 .
drwxr-sr-x 7 bigred bigred 4.0K May 17 15:29 ..
-rwxr-xr-x 1 bigred bigred 5.8M May 17 15:29 main
-rw-r--r-- 1 bigred bigred  228 May 17 15:29 main.go
```
所有人

2. 設定 main 只允許 擁有者 可執行
```
bigred@alp:~$ sudo chmod 700 /home/bigred/mygo/main
bigred@alp:~$ dir mygo
total 5.9M
drwxr-sr-x 2 bigred bigred 4.0K May 17 15:29 .
drwxr-sr-x 7 bigred bigred 4.0K May 17 15:29 ..
-rwxr--r-- 1 bigred bigred 5.8M May 17 15:29 main
-rw-r--r-- 1 bigred bigred  228 May 17 15:29 main.go
```

---

### Write Once Run Anywhere - Java Program

- Write Once Run Anywhere
程式在 Linux 寫，在 Windows 也可以執行

![](https://i.imgur.com/KNch44Z.png)

- 當我們寫一隻 Java 的程式（`*.Java`），系統要執行它之前，會先將我們寫好的程式透過一個編譯器將程式碼轉換成 bytecode(`*.class`)，而 JVM(Java vitual machine) 會將 bytecode 再轉換成 machine code，就可以達到在不同作業系統都可以執行 Java 程式
- JVM 由 C 語言寫的，它的特性是會咬住它所在的作業系統，而這個 JVM 就是課程之前練習裝過的 `jdk-8`，只要作業系統可以安裝 JVM，就可以讓我們在不同的作業系統執行 Java 程式。

```bash=
# 建立工作目錄
$ cd ~; mkdir myjava; cd myjava

# coding java program
$ echo $'public class ReverNum {
>   public static void main(String[] args) {
>
>     int num = 1234, reversed = 0;
>     while(num != 0) {
>       int digit = num % 10;
>       reversed = reversed * 10 + digit;
>       num /= 10;
>     }
>
>     System.out.println("Reversed Number: " + reversed);
>   }
> }'> ReverNum.java

# 將我們寫好的程式碼編譯成 byte code
$ javac ReverNum.java

$ dir
total 16K
drwxr-sr-x 2 bigred bigred 4.0K May 18 10:14 .
drwxr-sr-x 8 bigred bigred 4.0K May 18 10:13 ..
-rw-r--r-- 1 bigred bigred  714 May 18 10:14 ReverNum.class
-rw-r--r-- 1 bigred bigred  278 May 18 10:14 ReverNum.java

# 這邊的Java就是JVM，注意檔名不能加.class
$ java ReverNum
Reversed Number: 4321
```


### 討論

- Is Python interpreted or compiled ?
    - [答案網站連結](https://stackoverflow.com/questions/6889747/is-python-interpreted-or-compiled-or-both) ，與 Java 語言的運作模式類似
- Python 它兩種都可以 
	- 使用 CPython 標準開發時為 interpeterd 
	- 使用 PVM 標準開發時，與 Java 相同為 Compilerd

> 事實上應該要問 Python 有幾種運作模式
> Stack Overflow 這個網站非常具有權威

---


# Process 程序


Process 就是運作中的程式，分為兩種
- 前景執行的應用程序
- 背景執行的程序


![](https://i.imgur.com/QIjTSNw.png)

- 當我們在系統裡面執行程式，電腦會把程式載入記憶體，而這時候記憶體裡面會有程式的 PID 和執行者的權限屬性（UID或GID），還有程序的程式碼（machine code）和相關資料

## 檔案的特殊權限 (SUID, SGID, Sticky) 
- SUID(4): 執行檔案時，Process 的 UID 為檔案擁有者，而非檔案執行者 
- SGID(2): 執行檔案時，Process 的 GID 為檔案的群組，而非依檔案執行者的群組 
- Sticky(1): 設定目錄，每個使用者可以創建/刪除自己的檔案，但無法刪除別人創建的檔案

> setuid 功能對 bash script 無效，也就是說對於 interpeter 是無效的



### setuid with bash script can become effective ?

```bash=
# 編輯一支程式
$ nano md.sh
#!/bin/bash
mkdir /test

# 程序被賦予的權限，是根據執行者的權限參數（UID GID）來決定，
# 所以當執行者沒有對應的執行權限，程式就會無法進行對應的指令操作
$ ./md.sh
mkdir: cannot create directory ‘/test’: Permission denied

# 命令前面加 sudo，代表現在是由 root 的權限在執行程式
$ sudo /home/bigred/md.sh
```
==**程式在跑時一定會用執行者的權限參數（UID GID）去執行**==

更改 `md.sh` 的擁有者為 : root

```bash!
$ ls -l md.sh
-rwxr-xr-x 1 bigred bigred 24 May 18 11:25 md.sh

$ sudo chown root:bigred md.sh

$ ls -l md.sh
-rwxr-xr-x 1 root bigred 24 May 18 11:25 md.sh
```

請問 bigred 可以執行 `md.sh` 嗎？

```bash!
$ ./md.sh
mkdir: cannot create directory ‘/test’: Permission denied
```
答：可以，但不會成功，因為執行者是 bigred，他的權限參數不足


再度更改權限

`4` 代表 set uid，通常設在執行檔，一概用程式的 owner 去執行，但 bash script 裡面無法用 set uid
```
$ sudo chmod 4755 md.sh

$ ls -al md.sh
-rwsr-xr-x 1 root bigred 24 May 18 11:25 md.sh

$ stat md.sh
  File: md.sh
  Size: 24              Blocks: 8          IO Block: 4096   regular file
Device: 8,3     Inode: 25953103    Links: 1
Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: ( 1000/  bigred)
Access: 2022-05-18 13:06:13.239195839 +0800
Modify: 2022-05-18 11:25:33.183472747 +0800
Change: 2022-05-18 13:15:08.666097079 +0800
 Birth: -
```

### 提問：為何 rbean 這位一般使用者可以執行 `ls -al /etc/shadow`

```bash
# 透過 ls 顯示 /etc/shadow 的檔案資訊
rbean@alp:~$ ls -al /etc/shadow
-rw-r----- 1 root shadow 906 May 18 09:30 /etc/shadow

# 透過 stat 顯示 /etc/shadow 的檔案資訊
rbean@alp:~$ stat /etc/shadow
  File: /etc/shadow
  Size: 906             Blocks: 8          IO Block: 4096   regular file
Device: 8,3     Inode: 19267677    Links: 1
Access: (0640/-rw-r-----)  Uid: (    0/    root)   Gid: (   42/  shadow)
Access: 2022-05-18 09:30:41.193045223 +0800
Modify: 2022-05-18 09:30:03.713243333 +0800
Change: 2022-05-18 09:30:03.713243333 +0800
 Birth: -
```

- `stat`，gives information about the file and filesystem.
- Usage : `stat --options filenames`


答：因為 `/etc` 的目錄在權限屬性上的其他人是 `r-x`，代表可以看到目錄裡的內容

```bash
bigred@alp:~$ ls -al / | grep etc
drwxr-xr-x  42 root root  4096 May 18 13:30 etc
```
rbean 不能看到檔案內容

```bash
rbean@alp:~$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

修改 rbean 使用者的密碼

```bash
rbean@alp:~$ passwd rbean
Changing password for rbean
Old password:
New password:
Retype password:
passwd: password for rbean changed by rbean
```

提問：為何 rbean 看不到密碼檔（/etc/shadow），但卻可以更改自己的密碼？
答：我們的 `passwd` 這個 program 的執行檔（`/bin/bbsuid`）有設定 `s` 的權限，代表 `setuid`，數字為 4，當我們執行這個程式，系統在記憶體裡執行這個 process 時，執行者的 uid 是這個 program 的 owner，但注意，此功能不適用 bash script。

```bash!
$ which passwd
/usr/bin/passwd

$ ls -l /usr/bin/passwd
lrwxrwxrwx 1 root root 11 May 17 15:49 /usr/bin/passwd -> /bin/bbsuid

$ ls -l /bin/bbsuid
---s--x--x 1 root root 14200 Apr  4 18:19 /bin/bbsuid
```




### 改造無敵貓

檢查 `cat` 的執行檔在哪，他的 owner 是誰

```bash
$ which cat
/usr/bin/cat

$ ls -l /usr/bin/cat
-rwxr-xr-x 1 root root 43416 Sep  5  2019 /usr/bin/cat

$ stat /usr/bin/cat
  File: /usr/bin/cat
  Size: 43416           Blocks: 88         IO Block: 4096   regular file
Device: 801h/2049d      Inode: 1583        Links: 1
Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-05-18 06:21:53.210823732 +0000
Modify: 2019-09-05 10:38:40.000000000 +0000
Change: 2022-04-19 19:34:46.435775857 +0000
 Birth: -
```

將 `cat` 設定 setuid 的功能

```bash
$ sudo chmod 4755 /usr/bin/cat

# 檢查
$ stat /usr/bin/cat
  File: /usr/bin/cat
  Size: 43416           Blocks: 88         IO Block: 4096   regular file
Device: 801h/2049d      Inode: 1583        Links: 1
Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-05-18 06:21:53.210823732 +0000
Modify: 2019-09-05 10:38:40.000000000 +0000
Change: 2022-05-18 06:50:08.981085444 +0000
 Birth: -
```

改造無敵貓成功

```
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
```

### ping 指令


```bash
$ which ping
/bin/ping

$ ls -l /bin/ping
-rwsr-xr-x 1 root root 64120 Jul 24  2021 /bin/ping

$ stat /bin/ping
  File: /bin/ping
  Size: 64120           Blocks: 128        IO Block: 4096   regular file
Device: 8,3     Inode: 23330906    Links: 1
Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-05-16 17:47:14.463395952 +0800
Modify: 2021-07-24 03:58:27.000000000 +0800
Change: 2021-11-27 20:36:50.420444522 +0800
 Birth: -
```
- ping 指令會用到 icmp 封包相關功能，這件事只有 root 能做，因為有設定 setuid 所以任何使用者都可以直接使用 ping 指令

---

## Linux 程序管理

###  **顯示程序資訊**


條列式顯示所有程序

```bash
$ ps -eaf | head -n 1; ps -eaf | tail -n 15
UID         PID   PPID  C STIME TTY          TIME CMD
root       4033   3779  0 10:15 ?        00:00:00 sshd: bigred [priv]
bigred     4046   4033  0 10:16 ?        00:00:00 sshd: bigred@pts/0
bigred     4047   4046  0 10:16 pts/0    00:00:00 -bash
root       6428      2  0 10:54 ?        00:00:42 [kworker/1:0-events]
root       8604   3779  0 11:23 ?        00:00:00 sshd: rbean [priv]
rbean      8608   8604  0 11:23 ?        00:00:00 sshd: rbean@pts/1
rbean      8609   8608  0 11:23 pts/1    00:00:00 -bash
root       9097   3779  0 11:30 ?        00:00:00 sshd: gbean [priv]
gbean      9101   9097  0 11:30 ?        00:00:00 sshd: gbean@pts/2
gbean      9102   9101  0 11:30 pts/2    00:00:00 -bash
root      21898      2  0 14:46 ?        00:00:00 [kworker/u256:1-events_unbound]
root      22292      2  0 14:53 ?        00:00:00 [kworker/u256:2-events_unbound]
root      22621      2  0 14:58 ?        00:00:00 [kworker/u256:0-events_unbound]
bigred    22660   4047  0 14:59 pts/0    00:00:00 ps -eaf
bigred    22661   4047  0 14:59 pts/0    00:00:00 tail -n 1
```

- `ps` ，processes status
- `-eaf` ， list all current processes


## 顯示背景程序資訊

### 依照 CPU 使用量，從大到小排序程序


```bash!
$ timeout 20s yes >/dev/null & 
```

- `timeout` ，Start COMMAND, and kill it if still running after DURATION
- `timeout 20s` ，20 秒後結束 `yes` 的程式
- `yes` 命令的特性，能用多少 CPU 就用多少
- `>/dev/null` ，把標準輸出 ( STDOUT ) 重導到黑洞裝置
- `&`，把程序推到背景執行

```
$ ps -eo user,pid,cmd,%mem,%cpu --sort=-%cpu | head -n 3
USER        PID CMD                         %MEM %CPU
bigred    95970 yes                          0.0  103
root        512 [kworker/1:2-events]         0.0  0.1
```

- `ps` ，processes status
- `-e` ，輸出所有程序的資訊
- `-o` ，指定輸出欄位，後面接所有想要輸出的欄位名稱，這裡我們讓 ps 輸出以下幾個欄位：
    - `PID`：程序 ID（process ID）。
    - `CMD`：程式名稱。
    - `%MEM`：記憶體使用量（百分比）。
    - `%CPU`：CPU 使用量（百分比）。
- `--sort` ，預設由小到大，在欄位名稱前加上減號(-)，表示由大到小
- `|`，上一個命令的輸出作為輸入傳遞給下一個命令。
- `| head -n 3` ，透過 pipe 將左邊命令輸出作為輸入傳遞給 head 命令，來篩選 CPU 用最兇猛的前兩個 process 。
- [What is the difference between "Redirection" and "Pipe"? 連結](https://askubuntu.com/questions/172982/what-is-the-difference-between-redirection-and-pipe)


---

### 依照 Memory 使用量，從大到小排序程序

```bash!
$ cat /dev/zero
```

- 上面這個指令，會一直在螢幕上輸出 `ASCll code 0` 的數字，它代表的是 `null`，空值，所以螢幕上看不到，直到按 Ctrl + c 終止
- Null（或Nul）也是一個零值 ASCII 字符，通常用於終止文件中的行尾
- Null String 是計算機編程中長度為零的字符串變量。
- NULL 指針在計算機編程中用於表示尚未初始化或只是空的值。
- 在電腦中儲存時也要使用二進位數來表示，而具體用哪些二進位數字表示哪個符號，這就是編碼。如果不同的電腦要想互相通信而不造成混亂，那麼每台電腦就必須使用相同的編碼規則，就出現了 ASCII 編碼。
- :::spoiler ASCII Table ( 點我展開 )

  ![](https://i.imgur.com/05rXuOD.png)
  
  > [資料來源連結](https://www.alpharithms.com/ascii-table-512119/)
  :::


```bash!
$ (cat /dev/zero | head -c 500m | tail) &
[1] 96961

$ ps -eo user,pid,cmd,%mem,%cpu --sort=-%mem | head -n 3
USER        PID CMD                         %MEM %CPU
bigred    96964 tail                        29.2 51.1
root      74943 sshd: rbean [priv]           0.1  0.0
```


- `head -c` ，print the first NUM bytes of each file
- `tail` ，會霸佔記憶體 500 mb 的資料量
- 輸出 `ASCll code 0` 的數字總共 500 mb，然後輸出的結果會丟給 `tail`，`tail` 會再幫我們把結果丟到記憶體中，最後把以上的程序都推到背景去執行。


---

### **以樹狀顯示程序資訊**

```bash
$ pstree -ph
pstree: unrecognized option: h
BusyBox v1.34.1 (2022-04-04 10:19:27 UTC) multi-call binary.

Usage: pstree [-p] [PID|USER]

Display process tree, optionally start from USER or PID

        -p      Show pids
```

- 因一開始的 `pstree` 是由 busybox 執行，在功能上來說是簡易型的，如果要安裝全功能的，要自行安裝該套件

```
$ sudo apk add pstree
```

安裝好後，下 `pstree -h`，`-h` 為安裝手冊的意思，可以看到功能比較完整

- `pstree -g 3`，輸出時用 `utf-8` 的圖形 ( 樹狀的線 )

```bash!
$ pstree -g 3
 # 天字第一號老祖宗 /sbin/init
─┬= 00001 root /sbin/init
 ├──= 03186 root /sbin/getty 38400 tty6
 ├──= 03185 root /sbin/getty 38400 tty5
 ├──= 03184 root /sbin/getty 38400 tty4
 ├──= 03182 root /sbin/getty 38400 tty3
 ├──= 03180 root /sbin/getty 38400 tty2
 # 03178 是 pid，root 為 uid，/bin/login 為程序名，
 # -f 是登入時不進行身份認證，代表自動登入，使用 tty1 本機終端機
 ├─┬= 03178 root /bin/login -f bigred
 # bigred 帳號執行重裝貝殼程式
 │ └─┬= 03187 bigred -bash
 # 利用 dialog 顯示硬體和網路規格
 # --title，標題
 # --textbox，後面接檔案 高度 寬度
 │   └─── 03242 bigred dialog --title  Cloud Native Trainer  --textbox /tmp/sinfo 24 85
 # 啟動 OpenSSH Server
 ├─┬─ 03091 root sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups
 # 記錄 bigred 使用者連接時跑的 process
 │ ├─┬= 88863 root sshd: bigred [priv]
 │ │ └─┬─ 88867 bigred sshd: bigred@pts/0
 │ │   └─┬= 88868 bigred -bash
 │ │     └──= 31291 bigred pstree -g 3
 # 記錄 rbean 使用者連接時跑的 process
 │ └─┬= 31250 root sshd: rbean [priv]
 # pts 代表，系統幫我們準備遠端終端機
 │   └─┬─ 31266 rbean sshd: rbean@pts/1
 # 啟動貝殼程式
 │     └──= 31267 rbean -bash
 # crond 排程器，定期執行命令或指定程式的服務的一種服務或軟體。
 ├──= 03053 root /usr/sbin/crond -c /etc/crontabs
 # 啟動 chrony 時間校正器
 ├─── 03025 chrony /usr/sbin/chronyd -f /etc/chrony/chrony.conf
 # 備用電源程式 ups，如果突然斷電，系統會判定是否要正常關機，acpid 會提供前面的功能
 ├──= 02918 root /sbin/acpid
 # 啟動 log 記錄功能
 ├──= 02867 root /sbin/syslogd -t
 # dhcp client ，這個 process 會跟 網路上的 DHCP Server 要一個 IP 。
 ├──= 02831 root /sbin/udhcpc -b -R -p /var/run/udhcpc.eth0.pid -i eth0 -x hostname:alp
 └──= 02714 root /sbin/udevd
```

- `dialog` ，可以在 Terminal 上快速建立圖形交互介面的工具
    - `--title <title>` ，設定標題
    - `--textbox` ，設定要顯示文字內容的檔案，以及高度和寬度
    - [dialog 使用教學連結](https://codychen.me/2020/29/linux-shell-%E7%9A%84%E5%9C%96%E5%BD%A2%E4%BA%92%E5%8B%95%E5%BC%8F%E4%BB%8B%E9%9D%A2-dialog/)


```bash!
$ ps aux | grep sshd
# /usr/sbin/sshd 這支程序代表 OpenSSH Server有啟動
root       3091  0.0  0.1   4468  2104 ?        S    May17   0:00 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups
root      31250  0.0  0.1   4488  3696 ?        Ss   09:29   0:00 sshd: rbean [priv]
rbean     31266  0.0  0.1   4424  2632 ?        S    09:29   0:00 sshd: rbean@pts/1
bigred    31308  0.0  0.0   1584   804 pts/0    S+   09:29   0:00 grep sshd
root      88863  0.0  0.1   4488  3688 ?        Ss   May18   0:00 sshd: bigred [priv]
bigred    88867  0.1  0.1   4688  2864 ?        S    May18   2:20 sshd: bigred@pts/0
```

---

### 子程序與父程序
```
$ nano parent.sh 
#!/bin/sh
echo "parent.sh running"
./child.sh

$ nano child.sh
#!/bin/sh
## -o，指定輸出欄位。comm，command欄位。-p，查詢pid。PPID，父項程序的PID
parent="$(ps -o comm= -p $PPID)"
if [ "$parent" != parent.sh ]; then
    echo this script should be directly executed by parent.sh, not by $parent
    exit 1
fi

echo "child.sh proceeding"
```


```
PPID，父程序的pid
bigred@alp:~/mygo$ echo $PPID
88867
$$ ，程序本身的pid
bigred@alp:~/mygo$ echo $$
88868

bigred@alp:~/mygo$ pstree
 | | \-+- 88867 bigred sshd: bigred@pts/0
 | |   \-+= 88868 bigred -bash
```

```
bigred@alp:~$ nano other.sh
#!/bin/sh
echo other.sh running
./child.sh

bigred@alp:~$ chmod +x other.sh

bigred@alp:~$ ./other.sh
other.sh running
this script should be directly executed by parent.sh, not by other.sh
```

---

### 研究 GCP 上 Ubuntu Linux 的父子程序

- Ubuntu Linux 啟動的 daemon 一定比 Alpine linux 還多，因為啟動的功能比較多所以功能比較強大
    > daemon 背景執行程序。
- Ubuntu linux 第一支程式是systemd

查看 pstree 的結構

```
sreantony@datacenter:~$ pstree -A -sp
systemd(1)-+-accounts-daemon(488)-+-{accounts-daemon}(490)
           |                      `-{accounts-daemon}(509)
...
           |-sshd(730)---sshd(868)---sshd(967)---bash(968)---pstree(1308)
           |-systemd(871)---(sd-pam)(872)
           |-systemd-journal(171)
           |-systemd-logind(824)
           |-systemd-network(450)
           |-systemd-resolve(454)
           |-systemd-udevd(201)
           `-unattended-upgr(712)---{unattended-upgr}(718)
```

- `pstree` 參數說明
- `-p` ，Show PIDs.  PIDs are shown as decimal numbers in parentheses after each process name.  -p implicitly disables comes after each process name.  -p implicitly disables compaction.
- `-s` ，Show parent processes of the specified process.


### **程序與環境變數**
```
$ nano doecho.sh
#!/bin/sh
PLACE=Hollywood
echo "doecho says Hello " $PLACE

$ chmod +x doecho.sh

$ ./doecho.sh
doecho says Hello  Hollywood

$ nano echoplace.sh
#!/bin/sh
echo "echoplace says Hello " $PLACE

$ chmod +x echoplace.sh 
```
無法顯示 `$PLACE`
```
$ ./echoplace.sh 
echoplace says Hello 
```

再度修改，讓 `echoplace.sh` 變 `doecho.sh` 的子程式
```
$ nano doecho.sh 
#!/bin/sh
PLACE=Hollywood
echo "doecho says Hello " $PLACE
./echoplace.sh
```
還是沒有顯示
```
$ ./doecho.sh 
doecho says Hello  Hollywood
echoplace says Hello 
```

`export` 變數

```
$ nano doecho.sh 
#!/bin/sh
export PLACE=Hollywood
echo "doecho says Hello " $PLACE
./echoplace.sh
```

成功在子程式顯示父程式的變數

```
$ ./doecho.sh 
doecho says Hello  Hollywood
echoplace says Hello  Hollywood
```
---

### **程序的中斷信號**

```bash=
$ nano sleep30.sh
#!/bin/bash

trap "echo signal received!" SIGINT

echo "The script pid is $$"
sleep 30
```
- `SIGINT` 是 Linux 系統裡面的 `Ctrl +C` 的代號
- `trap` 是一個 shell 內建命令，它用來在腳本中指定信號如何處理。比如，按 `Ctrl+C` 會使腳本終止執行，實際上系統發送了 `SIGINT` 信號給腳本進程，`SIGINT` 信號的默認處理方式就是退出程式。如果要在 `Ctrl+C` 不退出程式，那麼就得使用 `trap` 命令來指定一下 `SIGINT` 的處理方式了。

執行
```
$ ./sleep30.sh 
The script pid is 39967
^Csignal received!
```

撰寫永不停止程式

```bash=
$ nano nostop.sh
#!/bin/bash
# trap ctrl-c and call ctrl_c()
trap ctrl_c INT

function ctrl_c() {
   echo -e "\n** Trapped CTRL-C"
   # 下式顯示游標
   tput cnorm
   exit 0 
}

clear
# 下式隱藏游標
tput civis

while [ 1 ]
do
  echo -ne "\033[10;10f Hi nobody : "   
  date
done

```
> `INT` 就是 `SIGINT` 的簡寫

執行 `nostop.sh` 程式

```bash!
$ chmod +x nostop.sh

$ ./nostop.sh
./nostop.sh: line 14: tput: command not found





          Hi nobody : ^Ce Jul 20 16:26:48 CST 2021
** Trapped CTRL-C


[註] 直接中斷程式, 按 ctrl+c 複合鍵, 中斷程式執行
```

安裝 `tput` 命令

```
$ sudo apk add ncurses
```

執行

```
$ ./nostop.sh
```

output : 

```






          Hi nobody : Thu May 19 12:59:00 CST 2022









```
---

### **程序的暫停與中斷**

執行 `nostop.sh` 程式
```
$ ./nostop.sh
```

按 `Ctrl +Z` 暫停，並推到背景休息
```
[1]+  Stopped                 ./nostop.sh
```
打`bg`，background 的意思
```
bigred@alp:~$ bg
[1]+ ./nostop.sh &
```
> 會出事，不要打bg

打`fg`會讓程式繼續在情景執行，按 `Ctrl +C` 中斷

```
$ fg
./nostop.sh
Thu Jul 22 18:06:45 CST 2021

          Hi nobody : ^Cu Jul 22 18:06:57 CST 2021

```

---

## 前景與背景程序

前景
- A process that connects to the terminal is called a foreground job. A job is said to be in the foreground because it can communicate with the user via the screen and the keyboard.

背景
- On the other hand, a process that disconnects from the terminal and cannot communicate with the user is called a background job.


### 背景執行程序

撰寫永不停止程式 ( 輸出數字到 `/tmp/counter` 檔 )

```bash=
$  nano nostop01.sh
c=0; echo "" > /tmp/counter
while  [  true  ]
do 
   echo  "$c"  >>  /tmp/counter
   let  "c=$c+1"
done

$ chmod  +x  nostop01.sh
```

- `c=0; echo "" > /tmp/counter` ，宣告變數 c = 0，然後將一個換行符號重導到 `/tmp/cunter` 這個檔案中
- `while [ true ]` ，條件式永遠成立，所以迴圈會一直被執行
- `echo  "$c" >>  /tmp/counter` ，將變數 `c` append 到 `/tmp/cunter` 這個檔案中
- `let  "c=$c+1"` ，宣告變數 `c` 會等於變數 `c` + 1，所以迴圈跑一次，變數 c 就會 + 1
- 程式第一行沒宣告 `#!/bin/bash` 的話，推背景執行程式時，打`ps -eaf | head -n 1; ps -eaf | tail -n 15`  去看會發現 pid 對應的 command 只會顯示 `-bash` ，所以這行一定要打

直接背景執行命令
```
bigred@alp:~$ ./nostop01.sh &
[1] 85764    ## PID
```

檢查程式有沒有執行
```
bigred@alp:~$ tail -n 2 /tmp/counter
1374701
1374702
bigred@alp:~$ tail -n 2 /tmp/counter
2411603
2411604
```

移除背景程序

```bash!
# 強迫停止程序
bigred@alp:~$ kill -9 85764

# 送出正常關機信號(15) 給 pid 2552
$ kill -15 25515

bigred@alp:~$ tail -n 2 /tmp/counter
6320287
6320288
[1]+  Killed                  ./nostop01.sh
bigred@alp:~$ tail -n 2 /tmp/counter
6320287
6320288
```

如果使用前景執行，按 Ctrl +Z 暫停
```
bigred@alp:~$ ./nostop01.sh
^Z
[1]+  Stopped                 ./nostop01.sh
```

檢查真的沒在跑了
```
bigred@alp:~$ tail -n 2 /tmp/counter
311454
311455
bigred@alp:~$ tail -n 1 /tmp/counter
311455
```
按`bg`，讓他在背景繼續執行
```
bigred@alp:~$ bg
[1]+ ./nostop01.sh &
```
再次檢查
```
bigred@alp:~$ tail -n 3 /tmp/counter
4977563
4977564
4977565
```
> 真的有在跑！

如果再打`fg`
```
bigred@alp:~$ fg
./nostop01.sh

```
> 前端的游標會被佔用，如果想暫停就按 Ctrl +Z，如果不想跑了就按 Ctrl +C。

### 背景執行程序範例二


```bash
$ sudo mkdir -p /opt/www; echo "<h1>go out</h1>" | sudo tee /opt/www/index.html

$ httpd
2021/04/20 22:57:16 Serving /opt/www/ on HTTP port: 8888
^C

$ httpd &
[1] 14527
```

- `httpd` ，is the Apache HyperText Transfer Protocol (HTTP) server program.
- Usage of httpd:
    -   `-d` string
        the directory of static file to host (default "/opt/www/")
  - `-p` string
        port to serve on (default "8888")

透過 `netstat` 看網站開的 port : 8888

```bash
$ netstat -anp
```

- `netstat` ，network statistics
- 用於故障排除和配置的網絡工具，也可以作為網絡連接的監控工具。
- `-a` ，To list all listening ports, using both TCP and UDP
- `-n` ，Don't resolve names
- `-p` ，Displaying service name with their PID number


output : 

```bash
netstat: showing only processes with your user ID
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 192.168.61.131:22       192.168.61.1:50895      ESTABLISHED -
tcp        0      0 192.168.61.131:22       192.168.61.1:50900      ESTABLISHED -
tcp        0      0 192.168.61.131:22       192.168.61.1:50868      ESTABLISHED -
tcp        0      0 :::8888                 :::*                    LISTEN      28980/httpd
tcp        0      0 :::22                   :::*                    LISTEN      -
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node PID/Program name    Path
unix  2      [ ACC ]     SEQPACKET  LISTENING      20611 -                   /run/udev/control
unix  2      [ ]         DGRAM                     20734 -                   /var/run/chrony/chronyd.sock
unix  4      [ ]         DGRAM      CONNECTED      19786 -                   /dev/log
unix  3      [ ]         STREAM     CONNECTED      30218 -
unix  3      [ ]         STREAM     CONNECTED      29713 -
unix  3      [ ]         STREAM     CONNECTED      30219 -
unix  2      [ ]         STREAM     CONNECTED      30214 -
unix  3      [ ]         STREAM     CONNECTED      29714 -
unix  2      [ ]         DGRAM      CONNECTED      19871 -
unix  3      [ ]         DGRAM      CONNECTED      20615 -
unix  3      [ ]         STREAM     CONNECTED      20147 -
unix  2      [ ]         STREAM     CONNECTED      29708 -
unix  3      [ ]         DGRAM      CONNECTED      20616 -
unix  2      [ ]         STREAM     CONNECTED      20133 -
unix  2      [ ]         DGRAM      CONNECTED      20730 -
unix  3      [ ]         STREAM     CONNECTED      20148
```

用 `curl` 來連連看網站

```
$ curl http://localhost:8888
<h1>go out</h1>
```

- `curl`，Client URL 的縮寫
- `curl [options/URLs]`
- 支持通過各種網絡協議進行數據傳輸。 它通過指定相關的 URL 和需要發送或接收的數據與 Web 或應用程序服務器進行通信。


## 程序與環境變數

```bash!
# 顯示系統中的環境變數
$ env
SHELL=/bin/bash
CHARSET=UTF-8
PWD=/home/bigred
LOGNAME=bigred
GWIF=eth0
HOME=/home/bigred
LANG=C.UTF-8
GW=192.168.61.2
SSH_CONNECTION=192.168.61.1 51910 192.168.61.131 22
NETID=192.168.61
IP=192.168.61.131
TERM=xterm-256color
USER=bigred
SHLVL=1
PAGER=less
PS1=\u@\h:\w$
SSH_CLIENT=192.168.61.1 51910 22
LC_COLLATE=C
PATH=/home/bigred/bin:/home/bigred/vmalpdt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAIL=/var/mail/bigred
SSH_TTY=/dev/pts/0
_=/usr/bin/env
```

- `/etc/profile` ，只要在 Linux 系統登入成功，第一個被執行的程式，不管是用 SSH 命令登入，還是其他登入方法，只要登入成功，就會執行 `/etc/profile` 這支程式
- `export` ，宣告全域變數

## 解析老師的 /etc/profile

```bash!
# Alpine 的啟動系統是 openrc, 登入前會執行 /etc/local.d/rc.local.start, 登入後會執行 /etc/profile
gw=$(route -n | grep -e "^0.0.0.0 ")
export GWIF=${gw##* }
ips=$(ifconfig $GWIF | grep 'inet ')
export IP=$(echo $ips | cut -d' ' -f2 | cut -d':' -f2)
export NETID=${IP%.*}
export GW=$(route -n | grep -e '^0.0.0.0' | tr -s \ - | cut -d ' ' -f2)
export PATH="/home/bigred/bin:/home/bigred/vmalpdt/bin:$PATH"
# source /home/bigred/bin/myk3s
clear && sleep 2
echo "Welcome to Alpine Linux : `cat /etc/alpine-release`"
[ "$IP" != "" ] && echo "IP : $IP"
echo ""
```

- `gw=$(route -n | grep -e "^0.0.0.0 ")` ，從不解析名字的路由表，找 Default Gateway
    - `route -n` ，顯示不解析名字的路由表
    - `|`，上一個命令的輸出作為輸入傳遞給下一個命令。
    - `grep -e` ，指定字串作為查找檔內容的範本樣式
        - `-e` 的目的實際上只是為了消除正則表達式以破折號開頭的轉義。
    - `grep -e "^0.0.0.0 "`，指定要 `0.0.0.0` 開頭的那一行
        - `0.0.0.0` ，表示所有 IPV4 的網際網路
    - `gw=$()`，宣告變數 `gw` 等於括號裡的命令執行結果
    - ```bash
      $ route -n
      Kernel IP routing table
      Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
      0.0.0.0         192.168.61.2    0.0.0.0         UG    202    0        0 eth0
      192.168.61.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
      ```
    - 以上面路由規則來說，只要目的地 ( Destination ) 是 `0.0.0.0` 的封包，就會經由 `eth0` 這張網卡，將封包往 `192.168.61.2` 這個 Gateway 送
    - 第二行 Gateway 的 `0.0.0.0` ，代表沒有特別指定 Gateway ，會由 `eth0` 這張網卡，將封包往目的地 ( Destination )為  `192.168.61.0` 這個 Network ID 裡面的機器送
    - output : `0.0.0.0 192.168.61.2 0.0.0.0 UG 202 0 0 eth0`
- `export GWIF=${gw##* }`，宣告會將封包往 Default Gateway 送的網卡設為環境變數
    - `${變數##* }` : 從頭開始刪除到最後一個空格(含)為止
    - output : `eth0`



```bash!
$ nano su
#!/bin/bash

if [ "$USER" != "" ]; then
   echo "Hi $USER"
else
   echo "Nobody"
fi

$ chmod +x su

$ ./su
Hi bigred

$ echo $PATH
/home/bigred/bin:/home/bigred/vmalpdt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

$ which su
/bin/su

$ sudo mv su /usr/bin

$ which su
/usr/bin/su

$ su
Hi bigred

$ sudo mv /usr/bin/su ~/

$ which su
/bin/su
```

如何修改 PATH 系統環境變數 ?


## 小總結

Process
- 定義
	- 運作中的 Program
- 運作流程
	- 運作時會把程式的 PID、執行者的權限參數（UID和GID）和程序的程式碼（machine code）以及相關資料從硬碟載入記憶體去執行。
		- `setuid` 執行檔案時，Process 的 UID 為檔案擁有者，而非檔案執行者
- 程序管理
	- `ps` 使用條列式清單查看執行中程序
	- `pstree` 樹狀顯示程序資訊
	- Ubuntu Linux 第一支程式為 `systemd`
	- Alpine Linux 第一支程式為 `/sbin/init`
	- 子程序中用到父程序的變數須在父程序宣告變數前加 `export`
- 運作介面
	- 前景執行
	- 背景執行，指令後面加 `&`
- 動作
	- 可按 `Ctrl +C` 停止程序
	- 可按 `Ctrl +Z` 暫停程序
		- `fg` 讓暫停的程式在前景恢復執行
		- `bg` 讓暫停的程式在背景恢復執行
- 環境變數
	- `$$`，目前程序的 PID 
	- `$0`，目前程序的名稱 
	- `$PPID`，目前程序的父程序的 PID



---

###### tags: `系統工程`

















