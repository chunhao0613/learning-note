# Project Quay 練習

[TOC]

## 學習目標

1. Red Hat Quay / Project Quay
2. Project Quay 部署
3. Image 上傳落地 Quay
4. K8S 使用落地 Quay
5. Docker Image 設計 (httpd)
6. Docker Image 設計 (sshd)
7. Image 備份與還原


---

## Red Hat Quay

Red Hat 這家公司有做了一個系統 OpenShift，裡面就是 k8s Cluster 的完整生態系。
而 [Quay](https://Quay.io) 就是 image 儲存的空間 （ 光片中心 ），它是包裝在 OpenShift 裡面。

- docker hub
	- docker 的 image 儲存空間
	- 1個匿名 ip 在六小時內，只能下載 100 次光碟片，超過要付錢。
- Red Hat Quay 
	- 也可以下載 image ，不過下載都不用錢，無限制。
- Projecct Quay 
	- 企業如果要離線儲存 image 就要自己建立 Project Quay 來達到目的，也可以個人自己建立來使用，Red Hat Quay 的開源發行版。


## 部署前置作業

```bash=
在 K8s Master 執行

## 安裝 acl 套件
$ sudo apk update; sudo apk add acl

## 透過 podman 來 run project-Quay (落地Quay or 個人光片中心)
$ wget wget https://web.flymks.com/quay-podman.tar.gz
$ tar xvfz quay-podman.tar.gz; cd quay-podman

## 執行程式
$ ./keyman.sh
Usage:
  keyman.sh [options]

Available options:

ps       podman ps -a
start    deploy quay
stop     delete quay
stop -v  delete quay and hostPath volume
clean    delete hostPath volume
```

檢視目錄結構

```
$ tree
.
├── config
│   └── config.yaml
├── initdb.sql
├── keyman.sh
└── quay-pod.yaml   ## 定義落地 quay 
```

更改 k8s master node (m1) 和 worker nodes ( w1 , w2 ) 的 /etc/hosts
```bash=
## 修改後的結果
$ sudo nano /etc/hosts
127.0.0.1 localhost alp
192.168.61.0 x1
192.168.61.1 x2
192.168.61.2 x3
192.168.61.3 a1
192.168.61.4 m1 quay
192.168.61.5 m2
192.168.61.6 w1
192.168.61.7 w2
192.168.61.8 w3
192.168.61.9 w4
```

## 部署 Quay

Quay 部署時需要初始化，需等待3~5分鐘
```
$ sed -i "s|192.168.1.197|${IP}|g" keyman.sh


$ ./keyman.sh start
```

## Image 上傳落地 Quay

```
$ sudo podman images | grep 1.35.0
REPOSITORY                 TAG     IMAGE ID      CREATED      SIZE
quay.io/flysangel/busybox  1.35.0  af4897967787  5 days ago   1.47 MB
```


設定 image 的新 tag
```
$ sudo podman tag quay.io/flysangel/busybox:1.35.0 $IP/myquay/busybox:1.35.0
```
> 後面的`$IP/myquay/busybox:1.35.0` ，$IP，代表我們的落地 quay 的 ip 位址
> myquay 是落地 quay 的帳號
> `busybox:1.35.0` ， image 的名字和代號

- 當版本是 latest 時，代表該光片維護者認定的最新安全穩定的版本，不一定是功能最多最新的。

檢查
```
$ sudo podman images | grep 1.35.0
REPOSITORY                   TAG     IMAGE ID      CREATED      SIZE
192.168.61.4/myquay/busybox  1.35.0  af4897967787  5 days ago   1.47 MB
quay.io/flysangel/busybox    1.35.0  af4897967787  5 days ago   1.47 MB
```

登入落地 Quay
```
$ sudo podman login $IP

Username: myquay
Password: myquay2022
```

噴錯
```
Error: authenticating creds for "192.168.61.4": pinging container registry 192.168.61.4: Get "https://192.168.61.4/v2/": dial tcp 192.168.61.4:443: connect: connection refused
```

加上參數 `--tls-verify=false` 再傳一次
```
$ sudo podman login --tls-verify=false $IP
Username: myquay
Password: myquay2022
Login Succeeded!
```

> 因為我們沒有 https ，所以要靠 `--tls-verify=false`解決它


Image 上傳落地 Quay

```
$ sudo podman push --tls-verify=false $IP/myquay/busybox:1.35.0
```

到quay 的網站上檢查

![](https://i.imgur.com/P8Vvqyi.png)

## 練習

請先 `pull quay.io/flysangel/alpine:3.15.4` 並將 image push 至落地 quay

請先 `pull quay.io/flysangel/nginx:1.22.0` 並將 image push 至落地 quay



```bash=
# 第一題
## 拉 image
sudo podman pull quay.io/flysangel/alpine:3.15.4

## 檢查
sudo podman images | grep alpine

## 改 image 的標籤
sudo podman tag quay.io/flysangel/alpine:3.15.4 $IP/myquay/alpine:3.15.4

## 檢查
sudo podman images | grep alpine

## 將 image 上傳到落地 quay
sudo podman push --tls-verify=false $IP/myquay/alpine:3.15.4

# 第二題
## 拉 image
sudo podman pull quay.io/flysangel/nginx:1.22.0

## 檢查
sudo podman images | grep nginx

## 改 image 的標籤
sudo podman tag quay.io/flysangel/nginx:1.22.0 $IP/myquay/nginx:1.22.0

## 檢查
sudo podman images | grep nginx

## 將 image 上傳到落地 quay
sudo podman push --tls-verify=false $IP/myquay/nginx:1.22.0
```

---


Image 上傳完之後通常都是私人的（不開放公開下載）， 如果要公開設定如下

![](https://i.imgur.com/TpZRpKX.png)

點右邊的齒輪，設成 Make Public



---

查看 k8s 提交的物件 運作資訊

```
$ kubectl describe < 物件 > < 物件名稱 >

## 舉例 pod 
$ kubectl describe pod a1
```

> describe 裡面的詳細資訊都存在 etcd 裡面

看 pod 裡面 Container 運作過程的 log

```
$ kubectl logs  < pod name >
```

> logs 的命令是 k8s 進到 Container 裡面把資訊輸出到螢幕上，Container 如果掛了，就抓不到他的 logs 了。


---

## Container Image 設計

:::success


```bash=
$ mkdir alpine.httpd
$ nano alpine.httpd/Dockerfile
FROM 192.168.61.4/myquay/alpine:3.15.4
RUN \
  apk update && \
  apk add --no-cache nano sudo bash wget curl git tree grep && \
  wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
  chmod +x busybox-x86_64 && \
  mv busybox-x86_64 bin/busybox1.28 && \
  mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

ENTRYPOINT ["/bin/busybox1.28"]
CMD ["httpd", "-f", "-h", "/opt/www"]
```

`ENTRYPOINT ["/bin/busybox1.28"]`，代表等等透過 Dockerfile 建立出來的 Container 裡面第一支跑的程式為`/bin/busybox1.28`

`CMD ["httpd", "-f", "-h", "/opt/www"]` ，代表程式後面接的參數為 `httpd -f -h /opt/www` ( `-f` ，前景執行; `-h` ，指定網頁目錄 )

:::



:::success

```bash=
$ mkdir alpine.sshd
$ echo 'FROM 192.168.61.4/myquay/alpine.base:1.0.0
RUN \
  apk update && \
  apk add --no-cache openssh-server tzdata && \
  # 設定時區
  cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
  ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
  echo -e "Welcome to ALP SSHD 6000\n" > /etc/motd && \
  # 建立管理者帳號 bigred
  adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && \
  echo "%wheel   ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
  echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && \
  rm /sbin/reboot && rm /usr/bin/killall
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]' > alpine.sshd/Dockerfile 
```

- `/usr/sbin/sshd` ，執行 ssh Server 的命令
- `-D` ，前景執行 (sshd will not detach and does not become a daemon)
- `ENTRYPOINT`，Container 啟動後預設不可隨意更改的程式
- `CMD`，當前面有 `ENTRYPOINT` 時，會變成給參數

```
$ kubectl run a1 --image=$IP/myquay/alpine.sshd:1.0.0
pod/a1 created
```



失敗原因：CRI-O 原本就不允許使用 SSH 連線到 Container
要自訂 pod 的 yaml 檔，破格 CRI-O 的 `capabilities`

```bash=
$ nano pod-ssh.yaml
apiVersion: v1
kind: Pod
metadata:
  name: a1
  labels:
    app: a1
spec:
  containers:
    - name: a1
      image: 192.168.61.4/myquay/alpine.sshd:1.0.0
      imagePullPolicy: IfNotPresent
      securityContext:
        capabilities:
          add: ["AUDIT_WRITE", "SYS_CHROOT", "CAP_NET_RAW"]
  restartPolicy: Always
```
:::


:::info
CRI-O default 的 capabilities

```
  default_capabilities = [
	  "CHOWN",
	  "DAC_OVERRIDE",
	  "FSETID",
	  "FOWNER",
	  "SETGID",
	  "SETUID",
	  "SETPCAP",
	  "NET_BIND_SERVICE",
	  "KILL",
  ]
```
網址：[CRI-O default 的 capabilities 連結](https://github.com/cri-o/cri-o/blob/main/docs/crio.conf.5.md)
:::



---

## 練習2

:::success

題目：
請設計一個 Container Image 名為 alpine.base，代號 1.0.0，並滿足以下功能
需要能啟動 httpd
具有 bigred、rbean、gbean 帳號，密碼與帳號相同
登入 alpine 提示為 `Welcome to ALP base 6666`
第一隻程式為 sshd

將 Docker Image 上傳至落地 Quay
在 K8S 使用 alpine.base
`kubectl apply -f pod-base.yaml`
ssh bigred@a1_ip 並執行 sudo busybox1.28 httpd -f -h /opt/www
ssh rbean@a1_ip 並執行 curl localhost

答案：

```bash=
$ nano alpine.base/Dockerfile
FROM 192.168.61.4/myquay/alpine:3.15.4
RUN \
  apk update && \
  apk add --no-cache nano sudo bash wget curl git tree grep \
    elinks shadow procps util-linux coreutils binutils \
    findutils openssh-server tzdata && \
  # 啟動httpd
  wget https://web.flymks.com/busybox-x86_64 && \
  chmod +x busybox-x86_64 && \
  mv busybox-x86_64 bin/busybox1.28 && \
  mkdir -p /opt/www && echo "let me go" > /opt/www/index.html && \
  # 設定時區
  cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
  ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
  echo -e 'Welcome to ALP base 6666\n' > /etc/motd && \
  # 建立管理者帳號 bigred
  adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && \
  echo "%wheel   ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
  echo -e 'bigred\nbigred\n' | passwd bigred &>/dev/null && \
  rm /sbin/reboot && rm /usr/bin/killall && \
  # 建立一般使用者 rbean gbean
  adduser -s /bin/bash -h /home/rbean -D rbean && \
  echo -e 'rbean\nrbean\n' | passwd rbean && \
  adduser -s /bin/bash -h /home/gbean -D gbean && \
  echo -e 'gbean\ngbean\n' | passwd gbean
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```

```
sudo podman build --tls-verify=false -t $IP/myquay/alpine.base:1.0.0 alpine.base
```

```
sudo podman images | grep base

sudo podman push --tls-verify=false $IP/myquay/alpine.base:1.0.0
```


```
$ nano pod-base.yaml
apiVersion: v1
kind: Pod
metadata:
  name: a1
  labels:
    app: a1
spec:
  containers:
    - name: a1
      image: 192.168.61.4/myquay/alpine.base:1.0.0
      imagePullPolicy: IfNotPresent
      securityContext:
        capabilities:
          add: ["AUDIT_WRITE", "SYS_CHROOT", "CAP_NET_RAW"]
  restartPolicy: Always
```


```
$ kubectl apply -f pod-base.yaml
$ kg pods -o wide
$ ssh bigred@10.244.2.13 sudo busybox1.28 httpd -f -h /opt/www
$ ssh rbean@10.244.2.13
```










:::




---


## Container Image 設計 (fbs)




---





```bash=
$ nano svc-fbs.yaml
kind: Service
apiVersion: v1
metadata:
  name: svc-fbs
spec:
  externalIPs:
    - 192.168.61.4
  ports:
    - port: 8080
      targetPort: 4000
  selector:
    name: pod-fbs
```

`externalIPs` 可以讓我們的 pod 從外面連進來




---


[Dockerfile 物件的參考連結](https://https://docs.docker.com/engine/reference/builder/)
[k8s pod](https://kubernetes.io/docs/concepts/workloads/pods/)


## Distroless Image

```b!
$ mkdir distroless
$ wget https://web.flymks.com/app_golang/v1/goapp -O distroless/goapp; chmod a+x distroless/goapp
```


測試 goapp

```b!
$ ./distroless/goapp &
$ curl -w '\n' localhost:8080 
{"message":"Hello Golang"}
$ pkill -f 'goapp'
```

```b!
$ echo 'FROM scratch
COPY goapp /
ENTRYPOINT ["/goapp"]' > distroless/Dockerfile
```

- 因 執行的是編譯好的 GO 語言的網站，在任何作業系統都能直接跑，
- 所以可以從一個空白的 image 開始，而不用 `FROM alpine`
- 好處是 image 會更小

## Distroless Image 建立與測試

```bash!
$ sudo podman build --tls-verify=false -t $IP/myquay/goapp:1.0.0 distroless 
```

- `build` ，Build an image using instructions from Containerfiles
- `--tls-verify` ，require HTTPS and verify certificates when accessing the registry (default true)
- `-t, --tag name` ， tagged name to apply to the built image
- `distroless`，目錄名稱，裡面放需要 build image 的檔案 (Dockerfile)

output : 

```bash!
STEP 1/3: FROM scratch
STEP 2/3: COPY goapp /
--> 0411ca03d77
STEP 3/3: ENTRYPOINT ["/goapp"]
COMMIT 192.168.64.21/myquay/goapp:1.0.0
--> 68227ba71f7
Successfully tagged 192.168.64.21/myquay/goapp:1.0.0
68227ba71f76403c49144cc9edcbe7cb15ae132f691c594e8946102b8c830c7
```

```bash!
$ sudo podman run --rm --name a1 -d -p 8080:8080 $IP/myquay/goapp:1.0.0
```

```bash!
$ curl -w '\n' localhost:8080 
```

output : 

```bash!
{"message":"Hello Golang"}
```

## Distroless Image 上傳落地 Quay

```bash!
$ sudo podman stop a1
```

```bash!
$ sudo podman login --tls-verify=false $IP
Username: myquay
Password: myquay2022
```

Login Succeeded!

```bash!
$ sudo podman push --tls-verify=false $IP/myquay/goapp:1.0.0 
```

## K8S 使用 goapp


```bash!
K8S 執行
$ kubectl run a1 --image=$IP/myquay/goapp:1.0.0
$ kubectl get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP             NODE
a1     1/1     Running   0          2m1s   10.233.1.118   w1

$ curl -w '\n' 10.233.1.118:8080 
{"message":"Hello Golang"}

刪除 a1 pod
$ kubectl delete pod a1
```


# image 建置流程

1. 建存放 image 的資料夾
2. 編輯 Dockerfile
3. 建置 image (`podman build image `)
4. 測試 image 功能是否正常 (`podman run`)
5. 上傳至 image 中心 (`podman push`)
6. image 設公開 ( Quay make public )
7. 用 K8s 跑一次 ( `kubectl run ` )


## 進企業後 image 建置流程

1. 建立 image 的資料夾
2. 編輯 Dockerfile
3. podman buid
4. 測試 podman run 
    - 本地測試
    - CI ( 公司內部規定的一些測試 )
    - 測試環境
    - 正式環境
5. image 上傳至落地 QUAY
6. image public
7. kubectl run 












###### tags: `系統工程`








