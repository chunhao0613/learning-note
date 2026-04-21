# Container-Core-Tech

:::warning

:::spoiler 目錄

[TOC]

:::

# Container 的目標

- Container 的中文翻譯：貨櫃
- Container 的目標：==**把一個 Process 變成一台電腦**==
- [Linux Process 詳細介紹連結](https://hackmd.io/@QI-AN/linux-process-part1)

## <font color=red> 運作架構圖 </font>

![image](https://hackmd.io/_uploads/rJxbaqHQA.png)


- App 企業要用的應用程式，也就是 Process
- 執行程式時，會用到 Libs （ System Library ）和 Bins ( 人家寫好的命令 )
- **Container 技術會把 Application + Libs + Bins 都隔離起來 ( isolated )，它就會變成一台電腦**
  > Libs 和 Bins 是相依檔
- 多個 Container 會共用 OS Kernel 和實體硬體 ( Server 伺服器 )
- OS Kernel : 包含 CPU 、記憶體、網路和檔案系統 ...等

如何證明程式再跑一定會用到相依檔 ?

```bash!
$ ldd /bin/sleep
        /lib/ld-musl-x86_64.so.1 (0x7fe2d7754000)
        libacl.so.1 => /lib/libacl.so.1 (0x7fe2d7622000)
        libattr.so.1 => /lib/libattr.so.1 (0x7fe2d761b000)
        libgmp.so.10 => /usr/lib/libgmp.so.10 (0x7fe2d75b5000)
        libutmps.so.0.1 => /lib/libutmps.so.0.1 (0x7fe2d75b0000)
        libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7fe2d7754000)
        libskarnet.so.2.11 => /lib/libskarnet.so.2.11 (0x7fe2d7578000)

$ ldd /bin/bash
        /lib/ld-musl-x86_64.so.1 (0x7faeff99a000)
        libreadline.so.8 => /usr/lib/libreadline.so.8 (0x7faeff888000)
        libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7faeff99a000)
        libncursesw.so.6 => /usr/lib/libncursesw.so.6 (0x7faeff82b000)

$ ldd /bin/sh
        /lib/ld-musl-x86_64.so.1 (0x7ff33072b000)
        libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7ff33072b000)
```


- 虛擬技術會有自己的 OS（system library），並透過 Hypervisor 和 Host OS 溝通，而 Container 技術是共用 Host OS 的 Kernel， 使用已經開啟過的 Kernel ，所以沒有自己的 OS。
- Container 是一台電腦，虛擬化也是一台電腦，哪個啟動速度快？
  Container，因為他沒有開機程序和關機程序，記憶體空間只會有 process 和相依檔，所以佔用不大。
- 為何 Container 不需要開機？
  Application 啟動啟動代表 Container 啟動，Application 關閉啟動代表 Container 關閉
- 甚麼是 Container ?
  - 把一個 process 和它的相依檔變成一台電腦，這台電腦我們又稱它為 Application Container
  - Application Container 的檔案系統為：Chroot + Overlay2 堆疊型檔案系統
  - Application Container 的網路系統 : Network Namespace + 虛擬網路 ( 虛擬網卡、虛擬網路線、slirp4netns 撥接網路和虛擬橋接器 )
  - 透過 CGroup 來控制 Application Container 的 CPU、記憶體...等硬體資源
  - 透過 Linux Namespace 可以隔離出自己的 uts ( 主機名稱 )、UserID、PID...等隔離空間
  - 會共用 Host OS 實體主機的 Kernel ( Linux Namespace、Cgroup、Overlay2...等就是 Kernel 的功能 )
  - seccomp + linux Capabilities + SElinux ( Centos 主力 ) + APPArmor ( Ubuntu 主力 ) 為 Application Container 的安全防護技術，可以保護 Application Container 以及防止 Application Contianer 破壞 Host OS 的 Kernel
- 如果在 Windows 作業系統上跑 Docker Desktop ，其實還是要透過 WSL 和 Hyper-v 來安裝 Linux OS ，Container 才會在上面跑
  > [參考文件](https://docs.docker.com/desktop/windows/install/)



---

# Container 核心關鍵技術

Container 使用以下 Linux 系統模組將作業系統的 Process 做成一部電腦
- **Namespace** 
	- 用來提供軟體程序不同的隔離功能
- **CGroup (Contol Group)**
	- 用來分配硬體資源
		- 可以限制process用多少記憶體，CPU用哪個核心
- **Chroot** 
	- 針對指定的軟體程序，改變它的根目錄
	- 針對一個Process有自己獨立的根目錄，可以將Linux檔案系統中的一個目錄當成自己的根目錄
- **Bridge/slirp4netns** 
	- 建立虛擬網路, 連接 Internet 
	- 針對一個Process設定專屬的網路系統（虛擬橋接器）
- **Overlay2** 
	- 用來建立 Container 的檔案系統
	- 堆疊型的檔案系統


> 2022年電腦基礎概論

==**Container 可以降低Human Error**==


:::spoiler **重點整理 by 程威**
:::danger
- Container 運作架構
    - Container [貨櫃技術] 主要目標：將一個程序變成一台電腦
        - 由 App、bins/libs 組成，將他們 isolated 變成一台電腦，這就是 Container
        - App｜程式主體
        - bins、libs｜相依檔案
        - bins｜別人寫好的命令
        - Libs｜提供系統的 API
        - 複數 Container 共用 HostOS 的 Kernel
- VM 本身屬於完全隔離，資訊安全性高，但資源消耗會比較大
- Container 與 VM 的差異
    - 在一個 Hypervisor/HostOS 之上
        - 虛擬機技術會存在複數的 OS（架設幾台虛擬機，就會有幾個 OS）
        - Container 中沒有 OS 的存在，不論幾個 Container 都只會有一個底層的 HostOS
        - Container 不存在 OS 開機程序，所以啟動速度較 VM 快
        - 因為沒有 OS 所以記憶體佔用也比 VM 少
        - 複數 Container 共用一個系統，App 開啟就等於電腦開機，App 關閉就等於電腦關機
- Container 的關鍵技術
    - Namespace｜提供程序不同的隔離功能
    - CGroup [Control Group]｜分配硬體資源，可以限制程序使用哪個實體核心與記憶體的多寡
    - Chroot [Change Root]｜改變程序的根目錄（指定 HostOS 的特定目錄為程序根目錄）
    - Bridge/slirp4netns｜為 Container 建立虛擬網路，連接 Internet（slirp4netns 用於撥號網路）
    - Overlay2｜建立 Container 的檔案系統（堆疊型的檔案系統）
:::

---

## Chroot （Change root）

主要功能：<font color=red> **重新設定指定 Process 的根目錄。** </font>

```bash!
Usage: sudo chroot < 新的根目錄 > < 目標程式 >
```

實作：

建目錄 ( 仿照 linux 檔案系統目錄 )

```bash!
$ mkdir -p ~/r2d2/rootfs/{bin,sbin,usr/{bin,sbin},dev,lib,proc}; cd ~/r2d2

$ tree
.
└── rootfs
    ├── bin
    ├── dev
    ├── lib
    ├── proc
    ├── sbin
    └── usr
        ├── bin
        └── sbin

9 directories, 0 files
```


`ldd` 命令查看 `sh` 命令的相依檔

```bash!
$ which sh
/bin/sh

$ ldd /bin/sh
        /lib/ld-musl-x86_64.so.1 (0x7f6b1d0b2000)
        libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7f6b1d0b2000)

```



拷貝 sh 命令的本體和它的相依檔

```!
$ cp /bin/sh rootfs/bin; cp /lib/ld-musl-x86_64.so.1 rootfs/lib/

$ tree
.
└── rootfs
    ├── bin
    │   └── sh
    ├── dev
    ├── lib
    │   └── ld-musl-x86_64.so.1
    ├── proc
    ├── sbin
    └── usr
        ├── bin
        └── sbin

4 directories, 2 files
```

- `/bin/sh` 拷貝到 `rootfs/bin/` 下面的原因 ?
- 相依檔拷貝到 `rootfs/lib/` 的原因 ?


更改 rootfs 目錄的權限屬性

```bash!
$ sudo chown -R root:root rootfs
```

- `-R` ，將目錄裡的子目錄和檔案的權限都更改



重新設定指定 Process 的根目錄

```bash!
$ sudo chroot rootfs /bin/sh
@alp:/$ ls -al
/bin/sh: ls: not found

@alp:/$ mkdir zzz
/bin/sh: mkdir: not found

$ echo $PATH
/home/bigred/bin:/home/bigred/vmalpdt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

- `echo` 可以使用的原因，sh 命令自帶的功能。
- `chroot` 會到 `rootfs` 目錄裡面執行 `/bin/sh` ，此時程式會載入記憶體會變成 process，並且把 rootfs 這個目錄變成 `/bin/sh` 這個 process 的根目錄

```
$ sudo chroot rootfs/ /bin/bash
chroot: failed to run command ‘/bin/bash’: No such file or directory
```

- 失敗的原因是因為，在 rootfs 目錄裡面，沒有 /bin/bash 這個 program 
- 提問：`/bin/sh`的根目錄在哪？
  `/home/bigred/r2d2/rootfs`
- 提問：`/bin/sh`的檔案在哪？
`/home/bigred/r2d2/rootfs/bin`


- 把 busybox 這個 program 拷貝到 `rootfs/bin` 目錄裡面，並執行 `chroot`

```bash!
$ sudo cp /bin/busybox rootfs/bin/

$ sudo chroot rootfs /bin/sh

@alp:/$ /bin/busybox --install -s 

@alp:/$ ls -al /bin
total 1648
drwxr-sr-x    2 0        0       4096 Jun 24 13:24 .
drwxr-sr-x    7 0        0       4096 Jun 24 13:23 ..
lrwxrwxrwx    1 0       0          12 Jun 24 13:24 arch -> /bin/busybox
lrwxrwxrwx    1 0       0          12 Jun 24 13:24 ash -> /bin/busybox
........

# ps 這個命令，內定一定會去讀取 /proc 目錄裡面的內容
@alp:/$ ls -al /proc
total 8
drwxr-sr-x    2 0        0             4096 Jun 24 13:03 .
drwxr-sr-x    7 0        0             4096 Jun 24 13:23 ..

# 因 /proc 目錄內容是空的，所以 ps 命令顯示的結果也是空的
@alp:/$ ps
PID   USER     TIME  COMMAND

@alp:/$ exit
```

---


### 建立rootfs目錄

```bash!
$ mkdir -p ~/c3po/rootfs; cd ~/c3po/rootfs; curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.16/releases/x86_64/alpine-minirootfs-3.16.1-x86_64.tar.gz; 

# 到 Alpine 的官網下載最小的檔案系統，版本代號 3.16.1
```

- curl 指令補充
- `-s` ，安靜模式，不顯示任何資訊
- `-o` ，將下載檔輸出成一個檔案

解壓縮 `alpine.tar.gz`，並把標準輸出丟到黑洞，最後刪除壓縮檔，回到上一層目錄。

```bash!
$ tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd .. 
```

我們現在的

```
bigred@alp:~/c3po$ dir rootfs/bin
total 820K
..........
lrwxrwxrwx  1 bigred bigred   12 May 29 14:20 base64 -> /bin/busybox
lrwxrwxrwx  1 bigred bigred   12 May 29 14:20 bbconfig -> /bin/busybox
-rwxr-xr-x  1 bigred bigred 822K May 22 06:59 busybox
lrwxrwxrwx  1 bigred bigred   12 May 29 14:20 cat -> /bin/busybox
..........

[註]  在 /bin 目錄中的 命令, 大都是交由 busybox 命令執行 
```

Linux 目錄解析

```bash!
$ dir rootfs/
total 76K
drwxr-sr-x 19 bigred bigred 4.0K May 20 09:33 .
drwxr-sr-x  3 bigred bigred 4.0K May 20 09:24 ..
# binary，裡面放Linux系統要用的命令/program，以後看到這個目錄裡面一定是執行檔
drwxr-sr-x  2 bigred bigred 4.0K May 20 09:33 bin 
# device，Linux系統所用的硬體設備資訊，例如/dev/zero，只要使用它就會一直噴出null
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 dev    
# Linux系統用的軟體和設定檔所在目錄
drwxr-sr-x 15 bigred bigred 4.0K May 20 09:33 etc
# Linux 使用者的家目錄
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 home
drwxr-sr-x  7 bigred bigred 4.0K May 20 09:33 lib
drwxr-sr-x  5 bigred bigred 4.0K May 20 09:33 media
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 mnt
# option，放Linux系統不是內建的套件，也就是原生不在Linux系統的應用
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 opt
# Process，放Linux系統所有的Process資訊（PID...等）
dr-xr-sr-x  2 bigred bigred 4.0K Aug  5  2021 proc
# 他是root帳號的家目錄
drwx--S---  2 bigred bigred 4.0K Aug  5  2021 root
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 run
drwxr-sr-x  2 bigred bigred 4.0K May 20 09:33 sbin
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 srv
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 sys
drwxr-sr-t  2 bigred bigred 4.0K Aug  5  2021 tmp
drwxr-sr-x  7 bigred bigred 4.0K May 20 09:33 usr
drwxr-sr-x 12 bigred bigred 4.0K May 20 09:33 var
```


```
$ sudo chroot rootfs/ /bin/sh
root@alp:/$ whoami
root
root@alp:/$ ls -al /proc
total 8
dr-xr-sr-x    2 1000     1000          4096 Aug  5  2021 .
drwxr-sr-x   19 1000     1000          4096 May 20 01:33 ..
root@alp:/$ mkdir /proc/hi
root@alp:/$ ls -al /proc
total 12
dr-xr-sr-x    3 1000     1000          4096 May 20 02:16 .
drwxr-sr-x   19 1000     1000          4096 May 20 01:33 ..
drwxr-sr-x    2 root     1000          4096 May 20 02:16 hi
```

兩個 `/proc` 不一樣

```
$ ls -al /proc
total 4
dr-xr-xr-x 177 root   root      0 May 17 14:15 .
drwxr-xr-x  23 root   root   4096 May 16 17:53 ..
dr-xr-xr-x   9 root   root      0 May 17 14:15 1
dr-xr-xr-x   9 root   root      0 May 17 14:15 11
dr-xr-xr-x   9 root   root      0 May 17 14:15 12
dr-xr-xr-x   9 root   root      0 May 17 14:15 13
dr-xr-xr-x   9 root   root      0 May 17 14:15 14
dr-xr-xr-x   9 root   root      0 May 17 14:15 15
dr-xr-xr-x   9 root   root      0 May 17 14:15 16
dr-xr-xr-x   9 root   root      0 May 17 14:15 17
dr-xr-xr-x   9 root   root      0 May 17 14:15 179
dr-xr-xr-x   9 root   root      0 May 17 14:15 18
dr-xr-xr-x   9 root   root      0 May 17 14:15 1867
dr-xr-xr-x   9 root   root      0 May 17 14:15 1868
dr-xr-xr-x   9 root   root      0 May 17 14:15 1873
...
```
> 檔案名稱的數字就是 PID


```
$ sudo chroot rootfs /bin/sh
root@alp:/$ rm -r /bin/*

root@alp:/$ ls -al /bin
/bin/sh: ls: not found

root@alp:/$ exit

$ ls -al rootfs/bin
total 8
drwxr-sr-x  2 bigred bigred 4096 May 20 10:21 .
drwxr-sr-x 19 bigred bigred 4096 May 20 09:33 ..
```

將 Process 的執行者權限屬性設為 `bigred:bigred`

```
$ sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh
@alp:/$ whoami
whoami: unknown uid 1000
```

> 出現錯誤提示 `unknown uid 1000`，因為 `~/c3po/rootfs/etc/passwd` 沒有 bigred 資料

證明 `/etc/passwd` 沒有 bigred 的使用者資料
```
@alp:/$ cat /etc/passwd | tail -n 5
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
```



---

## 探索 Overlay2

- Overlay2 堆疊型檔案系統
只有上下兩層，overlay --> 覆蓋的意思
- 提問：目前使用過多少個檔案系統？
Windows 的檔案系統 ntfs
Linux 的檔案系統 ext4
Hadoop 的檔案系統 HDFS
Mac OS 的檔案系統 APFS
USB 的檔案系統 fat32
- 只要是檔案系統一定會有 Meta Data 和 Data

### mount 實作範例

```bash=
$ mkdir -p m/{work,a,b}
$ sudo mount --bind m/a m/work
$ mkdir m/work/haha
$ ls m/a
haha
$ ls m/work/
haha
$ sudo umount m/work
$ ls m/work/
$ ls m/a
haha
```

- 由以上結果得知，當 `m/a` 目錄掛載到 `m/work` 目錄上，我們在 `m/work` 上面建目錄時，實際上是建在 `m/a` 目錄上面


### 準備 Overlay2 檔案系統目錄

![](https://i.imgur.com/QfNgHNg.png)


- Overlay2 的檔案系統分為兩層 lower 目錄和 upper 目錄，這兩個目錄之間的關係是堆疊，也就是說 upper 目錄會在 lower 目錄之上，會用 linux 的命令 `mount` 掛載到一個目錄（merged），當 lower 和 upper 這兩層目錄有同一個檔案時，upper 目錄的檔案會覆蓋掉 lower 那層目錄的檔案。
- 操作都是在 merged 這層目錄執行，底層作業的就是 overlay2

#### 實作

```
$ cd ~;mkdir -p ov2/{upper,lower,merged,work}; cd ov2

$ echo "I'm from lower!" > lower/in_lower.txt

$ echo "I'm from upper!" > upper/in_upper.txt

$ echo "I'm from lower!" > lower/in_both.txt

$ echo "I'm from upper!" > upper/in_both.txt

$ tree
.
├── lower
│   ├── in_both.txt
│   └── in_lower.txt
├── merged
├── upper
│   ├── in_both.txt
│   └── in_upper.txt
└── work

4 directories, 4 files
```

- `work dir` 為 overlay2 檔案系統運作的工作目錄


### 建立與使用 Overlay2 檔案系統


```bash!
$ sudo mount -t overlay myov -o lowerdir=/home/bigred/ov2/lower,upperdir=/home/bigred/ov2/upper,workdir=/home/bigred/ov2/work  /home/bigred/ov2/merged
```

- `-t`，type，意思就是說要掛載到哪個檔案系統
- `myov` 是 mount 的名稱, 可由 執行 mount 得知
- `-o`，option，參數設定，lower 的目錄區在哪，upper 的目錄區在哪，work（工作目錄）的目錄放哪，
- `/home/bigred/ov2/merged`，最後掛載的目錄區


檢查

```
$ tree
.
├── lower
│   ├── in_both.txt
│   └── in_lower.txt
├── merged
│   ├── in_both.txt
│   ├── in_lower.txt
│   └── in_upper.txt
├── upper
│   ├── in_both.txt
│   └── in_upper.txt
└── work
    └── work [error opening dir]

5 directories, 7 files

```


證明 merged 這層目錄裡，會看到上層 upper 這層目錄的檔案內容

```bash!
$ cat merged/in_both.txt
I'm from upper!
```


在 merged 這層新增檔案

```!
$ echo 'new file' > merged/new_file

$ tree
.
├── lower
│   ├── in_both.txt
│   └── in_lower.txt
├── merged
│   ├── in_both.txt
│   ├── in_lower.txt
│   ├── in_upper.txt
│   └── new_file
├── upper
│   ├── in_both.txt
│   ├── in_upper.txt
│   └── new_file             ## 實際上new_file這個檔案，新增在upper這層目錄區
└── work
    └── work [error opening dir]

5 directories, 9 files
```


---


#### 小結論

在Overlay2檔案系統中，
- lower目錄區
<font color=red> **read only**，代表只能讀取不可被修改 </font>
- upper目錄區
<font color=red> **read write**，可以被讀取和修改 </font>
- merged這層目錄
<font color=red> **顯示最後的結果** </font>


**真正的操作都是在底下 upper 和 lower 這兩層目錄操作，當我們在 merged 這層目錄新增檔案和目錄時，只會在 upper 這層新增，因為 lower 只能讀取不可修改。**


---

#### unmount

```bash!
$ sudo umount merged/

$ ls -al merged/
total 8
drwxr-sr-x 2 bigred bigred 4096 May 20 11:21 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
```


---


### whiteout 操作範例

- whiteout ( 立可白 ) 在 merged 這層目錄，刪除重複的檔案名稱

刪除 merged 這層目錄的 in_both.txt

```bash!
$ rm merged/in_both.txt

$ ls -al merged/
total 20
drwxr-sr-x 1 bigred bigred 4096 May 20 13:11 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_lower.txt
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_upper.txt
-rw-r--r-- 1 bigred bigred    9 May 20 11:45 new_file
```

發現 upper 這層目錄的 in_both.txt 竟然還在！？

```bash!
$ ls -al upper
total 16
drwxr-sr-x 2 bigred bigred 4096 May 20 13:11 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
c--------- 2 root   root   0, 0 May 20 13:11 in_both.txt
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_upper.txt
-rw-r--r-- 1 bigred bigred    9 May 20 11:45 new_file
```

lower 這層在是正常的

```bash!
$ ls -al lower/
total 16
drwxr-sr-x 2 bigred bigred 4096 May 20 11:22 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_both.txt
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_lower.txt
```

**解答：**

- <font color=red>**當上下兩層有相同的檔案名稱時，刪除 merged 裡面的檔案後，會在 upper 這層目錄產生一個標籤檔，實際上檔案裡面沒有內容，會產生標籤檔的原因是因為 lower 這層的檔案還在，用標籤檔來遮住 lower 那層的目錄，以此來達成刪除的動作。**</font>
- 如果檔案只存在在 upper 目錄，我們去刪除它，檔案會直接被刪除，如果檔案只存在在 lower 目錄，則一樣產生一個標籤檔。

```bash!
# c 事實上是一個裝置檔
$ ls -al upper/ | grep in_both.txt
c--------- 2 root   root   0, 0 May 20 13:11 in_both.txt

$ cat upper/in_both.txt
cat: upper/in_both.txt: Permission denied

$ sudo cat upper/in_both.txt
cat: upper/in_both.txt: No such device or address
```

修改 upper 那層的 in_both.txt

```bash!
$ echo "back" > merged/in_both.txt
```

檢查

```bash!
$ ls -l upper/
total 12
-rw-r--r-- 1 bigred bigred  5 May 20 13:28 in_both.txt
-rw-r--r-- 1 bigred bigred 16 May 20 11:22 in_upper.txt
-rw-r--r-- 1 bigred bigred  9 May 20 11:45 new_file
```

---


### copy_up or copy on write

在 merged 這層更改只在 lower 這層存在的檔案內容

```bash!
$ ls -al upper
total 20
drwxr-sr-x 2 bigred bigred 4096 May 20 13:28 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
-rw-r--r-- 1 bigred bigred    5 May 20 13:28 in_both.txt
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_upper.txt
-rw-r--r-- 1 bigred bigred    9 May 20 11:45 new_file

$ echo "go or no go" >> merged/in_lower.txt
```

實際上會先在 lower 目錄區的這個檔案拷貝一份到 uppper 這層目錄，再進行修改

```bash!
$ cat upper/in_lower.txt
I m from lower!
go or no go

$ ls -al upper
total 24
drwxr-sr-x 2 bigred bigred 4096 May 20 13:28 .
drwxr-sr-x 6 bigred bigred 4096 May 20 11:21 ..
-rw-r--r-- 1 bigred bigred    5 May 20 13:28 in_both.txt
-rw-r--r-- 1 bigred bigred   28 May 20 13:35 in_lower.txt
-rw-r--r-- 1 bigred bigred   16 May 20 11:22 in_upper.txt
-rw-r--r-- 1 bigred bigred    9 May 20 11:45 new_file
```


- <font color=red>**在 Container 技術裡面，只能局部的這樣修改資料，如果大量這樣修改會造成 Container 掛掉。**</font>
- <font color=red>**如果需高頻率且大量修改資料，可以透過 `by pass overlay2` 來使用 Linux ext4 的檔案系統。**</font>


### overlay2 + chroot

將這個目錄區的內容作為 overlay2 的 lower 目錄區

```bash!
$ dir c3po/rootfs/
total 76K
drwxr-sr-x 19 bigred bigred 4.0K May 20 10:30 .
drwxr-sr-x  4 bigred bigred 4.0K May 20 11:21 ..
drwxr-sr-x  2 bigred bigred 4.0K May 20 10:30 bin
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 dev
drwxr-sr-x 15 bigred bigred 4.0K May 20 10:30 etc
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 home
drwxr-sr-x  7 bigred bigred 4.0K May 20 10:30 lib
drwxr-sr-x  5 bigred bigred 4.0K May 20 10:30 media
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 mnt
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 opt
dr-xr-sr-x  2 bigred bigred 4.0K Aug  5  2021 proc
drwx--S---  2 bigred bigred 4.0K May 20 10:30 root
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 run
drwxr-sr-x  2 bigred bigred 4.0K May 20 10:30 sbin
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 srv
drwxr-sr-x  2 bigred bigred 4.0K Aug  5  2021 sys
drwxr-sr-t  2 bigred bigred 4.0K Aug  5  2021 tmp
drwxr-sr-x  7 bigred bigred 4.0K May 20 10:30 usr
drwxr-sr-x 12 bigred bigred 4.0K May 20 10:30 var
```

命令

```bash!
$ sudo mount -t overlay myov -o lowerdir=/home/bigred/c3po/rootfs,upperdir=/home/bigred/ov2/upper,workdir=/home/bigred/ov2/work  /home/bigred/ov2/merged

$ sudo chroot ov2/merged/ sh

root@alp:/$ ls -al
total 92
drwxr-sr-x    1 1000     1000          4096 May 20 05:58 .
drwxr-sr-x    1 1000     1000          4096 May 20 05:58 ..
drwxr-sr-x    2 1000     1000          4096 May 20 02:30 bin
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 dev
drwxr-sr-x   15 1000     1000          4096 May 20 02:30 etc
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 home
-rw-r--r--    1 1000     1000            28 May 20 05:35 in_lower.txt
-rw-r--r--    1 1000     1000            16 May 20 03:22 in_upper.txt
drwxr-sr-x    7 1000     1000          4096 May 20 02:30 lib
drwxr-sr-x    5 1000     1000          4096 May 20 02:30 media
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 mnt
-rw-r--r--    1 1000     1000             9 May 20 03:45 new_file
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 opt
dr-xr-sr-x    2 1000     1000          4096 Aug  5  2021 proc
drwx--S---    1 1000     1000          4096 May 20 02:30 root
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 run
drwxr-sr-x    2 1000     1000          4096 May 20 02:30 sbin
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 srv
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 sys
drwxr-sr-t    2 1000     1000          4096 Aug  5  2021 tmp
drwxr-sr-x    7 1000     1000          4096 May 20 02:30 usr
drwxr-sr-x   12 1000     1000          4096 May 20 02:30 var

root@alp:/$ mkdir /opt/haha

root@alp:/$ ls -l /opt/
total 4
drwxr-sr-x    2 root     1000          4096 May 20 06:32 haha

root@alp:/$ exit

$ ls -al ov2/merged/opt/
total 12
drwxr-sr-x 1 bigred bigred 4096 May 20 14:32 .
drwxr-sr-x 1 bigred bigred 4096 May 20 13:58 ..
drwxr-sr-x 2 root   bigred 4096 May 20 14:32 haha
```

---

## **探索 CGroup**



## CGroup 應用架構


![](https://i.imgur.com/NNWUXgE.png)

Control group 的功能可以依據 process 的 PID，去控管
- cpu 他用第幾個核心的 cpu
- memory 記憶體使用量，超出則會報錯 ( out of memory )
- net_cls 使用網路的情形，例：每秒最多可以傳送多少量的資料
- 只要將要限制的 process 的 PID 加入 控制目錄裡的 `tasks` 這個檔案，process 就會被控制。



---

### **檢視與建立 CGroup**

:::spoiler CGroup 會透過目錄與設定項目檔案來管理

```
bigred@alp:~$ ls -al /sys/fs/cgroup/
total 0
drwxr-xr-x 16 root root 320 May 20 14:27 .
drwxr-xr-x  7 root root   0 May 20 14:27 ..
dr-xr-xr-x  2 root root   0 May 20 14:27 blkio
dr-xr-xr-x  2 root root   0 May 20 14:27 cpu
dr-xr-xr-x  2 root root   0 May 20 14:27 cpuacct
dr-xr-xr-x  2 root root   0 May 20 14:27 cpuset
dr-xr-xr-x  2 root root   0 May 20 14:27 devices
dr-xr-xr-x  2 root root   0 May 20 14:27 freezer
dr-xr-xr-x  2 root root   0 May 20 14:27 hugetlb
dr-xr-xr-x  2 root root   0 May 20 14:27 memory
dr-xr-xr-x  2 root root   0 May 20 14:27 net_cls
dr-xr-xr-x  2 root root   0 May 20 14:27 net_prio
dr-xr-xr-x  5 root root   0 May 20 14:27 openrc
dr-xr-xr-x  2 root root   0 May 20 14:27 perf_event
dr-xr-xr-x  2 root root   0 May 20 14:27 pids
dr-xr-xr-x  5 root root   0 May 20 14:27 unified
```
:::

:::spoiler 在 `/sys/fs/cgroup/memory/` 每一個檔案都是對記憶體的一種控制
```
bigred@alp:~$ ls -al /sys/fs/cgroup/memory/
total 0
dr-xr-xr-x  2 root root   0 May 20 14:27 .
drwxr-xr-x 16 root root 320 May 20 14:27 ..
-rw-r--r--  1 root root   0 May 20 15:28 cgroup.clone_children
--w--w--w-  1 root root   0 May 20 15:28 cgroup.event_control
-rw-r--r--  1 root root   0 May 20 15:28 cgroup.procs
-r--r--r--  1 root root   0 May 20 15:28 cgroup.sane_behavior
-rw-r--r--  1 root root   0 May 20 15:28 memory.failcnt
--w-------  1 root root   0 May 20 15:28 memory.force_empty
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.failcnt
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.limit_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.max_usage_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.tcp.failcnt
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.tcp.limit_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--  1 root root   0 May 20 15:28 memory.kmem.tcp.usage_in_bytes
-r--r--r--  1 root root   0 May 20 15:28 memory.kmem.usage_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.limit_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.max_usage_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.memsw.failcnt
-rw-r--r--  1 root root   0 May 20 15:28 memory.memsw.limit_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.memsw.max_usage_in_bytes
-r--r--r--  1 root root   0 May 20 15:28 memory.memsw.usage_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.move_charge_at_immigrate
-rw-r--r--  1 root root   0 May 20 15:28 memory.oom_control
----------  1 root root   0 May 20 15:28 memory.pressure_level
-rw-r--r--  1 root root   0 May 20 15:28 memory.soft_limit_in_bytes
-r--r--r--  1 root root   0 May 20 15:28 memory.stat
-rw-r--r--  1 root root   0 May 20 15:28 memory.swappiness
-r--r--r--  1 root root   0 May 20 15:28 memory.usage_in_bytes
-rw-r--r--  1 root root   0 May 20 15:28 memory.use_hierarchy
-rw-r--r--  1 root root   0 May 20 15:28 notify_on_release
-rw-r--r--  1 root root   0 May 20 15:28 release_agent
-rw-r--r--  1 root root   0 May 20 14:27 tasks
```
:::

當你在 `/sys/fs/cgroup/memory/` 目錄裡面新增一個目錄
```
bigred@alp:~$ sudo mkdir /sys/fs/cgroup/memory/demo
```

:::spoiler 你會發現剛剛的目錄裡面馬上會將 `/sys/fs/cgroup/memory/` 目錄裡面的所有檔案拷貝到 `demo` 目錄中
```
bigred@alp:~$ ls -al /sys/fs/cgroup/memory/demo
total 0
drwxr-xr-x 2 root root 0 May 20 15:31 .
dr-xr-xr-x 3 root root 0 May 20 14:27 ..
-rw-r--r-- 1 root root 0 May 20 15:31 cgroup.clone_children
--w--w--w- 1 root root 0 May 20 15:31 cgroup.event_control
-rw-r--r-- 1 root root 0 May 20 15:31 cgroup.procs
-rw-r--r-- 1 root root 0 May 20 15:31 memory.failcnt
--w------- 1 root root 0 May 20 15:31 memory.force_empty
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.failcnt
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.limit_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.max_usage_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.tcp.failcnt
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.tcp.limit_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- 1 root root 0 May 20 15:31 memory.kmem.tcp.usage_in_bytes
-r--r--r-- 1 root root 0 May 20 15:31 memory.kmem.usage_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.limit_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.max_usage_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.memsw.failcnt
-rw-r--r-- 1 root root 0 May 20 15:31 memory.memsw.limit_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.memsw.max_usage_in_bytes
-r--r--r-- 1 root root 0 May 20 15:31 memory.memsw.usage_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.move_charge_at_immigrate
-rw-r--r-- 1 root root 0 May 20 15:31 memory.oom_control
---------- 1 root root 0 May 20 15:31 memory.pressure_level
-rw-r--r-- 1 root root 0 May 20 15:31 memory.soft_limit_in_bytes
-r--r--r-- 1 root root 0 May 20 15:31 memory.stat
-rw-r--r-- 1 root root 0 May 20 15:31 memory.swappiness
-r--r--r-- 1 root root 0 May 20 15:31 memory.usage_in_bytes
-rw-r--r-- 1 root root 0 May 20 15:31 memory.use_hierarchy
-rw-r--r-- 1 root root 0 May 20 15:31 notify_on_release
-rw-r--r-- 1 root root 0 May 20 15:31 tasks
```
:::

> 在 memory 目錄裡面可以新增任意數量的目錄，來做控管


:::spoiler 編輯一支程式
```
$ nano memlimit.sh 
#!/bin/bash

## 指定目錄/sys/fs/cgroup/memory/demo不存在，則建立
[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo

## 
echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null

## echo $$ 代表當前process的PID
echo $$ | sudo tee /sys/fs/cgroup/memory/demo/tasks &>/dev/null

## The line, read from /dev/zero which outputs only null bytes and no newlines, will be infinitely long, but is limited by head to BYTES bytes, thus tail will use only that much memory. 
cat /dev/zero | head -c $1 | tail
```

- `cat /dev/zero | head -c $1 | tail` ，當我們 `cat` 零號裝置 (`/dev/zero`) ，會一直往螢幕上輸出 ASCll:0 這個 NULL 值，但因為後面有接 `|` pipe ，所以會把前面命令執行的結果，輸入給後面的 `head` 命令，`head -c` 會選擇只要幾 mb 的資料，後面再接一個 `|` pipe ，這時候要利用 `tail` 這個命令的特性，他會一直將資料存在記憶體中，一直等到有換行符號(ASCll:27)，才會將記憶體空間釋放，所以一但在記憶體霸佔的資料區塊大小超過我們透過 `CGROUP` 限制的 95m ，這支程序就會被終止。
:::


:::spoiler 為何會出錯？
```
bigred@alp:~$ sudo echo "100000000" > /sys/fs/cgroup/memory/demo/memory.limit_in_bytes
-bash: /sys/fs/cgroup/memory/demo/memory.limit_in_bytes: Permission denied
```

因為導向(>) 也是 bash 命令，這個範例只有 echo 指令有用到 root 身分執行

```
bigred@alp:~$ chmod +x memlimit.sh
bigred@alp:~$ ./memlimit.sh 82m
```

:::

當程式使用的記憶體超過我們設定的上限值，就會被 kill 掉
```
bigred@alp:~$ ./memlimit.sh 100m
./memlimit.sh: line 7:  6638 Broken pipe             cat /dev/zero
      6639                       | head -c $1
      6640 Killed                  | tail
```


---

#### 自定 cpu 控制群組

CPU總量預設設定1024
```
bigred@alp:~$ cat /sys/fs/cgroup/cpu/cpu.shares
1024
```

以 root 來建立 low 控制子群組，low 繼承了 cpu controller 的所有屬性

```
bigred@alp:~$ sudo mkdir /sys/fs/cgroup/cpu/low
bigred@alp:~$ sudo ls -al /sys/fs/cgroup/cpu/low
total 0
drwxr-xr-x 2 root root 0 May 23 09:55 .
dr-xr-xr-x 3 root root 0 May 23 08:54 ..
-rw-r--r-- 1 root root 0 May 23 09:55 cgroup.clone_children
-rw-r--r-- 1 root root 0 May 23 09:55 cgroup.procs
-rw-r--r-- 1 root root 0 May 23 09:55 cpu.cfs_burst_us
-rw-r--r-- 1 root root 0 May 23 09:55 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 May 23 09:55 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 May 23 09:55 cpu.idle
-rw-r--r-- 1 root root 0 May 23 09:55 cpu.shares
-r--r--r-- 1 root root 0 May 23 09:55 cpu.stat
-rw-r--r-- 1 root root 0 May 23 09:55 notify_on_release
-rw-r--r-- 1 root root 0 May 23 09:55 tasks
```


:::spoiler 設定 low 控制子群組權限, 將權限開給 bigred 使用者
```
$ sudo chown bigred:root -R /sys/fs/cgroup/cpu/low
```
:::


:::spoiler 將 low 控制子群組分配比率設成 512
```
bigred@alp:~$ echo 512 | sudo tee /sys/fs/cgroup/cpu/low/cpu.shares
512
```
:::


:::spoiler 建立 cpu 的 high 控制子群組
```
$ sudo mkdir /sys/fs/cgroup/cpu/high
```

:::


:::spoiler 設定 high 控制子群組權限
```
bigred@alp:~$ sudo chown bigred:root -R /sys/fs/cgroup/cpu/high
bigred@alp:~$ echo 2048 | sudo tee /sys/fs/cgroup/cpu/high/cpu.shares
2048
```

> low = 512/(512+2048) = 20%, 
> high = 2048/(2048+512)= 80%
:::

---

**自定 cpuset 控制群組**

```
bigred@alp:~$ sudo mkdir /sys/fs/cgroup/cpuset/first
bigred@alp:~$ sudo chown bigred:root -R /sys/fs/cgroup/cpuset/first
## 事實上可以通過這串指令檢查電腦的CPU有幾顆核心
## 有二個 cpu core, 分別是 0 和 1
bigred@alp:~$ cat /sys/fs/cgroup/cpuset/cpuset.cpus
0-1


bigred@alp:~$ cat /sys/fs/cgroup/cpuset/first/cpuset.cpus

## 0，代表的是Core 0，我們可以很精準控制process使用哪個核心
bigred@alp:~$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.cpus
0
```



---

### **建立測試執行檔**

把yes的命令推到背景執行
```
bigred@alp:~$ yes &>/dev/null &
[1] 8359
bigred@alp:~$ yes &>/dev/null &
[2] 8362
```

將測試程式的 PID 加入到 first 控制子群組

```
## 關閉CPU直接存取記憶體的功能（並不是所有PC都有這種功能，所以都先關閉）
bigred@alp:~$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.mems
0

## 指定8359用Core 0
$ echo 8359 | sudo tee /sys/fs/cgroup/cpuset/first/tasks

## 指定8362用Core 0
$ echo 8362 | sudo tee  /sys/fs/cgroup/cpuset/first/tasks

## 檢查
bigred@alp:~$ cat /sys/fs/cgroup/cpuset/first/tasks
8359
8362
```

檢視 CPU 執行分配比率
```
## 按1，可以選看核心。按Q脫離。
bigred@alp:~$ top
top - 10:26:00 up  1:31,  0 users,  load average: 2.01, 1.65, 0.85
Tasks: 119 total,   3 running, 116 sleeping,   0 stopped,   0 zombie
%Cpu0  :  68.9/31.1  100[||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||]
%Cpu1  :   0.0/0.0     0[                                                                                                    ]
GiB Mem :  4.9/1.9      [                                                                                                    ]
GiB Swap:  0.0/0.0      [

   PID USER      PR  NI    VIRT    RES  %CPU  %MEM     TIME+ S COMMAND
  8359 bigred    20   0    2.2m   0.6m  49.3   0.0   7:08.15 R `- yes
  8362 bigred    20   0    2.2m   0.6m  49.3   0.0   7:06.62 R `- yes
  8819 bigred    20   0    2.3m   1.6m   0.0   0.1   0:00.18 R `- top
```

上方可看到二個程式 (yes_high, yes_low)，大約佔 cpu 50% 使用率



接著我們將 yes_low 加入到 low 控制群組
```
$ echo 8359 | sudo tee /sys/fs/cgroup/cpu/low/tasks
8359
```

將 yes_high 加入到 high 控制群組
```
$ echo 8362 | sudo tee  /sys/fs/cgroup/cpu/high/tasks
8362
```

檢查
```
bigred@alp:~$ top
top - 10:31:05 up  1:36,  0 users,  load average: 2.37, 2.00, 1.23
Tasks: 119 total,   3 running, 116 sleeping,   0 stopped,   0 zombie
%Cpu0  :  70.9/29.1  100[||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||]
%Cpu1  :   0.0/0.7     1[|                                                                                                   ]
GiB Mem :  4.9/1.9      [                                                                                                    ]
GiB Swap:  0.0/0.0      [

   PID USER      PR  NI    VIRT    RES  %CPU  %MEM     TIME+ S COMMAND
  8359 bigred    20   0    2.2m   0.6m  20.7   0.0   8:58.38 R `- yes
  8362 bigred    20   0    2.2m   0.6m  80.7   0.0   9:47.10 R `- yes
  9169 bigred    20   0    2.3m   1.5m   0.0   0.1   0:00.05 R `- top
```

如果執行環境太複雜，要確定精準的數字會非常困難，如果執行環境很簡單，就能做比較精確的資源掌控


### 安裝與使用 CGroup 命令套件

```bash!
$ sudo apk add cgroup-tools --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/community/ --allow-untrusted

# 自定 cpu 控制群組
$ sudo cgcreate -g cpu:/low; sudo cgcreate -g cpu:/high
$ sudo cgset -r cpu.shares=512 low; sudo cgset -r cpu.shares=2048 high
```

![](https://i.imgur.com/a0rgC13.png)

- 最上層的 `/` 為 CPU，第二層分別是 3 個 CPU 的控制項目，每個控制項目可以用到多少 CPU 的時間，像 `/user.slice` 可以用多少的算法為 `1024/1024+1024+1024 = 33 %`，這個控制項目又有 3 個使用者，分別可以使用 CPU 時間為 `768`, `512`, `2048` ，以 `user1` 可以使用多少 CPU 為例，`768/768+512+2048 = 23.1%`，所以 `user1` ，只能用 `33%` 裡面的 `23.1%` ，實際上約等於可以用到 `7.6%` 的 CPU 時間


```bash!
# 自定 cpuset 控制群組
$ sudo cgcreate -g cpuset:/first
$ sudo cgset -r cpuset.cpus=0 first

# 設定本機沒有 多顆實體 CPU 架構 (NUMA)
$ sudo cgset -r cpuset.mems=0 first


$ sudo cgexec -g cpu:high -g cpuset:first yes &>/dev/null &

$ sudo cgexec -g cpu:low -g cpuset:first yes &>/dev/null &

```

---

## chroot, cgroup, overlay2 整合應用

### **設定 Lower 目錄 = 堆疊多層目錄**
```
bigred@alp:~$ sudo mount -t overlay overlay -o lowerdir=/home/bigred/ov2/lower:/home/bigred/c3po/rootfs,upperdir=/home/bigred/ov2/upper,workdir=/h
ome/bigred/ov2/work  /home/bigred/ov2/merged

## lowerdir=/home/bigred/ov2/lower:/home/bigred/c3po/rootfs
意思是說lowerdir 可以由多個目錄堆疊起來，後者蓋前者，最後的結果是read only

## upperdir 只能一個目錄

## workdir 存放暫時的資訊

## merged 最後掛載的目錄
```

檢查，確認Overlay2啟動
```
bigred@alp:~$ ls -al ov2/merged/
total 96
drwxr-sr-x  1 bigred bigred 4096 May 20 13:58 .
drwxr-sr-x  6 bigred bigred 4096 May 20 11:21 ..
drwxr-sr-x  2 bigred bigred 4096 May 20 10:30 bin
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 dev
drwxr-sr-x 15 bigred bigred 4096 May 20 10:30 etc
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 home
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_both.txt
-rw-r--r--  1 bigred bigred   28 May 20 13:35 in_lower.txt
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_upper.txt
drwxr-sr-x  7 bigred bigred 4096 May 20 10:30 lib
drwxr-sr-x  5 bigred bigred 4096 May 20 10:30 media
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 mnt
-rw-r--r--  1 bigred bigred    9 May 20 11:45 new_file
drwxr-sr-x  1 bigred bigred 4096 May 20 14:32 opt
dr-xr-sr-x  2 bigred bigred 4096 Aug  5  2021 proc
drwx--S---  1 bigred bigred 4096 May 20 10:30 root
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 run
drwxr-sr-x  2 bigred bigred 4096 May 20 10:30 sbin
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 srv
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 sys
drwxr-sr-t  2 bigred bigred 4096 Aug  5  2021 tmp
drwxr-sr-x  7 bigred bigred 4096 May 20 10:30 usr
drwxr-sr-x 12 bigred bigred 4096 May 20 10:30 var
```

執行Change root 
```
bigred@alp:~$ cd ~/ov2; sudo chroot merged /bin/sh
root@alp:/$ ls -al
total 96
drwxr-sr-x    1 1000     1000          4096 May 20 05:58 .
drwxr-sr-x    1 1000     1000          4096 May 20 05:58 ..
drwxr-sr-x    2 1000     1000          4096 May 20 02:30 bin
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 dev
drwxr-sr-x   15 1000     1000          4096 May 20 02:30 etc
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 home
-rw-r--r--    1 1000     1000            16 May 20 03:22 in_both.txt
-rw-r--r--    1 1000     1000            28 May 20 05:35 in_lower.txt
-rw-r--r--    1 1000     1000            16 May 20 03:22 in_upper.txt
drwxr-sr-x    7 1000     1000          4096 May 20 02:30 lib
drwxr-sr-x    5 1000     1000          4096 May 20 02:30 media
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 mnt
-rw-r--r--    1 1000     1000             9 May 20 03:45 new_file
drwxr-sr-x    1 1000     1000          4096 May 20 06:32 opt
dr-xr-sr-x    2 1000     1000          4096 Aug  5  2021 proc
drwx--S---    1 1000     1000          4096 May 20 02:30 root
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 run
drwxr-sr-x    2 1000     1000          4096 May 20 02:30 sbin
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 srv
drwxr-sr-x    2 1000     1000          4096 Aug  5  2021 sys
drwxr-sr-t    2 1000     1000          4096 Aug  5  2021 tmp
drwxr-sr-x    7 1000     1000          4096 May 20 02:30 usr
drwxr-sr-x   12 1000     1000          4096 May 20 02:30 var
```

離開sh的同時，會關閉chroot

```
root@alp:/$ exit
bigred@alp:~/ov2$
```

關閉Overlay2
```
bigred@alp:~/ov2$ sudo umount merged

bigred@alp:~/ov2$ ls -l merged/
total 0
```

### **執行 Chroot 與使用 Overlay2 檔案系統**

拷貝檔案
```
bigred@alp:~/ov2$ sudo cp -rP ../c3po/rootfs/* lower/
bigred@alp:~/ov2$ ls -l lower/
total 76
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 bin
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 dev
drwxr-sr-x 15 root   bigred 4096 May 23 11:25 etc
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 home
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_both.txt
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_lower.txt
drwxr-sr-x  7 root   bigred 4096 May 23 11:25 lib
drwxr-sr-x  5 root   bigred 4096 May 23 11:25 media
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 mnt
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 opt
dr-xr-sr-x  2 root   bigred 4096 May 23 11:25 proc
drwx--S---  2 root   bigred 4096 May 23 11:25 root
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 run
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 sbin
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 srv
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 sys
drwxr-sr-t  2 root   bigred 4096 May 23 11:25 tmp
drwxr-sr-x  7 root   bigred 4096 May 23 11:25 usr
drwxr-sr-x 12 root   bigred 4096 May 23 11:25 var
```

:::spoiler cp命令補充
```
$ cp --help
  -P, --no-dereference     never follow symbolic links in SOURCE
  -r, --recursive          copy directories recursively
```
:::


掛載Overlay2到mergeddir
```
$ sudo mount -t overlay overlay -o lowerdir=/home/bigred/ov2/lower,upperdir=/home/bigred/ov2/upper,workdir=/home/bigred/ov2/work  /home/bigred/ov2/merged
```

檢查有無掛載成功
```
bigred@alp:~/ov2$ ls -l merged/
total 88
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 bin
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 dev
drwxr-sr-x 15 root   bigred 4096 May 23 11:25 etc
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 home
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_both.txt
-rw-r--r--  1 bigred bigred   28 May 20 13:35 in_lower.txt
-rw-r--r--  1 bigred bigred   16 May 20 11:22 in_upper.txt
drwxr-sr-x  7 root   bigred 4096 May 23 11:25 lib
drwxr-sr-x  5 root   bigred 4096 May 23 11:25 media
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 mnt
-rw-r--r--  1 bigred bigred    9 May 20 11:45 new_file
drwxr-sr-x  1 bigred bigred 4096 May 20 14:32 opt
dr-xr-sr-x  2 root   bigred 4096 May 23 11:25 proc
drwx--S---  1 bigred bigred 4096 May 20 10:30 root
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 run
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 sbin
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 srv
drwxr-sr-x  2 root   bigred 4096 May 23 11:25 sys
drwxr-sr-t  2 root   bigred 4096 May 23 11:25 tmp
drwxr-sr-x  7 root   bigred 4096 May 23 11:25 usr
drwxr-sr-x 12 root   bigred 4096 May 23 11:25 var
```

執行chroot
```
bigred@alp:~/ov2$ sudo chroot merged/ /bin/sh
```

看當前process的pid，也就是sh的pid
```
root@alp:/$ echo $$
12883
```

可再開一個CMD，檢查
```
bigred@alp:~$ ps aux | grep 12883
root      12883  0.0  0.0   1696  1168 pts/0    S+   11:31   0:00 /bin/sh
```

再回到原本的CMD，建立 /dev/zero 裝置, 因新的檔案系統, 沒有 /dev/zero 裝置
```
root@alp:/$ mknod -m 0666 /dev/zero c 1 5
```

設定 CGroup 記憶體限制

開啟一個新的 CMD 視窗, 執行以下命令
```
$ ssh bigred@<CTN.ALP.Docker IP>

## 在/sys/fs/cgroup/memory/下建立demo目錄
$ sudo mkdir /sys/fs/cgroup/memory/demo


$ echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes
100000000

$ echo 12883 | sudo tee /sys/fs/cgroup/memory/demo/tasks
12883

切換到 chroot 的 CMD 視窗
root@alp:/$ cat /dev/zero | head -c 80m | tail

root@alp:/$ cat /dev/zero | head -c 100m | tail
Killed
root@alp:/$ exit

$ sudo umount merged
```



---

## 探索 Namespace

- Namespace
	- Mount (mnt)
	- Process ID (pid)
	- Network (net)
	- Interprocess Communication (ipc)
	- UTS (UNIX Time-Sharing)
		- 讓process有獨立的電腦名稱
	- User ID (user)
	- Control group (cgroup) Namespace
	- Time Namespace



---

### **檢視 ALP 虛擬主機的 Namespace**

```
bigred@alp:~/ov2$ ls -l /proc/self/ns/
total 0
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 net -> 'net:[4026531840]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 time -> 'time:[4026531834]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 user -> 'user:[4026531837]'
lrwxrwxrwx 1 bigred bigred 0 May 23 13:03 uts -> 'uts:[4026531838]'
```


---

### **Isolating the Hostname**

執行 Namespace 功能的命令 `unshare`
```
$ sudo unshare --uts sh
```

```bash!
## 檢查主機名稱
root@alp:/home/bigred/ov2$ hostname
alp
## 更改主機名稱
root@alp:/home/bigred/ov2$ hostname ssn763
## 檢查主機名稱
root@ssn763:/home/bigred/ov2$ hostname
ssn763
## 離開
root@ssn763:/home/bigred/ov2$ exit
## 主機名稱變回來alp
bigred@alp:~/ov2$
```

### Isolating Process IDs


- **只要porcess有啟動uts的隔離技術，主機名稱就可以擁有獨立的電腦名稱，但process結束，隔離技術就會消失，變回原來的設定。**
- 在 linux 的 ps 命令顯示的資料，資料一定從 `/proc` 目錄來，那這個目錄裡面一定放 process 的資訊

```
$ sudo unshare --pid --fork sh
以pid進行隔離，可以讓環境內有獨立的Process資訊
>從 echo $$ 可以得知目前的PID為1

但因為沒有chroot，所以ps命令仍是讀取Host OS的/proc
```

- `/proc` 裡面的資訊，實際上是儲存在記憶體中，裡面記錄著現在作業系統中所有 process 的資訊。
- procfs 是 行程 檔案系統 (process file system) 的縮寫，包含一個偽檔案系統（啟動時動態生成的檔案系統），用於通過核心存取行程資訊。這個檔案系統通常被掛載到 `/proc` 目錄。由於 `/proc` 不是一個真正的檔案系統，它也就不占用儲存空間，只是占用有限的記憶體。
- proc 還有另一個功能，我們利用它實現 Linux 核心空間與使用者空間 的通訊。在 proc 檔案系統中，我們可以將對虛擬檔案的讀寫作為與核心中實體進行通訊的一種手段，但是與普通檔案不同的是，這些虛擬檔案的內容都是動態建立的。這個虛擬檔案系統會掛載在 /proc 目錄, 而 ps 命令一定會讀取 /proc 目錄, 來顯示目前系統所有 process 資訊

![](https://i.imgur.com/96XQhYj.png)

雖然啟用 PID Namespace, ps 命令還是會看到 Host (alp) 的所有 Process 資訊, 這是因爲 ps 命令會讀取 ALP 的 /proc 目錄資訊

```
$ sudo unshare --pid --fork sh
root@alp:/home/bigred$ ps
   PID TTY          TIME CMD
  4418 pts/0    00:00:00 unshare
  4419 pts/0    00:00:00 sh
  4421 pts/0    00:00:00 ps
```



```
$ sudo unshare --pid --fork --mount-proc sh
```

**--mount-proc   這參數, 會將 sh 的 PID Namespace, 掛載到 ALP   的 /proc   目錄, 由以下 mount 命令, 得知 /proc 目錄被掛載二次**




---

## Linux Network Namespace

![](https://i.imgur.com/2zocMXY.png)

- Network Namespace 可以隔離 process 的網路空間出來。
- Default Network Namespace 是系統預設的網路隔離空間
- `ifconfig` 只會看到 Default 這個 Network Namespace 網路空間的網路卡的網路資訊
- **右邊有鎖頭的那個範圍，是我們自己可以創造的 Network Namespace（另外一個網路的隔離空間），裡面可以放我們自己所產生的虛擬網路卡，在 Default Namespace 裡，也可以產生虛擬網路卡，這兩個網路卡可以拉一條虛擬的網路線把它們連接起來，代表兩個 Network Namespace 的網路隔離空間可以互通有無，且虛擬網路卡和網路線是產生在記憶體裡面，所以除非實體電腦的主機板或記憶體壞掉，否則正常情況下是很穩定的。**



---

### 實作Network Namespace

再開啟一個新的命令提示字元 視窗, 執行以下命令 `$ ssh bigred@<ALP.Docker IP>`

`--net` 給 `sh` 這個 process 產生新的 Network Namespace
```
$ sudo unshare --pid --fork --mount-proc --net --uts sh
```

透過 `ip a` 可以查看 `sh` 這個 process 的 Network Namespace 

```
$ ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

> `lo` ，LOOPBACK ，提供自我檢查的網路卡


檢查 PID 的 Namespace 有無啟動，可以看到只有 1 和 2，代表有成功啟動！

```
root@alp:/home/bigred$ ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0   1684  1168 ?        S    21:17   0:00 sh
root          2  0.0  0.0   1708   836 ?        R+   21:17   0:00 ps aux
```

執行 `ifconfig` 空的原因，是因為它看的是這個 Process 的 Default Namespace，全新都沒有設定，所以空空的。

如果沒打`--net`，執行 `ifconfig -a` 的結果，會看到 Host 主機的
```
root@alp:/home/bigred$ ifconfig

root@alp:/home/bigred$ ifconfig -a
lo        Link encap:Local Loopback
          LOOPBACK  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
		  
```

回到原先 命令提示字元 視窗, 執行以下命令

看剛剛新產生的 Network Namespace

```
$ sudo lsns -t net
        NS TYPE NPROCS   PID USER    NETNSID NSFS COMMAND
4026532000 net     115     1 root unassigned      /sbin/init
4026532555 net       2  4582 root unassigned      unshare --pid --fork --mount-proc -
```
- `lsns = List Neamespace`
- `-t` ，項目，net (Network)
-  用法：`lsns [options] [<namespace>]`
-  `4026532000` 是 Default Namespace
-  `4026532555` 是 sh process 的 Namespace


```
$ sudo ip link add ve1 netns 1 type veth peer name ve2 netns 4582
```

- 新增 `ve1` 虛擬網路卡到 Default Namespace ( pid = 1 )
- 新增 `ve2` 虛擬網路卡到 ${shNPID} 的 Network Namespace ( pid = 4582 )
- 以上這兩張網路卡會透過 `ip link` 用一條網路線連接起來
- `veth` ，Virtual Ethernet Device
- `peer` ，一組的意思

將 ve1 這張網路卡啟動，因為內定網路卡沒有啟動

```bash!
$ sudo ip link set ve1 up
```

設定 ve1 網路卡的 ip 位址

```bash!
$ sudo ip addr add 192.168.1.100/24 dev ve1
```

---

### Isolating Process IDs and chroot

```
掛載記憶體的程序資訊
$ cd; dir r2d2/rootfs/proc
total 8.0K
drwxr-sr-x 2 root root 4.0K Apr 29 22:05 .
drwxr-sr-x 8 root root 4.0K Apr 29 22:05 ..


$ sudo unshare --pid --fork --mount-proc --uts -R r2d2/rootfs sh
@alp:/$ whoami
sh: whoami: not found
@alp:/$ ps 
sh: ps: not found
@alp:/$ ls -al /
sh: ls: not found
@alp:/$ exit


[註] --mount-proc 這個參數, 會自動 執行 "mount -t proc /proc /proc" 這行命令
```

:::spoiler unshare的簡易手冊
```
$ unshare --help
-p, --pid[=<file>]        unshare pid namespace
-f, --fork                fork before launching <program>
-u, --uts[=<file>]        unshare UTS namespace (hostname etc)
--mount-proc[=<dir>]      mount proc filesystem first (implies --mount)
-n, --net[=<file>]        unshare network namespace
-R, --root=<dir>          run the command with root directory set to <dir>
```
:::


---


### User Namespace

Container 被啟動時，是由 host 主機的 User 啟動的，但 Container 內部運作，可以有自己的 user 來執行程式

```bash!
# 需使用 sudo 執行以下命令, 才會成功
$ unshare --pid --fork --mount-proc --uts -R c3po/rootfs sh
unshare: unshare failed: Operation not permitted

# 沒使用 sudo 執行 unshare 命令, 必需要有 --user 參數才可執行, 以下命令 container 內定使用者為 nobody (id:65534), 執行 unshare 命令, 則是 bigred 帳
號
$ unshare --user whoami
nobody

$ unshare --user -c --pid --fork --mount-proc --uts -R ~/c3po/rootfs sh
@alp:// $ ls -ald /
drwxr-sr-x   20 1000     1000          4096 Aug 15 16:00 /
@alp:// $ exit

$ unshare --user -r --pid --fork --mount-proc --uts -R ~/c3po/rootfs sh
root@alp:// $ ls -ald /
drwxr-sr-x   20 root     root          4096 Aug 15 16:00 /
root@alp:// $ exit
```

- `unshare --user -c` ，map current user to itself (implies --user)
- `unshare --user -r` ，map current user to root (implies --user)



### 加入 Linux 的系統命令集 (busybox)

```
$ sudo cp /bin/busybox  r2d2/rootfs/bin

## 新增busybox所需的目錄檔
$ sudo mkdir -p ~/r2d2/rootfs/{sbin,usr/{bin,sbin}}

$ sudo unshare --pid --fork --mount-proc --uts -R r2d2/rootfs sh
# /bin/busybox --install -s

```

### Namespace & CGroup

/sys的目錄是存在在記憶體，所以重新開機後就會消失
```
bigred@alp:~$ cd ~/c3po/
bigred@alp:~/c3po$ sudo mkdir /sys/fs/cgroup/memory/demo
```

:::spoiler 關閉記憶體的一項功能，當電腦記憶體不夠時，會拿硬碟一部份的儲存空間作為記憶體，當電腦啟用這個功能時，如何識別？
你會看到硬碟存取的燈不斷閃爍，因為記憶體大量在使用它，記憶體不夠了，這時候會把我們的硬碟拿來當作記憶體的延伸（記憶體的後半部），如果硬碟不快、記憶體又不夠，電腦就會呈現昏迷狀態，每打一個按鍵、滑鼠點一下，整個系統就會大lag，所以我們不需要這個功能，現在記憶體通常都已關閉這個功能，寧願電腦跟我們說記憶體不夠（out of memory），也不要電腦失智/昏迷，慢到吐奶。
:::
```
$ echo "0" | sudo tee /sys/fs/cgroup/memory/demo/memory.swappiness
0
````

最前面的C是裝置檔
```
bigred@alp:~/c3po$ ls -al /dev/zero
crw-rw-rw- 1 root root 1, 5 May 24 09:48 /dev/zero
```
> 在我們Alpine虛擬電腦裡有零號裝置


在我們rootfs的目錄裡，/dev裡面是空的
```
bigred@alp:~/c3po$ ls -al rootfs/dev
total 8
drwxr-sr-x  2 bigred bigred 4096 Aug  5  2021 .
drwxr-sr-x 19 bigred bigred 4096 May 20 10:30 ..
```

將 Alpine 的 `/dev` 目錄掛載到 `rootfs/dev` 的目錄上 ( `ro` ，read only )
在 Linux 系統中， 目錄要掛載在另一個目錄上，一定要下 `--bind` 這個參數

```
bigred@alp:~/c3po$ sudo mount --bind -o ro  /dev  rootfs/dev
```

```
$ mount --help
 -B, --bind              mount a subtree somewhere else (same as -o bind)
 -o, --options <list>    comma-separated list of mount options
```

測試有沒有正常

```
oot@alp:/$ </dev/zero head -c 200m | tail
```

再啟動一個新的 CMD 視窗, 執行以下命令
`$ ssh bigred@<alp.docker IP>`

查詢 rootfs sh 的 pid
```
bigred@alp:~$ ps aux | grep " sh$"
root       4792  0.0  0.0    820     4 pts/0    S    10:13   0:00 unshare --pid --fork --mount-proc --uts -R rootfs sh
root       4793  0.0  0.0   1696  1072 pts/0    S+   10:13   0:00 sh
```

將 sh 這個 process 的 pid 加入 tasks

```
echo "4792" | sudo tee /sys/fs/cgroup/memory/demo/tasks
```

此時 sh 這個 process 可以用的記憶體用量就會被限制

```
@alp:/$ </dev/zero head -c 200m | tail
Killed

$ exit

$ sudo umount rootfs/dev/
```


## 小總結

`unshare` 雖然可以做出很多 Namespace 隔離空間，但它只是把隔離空間做出來，我們還要手動去設定很多細節，才能做出一台 Container 。


---


# 練習做一台Cintainer

## 題目
![](https://i.imgur.com/t11MisD.png)


---

## 答案
```
## 建立rootfs目錄
$ mkdir -p ~/test/rootfs; cd ~/test/rootfs; curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-minirootfs-3.14.1-x86_64.tar.gz; 

## 解壓縮alpine最小的檔案系統
bigred@alp:~/test/rootfs$ tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd ..

## 檢查
bigred@alp:~/test$ dir rootfs/bin/
total 820K

## 建立Overlay2 檔案系統
$ cd ~;mkdir -p ov22/{upper,lower,merged,work}; cd ov22

## 檢查
bigred@alp:~/ov22$ tree

## 設定 Lower 目錄 = 堆疊多層目錄
$ sudo mount -t overlay overlay -o lowerdir=/home/bigred/ov22/lower:/home/bigred/test/rootfs,upperdir=/home/bigred/ov22/upper,workdir=/home/bigred/ov22/work  /home/bigred/ov22/merged

## mount零號裝置
$ sudo mount --bind -o ro  /dev  /home/bigred/ov22/merged/dev

## 在/sys/fs/cgroup/memory/目錄下建立demo2目錄
$ sudo mkdir /sys/fs/cgroup/memory/demo2

## 關閉記憶體造成電腦lag的功能
$ echo "0" | sudo tee /sys/fs/cgroup/memory/demo2/memory.swappiness
0

## 限制記憶體大小 1G
$ echo "1073741824" | sudo tee /sys/fs/cgroup/memory/demo2/memory.limit_in_bytes &>/dev/null

## 建立 cpuset 控制子群組 - first
$ sudo mkdir /sys/fs/cgroup/cpuset/first

## 設定 first 控制子群組權限
$ sudo chown bigred:root -R /sys/fs/cgroup/cpuset/first

## 檢視原本 cpuset 裡的設定：
$ cat /sys/fs/cgroup/cpuset/cpuset.cpus
0-1

## 設定 first 控制子群組, 將 first 控制子群組設成第一個 cpu core
bigred@alp:~/ov22$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.cpus
0

## 啟動Namespace的功能
$ sudo unshare --pid --fork --mount-proc --net --uts -R /home/bigred/ov22/merged sh

## 更改hostname 為LCS22
$ hostname LCS22

## 檢查
$ hostname

# 到另一個終端機連到m1的bigred
## 查詢我們process的pid
$ ps aux | grep " sh$"
14277

## 將PID加入記憶體限制
$ echo 14277 | sudo tee /sys/fs/cgroup/memory/demo2/tasks

## 關閉CPU直接存取記憶體的功能
$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.mems

## 將PID加入CPUSET限制
$ echo 14277 | sudo tee /sys/fs/cgroup/cpuset/first/tasks

# 回到Container的電腦
## 測試CPU
$ yes &>/dev/null &

## 回alp電腦
## 檢查CPU使用哪一核和使用率

# 回到Container的電腦
## 測試記憶體
$ </dev/zero head -c 200m | tail

# 回alp電腦
## 加網卡
$ sudo ip link add ve1 netns 1 type veth peer name ve2 netns 9579

## 將ve1網卡啟動
$ sudo ip link set ve1 up

## 給ve1網卡一個IP位址
$ sudo ip addr add 192.168.1.100/24 dev ve1

# 回到Container的電腦
## 看一下是不是有ve2那張網卡
root@alp:/$ ip a

## 將ve2網卡啟動
root@alp:/$ ip link set ve2 up

## 給ve2網卡一個IP位址
root@alp:/$ ip addr add 192.168.1.200/24 dev ve2

## 測試兩張網卡有沒有通
root@alp:/$ ping -c 1 192.168.1.100
```


---


# Container 組成
- Application(Process)
	- 我們執行的process，可以是Deamon
- Namespace
	- 指令：unshare
	- 提供process不同的隔離功能，包含 network + cpu + memory ...等的隔離空間
- FileSystem（檔案系統）
	- Chroot（重新定義process的根目錄） + Overlay2（堆疊型檔案系統）
- CGroup
	- 透過process的PID以及目錄與檔案來管理cpu、記憶體和網路
- Bridge
	- 指令：brctl show
	- 虛擬橋接器，可以連接網路
- Host OS(Kernel)
- Hareware
	- 硬體周邊

:::spoiler Container組成的數學公式
Container = Application(Process) + Namespace(unshare) + Chroot + Overlay2 + CGroup + bridge/slirp4netns + Host OS (Kernel) + Hardware
:::


---

## Open Container Initiatives (OCI)

[OCI 組織網站連結](https://opencontainers.org/)

- OCI 組織制定一個標準：OCI Container Runtime
- 這個標準負責將“Application(Process) + Namespace(unshare) + Chroot + CGroup ”整合成一個設定檔。
- <font color=red> 注意：沒有 Overlay2 和 Bridge 的功能 </font>
- 有人將設定檔寫成程式，程式名：runc ( Golang 語言 )/ crun ( C語言 )


```
$ sudo apk update ; sudo apk upgrade
```
> 如果遇到很多更新，最好直接`sudo reboot`將虛擬機器重新開機

:::spoiler 遇到Alpine大改版（3.15 -> 3.16）
```
Welcome to DT Platform Version 21.08 (Alpine Linux : 3.16.0)
IP : 192.168.61.144

bigred@alp:~$
```
:::

:::spoiler runc的使用手冊
```
$ runc --help
NAME:
   runc - Open Container Initiative runtime

runc is a command line client for running applications packaged according to
the Open Container Initiative (OCI) format and is a compliant implementation of the
Open Container Initiative specification.

runc integrates well with existing process supervisors to provide a production
container runtime environment for applications. It can be used with your
existing process monitoring tools and the container will be spawned as a
direct child of the process supervisor.

Containers are configured using bundles. A bundle for a container is a directory
that includes a specification file named "config.json" and a root filesystem.
The root filesystem contains the contents of the container.

To start a new instance of a container:

    # runc run [ -b bundle ] <container-id>

Where "<container-id>" is your name for the instance of the container that you
are starting. The name you provide for the container instance must be unique on
your host. Providing the bundle directory using "-b" is optional. The default
value for "bundle" is the current directory.

USAGE:
   runc [global options] command [command options] [arguments...]

VERSION:
   1.1.2
commit: a916309fff0f838eb94e928713dbc3c0d0ac7aa4
spec: 1.0.2-dev
go: go1.18.2
libseccomp: 2.5.2

COMMANDS:
   checkpoint  checkpoint a running container
   create      create a container
   delete      delete any resources held by the container often used with detached container
   events      display container events such as OOM notifications, cpu, memory, and IO usage statistics
   exec        execute new process inside the container
   kill        kill sends the specified signal (default: SIGTERM) to the container's init process
   list        lists containers started by runc with the given root
   pause       pause suspends all processes inside the container
   ps          ps displays the processes running inside a container
   restore     restore a container from a previous checkpoint
   resume      resumes all processes that have been previously paused
   run         create and run a container
   spec        create a new specification file
   start       executes the user defined process in a created container
   state       output the state of a container
   update      update container resource constraints
   features    show the enabled features
   help, h     Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug             enable debug logging
   --log value         set the log file to write runc logs to (default is '/dev/stderr')
   --log-format value  set the log format ('text' (default), or 'json') (default: "text")
   --root value        root directory for storage of container state (this should be located in tmpfs) (default: "/run/runc")
   --criu value        path to the criu binary used for checkpoint and restore (default: "criu")
   --systemd-cgroup    enable systemd cgroup support, expects cgroupsPath to be of form "slice:prefix:name" for e.g. "system.slice:runc:434234"
   --rootless value    ignore cgroup permission errors ('true', 'false', or 'auto') (default: "auto")
   --help, -h          show help
   --version, -v       print the version
```
:::

檢查runc版本
```
$ runc -v
runc version 1.1.2
commit: a916309fff0f838eb94e928713dbc3c0d0ac7aa4
spec: 1.0.2-dev
go: go1.18.2
libseccomp: 2.5.2
```

產生OCI的標準
```
$ runc spec
bigred@alp:~/c3po$ dir
total 20K
drwxr-sr-x  4 bigred bigred 4.0K May 24 14:11 .
drwxr-sr-x 15 bigred bigred 4.0K May 24 13:46 ..
-rw-r--r--  1 bigred bigred 2.5K May 24 14:11 config.json
drwxr-sr-x  6 bigred bigred 4.0K May 20 11:21 ov2
drwxr-sr-x 19 bigred bigred 4.0K May 20 10:30 rootfs
```

:::spoiler 認識`config.json`

```bash=
{
        "ociVersion": "1.0.2-dev",
        "process": {
                "terminal": true,
                "user": {
                        "uid": 0,
                        "gid": 0
                },
                "args": [
                        "sh"       ## 我們的process
                ],
...
	##Chroot，重新定義根目錄到rootfs
        "root": {
                "path": "rootfs",
                "readonly": true  ## 將true -> false
        },
        "hostname": "runc", ## 將runc -> r2d2
		## 將記憶體型的proc掛載到硬碟上的/proc
        "mounts": [
                {
                        "destination": "/proc",
                        "type": "proc",
                        "source": "proc"
                },
                {
                        "destination": "/dev",
                        "type": "tmpfs",

```
:::


修改後
```bash=
        "root": {
                "path": "rootfs",
                "readonly": false
        },
        "hostname": "r2d2",
        "mounts": [
                {
                        "destination": "/proc",
                        "type": "proc",
                        "source": "proc"
                },
```

啟動Container

```
bigred@alp:~/c3po$ sudo runc run bb8
/ #
```

上面的命令成功執行的必要條件：必須要有==rootfs + config.json（一個檔案系統的目錄 + 設定檔）==

```
/ # hostname
r2d2
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps aux
/ # whoami
root
/ # mkdir zzz
mkdir: can't create directory 'zzz': Permission denied
```

:::spoiler 提問：什麼是runc，怎麼用？
答：
有人根據OCI Container Runtime的標準寫出來的程式，這支程式的語言是Golang
指令：runc ，命令成功執行的必要條件：所在的工作目錄必須要有==rootfs + config.json==
config.json 產生指令：`runc spec`，成功的話，工作目錄裡rootfs目錄裡面必須要有linux的檔案系統
runc的使用手冊命令：`runc --help`
runc的版本查詢命令：`ruc -v`

---

by 程威
- OCI｜Open Container Initiative 制定 Container 標準的組織
- OCI Container Runtime｜Container 的部署標準
（包含：APP（Process） + namespace（unshare）+ chroot + Overlay 2 + cgroup）
 - 這項標準有兩套程式可以使用
    - runc｜使用 go 語言
    - crun｜使用 C 語言
- 使用 OCI 標準建立 Container
    - 使用 runc（go 語言）指令建立的 Container 指具備基礎功能，其他進階功能需要額外調整與部署
    - 使用 runc 建立 Container 要具備的兩個重要項目
    - rootfs｜程序的根目錄環境
    - config.json｜OCI 指定的設定檔 

#檢查 runc 版本
runc -v

#生成標準設定檔案 config.json
runc spac

#建立一個名稱為 bb8 的 Container
sudo runc run bb8
:::


---


系統工程由[cncf.io](https://www.cncf.io) (Cloud Native Computing Foundation)裡面的人主導

資料科技由[Apache.org](https://apache.org/) 裡面的人主導

[Containerd](https://www.cncf.io/projects/containerd/) (d 是deamon的意思)
- 這個專案有本事管理一台電腦裡面多個Container，但真正建立Container的還是透過runc來完成，而rootfs的內容可以請Containerd上網下載他要管理的Container所需的rootfs的內容

- Containerd的功能：創造、啟動、監控（紀錄運作時的log檔）、移除和關閉Container


Docker這家公司設計一個程式名稱: `docker`
他是透過Containerd來管理一台電腦中很多台Container，而Container之間的網路由docker自己來做

安裝docker的命令
```
$ sudo apk add docker
(1/9) Installing containerd (1.6.4-r1)
(2/9) Installing containerd-openrc (1.6.4-r1)
(3/9) Installing ip6tables (1.8.8-r1)
(4/9) Installing ip6tables-openrc (1.8.8-r1)
(5/9) Installing tini-static (0.19.0-r0)
(6/9) Installing docker-engine (20.10.16-r0)
(7/9) Installing docker-openrc (20.10.16-r0)
(8/9) Installing docker-cli (20.10.16-r0)
(9/9) Installing docker (20.10.16-r0)
Executing docker-20.10.16-r0.pre-install
Executing busybox-1.35.0-r13.trigger
OK: 1263 MiB in 265 packages
```

將docker的demon設為開機自動執行
```
bigred@alp:~$ sudo rc-update add docker default
```

重新開機，並執行`ps aux | grep docker`
```
bigred@alp:~$ ps aux | grep docker
root       3093  0.0  0.0    936   440 ?        S    15:53   0:00 supervise-daemon docker --start --retry TERM/60/KILL/10 --stderr /var/log/docker.log --stdout /var/log/docker.log /usr/bin/dockerd --
root       3095  1.9  3.1 776508 62272 ?        Ssl  15:53   0:00 /usr/bin/dockerd
root       3249  0.8  1.9 756164 38852 ?        Ssl  15:53   0:00 containerd --config /var/run/docker/containerd/containerd.toml --log-level info
bigred     3478  0.0  0.0   1596   760 pts/0    S+   15:54   0:00 grep docker
```

###### tags: `系統工程`














































































































































































