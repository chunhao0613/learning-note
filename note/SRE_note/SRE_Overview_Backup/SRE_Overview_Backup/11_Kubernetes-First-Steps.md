# Kubernetes-First-Steps

:::warning

:::spoiler 目錄

[TOC]


---


:::

## 建立 K8S 應用物件方式

- `kubectl run` (命令式 Imperative) - Manage K8s object (POD, Controller, Service) using CLI
    - 透過以上命令式建立 pod 能做到的功能是有限的。
    - e.g., 像要讓 pod 到指定 node 去 run 是做不到的; K8S 還有很多物件是無法用 `kubeclt run` 的方法產生
    - `kubectl run` (Imperative) 這命令適合做概念性(物件的雛形)驗證 (Proof of Concept；POC)
    - 結論 : `kubeclt run` 的方法 對於 K8S 能產生和設定多少物件是有限的
- `kubectl create/apply` (聲明式 Declarative) - By defining K8s objects in yaml file
    - K8S 產生物件的正規做法，但有個必要條件，須先設計 yaml 檔，才能產生對應的物件
    - `kubectl create` (Declarative) 與 yaml 設定檔適合建立複雜的企業應用服務
- "Imperative" is a command - like "create 42 widgets".
- "Declarative" is a statement of the desired end result - like "I want 42 widgets to exist".
- For creating multiple complex object, declarative approach is preferred as it follows Infrastructure as a Code approach. However, imperative approach is also useful when we want to do quick POC and save yourself from writing yaml files. Let's take a look at some imperative commands.


# Using Namespace

- 在 K8S 裡的 Namespace ，可以想像成大樓裡的辦公室，裡面可以有很多人 (Container)，辦公室中還可以有很多桌子、椅子和影印機...等物件(pod, deployment, Service...等)
- **Partition Cluster resources**.
- Kubernetes Namespaces can be used to divide a cluster into logical partitions allowing a single large Kubernetes cluster to be used by multiple users and teams, or a single user with multiple applications. Each user, team, or application running in a Namespace, is isolated from every other user, team, or application in other Namespaces and they operate as if they are the sole user of the cluster (note that Namespaces do not provide network segmentation).
- Namespaces 有以下幾個特點
    - 在同一個 Kubernetes Cluster 中，每個 Namespaces 的名稱都是要獨特的
    - 當一個 Namespaces 被刪除時，在該 Namespace 裡的所有物件也會被刪除
        - 注意!只會刪除 K8S 產生的物件，如果 K8S 的物件有產生檔案或目錄是不會被刪除的。
    - 可以透過 Resource Quotas 限制一個 Namespaces 所可以存取的資源
        - 不只限制可用的運算資源，還能限制 K8S 的物件
        - 有防火牆的機制，可以允許或禁止 pod 連到另一個 Namespace 的 pod ，或是讓一個 pod 可以上網
- 結論 : K8S 的物件一定要指定一個 Namespace 給它 run。開發應用系統必定會建立專案目錄, 設計 K8S Application 需在 Namespace, 來運作

**檢視 K8S 當前所有 Namespace 的資訊**

```bash!
$ kubectl get ns
NAME              STATUS   AGE
default           Active   10d
kube-flannel      Active   10d
kube-node-lease   Active   10d
kube-public       Active   10d
kube-system       Active   10d
```

- `default`，預設的 Namespaces 名稱為 default，過去我們產生的物件像是， Deployment， Services 等若沒特別指定 Namespace 都是存放在名稱為 default 的 namespaces 中。
- `kube-public`，是個特殊的 namespace，存放在裡面的物件可被所有的使用者讀取。

```bash!
$ kubectl get pods -n kube-system
NAME                         READY   STATUS    RESTARTS   AGE
coredns-6d4b75cb6d-tslq8     1/1     Running   5          10d
coredns-6d4b75cb6d-vqz9h     1/1     Running   5          10d
etcd-m1                      1/1     Running   5          10d
kube-apiserver-m1            1/1     Running   5          10d
kube-controller-manager-m1   1/1     Running   5          10d
kube-proxy-274g9             1/1     Running   4          10d
kube-proxy-kqlrm             1/1     Running   4          10d
kube-proxy-zw6sb             1/1     Running   5          10d
kube-scheduler-m1            1/1     Running   5          10d
```

