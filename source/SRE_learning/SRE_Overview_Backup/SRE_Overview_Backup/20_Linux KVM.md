# Linux KVM
## 介紹KVM
KVM是linux的核心技術，它可以啟動虛擬機器，和虛擬化技術，市面上所有linux都有這個功能，linux kennel是由Linus Torvalds寫的，他在2006年12月將Linux KVM寫進linux系統，只要是linux系統，CPU是intel的一定要有VT-X的功能，AMD的話要有開啟SVM功能，就能啟動KVM

我們透過VMware player建立Ubuntu或是Alpine虛擬機器，只要在原生的Linux系統上有啟動核心KVM的功能，這時候Linux系統搖身一變，變成可以啟動很多很多虛擬機器的平台。

只要是linux系統，有linux kernel，CPU具備VT-x功能(or SVM)，那麼你就可以直接啟動KVM。

### 硬體輔助虛擬化（Hardware-assisted virtualization）

-    Intel Virtualization Technology（VT-x）
-    AMD Secure Virtual Machine（SVM）

有此技術，各家虛擬技術均可稱為完全虛擬化，它們的差別在週邊硬體（網卡、硬碟）虛擬化支援，這兩個技術會**直接穿透OS，直接使用CPU和記憶體，但其他的硬體周邊不支援**
> 用穿透是比較誇張的詞，還是需要透過Hypervisor來調節資源，只是能夠用到真實的硬體而不是模擬的。


> 過去的專有名稱
    半虛擬化   : 需修改系統核心（Windows 不支援）
    完全虛擬化  : 需執行   CPU 指令轉碼
   如果有看到以上兩個名詞跟硬體輔助虛擬化比較的文章再看 

### QEMU
早期硬體沒有硬體輔助虛擬，所以使用 qemu 啟動虛擬機，後來硬體輔助虛擬出現，就可以讓 qemu 搭配 kvm

QEMU他是虛擬技術的一個軟體，這個軟體能夠幫你產生一台虛擬機器，但是他的CPU,記憶體,硬碟和網卡都是假的，此時搭配KVM產生的虛擬機器，代表底層有啟動VT-X或SVM，這台機器的CPU,記憶體就是真的，也就是用實體機器的


