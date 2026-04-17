# SRE課程頁面 2 (Container, K8s)
[SRE課程1](https://hackmd.io/@RayLin9981/Sy6eRKHzt)

[專題製作部分](https://hackmd.io/@RayLin9981/Hk6m_HFIY)

[作業與 yaml檔紀錄](https://hackmd.io/@RayLin9981/S1zTa1h7K)

[TOC]

# 陳松林老師 Container 關鍵技術與實務運用
接下來要學這些東西，
- docker, kubernetes 在2021已成為電腦基礎概論(全球)
    - docker 研發出 container(貨櫃) ，可以代表一台電腦
        - 比起虛擬技術輕量許多
        - 重點要放在 `application container`
    - kubernetes：管理 container , 由 google 公司研發開源出來
    - 以上兩個技術是為了做出底下兩個應用產生產值
- hadoop : 大數據分析系統，商業智慧，為公司分析商業報告
- spark ： AI 人工智慧，做預測，明年要賣什麼


學習以上架構大概需要 6 台機器比較好，目前就使用虛擬機
![](https://i.imgur.com/XKJ6tD7.png)
我們預計使用自己寫架構，所以ansible 不會用到
VMware Player 至少要 16G 記憶體起跳
alpine 現在很火紅，有以下特色

- 很小，執行效率很高
- 資訊安全等級高，因為很多軟體都沒有(沒有攻擊的對象)


podman 也是 container 技術，很可能會取代 docker
musl： C 的lib
Linux 是由幾百萬行的程式碼組成的，主要的精神是可以 "客製化" 作業系統

- lib=library=程式庫 (儲存別人寫好的程式、套件)
- busybox (容量800多K) 可以模擬出很多 Linux 的指令， e.g. `ping,rm,mv,ifconfig,ls`
    - `busybox ls -alh` ,其中 -h 為 human-readable ，把容量的部分轉成人類易讀的 (KB,MB等)
    - 一般來說，alpine 的指令都會指向 `/bin/busybox` (除非有改過)，例如 `/bin/cat` 則會到 -> `/bin/busybox`
    - sudo apk update 更新最新的套件清單，sudo apk upgrade 將以安裝的套件去對照清單，然後更新
    - 如果需要圖形介面可以使用 `XWindow` 其他 Linux 發行版也是用這個
    - alpine 這種輕量化的 OS 已經是未來趨勢

terminal 上面會執行 shell (貝殼程式，有sh,bash,csh等) ，來提供使用者輸入指令跟主機溝通
Linux 一開始就提供了多工的介面，達到協同作業，連接很多終端機
bash script 也是一種程式語言，<font color=red > 熟bash script 月薪五萬以上!!</font>
### 認識 Computer Program

program 是一個檔案(節目)，要有執行權限，interpreter 一行一行的執行 , compiler 則是把人類看得懂的程式轉成電腦看得懂的(011010 bin 檔) machine code  再執行
shell script 的 interpreter 是由開頭的 #! 來指定(與python類似) ，如果是 `bash script` 就會指定 `#!/bin/bash` 來翻譯
> lint 是一種可以檢查程式語言錯誤的程式 [ShellCheck](https://www.shellcheck.net/) 可用來檢查 Shell script 的錯誤，也可以使用 `apt` 來安裝

`go` ： 將 python 的簡潔 ， Java 的物件 ， C 的速度，集於一身。docker, kubernetes 都由此開發 (go 語言還沒紅，但他開發的產品紅了)
指標是直接指向記憶體位置，所以快(就像餐廳，等服務生帶與直接找位置坐下速度的差異)
`head -c 10` 看前10byte

> alpine 的 acl (不是 FACL!!) 功能需要額外啟動，預設是沒有


JAVA 經過編譯後產生 JVM 看得懂的 byte code(.class) ，再交由 JVM 直譯來一行一行執行程式
- 原本經過編譯後的 01 碼是綁定於某一個 OS ，但是如果是 JAVA 的 JVM 則可以使用同一個 class ，透過不同的 JVM 共用
    - JVM 與 一般的 VM 不同，是可以思考成 "只執行一次的 container" ，但那種架起來的網站就是一直執行的 JVM (也可以指定大小 memory 等)
    - JVM 也可用來執行 python 稱作 Jpython
    - JVM (一代)是使用 interpreter 的方式來執行，**從 JAVA 6 (jdk1.6)之後就棄用了(效率不好)**，目前最廣泛使用的為 JAVA 8(jdk1.8)
    - 可以把 java 這個指令就當作是 JVM (讀取.class 檔案的 bytecode 並執行)
- ![java 架構](https://i.imgur.com/Dmlw36p.png)
$PATH 告知 OS 哪裡會存放可執行的檔案，會依照先後順序找檔名，並且執行。若要執行的檔案不放在 $PATH 的目錄裡的話則需要使用相對/絕對路徑來執行
interpreted/compiled 不是程式語言的特色，而是要看如何 "實作"，才能確定是哪種
- 以Python 舉例，原本為 interpreter ，但如果是 Jpython 的話就不一定
### 認識 Process

process (程序) ，thread (執行序)
- 有沒有執行的權限，與本身是否能被執行
- Process 由下面三個成分組成，一定是在 memory 裡面跑
    - PID
    - 使用者的一些權限屬性參數資訊 UID,GID 等
    - ~~電腦看得懂的 machine code~~ ，若直譯式的是存放文字(下面有圖，上面會放bash來負責轉換給CPU聽)，編譯的話就是放編譯後的檔案 (其實執行時就是指定 bin 檔案)
    - ~~#! 可稱作 bang bang? 當作指定直譯器~~，[老師說是原文書的用法，不過只查到 hash-bang 的講法@_@](https://zh.wikipedia.org/wiki/Shebang)
![](https://i.imgur.com/FQ3RUpP.png)


[烏龜 shell scipt 原文 PDF](https://doc.lagout.org/operating%20system%20/linux/Classic%20Shell%20Scripting.pdf)

`timeout 20s yes >/dev/null &`  timeout 可以指定多久要退出
- yes 是一個會不斷產生 y 的程式，常用於 cisco 工程師在設定一堆裝置時使用 (不用打y)

每一個 tty 都是一個終端機介面,/pts為遠端登入的(SSH) ，在一開始開機時就會占用 tty1 做為登入用，在登入之後會執行 `/etc/profile` (program) ，為登入時第一個 process
- rc 檔可說是 run commands 或 run control ，以.bashrc 來說，就是放了一些指令來執行

如果要關閉 root ，就把密碼設過期就好了
### 程序的中斷信號
 `ps -o comm= -p $PPID` 可以看父親為哪個 process ,如果是用 ssh 登入的為 sshd , 如果是本機登入的為 login
 `trap` 可以捕捉電腦發送的 ctrl+c 訊號，在後面可以直接接 function ，trap到之後就執行
 ```bash=
 #!/bin/bash

trap "echo signal received!" SIGINT # 按 crtl+c 後出現
echo "The script pid is $$"
sleep 30
###
#!/bin/bash
# trap ctrl-c and call ctrl_c()
trap ctrl_c INT # 當接收到 ctrl+c 之後，執行 ctrl_c 函式
function ctrl_c() {
   echo -e "\n** Trapped CTRL-C"
   # 下式顯示游標
   tput cnorm # 需要安裝 sudo apk add ncurses
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
[一些bash 的變數替換指令介紹](http://benjr.tw/96951)


執行權限練習：如果一個檔案只有 x 的權限而沒有 r 的權限，則沒辦法執行
```bash= 
ls -l onepingonly.sh
-rwxr-x--x 1 bigred bigred 82 Sep 27 10:15 onepingonly.sh
zbean@alp:/home/bigred$ ./onepingonly.sh
/bin/bash: ./onepingonly.sh: Permission denied # 即使有 x 的權限，沒辦法 r 也沒辦法執行
```
接下來是 `passwd` 的練習，先檢查了 `/etc/shadow` 的權限，再來使用 `passwd` 來改使用者的密碼
`which` 是透過 `PATH 環境變數` 來搜尋指令位置
### Setuid
setuid ：不需要 r 的權限也可以執行(但 shell scirpt 沒辦法，會忽略SUID，編譯後的 go 是可以只有 x 權限的)，在執行之後(不論是誰)都是使用 owner 的 UID 來執行 process
- 執行到底需不需要 r 權限? -> 若是編譯後的 machine code , 則直接整份搬到 memory 中執行，那 shell script 就必須要有 r 的權限(bash 還需要一行一行讀入，並且 SUID 也沒有任何意義，會直接被省略)
- 也就是說以 passwd 為例，檔案指向 /bin/bbsuid 權限為 `--s--x--x root root` ，當 other 執行時，則會帶入 root 的 uid ，進而有權限去修改 /etc/shadow 這個檔案
- suid 適用於 <font color=red>二進制檔案(也就是 machine code) </font> ，沒辦法用於 interpreter 的程式碼(shell script

`nc -w 1 -z 120.96.143.15 22` ,`-w` 設定 timeout` -z` 不輸出東西，`22` 代表 ssh 的 port , 下一行再用 `$?` 來檢查有沒有成功
- 是用來替代 `ping` 的網路是否有通，目前的趨勢是直接把 `ping` 拿掉(漏洞之一) ，然後用就 `nc` 來檢查
- 測試是否為 windows ：使用 445 port 來測試, `nc -w 1 -z 120.96.14.15 445` ，很多的病毒都從這裡侵入，是微軟心中最軟的那一塊
- container 後來都不裝 `ping` 囉
- `fping -c 1 -g -q $netid &> "/tmp/netid.chk"` -c 設定 timeout , -g 設定網路位置，格式有兩種 192.168.24.0/24 或 192.168.24.1 192.168.24.254，-q 只顯示結果
    - `nip=$(grep "min/avg/max" "/tmp/netid.chk" | cut -d' ' -f1 | tr -s ' ')` 每一行有 `ping` 成功的都有 min/avg/max 這行，也就是過濾掉 ping 失敗的那些IP
- nc `-w` 設定 timeout , `-z` 只確定該 port 有開啟而不傳送任何資料 (跟 ping 不同) , 最後的數字為檢查的 port
    - e.g.  `nc -w 2 $ip 22 &>/dev/null` 檢查 `22` port


![process 運作架構](https://i.imgur.com/vjtXyba.png)
user program ：其實就是由使用者啟動的 process -> 一定會去呼叫 System Library (由 C 語言寫的，也就是 Liunx 提供)
- process 就一定會用到
- hacker 攻擊的方式就是：使用者寫的程式會去用 System Library , 就可以進入到 kernel
    - 也就是說，hacker 就有能力使用到 kernel 做壞壞的事 (竄改使用者寫的程式)
- seccomp (security computing mode ) :第一層防守，保護 System call (其實就是 System Library)(也是 SELinux 的一部分)
    - 是控制 process 的 syscall , 因此是控制 PID 的權限，**所以說甚至是 `root` 也可以做限制!!**
    - 可使用 `strace -qcf mkdir /tmp/x` 來看一下 `mkdir` 用到哪些 system call
    - -q 壓縮一些輸出 , -c 表格化顯示執行的時間,次數和出錯的次數等(summry) , -f 顯示所有子程序(使用 fork 呼叫的)

go 為 通用型的 程式語言，也就是什麼都可以做啦(未來趨勢，因此範例都用這個)
alpine 剛裝起來，就只會有基本的 `C lib(system call)` , `busybox` , 與 `apk` 可以來擴充
go的 seccomp 實作：
- 透過 go 裡面的 libseccomp-golang 來和 Linux 的 seccomp 溝通
- 使用以上 lib ，位置是在 system lib 的上方，再來控制 seccomp 是否放行
- 接著再使用 go 裡面的 `syscall `來呼叫
- 在程式語言中的` os lib` 為 比較高階的 `syscall` ，使用 os.Getpid() 時 ，若無權限則回傳 -1

> 在 ubuntu 裡面，如果你要用 sudo 來執行自己寫的 shell script , 則需要加 x 的權限，否則打./連看都看不到，並且會說 sudo command not found [stackoverflow](https://stackoverflow.com/questions/12996397/command-not-found-when-using-sudo)
### Linux System Call & Seccomp  


如果需要再進一步管理 **特定使用者是否能在特定目錄新增刪除檔案(這邊通常是限制root)** 的話，Centos 使用 SELiunx , ubuntu 則使用 AppArmor
### Linux Capabilities 

> 問題：Capabilities 中的 SetUID 與檔案權限的 SetUID(owner 的s) 是不是完全不一樣?

Capabilities：`sudo setcap  cap_setuid+ep  /home/ybean/python3` <font color=red>~~讓 python3 擁有設定其他檔案的 SUID 的功能~~ ，是有能力改變其他 process UID 的能力 ，並非設定 SUID 在 python3 上</font>

- `getcap -r / 2>/dev/null` 從根目錄開始，搜尋有哪些程式擁有 cap 能力(目前就會顯示 `/home/ybean/python3` )
- `./python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'` os.setuid後 ，會去更改後面的 `/bin/bash` uid ，畫面會立刻變成 root 的身分，可以利用 id 看一下 ， group 還是原本的使用者
- 所以就可以利用一般使用者來進行管理，先設定一個能 getuid 的 python3 ，就可以做任何事了，不需要使用 root

cloud native ：形成雲端技術的基礎東西
#### 程式檔案的 capabilities
一共有三個：
- Permitted ： 與 cap_bset 做 and 運算，若都有就會提供 P'(permitted) 的cap
- Inheritable： 表示能夠被當前程序執行的程式繼承的能力。
- Effective：主要影響 P'(effective)


> P'(permitted) = (P(inheritable) & F(inheritable))| (F(permitted) & cap_bset)
P'(effective) = F(effective) ? P'(permitted) : 0
P'(inheritable) = P(inheritable)    [i.e., unchanged]
execve(2) 就是執行程式的 syscall，以這個動作的先後來區分`P'` 與 `P`
[process與thread的差異](https://totoroliu.medium.com/program-process-thread-%E5%B7%AE%E7%95%B0-4a360c7345e5)
#### Capabilities and execution of programs by root
1. 若是 <font color=red>set-user-ID-root program(是指檔案權限的suid嗎??)</font>  則會給予所有的權限：all capabilities enabled -> 沒錯，意思是：若是由 root 來執行的 process , 則所有的 cap 都會給(但還是會被seccomp管理)
2. If a set-user-ID-root program is being executed，和上面一樣，則 effective bit 設置為 1

#### Thread capability sets
一樣跟檔案的 cap 分成三個
- Permitted ： Thread(待確認)，是否被許可有權力，但實際有沒有能力還是得看 Effective 的能力
- Inheritable ： 從上面繼承下來的
- Effective ： 能力，在被允許有能力之後，才能展現自己的能力


## 10-Container-Overview
![](https://i.imgur.com/TeIOVae.png)
- APP = process，也就是打` ps aux `顯示的東西
- Host OS ：PC通常為 windows ， Server 為 Liunx 為主
- Server(PC) = 實體機器電腦
- 所寫的程式(app)要執行成功，一定需要"相依檔"(可以是自己寫的程式binary或別人寫好的函式庫library)
- 而 isolated 技術就是將 process 及他的相依檔做隔離後產生個別的電腦，稱為 container
- container 沒有開機程序，因為共用 kernel,
    - 若擔心 container 導致 HostOS(kernel) 受傷害，可以設定 `seccomp`,`capabily`,`setuid` 等，來限制 container 存取 kernel 的權限

1. Namespace - 用來提供軟體程序不同的隔離功能
2. CGroup - 用來分配硬體資源(CPU,Memory等)
3. Chroot - 針對指定的軟體程序，改變它的根目錄，把 process 的根目錄改掉
4. Bridge/slirp4netns(撥號網路) - 建立虛擬網路, 連接 Internet 
5. Overlay2 - 用來建立 Container 的檔案系統，和其他的完全不一樣
    6. 複習：windows 的檔案系統 NTFS , Linux ：EXT4,ExtFAT ,XFS

### 探索 Chroot
Chroot : 針對 process 的根目錄進行修改

```bash=
# 使用 bigred 來執行
mkdir -p r2d2/rootfs/{bin,lib,proc}; cd r2d2
cp /bin/sh rootfs/bin; cp /lib/ld-musl-x86_64.so.1 rootfs/lib/
# musl 為 c 語言的 lib
sudo chown -R root:root rootfs
sudo chroot rootfs /bin/sh # chroot 把process 的目錄轉換至指定的目錄區
#執行 chroot 後，一堆指令都沒辦法使用了，可以使用 pwd 發現在根目錄
# 但可以使用 exit 離開 
sudo chroot rootfs /bin/bash # 會報錯，因為 rootfs 這個資料夾並沒有 /bin/bash

```
- ldd 可以查詢相依性，用法 `ldd /bin/sh` ，會出現兩個檔案(但其實是同一個檔案)
- chroot 把 process 的目錄轉換至指定的目錄區(使用 pwd 發現在 /)
- 會只剩下 sh 提供的簡易功能，如 exit,echo 等(可以按兩下tab看),ls,ifconfig 都不能使用

`curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-minirootfs-3.14.1-x86_64.tar.gz`

`tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd ..` 
所有放在 rootfs/bin 的檔案，全部都是透過 busybox 提供(簡易版的)(使用 link 的方式)
> chroot 就是 docker run 的其中一個 process

windows 如果不想要裝防毒軟體，可以切換至**沒有管理員權限**的帳號登入，這樣就很安全了
`$ sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh`
<font color=red>**在操作 container 時，若是使用一般帳號管理而不使用 root ，安全性立刻飆升**</font>

chroot 可以用來誘捕駭客
- 意思是設計出一個系統，就算被駭客攻擊也沒差(駭客以為是根目錄，但在實際的電腦上其實只是一個另外的目錄)

### 探索 Overlay2
![](https://i.imgur.com/7RlRuos.png)

Overlay2 為堆疊型的檔案系統，分成 `lowerdir`(ro), `upperdir`(rw), `workdir`(功能用，實際上不存放東西), `merged`
- lowerdir ：唯讀，檔案有幾個就是幾個
- upperdir ：可讀寫，可以被管理、修改的部分
    - 視角是由上往下看，如果上面(upper)有跟(lower)同名的檔案的話，就不會去抓 lower 的檔案
    - mount -t 指定type , -o 根據前面的type 給予選項，兩次的 overlay 是固定用法
    - `mount -t overlay overlay` 這樣的用法，第二次算是給名稱，~~不過似乎不能亂改~~ -> 可以給自己要的名字, e.g. ov2
- merged ：主要是在呈現以上兩個目錄的內容
    - 直接在 merged 新產生的檔案，實際上是在 upper 創建 ；在 merge 刪除檔案的話，假如  upper 跟 lower 有同名的檔案，則會在 upper 的位置產生一個 容量為0的檔案
        - 產生的檔案為 c 字元檔，此操作稱為 **whiteout**：
        - 為 overlay 檔案系統為了不讓 merge 刪除之後檔案還在(lower提供的) ，所以蓋了一個暗樁在上面阻擋 merge 的視角，使其看不到 lower 的檔案
    - **copy_up** ： 當在編輯 merged 的 lowerdir 檔案時，實際上是這樣：
        - upperdir 先從 lowerdir copy 一份到 upperdir，再進行編輯
        - 也就是說，後來在 merged 上看到的檔案，實際上是在 upperdir 的位置，並且 lowerdir 內的檔案並沒有被變動到 

~~先進到CNT.2022.v1目錄 cd CNT.2022.v1
▲跑老師建立的vmrungo.bat mkct.bat
▲runct.bat就會打開兩個alpine(docker,podman)~~

### CGroup (Control Group )
CGroup 對象是 process (PID) 。用來限制、隔離資源使用，分配資源
**為何要學 CGroup? ：因為接下來要學 container ，要依據每一個 container 的需求來調配硬體資源**
![](https://i.imgur.com/xnX3iG7.png)
- 右邊指的是網路部分，可以限制網路速度等等的
    - 以 cat 6 來說，最高可傳輸到 100M ，但可以限制成 20M
- 左邊是管理 CPU 與 memory 部分
- 中間來說，httpd 被執行之後就成為了一個 process(有PID)，接著就被分配到左右兩個CGroup 來限制他的存取資源能力
    - `ls -al /sys/fs/cgroup` 若有東西代表有啟動 CGroup ，並且所有的東西都是目錄
    - task 裡面放的是 PID ，PID 被放進去之後就開始限制 process 存取 (e.g. memory) 的能力
    - 以 /sys/fs/memory 來說
        -  memory.limit_in_bytes 限制 process 能使用多少的記憶體

若有很多種不同的管理方法，則可以在目錄內再創建一個目錄
- `sudo mkdir /sys/fs/cgroup/memory/demo` 裡面會自動跑出很多檔案可以供設定
    - mkdir 後會自動把上一層的所有內容 cp 進去唷

#### memory set
也就是使用 `/sys/fs/cgroup/memory` 中的檔案
- memory.limit_in_bytes : 裡面給數字，代表限制的記憶體數量（byte)
```bash=
#!/bin/bash

[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo
echo "100000000" | sudo tee  /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null # 10000000 單位是 byte,約95M
echo $$ | sudo tee /sys/fs/cgroup/memory/demo/tasks &>/dev/null

cat /dev/zero | head -c $1 | tail #$1 指定要使用多少記憶體 e.g. 85M
####執行結果
bigred@alp:~$ ./memlimit.sh 100M
./memlimit.sh: line 7:  8384 Broken pipe             cat /dev/zero
      8385                       | head -c $1
      8386 Killed                  | tail
bigred@alp:~$ ./memlimit.sh 85M
./memlimit.sh: line 7:  8396 Broken pipe             cat /dev/zero
      8397                       | head -c $1
      8398 Killed                  | tail
bigred@alp:~$ ./memlimit.sh 80M
bigred@alp:~$
```
若是 head 的指令要用到 100M ，大約就要給到 116M 以上(因為還是有一些系統需要的空間)

#### CPU
使用 `/sys/fs/cgroup/cpu` 中的檔案
- cpu.shares : 限制 cpu 存取的權限，是依照比例去分的，原始檔案為 <font color=red>1024</font> ，最小值可以給到 2
- 在 cpu.shares 這個檔案中，能給予的最低數字為 `2` ，若打 `1` 進去也會被強制改成 `2` (原因不明)
    - 以下面的範例來說，若是給 512,2048 則是 1:4 (20%,80%) 的比例
    - 若是給 3,7 則是 3:7 (30%,70%) 這樣
```bash=
sudo mkdir /sys/fs/cgroup/cpu/low
sudo chown bigred:root -R /sys/fs/cgroup/cpu/low
#echo 512 | sudo tee /sys/fs/cgroup/cpu/low/cpu.shares
sudo mkdir /sys/fs/cgroup/cpu/high
sudo chown bigred:root -R /sys/fs/cgroup/cpu/high
#echo 2048 | sudo tee /sys/fs/cgroup/cpu/high/cpu.shares
echo 3 | sudo tee /sys/fs/cgroup/cpu/low/cpu.shares
echo 7 | sudo tee /sys/fs/cgroup/cpu/high/cpu.shares
# 以上兩個先測試看看是否數字只是比例問題
#也就是說 low 會佔 30% (3/10) ,high 會佔 70% (7/10)
```
#### CPUset
使用 `/sys/fs/cgroup/cpuset` 內的檔案
- cpuset.cpus : 指定要用哪一顆，若用兩顆則可以寫 `0-1` , 指定使用第一顆就寫 `0`
- cpuset.mems : 關閉 CPU 直接存取記憶體的技術 (DNA 晶片吧) 
```bash=
sudo mkdir /sys/fs/cgroup/cpuset/first
sudo chown bigred:root -R /sys/fs/cgroup/cpuset/first
echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.cpus # 指定要用 core 0 這個CPU
echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.mems # 讓 core 0 不需要做多餘的運算? 把 DNA 關掉
# DNA 指的是讓 CPU 可以不用透過其他額外的晶片就存取到 memory
yes &>/dev/null & # 執行 yes 是一個無窮迴圈，目的是要盡可能使用 CPU 的效能
yes &>/dev/null & # 會出現兩次 PID 下面再微調
echo 11730 | sudo tee /sys/fs/cgroup/cpuset/first/tasks
echo 11732 | sudo tee /sys/fs/cgroup/cpuset/first/tasks
echo 11730 | sudo tee /sys/fs/cgroup/cpu/low/tasks
echo 11732 | sudo tee /sys/fs/cgroup/cpu/high/tasks
top #確認 CPU 佔比
```
`cat /sys/fs/cgroup/cpuset/cpuset.cpus` `0-1` 代表雙核 `0-3 `代表四核
`cat /sys/fs/cgroup/cpuset.cpus.effective`

rootfs :alpine 作業系統的完整檔案系統
:::spoiler 上課練習：cgrun.sh 設定 low,high,all 的 cgroup
```bash=
#!/bin/bash

# killall yes
[ "$1" == "" ] && echo "cgrun.sh [low|high]" && exit 1
OPTION=${1,,}

echo $OPTION
#YESPID=$(ps | grep yes  | cut -d ' ' -f 2 | tail -n 1 )
#echo $YESPID
#echo $$
YESPID=$$

if [ "$OPTION" == low ];then
  echo low
  [ ! -d /sys/fs/cgroup/memory/0929_low_demo ] && sudo mkdir /sys/fs/cgroup/memory/0929_low_demo >/dev/null
  [ ! -d /sys/fs/cgroup/cpu/0929_low_demo ] && sudo mkdir /sys/fs/cgroup/cpu/0929_low_demo >/dev/null
  [ ! -d /sys/fs/cgroup/cpuset/0929_first_demo ] && sudo mkdir /sys/fs/cgroup/cpuset/0929_first_demo >/dev/null
  sudo chown -R bigred:root /sys/fs/cgroup/memory/0929_low_demo
  sudo chown -R bigred:root /sys/fs/cgroup/cpu/0929_low_demo
  sudo chown -R bigred:root /sys/fs/cgroup/cpuset/0929_first_demo

  echo $((512*1024*1024)) | sudo tee /sys/fs/cgroup/memory/0929_low_demo/memory.limit_in_bytes  >/dev/null
  echo 512 | sudo tee /sys/fs/cgroup/cpu/0929_low_demo/cpu.shares >/dev/null
  echo 0 | sudo tee /sys/fs/cgroup/cpuset/0929_first_demo/cpuset.cpus >/dev/null
  echo 0 | sudo tee /sys/fs/cgroup/cpuset/0929_first_demo/cpuset.mems >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/memory/0929_low_demo/tasks >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/cpu/0929_low_demo/tasks >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/cpuset/0929_first_demo/tasks >/dev/null
elif [ $OPTION == high ];then
        echo high
  [ ! -d /sys/fs/cgroup/memory/0929_high_demo ] && sudo mkdir /sys/fs/cgroup/memory/0929_high_demo  >/dev/null
  [ ! -d /sys/fs/cgroup/cpu/0929_high_demo ] && sudo mkdir /sys/fs/cgroup/cpu/0929_high_demo >/dev/null
  [ ! -d /sys/fs/cgroup/cpuset/0929_all_demo ] && sudo mkdir /sys/fs/cgroup/cpuset/0929_all_demo >/dev/null
  sudo chown -R bigred:root /sys/fs/cgroup/memory/0929_high_demo
  sudo chown -R bigred:root /sys/fs/cgroup/cpu/0929_high_demo
  sudo chown -R bigred:root /sys/fs/cgroup/cpuset/0929_all_demo

  echo $((1*1024*1024*1024)) | sudo tee /sys/fs/cgroup/memory/0929_high_demo/memory.limit_in_bytes >/dev/null
  echo 2048 | sudo tee /sys/fs/cgroup/cpu/0929_high_demo/cpu.shares >/dev/null
  echo 0 | sudo tee /sys/fs/cgroup/cpuset/0929_all_demo/cpuset.cpus >/dev/null
  echo 0 | sudo tee /sys/fs/cgroup/cpuset/0929_all_demo/cpuset.mems >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/memory/0929_high_demo/tasks >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/cpu/0929_high_demo/tasks >/dev/null
  echo $YESPID | sudo tee /sys/fs/cgroup/cpuset/0929_all_demo/tasks >/dev/null
else
  echo plz enter high or low
fi

yes &> /dev/null
```
:::

### chroot, cgroup, overlay2 整合
整合起來之後，就越來越像 container 了
- 使用 chroot 切換至新電腦(新目錄)
- cgroup 控管新電腦的硬體使用
- overlay2 獨立出來的檔案系統，可以確保底層的內容為最原始的

```bash=
sudo mount -t overlay ov2 -o \ #名字可以改
lowerdir=/home/bigred/ov2/lower:/home/bigred/c3po/rootfs,\ #可以指定多個路徑，使用 : 隔開
upperdir=/home/bigred/ov2/upper,\
workdir=/home/bigred/ov2/work \
/home/bigred/ov2/merged
##
# 進到新掛載的 ov2 之後執行 chroot
cd ~/ov2; sudo chroot merged /bin/sh
mknod -m 0666 /dev/zero c 1 5 # 產生裝置 /dev/zero
# 會一直產生 ascii 的0 ，也就是 null (再螢幕上看不到)
```
接下來再透過 cgroup 來測試裡面的 memory 使用限制
```bash=
echo $$ # 在chroot裡
5954
#到原本的系統，請開另一個 cmd
sudo mkdir /sys/fs/cgroup/memory/demo
echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes
echo 5954 | sudo tee /sys/fs/cgroup/memory/demo/tasks
# 回到 chroot 的介面
cat /dev/zero | head -c 80m | tail # 成功
cat /dev/zero | head -c 100m | tail # 出現錯誤，因為記憶體不夠
```
然後在這台新電腦測試能不能用 apk 來裝軟體
```bash=
sudo chroot merged /bin/sh
echo nameserver 8.8.8.8 > /etc/resolv.conf
apk update
apk add nano vim tree #可以使用了
```

### 認識 Linux namespace
namespace : process 隔離技術，但 Windows 沒有提供 (甚至新的 win11 也沒有 )
windows 的 docker 實際上是偷偷開了一個 Linux 虛擬機 ，才有 namespace 能用
一共有 8 大項要隔離：使用 `unshare` 命令來實作 namespace
- Mount (mnt) ： mount point 的隔離
- Process ID (pid)：主要是 `/proc` 目錄的隔離，ps 指令是查看該目錄
    - 一共要加上三個參數 `sudo unshare --fork --uid --mount-proc` 才能實作出完整的 pid namespace
- Network (net) ：網路設備的隔離
- Interprocess Communication (ipc)
- UTS (UNIX Time-Sharing) ： 隔離電腦名稱，跟時間無關
    - `sudo unshare --uts sh` 會開啟一個新的 sh , 接下來使用 `hostname` 指令更改名稱，會發現原本的電腦名稱不會動到
- User ID (user)
- Control group (cgroup) Namespace
- Time Namespace

> 2021/10/4 問： 以執行 `sudo unshare  --pid --fork --mount-proc -R /home/bigred/c3po/rootfs sh` 來說
> 1. `/proc` (於實際電腦中為`~/c3po/rootfs/proc`) 目錄為何在使用 pid namespace 後，在外面的終端機看不到裡面的內容呢?
> 2. `--fork` 指令不加的話，會造成 out of memory 的情況是為何
> 3. 在 `~/c3po/rootfs/dev` 我們使用了原本機器的 `/dev` 目錄 (mount) 上去，之後再執行中使用 `sudo umount ~/c3po/rootfs/dev` (把 /dev 掛載拿掉)，為何在 namespace 裡面還看的到 /dev 目錄並且可以使用呢？
> 4. 為何 `unshare` 指令明明是 user command(1) 但卻需要使用 sudo?是為了執行 unshare(2) 的 syscall 嗎？所以如果使用 set_cap 設定適當的 capabilities 到 `/usr/bin/share` 後是否就可以不加 sudo 了呢？ (因為使用 root uid 的話 cap 部分會全部放行)  


#### pid namespace
`/proc` 目錄使用 pid namespace 的內容來記錄 process 執行的情況，`ps aux` 就是去抓那個目錄，`ls -l /proc/self/ns/` 裡面有一些原本的檔案(吧)
- proc 是一個虛擬的檔案系統 ，裡面的內容都是**動態**產生的
- 是由 kernel 產生的 ，因為所有的 process 都是由 kernel 做管理
- 為了要讓其他 process 參考到，所以就把這個記憶體空間掛載(mount)到 `/proc` 目錄
- 所有的 namespace （隔離空間）就是一個記憶體區塊，而所有的 process 資訊都放在 pid namespace
- net namespace 存放網路系統資訊 (`ifconfig`)
    - 所有運作原理都放在裡面，e.g. ip,mac,MTU,netmask,etc..

`sudo unshare --pid --fork sh` ：幫 `sh` 這個 process 產生 pid 隔離空間，`--fork` 代表再多一層才執行 `sh`
- 沒給 fork ， 在執行指令的時候會說 out of memory (原因不明)
因為沒有把 sh 的 /proc 也 mount 到上面，所以沒辦法真正抓到新的隔離空間 (使用 `ps` 依然是抓原來的 `/proc` 內容)
- 使用 `sudo unshare --pid --fork --mount-proc sh` 就可以在裡面用 `ps` 發現 `sh` 的 PID 為 1 了 
    - `--mount-proc` 會自動執行 `mount -t proc /proc /proc` 這個命令
- 接著再使用 `findmnt proc` 就會發現有兩個掛載，一個是原本的，一個是新獨立出來的


> k8s 是由 `sleep` 指令形成的

#### network namespace
Linux 一啟動，就會產生 default network namespace (default 為名稱) 。也就是 container 
- 提供給 process 使用的網路空間，包含 iptable 與 routing table
- `ifconfig` 可以查看
- 在 `/proc/self/ns/` 裡的 `net -> net:[12345124123]` 就是名稱為 default 的 network namespace
```bash=
sudo unshare --pid --fork --mount-proc --net --uts sh # 這裡重點是 net 參數
# 上面那行指令會進到新的 namespace,以下指令請回到原本alp主機
sudo ip link add ve1 netns 1 type veth peer name ve2 netns 4582 # 4582 要替換成 sh 的 PID , 可用 ps aux | grep sh$ 搜尋
sudo ip link set ve1 up
sudo ip addr add 192.168.1.100/24 dev ve1
#接下來去設定 sh 的網卡
sudo ip link set ve2 up
sudo ip addr add 192.168.1.200/24 dev ve2
ping 192.168.1.100 # 可以通

```
![](https://i.imgur.com/WVlXjrV.png)


- net 參數會新增出一個 network namespace 網路隔離環境
    - 在該環境中輸入 `ifconfig -a` 會發現 eth0 消失了只剩下 lo
`sudo ip link add ve1 netns 1 type veth peer name ve2 netns 4582`
- ve1,ve2 網卡名稱 ， 1 ,4582 是 PID, 使用 `sudo lsns -t net` 查看 ， veth 虛擬的 ethernet 裝置 ， peer 代表兩張網卡連接起來
- 設定完後在原本的電腦會產生 ve1 , 在新的 namespace 裡會產生 ve2 網卡
    - 但兩邊都沒有啟動 ， 因此還是要輸入 `ifconfig -a ` 才看得到
> 網路口訣：去頭去尾，前10後5(前server後network)，中間切一半（前手動後DHCP)

### Namespace & Chroot

`sudo unshare --pid --fork --mount-proc --uts -R c3po/rootfs sh` -R 代表就是 `chroot` 指令
- 擁有自己的 pid 與 uts namespace ，差網路裝置
- apk update 失敗的原因 ：沒有 DNS server ，執行 `echo nameserver 8.8.8.8 > /etc/resolv.conf`

針對陽春的 r2d2 的 rootfs ，來安裝基本功能 (透過 busybox)
```bash=
sudo cp /bin/busybox r2d2/rootfs/bin
sudo unshare --pid --fork --mount-proc --uts -R r2d2/rootfs sh
/bin/busybox --install -s # 會出現一些有關/usr/bin 的錯誤 不管他
ls -la /bin # 可以用了

```
### Namespace (PID,UTS,NETNS) + CGroup + Chroot 
以上三個功能組合起來已經很接近 container 了

```bash=
cd ~/c3po #內容是 rootfs ，裝最小版 alpine
sudo mkdir /sys/fs/cgroup/memory/demo
echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes
echo "0" | sudo tee /sys/fs/cgroup/memory/demo/memory.swappiness
sudo mount --bind -o ro  /dev  rootfs/dev # 在 rootfs/dev 上蓋上 /dev 的內容 , --bind 用於目錄蓋目錄
sudo unshare --pid --fork --mount-proc --uts -R rootfs sh
#再開另一個終端機，回到原本的系統
ps aux | grep " sh$"
echo "6524" | sudo tee /sys/fs/cgroup/memory/demo/tasks # 要看PID是多少
# 回到 chroot 後的 sh
</dev/zero head -c 200m | tail # 因為 memory 被限制了，所以會出現 killed

```
我們在網路上抓的 alp 沒有提供 dev 與 proc 裡面的東西
`ls -la /dev/zero` 為一個 c 的裝置，會一直丟0
## 20-Docker-overview
### 探索 runc
runc 的 c 為 container
runc ：一個符合 [OCI](https://opencontainers.org/) 標準的指令，用來建立 container 用
- 使用 golang 開發 ， 由 docker 公司開發
- 包含 Namespace,chroot,CGroup
- 產生 container 的高階命令，就不用自己再搞底層
- 還有一個由 google 開發的 `crun`
    - 由 c 語言開發，速度比較快
- 安裝 `sudo apk add runc ; runc -v`
    - `runc -v` 會顯示使用的版本，有：`go`, `libseccomp`(安全性),目前為 1.0.0 版本，代表很穩定 
```bash=
runc spec # 會在當前目錄產生 config.json
vim config.json # 搜尋 "readonly" : false , "hostname": "bb8"
# config.json 裡面可以設定 cap , uid gid,
sudo chown root:root -R rootfs/ #runc 預設是使用 root
sudo runc run bb8
# 執行之後就打開了 container 了
ifconfig -a # 現在還沒有網路卡可以使用
id # 雖然是 root 但並不是擁有所有權限的 (有被限制)
cat /proc/meminfo  | head -n 3 # 有 2G 的記憶體
rm /proc/meminfo #不行
# 因為 container 是共用 kernel ，因此在 container 中
# 若要動到 kernel 的部分會被限制
# 回到host
sudo runc list # 可以看使用 runc 執行的
sudo runc delete -f bb10 # 強制關掉執行中的 container
```
不管是 `runc` 還是 `crun` 都絕對要使用以下兩個東西用來建立 container (鐵律，牛頓第四定律)：
- config 檔 (以 `runc` 而言，可使用 `runc spec` 產生預設 config.json 檔)
- rootfs 目錄

config.json 有改過的值紀錄：
- `linux.resources.memory.limit`： 100000000 (byte)
- `root.readonly`: true (`boolean` 是否可以再裡面新增檔案,rw)
- `process.arg` ： "sh" (`string` 決定shell 或要執行的指令)
- `hostname` : "bb8" (`string`)

> Permission denied 與 Operation not permitted 的差異?
>  -> 一個是 檔案權限，一個是 kernal 的 cap 權限問題

### 探索 containerd

containerd: d 指的是 daemon(小惡魔LOGO)，使用 `go` 語言開發
- 在linux最後加個"&"把程式推到背景去做 
![](https://i.imgur.com/gBJsLGz.png)

> 問：為何不是由 init 管理?
containerd 會幫我們做好 rootfs 與 config.json
ctr ： 一個前端指令，透過 http 協定找到 docker hub 下載光碟
- 有三個功能：`Create,Delete,List`
    - 執行 create 就會走圖右邊的流程，下載完資料後執行 runc 指令
docker 公司的光碟中心 (docker hub)
- root_file_system ： 就是前面我們自己抓的 alpine (rootfs 目錄)
- container.json ： 前面使用 `sudo runc spec` 產生的檔案

但實際上還是透過 <font color=red bold > runc</font> 這個 OSI 的通用標準來創建 container
```bash=
sudo apk add containerd
sudo containerd &
sudo ctr image pull quay.io/cloudwalker/busybox:latest
sudo ctr image ls -q
sudo ctr run --tty quay.io/cloudwalker/busybox:latest b1 sh # 裡面 /bin 為 hardlink
#會進入 container 內，可以打指令
#下面指令是回到 host
sudo ctr container list
sudo ctr container rm b1 # 把開啟的 container 刪掉，並且沒辦法使用 --force 指令
# 因為 ctr 是比較底層的指令，就沒那麼方便
sudo ctr image remove quay.io/cloudwalker/busybox:latest # 連 image 也刪掉
sudo ctr image pull docker.io/library/ubuntu:latest
sudo ctr image pull docker.io/library/alpine:latest
```
BusyBox會偵測其被連結時的名稱，因此可以使用
image = 光碟片 ，因為是 read-only (也就是用於 overlay2 中的 lower 層)
查看版本號 `/etc/issue, /etc/os-release, /etc/lsb-release` (不要用 `uname` ， 因為在 container 中和 host 共用 kernel ，因此會查到 host 的版本號)

stack ： 為目前的趨勢，一層一層堆疊起來
- 之後的課程也會不斷用到，像是大數據分析的程式
- 利用 stack 的方式，就可以利用到很多開源的程式碼，節省自己開發的時間


### 認識 Docker & 安裝

Application Container(docker 軟體貨櫃) ：真正造成革命的技術
![](https://i.imgur.com/AgTNmlg.png)

- 圖中的 docker engine 就是 `dockerd`
    - 新增，修改，刪除 ： 使用 `docker` 命令對 `dockerd` 做維運
    - 並且提供網路連接 (bridge network (NAT)) ，上面只使用 `containerd` 建出來的都沒有網路連線
- shim ： 墊片，因為 `runc` 在執行完之後就跑了，如果沒有他的話，`containerd` 會沒辦法知道 container 的情況
    - `containerd` 會創建一個獨立的 process ： `containerd-shim` 並由其呼叫 runc 來真正創建 container.
    - 透過 shim 監控 container 的狀態回報給 `containerd` ， 如：執行效率，使用記憶體數等 
    - 才能透過 `list` 知道目前啟動了哪些 container

```bash=
sudo apk add docker # 若 runc 與 containerd 沒裝也會幫忙裝
sudo rc-update add docker boot # 設定 containerd 與 dockerd 開機時啟動
sudo addgroup bigred docker
sudo reboot # 設定完後重開
docker info
ps aux | grep -v grep | grep dockerd # 會有兩個 supervise-daemon 代表開機會啟動
ps aux | grep -v grep | grep containerd
docker run --rm -d quay.io/cloudwalker/alpine sleep 120
# -d detach 丟到旁邊(背景)執行, --rm 測試用的環境，執行完就刪掉
ps aux | grep -v grep | grep 'containerd-shim' #會出現 /usr/bin/containerd-shim-runc-v2 
# 每多 run 一個 container ，就會多一個 shim(墊片)

```

> server 也可以裝 alpine? ：是的，並且其他作業系統(ubuntu) server 與 desktop 版最大的差異就是：驅動程式不同，可能有一些高階的裝置是 Desktop 版沒有的這樣(但也可以請原廠支援)

## 21-Docker-AppContainer-Part1
這邊指的 App 是**企業用的**系統 (也是 process)
![](https://i.imgur.com/AFn0W4D.png)

### 搜尋與下載 Docker Image

開始逛街
經過某次改版， docker hub 開始會限制下載次數，所以很多人逃難到 quay.io
```bash=
docker search quay.io/busybox
docker search busybox # 就去搜尋 docker hub
# 關鍵是：只要沒有斜線的，就是由 docker 公司自己提供的
# 會有 star,offical,automatrd 等欄位可參考
# 標準格式： docker.io/libary/busybox # libary 可能可以替換成使用者，組織，類別等
docker pull quay.io/quay/busybox
#image ID ：全球唯一的代號
```

```bash=
docker run --name a1 -it quay.io/cloudwalker/alpine sh
busybox | grep httpd # 會發現找不到
wget https://busybox.net/downloads/binaries/1.31.0-defconfig-multiarch-musl/busybox-x86_64 #到官網下載
chmod +x busybox-x86_64; mv busybox-x86_64 bin/busybox
busybox | grep httpd
mkdir www; echo '<h1>Busybox HTTPd</h1>' > www/index.html
apk update; apk add elinks curl
docker ps -a
docker commit a1 raylin9981/a1 #這個是在本機先記錄，所以可以不用先登入
docker login 
docker push raylin9981/a1
docker logout
# [重要] 關閉 Docker Host 之前, 記得執行 docker logout, 然後刪除 /home/bigred/.docker 目錄
# 養成登出的好習慣
docker rm a1 
docker run -d --restart=always -p 30001:80 --name n1 quay.io/cloudwalker/nginx

```
## 21-Docker-AppContainer-Part2

### docker 網路架構
有以下三種模式：
- host ： container 直接使用 host 的網路，也就是把 network namespace 關掉~(可使用 `sudo lsns` 觀察)
- bridge ： 透過虛擬橋接器來完成連線，需要使用 NAT 偽裝功能
- None ： container 不提供網路服務，可用於內部測試。連網路介面卡都沒有所以區網也連不了

![](https://i.imgur.com/QSDmiJM.png)
圖片解釋：
- host 為 dokcer host ,就是我們現在開的虛擬機 `alp` , 用來產生 container 的
- docker0 (bridge) 為虛擬橋接器 (在電腦內部的 switch，使用 `brctl` 指令模擬出 ) ，需要在 host 的系統上開啟 ipv4.ip_forward 才能使用封包轉送功能
    - 可使用 `sysctl -a 2>/dev/null| grep 'ipv4.ip_forward '` 確認 host 的設定
- eth0 <-> veth ：也就是之前教的 ifconfig 設定出的 peer (對接)網卡


* ~~`sysctl -a 2>/dev/null| grep 'ipv4.ip_forward '`~~ , `sysctl net.ipv4.ip_forward` 就可看設定
* `route -n` 中的 `Des 0.0.0.0` 代表網際網路(defalut gateway)
* `cat /etc/resolv.conf` 每個 Linux 都是這樣做 DNS 的設定的
* 裝起來 `docker` 之後就會新增一張網卡叫做 `docker0` ，ip 為 `172.17.0.1/16` (國際標準)

> route -n 是看整張表，一次看完再做判斷，120.96.143.150 這組 IP 究竟是會走 120.96.143.0/24 還是 120.96.143.128/25 這組規則？
> 老師答 ： 每個系統不一樣，不會有人這樣寫 (神明系統)

### 新增 docker 網路
:::spoiler 新增 `mynet1` 與 `mynet2` ，都是 `bridge` 模式的 , `docker run -h`
```bash=
docker network create mynet1 # 172.18.0.1
docker network inspect --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}'  mynet1 # 顯示 mynet1 的 network id
docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet2
docker network ls # 會有三張 bridge 的網卡
sudo iptables -t nat -L -n | grep -B2 MASQUERADE #確認 iptables 的設定，應該會有三行是跟 docker 有關
docker run --rm --net mynet1 --name a1 -h ssn763  quay.io/cloudwalker/alpine  cat /etc/hosts
# 下面的是用來測試說：是否內部 DNS 有運作
docker run --net mynet2 --ip=192.168.166.3 --name  a2  -h  ddg52  -idt quay.io/cloudwalker/alpine sh # -h 指定 hostname 主機名稱
docker  network  connect  mynet1  a2 # 在 a2 上設定 mynet1 網卡(雙網卡)，可進 a2 看
docker run --net mynet1 --name a1 -h cg62  -itd quay.io/cloudwalker/alpine sh
docker  exec -it a2 sh # 開啟 a1 後，重新進到 a2 測試
ping -c1 cg62
ping -c1 cg62.mynet1

```
上面 `docker run` 指令新增的 `-h` 參數是要設定 container 的 `hostname` ，並且透過系統的 DNS 來讓 container 之間用 `hostname `做溝通
:::



docker 一開始出來預設的網路 gateway 為 `172.0.1.147` (莫名其妙)，甚至老師還有寄信過去給建議，用最前面不然用最後面，所以過了一年之後才改成了 `172.0.1.1`
- 目前 docker 內部的 `/etc/resolv.conf` , nameserver 為 `127.0.0.11` (耍帥用?)
    - 127.0.0.11 是用在有<font color=red >自建網路 `docker network create`</font> 的，如果是用原本的 `docker0` 會使用 Host 的 DNS 設定 (直接把 `/etc/resolv.conf` 複製一份進去)
    - 可以用於 docker 的網路內部互通有無，利用 hostname 來測試，但如果是預設的網路就沒辦法使用主機名稱來測試

![](https://i.imgur.com/3rSZyXs.png)

docker engine 負責網路，裡面有 DNS 的一些設定
- 先不要看黃色部分與 DNS 設定那邊
- 以在 `docker` 中執行 `curl www.google.com` (需要DNS解析) 時，是走圖中藍色箭頭 ，透過 `Engine DNS server ` 來和 external DNS 取得網域解析服務
    - 自建網路中，是使用 container host 的 external DNS `(/etc/resolv.conf)` 來做到外部的網域解析

### 認識 Dockerfile

Dockerfile 就是建置 Docker Image 的腳本。真正業界的做法 (不會有人自己在那邊改改改然後 commit)

:::spoiler `scratch` 空白光碟片版 Dockerfile ：
```bash=
FROM scratch # scratch 是一張空白光碟片
ADD main / # 把剛剛翻譯完的 go 語言(main 是檔案名稱)，丟到 / 目錄
CMD ["/main"] # container 一啟動就執行這個檔案 
```
- `docker inspect g1 | grep '"IPAddress"'` 可以檢查剛剛開的 `g1` 的 IP
- `docker build -t goweb  .` , `docker build` 要讀取 `docker file`, 後面的 . 是代表去找當前目錄的 `Dockerfile` (大小寫很重要)
    - 其實 `docker build` 可以給 `-f` 參數來指定 dockerfile 的名稱，只是默認為 `Dockerfile`
- `docker history alpine` 可以看 image 的執行指令結果
:::
:::spoiler `alpine` 版 Dockerfile:
```bash=
FROM alpine:3.14.0 # 使用 alpine 光碟，從 docker hub 下載
# 接下來 RUN 是要對 container 內部開始改裝
# 把所有東西都放進 Rootfs 目錄區內
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
    chmod +x busybox-x86_64 && mv busybox-x86_64 bin/busybox1.28 && \
    mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

CMD ["/bin/bash"]
CMD ["busybox1.28","httpd","-f","-h" , "/opt/www"]
```
```bash=
docker build --no-cache  -t alpine.base base/ # 不使用 cache 來建立檔案，可確保檔案內容是重新再被建立一次
docker run --rm -it alpine.base
# 進到 container
busybox1.28 httpd -h /opt/www # 執行一下 busybox1.28 測試
curl http://localhost # 有東西代表ok
exit
# 準備設定要拿來上傳用的 container
docker run --rm --name b1 -d -p 80:80 alpine.base busybox1.28 httpd -f -h /opt/www # 後面的 -f 是要讓 busybox 在前景執行，不然 container 的 PID1 在背景執行會關掉
#Container 中的第一個啟動 process 必須前景執行, 這樣 Container 才能保持在執行狀態
docker commit b1 raylin9981/alpine.base # commit 指令：針對 container 產生 image
# 所以這是用 b1 來產生 raylin9981/alpine.base
docker login
docker push raylin9981/alpine.base
```
:::
[docker build cache 解釋：最佳化 Dockerfile - 活用 cache](https://ithelp.ithome.com.tw/articles/10246065)
- FROM：從哪張光碟開始，有可能上張光碟也用到其他光碟，就會一層一層堆上去
- ADD ：從 hosts 把檔案丟進 container
- CMD ：用於執行 container 時的默認指令 (也就是`docker run ` 最後若沒給指令時執行的指令 )一個 Dockerfile 中只能有一個，若有多個會使用最後一個。
    - 有兩種寫法，其中一種是使用 `[]` 來分隔命令. e.g. `["/bin/bash","echo","123"]`
- ENTRYPOINT ： container 建立時一定會做的指令，不會被 `docker run` 最後面的參數取代
    - ["/usr/sbin/sshd"] ：確保 Container 一起來就執行 `sshd`
    - 配上 CMD ["-D"] ， 代表是丟給 `sshd` 做參數。`-D` 代表前景執行

==標記文字測試==
`apk add --no-cache` 目的是希望把 container 的容量降至最小
ssh 連接至 container
在大型系統上：時間的校準是很重要的，如果沒有做好那那個系統一定很容易掛掉
:::spoiler sshd alpine 建立
```bash=
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e 'Welcome to ALP SSHd 6000\n' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"] ' > sshd/Dockerfile 

```
只要叫做電腦，開機後一定有一個程式在前景執行
- 雖然是這樣，但 Linux 的 init 卻在背景執行，而不是成為維持電腦的前景 -> init 會負責不斷產生 `getty` 當作前景程式



:::
:::spoiler sshd
```bash=
# 利用 echo 時，可以使用 $讓裡面的單引號跳脫掉
# 不過這樣問題多多 ， 建議還是直接寫檔案
echo $'FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e \'Welcome to ALP SSHd 6000\n\' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo \'%wheel ALL=(ALL) NOPASSWD: ALL\' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"] ' > sshd/Dockerfile
```
Dockerfile：
```bash=
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e 'Welcome to ALP SSHd 6000\n' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```
:::

:::spoiler 再設計一個更安全的image: `alpine.sshd`
更改管理者帳號(bigred)的密碼，把 root 關掉，並且留一個無 `sudo` 權限的 rbean
```bash=
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e 'Welcome to ALP SSHd 6000\n' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred333\nbigred333\n" | passwd bigred &>/dev/null && \
    adduser -s /bin/bash -h /home/rbean -D rbean && \
    echo -e "rbean\nrbean" | passwd rbean &> /dev/null && \
    passwd -dl root && \
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```

:::


> `sudo apk add figlet` 居然會把 /sbin/reboot 重新抓進來?
> -> 不對，是只要用 apk add 裝指令的話就會重新弄回來 ，並且是從 busybox 連結過去的
> 為何 `busybox` 可以使用一堆連結的方式就完成一堆指令?，
推測是使用 $0 變數來完成指令的判斷，因為若是用連結的方式來執行的話 $0 會是連結檔的名字
就可以利用這個特性來寫條件式


`docker history --no-trunc alpine.sshd` 可以看到所有 docker file 的內容
- 要是密碼寫在裡面就出大事囉，下面會教怎麼清乾淨


`docker save alpine.sshd > alpine.sshd.tar` 把 image 保存到一個打包檔 (沒有做壓縮)
`docker load < alpine.sshd.tar` 利用 `docker load` 重建 image
很多企業的電腦都不給上網，所以可以利用這樣的方式，使用隨身碟把 image 用到公司電腦來用
### Docker Container 備份與還原
因為我們上面的 `Dockerfile` 裡面有寫到密碼，要是直接拿來用的話只要使用 `docker history` 就可以看到內容
可以進行下面的動作來把 history 清乾淨：

:::spoiler 備份與還原，消磁動作 (把 Dockerfile 內的敏感資訊砍光光)
是針對 `container` 做備份，還原是變成 `image`
透過 `docker import ` 與 `docker export`
```bash=
docker run --name s2 -h s2 -d alpine.sshd
docker exec -it s2 bash
sudo useradd -m -s /bin/bash rbean
echo "rbean:rbean" | sudo chpasswd
exit
# 離開之後 最好要關掉再備份
docker stop s2 # 因為可能會有 openssh 的暫存檔，所以關機確保不要有一些暫存檔被備份
docker export s2 > s2.tar 
docker rm s2 && docker rmi alpine.sshd
cat s2.tar | docker import - alpine.sshd # 產生的是 image 檔而不是 container
# 進行消磁作業，但我們的 ENTRYPOINT 也掛了，這樣就不會幫忙產生 sshd
docker run -d --name t1 alpine.sshd /usr/sbin/sshd -D
docker commit t1 alpine.sshd # 使用 commit 指令再透過 container 來更新 image
docker history alpine.sshd # 可看到 ENTRYPOINT
```
`sudo apk add docker-bash-completion`
:::

### Docker Data Volume
![](https://i.imgur.com/ib7IeUh.png)

- Linux 是最適合的 OS 
- Container 內**一定**有一個在前景執行的 app , 並且有獨立的相依檔
- 右邊是代表使用光碟片 (image) 來製作出2個 container (使用 overlay2 檔案系統)
    - Layer 1 ： lower , 也就是放 `rootfs` 的內容 (ro)
    - Layer 2 ： upper , 但畫得不好? 應該要切成兩邊才對，是兩個目錄分別提供給 container 使用的 (rw)
    - Layer 3 ： merge , 就是下面兩層堆疊後掛載的地方，實際運作的目錄
    - overlay2 有 copy-up 與 whiteout(立可白) 功能，但是會造成速度變慢(一直要搜尋是不是在 lower 內有檔案的)
        - 這樣就很不適合用在 **資料庫,hadoop,訊息佇列?** 等系統，因為會有大量的檔案讀寫導致速度變慢

data volume 會存在 `/var/lib/docker/volumes/DOCKER亂碼` 中，是使用原本 Host 的主機的檔案系統 (EXT4)
- 就可以用來配置上面提到的如資料庫系統做使用

#### container mariadb 測試
```bash=
docker pull quay.io/cloudwalker/mariadb
docker history quay.io/cloudwalker/mariadb # 可以看一下 裡面有一行 Volume 的部分
docker run -d -p 3301:3306 -e MYSQL_ROOT_PASSWORD=admin --name m5 quay.io/cloudwalker/mariadb # 建立
docker inspect --format='{{index .Mounts 0}}' m5 | cut -d' ' -f3 #會顯示一個在 host 的目錄位置
# container m5 中的 /var/lib/mysql 對應到 host 的目錄位置
# 這個 volume 就算 container 刪掉也不會刪掉，並且在重開之後也會在新增一個來占空間用
docker rm -f m5
docker volume ls -f dangling=true # 會顯示目前沒有被使用的 volume 
docker volume prune # 這個指令可以把隱藏的 volume 抓出來砍掉
```
以上都不會用到，在外面請使用 <font color='red'>Host Data Volume</font>

### Host Data Volume 
![](https://i.imgur.com/vfBFpKr.png)

- 可以自己取目錄名稱，目錄結構自己決定，就不用再藏在深深的目錄區中
- 在右邊的兩個 container 中 ，就會共用 host 的這個目錄

#### Host Data Volume 測試
```bash=
# derby 是java 很初始的資料庫?
cd ~/wulin; mkdir db
docker run --name w1 -d -p 8888:8888 -v ~/wulin/db:/derby/db  quay.io/cloudwalker/alpine.derby
curl http://localhost:8888/db/cars/create;echo ""
curl 'http://localhost:8888/db/cars/insert?id=123&name=star&price=123' ;echo ""
curl 'http://localhost:8888/db/cars/insert?id=234&name=sun&price=123';echo ""
curl http://localhost:8888/db/cars/list;echo ""
docker rm -f w1 #刪掉重建一個，會發現資料還在
docker run --name w1 -d -p 9999:8888 -v ~/wulin/db:/derby/db  quay.io/cloudwalker/alpine.derby
curl http://localhost:9999/db/cars/list;echo ""

```
:::spoiler 練習魷魚遊戲 設計兩道關卡
目錄區內容： netscript.sh 用來偵測 ping 跟 curl 的， password.sh 為 EntryPoint , autoclean 只是測試用，方便我把 build 起來的 image 快速清空
`ls`
Dockerfile  autoclean  netscript.sh  password.sh
Dockerfile:

```bash=
FROM alpine
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl bash && \
    rm /bin/ping && \
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
    echo 'echo Hi~~' | sudo tee  -a /etc/profile
ADD password.sh /
ADD netscript.sh /
RUN ln -s /netscript.sh /usr/local/bin/curl && ln -s /netscript.sh /bin/ping

ENTRYPOINT ["/password.sh"]
```
netscript.sh
```bash=
#!/bin/bash
echo you have 20sec to reply this problem
echo 123.146.231.170/27 network id =?
read -p 'your ans :  ' -t 20 REPLY
if [ "$REPLY" == "123.146.231.160" ];then
        [ "$0" == "/usr/local/bin/curl" ] && /usr/bin/curl $@
        [ "$0" == "/bin/ping" ] && /bin/busybox ping $@
else
        echo you network bye bye
        rm -f $(which nc)  $(which ifconfig)  $(which route) $(which ip) $(which ping) $(which curl)
        echo but you can use busybox...
fi
```
password.sh
```bash=
#!/bin/bash
[ -f 'first_pass' ] && echo HI,smart people , && /bin/bash
[ -f 'no_pass' ] && echo can not enter this computer && exit
if [ -f 'first_pass' ];then
        exit
fi
echo 123+456+789 = ?
read -p  'your ans: ' -t 5 REPLY
echo your ans: $REPLY
if [ "$REPLY" == "1368" ];then
        touch first_pass
        /bin/bash
fi
if [ ! -f 'first_pass' ];then
touch no_pass && echo 'YOU DIED!!!' || exit
fi
```
以上檔案都建立好後，可參考 build 指令
`docker build --no-cache  -t 1012alpine .`
`docker run -it --name abcd 1012alpine` 進入後輸入答案
`ping 8.8.8.8` 輸入 network 答案
`curl www.google.com` 輸入 network 答案後即可獲得網站資訊
```bash=
#助教範例
#!/bin/sh
echo '5 second'
read -t 5 -p '1632457 byte = ?M ' ans
if [ "$ans" = "1.6" ]
then
  rm `which $0`
  sh
else
  echo "gg"
  rm `which busybox`
fi
```
:::

## 300-Kubernetes-Overview
![](https://i.imgur.com/TFUNyZ0.png)


`Kubernetes` ：船舵
所管理的船叫做：`moby` ，沒有對這個多做解釋
**重點就是當實體機器太多的時候難以管理，要透過一套系統來進行管理**
叢集：同一個系統一次開好幾份
- 原本這種技術台灣沒人會，都要去原廠上課。但現在 k8s 很輕鬆直接幫忙管理

> 關貿公司：台灣的軟體龍頭?但疫苗預約系統被罵爆

以下是 k8s 的幾個功能：
- `Replication of components` ： 叢集，同一個服務開好幾個機器來提供服務，避免單點損毀的情況，提高可用性
- `Auto-scaling` ：橫向擴充，可以依照暴漲的流量擴充機器
- `Load balancing` ： 跟上面的功能一起使用，分散流量，(負載平衡)
- `Rolling updates` ： 給程式設計師用，原始碼掃描 (檢查 code 裡面是否有禮貌、有沒有死邏輯等) ，壓力測試後才能上到 production 環境
    - 可達到不停機，持續提供服務，可參考[k8s的文件](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
    - 是屬於 CI/CD 的 CD 部分，持續部屬 (吧)
- `Logging across components` ： 需要知道每台 pod 內部運作的狀態等，k8s 會幫忙整合每一個 pod 的 log，並且提供漂亮的儀表板來看。 [cncf.io，Cloud Native 的官網](https://cncf.io) 可看畢業專案，可以看一下有什麼能用
    - `prometheus` (普羅米修斯) ：管理系統 , `grafana` (瓜法納)：提供儀表板
- `Monitoring and health checking` ： 跟 log 是一起的，透過 log 來監控 pod 的狀態
- `Service discovery` ： DNS Server ，因為 ~~container~~ (其實都是講 pod ) 會不斷開關，IP 實在記不住。
    - 因此就使用名稱來溝通
- `Authentication` ： 使用憑證 (pub + pri key) 來做各服務間的認證

k8s 就是透過上面的服務，讓企業的應用系統跑得很穩定，並且是**自動化**的管理
**並且 k8s 為雲端作業系統，很多應用程式要另外裝 (就好像 win10 需要再額外裝 excel 的意思)**
world computer ：世界電腦，大概就像是雲端平台的意思(azure)
autopilot ： google 在推的，加入 AI 讓 k8s 越來越接近全自動化管理
三家雲端 k8s ： Google GKE , Amazon EKS , Azure AKS
若要本機不上雲 ： Redhat OpenShift (台灣最多企業採用) , VMware Tanzu , Rancher
每一家都有一些獨特的改裝，所以聊起天來專有名詞很多

### 環境介紹

![](https://i.imgur.com/o8xlyxi.png)
接下來我們要裝3台電腦
- 一台當 `Master` , 兩台當 `Worker Node`
- 使用的 OS 都是 `Alpine`

`pod` 是 k8s 的運作最小單元，裡面可以跑多個 container (至少一個)
Master 裡面一定會有4個 `pod` 
- `API Server`：此 `pod` 中含1個 container，專門用來接收命令(從 WEB UI 或 CLI(`kubectl`) 來的)
    - 提供 k8s 資源的 CRUD ，如授權、認證、存取控制與 API 註冊等機制 
- `Scheduler`：接受從 API Server 來的創建 `pod` 指令，決定哪一個 Node 比較適合開新的 `pod` 並且進行排程 (決定 `pod` 要在哪開)
    - 呼叫對應節點上的容器引擎 (e.g. docker,cri-o) 執行，而排程受到 QoS 要求、軟硬體約束、親和性(Affinity)等等規範影響。
- `Controller-Manager`：照顧 `pod` ，若 node 掛掉會把 `pod` 移去其他點。若 `pod` 工作負荷太大，也會再開啟新的 `pod` 來分散流量 (auto-scaling + Load-balancing)
    - 透過核心控制循環(Core Control Loop)監聽 Kubernetes API 的資源來維護叢集的狀態
    - 會使用不同的控制器來管理 (e.g. Namespace Controller)
- `etcd`：最重要的區塊，存放整個 k8s 系統運作的重要資訊 (資料庫)
    - 用來保存叢集所有狀態的 Key/Value 儲存系統，所有 Kubernetes 元件會透過 `API Server` 來跟 `etcd` 進行溝通來保存或取得資源狀態

Worker node :
- `kubelet`：呼叫 `docker` 產生 `container` ，接收由 `Scheduler` 傳送來的指令
    - 會負責存放一些 `pod` 的 metadata -> 暫存的概念，印象中會設定一個期間去跟 `etcd` 更新資料
- `kube-proxy`：提供兩 node 間的 `pod` 可以互通網路

### Open Interfaces in Kubernetes (CRI,CNI,CSI)
![](https://i.imgur.com/PhDXqP0.png)
k8s 制定三個標準，讓大家都可以嘗試來寫寫看。會不斷的有新公司來挑戰是否能做得更好
- **CRI (Container Runtime Interface)** : 產生 container 的標準，提供運算資源，如：`CRI-O` , ~~`docker`~~ (可能再兩三年要消失了) 
- **CNI (Container Network Interface)** : 提供網路資源的標準，可以用**硬體**或**軟體**，買不起硬體設備就使用軟體來做到
    - 三大雲端供應商都是使用**硬體CNI** ，因為速度快。
- **CSI (Container Storage Interface)** : 提供儲存資料的標準，也是有分**軟體**跟**硬體**。硬體：DELL EMC (一個百萬起跳)
- 以後跟別人聊 k8s 的時候，直接問他上面這三個再開始聊內部的細節

### kubernetes CRI 的架構演進圖

![](https://i.imgur.com/RNDl23G.png)
1. 最上面是第一代的流程，速度慢，效率差。透過 docker engine 來做到內部網路 (`127.0.0.11`,上面 `docker network`部分有用過)
2. 第二代：把 docker 給砍掉了，並且 k8s 自己寫了網路架構，不依賴 docker engine
3. 第三代：containerd 討好 k8s ，直接提供 plugin 給 k8s 
4. 目前：使用 `CRI-O` 這個專案，特別為 k8s 量身訂做，直接翻譯 `kubelet` 的話。
    - 而且直接對 `runc` 溝通，不用再經過好多流程才產生 container
- **結論：k8s 不需要 docker 就可以開啟 container 了**
### CRI (Container Runtime Interface)

Pod能用的硬體資源上限，是根據 node 的硬體規格(CPU核心數、記憶體容量)

### CNI (Container Network Interface)
![](https://i.imgur.com/2PGcDmI.png)

啟用後，可以讓不同 node 間的 pod 連線
但 pod 間是不能使用 `localhost` 連線的
- pod 裡面的 container 可以使用 `localhost` 來連線

目前環境中使用的 CNI 為 ： `Flannel`

### CSI (Container Storage Interface)
![](https://i.imgur.com/CKT3NvF.png)
使用 NFS 來做資料儲存
左邊的藍色部分是一個 Linux 作業系統

- QNAP 威連通 ： 有比較低階的儲存裝置，裡面都是跑 Linux 系統 [一台黑黑的盒子](https://24h.pchome.com.tw/prod/DRAB7M-A900ATAJ7?fq=/S/DRAB82)
    - 裡面有用 busybox 模擬出的網頁提供操作
    - 其實賣的只有插槽，要自己去擴充 SATA 硬碟
- DELL EMC ： 高階伺服器用 ，一個要破百萬的

###  (Scheduling) Kubernetes Pods 
![](https://i.imgur.com/sSwxJlj.png)
可以在 Node 上貼一些標籤
- 最主要是 `env` 的部分，分成了 `prod` 與 `test`
    - `test` 就用來測試程式碼用，`Scheduling` 就可以依照這些標籤來部屬 pod
- 也多貼了一個 `zone` ， 可以分成台北、台中做到兩邊的分散風險

### (High Availability)  Kubernetes Pods 
就是同一個服務分在兩個 Node 存放，提高可用性 (HA)
- 同一套會計系統啟動兩套，如果一套掛了還是可以使用

### (Raft 演算法) Kubernetes Master 容錯 
![](https://i.imgur.com/Ro4lBhU.png)
> Raft QQ

:::spoiler Raft 小筆記
Raft ： 為了避免單點損壞，所創的叢集式架構演算法
當一台機器有 10% 會當機，則代表一年內有 36.5 天會不能使用服務 (慘)

- 所以若準備三台，只要一台沒壞就可以提供服務，因此不能使用的機率為 10% * 10% * 10% (0.001) 大約一年中剩 5 分鐘不能使用

Raft 主要依賴以下兩個機制做運行：

- Leader Election(majority) ：選出一個 leader ，由他來發號施令
- Log Replication ：多台主機時，環境要保持一致不容易，所以就透過 leader 來統一管理
    - 所有的變動都是交由 majority 在發送給其他 follower 來統一更新

[Raft 中文部落格](http://hushi55.github.io/2016/01/22/Raft)
[原文 Raft github文件](https://raft.github.io/)

> 會需要三台才夠的原因：因為在選 majority 時，是透過選舉的方式，當有衝突 (有兩台在選舉時) ，就必須要有奇數的票型才能正確選出 

:::

使用 `Raft` 演算法 ，分成了 `Majority` 跟 `Failure Tolerance`
都是以 `Cluster Size` (master node 的數量 ) 來做計算，當作 N 使用

- Majority = (N/2)+1 ： 決定 k8s 能開放的功能數量?
    - 企業等級的要至少 3 個 master 才夠，做到 2 台 majority ， 1 台 tolerance
    - 雖然其中一台是備份用，不過還是會平均分散流量到 3 台做事
- Failure Tolerance = (N-1)/2 ：重點是當有 2 台的時候還是 0 ，因為如果有一台掛掉，則 k8s 直接鎖死不能動
    - 不過舊有的 Pod 依然會運作，只是就沒辦法開新的
- 如果真的只能搞三台怎麼辦？
    - ~~1 master 2 worker ： 嘿嘿，一台 master 掛了整個系統掛囉，也救不回來，請你下台~~
    - 2 master 1 worker ： 結果一樣 一台 master 掛掉鎖死，還是不能用
    - 3 master 兼做 worker ： 正解 ，既可以做到備份，也可以正常運作
        - 當然，這是最糟情況。最好還是 3 master 加上盡可能多的 worker


## 301-Kubernetes-cluster

### 安裝 CRI-O 及 CRI-Tools

```bash=
# m1
sudo apk update; sudo apk add cri-o cri-tools --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted #指定 testing 的 repository
sudo rc-update add crio default
# crictl images ppt 沒有，可以試試看會發現錯誤
# 他預設還是會先去找 docker ，不採用自己家的服務 
sudo vim /etc/crictl.yaml 
runtime-endpoint: unix:///var/run/crio/crio.sock
image-endpoint: unix:///var/run/crio/crio.sock
timeout: 2
# crictl 預設沒有本機儲存的 image 方法
# 因此要搭配 podman 做使用 ， 使用 crictl pull 下載的 image 也同步會更新到 podman ，反之亦然
# 接下來也要在 w1 與 w2 都安裝 上面的指令
# 然後再來安裝 k8s 的命令套件
sudo apk add kubeadm kubelet kubectl --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted
sudo mv /etc/cni/net.d/87-podman-bridge.conflist ~ # 這個是 podman 使用的網路設定
#不適用於 k8s 所以把它移開 一定要做不然後面會出狀況!!
#以上兩個指令一樣 w1 m1 m2 都要做
#做到這邊可以備份一下 k8s1
#m1

```

### 建立 K8S Master
> podman 有使用 sudo 與 沒有使用有差異的!!
> 使用 sudo podman 創建的為 rootful container
> 不使用 sudo (only podman) 創建的為 rootless container , 不需要用到 CNI

```bash=
#m1
#crictl -v 確認一下 cri-o 有安裝 , kubeadm version 也有裝
# alias docker='sudo podman' 在 /etc/profile 新增，把 podman 取代 docker
# podman 可以取代 docker 95% 的功能
########## step 1 ##############
sudo kubeadm init --service-cidr 10.98.0.0/24 --pod-network-cidr 10.233.0.0/16 --apiserver-advertise-address $IP
# --service-cidr 是給服務用的IP , --pod-network-cidr 是給 pod 用的 , apiserver 架在 m1 的電腦 $IP 代表目前的 IP
# 最多可以提供 254 服務(/24)
sudo rc-update add kubelet default
#將 bigred 設成 K8S 管理者
mkdir -p $HOME/.kube; sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config; sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes #檢查一下
#設定 K8S Master 可以執行 Pod
kubectl taint node m1 node-role.kubernetes.io/master:NoSchedule- # node/m1 untainted
sudo reboot
############ step 2 ###############
kubectl get pods -n kube-system # 可使用 ks 查看
# 可看到 apiserver,scheduler,etcd 等，但 coredns 因為 CNI 還沒裝，所以網路目前還是 pending
# 來安裝 CNI ：Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml # yml 是 yaml 的簡寫 是一樣的東西
# kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
# curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml -o kube-flannel.yml
# 目前環境中的 VIP 是設定 10.233 ， 不過預設是 10.244 -> 改 yml 檔變成10.233
# rancher/mirrored-flannelcni-flannel-cni-plugin:v1.2 -> quay.io/ict39/rancher.flannel
 sed -ie 's/244/233/' kube-flannel.yml
 sed -ie 's/rancher\/mirrored-flannelcni-flannel-cni-plugin:v1.2/quay.io\/ict39\/rancher.flannel:v1.2/' kube-flannel.yml
# kubectl apply -f kube-flannel.yml
# cat /run/flannel/subnet.env
#裝完之後 所有都變成 running 了
kubectl get pod -n kube-system
# 確定所有的 pod 都 running 再重開
sudo reboot
#重開後再打一次 kg 確認所有 pod 都在 running
```

> `kubectl version` 會吐出兩個版本，一個是 client (`kubectl`) , 另一個是 server (安裝的叢集版本)
> 這兩個版本可以不同，但最後好是在兩個 `minor` 之內，並且不要使用新的功能到舊的叢集上。

### 開始建置 Kubernetes Worker
其實剛剛上面做過了，就是安裝 cri-o 那邊的幾個步驟 + 安裝命令套件等
```bash=
# 在 m1 執行
## 第一種方式 -> 分別連進兩台去做
echo " sudo `kubeadm token create --print-join-command 2>/dev/null`"
# 會輸出一個很長的命令，是等等要在 w1,w2 執行的
# ssh w1 , ssh w2
sudo kubeadm join 192.168.234.4:6443 --token cx1pcd.rbp5s7yy8i7tqhgp --discovery-token-ca-cert-hash sha256:9c465f2c2121623c628d9ded82377855dfe6b2977f56cceebe454e3390761312
sudo rc-update add kubelet default # 記得把 kubelet 開起來
sudo reboot # 做完之後 m1 輸入 kg nodes 就會多出幾台 ready的
### 在 m1 設定 w1,w2 label
kubectl label node w1 node-role.kubernetes.io/worker=worker
kubectl label node w2 node-role.kubernetes.io/worker=worker
## 第二種方式 -> 把上面連線的三行指令簡化成一行 ###################
a="sudo `kubeadm token create --print-join-command`"
ssh w1 $a ;ssh w1 sudo rc-update add kubelet default ;ssh w1 sudo reboot
ssh w2 $a ;ssh w2 sudo rc-update add kubelet default ;ssh w2 sudo reboot
kubectl label node w1 node-role.kubernetes.io/worker=worker
kubectl label node w2 node-role.kubernetes.io/worker=worker
```
### 建置 Kubernetes 叢集服務
可使用 `kubectl run` 跟 `kubectl create` 
- `kubectl run` ： (命令式 Imperative) 測試用產生 k8s 資源物件，可建立 `pod` , `Controller` , `service` 等，**直接使用 CLI**
    - 作概念性驗證 (Proof of Concept：POC) ，也就是測試用
- ~~`kubectl create`~~ `kubectl apply`：(聲明式 Declarative)  使用一些寫好的 k8s object yaml file ，用於建立企業複雜的環境
    - 注：`kubectl create` 應該也是命令式才對，只是把 `kubectl {run,expose}` 等整合在一起而已

![a](https://i.imgur.com/hDSIuDA.png)

- 六角形代表的是 Node ，一台實體電腦
- 圈圈為 pod -> 每一個圈圈都是一個 service, 可以放公司的 ERP 等 ，把很龐大的系統整合起來
    - 並且 k8s 可以同時產生好幾個 pod ，提高可用性 (HA) 
    - 若是遇到流量暴增 ， 也可以很輕易地做到 scaling , 處理多餘的流量 Load-balancing 等
- 正方形為 container ,containerized app

pod 內部共用 `Storage`, `Linux namespaces`, `cgroups`, `IP addresses` 等等資源，並且是不長壽的 (很常重開)

```bash=
# m1
kubectl run a1 --image=quay.io/cloudwalker/alpine.derby --port=8888 
# 從 1.18 這個版本開始, 上面命令只會產生 pod, 不再會產生 deployment object 及 replicaset controller.
k get pods -o wide
kubectl port-forward --address $IP pod/a1 8888:8888
# --address 只能給 m1 的 IP
# 在 windows 的終端機
curl http://192.168.234.4:8888 # 可以連線到
```

`kubectl delete pod a1 --grace-period=0 --force` 強制刪除 pod 指令

> master 上節點的服務開啟，好像是新版才使用 pod 來做? v1.5.2 2017-07-03T15:31:10Z
> alpine : v1.22.2 2021-09-16T19:24:26Z

除錯：

```bash=
kubectl describe pods kube-flannel-ds-f9dj8 -n kube-system
kubectl logs  kube-flannel-ds-f9dj8 -n kube-system
# 老師這邊遇到了 flannel 的問題 ，重開 pod 通常沒用
kubectl delete -f kube-flannel.yml
kubectl delete pod kube-flannel-ds-f9dj8 --grace-period=0 --force -n kube-system
```

跟 `api-server` 這些有關的 pod ，都是放在 `kube-system` 這個 namespace 中
如果沒有指定 namespace ，就是去找 `default` 的 ， 可用 `k get namespaces` 看

`minik8s` , `microk8s` ：單機版的 k8s ，CNI 使用 podman 的 (沒辦法跨 node 傳送網路流量)

> docker hub 的下載限制：6小時只能下載100次

虛擬 IP (VIP) 不綁定介面，透過 `iptables` 來導向
-> 在每一個 node 中都可以使用這個 VIP 來存取 pod , 因為各 node 都有設定好 `iptables`
`docker tag ` 可以改 image 的名稱

`kubectl run a1 --image=quay.io/cloudwalker/alpine -- sleep 15` `--` 的空白沒有打錯，代表對 pod 要執行的指令

- `--` 前後兩邊都要，從去年年底(2020)開始的規範 
- 企業通常不會用到這麼新版 (v1.22.2,2021-09-16)

pod 正常結束，k8s 會產生一個新的、一模一樣的 pod 並啟動起來

- k8s 重開 pod 是從 image 的內容再重開一個 pod ，因此手動更新的部分重開之後就會消失了
- 如果要保存內容，就要把新增的東西放在 host data volume 中 ，重開 pod 時就會在了


```bash=
kubectl run a1 --image=quay.io/cloudwalker/alpine -- sleep infinite
kubectl get pods a1 -o wide # 看是開在哪個 node
ssh w1 sudo poweroff # w1 要替換成上面顯示的 node
# 把 node 關掉之後， pod 不會變成不能用
# 接下來再把 w1 機器打開，會發現 restart 多了 1 ，並且 IP 也跟原本不一樣
###
kubectl run a1 --image=quay.io/ict39/alpine.derby --port=8888
kubectl run a1 --restart='Never'  --image=quay.io/cloudwalker/alpine -- sleep 15 # 設定不要重開，15秒過了就 completed

```

- 重開 pod 時會給一個再重新指派一個虛擬 IP 給 pod 用

老師抽考基礎：
1.我們自己停止，K8S會產生新的並啟動
2.我們把pod存在的那台主機 w1 關機重開，w1內的所有 pod 會浴火重生
3.我們做永不重啟，加參數`--restart='Never'`

### 佈署 Deployment Object Application (HR,MRP,etc...)
準備把 k8s 應用在真實的企業的應用系統中
Deployment Object 就是要把這些系統部屬上去的工具
![](https://i.imgur.com/q5FOZRp.png)

旁邊三個是 worker node (w1,w2) , 中間的是 m1
在 m1 裡面的 D 代表 Deployment ，存在

- **Deployment Object** ： 是一個資訊區塊，存放在 `etcd` ，會產生 `Replicaset Controller`
    - **Replicaset Controller** ：掛在 `controller-manager` 底下 ，負責照護 `pod` ， 可做到不同 Node 的 pod 重開
        - `Replicaset Controller` 是一種 `Controller object`
        - 當 `pod` 數量有減少時，會立刻補全 (浴火重生)
        - 負責決定要產生幾個 `pod` ，去通知 `scheduler`
        - 都是負責同樣的應用系統，做負載平衡與提高可用性

```yaml=
# depobj.yml
# 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depobj
spec:
  replicas: 2
  selector: # selector 決定現在要照顧的 pod 是誰
    matchLabels:
      app: deppod # 選到 app: deppod label 的 pod 並且依照他的規格產生 2 個 pod 出來
  template: # template pod 的宣告
    metadata:
      labels:
        app: deppod # 對應到上面 selector 的 label
    spec:
      containers:
      - name: myderby
        image: quay.io/ict39/alpine.derby
        imagePullPolicy: Never # image 不上網抓 , 可改成 IfNotPresent 若在本機沒有就上網抓
        ports:
        - name: http
          containerPort: 8888
```

- `spec.template` 底下的內容就是要決定 pod 的內容
    - `spec.replicas` ：用來決定要開幾個 `pod`
    - `spec.template.metadata.labels` 決定要為 pod 貼上什麼標籤
    - `spec.template.spec.containers` 只有一個 image 代表開啟的 pod 裡面只會開一個 container

`kubectl apply -f depobj.yml`   
`kg all`
`ssh w2 sudo podman pull quay.io/ict39/alpine.derby` 好像是 `crictl ` 比較正確
`kubectl delete -f depobj.yml`
利用以上 yaml 創的 pod ，雖然會給虛擬 IP ，但其他人是連不到的
- 能使用這個虛擬 IP 的(VIP) 只有三個：叢集內的 `Node` , `pod` 以及 `service`

### 佈署 Kubernetes Service

![](https://i.imgur.com/jtvdyO4.png)

- 放在 master 的 A 跟 B 就是上面的 **Deployment Object** ， 上面有貼標籤 e.g. APP=A
- 靠標籤 (label) 來決定這個 object 要管哪些 pod
    - 所有相同標籤 (透過 label selector 來篩選 ) 的 pod 會被包成一個 service , 並且提供一個固定對外的 IP 提供服務 
    - 同一個服務開多個 pod , 且放在不同的 node 上即可做到 負載平衡 與 高可用性

```yaml=
#mysrv.yml
kind: Service
apiVersion: v1
metadata:
  name: mysrv
spec:
  selector:
    app: deppod
  ports:
  - protocol: TCP
    port: 4000
    targetPort: 8888
```
> `kubectl create `  與 `kubectl apply` 的差異：
> `create` 是定稿，沒辦法再修改
> `apply` 的意思是之後還可以在修改 yaml 檔案，更新 object 


`kubectl create -f mysrv.yml`
`kg service -o wide` 去看 `mysrv` 的 VIP 下面的 `curl` 是使用這裡看到的 VIP
`curl 10.98.0.222:4000/hostname` 多次執行，可以發現會到不同台機器
:::spoiler 使用 `kubectl` 可以開一個 pod ，在裡面測試 DNS
在這個 container 
`kubectl run a1 -it --rm  --image=quay.io/cloudwalker/alpine -- sh`
`apk add curl && curl mysev:4000/hostname`
```bash=
cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local localdomain
nameserver 10.98.0.10
```
:::

## 303-Kubernetes-Pod
![](https://i.imgur.com/cnYAEAH.png)

Pod 是 k8s 內的最小單元，可以很小 (裡面只有一個 container )，或很大 (包含 volume 與多個 container)

> 當引入新系統：好老闆是問你：有什麼好處？爛老闆是問你：能不能賺錢

![](https://i.imgur.com/FVZy32H.png)

- host 是我們的 worker node ，裡面會裝 `kubelet` 與 `kube-proxy` 
- 三個 `container` 是共用 `network namespace` , 所以可以共用網路卡 `veth0`
    - `container 1` 可想成是 `nginx` ，會占用 `veth0` 的 80 port (http)
    - `container 2` 可想成是 `mariadb` ， 會占用 `veth0` 的 3306 port
    - 這兩個 port 都是開在 `veth0` 這個網路裝置
- k8s 產生 `pod` 時，一定會產生 `pause` 這個 `container` , 並且是隱藏的
    - 裡面執行的命令為 ：`sleep infinite`
        - 因為 `sleep` 沒有使用到其他 `process` ，<font color=red> 所以安全等級非常高!!</font>
    - `pause` 這個容器會負責產生 `各類 namespace` 來給 `pod` 內的其他 `container` 共用
    - `pause` 是 k8s 的核心，若是把這個容器停掉的話，整個 k8s 就被癱瘓了

```yaml=
# sharepid.yml
apiVersion: v1
kind: Pod
metadata: # metadata 指的是 etcd
  name: sharepid
spec: # container 的細節
  shareProcessNamespace: true #container 內部會共用 pid namespace (pause 先創好的)
  hostname: xyz  #指定 UTS namespace 共用
  containers: # 有兩個 image ， 因此在 pod 內會有不同的 contatiner
  - name: derby # 前面的 - 代表 list , 
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Always
  - name: shell
    image: quay.io/cloudwalker/alpine
    imagePullPolicy: IfNotPresent
    stdin: true
    tty: true  #這兩個等同於 -it
```

- kind ： 代表要產生的內容，這邊是產生 `pod`
- metadata ： 要存放到 etcd 內的資料
- spec ： 代表是 container 內的資料
    - shareProcessNamespace： pod 內的 container 共用 namespace ， 預設為 false ，設定為 false 後就會看不到 `pause` 了
    - hostname ： 指定 container hostname name
    - containers：是一個 list ，在這個例子中會在 pod 中產生兩個 container

`k create -f sharepid.yml`
` kubectl get pod/sharepid -o jsonpath='{.spec.containers[*].name}';echo ""` 會看到兩個 container , pause 被隱藏起來了
`kubectl exec sharepid  -c shell -- hostname; kubectl exec sharepid  -c derby -- hostname` , `kubectl exec sharepid  -c shell -- hostname -i; kubectl exec sharepid  -c derby -- hostname -i ` 可以發現兩個 container 的 hostname 與 IP 是一樣的
`kubectl exec -it sharepid  -c shell -- sh` 進入之後，打 ps 會發現 pid 1 為 `/pause` ，執行的是 `sleep infinite`
- 並且輸入 `kill -9 1` 也沒辦法砍掉大哥
    - 為何 `kill -15 1` 會跳出來？ -> ~~因為 ctrl+c 會導致 container 的命令中斷，每個 container 的程序都會收到這個訊號而中斷 所以不是 `pause` 被中斷，而是底下的程序掛了，container 被關閉~~
    - **老師隔天更正**： `kill -15 ` 是送正常關機的訊號，因此 `pause` 當然可以被關掉
    - k8s 1.22.2 版 ，所配置的 `pause` 為 v3.5 (`k8s.gcr.io/pause:3.5`)
    - `sudo vi /etc/containers/containers.conf` 找到 `infra_image=pause` 可以修改版本
        - 版本代號改成 3.2 測試 ，並使用 `sudo podman pull k8s.gcr.io/pause:3.2` 抓下來
- `ls /proc/7/root` pid 7 為 derby container , 這樣就可以看到 derby 內的檔案系統，可以直接傳輸資料
    - 使用記憶體的速度，快速且穩定
    - ` ls /proc/1/root` 可以看到 pause 的內容 (1.20 版不行，現在是使用 1.22) ，內容約為 800K

```yaml=
#sharepid.2.yml
apiVersion: v1
kind: Pod
metadata:
  name: sharepid
spec:
  shareProcessNamespace: true
  containers:
  - name: derby
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Never
  - name: shell
    image: quay.io/cloudwalker/busybox
    command:
      - sleep
      - "60"
  restartPolicy: Always  # 其實 pod 的預設就是 Always , 可改成 Never 代表不重開
  
```

```bash=
FROM quay.io/ict39/alpine
RUN apk update &&  apk add bash curl && apk upgrade && \
 curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash && \
 mkdir /opt/fbs
ENTRYPOINT ["filebrowser", "-a", "0.0.0.0", "-p", "8080", "-r", "/opt/fbs"]
```
apply 不是所有設定都可以用，是局部的 `imagePullPolicy` 不行，`portnumber` , `replica` 可以
- `spec.nodeSelector.*` 就不行
- `service 的 externalIP`可以


可以透過 label selector 來決定要在哪個 Node 開 pod
```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: p1
spec:
  containers:
  - name: c1
    image: quay.io/cloudwalker/busybox
    imagePullPolicy: Never
    stdin: true
    tty: true
  nodeSelector:
    kubernetes.io/hostname : m1 
```

在指定 `nodeSelector` 的部分也可以給予多個值

```yaml=
#nodeselect.yaml
apiVersion: v1
kind: Pod
metadata:
  name: p1
spec:
  containers:
  - name: c1
    image: quay.io/cloudwalker/busybox
    imagePullPolicy: Never
    stdin: true
    tty: true
  nodeSelector: # 設定多個 Node 選擇，但如果沒辦法找到就會一直停在 pending
    kubernetes.io/hostname : m1
    disktype: ssd  
```
`kubectl label nodes m1 disktype=ssd`

> kubernetes.io/control-plane=,node-role.kubernetes.io/master= 在 master 上的標籤，為何是空的?
> 問： stdin : true 的用途? -> 很可能是跟未來的 log 紀錄有關，當 process 在螢幕上有輸出一些東西時， stdin 會決定要不要存到 buffer 裡 (吧，尚未證實)

`kubectl label nodes m1 disktype=ssd`
`kubectl attach` 可以直接進入 `pod` 內，



:::spoiler
```yaml=
#c.dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: filebrowser
spec:
 replicas: 3
 selector:
  matchLabels:
   app: ict39.fbs
 template:
  metadata:
   labels:
    app: ict39.fbs
  spec:
   containers:
   - name: filebrowser
     image: quay.io/ict39/alpine.fbsz
     imagePullPolicy: IfNotPresent
     ports:
     - name: fbsfk
       containerPort: 8080
```

```yaml=
#c.service.yml
kind: Service
apiVersion: v1
metadata:
  name: ict39srv
spec:
  selector:
    app: ict39.fbs
  ports:
  - protocol: TCP
    port: 8888
    targetPort: 8080
  externalIPs: ['192.168.234.4']
```
:::

再次複習：
- kubelet ： 負責開啟 pod ，是在背景執行的 daemon
- cri ： 開啟 container 的，可使用 `docker` 或 `cri-o(目前使用)` 

### Liveness and Readiness Probes
這兩個都是由 `kubelet` 產生的偵測系統

- **Liveness Probes** ： 偵測 pod 是否活著，浴火重生
    - 偵測到錯誤後，就會針對 pod 浴火重生 (restart)
- **Readiness Probes** ： 偵測 pod 是否準備好
    - 若沒準備好，在 Ready 欄會 -1 ，但不是浴火重生 (依然維持 running)
    - 並且失敗的話就會把 `endpoint` 隱藏

一共有三種偵測方式：
1. ExecAction：到 `pod` 內部執行 `Linux` 指令 (e.g. `cat` )，看 exit code 是否為 0 (正常執行)
2. TCPSocketAction： 使用 `nc` 檢查 TCP 的 port
3. HTTPGetAction： 使用 `wget,curl` ， 看回傳代碼是否在 200~400 之間 

```yaml=
# exec-liveness.yml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 20; rm -rf /tmp/healthy; sleep 300 # 睡完 300 秒之後就離開了
    livenessProbe: # kubelet
      exec:
        command: # 發呆五秒之後執行 ， 是在 pod 裡面執行的 
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5 # 先發呆五秒
      periodSeconds: 5 # 每五秒確定一次
```

以上的例子中，`livenessProbe` 會每五秒執行 `cat /tmp/healthy` 確定這個 pod 是否正常運作
- 當這個指令回傳的 exit code 不為 0 時，就代表 `pod` 掛了，需要浴火重生
- 通常是針對其中一個**最主要**的 `container` 來做偵測
    - 因為 `application` 會有一個最主要的 `container` 來執行，我們只需要監控他就好了
    - 若有需求偵測多個的，應該有另外的寫法可以指定
- args 與 command 的差異：
    - command -> dockerfile 中的 **Entrypoint**
    - args -> dockerfile 中的 **CMD**

補充：args 與 command 雖然可以對應到 dockerfile 的 Enterpoint 與 CMD，但在混用上比較沒辦法
大概測試了以下幾種情況：
1. image 有定義 CMD `python`, yaml 定義 command (entrypoint) `app.py` -> 失敗，無法執行
2. image 定義 CMD `python app.py`, yaml 定義 command `python app.py` -> pod 變成執行 `python app.py python app.py`，因為第一個 python app.py 之後會變成參數，所以可以順利執行

> 結論：是可以混用的，只是很不方便（還要去查 image 的設定再來定義 yaml)，比較簡單的管理方法就是直接在 yaml 上定義 command + args, 就可以避免一些意外狀況

> 浴火重生是 vmware tanzu 在發表會時所說的

```yaml=
# tcp-readiness.yml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-readiness
spec:
  containers:
  - name: tcp-readiness
    image:  docker.io/library/alpine
    args:
    - /bin/sh
    - -c
    - sleep 10; timeout 10s nc -l -p 9999; sleep 30; nc -l -p 9999 # timeout 是 linux 的指令
    ports:
    - containerPort: 9999
    readinessProbe:
      tcpSocket:
        port: 9999
      initialDelaySeconds: 5
      periodSeconds: 5
```
以上的例子 `readiness` 去偵測 TCP 9999 port ，若不通的話就把 Ready -1
- timeout 20s ： 代表後面的指令我只想讓他執行 20 秒 (`nc`)
- 使用 `tcpSocket.port` 指定要偵測 TCP 9999 port ，每五秒偵測一下
- 所以以上的結果會是：先 0/1 ,10 秒後 1/1 , 再有 30 秒 0/1 , 再變成 1/1 後就不變了

#### Startup Probe - Exec
在 `readiness` 與 `liveness` 之前，先確認 pod 有正確開啟
- 如果 startup 失敗，後面那兩個探針都不會運作
- 重點參數：
    - failureThreshold ： 給予的次數，超過失敗次數之後就直接關掉，對 pod 浴火重生
    - periodSeconds ： 嘗試的間隔秒數

:::spoiler `Startup Probe.yaml`
```yaml=
# exec-startup-liveness.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: startup-liveness-exec
spec:
  containers:
  - name: startup-liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - sleep 30; touch /tmp/healthy; sleep 20; rm -rf /tmp/healthy; sleep infinite
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      failureThreshold: 4 # 給他 4 次機會，4次都失敗才會說他失敗，就把 pod 關掉
      periodSeconds: 10 # 每次的間隔
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 10 
```
:::

- k8s 會使用一個 pause 容器來預先建立好 pod 要使用的 namespace
    - 裡面run的是 `sleep infinite` 睡夢羅漢
- pod 是 k8s 的運作最小單元，裡面可以跑多個 container (至少一個)
    - 內部的 container 要求至少要有一個**前景**執行的 process
- 同一個 pod 不行跨 node 存放
- 可以設定 pod 讓內部的 container 共用 namespace (`spec.shareProcessNamespace: true`)
    - 如果共用的話，pod 內的 container 可以透過記憶體傳輸資料 (就不用透過網路)
- 設定 CNI 之後，pod 內部的 container 可以共用網路介面 -> 可以使用 `localhost` 來互連

### Pod Namespace

跟 Liunx Namespace 不一樣哦!!

```yaml=
#pod-n1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: n1
  namespace: myring # 指定 namespace
  labels:
    name: n1
spec:
  hostname: n1
  containers:
  - image: quay.io/cloudwalker/busybox
    command:
      - sleep
      - "3600"
    name: n1
```

![](https://i.imgur.com/0JhkakC.png)

但因為現在沒有 namespace : myring ，會報錯 , 所以要創 `kubectl create namespace myring`
不同使用者會有不同的 `namespace` 可以使用
- 可以在同一個 k8s 叢集中，新增出隔離環境給不同的使用者使用
- 並且 `namespace` 可以設定 `firewall` 阻隔之間的網路傳輸 (v1.2x 後才有的)
- 可以針對 `namespace` 來限制硬體資源 (CPU,memory) 等

### Multi-Container Pod Design Patterns
先跳過惹
說是論文等級的技術


## 307-Kubernetes-Service

![](https://i.imgur.com/CdowzsY.png)

開啟一個對外的 Service ，以免 pod 浴火重生之後 IP 改變
會有以下這些 Service type 可以選擇：
- ClusterIP ： 為預設值，會使用 Cluster-internal IP 來連線，內部測試用的
    - 只有內部的 node,pod,與其他 service 可以連到，網路上的使用者也沒辦法連進來
- NodePort
- LoadBalancer
- ExternalName

```yaml=
# s1.dep.yml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: s1.dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: s1.pod
  template:
    metadata:
      labels:
        app: s1.pod
    spec:
      containers:
      - name: derbyapp
        image: quay.io/cloudwalker/alpine.derby
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 8888  
```

```yaml=
#s1.service.yml
kind: Service
apiVersion: v1
metadata:
  name: s1-service
spec:
  selector:
    app: s1.pod
  ports:
    - port: 9999
      targetPort: 8888
```

### Service NodePort

- nodePort ： 全面開戰，意思是所有的 `Node 的 IP` 都會開啟這個 Port 提供服務
    - 若有 200 個主機，這 200 台主機都會開啟這個 Port 來用 (有 200 個端點可以使用)
    - 所以正常的 production 環境不會這樣用，workload 太大了，代表所有主機都要運作
    - **實際上下面的 external IP 比較常使用** 

```yaml=
# s1.nodetype.yml
apiVersion: v1
kind: Service
metadata:  
  name: my-nodeport-service
spec:
  selector:
    app: s1.pod
  type: NodePort # 預告服務為 "NodePort"
  ports:
  - port: 9999
    targetPort: 8888
    nodePort: 30036 # nodePort 一定要在 30000~32767 中，不然會不給開
    protocol: TCP
```
![](https://i.imgur.com/rdAYeOS.png)

### Service External IP

可以在 `yaml` 中指派多個 node 的 IP ，決定要用那些 Node ，就不用全面開戰
```yaml=
# s1.externalip.yml
kind: Service
apiVersion: v1
metadata:
  name: myextip
spec:
  externalIPs:
  - $IP # 這個是 Linux 的環境變數，YAML 檔是不吃環境變數的!! , 可使用 envsubst
  - 192.168.234.6 # 用陣列的方式，一次給一個
  selector:
    app: s1.pod
  ports:
  - port: 8080
    targetPort: 8888

```
`cat s1.externalip.yml | envsubst | kubectl create  -f  -` 先把 $IP 轉換成數字，再當作 stdin 丟給 `kubectl create`
使用 `cluster-ip` 與 `external-ip` 都可以連到 8080 port

### DNS for Service
`kubectl -n kube-system get svc`
`kubectl describe service kube-dns -n kube-system`
會發現有兩個 `pod: core-dns` 在提供服務，並且 `service` 的兩個 `endpoint` 就是對應到這兩個 `pod`

> 這個 DNS 是給誰用的??
> -> pod,service ，沒有 Node (若 Node 也要使用就把 /etc/resolv.conf 改成 DNS service 的)
> pod 使用 , service 紀錄

:::spoiler service-dns.yaml (三個大包裝)
```yaml=
# service-dns.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-nginx
spec:
  selector:
    matchLabels:
      run: k8s-nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: k8s-nginx
    spec:
      containers:
      - name: k8s-nginx
        image: nginx
        ports:
        - containerPort: 80
--- # 三個 --- 代表要另外再定義一個，通通寫在同一個檔案內
apiVersion: v1
kind: Service
metadata:
  name: svc-cluster
spec:
  selector:
    run: k8s-nginx
  ports:
  - name: http
    port: 80
    protocol: TCP
  
---
kind: Service
apiVersion: v1
metadata:
  name: svc-headless # headless 代表不給 Cluster IP , 下面有定義
spec:
  selector:
    run: k8s-nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None # 這裡

```
:::

- headless ： 沒有提供 `Cluster IP` 的 `service`

![](https://i.imgur.com/QCRR8QU.png)

接下來使用 `nslookup` 來對 DNS 做搜尋
- `server 10.98.0.10` 指定 DNS server
- 格式為： `SERVICE_NAME.NAMESPACE.svc.cluster.local`
    - e.g. `svc-headless.default.svc.cluster.local`
        - 這個 `headless` 版的會回傳所有 `pod` 的 IP，因此這個範例會噴 3 個 IP
        - 如果在 `pod` 內去 ping 他的話，就會從三個 pod 隨意回傳 (load-balancing)
    - `svc-cluster.default.svc.cluster.local`
        - 只會吐出一個 Service 的 IP


## 305-Kubernetes-Volume (CSI)

![](https://i.imgur.com/aHacMGE.png)

目前有三種
- Container (Overlay2) ： container 關掉就消失
- Volume (emptydir) ： pod 關掉就消失
- Persistent volume (hostPath,NFS) ： cluster 關掉就消失

```yaml=
# pod-hp.yaml 
kind: Pod
apiVersion: v1
metadata:
  name: pod-hp
spec:
  containers:
    - name: pod-hp
      image: quay.io/cloudwalker/nginx
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: hp-volume
  volumes:
    - name: hp-volume
      hostPath:
        path: /opt/hostpath # 這個目錄要手動在每個 node 創 (m1,w1,w2 都要)

```

## 50-DT-Overview (hadoop)
![](https://i.imgur.com/6HQx8kw.png)

Cloudera  ： 大數據的權威公司，開發 `hadoop` 的人就是裡面的員工
CDH（Cloudera's Distribution Including Apache Hadoop )，將部署和維護 Hadoop 所需的工具，打包到一個發行套件中
- 原本是免費的，結果去年開始要收錢了，而且是每年，每台主機每年需要 1W 美金 ，7台就要燒 7W 美金 (OMG)
- 但我們可以透過自己裝 `bare metal hadoop` 就不用錢了啦 

以前會叫做**大數據平台**，不過現在新的講法是 `DT`
-> `Data technology` ： 資料科技，要產生 `IOT` , `BI` , `machine learning` 。以 `hadoop`(大數據)為基底去做堆疊成 `DT`
> 資料管理師，資料工程師，資料分析師，資料科學家 (數學，統計機率要好)，最大的是資料架構師 (前後端都要會)

ecosystem ：生態系統，代表一個系統底下有很多細節
在台灣的話，資料到 T 就是大數據 (google 已經到 ZB)

![](https://i.imgur.com/6qdyUlO.png)

Hadoop 是資料科技生態系統的基礎，以下開始介紹這張圖

- 左邊指的是資料來源，可能有以下的：這些都是要用來丟進 `hadoop` 的
    - IT 應用系統，資料庫 (e.g. mariadb)
    - opendata (open API)
    - IoT
    - 要注意的是保證 data 的品質，不要 garbage in , garbage out
- 右上角那個是 `apache hive` 可以執行一些 `SQL` 命令，來分析我們的資料
    - 商業智慧 Business Intelligence (BI) ：`hive` 幫忙做到，採用線上分析處理技術 (IT -> BI)
- 左下角 `Spark` 可做到 `machine learning` ，使用 `python` 語言
- 右下角 `HBase` ， 物聯網架構，為 `NoSQL` ，架構在 `hadoop` 上，是以時間序為主的資料，很適合處理物聯網資料

![](https://i.imgur.com/YqN1bEE.png)

RAW data ： 由 `sensor` 傳過來的資料，通常是不允許去修改的
- variety ： 
    - 紀錄 IT 的 log 檔
    - 社群媒體 (但很多假資料)
    - Opendata
- VOLUME ：至少為 TB 的資料
- veracity ：最重要的**資料品質**，直接決定生成的 BI 是否有參考價值

![](https://i.imgur.com/UnSwIfn.png)

一定要有多台電腦才能形成 `hadoop` 系統，並且一定有包含以下四個東西

* **Hadoop Common**: 支援大多數的 Hadoop 模組, The common utilities that support the other Hadoop modules.
* **Hadoop Distributed File System (HDFS™)**: Distributed 分散存放，代表會由很多台實體電腦組成，這樣才厲害
* **Hadoop YARN**: 負責跑程式的 (e.g.： `.net` , `jvm` ) ，並且可以在多台電腦平行處理
* **Hadoop MapReduce**: 資料分析的引擎 (但我們應該會使用 `spark` 來取代 ),A YARN-based system for parallel processing of large data sets.


> * Small File Concerns
> * Slow Processing Speed
> * No Real Time Processing
> * No Iterative Processing
> * Ease of Use
> * Security Problem

![](https://i.imgur.com/WRFRYwD.png)

- **ds101** ：給資料分析師用
    - 需要永存資料 (PV)
- **nna** ： `active,Secondary name node` `Java` 程式，跑這兩個程式
    - 裡面是存放 `HDFS` 的 metadata (檔名大小等) ，也需要永存資料區
    - 使用 `NFS+PV` 來做到共享 `share edit logs`
    - `HDFS = SNN + NN + SNN + DN`
- **rma** ： `Resource Manager` ，也是一隻 `Java` 程式
    - 由 `yarn` 組成的系統，搭配 `Resource Manager` 與 `Node manager`
- **wka01~04** ： `data node` , `node manager` ，也都是 `Java` 寫出來的
    - 其實左右兩邊沒差啦，01~04 都是做一樣的事
    - `data node` 是跟 `name node` 一起的
    - <font color=red>但不會使用 NFS 來共享檔案，因為網路速度沒那麼快</font>
        - **企業等級要至少到 100Gps 才有可能使用網路儲存**
- standby 跟 shared log 沒做，因為環境不夠了

總共會做這麼多台電腦，在我們的環境就是產生好這些 pod (因為實際上 `hadoop` 也是用 JAVA 組成的 (`application container`)

> hadoop需要多台電腦叢集建立，我們可用學過的k8s叢集功能，建立多個pod來跑hadoop裡面的java程式，形成一個資料科技平台，並修改建立成為自創的獨一無二的資料科技平台 (謝謝班長)

## 328-Kubernetes-DT

### 製作 myus20.04 Image 

```bash=
wget http://www.oc99.org/dt/hadoop/myus20.04.zip
unzip myus20.04.zip
cd myus20.04
# 裡面有包含 ssh 憑證檔，之後就可不用密碼使用 ssh 連線

```

```dockerfile=
FROM localhost/ubuntu:20.04

COPY bin/bash.append /tmp/
RUN  mkdir -p /etc/skel/.ssh
# 只要是 Linux 就會把 /etc/skel/.ssh 裡面的所有內容丟至 ~/.ssh 所有使用者的家目錄裡面
COPY ssh/* /etc/skel/.ssh/

# DEBIAN_FRONTEND noninteractive 把任何的互動介面，使用預設的選項來回答
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -y nano locales apt-utils elinks tree unzip curl wget jq psmisc npm nfs-common && \
    apt-get install -y openssl netcat net-tools ssh zip sudo golang iputils-ping dnsutils gettext-base && \
    # /opt/bin 之後要用的程式會放這 , /opt/www 前端網站 , /opt/dataset 做分析的資料檔放的位置 /opt/mdb 放 SQL 命令 ,/mnt/sysbak k8s 的 yaml
    mkdir /opt/bin /opt/www /opt/dataset /opt/mdb /mnt/sysbak && \
    # 能夠顯示 en_US (ascii 碼) en_US (UTF-8 萬國碼) , 以及繁體中文包 passwd -dl root 冷凍 root 的帳號
    locale-gen en_US && locale-gen zh_TW.UTF-8 && locale-gen en_US.UTF-8 && sudo passwd -dl root && \
    # 設定時區
    sudo ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \

    # 以下命令會同時安裝 Python2 & Python3 , spark 要用的
    apt-get install -y python3 python3-numpy python3-pandas python3-pip python-is-python3 && \
    # 把一些用不到的檔案清一清，包含快取檔
    apt-get -y upgrade && apt-get autoremove -y && apt-get clean && rm -rf /var/cache/apt/archives && \

    # 建立管理者帳號 bigred
    groupadd bigboss && echo '%bigboss ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    # bash.bashrc 當互動非登入式介面時，就會執行 -> 執行 /bin/bash 時
    cat /tmp/bash.append >> /etc/bash.bashrc && rm /tmp/bash.append && \
    useradd -m -s /bin/bash bigred && echo 'bigred:bigred' | chpasswd && \
    usermod -a -G bigboss bigred && chmod -R 700 /home/bigred/.ssh
 # PermitUserEnvironment yes 讓使用的目錄的 .ssh key 被使用
 # StrictHostKeyChecking no 印象中是不用回答 yes
RUN echo 'PermitUserEnvironment yes' >> /etc/ssh/sshd_config && \
    echo 'StrictHostKeyChecking no' >> /etc/ssh/ssh_config && \
    # 登入訊息處理
    rm /etc/update-motd.d/* && echo "Ubuntu 20.04 LTS : IP `hostname -I`" >/etc/update-motd.d/00 && \
    sudo chmod +x /etc/update-motd.d/00

USER bigred
WORKDIR /home/bigred/
CMD ["/bin/bash"]
```
```bash=
cat bin/bash.append
alias ping='ping -c 4'
alias dir='ls -alh '
alias bye='sudo shutdown -h now'

# [ -z $SSH_TTY] 的意思是，如果不是使用 ssh 登入則執行(設定英文語系) ，如果是 ssh 登入的話就用中文語系
if [ -z "$SSH_TTY" ]; then
   export LANG=en_US.UTF-8
   export LANGUAGE=en_US
   export LC_ALL=en_US.UTF-8
else
   export LC_ALL=zh_TW.UTF-8
fi
#當 image 產生 pod 時成立，因為 container 沒有開機程序，所以自己模擬出開機程序
if [ "$USER" == "" ]; then
    # 這裡面就會放維運相關的內容 ，先準備好要的開機過程
   if [ -f /opt/bin/dkc.boot ]; then
      /opt/bin/dkc.boot
   fi
   sudo /etc/init.d/ssh start
else
    # 把我們自己準備好的 rc 檔準備好，用 ssh 登入之後會執行
   if [ -f /opt/bin/dkc.bash ]; then
      source  /opt/bin/dkc.bash
      [ ! -d "/home/$USER/bin" ] && mkdir /home/$USER/bin
      export PATH=/home/$USER/bin:$PATH
   fi
fi
```
### image 的開機流程
以上 image 的開機流程 -> 執行 CMD的 `/bin/bash` -> `/bin/bash` 為非互動非登入的形式，因此會執行 `/etc/bash.bashrc` -> 裡面在 `if [ "$USER"  == "" ]` 的部分有包含 `/opt/bin/dkc.boot` 與 `sudo /etc/init.d/ssh start`
之後若用 `ssh` 登入的話就是會執行 `/opt/bin/dkc.bash` 的部分
統整：分成開機流程與 `ssh` 登入
- 開機流程
    - `CMD ['/bin/bash']`
    - `/etc/bash.bashrc`
    - `/opt/bin/dkc.boot && sudo /etc/init.d/ssh start`
- ssh 登入流程
    - `/opt/bin/dkc.bash`
    - `/etc/environment (下一張image的)` 是 `ssh` 的高階用法

[apache.org top project ，裡面有不少中國的專案 e.g. kylin](https://www.apache.org/index.html#projects-list)
看 project 的時候版本號怎麼看？以 `hadoop` 為例 ，官網上就有三種版本 `3.3.1` ,`3.2.2`, `2.10.1`

- 版本號是這樣 `major.minor.patch`
- **major** ： 代表大更新，所以現在有兩種大版本可以選擇 (2,3)
    - 2 會比起 3 有著更多的堆疊功能
- **minor** ： 代表小更新，所以 `2.10.1` 就代表第二版已經有更新了 10 個版本 ，可以依照需求的功能去選擇版本
- **patch** ： 代表前面版本的修正 (`2.10.1` 代表修正過一次) ，也就是說看**穩定度可以參考這個**
    - 不會針對內容進行更新，而是抓一些 bug 之類的
    - 其中最急迫的更新為 `hotfix`, 但就不會再額外寫一個版本號

> HDinsight 是 azure 的產品，裡面也是用到了一堆 apache 的專案，如 `kafka,hive,Hbase,zookeeper` ，也有依照版本的不同去對應到每個專案的版本

### 製作 dt210 Image 

```dockerfile=
# Dockerfile
FROM localhost/myus20.04

# 把剛剛上網下載的那些 tar 檔丟進去
ADD hadoop-2.10.1.tar.gz apache-hive-3.1.2-bin.tar.gz pig-0.17.0.tar.gz spark-3.0.1-bin-hadoop2.7.tgz /opt/
COPY parquet-pig-bundle-1.6.0.jar /opt/pig-0.17.0/lib/
# 讓 hive 可以讀取 mariadb
COPY mariadb-java-client-2.3.0.jar /opt/apache-hive-3.1.2-bin/lib
# 放 hadoop 設定檔
COPY hdp210/* /opt/hadoop-2.10.1/etc/hadoop/
# 放 spark 設定檔
COPY spk301/* /opt/spark-3.0.1-bin-hadoop2.7/conf/

COPY bin/* /opt/bin/
# 使用 ssh 執行命令, 會讀取 /etc/environment 這個設定檔
# 裡面是定義了很多變數，路徑等等
COPY ssh/environment /etc/environment
# 裡面兩行 Proxy 設定，給 apt 看的，設定 APT 透過 Proxy 來抓取、更新套件
COPY apt.temp /etc/apt/
# 安裝 JAVA jdk8
RUN sudo apt-get update && DEBIAN_FRONTEND=noninteractive sudo -E apt-get install -y mariadb-client nfs-common openjdk-8-jdk libisal-dev libsnappy-dev libz-dev && \
    DEBIAN_FRONTEND=noninteractive sudo -E apt-get -y upgrade && DEBIAN_FRONTEND=noninteractive sudo -E apt-get autoremove -y && \
    sudo apt-get clean && sudo rm -rf /var/cache/apt/archives && sudo chown -R root:root /opt && \
    mkdir -p /opt/dataset

CMD ["/bin/bash"]
```

```bash=
# m1 ,w1,w2 都要裝
scp gamma@120.96.143.35:~/dt210.export.tar ~/dt210
sudo podman import ~/dt210.export.tar dt210
sudo podman run --name d2 -it -d -p 22100:22 localhost/dt210 bash
ssh localhost -p 22100
#進到 d2 
hadoop checknative
# 可以看到很多東西，除了 SSL 是 false
# 因為 SSL 會影響到效能，我們現在的環境不夠好
```

### 建置 資料科技 平台 

會用到的 `.yaml` 檔放到作業那邊了
```bash=
cd; wget -q http://www.oc99.org/dt/hadoop/dt.zip; unzip dt.zip &>/dev/null; cd dt/
bin/dtadd # 裡面可能需要改一下 上面 mkdir 的部分要 sudo
kubectl apply -f ~/dt/yaml/. 一次部
# bin/dtdel 刪除
# 會用到 bin/mydy 裡面的各種名稱
```
可以在定義 pod 時指定 `subdomain` 與 `hostname` 來記錄 pod 的 DNS
在創建的時候，會直接指定 pod 的 Node ，原因是 `hadoop` 自己就有容錯能力，所以就不需要 `k8s` 來幫忙容錯
-> 並且 `hadoop` 的容錯機制也跟 `k8s` 不同，還有指定 Node 是因為資料要放在同個地方

```bash=
#adm100
formathdfs # 最主要是裡面的 ssh nna 'hdfs namenode -format -clusterID cute'
# 可能會遇到檔案權限問題
starthdfs
#會開啟各種 name node 等等
# 其實 hdfs format 就只是在 nna 上面建立 nn 目錄，其他的就是指 rm -rf 清理而已

# nna
jps #120 NameNode 283 Jps 207 SecondaryNameNode

# adm100
# hdfs 現在是使用這個新的檔案系統，不要搞錯惹
hdfs dfs -ls / # 去讀取 fsimage 的內容
hdfs dfs -mkdir /zzz 
hdfs dfs -rm -r /zzz
#
mkdir -p ~/opendata/dip
wget 'https://data.moi.gov.tw/MoiOD/System/DownloadFile.aspx?DATA=BC553073-6E98-47CC-92FA-6DB9720D9993' -O ~/2016dip.zip
wget 'https://data.moi.gov.tw/MoiOD/System/DownloadFile.aspx?DATA=105529BB-C39A-4B0C-8C26-A2FE88FD51C0' -O ~/2017dip.zip
wget 'https://data.moi.gov.tw/MoiOD/System/DownloadFile.aspx?DATA=CBA7C5C1-D7F2-499E-94EA-DCBAB9F66EBE' -O ~/2015dip.zip
mkdir /tmp/data; cd /tmp/data; unzip -n ~/2016dip.zip ; unzip -n ~/2017dip.zip ; unzip -n ~/2015dip.zip
mv 2015-DailyImmigPort-All.csv 2016-DailyImmigPort-All.csv 2017-DailyImmigPort-All.csv ~/opendata/dip/ 
cd; rm -r /tmp/data 

cat 2015-DailyImmigPort-All.csv  | cut -d',' -f1,2 >> ~/opendata/dip/dailydip.csv
cat 2016-DailyImmigPort-All.csv  | cut -d',' -f1,2 >> ~/opendata/dip/dailydip.csv
cat 2017-DailyImmigPort-All.csv  | cut -d',' -f1,2 >> ~/opendata/dip/dailydip.csv
```

`fsimage_00000000` 是在 `nna(namenode)` 裡面，記錄所有的檔案資訊

## 53-DT-ELT-HDFS
```bash=
# 要修改 20196 月的網址
wget -qO - http://www.oc99.org/dt/mydip1.sh | bash
tree opendata

cat opendata/dip/airport.list
cat opendata/dip/seaport.list
# 要修正資料的雜質
wget -qO - http://www.oc99.org/dt/mydip2.sh | bash
tree opendata/dip/all
```
### HDFS 基本操作命令


```bash=
# adm100
wget -qO - http://www.oc99.org/dt/getcsv.sh | bash
#顯示使用者 HDFS 家目錄
pig -e 'pwd'  2>/dev/null
hls
# starthdfs 如果上面的指令有問題的話
pig # 進入 quit 離開 ，因為現在還沒有家目錄，所以要先離開創一下
hdfs dfs -mkdir -p /user/bigred
pig
grunt> ls
grunt> mkdir zzz
grunt> ls
# hdfs://nna:8020/user/bigred/zzz <dir>
grunt> rm zzz
hdfs dfs -put ~/myopendata/customer.csv  mydataset/
hdfs dfs -ls  mydataset/
hdfs dfs -cat  mydataset/customer.csv | head -n 3

hdfs dfs -put -f  ~/myopendata/customer.csv  mydataset/ # 強制覆蓋
hdfs dfs -appendToFile ~/myopendata/customer.csv mydataset/customer.csv # 把內容加在後面

wget --no-check-certificate https://stats.moe.gov.tw/files/school/103/u1_new.txt
#file -bi u1_new.txt
## text/plain; charset=utf-16le
  
# iconv -f utf16 -t utf8 u1_new.txt
iconv -f utf-16le -t utf8 u1_new.txt >103u.txt
```

utf-16 ：所有字都用 `2byte` 來儲存，每一國的語言都顯示
utf-8 ： 英文 `1 byte` , 中文 `3 byte`

**hadoop 檔案系統不可以進去修改檔案，原因是因為要處理 Raw Data (不應該改)**
```bash=
echo "224.2.2.2 ICRT" > radio.txt
hdfs dfs -put radio.txt mydataset/TW/
sed -ie '1i224.1.1.1 FM989' radio.txt
hdfs dfs -put -f mydataset/TW/radio.txt

```

## HDFS 共享檔案系統

```bash=
# 創建其他使用者的家目錄
# adm100
hdfs dfs -mkdir -p /user/{rbean,gbean}
hdfs dfs -chown rbean:rbean /user/rbean
hdfs dfs -chown gbean:gbean /user/gbean
#ds101
sudo useradd -m -s /bin/bash rbean
echo rbean:rbean | sudo chpasswd
sudo useradd -m -s /bin/bash gbean
echo gbean:gbean | sudo chpasswd
# 各自登入之後 使用 pig -e 'pwd'  2>/dev/null 確認家目錄
#使用 gbean
echo 'test123...' > data.txt
hdfs dfs -put data.txt # 後面沒有指定路徑，就是在家目錄創
hdfs dfs -ls # 沒指定位置，也是指定一下家目錄
# 使用 rbean
hdfs dfs -get /user/gbean/data.txt # 把 hdfs 裡面的檔案抓出來到 linux
ls 
hdfs dfs -cp /user/gbean/data.txt /user/rbean # 複製 hdfs 的內容
#adm100 bigred
hdfs dfs -mkdir /app
hdfs dfs -setfacl -m user:rbean:rwx /app
hdfs dfs -put mysys.sh /app
# 執行程式
hdfs dfs -cat /app/mysys.sh | bash

```

```bash=
# 啟動 yarn
# adm 101
startyarn
hls
# m1
dtest
vim ~/dt/conf/hdpconf/mapred-site.xml # 確認一下版本
# /opt/bin/dtest mapreduce 2.10.1 , spark 3.0.1

```
```bash=
# 流程總結
# adm100
# formathdfs 可能要
starthdfs
startyarn
dtest  # 有可能會有版本問題，mapreduce與spark 的
```
![](https://i.imgur.com/epXf7qK.png)

要傳進去 HDFS 的檔案，會先被切割成好幾個 block 來儲存
![](https://i.imgur.com/s1f2vfQ.png)

Client(adm100) 先跟 Name Node 說要存檔案 (`-put`)
- Name Node 會先記錄第一個檔案放在哪邊
- 接下來再由 work Node 來負責存放
    - 每一個都代表一個主機，Rack 代表是機櫃
- 以上的圖只有 A 區塊是怎麼存放的，其他還有 B C 要處理
- 是相同的區塊存三個，為了提高容錯率 ，所以就需要三倍的容量
    - 客戶需求 100T ，就要準備 300T 的硬碟
    - 如果只給 100T ，大概存到 33T 的時候就會發生錯誤
    - 要存幾份是可以自己調整的，要只存一份也沒問題

當 client 要存取檔案的時候，就要去呼叫 Name Node 得到區塊存放的清單
會盡可能地選擇不同裝置取得資料，速度比較快

```bash=
hdfs dfs -put /etc/passwd
hdfs fsck passwd -files -blocks -locations # 查看檔案存在哪
#wka02
sudo rm dn/current/BP-1599525509-10.233.0.30-1636003257560/current/finalized/subdir0/subdir5/blk_1073743138*
#adm100
hdfs dfs -cat passwd # 還是抓的到，並且剛剛刪的檔案會復原
# 如果抓到檔案是缺乏的那個就會復原
hdfs fsck passwd -files -blocks -locations # 如果把所有的檔案都刪掉之後 就會變成 CORRUPT

```

## 56-DT-Hadoop-MapReduce
![](https://i.imgur.com/6SR7elq.png)

`MapReduce` 為資料處理的引擎
`MapReduce` 跟 Daemon 不同，執行完就結束不會留在背景
- 左邊的三台機器分別是 `wka{01,02,03}` , 右邊的則是三台中隨機挑一台做整合
- 就像汽車的引擎，到達目的地後就會關掉
- map 程式一開始只能用 `java` 寫
    - 現在可以用其他的了
- 每一個 Block 都會執行一個 Map 程式，會在不同的機器上執行
    - 處理同一個檔案的 Block ，進行擷取過濾
    - 紅色的實心箭頭是透過網路傳輸
    - `shuffle` (Java 程式) 就是 `HDFS` 的 Block ，所以每一塊都不會超過 128MB
        - `shuffle` 負責分類，排序資料，再丟給 `reduce` 程式
        - `reduce` 程式進行加總，平均等基本分析
        - 最後的 `part` 就是運算後的結果，存至 `HDFS` 的資料區塊

`hive` 跟 `pig` 會負責產生 `MapReduce` 程式，就不用自己寫囉
`yarn jar` 啟動資料處理引擎
```bash=
# 程式就是一個 .jar 打包檔，裡面放著一大堆 Map 與 Reduce 程式，有不同的應用
# tera = TB , gen 產生
# -Dmapreduce.job.maps=1  只跑一個 MapReduce 程式就好
# 每一個資料 100 byte ,所以有三億筆資料也才 32G 而已沒很厲害
# 下面的 320m 是資料夾 ， 有防呆機制所以前面要先把 320m 資料夾刪掉
hdfs dfs -rm -r 320m &>/dev/null; yarn  jar /opt/hadoop-2.10.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar teragen -Dmapreduce.job.maps=1 3355400 320m
# terasort
# 使用 time 命令來記錄時間
hdfs dfs -rm -r sortest/ &>/dev/null; time yarn  jar  /opt/hadoop-2.10.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar terasort -Dmapred.reduce.tasks=1  320m  sortest/
# 把每一個 block 從 128M 調整成 256M
hdfs dfs -rm -r 160m &>/dev/null; yarn  jar /opt/hadoop-2.10.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar teragen -Dmapreduce.job.maps=1 -Ddfs.blocksize=268435456 6710800  640m
hdfs fsck 640m/ # 檢查640MB 用到幾個 block (3)
# 把 block 變大就會需要更大的記憶體來讓 map 處理
# 使用兩個 reduce 來處理資料 ， 所以就會生成兩個 320M 的資料
hdfs dfs -rm -r sortest/; yarn  jar  /opt/hadoop-2.10.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar terasort -Dmapred.reduce.tasks=2  640m  sortest

```

spark 比 hadoop 快 100 倍
壓力測試可以使用上面的系統，來檢測一下 hadoop 的系統速度

## 58-DT-Hadoop-YARN
![](https://i.imgur.com/VCCulB3.png)

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

```bash=
# adm100
curl -s http://rma:8088/ws/v1/cluster/metrics | jq | grep -E "totalMB|totalVirtualCores"

# m1
vi dt/conf/hdpconf/yarn-site.xml # 改裡面 memory 的一欄
#adm100
sudo nano /opt/hadoop-2.10.1/etc/hadoop/mapred-site.xml # 

```
前端1024:mapred-site.xml 後端yarn 規範896:yarn-site.xml
4G/2C 8G/2C , VMWare Player 的 core 用越多反而慢
`yarn.nodemanager.resource.cpu-vcores` 設定可以提供 map 的cpu , 其實可以超過電腦的核心數很多，反正電腦是多工系統