- `kube-system`，在 Kubernetes 中，較特別的資源都會存放在 kube-system 這個 namespace。若是用 `kubectl get all -n kube-system` 查看，可以發現 kube-dns 或是 kube-apiserver...等維運系統的 pod 都是存放在該 namepsace 中。
- Namespace 參考文章連結 : https://ithelp.ithome.com.tw/m/articles/10197186

## K8S default Namespace

```bash!
$ kubectl create deployment --image quay.io/cloudwalker/nginx demo-nginx
deployment.apps/demo-nginx created

$ kubectl describe deployment demo-nginx | grep Namespace
Namespace:              default

$ kubectl create deployment --image quay.io/cloudwalker/nginx demo-nginx
Error from server (AlreadyExists): deployments.apps "demo-nginx" already exists

$ kubectl create deployment --image quay.io/cloudwalker/nginx demo-nginx -n kube-public
```



## Assigning Pods to Namespace

```bash!
# 建立工作目錄
$ mkdir -p ~/wulin/yaml; cd ~/wulin

# 建立新的 namespace
$ kubectl create namespace myoffice
namespace/myoffice created

# 檢查有無建立成功
$ kubectl get namespace
NAME              STATUS   AGE
default           Active   10d
kube-flannel      Active   10d
kube-node-lease   Active   10d
kube-public       Active   10d
kube-system       Active   10d
myoffice          Active   2s

# 刪除 myoffice namespace
$ kubectl delete namespace myoffice

# 再次建立 myoffice namespace
$ kubeclt create namespace myoffice

# 檢查
$ kubectl get namespace
```


建立 pod

```bash!
$ kubectl run n1 --image=quay.io/cloudwalker/alpine -n myoffice -- sleep infinity
```

- `kubectl run`，用指定 image 產生 pod
- `n1`，pod 的名字
- `-n myoffice`，指定等等的 pod 會建在 myoffice 這個 namespace
- `-- sleep infinity`，`--`後面放的是 Container 要執行的程式 ，這裡是 `sleep infinity`
    - forever sleeping ( never awake until escape )

檢視 pod 的狀態

```bash!
$ kg pods
NAME    READY   STATUS    RESTARTS   AGE
gocgi   1/1     Running   1          20h
mydb    1/1     Running   1          21h
pod1    1/1     Running   1          16h
```

- 這裡看到的是在 default namespace 的 pod 資訊

```bash!
$ kg pods -n myoffice
NAME   READY   STATUS    RESTARTS   AGE
n1     1/1     Running   0          22m
```

- 這裡看到的是 myoffice 這個 namespace 裡面 pod 資訊

```bash!
$ kubectl exec -it n1 sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Error from server (NotFound): pods "n1" not found
```
- 錯誤訊息翻譯:
    - `kubectl exec [POD] [COMMAND]` ，這個用法以被棄用，且即將在未來的版本被刪除
    - 要使用 `kubectl exec [POD] -- [COMMAND]`，Command 前面要加 `--`
    - `Error from server (NotFound): pods "n1" not found`，找不到 n1 這台 pod，因未指定 Namespace 的話，預設會到 default 這個 namespace 裡面找有沒有 n1 這個 pod
- 用法須更正為:
    - `kubectl exec -it n1 -n myoffice -- sh`

```bash!
$ kubectl exec -it -n myoffice n1 -- sh
/ # ping -c 1 www.hinet.net
PING www.hinet.net (163.28.83.113): 56 data bytes
ping: permission denied (are you root?)

# 檢視 NameServer 是誰
/ # cat /etc/resolv.conf
search myoffice.svc.k8s.org svc.k8s.org k8s.org localdomain
nameserver 10.98.0.10
options ndots:5

/ # exit
```

- `nameserver 10.98.0.10`，這是 K8S 的 DNS Server
- 它會幫我們把 `www.hinet.net` 這個名稱丟到網路去問，最後解析出 `163.28.83.113` 這個 IP 位址
- `ping: permission denied (are you root?)`，權限不足，Linux Capaibility 不給權限

用 yaml 檔建立 pod

```bash!
$ nano ~/wulin/yaml/pod-n2.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: n2
  namespace: myoffice
  labels:
    name: n2
spec:
  hostname: n2
  containers:
  - image: quay.io/cloudwalker/busybox
    command:
     - sleep
     - "60"
    name: n2
```

