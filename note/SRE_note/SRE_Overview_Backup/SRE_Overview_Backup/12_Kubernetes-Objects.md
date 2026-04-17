# Kubernetes-Objects

:::warning

:::spoiler 目錄

[TOC]


---


:::



## 認識 K8S 叢集物件

![](https://i.imgur.com/vVVHNS6.png)

- `CustomResourceDefinitions` ，自定義 K8s 的物件


## K8S Object Short-names

```
Short name       Full name
csr      	  certificatesigningrequests
cm	          configmaps
ds	          daemonsets
deploy	          deployments
ep	          endpoints
ev	          events
hpa      	  horizontalpodautoscalers
ing	          ingresses
ns	          namespaces
pvc	          persistentvolumeclaims
svc	          services
pv	          persistentvolumes
po	          pods
pdb	          poddisruptionbudgets
psp	          podsecuritypolicies
rs	          replicasets
rc	          replicationcontrollers
quota	          resourcequotas
sa	          serviceaccounts
```

# 建立 Kubernetes POD


:::spoiler Node & POD 示意圖

![](https://i.imgur.com/ji3dIzS.png)

:::


- volime 儲存體 ，供 pod 使用
- 一個 pod 可以跑很多台 Container ，代表可以 run 很多 企業要用的 Application


## 認識 Kubernetes Pod

- <font color=red>**Pods are not intended to live long.**</font>
	- pod 很容易領便當
	- K8s 內定，node 主機重開，裡面的 pod 會浴火重生
- This group of containers would share storage, Linux namespaces, cgroups, IP addresses.
	- pod 中的 Container 會共享 volume , Linux Namespaces, cgroups, IP 位址


## Single Container Pod 建立與連接

```bash
# 在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh bigred@<alp.m1 IP>

# 從 1.18 這個版本開始, 下面命令只會產生 pod, 
# 不再會產生 deployment object 及 replicaset controller.
# --image= ，等號右邊的格式為 image 網站/帳號/image 名稱
$ kubectl run a1 --image-pull-policy IfNotPresent --image=quay.io/cloudwalker/alpine.derby
pod/a1 created

# STATUS 的 ContainerCreating，
# 代表 Container 正在被 CRI-O 透過 pull 命令到網路上下載 Container 需要的 image
$ kg pods -o wide
NAME   READY   STATUS              RESTARTS   AGE   IP       NODE   NOMINATED NODE   READINESS GATES
a1     0/1     ContainerCreating   0          64s   <none>   w1     <none>           <none>

# 把 pod 的 IP 位址丟到一個變數
$ podip=$(kg pods -o wide | grep -e "^a1 " | tr -s ' ' | cut -d ' ' -f6)

# 檢查是否符合預期
$ curl http://$podip:8888
<h1>Welcome to Spring Boot</h1>

$ curl http://$podip:8888/hostname
Hostname : a1
```


進入a1 pod
```
$ kubectl exec -it a1 -- bash
```
> `--`分隔符號，後面放的是 pod 要執行的程式
> 連線的動作會經過加密，安全等級不輸 SSH


檢查有無啟動pid 的 namespace
```
# ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0   2180  1672 ?        Ss   06:29   0:00 bash -c /derby/app/startup
root          2  1.5  2.5 3496804 207136 ?      Sl   06:29   0:11 java -jar -Dderby.system.home=/derby/db /derby/app/dt-0.0.1-SNAPSHOT.war
root         44  0.0  0.0   2312  1696 pts/0    Ss   06:40   0:00 bash
root         45  0.0  0.0   1624   520 pts/0    R+   06:41   0:00 ps aux
```
有！

在 pod a1 裡搞破壞

```
rm -r /bin/*
bash-4.4# ls -al /bin/
bash: ls: command not found
bash-4.4# exit
exit
command terminated with exit code 127

bigred@m1:~$ kubectl exec -it a1 -- bash
2022-05-31T06:44:14.000679204Z: 2022-05-31T06:44:14.000679086Z: executable file `bash` not found in $PATH: No such file or directory
```

檢查 node 本機的 /bin

```
bigred@m1:~$ ls -al /bin/
```

可以發現沒被破壞


刪除pod a1

```
$ kubectl delete pod a1
```


---


## Single Container Pod 自動重新建立 


重新建立並執行 a1 pod
```
$ kubectl run a1 --image=quay.io/cloudwalker/alpine -- sleep 15
```

當 pod 的狀態呈現 Compelted 時， K8s 會將 pod 浴火重生，讓 pod 再次 runnning

:::success
監控 pod a1 的狀態

```
$ kubectl get pods a1 --watch
```

:::spoiler 結果

```
NAME   READY   STATUS             RESTARTS      AGE
a1     0/1     CrashLoopBackOff   4 (31s ago)   3m24s
a1     1/1     Running            5 (86s ago)   4m19s
a1     0/1     Completed          5 (101s ago)   4m34s
a1     0/1     CrashLoopBackOff   5 (16s ago)    4m49s
...以下省略
```
<font color=red>當STATUS 出現`CrashLoopBackOff` ，代表 pod 正在不斷被重啟。
	重啟 6 次後, 會將重啟時間拉長, 繼續重啟</font>
:::


刪除 pod a1

```
$ kubectl delete pod  a1
```


---


## K8S Node 系統重新啟動

:::success

```bash=
# 建立並執行 pod a1
$ kubectl run a1 --image=quay.io/ict39/alpine.derby 
pod/a1 created

# 查看 pod a1 的 ip 位址和所在主機
$ kubectl get pod a1 -o wide | grep a1
a1     1/1     Running   0          36s   10.244.2.10   w2     <none>           <none>

# 將 pod a1 所在主機重新開機
$ ssh w2 sudo reboot

# 檢查 pod a1 的狀態
$ kubectl get pod a1 -o wide | grep a1
a1     0/1     ContainerCreating   1          2m23s   <none>   w2     <none>           <none>

# 再次檢查 pod a1 的狀態
$ kubectl get pod a1 -o wide | grep a1
a1     1/1     Running   1          2m35s   10.244.2.11   w2     <none>           <none>

# 刪除 pod a1
$ kubectl delete pod  a1

```

<font color=red>**結論： pod a1 的 IP 位址會改變, 這是因爲 node 重啟時會產生一個新 Pod**</font>
:::


---

## Single Container Pod 永不重啟 

```bash=
$ kubectl run a1 --restart='Never'  --image=quay.io/cloudwalker/alpine -- sleep 15
pod/a1 created

$ kubectl get pods 
NAME   READY   STATUS    RESTARTS   AGE
a1        1/1        Running   0                 10s

$ kubectl get pods 
NAME   READY   STATUS       RESTARTS   AGE
a1        0/1       Completed   0                32s

$ kubectl delete pod a1
pod "a1" deleted

```


---


## 建立 Share PID Namespace Pod

先建立 wulin 工作目錄

```
$ mkdir -p ~/wulin/yaml; cd ~/wulin
```

:::success
宣告 yaml/yml 檔
<font color=red>**超級注意！縮編格式一定要對！不然絕對噴 Error 給你看!**</font>
```bash=
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: sharepid
spec:
  shareProcessNamespace: true
  hostname: xyz 
  containers:
  - name: derby
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Always
  - name: shell
    image: quay.io/cloudwalker/alpine
    imagePullPolicy: IfNotPresent
    tty: true  ' > yaml/sharepid.yml
```
- `kind` ， 宣告要產生什麼
- `metadata` ，會宣告 pod 的名稱
- `spec` ， pod 的結構說明
	- 有兩個 image 代表有兩台 Container ， 名字分別是：derby 和 shell ，他們共用 hostname: xyz
	- `shareProcessNamespace: true` ，兩台 Container 用同一個 pid 的 Namespace
	- `tty: true` ，給一個虛擬終端機，因為 `quay.io/cloudwalker/alpine` 這個 image 內定執行的命令是 sh ，所以一定要有一台虛擬終端機
	- `imagePullPolicy: Always` ，代表每次都要下載 image
	- `imagePullPolicy: IfNotPresent` ， 代表沒有 image 再下載
	- <font color=red>如果企業無法上網，就要宣告`imagePullPolicy: Never`</font>
- 最後要輸出的檔名可以`*.yaml` 或是 `*.yml` 都可以。
:::

	
k8s 標準的運作，透過 yaml 檔作業

```
$ kubectl create -f yaml/sharepid.yml 
```



---

## 檢視 Pod 內部資訊

```
$ kubectl get pod/sharepid -o jsonpath='{.spec.containers[*].name}';echo ""
derby shell
```


相同電腦名稱

```
$ kubectl exec sharepid  -c shell -- hostname; kubectl exec sharepid  -c derby -- hostname
xyz
xyz
```
> -c 指定Container


共用 IP 位址

```
$ kubectl exec sharepid  -c shell -- hostname -i; kubectl exec sharepid  -c derby -- hostname -i
10.244.1.4
10.244.1.4
```

---

## 檢測 Pod 內部 Process


登入 sharepid 中的 shell container

```
$ kubectl exec -it sharepid  -c shell -- sh
/ # whoami
root
/ # apk add curl
.......
/ # curl http://localhost:8888/hostname
Hostname : xyz
```
> 當我們透過localhost來連接不同的 Container 時，會更有效率，速度接近記憶體的速度。


```
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    6 root      0:00 bash -c /derby/app/startup
   11 root      0:18 java -jar -Dderby.system.home=/derby/db /derby/app/dt-0.0.1-SNAPSHOT.war
   53 root      0:00 /bin/sh
   ...........

/ # kill -9 1          無法 強制關閉 PID 1
/ # kill -15 1        可以 正常關閉 PID 1
/ # command terminated with exit code 137
```

> K8s 中所有的 pod 一啟動一定會產生 /pause 這台 Container ，裡面跑的程式是 sleep （睡一輩子）站在資安的角度：全球最安全的命令，老師翻成中文：睡夢羅漢


## 更多 POD 設定

:::success
```bash=
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: twoc
  annotations:
    kubectl.kubernetes.io/default-container: "shell"
spec:
  shareProcessNamespace: false
  containers:
  - name: derby
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Never
  - name: shell
    image: quay.io/cloudwalker/busybox
    command:
      - sleep
      - "60"
  restartPolicy: Never'> yaml/twoc.yml
```
- shareProcessNamespace: false ，如果不宣告，內定是 false。
- restartPolicy: Never ，設定 pod 永不重啟。
:::

建立 pod 

```
$ kubectl apply -f yaml/twoc.yml
pod/twoc created
```

檢查

```
$ kg pods -o wide
NAME       READY   STATUS              RESTARTS       AGE   IP           NODE   NOMINATED NODE   READINESS GATES
sharepid   2/2     Running             2 (6m4s ago)   32m   10.244.1.5   w1     <none>           <none>
twoc       1/2     ErrImageNeverPull   0              59s   10.244.2.6   w2     <none>           <none>
```

發現 w2 的 node 沒有 derby Container 的 image

修改yaml 檔
```
$ nano yaml/twoc.yml

```



# DNS for Service

取得 K8S 叢集的 DNS IP 位址

```bash!
$ kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP  EXTERNAL-IP   PORT(S)                          AGE
kube-dns   ClusterIP  10.98.0.10   <none>        53/UDP,53/TCP,9153/TCP  24h

$ kubectl describe service kube-dns -n kube-system
.........
IP:                10.98.0.10
IPs:               10.98.0.10
Port:              dns  53/UDP
TargetPort:        53/UDP
Endpoints:         10.244.0.6:53,10.244.0.7:53
Port:              dns-tcp  53/TCP
TargetPort:        53/TCP
Endpoints:         10.244.0.6:53,10.244.0.7:53
Port:              metrics  9153/TCP
.......

$ kubectl get pods -n kube-system -o wide | grep coredns
coredns-78fcd69978-5v6cd     1/1     Running   2          5d3h   10.244.0.6      m1     <none>           <none>
coredns-78fcd69978-j4z9r     1/1     Running   2          5d3h   10.244.0.7      m1     <none>           <none>
```

- `kube-dns` ，它的 IP 位址會固定在 `<network_id>.10` ，只有 network_id 可以自己設，預設尾數是 10 ，開的 port : 53
- 透過 `kubectl describe` 看 `kube-dns` 詳細的資訊，可以看到 `Endpoints:` 有兩個 IP 位址，代表後面有兩個 pod 在運作
- 因 pod 浴火重生 IP 位址可能會發生變化，只有 K8S 的服務會鎖在一個 IP 位址，透過 `Selector` 來抓到對應的 pod
    - 服務不會浴火重生，它只是一個儲存在 etcd 裡面的資料區塊，記錄著自己的 IP 位址，及自己真正在運作的 pod 是誰，還有紀錄整個 K8S pod 的 IP 位址及它們的 domain name

![](https://i.imgur.com/ud14Cy4.png)

- 當 K8S 內部的 pod 要上網時， `kube-dns` 會提供 **名稱解析的服務**，讓我們的 pod 得到要上網的 IP 位址。
- 當我們建立一個 mysrv 的 service ，這時候他會去和 `kube-dns` 註冊自己的網址和對應的 IP 位址 ( A record )


![](https://i.imgur.com/PK4jp46.png)

- 無頭服務 ( `headless` ) 的特性 : service 本身不會有 IP 位址，他一樣會去跟 `kube-dns` 註冊，不過註冊的資訊會有 pod 的名字及 pod 的 IP 位址 (以上是老師的小秘笈)
    - 標準的作法是 ，他一樣會去跟 `kube-dns` 註冊，註冊的資訊是自己 service 本身的名字及對應 pod 的 IP 位址


```bash!
$ kubectl run d1 -it --image=quay.io/cloudwalker/alpine
If you dont see a command prompt, try pressing enter.
/ # cat /etc/resolv.conf
search default.svc.k8s.io svc.k8s.io k8s.io localdomain
nameserver 10.98.0.10
options ndots:5

/ # ping www.hinet.net
PING www.hinet.net (203.66.32.110): 56 data bytes

/ # exit
```

**DNS for Service**


編輯 yaml 檔

```bash!
$ nano yaml/service-dns.yaml
```


```yaml=
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
        image: quay.io/cloudwalker/nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
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
  name: svc-headless
spec:
  selector:
    run: k8s-nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None
---
```

**透過 `kubectl create` 產生 yaml 檔中的物件(包含 : deployment, svc-cluster, svc-headless)**

```bash!
$ kubectl create -f yaml/service-dns.yaml
```

檢查

```bash!
$ kg all
NAME                             READY   STATUS    RESTARTS   AGE
pod/k8s-nginx-56975f8f86-f49nr   1/1     Running   0          4m40s
pod/k8s-nginx-56975f8f86-hf4zh   1/1     Running   0          4m40s
pod/k8s-nginx-56975f8f86-qk727   1/1     Running   0          4m40s

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes     ClusterIP   10.98.0.1     <none>        443/TCP   17d
service/svc-cluster    ClusterIP   10.98.0.179   <none>        80/TCP    4m40s
service/svc-headless   ClusterIP   None          <none>        80/TCP    4m40s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/k8s-nginx   3/3     3            3           4m40s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/k8s-nginx-56975f8f86   3         3         3       4m40s
```

檢視 Service endpoints，當前的服務用的是相同的 pod

```bash!
$ kubectl get endpoints svc-cluster
NAME          ENDPOINTS                                      AGE
svc-cluster   10.244.1.35:80,10.244.2.35:80,10.244.2.36:80   8m25s

$ kubectl get endpoints svc-headless
NAME           ENDPOINTS                                      AGE
svc-headless   10.244.1.35:80,10.244.2.35:80,10.244.2.36:80   8m31s
```

**查詢 Service IP**


```bash!
$ nslookup
> server 10.98.0.10
Default server: 10.98.0.10
Address: 10.98.0.10#53
> svc-headless.default.svc.k8s.org
Server:         10.98.0.10
Address:        10.98.0.10#53

Name:   svc-headless.default.svc.k8s.org
Address: 10.244.1.35
Name:   svc-headless.default.svc.k8s.org
Address: 10.244.2.35
Name:   svc-headless.default.svc.k8s.org
Address: 10.244.2.36

> svc-cluster.default.svc.k8s.org
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:   svc-cluster.default.svc.k8s.org
Address: 10.98.0.3
> exit
```

- 透過 `nslookup` ，設定 `DNS server` 是 `10.98.0.10` (Kube-dns) 後，來檢視我們建立的服務 dns 資訊
- `svc-headless.default.svc.k8s.org` 
    - 名字的組成 : `服務名稱.服務所在的namespace.svc.k8s.org`
    - `svc` ，代表是 service 的紀錄
    - `k8s.org` ，是我們在 init K8S 的時候設定的
- `A record`，由一個 `Name` + 一個 `IP Address` 組成


```bash!
> kube-dns.kube-system.svc.k8s.org
Server:         10.98.0.10
Address:        10.98.0.10#53

Name:   kube-dns.kube-system.svc.k8s.org
Address: 10.98.0.10
```

```bash!
$ kubectl run d1 --rm -it --image=quay.io/cloudwalker/alpine
/ # ping svc-cluster
PING svc-cluster (10.98.0.174): 56 data bytes
ping: permission denied (are you root?)

/ # ping svc-headless
PING svc-headless (10.244.1.6): 56 data bytes
ping: permission denied (are you root?)

/ # ping svc-headless
PING svc-headless (10.244.2.10): 56 data bytes
ping: permission denied (are you root?)

/ # ping svc-headless
PING svc-headless (10.244.1.6): 56 data bytes
ping: permission denied (are you root?)
/ # exit
```

- 為何只打 `ping svc-cluster` 名稱就能解析出 IP 位址 ?

```bash!
$ kubectl run d1 --rm -it --image=quay.io/cloudwalker/alpine
If you don't see a command prompt, try pressing enter.
/ # cat /etc/resolv.conf
search default.svc.k8s.org svc.k8s.org k8s.org mcu.edu.tw
nameserver 10.98.0.10
options ndots:5
```

- 因為 `search default.svc.k8s.org svc.k8s.org k8s.org mcu.edu.tw` 
    - 這行表示我們在 `ping svc-cluster` 時，會自動幫我們在名稱後面加上 search 後面的字串做收尋，如 : `ping svc-cluster.default.svc.k8s.org` 找不到時，會在換後面的字串 `ping svc-cluster.svc.k8s.org` 做收尋，依此類推，一直到最後一個字串，如果都沒有就表示無法解析


**建立 POD 之間的 DNS 名稱解析**

```bash!
$ nano ~/wulin/yaml/svcfqdn.yaml
apiVersion: v1
kind: Service
metadata:
  name: dt
spec:
  selector:
    name: busybox
  clusterIP: None

$ kubectl apply -f  ~/wulin/yaml/svcfqdn.yaml 
service/dt created

$ kubectl get svc dt
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
dt        ClusterIP   None           <none>          <none>    23s
```

**新增 Pod 的搜尋網域名**


透過 `nano ~/wulin/yaml/podfqdn.yaml` 來編輯 yaml 檔

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: b1
  labels:
    name: busybox
spec:
  hostname: b1
  subdomain: dt
  containers:
  - image: quay.io/cloudwalker/alpine
    command:
      - sleep
      - "3600"
    name: busybox
    securityContext:
      capabilities:
        add: ["CAP_NET_RAW"]
---
apiVersion: v1
kind: Pod
metadata:
  name: b2
  labels:
    name: busybox
spec:
  hostname: b2
  subdomain: dt
  containers:
  - image: quay.io/cloudwalker/alpine
    imagePullPolicy: Always
    command:
      - sleep
      - "3600"
    name: busybox
    securityContext:
      capabilities:
        add: ["CAP_NET_RAW"]
```

- ```
  spec:
  hostname: b1
  subdomain: dt
  ```
    - subdomain ，可以讓我們在 ping 的時候，在 pod 的名字後面 + `dt` ，就能被解析


```bash!
$ kubectl apply -f ~/wulin/yaml/podfqdn.yaml 

$ kubectl exec -it b2 -- sh
/ # ping -c 1 b1.dt.default.svc.k8s.org
PING b1.dt.default.svc.k8s.org (10.244.2.8): 56 data bytes
.........

# 可解析簡易名稱
/ # ping -c 1 b1.dt
PING b1.dt (10.244.2.8): 56 data bytes
.........

/ # apk add nano; nano /etc/resolv.conf
search  dt.default.svc.k8s.org default.svc.k8s.org svc.k8s.org localdomain
nameserver 10.98.0.10
options ndots:5

/ # ping -c 1 b1
PING b1 (10.244.2.8): 56 data bytes
.........
/ # exit
```


### 新增 Pod 的搜尋網域名

```bash!
$ kubectl delete -f ~/wulin/yaml/podfqdn.yaml

$ nano ~/wulin/yaml/podfqdn.yaml
..............
spec:
  hostname: b2
  subdomain: dt
  containers:
  - image: quay.io/cloudwalker/alpine
    imagePullPolicy: Always
    command:
      - sleep
      - "3600"
    name: busybox
    securityContext:
      capabilities:
        add: ["CAP_NET_RAW"]
  dnsConfig:
    searches:
    - dt.default.svc.k8s.org

$ kubectl apply -f ~/wulin/yaml/podfqdn.yaml 

$ kubectl exec -it b2 -- sh
/ # cat /etc/resolv.conf
search default.svc.k8s.org svc.k8s.org k8s.org localdomain dt.default.svc.k8s.org
nameserver 10.98.0.10
options ndots:5

/ # ping -c 1 b1
PING b1 (10.244.2.3): 56 data bytes
64 bytes from 10.244.2.3: seq=0 ttl=62 time=0.590 ms

--- b1 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.590/0.590/0.590 ms

/ # exit
```



## 佈署 Kubernetes Job Controller


建立 Kubernetes Job Controller

```bash!
$ echo 'apiVersion: batch/v1
kind: Job
metadata:
  name: job1
spec:
  template:
    spec:
      containers:
        - name: job
          image: quay.io/cloudwalker/busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 30s; echo exiting...; date
      restartPolicy: Never ' >  ~/wulin/yaml/job01.yaml

$ ka -f yaml/job01.yaml

$ kg job
NAME   COMPLETIONS   DURATION   AGE
job1   1/1           44s        5m26s

$ kubectl get pod -l=job-name=job1
NAME         READY   STATUS      RESTARTS   AGE
job1-9b8xz   0/1     Completed   0          119m
```


修改 yaml 檔

```yaml=
apiVersion: batch/v1
kind: Job
metadata:
  name: job1
spec:
  template:
    spec:
      containers:
        - name: job
          image: quay.io/cloudwalker/busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 30s; echo exiting...; date
      restartPolicy: Always
```

apply 它


```bash!
$ ka -f yaml/job01.yaml
The Job "job1" is invalid: spec.template.spec.restartPolicy: Required value: valid values: "OnFailure", "Never"
```

檢視 Kubernetes Job Controller


```bash!
$ echo 'apiVersion: batch/v1
kind: Job
metadata:
  name: job2
spec:
  activeDeadlineSeconds: 5
  template:
    spec:
      containers:
        - name: job2
          image: busybox
          args:
            - /bin/sh
            - -c
            - date; echo sleeping....; sleep 30s; echo exiting...; date
      restartPolicy: Never ' >  ~/wulin/yaml/job02.yaml

$ ka -f yaml/job02.yaml
```

- `activeDeadlineSeconds: 5` ，設定 Job 只能跑 5  秒，時間到會停止













###### tags: `系統工程`


