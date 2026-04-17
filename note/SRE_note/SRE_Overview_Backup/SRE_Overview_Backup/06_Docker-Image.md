# Docker-Image

:::warning

:::spoiler 目錄 (  :mag_right: 點我，可展開 )

[TOC]

:::

- [前篇 Docker-AppContainer-Part1 介紹連結](https://hackmd.io/@QI-AN/docker-app-1)

## 認識 Dockerfile

![](https://i.imgur.com/0vPkqrR.png)

- <font color=red>**Dockerfile 就是建置 Docker Image 的腳本**</font>
- 正式產生 image 的方法：<font color=red>**D**</font>ockerfile （D一定要大寫），好處是會有文件可以交接。
- 手工做的 image，過了半年會忘記當初的 image 怎麼做出來的。
- 貨櫃化要成功，一定是自己做 image ，而不是抓別人的來用


## 準備 Distroless Image 的系統目錄

```bash!
$ mkdir -p ~/r2d2/rootfs/{bin,sbin,usr/{bin,sbin},dev,lib,proc}; cd ~/r2d2

$ cp /bin/sh rootfs/bin; cp /lib/ld-musl-x86_64.so.1 rootfs/lib/; cp /bin/busybox rootfs/bin/; sudo chown -R root:root rootfs

# 將 /bin/busybox --install -s 這個命令的根目錄切換到 ~/r2d2/rootfs ，並在新的根目錄產生 busybox 提供所有命令的軟連結(soft link)。
# -s 它創建軟連結而不是硬鏈結。
$ sudo chroot rootfs /bin/busybox --install -s

$ sudo chroot rootfs /bin/sh
@alp:/$ find / -name busybox | wc -l
1
@alp:/$ exit

# 先切換工作目錄到 rootfs ，然後將 r2d2 的所有內容壓縮到當前目錄，再回到上一層目錄
$ cd rootfs; tar zcvf ../r2d2.tar.gz .; cd ..
```

- `z`，用 gzip 的方式(解)壓縮
- `c`，Create
- `v`，Verbose
- `f`，Name of TARFILE ('-' for stdin/out)



## 自製與使用 Distroless image

- “Distroless” images，這種 image 只有 應用程式 和 它的相依檔(bins/libs)，不可以有 apk (package manager 以 Alpine 為例)可以新增套件，或是貝殼程式，以及其他的應用程式
- “Distroless” images contain only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution.

```dockerfile=
$ nano Dockerfile 
FROM scratch
ENV PATH=/bin:/usr/bin:/sbin:/usr/sbin
ADD r2d2.tar.gz  /
CMD ["/bin/sh"]
```

- `FROM`，指定 image
- `scratch`，空白 image
- `ENV`，設定 Container 啟動後的 PATH 環境變數
- `ADD r2d2.tar.gz  /`，image 做出來後，將 r2d2.at.gz 這個壓縮檔解壓縮到 image 的根目錄
    - `ADD` 和 `COPY` 都是複製檔案到 image 的目錄，差別在下面
    - `ADD`，local tar file auto-extraction into the image
    - `COPY`， only supports the basic copying of local files into the container
    - [ADD 和 COPY 的官網文件連結](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy)
    - [「初學Docker」Dockerfile 易混亂的指令](https://www.tpisoftware.com/tpu/articleDetails/1216)
- `CMD ["/bin/sh"]`，設定 Container 啟動後，內定執行的命令為 `/bin/sh`

```bash!
$ docker build --no-cache -t r2d2  .

$ docker images | grep r2d2
r2d2              latest    61d53f8cccae   13 minutes ago   2.28MB

$ docker run --rm --name r2d2 -it r2d2
/ # ps
PID   USER     TIME  COMMAND
    1 0         0:00 /bin/sh
    7 0         0:00 ps
/ # exit
```

只要是 image 裡面有的命令，Container 都能 run

```bash=
$ docker run -d r2d2 sleep 600
$ docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS      NAMES
5dd0d471c849   r2d2            "sleep 600"              7 seconds ago   Up 6 seconds              dreamy_brattain
```

## CGI with Golang

- 用 Golang 語言寫一個網站，裡面跑 bash script 
- cgi ，Common Gateway Interface，透過一個 web server 來 run 指定的電腦語言，如 bash script, python, ...等 

```bash=
$ mkdir -p ~/wulin/gocgi; cd ~/wulin/gocgi

$ echo 'package main

import (
	"net/http"
	"net/http/cgi"
)

func cgiHandler(w http.ResponseWriter, r *http.Request) {
	handler := cgi.Handler{Path: "cgichild.sh"}
	handler.ServeHTTP(w, r)
}

func main() {
	http.HandleFunc("/", cgiHandler)
	http.ListenAndServe(":8080", nil)
}' > main.go

$ go mod init gocgi

$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main

$ echo $'#!/bin/sh
echo "Content-type: text/html"
echo ""
echo \'<html><body>\'
echo \'Hello From Bash <br/>Environment:\'
echo \'<pre>\'
/usr/bin/env
echo \'</pre>\'
echo \'</body></html>\'
exit 0 ' > cgichild.sh

$ chmod +x cgichild.sh
```

- `echo "Content-type: text/html"`，會將執行結果傳回去給瀏覽器。
    - `/html`，告訴瀏覽器，等等會回傳 `html` 的標籤
- `echo ""`，顯示一個換行符號，這是格式。
- `<pre>`，保持原本的格式，有換行就換行

![](https://i.imgur.com/UR2FSDK.png)

- 當我們在自己的電腦打 `http://120.96.143.110:8080/` 這個網址(URL)，會連線到我們自己用 Golang 語言寫的程式架的網站，程式裡有寫如果網址後面有`/`，



## 測試 CGI with Golang 網站


```bash!
$ dir
total 6.4M
drwxr-sr-x 2 bigred bigred 4.0K Aug 23 20:19 .
drwxr-sr-x 3 bigred bigred 4.0K Aug 23 20:18 ..
-rwxr-xr-x 1 bigred bigred  182 Aug 23 20:19 cgichild.sh
-rw-r--r-- 1 bigred bigred   22 Aug 23 20:18 go.mod
-rwxr-xr-x 1 bigred bigred 6.3M Aug 23 20:19 main
-rw-r--r-- 1 bigred bigred  273 Aug 23 20:18 main.go

啟動 CGI with Golang 網站 
$ screen -S gocgi

$ ./main

按 Ctrl + a, d 複合鍵, 回到原先 終端機 畫面

$ curl 'http://localhost:8080/student?name=melia&age=16'
<html><body>
Hello From Bash <br/>Environment:
<pre>
SERVER_NAME=localhost:8080
REMOTE_HOST=127.0.0.1
......
REQUEST_URI=/student?name=melia&age=16
QUERY_STRING=name=melia&age=16
.......
</pre>
</body></html>

關閉 CGI with Golang 網站  
$ screen -r gocgi

按 Ctrl + C 

$ exit

```


## 自製 CGI with Golang Image

```bash!
$ echo 'FROM scratch
ADD main /
ADD cgichild.sh / 
CMD ["/main"] ' > Dockerfile

$ docker build -t gocgi  .

$ docker images
REPOSITORY    TAG         IMAGE ID            CREATED             SIZE
gocgi               latest       4b8dcd855475    9 seconds ago      6.6MB
.....
```

## 開發 Golang 網站


建目錄

```
$ mkdir -p ~/wulin; cd ~/wulin; mkdir mygo; cd mygo
```

寫一個go語言的程式
```
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

宣告要用的相依檔

```
$ go mod init mygo
```

翻譯成machine code
```
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main
```

> 最後產生的main這個檔案就可以在任何作業系統都能執行，因為程式所需的相依檔已經在檔案裡


## 自製原生 Docker Image

```
$ echo 'FROM scratch
ADD main /
CMD ["/main"] ' > Dockerfile
```

> Docketfile的命令，指定使用哪個image，可以從網路抓取別人已做好已存在的image，也可用scratch(空白)來自製
> `ADD main /`，把剛剛用go語言寫的經過編譯的程式檔，加到rootfs的目錄裡面
> `CMD ["/main"]` ，宣告image內定執行的命令：/main

<font color=red>**編寫Dockerfile主要的目的是要做出image，而做出image就是要做出rootfs（系統檔案目錄）+ config.json (標準設定檔)**</font>

建立image
```
$ docker build -t goweb  .
```
> -t ， 後面接image的名稱
> . ，代表目前的目錄來讀取Dockerfile


檢查
```
$ docker images
REPOSITORY    TAG         IMAGE ID            CREATED             SIZE
goweb             latest       7f9652539fc0      14 minutes ago    6.13MB
```


## 建立 g1 container

```
$ docker run --rm --name g1 -d -p 8888:8888 goweb
```
> 因為Container跑的程式不是sh，所以不需要給`-it`
> 我們使用的go語言程式在翻譯時，已經打包到一個檔案，所以App+Bins+Libs會變成一個檔案，但還是會使用Host OS 的 kernel 和硬體周邊
> 為何抓不到 g1 這台 Container 的 IP 位址 ?
> 因為rootfs裡面並沒有那些ifconfig、ip a s 和hostanme -i這些程式，只有main


其實還是能看得到

```
$ docker inspect g1 | grep '"IPAddress"'
```

強制刪除運行中的Container
```
$ docker rm -f g1
g1
```

重新建立一次

```
$ docker run --rm --name g1 -d -p 8888:8888 goweb
```
> -p 8888:8888 ，前面4個8是alpine linux開的port，後面4個8是Container開的port

```
$ curl http://localhost:8888
Hello, 世界
```


```
$ docker stop g1
```

:::spoiler **docker 的網路架構示意圖**

![](https://i.imgur.com/sMy8PPA.png)
:::


---

## 檢視原廠 Image 內定執行命令

查看image內定的指令

命令：

```
$ docker history quay.io/cloudwalker/busybox
```

結果：

```
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
d3cd072556c2   12 months ago   /bin/sh -c #(nop)  CMD ["sh"]                   0B
<missing>      12 months ago   /bin/sh -c #(nop) ADD file:c423dc64e02718dd3…   1.24MB
```


查看nginx那張image

```
$ docker history --no-trunc quay.io/cloudwalker/nginx | tr -s " "
```

結果：

```
IMAGE CREATED CREATED BY SIZE COMMENT
...前面省略 CMD ["nginx" "-g" "daemon off;"] 0B
...後面省略
```

什麼是`["nginx" "-g" "daemon off;"]`？

將整個shell程式轉為前景能持續執行的程式

==**Container內定執行的程式，一定要在前景執行，Container才會啟動，因為在背景執行的話，系統會認為前景還有人要執行。**==


# 自製 Docker Image 實務範例
## 撰寫 alpine.base Dockerfile

以後要做image，最好都要先產生對應的目錄區

```
$ mkdir -p ~/wulin/base; cd ~/wulin
```

編輯Dockerfile
```
$ echo 'FROM quay.io/cloudwalker/alpine
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
    chmod +x busybox-x86_64 && mv busybox-x86_64 bin/busybox1.28 && \
    mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

CMD ["/bin/bash"] ' > base/Dockerfile
```

> **注意如果要用別人的image，例如：alpine，一定要鎖定版本代號，才不會發生不相容的問題。**
事實上build Dockerfile時，有做Chroot的動作，所以RUN後面的指令產生的檔案都會在rootfs裡面。
> busybox1.28並沒有蓋掉alpine原本的內建的busybox
> `CMD ["/bin/bash"]` ，這行命令是存在config.json裡面



建立 alpine.base image

```
$ docker build --no-cache  -t alpine.base base/
```
> docker build命令會自動去找我們指定目錄裡面的Dockerfile


執行 alpine.base image 內定命令
```
$ docker run --rm -it alpine.base
bash-5.0# /bin/busybox1.28 | head -n 1
BusyBox v1.28.1 (2018-02-15 14:34:02 CET) multi-call binary.

bash-5.0# /bin/busybox | head -n 1
BusyBox v1.30.1 (2019-06-12 17:51:55 UTC) multi-call binary.

bash-5.0# busybox httpd -h /opt/www
httpd: applet not found

bash-5.0# busybox1.28 httpd -h /opt/www
bash-5.0# curl http://localhost
let me go

bash-5.0# exit
exit
```

## 測試 alpine.base image

為何沒辦法成功啟動Container ?
```
$ docker run --rm --name b1 alpine.base busybox1.28 httpd -h /opt/www
bigred@alp:~/wulin$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

==**Container 中的第一個啟動 process 必須前景執行, 這樣 Container 才能保持在執行狀態**==

```
$ docker run --rm --name b1 -d -p 80:80 alpine.base busybox1.28 httpd -f -h /opt/www
7e5ed9af1918a1f9909f209ebf90492a58e274deac8d3e9f0ab62123e20b7e6c
```
> -f ，將httpd的程序推到前景執行


檢查有沒有符合預期

```
$ curl http://localhost
let me go
```

將Container 停止

```
$ docker stop b1
b1
```

檢查

```
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


---


# 自製 OpenSSH Image
## 撰寫 alpine.sshd Dockerfile

```
$ cd ~/wulin; mkdir sshd

$ nano sshd/Dockerfile
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    # 產生OpenSSH Server的key，不是給前端的使用者用的，是給Server使用的
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    # 當使用者成功登入linux時，會顯示在/etc/motd裡面的內容
    echo -e 'Welcome to ALP bobo 6000\n' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
    # 禁止使用者重新開機和通殺程式的命令
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"] 
```

- `cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime`，設定時區
    - 在大型應用系統建置時，一定要注意時區，像如果收集 IOT 的資料，一次 100 萬筆，時區錯了，資料換算時區，會算到吐血
- `ENTRYPOINT` 所指定的執行命令, 在建立 Container 時強制執行此命令並且不可被其它命令取代
- `CMD ["-D"] `，下達參數，意思是讓 OpenSSH Server 前景執行
    - 當 `ENTRYPOINT` 出現，`CMD` 的功能變成為 `ENTRYPOINT` 的命令加參數


## 建立與使用  alpine.sshd image

```
$ docker build --no-cache  -t alpine.sshd sshd/
```

```
$ docker images
REPOSITORY                            TAG       IMAGE ID       CREATED          SIZE
alpine.sshd                           latest    e475bfcdf2fd   32 seconds ago   57.7MB
```


```
$ docker run --rm --name s1 -h s1 alpine.sshd sh
Extra argument sh.
```
[重要] 因使用 entrypoint 宣告內定命令, 便無法自行指定執行命令


建立 s1 container
```
$ docker run  --rm --name s1 -h s1 -d -p 22100:22 alpine.sshd
```
> `-p 22100:22` ，`22100` alpine 虛擬主機的 port，`22` Container 的 port


用 ssh 命令登入測試 Container
```
$ ssh bigred@localhost -p 22100
Warning: Permanently added '[localhost]:22100' (RSA) to the list of known hosts.
bigred@localhost's password:
Welcome to ALP SSHd 6000

s1:~$
```

- 如果要新增帳號的話，修改 Dockerfile 比較好，還是在 Contianer 建立比較好 ?
    - 修改 Dockerfile 缺點 : 要新增一個帳號，就要改一次 Dockerfile ，且透過 `docker history` ，可以看到建立 image 的歷史紀錄(其中就會包含建立帳號時的密碼)
    - 用 Container 建立的話，Container 刪除，在重建回來，帳號消失。


---

# Docker Container 備份與還原


```bash=
$ docker run --name s2 -h s2 -d alpine.sshd
6e52bd5aee18c0aed8dbb17920037be0b9109158329b401c56b4f64417c2d784

$ docker exec -it s2 bash

$ sudo useradd -m -s /bin/bash rbean
$ echo "rbean:rbean" | sudo chpasswd

$ exit
```


>使用上述命令建立 s2 Container, 這個 Container 有啟動 openssh server, 所以這個 Container 有 openssh 的執行狀態資訊檔, 這時使用 docker export 命令 
匯出的 Tar 檔中就會有殘留 openssh 的執行暫存檔, 以至後續做出的 image 無法啟動 openssh server, 所以必須先關閉 s2 container, 才可備份此 container

停止 Container
```
$ docker stop s2  
```


備份 Container
```
$ docker export s2 > s2.tar
```

## 教室的網路架構

![](https://i.imgur.com/I3ces5o.png)
落地雲(Windows) IP 120.96.143.x
透過VMware虛擬橋接器(NAT)使用VMnet8網卡 IP 192.168.61.1/24
連到alp linux虛擬機 IP 192.168.61.x (x>=128)
虛擬機再透過docker虛擬橋接器使用docker0網卡 IP 172.17.0.1/16
連到S2 container IP 172.17.0.x

![](https://i.imgur.com/DXIa76R.png)

右邊這台windows主機建立一台bridged橋接模式的 VM(IP:120.96.143.xx) ，這台VM在建立一台Container，這時候VM會多一張 docker0 的網卡和 docker0 的虛擬橋接器，我們的 Container 會透過虛擬橋接裝置和VM溝通。

透過`docker run -p 8888:8888`，可以在 VM 和 Container 上都開一個 8888 的 port 號，這樣我們就可以在 windows 直接連到我們的 Container。


---

## Docker Container 備份與還原

刪除 Container 和它的 image
```
$ docker rm s2 && docker rmi alpine.sshd
```

還原 image
```
$ cat s2.tar | docker import - alpine.sshd &>/dev/null
```
> `|`，水管會把左邊 cat 看到的檔案內容丟給右邊的`docker import`
> `-`，標準輸入，`docker import`會把收到的檔案內容標準輸入新產生的 alpine.sshd 這個image


驗收有沒有還原成功

```
$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
alpine.sshd       latest    56fa83fb7a1a   12 seconds ago   50.2MB
```

深度研究

```
$ docker history alpine.sshd
IMAGE          CREATED              CREATED BY   SIZE      COMMENT
56fa83fb7a1a   About a minute ago                50.2MB    Imported from -
```

可以發現 image 的內容被消磁，連內定執行的命令都刪除了


## 使用重製 alpine.sshd image

這時，如果沒指定執行的命令，就會爆錯
```
$ docker run --name s2 -h s2 -d alpine.sshd
docker: Error response from daemon: No command specified.
See 'docker run --help'.
```

<font color=red>**重製後的 alpine.plus image 必須指定 執行命令**</font>


再次建立與執行 Container
```
$ docker run --name s2 -h s2 -d alpine.sshd /usr/sbin/sshd -D
```

> `-D` ，**是因為要讓 /usr/sbin/sshd 在前景執行**


強制刪除運行中的 Container
```
$ docker rm -f s2
```

---

# 為何企業要用 Container ？

- Container 相比 VM 消耗資源較少（每個 VM 都擁有 OS，Container 則沒有）
- 精簡功能的邏輯，VM 是減法，Container 則是加法，有需要使用的才加入，效率較高
- 硬體的選擇可以不受限於虛擬化技術（不需要支援 VT-x、AMD SVM）
- 相比 VM 有著快速備份及還原的優勢
- Container 可在不同環境中之間建置、部署，並簡化應用建置流程，降低 human error
- 可以按照企業的需求量身打造 image
- 將功能部署好之後，可以進行消磁處理，只有內部人員知道如何使用
- 製作好的的 image 可以共享、進行全球部署，就像進出口貨櫃一樣

by 第三組


---

# 研究 Docker image 

1. skopeo is a command line utility that performs various operations on container images and image repositories.

2. skopeo does not require the user to be running as root to do most of its operations.

3. skopeo does not require a daemon to be running to perform its operations.

4. skopeo can work with OCI images as well as the original Docker v2 images.

skopeo ( 天蠍 )可以在我們下載 image 之前，幫忙查看 image 的主要資訊

在 Alpine Linux 執行以下命令

```
$ sudo apk add skopeo

$ skopeo -v
skopeo version 1.8.0 commit: b7fc5b608b641cd9b9aec2647f9236e31f8f3b27
```




```bash=
查看 image 的版本代號
$ sudo skopeo list-tags docker://docker.io/apache/ozone
{
    "Repository": "docker.io/apache/ozone",
    "Tags": [
        "0.3.0",
        "0.4.0",
        "0.4.1",
        "0.5.0",
        "1.0.0",
        "1.1.0",
        "1.2.0",
        "1.2.1",
        "latest"
    ]
}

看 image 的資訊
$ sudo skopeo inspect docker://quay.io/cloudwalker/alpine
{
    "Name": "quay.io/cloudwalker/alpine",
    "Digest": "sha256:def822f9851ca422481ec6fee59a9966f12b351c62ccb9aca841526ffaa9f748",
    "RepoTags": [
        "latest"
    ],
    "Created": "2021-04-14T19:19:39.643236135Z",
    "DockerVersion": "19.03.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

## 下載並研究 Docker Image 內部結構


把 image 下載到指定的目錄

```
skopeo --insecure-policy copy docker://quay.io/cloudwalker/alpine.sshd dir:/tmp/alpine.sshd
```

```bash=
$ dir /tmp/alpine.sshd/
total 23M
drwxr-xr-x 2 bigred bigred 4.0K Jun 26 14:31 .
drwxrwxrwt 5 root   root   4.0K Jun 26 14:31 ..
-rw-r--r-- 1 bigred bigred 3.3K Jun 26 14:31 47460458ea8df43635019e60d7c7edead32155d051143b0107c03279a3a3cee0
-rw-r--r-- 1 bigred bigred 2.7M Jun 26 14:31 5843afab387455b37944e709ee8c78d7520df80f8d01cf7f861aae63beeddb6b
-rw-r--r-- 1 bigred bigred 3.1M Jun 26 14:31 8eb646da4db772329c36c99c3edb14fce7cb3415c516104c4333d1325e908cc5
-rw-r--r-- 1 bigred bigred  17M Jun 26 14:31 efde1a2cd25c6076ffd0302bf7bfdd24c2b044772b122460fa9ad4cb32da088a
-rw-r--r-- 1 bigred bigred  951 Jun 26 14:31 manifest.json  ## 結構說明檔
-rw-r--r-- 1 bigred bigred   33 Jun 26 14:31 version
```


manifest.json image 結構的說明檔，裡面說明了壓縮後的目錄 ( rootfs )，以及 config.json


```bash=
$ cat /tmp/alpine.sshd/manifest.json
{    
   "schemaVersion": 2, 
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json", ## config.json 檔
      "size": 3327,
      "digest": "sha256:47460458ea8df43635019e60d7c7edead32155d051143b0107c03279a3a3cee0"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",  ##壓縮檔
         "size": 2811478,
         "digest": "sha256:5843afab387455b37944e709ee8c78d7520df80f8d01cf7f861aae63beeddb6b"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",  ##壓縮檔
         "size": 17282773,
         "digest": "sha256:efde1a2cd25c6076ffd0302bf7bfdd24c2b044772b122460fa9ad4cb32da088a"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",  ##壓縮檔
         "size": 3146213,
         "digest": "sha256:8eb646da4db772329c36c99c3edb14fce7cb3415c516104c4333d1325e908cc5"
      }
   ]
```

後面的壓縮檔解壓縮以後得到的目錄會蓋掉前面的目錄
這些目錄和檔案會透過 overlay2 整合

:::spoiler Docker 的設定檔( 不符合 oci 標準 )

```

```

:::


# Docker Data Volume

- 因為 Container 的檔案系統 Overlay2 不適合用來儲存會有大量新增、修改和刪除資料的動作，所以要 `bypasses the Union File System`，這裡的 `Union File System` 指的就是 Overlay2 檔案系統，讓 Container 跳過使用 Overkay2 檔案系統，直接使用 Linux ext4 的檔案系統，來達到資料永存。
- 甚麼是 Data Volume ?
    - 一個或多個 Container 裡面特別指定的目錄，它會跳過 Overlay2 檔案系統

## Host-Based Persistence Per Container


```bash!
$ docker pull quay.io/cloudwalker/mariadb

# 檢視 mariadb image 內部資訊
$ docker history mariadb
IMAGE             CREATED             CREATED BY                                      SIZE                COMMENT
7aff94e60a52   2 weeks ago         /bin/sh -c #(nop)  CMD ["mysqld"]               0B                  
<missing>       2 weeks ago         /bin/sh -c #(nop)  EXPOSE 3306/tcp              0B                  
<missing>       2 weeks ago         /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B                  
<missing>       2 weeks ago         /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B                 
<missing>       2 weeks ago         /bin/sh -c #(nop) COPY file:f73461a79523c327…   5.65kB              
<missing>       2 weeks ago         /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B             
 .......
    
# 由以上資訊可得知 mariadb image 有建立 /var/lib/mysql 對應的 Data Volume

# 建立 m5 貨櫃主機
$ docker run -d -p 3301:3306 -e MYSQL_ROOT_PASSWORD=admin --name m5 quay.io/cloudwalker/mariadb

檢視 m5 內建 Data Volume 目錄
$ docker inspect --format='{{index .Mounts 0}}' m5 | cut -d' ' -f3
/var/lib/docker/volumes/7539e05831efe4887d4629b81184de82a4dbd13ae2a872cab892f2d4ec11bdc5/_data

$ docker rm -f m5 

$ docker volume ls -f dangling=true
DRIVER  VOLUME NAME
local      7539e05831efe4887d4629b81184de82a4dbd13ae2a872cab892f2d4ec11bdc5

$ docker volume prune
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
7539e05831efe4887d4629b81184de82a4dbd13ae2a872cab892f2d4ec11bdc5

Total reclaimed space: 35.71MB
```

- 由人家的 image 可能會有 Data volume ，當 Container 刪掉， Data Volume 會永存在主機，我們重新再建立一次 Container ，一樣會再次新增另一個 Data volume ，所以有 Data volume 的 Container 重複刪除又建立的話，會在 Host 主機殘留不需要的垃圾。
- `docker volume prune`，Remove all unused local volumes
- 如果要用別人的 image ，要先執行 `docker history <image name>` ，看他的 Dockerfile 有沒有宣告 `VOLUME`


## 認識 Host Data Volume 運作架構


![](https://i.imgur.com/2mzagAJ.png)

- 左邊的 `/opt/web/{config,logs}` 目錄由我們自己建立，可以將左邊的目錄掛載到右邊的 Container 裡面的 `/web/{config,logs}` 目錄上
- Host Data Volume 就是將主機的目錄掛載到 Container 的目錄，當我們在 Container 對目錄進行作業時，實際上是發生在主機的目錄
- 注意， Container 的其他目錄區，仍是使用 Overlay2 檔案系統的目錄，只有我們在 Container 指定的目錄可以跳過 Overlay2 檔案系統，直接使用 Host 主機的檔案系統

### 使用 alpine.derby Image

```bash!
$ docker run --name d1 -d -p 8888:8888 quay.io/cloudwalker/alpine.derby

$ curl http://localhost:8888/db; echo ""
Database Name  = Apache Derby<br>Database Version = 10.14.2.0 - (1828579)<br>Driver Name  = Apache Derby Embedded JDBC Driver<br> 

$ curl http://localhost:8888/hostname; echo ""
Hostname : 5a6e238e1040

$ docker rm -f d1
d1
```

- apache derby ，全世界第一個用 Java 語言寫的資料庫 (Hive 有使用)

### Host Data Volumes


```bash!
$ cd ~/wulin; mkdir db
```
```bash!
$ docker run --name w1 -d -p 8888:8888 -v ~/wulin/db:/derby/db  quay.io/cloudwalker/alpine.derby
```

- `-v ~/wulin/db:/derby/db`，Host Data Volume
    - 將 Host 主機的 `~/wulin/db` 目錄掛載到 Container 主機上的 `/derby/db` 目錄

```bash!
# 在 derby 資料庫產生一個資料表
$ curl http://localhost:8888/db/cars/create;echo ""
cars table created

# 新增兩筆資料
$ curl 'http://localhost:8888/db/cars/insert?id=123&name=star&price=123';echo ""
Add Records ok
$ curl 'http://localhost:8888/db/cars/insert?id=234&name=sun&price=123';echo ""
Add Records ok

# 看資料表的所有資料
$ curl http://localhost:8888/db/cars/list;echo ""
123,star,123<br>234,sun,123<br>

$ docker rm -f w1

$ docker run --name w1 -d -p 9999:8888 -v ~/wulin/db:/derby/db  quay.io/cloudwalker/alpine.derby


$ curl http://localhost:9999/db/cars/list;echo ""
123,star,123<br>234,sun,123<br>

$ docker rm -f w1
```











###### tags: `系統工程`