- `kind` ，指定要建立 pod 這個物件
- `metadata`，設定重要資訊
    - `name: n2`，pod 名字 : n2
    - `namespace`，設定 pod 所在的 namespace : myoffice
    - `labels`，pod 上貼一個標籤
- `spec`，設定 pod 內部的結構
    - `hostname: n2`，設定 Container 的 hostname，多個 Container 也會共用同一個 hostname
    - `contaienrs`，設定 Container 的資訊
    - `command`，container 要先執行的命令
        - yaml 檔中的 `command:` 這個宣告, 會覆蓋掉 image 的內定命令, 包括 entrypoint 指定的命令
    - `name`，設定 Container 在系統上的名稱


apply 它

```bash!
$ kubectl apply -f ~/wulin/yaml/pod-n2.yaml
```

檢查是否符合預期

```bash!
$ kubectl get pods -n myoffice
NAME   READY   STATUS    RESTARTS   AGE
n1     1/1     Running   0          61m
n2     1/1     Running   0          14s
```

### 切換 Namespace

```bash!
$ kubectl config set-context --current --namespace=myoffice
```

- `set-context`，進入 K8S 的路口
- `set-context` ，入口的意思，進入商業辦公大樓的大門
- `--current` ，入口指定預設的入口，因為進入的方法有很多種，我們就走預設的入口
- `--namespace=myoffice` ，進到 myoffice 這個 namespace

```bash!
$ kubectl get pods
NAME   READY   STATUS    RESTARTS       AGE
n1     1/1     Running   0              25m
n2     1/1     Running   5 (105s ago)   8m32s

$ cat ~/.kube/config | grep -A 6 contexts:
contexts:
- context:
    cluster: kubernetes
    namespace: myoffice
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
```

- `~/.kube/config`，進入 K8S 這棟大樓的磁卡 (裡面記錄著使用者在 K8S 的名字和識別代號，及擁有多大的權限可以對 K8S 的物件進行訪問...等資訊)
- `contexts`，大樓的入口，後面加 `s` 是因為可能會有很多入口
    - `context`，大樓的入口
    - `cluster: kubernetes`，大樓的名字 : `kubernetes`
    - `namespace: myoffice`，大樓裡的辦公室名字: `myoffice`
    - `user: kubernetes-admin`，使用者進入大樓辦公室用的身分名稱

```bash!
$ cat ~/.kube/config | grep users -A1
users:
- name: kubernetes-admin
```

- 在 K8S 系統中，我們 bigred 的帳號，是 kubernetes-admin (由 `~/.kube/config` 這個檔案得知)


# OpenSSH 實務應用

## Container Image 設計與建立

在 m1 終端機, 執行以下命令

```bash!
$ mkdir -p ~/wulin/{sshd,fbs}

$ echo $'FROM quay.io/cloudwalker/alpine
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux git coreutils binutils \
    findutils grep openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e \'Welcome to ALP sshd 6000\\n\' > /etc/motd && \
    # 建立管理者帳號 bigred
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && \
    echo "%wheel   ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    echo -e \'bigred\\nbigred\\n\' | passwd bigred &>/dev/null && \
    rm /sbin/reboot && rm /usr/bin/killall

EXPOSE 22

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]' > ~/wulin/sshd/Dockerfile
```

- `ENTRYPOINT ["/usr/sbin/sshd"]` ，強制宣告等下 run 的 Container 等等只能跑 `/usr/sbin/sshd` 這個命令
- `CMD ["-D"]` ，上面命令的參數


Container Image 建立與測試  

```bash=
$ sudo podman build -t alp.sshd ~/wulin/sshd/

$ sudo podman images | grep alp.sshd
localhost/alp.sshd   latest  01424a71e3d6  3 minutes ago  67.6 MB

$ sudo podman run --name a1 -h a1 -d -p 22100:22 alp.sshd
220ce94e636ce9270604bd31a7dd5d515be6008ab6351295cd10a66da5d978d1

$ ssh bigred@$IP -p 22100
bigred@192.168.61.4 s password: bigred
Welcome to ALP sshd 6000

a1:~$ git version
git version 2.30.2

a1:~$ exit

$ sudo podman rm -f a1
```

