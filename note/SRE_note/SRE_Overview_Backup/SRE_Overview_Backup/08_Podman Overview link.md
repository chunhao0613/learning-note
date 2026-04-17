# Podman Overview

:::warning

:::spoiler 目錄 (  :mag_right: 點我，可展開 )

[TOC]

:::



---


## Docker VS Podman


![](https://i.imgur.com/hCMoTtE.png)


**Docker**
- Docker deamon 會開啟 Internet Socket，當我們在 Docker CLI 下達指令時，就有可能會透過這個 Socket，資安漏洞
- Docker deaman 建立 image、上網抓 image 內定執行的帳號為 root 作業

**Podman**
- 無 deaman
- `unshare` 的進階版，它的底層運作就是透過 `unshare` 這個命令加參數，做出 Container
- - 運作時不需 root 帳號
- 透過 **skopeo** 到 `docker.io` / `quay.io` ...等 光片中心，幫我們下載 image
    - 在資訊界的 image 有兩個版本，skopeo 兩者都可以處理
        - docker 版本的 image
        - OCI 組織的 image
    - skopeo 也能幫我們 push 到 image 中心
    - 還能排時程將 image 從 一個光片中心移到另一個 image 中心
- **buildah** 能幫我們製作 image ，且也不須 root 帳號作業


## 在 Alpine Linux 安裝 Podman


```bash=
在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh bigred@<alp.podman IP>

$ sudo apk update; sudo apk upgrade

$ sudo apk add podman
(1/29) Installing conmon (2.1.2-r0)
(2/29) Installing yajl (2.1.0-r4)
(3/29) Installing crun (1.4.5-r0)
(4/29) Installing ip6tables (1.8.8-r1)
(5/29) Installing ip6tables-openrc (1.8.8-r1)
(6/29) Installing libslirp (4.7.0-r0)
(7/29) Installing slirp4netns (1.2.0-r0)     ---> 撥號網路
(14/29) Installing netavark (1.0.3-r0)       ---> 虛擬橋接網路
(15/29) Installing aardvark-dns (1.0.3-r0)   ---> DNS Server
........
(28/29) Installing podman (4.1.0-r1)
.......

$ podman -v
podman version 4.1.0

$ sudo reboot
```

- podman 內建 crun
- 重要 ! 安裝完 podman 一定要重開機

### 小補充

如果不是老師是乾淨的 Alpine Linux 系統

要執行以下命令

```bash!
# enable the cgroups service, consider enabling cgroups v2.
$ rc-update add cgroups
$ rc-service cgroups start
```

## 建立 Container

```bash=
# 檢視 podman 有無 daemon
$ ps aux | grep -v grep | grep podman

# 第 21 行，新增 image 的下載點
$ sudo nano /etc/containers/registries.conf
......
unqualified-search-registries = ["docker.io","quay.io"]
......

# podman run 前面加 sudo ，代表我們用 root 的身分建立 Container
$ sudo podman run quay.io/podman/hello
........
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

.........

```

## 檢視 與 刪除 Container

```bash!
# 檢視 Container
$ sudo podman ps -a
CONTAINER ID  IMAGE                        COMMAND               CREATED         STATUS                     PORTS       NAMES
f8020fa8e94d  quay.io/podman/hello:latest  /usr/local/bin/po...  11 minutes ago  Exited (0) 11 minutes ago              affectionate_visvesvaraya

# 刪除 Container
bigred@alp:~$ sudo podman rm f8020
f8020fa8e94dc99f024e611b65e31b739032747253366b4d5f2af4b34da27fd0
```


- **pdoman 前面加 sudo ，就代表用 root 作業，這種運作架構稱為 rootful mode ，事實上有 95% 就等於 docker 這個命令**


## 撰寫 Dockerfile

```bash=
# 建立工作目錄
$ mkdir ~/fbs

$ nano ~/fbs/Dockerfile
FROM quay.io/cloudwalker/alpine:latest

RUN apk --update add ca-certificates bash curl \
    && adduser -h /opt/app -D app \
    && curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash

COPY entrypoint /opt/app/entrypoint
RUN chmod a+x /opt/app/entrypoint

VOLUME /srv
EXPOSE 4000
USER app
WORKDIR /opt/app

ENTRYPOINT [ "/opt/app/entrypoint" ]
```

- image 主要是要 run filebrowser ( 網站型的檔案儲存伺服器 )
- `ENTRYPOINT` 宣告 Container 只能 run 後面的命令


## 建立與測試 alp.fbs image


```bash!
$ nano ~/fbs/entrypoint
#!/bin/bash
filebrowser config init --port 4000 --address "" --baseurl "" --log "stdout" --root="/srv" --auth.method='noauth' --commands "" --lockPassword --perm.admin=false --perm.create=false --perm.delete=false --perm.execute=false --perm.modify=false --perm.rename=false --signup=false
filebrowser users add anonymous "anonymous"
filebrowser

$ sudo podman build -t alp.fbs ~/fbs/

$ sudo podman  run --name f1 -d -p 80:4000 --volume /tmp:/srv:ro alp.fbs

$ curl -I localhost/
.....
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Thu, 03 Feb 2022 13:56:12 GMT
Content-Length: 14

$ sudo podman rm -f f1
```

- `--volume /tmp:/srv:ro` | 將 host node 的 /tmp 目錄掛載到 Container 的 /srv 目錄，並且 read only ( 唯讀 )

## 建立 MySQL 的 Container

編輯 Dockerfile

```dockerfile=
FROM docker.io/mysql/mysql-server

ENV MYSQL_ROOT_PASSWORD bigred
ADD myuser.sql /docker-entrypoint-initdb.d

EXPOSE 3306
```

- `/docker-entrypoint-initdb.d` ，當從 `docker.io/mysql/mysql-server` 這個 image 做出來的 Container ，在建立時，會到這個目錄去找有沒有可以執行的 `SQL` 命令檔，如果有就會在等等的 SQL Server 裡面 run 一遍 `SQL` 的命令檔


---


# Podman Rootful Container


## Podman Rootful 運作架構


```bash=
在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh bigred@<ALP.Podman IP>

$ alias docker='sudo podman'

$ docker run --name a1 -it -d quay.io/cloudwalker/alpine sh
fc115717b17c4288b3618368f216d5fa5c170e7b08bcd3ac26fd1a5cb3717b01

$ ps aux | grep ' sh$'
root       12629  0.0  0.0   1684   944 pts/0    Ss+  22:03   0:00 sh

$ docker exec -it a1 whoami
root

$ docker rm -f a1
```

- 在 Podman Rootful 運作架構產生 Container 的使用者，與 Container 的內定使用者，帳號都是 root


## 撰寫 Dockerfile

```bash=
$ mkdir -p ~/fbs

$ nano ~/fbs/Dockerfile
FROM quay.io/cloudwalker/alpine:latest

RUN apk --update add ca-certificates bash curl \
    && adduser -h /opt/app -D app \
    && curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash

COPY entrypoint /opt/app/entrypoint
RUN chmod a+x /opt/app/entrypoint

VOLUME /srv
EXPOSE 4000
USER app
WORKDIR /opt/app

ENTRYPOINT [ "/opt/app/entrypoint" ]

```

## 建立與測試 alp.fbs image

```bash=
$ nano ~/fbs/entrypoint
#!/bin/bash
filebrowser config init --port 4000 --address "" --baseurl "" --log "stdout" --root="/srv" --auth.method='noauth' --commands "" --lockPassword --perm.admin=false --perm.create=false --perm.delete=false --perm.execute=false --perm.modify=false --perm.rename=false --signup=false
filebrowser users add anonymous "anonymous"
filebrowser

$ sudo podman build -t alp.fbs ~/fbs/

$ sudo podman  run --name f1 -d -p 80:4000 --volume /tmp:/srv:ro alp.fbs

$ curl -I localhost/
.....
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Thu, 03 Feb 2022 13:56:12 GMT
Content-Length: 14

$ sudo podman rm -f f1
```

## Podman Rootful Container Network


++**內定網路架構**++

```bash=
$ docker run --name b1 -itd quay.io/cloudwalker/alpine sh
$ docker run --name b2 -itd quay.io/cloudwalker/alpine sh

$ docker exec b1 hostname -i; docker exec b2 hostname -i  
10.88.0.7
10.88.0.8

$ docker exec b2 ping -c 2 10.88.0.7
PING 10.88.0.5 (10.88.0.8): 56 data bytes
64 bytes from 10.88.0.5: seq=0 ttl=42 time=1.916 ms
64 bytes from 10.88.0.5: seq=1 ttl=42 time=0.122 ms

$ docker exec b2 ping -c 2 b1
ping: bad address 'b1'

$ docker exec b1 cat /etc/resolv.conf 
search localdomain
nameserver 192.168.188.2

$ docker rm -f b1 b2

```

- Podman 的 DNS Server 與 Docker 內定的一樣


```bash=
$ brctl show
bridge name	bridge id		STP enabled	interfaces
podman0	8000.86e232192730	no		

$ ifconfig podman0
podman0   Link encap:Ethernet  HWaddr 2A:1F:75:70:E2:54
          inet addr:10.88.0.1  Bcast:10.88.255.255  Mask:255.255.0.0
          ..........

$ sudo iptables -t nat -L
... 以上省略
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
NETAVARK-HOSTPORT-MASQ  all  --  anywhere             anywhere
NETAVARK-1D8721804F16F  all  --  10.88.0.0/16         anywhere

Chain NETAVARK-1D8721804F16F (1 references)
target     prot opt source               destination
ACCEPT     all  --  anywhere             10.88.0.0/16
MASQUERADE  all  --  anywhere            !base-address.mcast.net/4

Chain NETAVARK-DN-1D8721804F16F (1 references)
target     prot opt source               destination
NETAVARK-HOSTPORT-SETMARK  tcp  --  10.88.0.0/16         anywhere             tcp dpt:http
NETAVARK-HOSTPORT-SETMARK  tcp  --  localhost            anywhere             tcp dpt:http
DNAT       tcp  --  anywhere             anywhere             tcp dpt:http to:10.88.0.5:4000
... 以下省略
```


## Rootful Container 自建網路架構



```bash!
# 透過 Podman network 自建網路、設定 Network ID 和 Gateway 
$ docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet

# 第一次建立自建的 Podman 網路, 不會建立 虛擬橋接器, 需產生連接 mynet 的 Container 才會產生 虛擬橋接器
$ brctl show
bridge name	bridge id		STP enabled	interfaces
podman0	             8000.86e232192730	no   

# 建立 Container 時，可自訂網路、 ip 位址
$ docker run --rm --net mynet  --ip=192.168.166.3 --name c1 -h cg61 -d alpine sleep 360

$ docker run --rm --net mynet  --ip=192.168.166.4 --name c2 -h cg62 -d alpine sleep 360

$ docker network ls
NETWORK ID    NAME        DRIVER
4af7f2ee4ac9  mynet       bridge
2f259bab93aa  podman      bridge

$ brctl show
bridge name	bridge id		STP enabled	interfaces
podman0	8000.4e7b314d9b71	no	
podman1	8000.f695873fdba7	no	veth0e1577bc
						vethdef97e4b

$ docker exec c2 ping -c 2 192.168.166.3
PING 192.168.166.6 (192.168.166.6): 56 data bytes
64 bytes from 192.168.166.6: seq=0 ttl=42 time=0.083 ms
64 bytes from 192.168.166.6: seq=1 ttl=42 time=0.125 ms

$ docker exec c2 ping -c 2 cg61
ping: bad address 'cg61'

$ docker stop c1 c2
```

- 在 podman 自建網路，可以 ping 主機名稱


## Rootful Macvlan 網路


- 相當於 VMware 的 bridge 模式


```bash=
$ docker network create -d macvlan -o parent=eth0 --subnet 192.168.61.0/24 newnet

* Create a Macvlan based network using the host interface eth0. Macvlan networks can only be used as root.

$ docker network ls
NETWORK ID    NAME        DRIVER
5d517fc03050  mynet       bridge
5b828fdc0811  newnet      macvlan
2f259bab93aa  podman      bridge

$ docker run --rm --net newnet --name c3 -h lcs12 -d --ip=192.168.61.222 quay.io/cloudwalker/alpine sleep 360

$ docker exec c3 hostname -i
192.168.61.222

$ docker exec c3 ping -c 2 192.168.61.1
PING 192.168.61.1 (192.168.61.1): 56 data bytes
64 bytes from 192.168.61.1: seq=0 ttl=42 time=0.171 ms
64 bytes from 192.168.61.1: seq=1 ttl=42 time=0.543 ms

# 目前的預設 Container 無法 ping 到 host 主機，但能 ping 到其他任何一台同 Network ID 的電腦
$ ping 192.168.61.122
PING 192.168.61.122 (192.168.61.122) 56(84) bytes of data.
From 192.168.61.152 icmp_seq=1 Destination Host Unreachable
From 192.168.61.152 icmp_seq=2 Destination Host Unreachable
From 192.168.61.152 icmp_seq=3 Destination Host Unreachable
^C
--- 192.168.61.122 ping statistics ---
4 packets transmitted, 0 received, +3 errors, 100% packet loss, time 3086ms


$ docker run --rm --net newnet --name c3 -h lcs12 -d --ip=192.168.61.222 quay.io/cloudwalker/alpine sleep 360
602af88e4fd12e2f1d3a3e6c11488b60647422d261fff0084e4422eb2f931200

$ docker run --rm --net newnet --name c4 -h lcs13 -d --ip=192.168.61.111 quay.io/cloudwalker/alpine sleep 360
af91e68f556402c7d1bca9a959e14cf98c3842ce1ec4d1f8fd30884ec956cbef

$ docker exec c4 ping -c 2 192.168.61.222
PING 192.168.61.222 (192.168.61.222): 56 data bytes
64 bytes from 192.168.61.222: seq=0 ttl=42 time=0.095 ms
64 bytes from 192.168.61.222: seq=1 ttl=42 time=0.046 ms

--- 192.168.61.222 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.046/0.070/0.095 ms
```

## Container 運算資源管理

Application Container 記憶體管理

```bash=
$ docker run --rm --name c1 -d -it quay.io/cloudwalker/alpine
$ docker stats c1
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O   PIDS
982a40d8711f  c1          --          1.446MB / 2.054GB  0.07%       726B / 54B  -- / --     1           4.084301ms  0.01%

按 Ctrl + C 停止監視

$ docker stop c1

$ docker run --rm --name c1 -m 300m -d -it quay.io/cloudwalker/alpine

$ docker stats c1 
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
7883c220aa4f   c1        0.00%     1.137MiB / 300MiB   0.38%     946B / 0B   0B / 0B     1
^C 

$ docker stop c1 
```

- Podman 與 docker 都能規範 Container 的運算資源


## Application Container CPU 管理


```bash=
新增二個 Container, 均指定使用第二核心, c1 設定使用 20% CPU, c2 設定使用 30% CPU
$ docker run --rm --name c1 --cpuset-cpus="1" --cpus="0.2"  -itd  quay.io/quay/busybox  yes

$ docker run --rm --name c2 --cpuset-cpus="1" --cpus="0.3"  -itd  quay.io/quay/busybox  yes


$ docker stats c1 c2
CONTAINER ID  NAME    CPU %       MEM USAGE / LIMIT   MEM %       NET I/O       BLOCK I/O        PIDS
18f8dc0418e5   c1         19.82%      472KiB / 5.306GiB     0.01%        0B / 0B        0B / 0B            1
CONTAINER ID  NAME     CPU %       MEM USAGE / LIMIT   MEM %        NET I/O       BLOCK I/O      PIDS
24f07c022cc0    c2          31.11%      500KiB / 5.306GiB    0.01%         0B / 0B        0B / 0B          1
^C

$ docker stop c1 c2
```


---


## Podman Container 帳號

破壞 Alpine Container


```bash=
$ docker run --rm -it quay.io/cloudwalker/alpine sh
/ # which rm
/bin/rm

/ # echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

/ # rm -r -f /* &>/dev/null
/ # ls -al /
sh: ls: not found
/ # exit
```

防止破壞 Alpine Container

```bash=
$ docker run --rm -it quay.io/cloudwalker/alpine sh
/ # echo $'#/bin/sh
args="echo $@ | tr -s \' \'"
argn="$#"
last=$($args | cut -d \' \' -f$(( $argn )))

for d in /bin /usr /var /etc /sbin /proc /dev /
do
  echo "$last" | grep $d &>/dev/null
  [ "$?" == "0" ] && echo "Oops" && exit 1
done

/bin/rm $@ ' > /usr/local/bin/rm; chmod +x /usr/local/bin/rm

/ # rm -r /
Oops
/ # rm -r /bin
Oops
/ # exit
```

## 指定 Container 執行的帳號


```bash=
# 取得 ALP 的 bigred 帳號 ID
$ echo $UID
1000

# 指定用 bigred 帳號執行以下 Container
$ docker run --rm -it -d --name u1 --user 1000 quay.io/cloudwalker/alpine

$ ps aux | grep -v grep | grep "/bin/sh"
bigred    13298  0.0  0.0   1692  1080 pts/0    Ss+  11:58   0:00 /bin/sh

$ docker exec -it u1 sh
/ $ whoami
whoami: unknown uid 1000

[註] 因 1000 這 UID, 在 Container 系統中找不到, 所以你會看到 "unknown uid 1000", 
以致於許多 Linux 命令無法執行, 例如 passwd 

/ $ rm -rf / &>/dev/null

/ $ ls /
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr

/ $ sudo reboot
sh: sudo: not found

/ $ exit 

$ docker stop u1
```



---


# Podman Rootless Container


- 執行 podman 時，前面不加 sudo 


```bash!
$ podman run -it quay.io/cloudwalker/alpine
ERRO[0000] cannot find UID/GID for user bigred: No subuid ranges found for user "bigred" in /etc/subuid - check rootless mode in man pages.
WARN[0000] Using rootless single mapping into the namespace. This might break some images. Check /etc/subuid and /etc/subgid for adding sub*ids if not using a network user
... 以下省略

$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```


```bash=
#在 ALP.Podman 終端機, 建立 rbean 及 gbean 帳號
$ echo -e "rbean\nrbean" | sudo adduser -s /bin/sh -h /home/rbean rbean
$ echo -e "gbean\ngbean" | sudo adduser -s /bin/sh -h /home/gbean gbean

# 宣告 bigred 還需要額外的 uid，從 10 萬開始，可以用 65535 個
# 宣告 rbean還需要額外的 uid，從 20 萬開始，可以用 65535 個
# 宣告 gbean 還需要額外的 uid，從 30 萬開始，可以用 65535 個
$ echo 'bigred:100000:65535
rbean:200000:65535
gbean:300000:65535' | sudo tee /etc/subuid

# 宣告 bigred 還需要額外的 gid，從 10 萬開始，可以用 65535 個
# 宣告 rbean還需要額外的 gid，從 20 萬開始，可以用 65535 個
# 宣告 gbean 還需要額外的 gid，從 30 萬開始，可以用 65535 個
$ echo 'bigred:100000:65535
rbean:200000:65535
gbean:300000:65535' | sudo tee /etc/subgid


# 一定要重新開機, /etc/subuid 及 /etc/subgid 的設定才會生效
$ sudo reboot
```

- 當透過 podman 建立 Rootless 的 Container 時，須要幫每位使用者配發額外可用的 UID 及 GID


```bash=
# 使用 rbean 帳號登入
$ ssh rbean@<ALP.Podman IP> 
rbean@172.29.0.55s password: rbean

# 一般使用者 rbean run Cotainer
$ podman run --name p1 -d quay.io/cloudwalker/alpine sleep infinite

# 檢視是否符合預期
$ podman ps -a
CONTAINER ID  IMAGE                              COMMAND         CREATED             STATUS                 PORTS       NAMES
b5f203c8d44d  quay.io/cloudwalker/alpine:latest  sleep infinite  About a minute ago  Up About a minute ago              p1

# bigred 使用者看不到
bigred@alp:~$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

# 檢視 host node 主機 ，Container 是由誰 run 的
$ ps aux | grep 'sleep infinite'
rbean      3673  0.0  0.0   1584     4 ?        Ss   13:22   0:00 sleep infinite

# 進入 Contianer 內部查看使用者
$ podman exec p1 id 
uid=0(root) gid=0(root)

# 刪除 p1 Container
$ podman rm -f p1
```

- 當一般使用者透過 podman 建立 Container 時，因為用 user namespace 隔離出不同的空間，所以 Container 內部的 User 為 root 。


### 檢視 Rootless Container 建立時所需的完整內容

```=
$ tree -L 4 .local/
.local/
└── share
    └── containers
        ├── cache
        │   └── blob-info-cache-v1.boltdb
        └── storage
            ├── defaultNetworkBackend
            ├── libpod
            ├── mounts
            ├── networks
            ├── overlay
            ├── overlay-containers
            ├── overlay-images
            ├── overlay-layers
            ├── secrets
            ├── storage.lock
            ├── tmp
            └── userns.lock

13 directories, 4 files
```

- 因每位使用者在建立 Rootless Container 時，各自有各自的目錄，所以才能有各自把 Rootless Cotainer 建立出來


如果內外都指定要一般使用者呢?

```bash=
# --userns keep-id 啟用 User Namespace 
$ podman run --rm --name u1 --userns keep-id -d alpine sleep 120

# 查看 Container 外部使用者
$ ps aux | grep -v grep | grep "sleep 120"
rbean      4579  0.0  0.0   1584     4 ?        Ss   13:34   0:00 sleep 120

# 進入 Container 內部，檢視使用者
$ podman exec -it u1 sh
~ $ whoami
rbean
~ $ id
uid=1002(rbean) gid=1002(rbean)

# 停止 u1 Container 
$ podman stop u1
```


```bash=
# 建立指定 uid 999 的 u1 Container
$ podman run --rm --name u1 -u 999 -d alpine sleep 360
ebda4ea769f17061966c692f7dc9c760785d7336a67b6ee4fd593c81faadd610

# 檢視外部使用者誰執行 Container 的第一支程式
# 因從 0 開始算 ，所以 uid 200998
$ ps aux | grep -v grep | grep 'sleep 360'
200998     5082  0.0  0.0   1584     4 ?        Ss   13:40   0:00 sleep 360

$ podman exec -it u1 sh
~ $ whoami
999
~ $ id
uid=999(999) gid=0(root)
```


## Podman Rootless image


```bash=
# 編輯 Dockerfile (名字自取 : myuser )
# USER appuser | 指定執行 sleep infinity 的使用者為 appuser
# ENTRYPOINT ["sleep", "infinity"] | Container 第一支 run 的程式
$ echo '
FROM quay.io/cloudwalker/ubuntu
RUN groupadd appgroup
RUN useradd -r -u 1005 -g appgroup appuser
USER appuser
ENTRYPOINT ["sleep", "infinity"]' > myuser

# "-" ，標準輸入
# "<" ，重導 redirection
# 將 myuser 這個檔案，透過重導，標準輸入給 podman build 建立 image
# 在 Rootless 的操作環境，也可以透過正規 Dockerfile 來做 image
$ podman rmi myuser; podman build -t myuser  -  < myuser

# 檢查是否符合預期
$ podman images
REPOSITORY                  TAG         IMAGE ID      CREATED         SIZE
localhost/myuser            latest      6994bb0ff970  16 seconds ago  75.5 MB

# 建立 Rootless Container u1
$ podman run --rm --name u1 -d myuser

# 檢視外部使用者誰執行 Container 的第一支程式
$ ps aux | grep -v grep | grep sleep
201004     6334  0.0  0.0   2516   592 ?        Ss   13:56   0:00 sleep infinity

# 進入 u1 Container 檢視內部使用者
$ podman exec u1 id
uid=1005(appuser) gid=1000(appgroup) groups=1000(appgroup)
```


## gbean 使用者可否使用 rbean 自製的 image ?


```bash=
# podman save 備份 image 變成 *.tar 打包檔
$ podman save base.img:v1.0 > base.img.tar

# 離開
$ exit

# 登入 gbean 使用者帳號
$ ssh gbean@172.16.119.2
gbean@172.16.119.2s password: gbean

# 檢視當前沒有任何 image
$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE

# 拷貝在 rbean 使用者的 image 打包檔，到當前目錄
$ cp /home/rbean/base.img.tar  .

# 還原 base.img.tar 打包檔，透過 redirection 丟給 podman load 命令還原
$ podman load < base.img.tar

# 檢查是否符合預期
$ podman images
REPOSITORY          TAG         IMAGE ID      CREATED      SIZE
localhost/base.img  v1.0        d4e63a3c44c7  3 hours ago  820 MB

# 建立 Container 
$ podman run --rm -it base.img 
 * Starting OpenBSD Secure Shell server sshd                  [ OK ] 
bigred@5f13ae2cdf56:~$ exit
```

## Podman Rootless Container Network


**slirp4netns** | slirp for network namespace 
- slirp | 撥號網路
- allows connecting a network namespace to the Internet in a completely unprivileged way, by connecting a TAP device in a network namespace to the usermode TCP/IP stack ("slirp").

![](https://i.imgur.com/yiEjpzR.png)


### 實做 slirp4netns


```bash=
# 安裝 Podman 會一併安裝 slirp4netns 這套件  
$ slirp4netns -v
slirp4netns version 1.2.0
commit: 656041d45cfca7a4176f6b7eed9e4fe6c11e8383
libslirp: 4.7.0
SLIRP_CONFIG_VERSION_MAX: 4
libseccomp: 2.5.2

# 在 bigred 使用者
$ sudo unshare --pid --fork --mount-proc --net --uts sh
root@alp:/home/bigred$ ifconfig -a
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


```bash!
# 再開啟一個的 CMD 視窗, 執行以下命令
$ ssh bigred@<alp.docker IP>

# 檢視 unshare 執行 sh 和 sh 的 pid
$ ps aux | grep ' sh$'
root       8367  0.0  0.0    844     4 pts/0    S    14:29   0:00 unshare --pid --fork --mount-proc --net --uts sh
root       8368  0.0  0.0   1712   956 pts/0    S+   14:29   0:00 sh

# 產生虛擬網路卡 tun
$ sudo modprobe tun

# 透過 slirp4netns 命令，讓 unshare 做出來的 Container 可以上網
# -mtu 設定總共有幾個封包能上網的值，
$ sudo slirp4netns --configure  --disable-host-loopback   --mtu=65520 8368 tap0
sent tapfd=5 for tap0
received tapfd=5
Starting slirp
* MTU:             65520
* Network:         10.0.2.0
* Netmask:         255.255.255.0
* Gateway:         10.0.2.2
* DNS:             10.0.2.3
* DHCP begin:      10.0.2.15
* DHCP end:        10.0.2.30
* Recommended IP:  10.0.2.100

# 回到 unshare 終端機 檢查
root@alp:/home/bigred$ ifconfig -a
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

tap0      Link encap:Ethernet  HWaddr CE:09:B4:E4:2B:2B
          inet addr:10.0.2.100  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::cc09:b4ff:fee4:2b2b/64 Scope:Link
          UP BROADCAST RUNNING  MTU:65520  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:726 (726.0 B)

[註] 按 ctrl + c 可停止上面命令執行, unshare 終端機中的 tap0 網卡會自動移除 

```


## Rootless Container 內定網路架構


```bash=
# bigred 使用者 run Rootless Container 
$ podman run --rm -it quay.io/cloudwalker/alpine

/ # ifconfig tap0
tap0      Link encap:Ethernet  HWaddr 2A:8A:55:71:52:50
          inet addr:10.0.2.100  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::288a:55ff:fe71:5250/64 Scope:Link
          inet6 addr: fd00::288a:55ff:fe71:5250/64 Scope:Global
          UP BROADCAST RUNNING  MTU:65520  Metric:1
          RX packets:1 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:110 (110.0 B)  TX bytes:430 (430.0 B)

/ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    0      0        0 tap0
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 tap0

/ # cat /etc/resolv.conf
nameserver 10.0.2.3
nameserver 172.16.119.1

/ # ping -c 1 www.hinet.net
PING www.hinet.net (61.221.82.5): 56 data bytes
64 bytes from 61.221.82.5: seq=0 ttl=42 time=24.640 ms

/ # exit
```


```bash!
# 建立 2 台 Rootless Container 
$ podman run --rm -itd --name a1 quay.io/cloudwalker/alpine
$ podman run --rm -itd --name a2 quay.io/cloudwalker/alpine

# 查看 a1 a2 的 ip 位址
$ podman exec a1 hostname -i; podman exec a2 hostname -i
10.0.2.100
10.0.2.100

* a1 與 a2 的 IP 位址一樣, 他們網路無法互通

$ ps aux | grep -v grep | grep -e "slirp"
rbean      6326  0.0  0.0   2816  1560 ?        S    13:56   0:00 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp --enable-ipv6 -c -e 3 -r 4 --netns-type=path /tmp/podman-run-1002/netns/netns-71ea094f-5c6a-859a-f7e6-04b0661a6c83 tap0
bigred     9947  0.0  0.0   2816  1552 pts/0    S    14:53   0:00 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp --enable-ipv6 -c -e 3 -r 4 --netns-type=path /tmp/podman-run-1000/netns/netns-c306bb49-44f0-2f82-21d1-2613696aa617 tap0
bigred     9977  0.0  0.0   2816  1628 pts/0    S    14:53   0:00 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp --enable-ipv6 -c -e 3 -r 4 --netns-type=path /tmp/podman-run-1000/netns/netns-e8f75675-352e-e4b4-c3b0-c5c5d10d999e tap0
$ podman rm -f a1 a2

```

## Rootless Container 內定網路對外連通


```bash=
$ mkdir html ; echo "<h1>Rootless Container</h1>" > html/index.html

# ` --publish 8080:80` | 在 podman host 主機開 8080 port
# 
$ podman run --rm -d --name n1 --publish 8080:80 --volume ${PWD}/html:/usr/share/nginx/html quay.io/cloudwalker/nginx
✔ docker.io/library/nginx:latest
Trying to pull docker.io/library/nginx:latest...
.........
5d6265214ef71ffce826d38b16dffd2557a0cfcd353dbecef92b87d4a6d47071

$ curl http://localhost:8080
<h1>Rootless Container</h1>

# nginx 伺服器，會啟動子程序，來提供連線的服務
# 在 Rootless Container 中，會有很多情況會需要用到額外 uid 
$ ps aux | grep -v grep | grep nginx
bigred    11649  0.0  0.2  10656  5920 ?        Ss   15:18   0:00 nginx: master process nginx -g daemon off;
100100    11675  0.0  0.1  11092  2644 ?        S    15:18   0:00 nginx: worker process
100100    11676  0.0  0.1  11092  2644 ?        S    15:18   0:00 nginx: worker process
```

![](https://i.imgur.com/bbMximU.png)


- 在 podman host 主機中，我們無法直接透過打 10.0.2.100，直接連到 Rootless Container 中

![](https://i.imgur.com/BPg0Dtv.png)


```bash=
# 建立 Rootless Container
$ podman run --name b1 -itd quay.io/cloudwalker/alpine.sshd


$ podman exec b1 netstat -an | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN 

$ ssh bigred@10.0.2.100
^C

$ podman rm -f b1

$ podman run --name b1 -h ssn763 -p 22100:22 -itd quay.io/cloudwalker/alpine.sshd

$ ssh bigred@localhost -p 22100
Warning: Permanently added '[localhost]:22100' (ED25519) to the list of known hosts.
bigred@localhost's password: bigred
bigred@ssn763:~$ exit
```



## Rootless Container 自建網路架構


```bash!
$ podman network create --subnet=192.168.188.0/24 --gateway=192.168.188.254  mynet2

$ podman network ls
NETWORK ID    NAME        DRIVER
c26de056b034  mynet2      bridge
2f259bab93aa   podman      bridge

$ podman run --name n1 --net mynet2 -itd quay.io/cloudwalker/alpine; podman run --name n2 --net mynet2 -itd quay.io/cloudwalker/alpine

$ podman exec n1 hostname -i; podman exec n2 hostname -i
192.168.188.2
192.168.188.3

$ podman exec n2 ping -c2 192.168.188.2
PING 192.168.188.4 (192.168.188.2): 56 data bytes
64 bytes from 192.168.188.2: seq=0 ttl=42 time=0.029 ms
64 bytes from 192.168.188.2: seq=1 ttl=42 time=0.096 ms

$ podman exec n2 ping -c2 n1
PING n1 (192.168.188.1): 56 data bytes
64 bytes from 192.168.188.1: seq=0 ttl=42 time=0.033 ms
64 bytes from 192.168.188.1: seq=1 ttl=42 time=0.153 ms

```




---


# Podman Pods 


![](https://i.imgur.com/a4kkGBP.png)

- podman 產生出來的 pod ，裡面**一定會有 Infra Container** 
- 一個 pod 對應一個企業的 soulation ，因 pod 裡可存在多台 Container，每台 Container 對應不同服務，像是 web server, Database Server... 等 ，最後包成一個 pod，就能成為企業的 Soulation。


## 建立 pod (Rootless)


```bash=
$ podman pod create -n mypod

$ podman pod list
POD ID        NAME        STATUS      CREATED         INFRA ID      # OF CONTAINERS
b02b99948314  mypod       Created     14 seconds ago  3635ac314b01  1

$ podman ps -a --pod
CONTAINER ID  IMAGE                                    COMMAND     CREATED         STATUS      PORTS       NAMES               POD ID        PODNAME
3635ac314b01  localhost/podman-pause:4.1.0-1658428998              25 seconds ago  Created                 b02b99948314-infra  b02b99948314  mypod
```

- The default infra container is based on the `k8s.gcr.io/pause:4.1.0-1658428998` image
- Container run 的 process 是 sleep infinite，沒動用網路和硬碟，只動用記憶體和 CPU
- Every podman pod includes an infra container by default. Its purpose is to **hold the namespaces associated with the pod** and allow podman to connect other containers to the pod. This also lets pods live, if the pod is not running any application containers.
- Kubernetes 開始 run 的是 VM。


## Add a container to a pod 


```bash=
$ podman run -d --pod mypod --name ap1 quay.io/cloudwalker/alpine top

$ podman ps -a --pod | grep mypod
3635ac314b01  localhost/podman-pause:4.1.0-1658428998              13 minutes ago  Up 28 seconds ago              b02b99948314-infra  b02b99948314  mypod
02a15e3c2fd4  quay.io/cloudwalker/alpine:latest        top         27 seconds ago  Up 28 seconds ago              ap1                 b02b99948314  mypod

$ podman run -d --pod mypod --name np1 quay.io/cloudwalker/nginx

$ podman exec ap1 hostname
mypod
$ podman exec np1 hostname
mypod
$ podman exec ap1 hostname -i
10.0.2.100
$ podman exec np1 hostname -i
10.0.2.100
```

- Container 的電腦名稱共用 pod 的名字及 ip 位址


## Generate a K8S YAML File 


```bash=
# 產生 mypod 的 yaml 檔
$ podman generate kube mypod -f mypod.yaml

# -f 強制，強制刪除運行中的 pod
$ podman pod rm -f mypod

# 檢查是否符合預期，都是空的
~$ podman pod list
POD ID      NAME        STATUS      CREATED     INFRA ID    # OF CONTAINERS

$ podman ps -a --pod
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES       POD ID      PODNAME
```

podman generate 用法

```bash=
Usage:
  podman generate [command]

Available Commands:
  kube        Generate Kubernetes YAML from containers, pods or volumes.
  systemd     Generate systemd units.
```


:::spoiler 檢視 podman generate 出來的 yaml 檔

```bash=
$ cat mypod.yml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.1.0
apiVersion: v1
kind: Pod
metadata:
  annotations:
    io.kubernetes.cri-o.ContainerType/ap1: container
    io.kubernetes.cri-o.ContainerType/np1: container
    io.kubernetes.cri-o.SandboxID/ap1: mypod
    io.kubernetes.cri-o.SandboxID/np1: mypod
    io.kubernetes.cri-o.TTY/ap1: "false"
    io.kubernetes.cri-o.TTY/np1: "false"
    io.podman.annotations.autoremove/ap1: "FALSE"
    io.podman.annotations.autoremove/np1: "FALSE"
    io.podman.annotations.init/ap1: "FALSE"
    io.podman.annotations.init/np1: "FALSE"
    io.podman.annotations.privileged/ap1: "FALSE"
    io.podman.annotations.privileged/np1: "FALSE"
    io.podman.annotations.publish-all/ap1: "FALSE"
    io.podman.annotations.publish-all/np1: "FALSE"
  creationTimestamp: "2022-07-23T01:51:20Z"
  labels:
    app: mypod
  name: mypod
spec:
  containers:
  - command:
    - top
    image: quay.io/cloudwalker/alpine:latest
    name: ap1
    resources: {}
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
  - args:
    - nginx
    - -g
    - daemon off;
    image: quay.io/cloudwalker/nginx:latest
    name: np1
    resources: {}
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
  restartPolicy: Never
status: {}
```

:::


重建 pod 

```bash=
$ podman play kube mypod.yml
```

`podman play` 用法 : 

```bash=
Usage:
  podman play [command]

Available Commands:
  kube        Play a pod or volume based on Kubernetes YAML.
```


```bash=
$ podman pod list
POD ID        NAME        STATUS      CREATED         INFRA ID      # OF CONTAINERS
8d4b1d6a29cc  mypod       Running     12 minutes ago  0118b4953bee  3
bigred@alp:~$ podman ps -a --pod
CONTAINER ID  IMAGE                                    COMMAND               CREATED         STATUS             PORTS       NAMES               POD ID        PODNAME
0118b4953bee  localhost/podman-pause:4.1.0-1658428998                        13 minutes ago  Up 13 minutes ago              8d4b1d6a29cc-infra  8d4b1d6a29cc  mypod
98e503e8ee12  quay.io/cloudwalker/alpine:latest                              13 minutes ago  Up 13 minutes ago              mypod-ap1           8d4b1d6a29cc  mypod
d1f37ffb76fe  quay.io/cloudwalker/nginx:latest         nginx -g daemon o...  13 minutes ago  Up 13 minutes ago              mypod-np1           8d4b1d6a29cc  mypod
```


## Wordpress Pod Application


單命令建立 Pod Application

```bash!
$ podman run -d --restart=always --pod new:wpapp_pod -e MYSQL_ROOT_PASSWORD="myrootpass" -e MYSQL_DATABASE="wp-db" -e MYSQL_USER="wp-user" -e MYSQL_PASSWORD="w0rdpr3ss"  -p 8080:80 --name=wptest-db quay.io/cloudwalker/mariadb
```

- `-d` ，Run container in background and print container ID
- `new` ，創建一個新的 pod，而不是嘗試將 Container 分配給現有的 pod。
- `-e` ，environment，mariadb 需要這些變數才能建立
- `-p 8080:80` ，先開好 80 port ，給等下的 Wordpress 這台 Container 用

檢視 wpapp_pod 結構

```bash=
$ podman ps -a --pod | grep wpapp_pod
```


wpapp_pod 加入 Wordpress Container 

```bash!
$ podman run -d --restart=always --pod=wpapp_pod -e WORDPRESS_DB_NAME="wp-db" -e WORDPRESS_DB_USER="wp-user"  -e WORDPRESS_DB_PASSWORD="w0rdpr3ss" -e WORDPRESS_DB_HOST="127.0.0.1" --name wptest-web quay.io/cloudwalker/wordpress
```

檢視 wpapp_pod 結構 

```bash=
$ podman ps -a --pod | grep wpapp_pod
```

連接 Wordpress 網站

```bash=
$ curl http://localhost:8080/wp-admin/install.php
.........
</head>
<body class="wp-core-ui language-chooser">
<p id="logo">WordPress</p>
.......
```

停止 刪除 pod 

```
# 停止 pod
$ podman pod stop wpapp_pod

# 刪除 pod
$ podman pod rm wpapp_pod
```







###### tags: `系統工程`