### KVM運作架構
![](https://i.imgur.com/WkJ7o2g.png)
1. Hareware：指的是CPU,記憶體,網路和主機板...等
2. 在作業系統啟動KVM的話，建立出來的VM，就可以穿透作業系統，直接使用CPU和記憶體，這樣的速度很快
3. 除了CPU記憶體，用QEMU模擬出硬碟或網卡，這時候硬碟和網卡就是假的，就會比較慢
4. VirtIO是一個虛擬機器周邊的加速引擎，他可以幫我們用QEMU模擬出來的硬碟和網卡加速，但啟動前有一個先決條件，Intel的CPU一定要啟動VT-d這項技術，（本次課程練習不到）


## GCP建VM
先連到GCP，在Compute Engine裡的設定 >>> 把預設位置改成台灣，區域Ｃ
![](https://i.imgur.com/6zXAHDv.png)

建立執行個體
機器名稱: mykvm
機器設定，選運算最佳化
系列：C2
機器類型：c2-standard-4（4個vCPU，16 GB記憶體）
![](https://i.imgur.com/7LgNah4.png)
開機磁碟選
作業系統：Ubuntu
版本：Ubuntu 20.04 LTS
開機磁碟類型：SSD 永久磁碟
大小（GB）: 100 GB
好了後按選取
![](https://i.imgur.com/lysH075.png)

### KVM-Check
先連線進去做一開始的環境設定

![](https://i.imgur.com/lLNMXkC.png)

sudo nano /etc/ssh/sshd_config
![](https://i.imgur.com/euGS2VL.png)

sudo systemctl restart ssh

sudo passwd sreantony

sudo passwd root

sudo nano /etc/sudoers
![](https://i.imgur.com/hLQkzIr.png)

在落地雲的CMD，ssh連進去
``


```
$ sudo apt-get update

$ sudo apt-get install cpu-checker
........
0 upgraded, 2 newly installed, 0 to remove and 130 not upgraded.
Need to get 16.9 kB of archives.
After this operation, 66.6 kB of additional disk space will be used.
Do you want to continue? [Y/n] y

$ kvm-ok
INFO: Your CPU does not support KVM extensions
INFO: For more detailed results, you should run this as root
```

## Hypervisor
虛擬機器的建立,開啟,關閉,刪除,還有虛擬機器和虛擬機器之間的隔離，以上這些技術要完成的就是Hypervisor，虛擬機器的一生，要怎麼去管理和監控都是靠Hypervisor，如果沒有它，以上這些事情通通都不能做
![](https://i.imgur.com/WdJnLVu.png)

分為兩個架構

1. Type 1 Hypervisor 裸機架構（Bare-Metal）
企業會使用這種架構，作業系統會內建Hypervisor，舉個例字：Linux KVM，只要你是linux系統，有linux kernrl ，那一定會有KVM，這個架構就可以統稱裸機架構，有了這種架構，你就可以直接啟動不同的作業系統


2. Type 2 Hypervisor 寄居架構 （Hosted）
作業系統沒有內定Hypervisor，在作業系統上額外安裝Hypervisor就是寄居架構，這時候運作效率就會變慢，因為它還要透過作業系統才能做管理，像是我們在落地雲windows電腦裡面安裝VMware就是，這種寄居架構

## 巢狀虛擬化
會有很多個Hypervisor，在虛擬機器裡面再開一個虛擬機器，一般企業不允許，在GCP裡面可以突破這個限制，在GCP裡面建立一台虛擬機器，裡面再建立一台虛擬機器

### GCP 巢狀虛擬化的限制
1. 一定要是linux作業系統
2. 機器類型不能使用**E2**和**N2D**
![](https://i.imgur.com/vX0DaNZ.png)

## 設定GCP巢狀虛擬
### 第一步 啟動Cloud shell
用cloud shell的原因是因為不用額外安裝gcloud這個命令
先點這個
![](https://i.imgur.com/7yhXbJt.png)
開啟編輯器
![](https://i.imgur.com/Zc4yApN.png)
把終端機的功能打開
![](https://i.imgur.com/IVQpP75.png)
會呈現以下畫面
![](https://i.imgur.com/TPXbu6i.png)

這就是gcp裡面的一台機器，我們已經透過cloud shell連進去這台機器了
這台機器不用錢，我們用這台虛擬機器是因為google在管理虛擬機器，有一項命令叫做gcloud，這樣就不用額外安裝
### 第二部 匯出
先下一個環境變數
```
$ export vmname='mykvm'
```


檢查區域
![](https://i.imgur.com/GBUaAbH.png)


```
$ gcloud compute instances export ${vmname} \

  --destination=${vmname}.yaml \  #設定等等匯出的名字叫mykvm.yaml

  --zone=**asia-east1-c**         #在gcloud上面要匯出你的VM的設定檔，一定會綁定機器在哪個地區

```

用tee命令在yaml檔後面再加兩筆資料，這就是巢狀虛擬化必須啟動的功能，可以用kvm-ok確定要不要設定
```
$ tee -a ${vmname}.yaml<<EOF
advancedMachineFeatures:
  enableNestedVirtualization: true 
``` 
![](https://i.imgur.com/pIEblFY.png)
                             
把剛剛設定好的yaml檔，傳回去給gcp，並且請他重新開機立即生效

```
$ gcloud compute instances update-from-file ${vmname} \
  --source=${vmname}.yaml \
  --most-disruptive-allowed-action=RESTART \
  --zone=asia-east1-c
```

好了後，再用ssh命令連進去機器，注意外部IP這時候有可能會更動

```
$   kvm-ok

INFO: /dev/kvm exists

KVM acceleration can be used
```

安裝Kvm
```
$   sudo apt install qemu-kvm

........

0 upgraded, 26 newly installed, 0 to remove and 130 not upgraded.

Need to get 19.6 MB of archives.

After this operation, 82.5 MB of additional disk space will be used.

Do you want to continue? \[Y/n\] y
```

```
$   sudo adduser ${USER} kvm

$   sudo reboot
```

檢查   kvm 是否正確安裝

```
$   kvm -version

QEMU emulator version 4.2.1 (Debian 1:4.2-3ubuntu6.21)

Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers
```

### Alpine系統安裝
```
$   mkdir wk wk/iso wk/disk

$   cd wk

$   wget https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/x86\_64/alpine-virt-3.15.4-x86\_64.iso -O iso/alpine-virt-3.15.4-x86_64.iso
```
使用qemu-img產生qcow2格式的硬碟

```
$ qemu-img create -f qcow2 disk/alpine.qcow2 10G

Formatting 'disk/alpine.qcow2', fmt=qcow2 size=10737418240 cluster\_size=65536 lazy\_refcounts=off refcount_bits=16
```

檢查硬碟格式
```
$ qemu-img info disk/alpine.qcow2
image: disk/alpine.qcow2
file format: qcow2
virtual size: 10 GiB (10737418240 bytes)
disk size: 196 KiB
```

```
$   sudo kvm -m 2048 \

-cdrom iso/alpine-virt-3.15.4-x86_64.iso \

-hda disk/alpine.qcow2 \

-nographic

   

參數說明：

-m 虛擬機記憶體設定   (單位   MB)

-cdrom 虛擬機開機放入光碟片

-hda   虛擬機開機掛載硬碟   (代號   a)

-nographic 不啟動圖形化介面

   

備註：

開機順序   hd(a) -> cdrom
```
alpine設定
![](https://i.imgur.com/tUUUI5N.png)
![](https://i.imgur.com/kVi1edt.png)

```
$   which kvm

/usr/bin/kvm
```

   

```
$   cat /usr/bin/kvm

#!/bin/sh

exec qemu-system-x86_64 -enable-kvm "$@"
```

   

   

Q.   請測試有無啟動   kvm，Alpine 系統開機速度差多少？

A. 
有用Kvm
```
sudo qemu-system-x86_64 -enable-kvm -m 2048 \

-hda disk/alpine.qcow2 \

-nographic
```

沒加kvm
```
sudo qemu-system-x86_64 -m 2048 \

-hda disk/alpine.qcow2 \

-nographic
```

## 電腦基本元件
4大元件：cpu 硬碟 記憶體 主機板
伺服器也是一台電腦，只是規格比較好，他也會有以上四大元件

### cpu
中央處理器，一台電腦要去跑程式，都是cpu(中央處理器)的功勞
兩大cpu廠商：Intel 和 AMD
我們現在比較的是x86架構，手機的cpu處理器不列入比較
![](https://i.imgur.com/fpcVYTb.png)

由上往下依序，是入門款到高階
```
Core  i7  -  11  700 k
     系列    世代 性能 尾碼
```
AMD的處理器都能超頻

以i7-8700為例，https://www.intel.com.tw/content/www/tw/zh/products/sku/126686/intel-core-i78700-processor-12m-cache-up-to-4-60-ghz/specifications.html


![](https://i.imgur.com/tbniUTk.png)

如何報cpu規格： 
i7-8700，6核12執行序，（3.20GHz）時脈

尾數Ｕ，代表就是筆電的處理器
![](https://i.imgur.com/4rXhwXS.png)

伺服器的（CPU）
因為企業裡面幾乎都是intel，所以不討論ＡＭＤ
命名（2017年以前）

2018後，怎麼看規格




看內部結構
真正在跑運算process的是core不是threads，除非程式裡面有寫

但是只要稱得上虛擬化技術的都是跑threads

甚麼是vCPU？
直接當threads看


### 記憶體
記憶體的重點，容量和速度
雙通道的效能會大於單通道，也就是說兩條4G的記憶體效能會大於一條8G
規格DDR5 > DDR4 > DDR3

### 硬碟
在規格相同的情況下，M.2 > SSD > HDD


## 設定KVM Disk

擴充   alpine 硬碟容量

```
$   qemu-img resize disk/alpine.qcow2 +10G

Image resized.
```

   

檢查   alpine 硬碟容量

```
$   qemu-img info disk/alpine.qcow2

image: disk/alpine.qcow2

file format: qcow2

virtual size: 20 GiB (21474836480 bytes)

disk size: 9.75 GiB
```

   
將alpine開機

```
$   sudo kvm \

-m 2048 \

-hda disk/alpine.qcow2 \

-nographic
```


延伸空間
```
alpine66:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1    0B  0 disk
sda      8:0    0   20G  0 disk
├─sda1   8:1    0  100M  0 part /boot  (開機使用)
├─sda2   8:2    0  2.5G  0 part [SWAP] （）
└─sda3   8:3    0  7.4G  0 part /
sr0     11:0    1 1024M  0 rom
```

**在業界，是用多少硬碟給多少，不會一次到位，因為給了就沒辦法壓縮回來**



在實物上，windows完全不建議一個磁碟分成兩個分割區



### 主機板（MB）
規格分3種，體積由大到小ATX >>> MICRO ATX >>> MINI ATX

### 認識企業伺服器
![](https://i.imgur.com/xwGnh0e.png)





![](https://i.imgur.com/tm5fu22.png)
vmdk是vmware 虛擬機器的硬碟檔
VHDX是微軟 Microsoft Hyper-V虛擬機器的硬碟檔
VPC是雅馬遜Amazon 虛擬機器的硬碟檔
QCOW2是KVM 虛擬機器的硬碟檔




## KVM Bridge Network 

學習目標
-    QEMU 網路設定
-    QEMU User Mode 網路
-    TUN/TAP 虛擬網卡
-    自製橋接網路 (Bridge)
-    dnsmasq DHCP Server 架設
-    KVM 虛擬機器透過 NAT 上網

### QEMU 網路設定
確認   QEMU   支援網卡型號

*預設為   E1000 (Intel® 82540EM)
```
$ kvm -net nic,model=?
```

確認   QEMU   支援網路類型
```
$ kvm --help | grep "\\-net "

-net nic[,macaddr=mac][,model=type][,name=str][,addr=str][,vectors=v]
-net [user|tap|bridge|socket][,option][,option][,...]
```

### QEMU User Mode 網路
![](https://i.imgur.com/2cfs5uu.png)
SLIRP主要是模擬撥接網路，對應在QEMU的USER Mode 就會非常慢

User Networking (SLIRP)

-    there is a lot of overhead so the performance is poor
-    in general, ICMP traffic does not work (so you cannot use ping within a guest)
-    on Linux hosts, ping does work from within the guest, but it needs initial setup by root (once per host)
-    the guest is not directly accessible from the host or the external network

--- 
#### 實作
先確定工作目錄
```
$   pwd
/home/desktop/wk
```

下載   Tiny Core Linux 光碟片
```
$ wget http://www.tinycorelinux.net/13.x/x86/release/Core-current.iso -O iso/tcl.iso
```

透過KVM開啟Tinycore Linux虛擬機器
```
kvm -m 128 -net nic -net user -cdrom iso/tcl.iso -curses
```
>-curses 指的是不要圖形化介面
    -net nic 是網卡，這邊不指定
    -net user 是網路要user mode

確認IP : 10.0.2.15
和Mac address ： 52:54:00:12:34:56
```
＃ ifconfig
```
確認Gateway : 10.0.2.2
```
# route -n
```

確認DNS : 10.0.2.3
```
# cat /etc/resolv.conf
```

* Tiny Core Linux 簡稱TCL，在TCL 虛擬機中確認網卡型號

安裝   lshw   套件
```
$ tce-load -wi lshw.tcz
```

確認網卡型號
```
$ sudo lshw -class network

  ::

product: 82540EM Gigabit Ethernet Controller
```
TCL   關機
```
$ sudo poweroff
```
---
### TUN/TAP 虛擬網卡
TAP (as in network tap) simulates a link layer device and it operates with layer 2 packets such as Ethernet frames. TUN (as in network TUNnel) simulates a network layer device and it operates with layer 3 packets such as IP packets. **TAP is used to create a network bridge**, while TUN is used with routing.

> TAP這張網卡主要模擬乙太網路的網卡，TAP通常用來做橋接網路

---
#### **使用** **tunctl** **指令**

安裝tunctl 套件
```
$ sudo apt install uml-utilities
```

產生 TAP 網路裝置
```
$ sudo tunctl -u ${USER}
Set 'tap0' persistent and owned by uid 1000
```

手動設定   TAP 網卡的   MAC 位址
```
$ sudo   ifconfig tap0 hw ether 4c:22:d0:b8:78:ae
```

查看   TAP 網路裝置
```
$ ifconfig tap0
tap0: flags=4098<BROADCAST,MULTICAST>  mtu 1500

ether 4c:22:d0:b8:78:ae  txqueuelen 1000  (Ethernet)

  ::
```

移除   TAP   網路裝置
```
$ tunctl -d tap0
Set 'tap0' nonpersistent
```
---

### Ethernet 重點回顧
有線乙太網路   Ethernet

一個乙太網路的要件包括了

-    乙太網路卡   (Ethernet card)
-    網路連接設備
-    連接設備的電纜   (cabling)

---

### TCP/IP Model
![](https://i.imgur.com/7XORo9s.png)
> Data-Link Layer 資料連接層
Physical Layer 實體層

---

### **Bridge & STP**
A network bridge is a computer networking device that creates a single, aggregate network from multiple communication networks or network segments. In the OSI model, bridging is performed in the data link layer (layer 2).

   

The Spanning Tree Protocol (STP) is a network protocol that builds a loop-free logical topology for Ethernet networks. The basic function of STP is to prevent bridge loops (迴路) and the broadcast radiation (廣播風暴) that results from them.

---
### 使用brctl 指令

安裝brctl 套件
```
$ sudo apt install bridge-utils
```

產生 bridge 網路裝置
```
$ sudo brctl addbr br01
```

查看   br01 的資訊

```
$ brctl show br01

bridge name   bridge id    STP enabled   interfaces

br01    8000.000000000000   no

*8000 是   id，句點後是橋接裝置的   MAC 位址
```

br01 裝置的內部網卡(網卡名稱與   Bridge   網路裝置同名)

```
$ ifconfig br01

br01: flags=4098<BROADCAST,MULTICAST>  mtu 1500

  ether 7e:49:b0:80:e9:62  txqueuelen 1000  (Ethernet)
```

移除 br01 裝置
```
$ sudo brctl delbr br01
```
---
### 自製 KVM橋接網路
設定   macsw   裝置的   IP
```
$ sudo ifconfig macsw 172.16.20.254 netmask 255.255.255.0 up
```

檢查
```
$ ifconfig macsw

macsw: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

inet 172.16.20.254  netmask 255.255.255.0  broadcast 172.16.20.255

  inet6 fe80::303b:1dff:fe2a:2324  prefixlen 64  scopeid 0x20<link>

  ether 32:3b:1d:2a:23:24  txqueuelen 1000  (Ethernet)

  RX packets 0  bytes 0 (0.0 B)

  RX errors 0  dropped 0  overruns 0  frame 0

  TX packets 37  bytes 4880 (4.8 KB)

  TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


產生二個   TAP 網路介面

```
$ sudo tunctl -b -u ${USER}

Tap0

$ sudo tunctl -b -u ${USER}

tap1
```

啟動新建   TAP 網路裝置

```
$ sudo ifconfig tap0 up

$ ifconfig tap0

$ sudo ifconfig tap1 up

$ ifconfig tap1
```

將   TAP 網路介面連接至   Bridge 網路裝置

```
$ sudo brctl addif macsw tap0

$ sudo brctl addif macsw tap1
```


檢視   macsw 網路裝置的   MAC 表格

```
$ sudo brctl showmacs macsw

port no   mac addr    is local?   ageing timer

  2   f6:af:90:76:95:ea   yes      0.00

  2   f6:af:90:76:95:ea   yes      0.00

  1   fa:47:ee:b7:e2:8b   yes      0.00

  1   fa:47:ee:b7:e2:8b   yes      0.00
  
$ ifconfig macsw
```

檢視   macsw 內部運作資訊
```
$ sudo brctl showstp macsw
```
   

顯示所有   bridge 網路裝置
```
$ brctl show

bridge name   bridge id    STP enabled   interfaces

macsw    8000.f6af907695ea   no    tap0

       tap1
```



### dnsmasq 介紹
dnsmasq is free software providing Domain Name System (DNS) caching, a Dynamic Host Configuration Protocol (DHCP) server, router advertisement and network boot features, intended for small computer networks.

dnsmasq has low requirements for system resources, can run on Linux, BSDs, Android and macOS, and is included in most Linux distributions. Consequently, it "is present in a lot of home routers and certain Internet of Things gadgets" and is included in Android.


檢查 53 Port 被誰佔用
```
$ sudo lsof -i :53
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 468 systemd-resolve   12u  IPv4  17221      0t0  UDP localhost:domain
systemd-r 468 systemd-resolve   13u  IPv4  17222      0t0  TCP localhost:domain (LISTEN)
```


停止並停用   systemd-resolved
```
$ sudo systemctl stop systemd-resolved

$ sudo systemctl disable systemd-resolved
sudo: unable to resolve host mykvm: Temporary failure in name resolution ##注意這行要sudo nano /etc/hosts進去修改
Removed /etc/systemd/system/multi-user.target.wants/systemd-resolved.service.
Removed /etc/systemd/system/dbus-org.freedesktop.resolve1.service.
```

在/etc/hosts 新增hosts name
```
$ sudo nano /etc/hosts
sudo: unable to resolve host mykvm: Temporary failure in name resolution
將127.0.0.1 localhost修改成127.0.0.1 localhost mykvm
```
ctrl o 儲存 ctrl x離開之後，在下sudo命令就不會噴錯了


   

查看   /etc/resolv.conf

```
$ ls -al /etc/resolv.conf

lrwxrwxrwx 1 root root 39 Apr 19 19:32 /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
```

   

刪除   /etc/resolv.conf
```
$ sudo rm /etc/resolv.conf
```
   

產生新的 /etc/resolv.conf
```
$ echo nameserver 8.8.8.8 | sudo tee /etc/resolv.conf

nameserver 8.8.8.8
```


安裝   dnsmasq 套件

```
$ sudo apt install dnsmasq
```
   

編輯   dnsmasq 設定，在最下方加入下列內容
```
$ echo "

dhcp-range=172.16.20.150,172.16.20.160,24h

dhcp-option=option:netmask,255.255.255.0

dhcp-option=option:router,172.16.20.254

dhcp-option=option:dns-server,8.8.8.8

" | sudo tee -a /etc/dnsmasq.conf
```
> 第一行，給一個IP的範圍，讓dhcp server派發．
> 24h 是IP租用時間，意思是說會給client端24小時的時間，當時間經過一半，client端會去跟server續約，如果沒續約的話，server端會收回IP

重啟   dnsmasq
```
$ sudo systemctl restart dnsmasq
```


在另一個終端機
啟動第一部   TCL   虛擬電腦

```
$ kvm -m 128 -cdrom iso/tcl.iso \

-net nic,macaddr=52:54:72:16:20:10 \

-net tap,ifname=tap0 -curses
```

   

**進入**   **TCL**   **虛擬主機後請檢查**

是否有自動取得IP
```
tc@box:~$ ifconfig
                eth0      Link encap:Ethernet  HWaddr 52:54:72:16:20:10
                          inet addr:172.16.20.150  Bcast:172.16.20.255  Mask:255.255.255.0
                          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
                          TX packets:4 errors:0 dropped:0 overruns:0 carrier:0
                          collisions:0 txqueuelen:1000
                          RX bytes:1328 (1.2 KiB)  TX bytes:1086 (1.0 KiB)

                lo        Link encap:Local Loopback
                          inet addr:127.0.0.1  Mask:255.0.0.0
                          UP LOOPBACK RUNNING  MTU:65536  Metric:1
                          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
                          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
                          collisions:0 txqueuelen:1000
                          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
Default Gateway是否設定正確
```
tc@box:~$ route -n
                Kernel IP routing table
                Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
                0.0.0.0         172.16.20.254   0.0.0.0         UG    0      0        0 eth0
                127.0.0.1       0.0.0.0         255.255.255.255 UH    0      0        0 lo
                172.16.20.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```
DNS是否設定正確
```
tc@box:~$ cat /etc/resolv.conf

nameserver 8.8.8.8
```


啟動第二部 TCL 虛擬電腦
$ kvm -m 128 -cdrom iso/tcl.iso \
-net nic,macaddr=52:54:72:16:20:11 \
-net tap,ifname=tap1 -curses




是否有自動取得IP
```
tc@box:~$ ifconfig
                eth0      Link encap:Ethernet  HWaddr 52:54:72:16:20:11
                          inet addr:172.16.20.151  Bcast:172.16.20.255  Mask:255.255.255.0
                          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
                          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
                          collisions:0 txqueuelen:1000
                          RX bytes:1268 (1.2 KiB)  TX bytes:1026 (1.0 KiB)

                lo        Link encap:Local Loopback
                          inet addr:127.0.0.1  Mask:255.0.0.0
                          UP LOOPBACK RUNNING  MTU:65536  Metric:1
                          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
                          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
                          collisions:0 txqueuelen:1000
                          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

Default Gateway是否設定正確
```
tc@box:~$ route -n
                Kernel IP routing table
                Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
                0.0.0.0         172.16.20.254   0.0.0.0         UG    0      0        0 eth0
                127.0.0.1       0.0.0.0         255.255.255.255 UH    0      0        0 lo
                172.16.20.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

DNS是否設定正確

```
tc@box:~$ cat /etc/resolv.conf

nameserver 8.8.8.8
```

### 企業IP網路規劃分法
IP 0~255 去頭去尾 1~254 前十後五 中間切一刀 前半靜態 後半 DHCP
頭：Network ID
尾：廣播位址
前10：系統服務（DNS Server , DHCP Server ...等）
後五：網路設備
切一半
前半：手動設定靜態IP
後半：DHCP Server自動配發IP


## **ARP** **重點回顧**
![](https://i.imgur.com/gSNDjAS.png)
當Host A要傳給Host B資料時， 第一步Host A主機會先檢查自己的ARP cache有沒有Host B的MAC位址，如果沒有紀錄，




它就會進到第二部，在Switch上面廣播"ARP requset"，ARP requset裡面會有：
* Host A自己的IP address 和 MAC address
* Host B的IP address和MAC address，但因為這時候不知道，所以顯示都是00:00:00:00:00:00（ 00:00:00:00:00:00 ）

第三部
Host B主機收到ARP request後，會在自己這邊的ARP cache紀錄Host A主機的MAC Address

第四部
Host B主機會將自己的IP位址和MAC address位址傳給Host A


第五部
Host A主機會在自己的ARP Cache中把收到的Host B 主機的IP位址和MAC address紀錄起來

---

### 清除ARP Cache
\[TCL   主機   172.16.20.150\]

```
$ arp -n

? (172.16.20.254) at 06:6f:48:e9:86:1b \[ether\]  on eth0

? (172.16.20.151) at 52:54:72:16:20:11 \[ether\]  on eth0
```

   

清除   ARP Cache

```
$ sudo   arp -d 172.16.20.151

   

$ arp -n

? (172.16.20.254) at 06:6f:48:e9:86:1b \[ether\]  on eth0
```

---

### **擷取封包**
\[HOST   主機\]

```
$ pwd

/home/qazstuser01/wk


   

$ sudo tcpdump -s 0 -i macsw8 -w tcpdump.pcap

tcpdump: listening on macsw8, link-type EN10MB (Ethernet), capture size 262144 bytes
```
   

\[TCL   主機   172.16.20.150\]
```
$ ping -c 4 172.16.20.151
```
   

\[HOST   主機\]

**回到正在執行**   **tcpdump**   **的終端機，按下**   **Ctrl + C**   **停止擷取**



###### tags: `Linux`