# 第一次接觸 Kubernetes App

建立 K8S 應用物件方式

1. `kubectl run` (命令式 Imperative) - Manage K8s object (POD, Controller, Service) using CLI
2. `kubectl create/apply` (聲明式 Declarative) - By defining K8s objects in yaml file
    - yaml ，全名 : Yet Another Markup Language ( 因為不用再看到 xml 檔了 )
    
For creating multiple complex object, declarative approach is preferred as it follows Infrastructure as a Code approach. However, imperative approach is also useful when we want to do quick POC and save yourself from writing yaml files. Let's take a look at some imperative commands.

> kubectl run (Imperative) 這命令適合做概念性驗證 (Proof of Concept；POC), kubectl create (Declarative) 與 yaml 設定檔適合建立複雜的企業應用服務


## 建立 Demo App

```bash!
# 建立一個 pod ，image 用的是 alp.sshd
$ kubectl run demo --image=alp.sshd

[註] 從 1.18 這個版本開始, 上面命令只會產生 pod, 不再會產生 deployment object 及 replicaset controller. 

# 為何 pod 會建立失敗 ? ErrImagePull ?
$ kg pods -o wide
NAME    READY   STATUS         RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
demo    0/1     ErrImagePull   0          13s   10.244.2.5   w2     <none>           <none>

# 因為在我們的工作主機上，沒有這片 image，所以先刪除 pod
# --force ，強制刪除
$ kubectl delete pods demo --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "demo" force deleted

# 打包 alp.sshd 這個 image
$ sudo podman save alp.sshd > alp.sshd.tar

# 將打包檔拷貝到 w1 和 w2 主機
$ scp alp.sshd.tar w1:alp.sshd.tar; scp alp.sshd.tar w2:alp.sshd.tar

# 在 w1 和 w2 主機，將打包檔的 image 還原
$ ssh w1 'sudo podman load < alp.sshd.tar'
$ ssh w2 'sudo podman load < alp.sshd.tar'

# 再 run 一次 pod
# --image-pull-policy IfNotPresent ，如果本地端有 image 就不用上網下載了
$ kubectl run demo --image-pull-policy IfNotPresent --image=localhost/alp.sshd
pod/demo created

# 以下命令會失敗的原因是因為， image 前面不加 localhost ，就會上網飆到 docker.io/library 抓 image
$ kubectl run demo --image=alp.sshd

# 其實不加 --image-pull-policy IfNotPresent 也會過
$ kubectl run demo --image=localhost/alp.sshd

# 檢查是否符合預期
$ kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
demo   1/1     Running   0    3m12s   10.244.2.2   w2     <none>           <none>
```

## 建立 OpenSSH Pod

編輯 sshd yaml 檔

```bash!
$ nano ~/wulin/sshd/pod-sshd.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: pod-sshd
spec:
  containers:
  - name: mysshd
    image: localhost/alp.sshd
    imagePullPolicy: IfNotPresent
    command:
      - /bin/bash
      - -c
      - |
        adduser -s /bin/bash -h /home/rbean rbean
        echo -e 'rbean\nrbean\n' | passwd rbean
        /usr/sbin/sshd -D
```

- 在 K8S yaml 檔的 `command` 能破格 Dockerfile 設定的 `ENTRYPOINT`，讓 Container 跑 `command` 宣告的命令
- 但是 Container run 的程式還是要在前景執行
- 所以 `command` 最後還是要宣告 `/usr/sbin/sshd -D`，讓 Container 有個 `SSH Server` 一直在前景執行的命令
- `/bin/bash -c`，`c` 的意思是 command，合起來就是讓 bash 貝殼程式跑下面的命令
- `|`，這裡的 pipe 意思是會保留以下宣告命令的格式，有空格就空格，有換行就換行，通通保留，如果沒有宣告，貝殼程式會把下面 3 行命令當成一行

用 apply 產生 pod

```bash!
$ kubectl apply -f ~/wulin/sshd/pod-sshd.yaml
```

檢查 pod 狀態

```bash=
$ kg po pod-sshd -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
pod-sshd   1/1     Running   0          2m    10.244.1.37   w1     <none>           <none>
```

SSH 連線測試 pod 功能

