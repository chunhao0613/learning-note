# Docker-AppContainer-Part1


:::warning

:::spoiler 目錄 (  :mag_right: 點我，可展開 )

[TOC]

:::


- [前篇 Container-Runtime-CRI 詳細介紹連結](https://hackmd.io/@QI-AN/container-runtime-cri)

---

## Docker 命令運作圖

![](https://i.imgur.com/611SqR7.png)


- Application Container｜軟體貨櫃
    - 別稱：Container host、Container machine
        > 在外面盡量用英文表達，才不會造成誤解。
- 當我們執行：`dcker pull`，他會去 docker 公司的 image 中心/ quay.io（另一個 image 的中心），抓一片光碟片下來
- 當我們拿到 image，代表我們擁有整個系統檔案目錄，再打 `docker run` 命令，他就會幫我們產生 Container
- `docker stop` : 關閉 container
- `docker start` : 開啟 Container
- `docker restart` : 將 Container 重新開機
- `docker save` : 將 image 變成一個打包檔 (`*.tar`)
- `docker load` : 將 image 的打包檔還原成 image
- `docker commit` 把 Container 變成 image，`push` (上載)到 Docker 公司的 image 中心/ quay.io（另一個 image 的中心）
- **當你寫了一大堆的應用系統（會計系統、倉儲系統...等），把它一個一個變成 image，`push` 到 image 中心，其他人就可以 `docker pull` 下來，再 `docker run` ，就能直接使用這些各式各樣的應用系統，還能降低 Human Error**
- image 裡面有多個設定檔和多個壓縮檔組成 bundle ，


:::spoiler 條列式重點整理 by 程威
:::danger
- Docker 運作流程
    - image 階段
        - docker pull 先從 Docker Registry、Quay.io 獲取 image
        - docker run 創建 >> containers 階段（務必要有 rootfs 如同電腦需要有硬碟的前提下才能跑）
    - containers 階段
        - docker stop 關機
        - docker start 開機
        - docker restart 重新啟動
        - docker commit 將更動內容再變成 image 封裝 >> image 階段 docker push
        - docker push 將 image 上載至 Docker Registry、 Quay.io 供其他工程師 pull 到設備上部署
- 落實 Docker 的操作可以實現順暢的交付、部署流程，有效降低 human error
:::


---


## 搜尋與下載 Docker Image

在 Windows 系統的 CMD 視窗, 執行以下命令

```
$ ssh bigred@<ALP.Docker IP>
```

上網搜尋 busybox 的 image 資訊

```
$ docker search quay.io/busybox
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
prometheus/busybox                                # Prometheus Busybox Docker Base Images  [![…   0
libpod/busybox                                                                                    0
quay/busybox                                      Busybox base image.                             0
...
```

下載 `quay.io` 的 busybox image
```
$ docker pull quay.io/quay/busybox
Using default tag: latest
latest: Pulling from quay/busybox
9c075fe2c773: Pull complete
ee780d08a5b4: Pull complete
Digest: sha256:ffd944135bc9fe6573e82d4578c28beb6e3fec1aea988c38d382587c7454f819
Status: Downloaded newer image for quay.io/quay/busybox:latest
quay.io/quay/busybox:latest
```

檢查我們的 alpine 主機有幾個 image
```
$ docker images
REPOSITORY                   TAG       IMAGE ID       CREATED         SIZE
quay.io/cloudwalker/alpine   latest    6dbb9cc54074   13 months ago   5.61MB
quay.io/quay/busybox         latest    e3121c769e39   20 months ago   1.22MB
```

![](https://i.imgur.com/pN8nzwl.png)

- `docker.io` 這個網站伺服器，會限制一個 IP 在 6 小時之內，只能下載 100 次，如果我們在一間公司，上網的時候有通過 NAT 轉址上網的話，很容易就會達到這個限制
- `docker run` 、 `docker pull` 都會去 `docker.io` 拉 image ，且只要有連接，不管中間有沒有中斷，都會被算次數。



---

## 建立與執行軟體貨櫃 (Container) 主機

建立並且執行Container
```
$ docker run --name b1 -it quay.io/cloudwalker/busybox  /bin/sh
```
> `--name`，給Container 一個名字，注意，不是Hostname！
> -it ，產生虛擬終端機 。貝殼程式要執行，一定要有虛擬終端機
> quay.io/cloudwalker 是陳松林老師在Quay.io裡的小店鋪


:::spoiler ` docker run` 幫我們連接了網路
```
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=127 time=3.715 ms
64 bytes from 8.8.8.8: seq=1 ttl=127 time=4.312 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 3.715/4.013/4.312 ms
/ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
```

:::


Container 用的 kernel 是 5.15.41-0-virt
```
/ # uname -r
5.15.41-0-virt
```
如何證明 Container 用的就是 Host OS 的 kernel
再開一個終端機

```
Welcome to DT Platform Version 21.08 (Alpine Linux : 3.16.0)
IP : 192.168.61.144

bigred@alp:~$ uname -r
5.15.41-0-virt
```

> 發現一模一樣，得證，Container 就是用 Host OS 的 kernel

檢查 uts Namespace 和 PID namespace
```
/ # hostname
651adac9666e

/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
   13 root      0:00 ps aux
```

刪除 Container 的 docker 命令：docker rm b1

```
$ docker rm b1
b1
bigred@alp:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

建立並且執行 Container 後刪除
```
bigred@alp:~$ docker run --name b1 -it quay.io/cloudwalker/busybox  /bin/sh
/ # exit

bigred@alp:~$ docker ps -a
CONTAINER ID   IMAGE                         COMMAND     CREATED          STATUS                     PORTS     NAMES
9fe67f884ad6   quay.io/cloudwalker/busybox   "/bin/sh"   19 seconds ago   Exited (0) 2 seconds ago             b1
```

- Container 被刪掉，Overlay2 檔案系統的 upper 的目錄也會被刪除
- 如何快速復原 ?
    - 



啟動Container
```
bigred@alp:~$ docker start b1
b1

bigred@alp:~$ docker ps -a
CONTAINER ID   IMAGE                         COMMAND     CREATED              STATUS         PORTS     NAMES
9fe67f884ad6   quay.io/cloudwalker/busybox   "/bin/sh"   About a minute ago   Up 7 seconds             b1
```

進入已啟動的Container，並且執行`ps aux`
```
bigred@alp:~$ docker exec -it b1 sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 sh
   13 root      0:00 ps aux
```
:::spoiler 提問：為何會有兩支sh程式？

`/bin/sh` 是`docker run -it`所連線的貝殼程式
`sh` 是我們`docker exec -it` 連線的另一個貝殼程式

![](https://i.imgur.com/ZRiKhSP.png)

:::



exit 離開Container後，為何Container還在跑？
```
/ # exit
bigred@alp:~$ docker ps -a
CONTAINER ID   IMAGE                         COMMAND     CREATED          STATUS          PORTS     NAMES
9fe67f884ad6   quay.io/cloudwalker/busybox   "/bin/sh"   28 minutes ago   Up 26 minutes             b1
```

因為我們是離開第二個連線的虛擬終端機上的貝殼程式，本體的貝殼程式並未結束。

如何進入本體/bin/sh貝殼程式？

答： `docker attach b1`

![](https://i.imgur.com/KjvMExK.png)

進入本體/bin/sh貝殼程式所在的虛擬終端機，exit 離開
```
bigred@alp:~$ docker attach b1
/ # exit
```

Container關機
```
bigred@alp:~$ docker ps -a
CONTAINER ID   IMAGE                         COMMAND     CREATED          STATUS                     PORTS     NAMES
9fe67f884ad6   quay.io/cloudwalker/busybox   "/bin/sh"   38 minutes ago   Exited (0) 2 seconds ago             b1
```

執行`brctl show` ( bridge control show)
```
bigred@alp:~$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242626524dc       no
```
> 發現interfaces 沒有人

啟動Container，並檢查interface
```
bigred@alp:~$ docker start b1
b1

bigred@alp:~$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242626524dc       no              veth19b375d
```
> 發現我們的b1正在使用它
> dockerd會幫我們產生docker0 虛擬橋接器
> 在Container裡面因為它沒有docker，所以沒人幫它產生docker0 這個虛擬橋接器。


---

## 重建 n1 Container 並設定自動啟動

```
$ docker run -d --restart=always --name n1 quay.io/cloudwalker/nginx
```
> `--restart=always` Container自動啟動


查詢IP指令 ：`hostname -i`
```
# hostname -i
172.17.0.2

#ip a
sh: 10: ip: not found
# ifconfig
sh: 11: ifconfig: not found
```

**Container裡，只要有Application的相依檔就好，不需要整個系統檔案目錄，資訊安全等級才會最高。**


---

## 討論 

**問題討論 (一) : 為何以下命令執行錯誤 ?**

```
$ docker run --rm --name=b1 quay.io/cloudwalker/busybox
$ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED      STATUS                  PORTS     NAMES
```

> `quay.io/cloudwalker/busybox`，這個image可以讓我們執行內定的命令，裡面有宣告跑sh貝殼程式

應將命令更正為：

```
$ docker run --rm -it --name=b1 quay.io/cloudwalker/busybox
/ #
```

> -i ，虛擬終端機
> -t ，交談模式

怎麼證明`--rm`的參數

```
$ exit

$ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED      STATUS                  PORTS     NAMES
```

> 空的，證明成功！


==任何一個image可以有內定執行的命令（也可以沒有）==

**問題討論 (二) : 為何以下命令執行錯誤 ?**

```
$ docker run --rm --name=b2 -i -t quay.io/cloudwalker/busybox  /bin/bash
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/bin/bash": stat /bin/bash: no such file or directory: unknown.
```

==因為`quay.io/cloudwalker/busybox `這個 image 裡面沒有bash貝殼程式==
 

**問題討論 (三) : 為何 Container 系統中不能修改日期與時間 ?**

```
$ docker run --rm --name=b2 -i -t quay.io/cloudwalker/busybox  /bin/sh
/ # whoami
root
/ # date --set "2006-10-10 18:00:00"
date: can't set date: Operation not permitted
Tue Oct 10 18:00:00 UTC 2006
/ # exit
```

==root 身份在 Container 裡不是萬能的，可以被 linux capability 管控和限制 root 的行為。==


---

## 建立 Alpine 軟體貨櫃主機

```
$ docker run --name a1 -it quay.io/cloudwalker/alpine sh
/ # busybox | grep httpd
```

> Docker Alpine Image 內建的 busybox 並沒提供 httpd 功能



下載 Busybox 執行檔

```
/ #  wget https://busybox.net/downloads/binaries/1.31.0-defconfig-multiarch-musl/busybox-x86_64
```


安裝 Busybox 執行檔

```
/ #  chmod +x busybox-x86_64; mv busybox-x86_64 bin/busybox
```


執行 Busybox 命令

```
/ # busybox | grep httpd
	hexedit, hostid, hostname, httpd, hush, hwclock, i2cdetect, i2cdump, i2cget, i2cset, id, ifconfig,
```



## 建立 Busybox Httpd 網站伺服器

製作網站目錄及首頁

```
/ # mkdir www; echo '<h1>Busybox HTTPd</h1>' > www/index.html
```

啟動 Busybox Httpd 網站伺服器

```
/ # busybox httpd -p 8888 -h www
```

安裝網頁工具

```
/ # apk update; apk add elinks curl
```

取得網頁

```
/ # curl http://localhost:8888
<h1>Busybox HTTPd</h1>
```



檢視網頁

```
/ # elinks -dump 1 http://localhost:8888
                                   Busybox HTTPd

/ # exit
```

> elinks ，文字型的瀏覽器，不會看到標籤，會直接顯示結果。（Hacker 專用）

---

## 申請 Docker HUB 帳號

[Docker HUB申請網站](https://hub.docker.com)

我的帳號：hahappyman


## 製作 a1 image 並上載至 Docker HUB

```
$ docker commit a1 hahappyman/a1
sha256:f81b48d760ff4e80633321de0bfaf6522f1d4ed55ce10dcd05dac4ecec51bc6e

$ docker login

$ docker push hahappyman/a1

$ docker logout
```

**[重要] 關閉 Docker Host 之前, 記得執行 docker logout, 然後刪除 /home/bigred/.docker 目錄**



---

## 使用自製 a1 image


```
重新建立 testweb 貨櫃主機
$ docker run --name testa1 -d <login name>/a1 httpd -f -p 8888 -h www

$ docker exec testa1 hostname -i
172.17.0.2

$ curl http://172.17.0.2:8888
<h1>Busybox HTTPd</h1>

以強制方式移除 testa1
$ docker rm -f testa1
```


---

因為ducker的網站對同一個IP的下載有做限制，所以發生了逃難潮
到[quay.io](https://quay.io/)申請帳號，申請完後，到[recovery開頭的quay.io的網站](https://recovery.quay.io/)操作image


透過 `docker history` 來看 image 的內定執行命令

```bash!
$ docker history quay.io/cloudwalker/nginx
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
d1a364dc548d   15 months ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      15 months ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      15 months ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      15 months ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      15 months ago   /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB
<missing>      15 months ago   /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB
<missing>      15 months ago   /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB
<missing>      15 months ago   /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB
<missing>      15 months ago   /bin/sh -c set -x     && addgroup --system -…   63.9MB
<missing>      15 months ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B
<missing>      15 months ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.5.3        0B
<missing>      15 months ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.0     0B
<missing>      15 months ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>      15 months ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      15 months ago   /bin/sh -c #(nop) ADD file:7362e0e50f30ff454…   69.3MB
```

- `history`，Show the history of an image
- `--no-trunc`，可以看到被切掉的資訊











###### tags: `系統工程`







































































































































































