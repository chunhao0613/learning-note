# Container-Runtime-CRI


:::warning

:::spoiler 目錄 (  :mag_right: 點我，可展開 )

[TOC]

:::

- [前篇 Container 核心技術詳細介紹連結](https://hackmd.io/@QI-AN/container-core-tech)

# Open Container Initiatives (OCI)

- OCI 組織制定一個標準 : Image spec
    - 這個標準是建置 image 的標準
    - 由多個設定檔 config + 多個目錄的壓縮檔 layers 組成
    - Image 的建置 有分 **OCI 組織訂的標準**與 **docker 的標準**
- 另一個標準：Runtime spec
    - 建置 Container 的標準。
    - 將 `Namespace ( unshare )` + `Chroot` + `CGroup` + `Overlay2` + `Virtual Bridge` 需要的參數設定整合到一個設定檔
    - <font color=red> 注意：runc 沒有 Overlay2 和 Bridge 的功能 </font>
    - 有人將設定檔寫成程式，程式名：runc ( Golang 語言 )/ crun ( C 語言 )


![](https://i.imgur.com/yALLRBI.png)

- 將 Container Image 裡的 多個設定檔 config 和 多個壓縮檔 layers 經過 unpack 解包後成為 Bundle ( runtime 設定檔 + rootfs 檔案系統目錄 )，再透過 runtime spec 的程式 create 成為 Container
- Create Container 的技術包含 Namespace ( unshare ) + Chroot + CGroup + Overlay2 + Virtual Bridge

# runc
提問：什麼是 runc，怎麼用？
答：
- 有人根據 OCI Container Runtime 的標準寫出來的程式，這支程式的語言是 Golang
- 指令：runc ，命令成功執行的必要條件：所在的工作目錄必須要有==rootfs + config.json==
- config.json 產生指令：`runc spec`，成功的話，工作目錄裡 rootfs 目錄裡面必須要有 linux 的系統檔案
- runc的使用手冊命令：`runc --help`
- runc的版本查詢命令：`ruc -v`

```
$ sudo apk update ; sudo apk upgrade
```
> 如果遇到很多更新，最好直接 `sudo reboot` 將虛擬機器重新開機

檢查 runc 版本

```bash!
$ runc -v
```

output : 

```bash!
runc version 1.1.2
commit: a916309fff0f838eb94e928713dbc3c0d0ac7aa4
spec: 1.0.2-dev
go: go1.18.2
libseccomp: 2.5.2
```


---


## 準備 Runtime Filesystem bundle

在 Windows 系統的 CMD 視窗, 執行以下命令

`$ ssh bigred@<ALP.Docker IP>`

```bash!
$ mkdir ~/alpminifs; cd ~/alpminifs; curl -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.16/releases/x86_64/alpine-minirootfs-3.16.2-x86_64.tar.gz; tar xvf alpine.tar.gz ;rm alpine.tar.gz; cd .. 
```

- <font color=red>因 runc 命令內定執行帳號為 root, 而 rootfs 目錄的 Owner 是 bigred, 所以需將 rootfs 目錄的 owner 改為 root</font>

```
$  sudo chown root:root -R rootfs/
```


---


## 產生 OCI 標準的 Container 設定檔

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

:::spoiler 認識 `config.json`

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


---

## 建立 bb8 Container (啟用 Namespace)


```
bigred@alp:~/c3po$ sudo runc run bb8
/ #
```

- 上面的命令成功執行的必要條件：必須要有==rootfs + config.json（一個系統檔案的目錄 + OCI 標準的 Container 設定檔）==

```bash!
/ # hostname
r2d2
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps aux
/ # whoami
root
/ # ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    9 root      0:00 ps aux
/ # ls -al /bin | head -n 5
total 820
drwxr-sr-x    2 root     root          4096 May 25 01:15 .
drwxr-sr-x   19 root     root          4096 May 25 01:15 ..
lrwxrwxrwx    1 root     root            12 May 25 01:15 arch -> /bin/busybox
lrwxrwxrwx    1 root     root            12 May 25 01:15 ash -> /bin/busybox

# 會直接修改 Runtime Filesystem bundle 的檔案目錄, 這樣另一個 Container 不適合再使用這個檔案目錄 
/ # mkdir /zzz

# 檢視記憶體資訊
/ # cat /proc/meminfo | head -n 3
MemTotal:        2005980 kB
MemFree:         1766180 kB
MemAvailable:    1828016 kB

# Linux kernel 的目錄及檔案會受到保護
/ # rm /proc/meminfo
rm: remove '/proc/meminfo'? y
rm: can't remove '/proc/meminfo': Permission denied

```


- Container 裡的 root 帳號，會受到 seccomp + Linux Capabilities 的限制

---

## 問題討論

:::spoiler 請問透過 RunC 建立 Container 電腦, 需準備那些內容 ?

rootfs + config.json（一個系統檔案的目錄 + OCI 標準的 Container 設定檔）
:::

:::spoiler 請問 bb8 Container 電腦系統中, 所啟動的 sh 程序, 在 Host (alp) 系統中, 是不是只是一個程序 (Process) ?
是，在 alp 系統中，他有自己的 pid，但在 pid namespace 裡，他的 pid 是 1
:::

:::spoiler 請問 bb8 Container , 有開機程序嗎 ?
沒有，因為 Container 共用 Host OS(Kernel)，沒有自己的 OS，所以沒有開機程序。
:::

- Container 做出來的應用系統，如果夠複雜，會須要停止應用程式的關機程序

:::spoiler 如何證明 runc 沒有使用 overlay2

```bash!
$ sudo runc run bb8
/ # mkdir /zzz
/ # exit

$ ls -ld alpminifs/zzz
drwxr-sr-x 2 root root 4096 Aug 18 13:31 alpminifs/zzz
```

- 如果有使用 overlay2 的話，alpminifs 這個目錄會是 lower dir ，所以在 bb8 Container 裡面的根目錄新增一個 zzz 目錄，如果 Host 主機上的 alpminifs 目錄也有剛剛新增的 zzz 目錄的話，就可以證明 runc 沒有使用 overlay2。

:::


### 設定可用 記憶體 大小

```bash!
$ nano config.json 
......
       "linux": {
                "resources": {
                        "memory": {
                              "limit": 100000000
                        },
                        "devices": [
                                {
                                        "allow": false,
                                        "access": "rwm"
                                }
                        ]
......
```

### 測試可用 記憶體 大小

```bash!
$ screen -r bb10

$ sudo runc run bb10
/ # cat /dev/zero | head -c 85m | tail

/ # cat /dev/zero | head -c 96m | tail
Killed

$ exit

回到原先終端機
$ exit
```

# 建立多個 Container 共用 Runtime Filesystem bundle

準備 Runtime Filesystem bundle，安裝 umoci skopeo 兩個套件

```bash!
$ sudo apk update; sudo apk add umoci skopeo
```

```bash!
$ sudo skopeo --insecure-policy copy docker://quay.io/cloudwalker/busybox:latest  oci:/home/bigred/busybox-oci:latest
```

- 上網抓 `quay.io/cloudwalker/busybox:latest` 這個 image ，
- 存成 oci image 的格式，然後放在家目錄下的 `busybox-oci/`
- `--insecure-policy`，run the tool without any policy check
- `copy`，Copy an IMAGE-NAME from one location to another

檢視 `busybox-oci/` 目錄結構

```
$ cd; tree busybox-oci/
busybox-oci/
├── blobs
│   └── sha256
│       ├── 67c32e0fe983b2f71469c2343e6747e3f664af16f1b414e14a70cbaabed53da6  
│       ├── 92f8b3f0730fef84ba9825b3af6ad90de454c4c77cde732208cf84ff7dd41208
│       └── 9dcbd1eb45054c937d5cc5816bff2a9af8fa4603cb81c7ab7161dc0224ef72b9
├── index.json
└── oci-layout
```

- `blob`， Binary Large Object


```bash!
# 先檢視 index.json ，看哪一個檔案是 image 的結構說明檔
$ cat busybox-oci/index.json | jq

# 檢視 image 的結構說明檔
$ cat busybox-oci/blobs/sha256/67c32e0fe983b2f71469c2343e6747e3f664af16f1b414e14a70cbaabed53da6 | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:9dcbd1eb45054c937d5cc5816bff2a9af8fa4603cb81c7ab7161dc0224ef72b9",
    "size": 575
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:92f8b3f0730fef84ba9825b3af6ad90de454c4c77cde732208cf84ff7dd41208",
      "size": 766640
    }
  ]
}
```

- 可以得知 9dcb 這個開頭的檔案為 config ，92f8 這個開頭的檔案為壓縮檔


## 產生 Runtime Filesystem bundle

```!
# 透過 umoci 將 image unpack 成 bundle
$ sudo umoci unpack --image busybox-oci:latest busybox-oci-bundle

$ sudo tree -L 2 busybox-oci-bundle/
busybox-oci-bundle/
├── config.json
├── rootfs
│   ├── bin
│   ├── dev
│   ├── etc
│   ├── home
│   ├── root
│   ├── tmp
│   ├── usr
│   └── var
├── sha256_67c32e0fe983b2f71469c2343e6747e3f664af16f1b414e14a70cbaabed53da6.mtree
└── umoci.json

9 directories, 3 files
```



# 探索 crun

crun is a lightweight fully featured OCI runtime and C library for running containers.

“While most of the tools used in the Linux containers ecosystem are written in Go, I believe C is a better fit for a lower level tool like a container runtime. runc, the most used implementation of the OCI runtime specs written in Go, re-execs itself and use a module written in C for setting up the environment before the container process starts.”

crun is faster than runc and has a much lower memory footprint.

安裝 Crun

```bash!
$ sudo apk add crun
```

crun 與 runc 效能比較

```bash!
$ mkdir -p ~/wulin/rt/rootfs; cd ~/wulin/rt/rootfs; curl -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.16/releases/x86_64/alpine-minirootfs-3.16.2-x86_64.tar.gz; tar xvf alpine.tar.gz ;rm alpine.tar.gz; cd ..

$ rm config.json; crun spec
$ cat config.json | head -n 2
  {
        "ociVersion": "1.0.0",

$ time for i in {1..100}; do echo "exit" | sudo runc run b1 &>/dev/null; done

real    0m2.842s
....

$ time for i in {1..100}; do echo "exit" | sudo crun run b1 &>/dev/null; done

real    0m0.779s
......
```

- overlay2 檔案系統的設定標準，並沒有在 config.json 中
- crun 的速度大約是 runc 的 3 倍，以前是 4 倍，代表 runc 有在努力。


---

# Google gVisor


![](https://i.imgur.com/739PsCe.png)

- 讓 Container 擁有自己的 user-space kernel
- 它也 follow OCI Runtime spec
- 程式名稱是 run<font color=red>s</font>c ( s 是 security 的意思 )
- 原本的 application 運作時會呼叫的 system call ，被攔截，變成呼叫到 gVisor 做出來的 system call

## gVisor Architecture


![](https://i.imgur.com/TADejP4.png)

- Linux 的 Syscall 總用有 319 個，這個數字會隨著 Linux 的版本變動。
- gVisor 會寫了兩支程式，一支名字叫 Sentry ，這支程式用 55 個 system call 模擬 211 個 system call
- 當 Container 會需要動用到檔案處理相關的 syscall ，就會透過 Gofer 這支程式來模擬
- 總用有 108 個 Syscall 不支援模擬
- 當我們的 Container run 的 application 動用到特殊的 syscall ，目前就無法用到 gVisor 模擬的 syscall
- gVisor 模擬的 Syscall 還是會動用到 Host 主機的 Syscall
- 因 gVisor 怕自己模擬的 Syscall 被攻陷，所以還有啟動 Seccomp 來保護

## 安裝 gVisor

```bash!
# 看自己的系統是用甚麼架構
$ ARCH=$(uname -m); URL=https://storage.googleapis.com/gvisor/releases/release/latest/${ARCH}

$ echo $ARCH
x86_64

# 下載 x86_64 架構的 gVisor
$ wget ${URL}/runsc ${URL}/runsc.sha512 ${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512

# 檢核 runsc 及 containerd-shim-runsc-v1 這二個執行檔
$ sha512sum -c runsc.sha512 -c containerd-shim-runsc-v1.sha512
runsc: OK
containerd-shim-runsc-v1: OK

# 刪除不需要的檢查檔
$ rm -f *.sha512

# 賦予 runsc 和 containerd-shim-runsc-v1 權限
$ chmod a+rx runsc containerd-shim-runsc-v1; sudo mv runsc containerd-shim-runsc-v1 /usr/local/bin
```

## 執行 runsc

```bash!
$ rm config.json; runsc spec

# gVisor 也 follow oci 標準
$ cat config.json | head -n 10
{
    "ociVersion": "1.0.0",
    "process": {
        "user": {
            "uid": 0,
            "gid": 0
        },
        "args": [
            "sh"
        ],


$ nano config.json
{
    "ociVersion": "1.0.0",
    "process": {
        "user": {
            "uid": 0,
            "gid": 0
        },
        "args": [
            "sleep", "infinite"
        ],
.........

$ sudo runsc run bb10 &
[1] 13101

$ ps -eaf | grep -v grep | grep sleep

$ sudo runsc list; echo ""
ID          PID         STATUS      BUNDLE        CREATED                  OWNER
bb10        10514       running     /home/bigred/wulin/rt   2022-09-02T23:04:27.669266165+08:00   root


$ sudo runsc list
ID          PID         STATUS      BUNDLE                  CREATED                               OWNER
bb10        13110       running     /home/bigred/wulin/rt   2022-09-03T10:47:48.420096624+08:00   root

$ sudo runc list
ID          PID         STATUS      BUNDLE      CREATED     OWNER

$ sudo crun list
NAME PID       STATUS   BUNDLE PATH                             CREATED                        OWNER
```

- 透過 runsc run 的 Container ，在 Host 主機上無法用 `ps -eaf` 命令看到 Container run 的 Application
- `runsc` 與 `runc` 和 `crun` 之間，無法看到對方 run 的 Container


```bash!
$ sudo runsc exec bb10 whoami
root

$ sudo runsc exec bb10 ps -eaf
PID   USER     TIME  COMMAND
    1 root      0:00 sleep infinite
    2 root      0:00 ps -eaf

$ sudo runsc exec bb10 cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.14.2
PRETTY_NAME="Alpine Linux v3.14"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://bugs.alpinelinux.org/"
```

檢視與刪除 runsc Container

```bash!
$ ps -eaf | grep runsc
root      10504   6400  0 23:04 pts/0    00:00:00 runsc run bb10
root      10509  10504  0 23:04 pts/0    00:00:00 runsc-gofer --root=/var/run/runsc gofer --bundle /home/bigred/wulin/rt --spec-fd=3 --mounts-fd=4 --io-fds=5 --apply-caps=false --setup-root=false --sync-userns-fd=-1
nobody    10514  10504  0 23:04 ?        00:00:00 runsc-sandbox --root=/var/run/runsc boot --attached --product-name VMware Virtual Platform --bundle=/home/bigred/wulin/rt --cpu-num 2 --total-memory 8318603264 --attached --io-fds=3 --mounts-fd=4 --start-sync-fd=5 --controller-fd=6 --spec-fd=7 --stdio-fds=8 --stdio-fds=9 --stdio-fds=10 bb10
bigred    10818   6400  0 23:08 pts/0    00:00:00 grep runsc

$ sudo runsc delete --force bb10
```

## 建立 Overlay2 檔案系統

```bash!
$ mkdir -p ~/c1/{rootfs,upper,work}

$ cd c1

$ sudo mount -t overlay c1 -o lowerdir=/home/bigred/busybox-oci-bundle/rootfs,upperdir=/home/bigred/c1/upper,workdir=/home/bigred/c1/work  /home/bigred/c1/rootfs

$ ls -al rootfs
```

- 用 Overlay2 的原因是因為，可以讓多個 Container 同時共用 image ，且又有各自獨立的檔案系統目錄可以作業。


---

# Container<font color=red>**d**</font>

- daemon 就是 Linux 系統裡在背景執行的 program，在微軟的系統裡稱為"服務" 
- Containerd 會自己 ( 不是透過 skopeo ) 上網幫我們下載 Container image，裡面有 Container 需要的系統檔案目錄 rootfs。
- 下載完後，會將 image `unpack` 產生 bundle ，其中包含 root_file_system ( `rootfs` ) 和 `config.json`，並交給 runc 建立 Container，最後再啟動 Overlay2 ( rootfs 設為 lowerdir )，作為 Container 的檔案系統
- 注意 ! Containerd 本身是不會產生 Container 的，是透過 runc 建立的。
- containerd 可以從外部接受 `ctr` 命令
- 管理命令
	- `create` ，建立 Container
	- `Delete` ，刪除 Container
	- `List`   ，列出 Container 的清單
	- `Pause`  ，將 Container 暫停
	- `Start`  ，將 Container 啟動
- 結論 : **透過 Containerd 可以幫助我們管理多台 Container。**

---

## 安裝與啟動 Container<font color=red>**d**</font>

```bash!
# 安裝 Containerd
$ sudo apk add containerd
(1/1) Installing containerd (1.5.4-r1)
Executing busybox-1.32.1-r6.trigger
OK: 1087 MiB in 215 packages

# 檢查 Containerd 版本
$ containerd -v
containerd github.com/containerd/containerd v1.5.4 a62e1d690afa2b9b1d43f8ece3ff4483

# 啟動 Containerd 並推入背景執行
$ sudo containerd &
......
INFO[2021-05-04T20:27:58.882462798+08:00] containerd successfully booted in 0.039280s  
INFO[2021-05-04T20:27:58.887494449+08:00] Start event monitor                          
INFO[2021-05-04T20:27:58.887720795+08:00] Start snapshots syncer                       
INFO[2021-05-04T20:27:58.887902496+08:00] Start cni network conf syncer                
INFO[2021-05-04T20:27:58.888012074+08:00] Start streaming server 

```

---

## Container<font color=red>**d**</font> 原生管理命令 - `ctr`

下載 Container Image

```bash!
$ sudo ctr image pull quay.io/cloudwalker/busybox:latest
```

- `quay.io` 這個網站是由 RedHat (紅帽)維護的，而講到紅帽它是 Linux 系統的大哥，他有培養一個小弟: Centos Linux 作業系統（免費版）
- `latest` 為最新版

檢查有沒有下載成功

```bash!
$ ctr image ls -q
quay.io/cloudwalker/busybox:latest
```

啟動 Container

```bash!
$ sudo ctr run --tty quay.io/cloudwalker/busybox:latest b1 sh
time="2022-05-25T10:18:14.200606732+08:00" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runtime=io.containerd.runc.v2 type=io.containerd.event.v1
time="2022-05-25T10:18:14.200719541+08:00" level=info msg="loading plugin \"io.containerd.internal.v1.shutdown\"..." runtime=io.containerd.runc.v2 type=io.containerd.internal.v1
time="2022-05-25T10:18:14.200728474+08:00" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io.containerd.runc.v2 type=io.containerd.ttrpc.v1
time="2022-05-25T10:18:14.201096167+08:00" level=info msg="starting signal loop" namespace=default path=/run/containerd/io.containerd.runtime.v2.task/default/b1 pid=40768 runtime=io.containerd.runc.v2
/ # hostname
alp
/ # hostname bb8
hostname: sethostname: Operation not permitted
/ # whoami
root
/ #
```

- Container他在運作時，他沒有開機程序，在跑時只有一個 Process，在 Container 裡面即使是 root 身份，在權限上還是會受到限制。

:::spoiler 提問：`ls -al /` 裡面的檔案內容從哪裡來？
```
/ # ls -al /
total 44
drwxr-xr-x    1 root     root          4096 May 25 02:18 .
drwxr-xr-x    1 root     root          4096 May 25 02:18 ..
drwxr-xr-x    2 root     root         12288 May 17  2021 bin
drwxr-xr-x    5 root     root           340 May 25 02:18 dev
drwxr-xr-x    3 root     root          4096 May 17  2021 etc
drwxr-xr-x    2 nobody   nobody        4096 May 17  2021 home
dr-xr-xr-x  185 root     root             0 May 25 02:18 proc
drwx------    1 root     root          4096 May 25 02:18 root
drwxr-xr-x    2 root     root            40 May 25 02:18 run
dr-xr-xr-x   13 root     root             0 May 25 02:18 sys
drwxrwxrwt    2 root     root          4096 May 17  2021 tmp
drwxr-xr-x    3 root     root          4096 May 17  2021 usr
drwxr-xr-x    4 root     root          4096 May 17  2021 var
```
:::

答：透過 `ctr` 指令從 `quay.io` 網站下載(`pull`)名為 busybox 的 Container image，rootfs （系統所需的檔案目錄）就是從這邊來的


透過`sudo ctr container list`來看 Container 的清單

```
$ sudo ctr container list
CONTAINER    IMAGE                                 RUNTIME
b1           quay.io/cloudwalker/busybox:latest    io.containerd.runc.v2
```

刪除 Container 的指令

```
$ sudo ctr container rm b1

$ sudo ctr container list
CONTAINER    IMAGE    RUNTIME
```


```
$ sudo ctr image remove quay.io/cloudwalker/busybox:latest
quay.io/cloudwalker/busybox:latest
$ sudo ctr image ls -q
```


## Containerd 設定檔

```bash!
$ sed -n 6,7p /etc/containerd/config.toml
root = "/var/lib/containerd"
state = "/run/containerd"
```

- `/var/lib/containerd/io.containerd.content.v1.content/blobs/sha256/`
    - 這目錄, 存儲 images 的 layer 壓縮檔 及 manifest 檔
- `/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots`
    - 這目錄, 存放 overlay2 檔案系統目錄
- `/run/containerd/io.containerd.runtime.v2.task/default`
    - 這目錄, 存放 container 的工作目錄

```bash!
$ sudo ls -al /var/lib/containerd/io.containerd.content.v1.content/blobs/sha256/
total 8
drwxr-xr-x 2 root root 4096 Aug 17 14:10 .
drwxr-xr-x 3 root root 4096 Aug 17 12:33 ..

# 拉 image
$ sudo ctr image pull k8s.gcr.io/pause:3.5

# 檢查
$ sudo ls -al /var/lib/containerd/io.containerd.content.v1.content/blobs/sha256/
total 312
drwxr-xr-x 2 root root   4096 Sep  3 14:47 .
drwxr-xr-x 3 root root   4096 Sep  3 14:19 ..
-r--r--r-- 1 root root 296517 Sep  3 14:47 019d8da33d911d9baabe58ad63dea2107ed15115cca0fc27fc0f627e82a695c1
-r--r--r-- 1 root root   3472 Sep  3 14:47 1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07
-r--r--r-- 1 root root    526 Sep  3 14:47 369201a612f7b2b585a8e6ca99f77a36bcdbd032463d815388a96800b63ef2c8
-r--r--r-- 1 root root    901 Sep  3 14:47 ed210e3e4a5bae1237f1bb44d72a05a2f1e5c6bfe7a7e73da179e2534269c459
```

## 建立與檢視 Container

run Container

```bash!
$ sudo ctr run -d k8s.gcr.io/pause:3.5 b1
```

檢視 Container 狀態

```bash!
$ sudo ctr c list
CONTAINER    IMAGE                   RUNTIME
b1           k8s.gcr.io/pause:3.5    io.containerd.runc.v2
```

透過 `ps -eaf` 觀察

```bash!
$ ps -eaf
UID         PID   PPID  C STIME TTY          TIME CMD
root       5076      1  0 14:20 pts/0    00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace default -id b1 -address /run/containerd/containerd.sock
65535      5098   5076  0 14:20 ?        00:00:00 /pause
```

- `/usr/bin/containerd-shim-runc-v2`，Containerd 的 shim (2022/09/03/14:17)


看 Overlay2 檔案系統有沒有被啟動

```bash!
$ mount | grep overlay
overlay on /run/containerd/io.containerd.runtime.v2.task/default/b1/rootfs type overlay (rw,relatime,lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/1/fs,upperdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/2/fs,workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/2/work)
```

- 答 : 有的 !

停止和刪除 Container and Image

```bash!
$ sudo ctr task attach b1
^CShutting down, got signal: Interrupt

$ sudo ctr container list
CONTAINER    IMAGE                   RUNTIME
b1           k8s.gcr.io/pause:3.5    io.containerd.runc.v2

$ sudo ctr container rm b1

$ sudo ctr image remove k8s.gcr.io/pause:3.5
k8s.gcr.io/pause:3.5
```

用同一片 image 建利多台 Container ，觀察 Overlay2 

```bash!
$ sudo ctr run -d k8s.gcr.io/pause:3.5 b1

$ sudo ctr run -d k8s.gcr.io/pause:3.5 b2

$ sudo ctr c list
CONTAINER    IMAGE                   RUNTIME
b1           k8s.gcr.io/pause:3.5    io.containerd.runc.v2
b2           k8s.gcr.io/pause:3.5    io.containerd.runc.v2

$ mount | grep overlay
overlay on /run/containerd/io.containerd.runtime.v2.task/default/b1/rootfs type overlay (rw,relatime,lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs,upperdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/4/fs,workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/4/work)
overlay on /run/containerd/io.containerd.runtime.v2.task/default/b2/rootfs type overlay (rw,relatime,lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs,upperdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/5/fs,workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/5/work)

$ sudo ls -l /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs
total 668
-rwxr-xr-x 1 root root 682696 Mar 16  2021 pause
```



---

# Docker-Engine

![](https://i.imgur.com/sF5cEZw.png)


- Docker 和 Container 是在同一層的，與虛擬化技術的 Hyperviser 不同，他並不會透過 Docker 與底層的 Host OS (Kernel) 和硬體周邊溝通，會直接和 Host OS (kernel) 做溝通。


---

## Docker 軟體貨櫃運作架構

![](https://i.imgur.com/Dkc7EBl.png)

- `Docker engine` 會上網下載 `Docker image`，`Containerd` 會將這個 image `unpack` 變成 `bundle` ，runc 會根據 `bundle` 裡面的 `config.json` 和 `rootfs` 將 Container 做出來。
- Container 建好後，`runc` 不再運作，`Containerd` 會去問 `shim`，container 的狀態， `shim` 這個 deamon 程式會負責監控 container 起來以後的 log 檔，然後回報給 `Containerd`，最後 `Containerd` 再回報給 `Docker engine`
- Docker 命令透過 `Docker Engine` ( Dockerd ) 管理 Docker Image 、 bridge (Network) 和 overlay2 堆疊型檔案系統 ，再透過 `Containerd` 來管理 Container， 而 `Containerd` 又透過 runc 來建立 Container ， 其中 runc 會透過 Chroot 、 Namespace ( UTS 、 PID 和 Net ...等)、CGroup 來建立與隔離出 Container。

![](https://i.imgur.com/1YpYoSH.png)


---

## 安裝 Docker


```bash!
$ sudo apk add docker
(1/10) Installing runc (1.1.2-r0)              ## 建立 Container
(2/10) Installing containerd (1.6.4-r1)        ## 負責管理 Container
(3/10) Installing containerd-openrc (1.6.4-r1)
(4/10) Installing ip6tables (1.8.8-r1)
(5/10) Installing ip6tables-openrc (1.8.8-r1)
(6/10) Installing tini-static (0.19.0-r0)
(7/10) Installing docker-engine (20.10.16-r0)  ## Docker 的 Deamon
(8/10) Installing docker-openrc (20.10.16-r0)
(9/10) Installing docker-cli (20.10.16-r0)
(10/10) Installing docker (20.10.16-r0)        ## 管理者用的命令
Executing docker-20.10.16-r0.pre-install
Executing busybox-1.35.0-r13.trigger
OK: 1263 MiB in 262 packages
```


設定 Docker Daemon 自動啟動

```
$ sudo rc-update add docker boot
```

將 bigred 帳號加入 docker 群組後, 就不需使用 sudo 命令執行 docker

```
$ sudo addgroup bigred docker
```

重新開機 (一定要執行)

```
$ sudo  reboot
```

---

## 檢視 Docker 運作資訊


顯示 Docker 版本

```
$ docker info
Client:
 Context:    default
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.16  ## 2020年10月，修改16次
 Storage Driver: overlay2
```

顯示 Docker 版本

```
$ docker info
.........
 Images: 0
 Server Version: 20.10.7
 Storage Driver: overlay2
.......
```

沒有任何 Container 執行時, 只有 dockerd 及 containerd 這二個 Daemon 在運作

```
$ ps aux | grep -v grep | grep dockerd
 3904 root      0:00 supervise-daemon docker --start --retry TERM/60/KILL/10 --stderr /var/log/docker.log --stdout /var/log/docker.log /usr/bin/dockerd --
 3906 root      0:00 /usr/bin/dockerd

$ ps aux | grep -v grep | grep containerd
 4070 root      0:06 containerd --config /var/run/docker/containerd/containerd.toml --log-level info
```


---

## 檢視 Docker 運作資訊 - 執行 Container

```
$ docker run --rm -d quay.io/cloudwalker/alpine sleep 120
```

output : 

```
## 下面這行不是錯誤訊息，是 docker 檢測到 Host 電腦沒有 quay.io/cloudwalker/alpine 這張光碟片
unable to find image 'quay.io/cloudwalker/alpine:latest' locally
latest: Pulling from cloudwalker/alpine
540db60ca938: Pull complete
Digest: sha256:def822f9851ca422481ec6fee59a9966f12b351c62ccb9aca841526ffaa9f748
Status: Downloaded newer image for quay.io/cloudwalker/alpine:latest
```

-  `docker run` ，產生一台 Container (貨櫃電腦)
- 到 RatHat 公司的網站 `quay.io` 下載 alpine linux 系統的 Image
- rootfs 裡面一定有 `sleep` 命令，讓系統去執行
- `--rm`，只要發現 Container 關閉，就會刪除 Container
- `-d` ，讓 Container 跑到背景執行，相當於在一個 process 加一個 `&`

檢查 Container 的狀態
```
$ docker ps -a
CONTAINER ID   IMAGE                        COMMAND       CREATED         STATUS         PORTS     NAMES
9fc18d849990   quay.io/cloudwalker/alpine   "sleep 120"   6 seconds ago   Up 5 seconds             nice_proskuriakova
```

看 `containerd-shim` 有沒有偷懶

```bash!
$ ps -eaf | grep -v grep | grep 'containerd-shim'
root       8966      1  0 15:17 pts/0    00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace default -id b1 -address /run/containerd/containerd.sock
```



兩分鐘後 `--rm` 指令生效，刪除 Container
```
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


## Docker 橋接網路


Docker 原生橋接網路架構圖


![](https://i.imgur.com/mfDQyFj.png)

![](https://i.imgur.com/R6rnBjU.png)

```bash!
$ docker run --rm -it quay.io/cloudwalker/alpine
/ # ifconfig eth0
eth0   Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          ........
/ # route -n
Kernel IP routing table
Destination     Gateway      Genmask      Flags Metric Ref    Use Iface
0.0.0.0           172.17.0.1   0.0.0.0       UG    0      0        0     eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0     0     eth0

/ # cat /etc/resolv.conf
search localdomain
nameserver 192.168.179.2
/ # ping -c 2 www.hinet.net
PING www.hinet.net (203.66.32.65): 56 data bytes
64 bytes from 203.66.32.65: seq=0 ttl=127 time=3.773 ms
64 bytes from 203.66.32.65: seq=1 ttl=127 time=4.372 ms
/ # exit

$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242ab7e7035       no


$ ifconfig docker0
docker0   Link encap:Ethernet  HWaddr 02:42:AB:7E:70:35
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:5 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:314 (314.0 B)  TX bytes:874 (874.0 B)
          
# 只要 source (來源)是 172.17.0.0/16 這個 net.id 的封包，destination (目的地)是 anywhere 的，都會偽裝成 eth0 的 ip 位址上網
$ sudo iptables -t nat -L | sed -n 12,14p
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        anywhere

# 透過 docker run 的命令，建立一台 Container，步驟有
$ docker run --rm -itd quay.io/cloudwalker/busybox sh

```


透過 docker 做出來的 Container 內定與 Docker Host (ALP) 相同 DNS IP

```bash!
# Docker Host DNS IP
$ cat /etc/resolv.conf 
nameserver 172.16.119.1

$ docker run --rm quay.io/quay/busybox cat /etc/resolv.conf
nameserver 172.16.119.1
```

- If the container cannot reach any of the IP addresses you specify, Google’s public DNS server 8.8.8.8 is added, so that your container can resolve internet domains.


**Published ports**

```bash!
# 透過 docker run 建立一台 Container
# image 後面如果沒放命令的話，就會執行內定的命令
$ docker run --rm --name n1 -p 8080:80 -d quay.io/cloudwalker/nginx

$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
ad82bd0803fe        nginx               "nginx -g 'daemon of…"   47 seconds ago      Up 46 seconds       0.0.0.0:8080->80/tcp   n1

$ curl http://localhost:8080 
.........
<p><em>Thank you for using nginx.</em></p>
</body>
</html>

$ docker exec n1 ifocnfig
OCI runtime exec failed: exec failed: unable to start container process: exec: "ifocnfig": executable file not found in $PATH: unknown

$ docker exec n1 `which ifconfig`
OCI runtime exec failed: exec failed: unable to start container process: exec: "/sbin/ifconfig": stat /sbin/ifconfig: no such file or directory: unknown

$ docker exec n1 hostname -i
172.17.0.2

$ docker inspect n1 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",

$ docker stop n1
```

![](https://i.imgur.com/Hiy4Kq3.png)


- Container 裡面只要存放 process 和它需要的相依檔就好，不需要一些用不到的命令。



---


# 透過 docker 將 MySQL Server 貨櫃化

![](https://i.imgur.com/ARrYqHB.png)

- 以後出去工作，會需要將公司傳統的應用系統貨櫃化，
- 網站伺服器在作業時，不一定會使用 root 帳號作業


```bash!
$ docker run --name m1 -e MYSQL_ROOT_PASSWORD=bigred -d -p 3306:3306 mysql/mysql-server:latest
```

- `-e`，設定環境變數，將 `MYSQL_ROOT_PASSWORD` 設為 `bigred`

將 image 做備份

```bash!
$ docker save mysql/mysql-server:latest -o mysql.tar

$ ls -l
-rw------- 1 bigred bigred 438820352 Aug 23 10:37 mysql.tar

# 將 mysql Server 的 Container 啟動
$ docker run --name m1 -e MYSQL_ROOT_PASSWORD=bigred -d -p 3306:3306 mysql/mysql-server:latest

# 進入 Container
bigred@alp:~/ $ docker exec -it m1 sh
# 登入 root 帳號
sh-4.4# mysql -uroot -pbigred
.........
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# 新增 bigboss 使用者，設定它可以透過任何主機登入，密碼設為 bigboss
mysql> CREATE USER 'bigboss'@'%' IDENTIFIED BY 'bigboss';
Query OK, 0 rows affected (0.01 sec)

# 因 8.0 版本的 密碼格式是新的，所以用舊密碼格式 mysql_native_password 重設密碼為 bigboss
mysql> ALTER USER 'bigboss' IDENTIFIED WITH mysql_native_password BY 'bigboss';

# 將以上兩行命令放在一起
mysql> CREATE USER 'bigboss'@'%' IDENTIFIED WITH mysql_native_password BY 'bigboss';
Query OK, 0 rows affected (0.00 sec)

# 將任何 Database 和 Table 的所有權限都給 bigboss 這個帳號在任何主機登入時
mysql> GRANT ALL PRIVILEGES ON *.* TO 'bigboss'@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)

# 檢查有無設定成功
mysql> SELECT user,host FROM mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| bigboss          | %         |
| healthchecker    | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)

# 建 Database
$ create database brave;

# 進入 Database
$ use brave;

# 建 customer 資料表
CREATE TABLE customer (
    ID int NOT NULL,
    NAME varchar(100) NOT NULL,
    AGE int,
    ADDRESS varchar(255),
    SALARY DECIMAL(10,2),
    PRIMARY KEY (ID)
);

# 新增資料進去
INSERT INTO customer
VALUES (1,'Ramesh',32,'Ahmedabad','2000.00'),
(2,'Khilan',25,'Delhi','1500.00'),
(3,'kaushik',23,'kota','2000.00'),
(4,'Chaitali',25,'Mumbai','6500.00'),
(5,'Hardik',27,'Bhopal','8500.00'),
(6,'Komal',22,'MP','4500.00'),
(7,'Muffy',24,'Indore','10000.00');

# 顯示 customer 的所有資料
MySQL [brave]> select * from customer;
+----+----------+------+-----------+--------+
| ID | NAME     | AGE  | ADDRESS   | SALARY |
+----+----------+------+-----------+--------+
|  1 | Ramesh   |   32 | Ahmedabad |   2000 |
|  2 | Khilan   |   25 | Delhi     |   1500 |
|  3 | kaushik  |   23 | kota      |   2000 |
|  4 | Chaitali |   25 | Mumbai    |   6500 |
|  5 | Hardik   |   27 | Bhopal    |   8500 |
|  6 | Komal    |   22 | MP        |   4500 |
|  7 | Muffy    |   24 | Indore    |  10000 |
+----+----------+------+-----------+--------+
7 rows in set (0.002 sec)

mysql> exit
Bye

sh-4.4# exit
exit
```


![](https://i.imgur.com/rA2tuf9.png)

###### tags: `系統工程`




