```bash!
$ ssh rbean@10.244.1.37
rbean@10.244.1.37s password:
Welcome to ALP sshd 6000

# 建立 zzz 目錄
pod-sshd:~$ mkdir zzz
pod-sshd:~$ ls -ld zzz
drwxr-sr-x 2 rbean rbean 4096 Sep  1 09:49 zzz
pod-sshd:~$ exit
logout
Connection to 10.244.1.37 closed.
```

刪除 pod 

```bash!
$ kubectl delete pod pod-sshd
pod "pod-sshd" deleted
```

再次建立 pod

```bash!
$ kubectl apply -f ~/wulin/sshd/pod-sshd.yaml
pod/pod-sshd created
```

連進 pod 檢視 zzz 目錄是否還在 ?

```bash!
$ kubectl get pod pod-sshd -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
pod-sshd   1/1     Running   0          73s   10.244.1.38   w1     <none>           <none>

# 用 rbean 帳號連進 pod
$ ssh rbean@10.244.1.38
Warning: Permanently added '10.244.1.38' (RSA) to the list of known hosts.
rbean@10.244.1.38s password:
Welcome to ALP sshd 6000

# 檢視剛剛建立的目錄
pod-sshd:~$ ls -l
total 0
```

- 結論 : pod 只要沒有動用 volume ，當 pod 被刪除時，資料通通消失

將 pod 所在的 w1 這台機器重開機

```bash!
$ ssh w1 'sudo reboot'
```

再次檢視 pod 狀態

```bash!
$ kubectl get pods pod-sshd -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
pod-sshd   1/1     Running   1          20m   10.244.1.39   w1     <none>           <none>
```

- 發現 pod 重新建立，Pod_IP 變了，資料也沒啦 !
- **在 K8S 的系統中，只要 node 主機重開，它上面 run 的 pod 都會浴火重生(重建)，pod 沒有 volume 的話，資料通通再見 !**


## Access Kubernetes Pods from outside

編輯 yaml 檔

```bash!
$ nano ~/wulin/yaml/podhostport.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: podhostport
spec:
  containers:
  - name: wk01
    image: quay.io/cloudwalker/alpine.sshd
    imagePullPolicy: IfNotPresent
    ports:
     - containerPort: 22
       hostPort: 22101
    securityContext:
      capabilities:
        add: ["AUDIT_WRITE", "SYS_CHROOT", "CAP_NET_RAW"]
    command: ["/usr/sbin/sshd"]
    args: ["-D"]
  nodeSelector:
    kubernetes.io/hostname: w1
```

- `image:` ，Container 的 image 可以換裝成 `localhost/alp.sshd`
- `hostport: 22101`，`kube-proxy` 會在 pod 所在主機開一個 22101 的 port，對應到 pod 裡 Container 開的 22 port，效能優於 `kubectl port-forward` 
- :::spoiler hostport 示意圖
  ![](https://i.imgur.com/tixjWdt.png)
  :::
- `nodeselector`，透過 node 上的 labels 來決定 pod 要建在哪台 node 上

## 讓 mydb pod 也用 hostport 開

先建立工作目錄

```bash!
$ mkdir -p ~/wulin/mydb/
```

編輯 yaml 檔

```bash!
$ nano wulin/mydb/pod-mydb.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mydb
  name: mydb
spec:
  containers:
  - image: localhost/mydb
    name: mydb
    ports:
    - containerPort: 3306
      hostPort: 3306
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

產生 pod

```bash!
$ ka -f wulin/mydb/pod-mydb.yaml
pod/mydb created
```

## 檢視 Linux capabilities

- **Linux capabilities ，規範 Container 可以做什麼事，而規範 Container 等同於規範 pod** 

```bash!
$ crio-status config

output : 
...
default_capabilities = ["CHOWN", "DAC_OVERRIDE", "FSETID", "FOWNER", "SETGID", "SETUID", "SETPCAP", "NET_BIND_SERVICE", "AUDIT_WRITE", "SYS_CHROOT", "KILL"]
...
```

- `NET_BIND_SERVICE` ，如果 Container 要開的 port number 小於 1024 ，會需要有這個 Capabilities
- `SSH` 通訊協定一定會需要 "`AUDIT_WRITE`", "`SYS_CHROOT`" 這兩個 Capabilities，才能讓 CRI-O 做出來的 Container run openssh 的時候，能夠讓 openssh Server 與外部連接，給別人從外面連進來，如果沒有以上兩個 Capabilities ，即使要用 root 去登入也沒用。設定後要記得重新開機。


### CRI-O 規範 Linux capabilities 的檔案

```
$ sudo nano /etc/crio/crio.conf

output : 

[crio.runtime]
.......
default_capabilities = [
  "CHOWN",
  "DAC_OVERRIDE",
  "FSETID",
  "FOWNER",
  "SETGID",
  "SETUID",
  "SETPCAP",
  "NET_BIND_SERVICE",
  "AUDIT_WRITE",
  "SYS_CHROOT",
  "KILL"
]
.......
```

++讓 demo 這個 pod 安裝 libcap 套件++

```bash
$ kubectl exec demo -- sudo apk add libcap
```

- `--`，兩個 `-` 後面接的命令就是要給 pod 執行

> 如果這行安裝失敗，可以進入 pod 的 Container 將 nameserver 的設定檔 ( `/etc/resolv.conf` )，新增一行 `nameserver 8.8.8.8`

++查看 demo 這個 pod 裡面的 Container 裡面有哪些 Linux capabilities++

```bash!
$ kubectl exec demo -- sudo capsh --print

output :

Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_audit_write=ep
...
```

## 連接與移除 Demo App 

```bash!
# K8S 叢集內連接 demo app，k8s 叢集內部的網路連線全程都會加密
# -it ，給一個虛擬終端機
# -- bash ，跑 bash 貝殼程式
$ kubectl exec -it pods/demo -- bash
bash-5.1# exit

# 檢視 demo 這個 pod 的 IP 位址
$ kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
demo   1/1     Running   0          26m   10.244.2.2   w2     <none>           <none>

# ssh 連進 demo 這個 pod
bigred@m1:~$ ssh bigred@10.244.2.2
Warning: Permanently added '10.244.2.2' (RSA) to the list of known hosts.
bigred@10.244.2.2 s password:
Welcome to ALP sshd 6000

# 離開 pod
demo:~$ exit

# K8S 叢集外連接 demo app
# --address 只能指定 Master 的 IP 位址
$ kubectl port-forward --address $IP pods/demo 22100:22 &
[1] 27780

# 在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh bigred@<alp.m1 IP> -p 22100
Handling connection for 22100
bigred@192.168.61.4 s password: bigred
Welcome to ALP sshd 6000

# 離開 pod
demo:~$ exit

# 移除 pod
$ kubectl delete pod  demo
```

以上動作運作架構圖

![](https://i.imgur.com/7fFcPeY.png)


# 第一次接觸 YAML File

## 建立 OpenSSH Pod

```bash!
# 建立專案目錄，編輯 yaml 檔
$ cd ~/wulin; nano sshd/pod-sshd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sshd
spec:
  containers:
  - name: mysshd
    image: localhost/alp.sshd
    imagePullPolicy: IfNotPresent
```

- `apiVersion: v1` ，這行是跟 API Server 說，我們的 yaml 檔是 v1 ，這行的設定由 K8s 的版本來決定
- `kind: Pod` ， 設定 K8s 的物件
- ```!
  metadata:
  name: pod-sshd ---> 設定 pod 的名字
  spec:
  containers:
  - name: mysshd ---> 設定 Container 的名稱
    image: localhost/alp.sshd ---> 設定 Container 用哪片 image 
    imagePullPolicy: IfNotPresent ---> 如果本地端有這片 image，不用上網下載
  ```  
- `imagePullPolicy: Never` ，不要上網下載 image 

```bash!
# apply yaml 檔，建立 pod
$ kubectl apply -f sshd/pod-sshd.yaml

# 檢查是否符合預期
$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE    IP           NODE   NOMINATED NODE   READINESS GATES
demo       1/1     Running   0          126m   10.244.2.2   w2     <none>           <none>
pod-sshd   1/1     Running   0          13s    10.244.1.3   w1     <none>           <none>
```

- `10.244.1.3`，看到這個 IP 位址 ，就知道 Flannel 網路套件在做事，將我們的 pod 分配到第一台工作機


## 再次修改 yaml 檔

```bash=
$ nano sshd/pod-sshd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sshd
spec:
  containers:
  - name: mysshd
    image: localhost/alp.sshd
    imagePullPolicy: IfNotPresent
    command:
      - /bin/bash
      - -c
      - |
        adduser -s /bin/bash -h /home/rbean rbean
        echo -e 'rbean\nrbean\n' | passwd rbean
        /usr/sbin/sshd -D
```

- `command:`，在 K8s 的 yaml 檔中宣告一段等下 Container 要跑的 shell script ，即使 Container 的 image 本身有 `ENTRYPOINT ["/usr/sbin/sshd"]` ，一樣會被破格，強制要跑命令


---


# 建立 File Browser Server


```bash!
$ cd ~/wulin/

$ nano fbs/Dockerfile
FROM quay.io/cloudwalker/alpine
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash && \
    # 設定 filebrowser
    filebrowser config init --port 4000 --address "" --log "/tmp/fbs" --root="/srv" && \
    # 建立管理者帳號
    filebrowser users add admin "admin" --perm.admin=true

CMD ["filebrowser"]
```

- `curl -fsSL 'URL/get.sh' | bash`，利用 curl 命令將網站上的程式輸出在螢幕，但是 `| bash`，會將輸出在螢幕的程式碼丟給重裝貝殼執行，這樣做的好處是不會留下紀錄。
    - `-f`，下載失敗不會輸出錯誤訊息
    - `-s`，Silent mode
- `filebrowser config init`，將網站初始化
    - `--port 4000` ，網站的 port number : 4000
    - `--address ""` ，任何 IP 位址的來源都能與我連線
    - `--log "/tmp/fbs"`，網站 log 檔的存放檔案
    - ` --root="/srv"` ，使用者丟檔案到網站後存放的目錄區，可自行設定
- `filebrowser users add`，建立帳號
    - `admin "admin"`，帳號 : `admin`，密碼 : `admin`
    - `--perm.admin=true`，打開管理員的權限

build image

```bash!
$ sudo podman build -t alp.myfbs fbs/

# 檢查是否符合預期
$ sudo podman images | grep fbs
localhost/alp.myfbs       latest      24d1b68d43f8  About a minute ago  74.3 MB
```

手動部署 alp.myfbs Image

```bash!
# 備份 Image 
$ sudo podman save localhost/alp.myfbs > alpine.myfbs.tar

$ scp alpine.myfbs.tar w1:~; scp alpine.myfbs.tar w2:~

$ ssh w1 'sudo podman load < alpine.myfbs.tar'; ssh w2 'sudo podman load < alpine.myfbs.tar'
```

建立 File Browser Server Pod

```bash=
# 編輯 yaml 檔
$ nano fbs/pod-fbs.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-fbs
  labels:
    name: pod-fbs
spec:
  containers:
  - name: myfbs
    image: localhost/alp.myfbs
    imagePullPolicy: IfNotPresent
    ports:
     - containerPort: 4000
```

用 yaml 檔 建立 pod 

```bash
$ kubectl apply -f fbs/pod-fbs.yaml
```

檢查 pod 的 4000 port 有無開啟

```bash!
# 看 pod 的 IP
$ kg pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
pod-fbs    1/1     Running   0          2m30s   10.244.1.6   w1     <none>           <none>

# 網貓測 4000 port
$ nc -w 1 10.244.1.6 4000

$ echo $?
0
```

### 建立 File Browser Server Service


```bash=
$ nano fbs/svc-fbs.yaml
kind: Service
apiVersion: v1
metadata:
  name: svc-fbs
spec:
  externalIPs:
  - 192.168.61.4
  selector:
    name: pod-fbs
  ports:
  - port: 8080
    targetPort: 4000
```

- `selector:` ，選擇有貼 pod-fbs 這個標籤的 pod
- `externalIPs:` ，設定對外服務的 IP 位址
- ```
  ports:
  - port: 8080   ---> 對外的 port number
    targetPort: 4000   ---> pod 開的 port number
  ```

建立 service/svc-fbs 服務

```
$ kubectl apply -f  fbs/svc-fbs.yaml
```

檢查是否符合預期

```bash=
$ kubectl get all -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod/pod-fbs    1/1     Running   0          14m   10.244.1.6   w1     <none>           <none>
pod/pod-sshd   1/1     Running   0          39m   10.244.2.4   w2     <none>           <none>

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE    SELECTOR
service/kubernetes   ClusterIP   10.98.0.1     <none>         443/TCP    4h5m   <none>
service/svc-fbs      ClusterIP   10.98.0.230   192.168.61.4   8080/TCP   83s    name=pod-fbs
```

看 pod 的標籤

```bash
$ kubectl get pods --show-labels
AME       READY   STATUS    RESTARTS   AGE   LABELS
pod-fbs    1/1     Running   0          20m   name=pod-fbs
pod-sshd   1/1     Running   0          44m   <none>
```

### FBS 架構示意圖

![](https://i.imgur.com/5G9pk3U.png)

- 多建立 Service 這個物件的原因是因為，K8S 的 pod 很容易浴火重生，一旦重建， pod IP 會發生變動，如果這時候我們 pod 是一個網站，這個網站的 IP 位址時不時更換，會搞死人。
- 所以這時候需要透過 Service ，服務的 IP 在 K8S 中是固定的，不會因為主機重開，Service IP 就變動，所以只要讓 Service 透過 labels 綁住我們的 pod ，這樣即使 pod 浴火重生，也沒關係，我們一樣可以靠 Service 連的到我們的 pod 



### 提問 : 如何得知檔案丟到網站後，存放在哪?
答 : pod-fbs 的 container 裡 `/srv` 目錄下

```bash=
# 進入 pod 
$ kubectl exec -it pod/pod-fbs -- bash

# 檢查是否符合預期
bash-5.1# ls -al srv
total 12
drwxr-xr-x 1 root root 4096 Jul 31 06:46 .
dr-xr-xr-x 1 root root 4096 Jul 31 06:23 ..
drwxr-xr-x 3 root root 4096 Jul 31 06:46 ALP3

bash-5.1# touch srv/zzz.txt
bash-5.1# ls -al srv/
total 12
drwxr-xr-x 1 root root 4096 Jul 31 06:51 .
dr-xr-xr-x 1 root root 4096 Jul 31 06:23 ..
drwxr-xr-x 3 root root 4096 Jul 31 06:46 ALP3
-rw-r--r-- 1 root root    0 Jul 31 06:51 zzz.txt
```

![](https://i.imgur.com/YpO3DbL.png)


### 災難篇

如果刪除 pod ，裡面存放的內容就不見了 ! ! !

```
$ kubectl delete pod pod-fbs
pod "pod-fbs" deleted

$ kubectl delete svc svc-fbs
service "svc-fbs" deleted
```

對於 K8s 的 Storage 怎麼處理 ?



## 對 pod 的 yaml 檔新增 hostPath PV

編輯 yaml 檔

```bash=
$ nano fbs/pod-fbs.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: pod-fbs
  labels:
    name: pod-fbs
spec:
  containers:
  - name: myfbs
    image: localhost/alp.myfbs
    imagePullPolicy: IfNotPresent
    ports:
     - containerPort: 4000
    volumeMounts:
    - mountPath: /srv
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /tmp
```

- `volumeMounts`，設定 Container 的目錄要將哪個目錄掛載到 `volumes` 的目錄
    - `mountPath`，要掛載的目錄
- `volumes`，下面可以宣告儲存體的設定
    - `hostPath`，volume 的型態
        - `path: /tmp`，這是一個在 Linux 系統的 `/tmp` 目錄，但注意 ! 這個目錄重開機裡面的內容會消失 (這是 Linux `/tmp` 目錄的規則)。
- [K8s 官網 hostPath 範例連結](https://kubernetes.io/docs/concepts/storage/volumes/)
- 如果刪除 pod 再重建一次， pod 可能會跑到別台 node 主機，如何解決 ?
答 : 針對第一次建立 pod 時，如果 pod 在 node 1 主機上，則每次重建 pod 時，就要安排在這台 node 主機上

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: pod-fbs
  labels:
    name: pod-fbs
spec:
  containers:
  - name: myfbs
    image: localhost/alp.myfbs
    imagePullPolicy: IfNotPresent
    ports:
     - containerPort: 4000
    volumeMounts:
    - mountPath: /srv
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /tmp
  nodeSelector:
    kubernetes.io/hostname: w2
```



###### tags: `系統工程`