# Kubernetes Deployment


## 前言

本篇主要介紹什麼是 Deployment object，並且透過這個 K8S 的 物件，**將一個網站服務進行進版或退版，且服務不中斷。**


## 目錄

[TOC]

---


# 認識 Deployment

A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.


![](https://i.imgur.com/of3Paqu.png)

- 一個在 etcd 中的控制區塊，裡面紀錄 ReplicaSet Controller 這個功能
- Deployment 由 ReplicaSet 和 Pods 組成
- Deployment 管理 ReplicaSet ， ReplicaSet 管理 Pods
- Deployment 可做到 **Replication of components** + **Auto-scaling** + **Load balancing** + **Rolling updates**
- Rolling updates，是由 Deployment 在做的
- 讓企業 run 的應用系統能夠長長久久穩穩當當


# 認識 ReplicaSet

![](https://i.imgur.com/dHJsNuQ.png)

- ReplicaSet 是用來確保在 k8s 中，在資源允許的前提下，指定的 pod 的數量會跟使用者所期望的一致，也就是所謂的 desired status。
- ReplicaSets 可以生成多個 pod (Scalability，橫向擴增或縮減)，主要目的是服務流量分散
    - 多個 pod 是一樣的 (本尊和分身的概念)
    - 所有 Pod 都共享同一 PersistentVolumeClaim，並與同一 PersistentVolume 綁定
- ReplicaSet 有自我療傷的功能
    - 當管理的 pod 毀損或被刪除時，會自動再產生同樣的 pod 出來
    - 只要啟動 PV ，pod 的資料就不會不見
- 可控管 pod 的狀態，如果 pod 負載太重，會變成離線的狀態，處理完手上的事情再回來


## 撰寫 Depolyment Object 部署檔案

```bash!
echo 'apiVersion: apps/v1
kind: Deployment
metadata:
  name: depobj
  labels:
    app: deppod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deppod
  template:
    metadata:
      labels:
        app: deppod
    spec:
      containers:
      - name: myalpine
        image: quay.io/ict39/alpine
        imagePullPolicy: IfNotPresent
        tty: true '> ~/wulin/yaml/depobj.yaml 
```

- `kind: Deployment`，生成 Deployment 的物件
    - `metadata.name`，Deployment 的名字
    - `labels:`，貼標籤，key 是 app，value 是 deppod
- `spec`，Deployment 的規格
    - `replicas: 2`，讓 ReplicaSet 產生 2 個 pod
    - `selector.matchLabels:`，透過標籤管理 pod
        - `app: deppod`，標籤
    - `template:`，對 pod 的宣告
        - `metadata:`，pod 的重要資訊
            - `labels.app`，對 pod 貼 `app: deppod` 的標籤
        - `spec`，pod 的規格
            - `containers`，Container 的設定
                - `name`，Container 的名字
                - `image`，Container 用的 image
                - `imagePullPolicy: IfNotPresent`，如果 node 主機沒有 image 就上網下載
                - `tty: true`，因 image 內定執行的命令是貝殼程式，所以要給終端機


## 建立與檢視 Depolyment Object

```bash!
$ kubectl apply -f wulin/yaml/depobj.yaml
deployment.apps/depobj created

$ kubectl get all --selector app=deppod
NAME                         READY   STATUS    RESTARTS   AGE
pod/depobj-7cb97745c-4k8jm   1/1     Running   0          2m32s
pod/depobj-7cb97745c-ssbf4   1/1     Running   0          2m32s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.98.0.1    <none>        443/TCP   24h

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/depobj   2/2     2            2           2m32s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/depobj-7cb97745c   2         2         2       2m32s
```

- 先看 Deployment 的名字， `depobj`
- 再看 Replicaset 的名字，`depobj-7cb97745c`
    - 因為是由 Deployment 建立的，所以 Replicaset 會有 deplopyment 的名字
- 再看 pod 的名字，`depobj-7cb97745c-4k8jm`
    - 因為 pod 是由 Deploy 的 ReplicaSet 建立的，所以 pod 名字前面會帶有他們的名字

## 測試 ReplicaSets Controller

刪除帶有 `app=deppod` 標籤的 pod

```bash!
$ kubectl delete pod --selector app=deppod
pod "depobj-7cb97745c-4k8jm" deleted
pod "depobj-7cb97745c-ssbf4" deleted
```

- ReplicaSets Controller 會一直執行 reconciliation loop 程序, 確保 Deployment Object 的 actual state 與 desired state 一致. 上面命令刪除二個 POD, 這時 reconciliation loop 程序會再生成二個新的 POD, 由以下命令得知

```bash!
$ kubectl get all --selector app=deppod
NAME                         READY   STATUS    RESTARTS   AGE
pod/depobj-7cb97745c-jg7fk   1/1     Running   0          2m23s
pod/depobj-7cb97745c-smjlt   1/1     Running   0          2m23s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/depobj   2/2     2            2           22m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/depobj-7cb97745c   2         2         2       22m
```

- pod 的名字後面流水號不同，代表 pod 真的有被重新建立


## 修改 Depolyment Object 部署檔案

```bash!
$ echo 'apiVersion: apps/v1
kind: Deployment
metadata:
  name: depobj
  labels:
    app: deppod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deppod
  template:
    metadata:
      labels:
        app: deppod
    spec:
      containers:
      - name: myderby
        image: quay.io/ict39/alpine.derby
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8888 '> ~/wulin/yaml/depobj.yaml 
```

不刪除原本的 Deployment Object，直接再次產生

```bash!
$ kubectl apply -f wulin/yaml/depobj.yaml
deployment.apps/depobj configured
```

檢視所有帶有 `app=deppod` 這個標籤的物件

```bash!
$ kubectl get all --selector app=deppod
NAME                          READY   STATUS    RESTARTS   AGE
pod/depobj-85987557b8-pq6f7   1/1     Running   0          47s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/depobj   1/1     1            1           32m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/depobj-7cb97745c    0         0         0       32m
replicaset.apps/depobj-85987557b8   1         1         1       47s
```

- `replicaset.apps/depobj-7cb97745c`，注意，這是上一個 Deployment 產生的 ReplicaSet
- 當 Deployment 裡面的 pod 換成產生不同的 Container ，原本的 pod 會被刪除，但是 ReplicaSet 不會被刪除


## 建立與檢視 Depolyment Object

```bash!
echo '
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depng
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: quay.io/cloudwalker/nginx:1.20
        ports:
        - containerPort: 80 '> ~/wulin/yaml/depng.yaml
```

```bash!
$ kubectl apply -f ~/wulin/yaml/depng.yaml 
deployment.apps/depng created

$ kg all --selector app=nginx
NAME                        READY   STATUS    RESTARTS   AGE
pod/depng-7bff78c9c-ntc7t   1/1     Running   0          2m
pod/depng-7bff78c9c-skj6f   1/1     Running   0          2m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/depng   2/2     2            2           2m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/depng-7bff78c9c   2         2         2       2m
```


## Updating a Deployment

```bash!
$ kubectl set image deployment.v1.apps/depng nginx=quay.io/cloudwalker/nginx:1.21 --record 
deployment.apps/depng image updated
```

- 將 image 換裝成新版 1.21
- `--record` ，記錄

```bash!
$ kubectl rollout status deployment.v1.apps/depng
Waiting for deployment "depng" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "depng" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "depng" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "depng" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "depng" rollout to finish: 1 old replicas are pending termination...
deployment "depng" successfully rolled out
```

- 這命令要再上一個命令做完後趕快執行，不然只會看到最後一行
- Deplyment 進版時，會先把新的 pod 建立起來，再把舊版本的 pod 刪除


檢查 image 是不是真的換裝城新版的

```bash!
$ kubectl describe deployments depng
Name:                   dep1
............
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        quay.io/cloudwalker/nginx:1.21
    Port:         80/TCP
    Host Port:    0/TCP

```

## Rolling Back a Deployment

將 Deployment 退回上一個版本

```bash!
$ kubectl rollout undo deployment.v1.apps/depng --to-revision=1
```

檢查是否符合預期

```bash!
$ kubectl describe deployment depng
........
  Containers:
   nginx:
    Image:        quay.io/cloudwalker/nginx:1.20
    Port:         80/TCP
    Host Port:    0/TCP
```

## Scaling a Deployment

將 Deployment 的 pod 擴充為 3 個

```bash!
$ kubectl scale deployment.v1.apps/depng --replicas=3
```

檢查是否符合預期

```bash!
$ kubectl get deployment depng
NAME   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
dep1   3         3         3            3           46m

$ kubectl get pods | grep depng
dep1-66f7f56f56-2rf8p   1/1     Running   0          39s
dep1-66f7f56f56-bggcn   1/1     Running   0          71s
dep1-66f7f56f56-mn589   1/1     Running   0          69s

$ kubectl delete deployment depng
```









-----

# 明城老師版本

## 建立 Local Persistent Volume

```bash=
$ echo 'apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rwo-5m
spec:
  capacity:
    storage: 5Mi
  accessModes:
    - ReadWriteOnce
  local:
    path: "/opt/pv/5m"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - w1' > pv-rwo-5m.yaml

$ kubectl apply -f pv-rwo-5m.yaml

$ kubectl get pv pv-rwo-5m
```

## 建立 Local PersistentVolumeClaim

```bash=
$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwo-3m
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Mi' > pvc-rwo-3m.yaml

$ kubectl apply -f pvc-rwo-3m.yaml
```


## 檢視 PV 與 PVC

```bash=
$ kubectl get pvc pvc-rwo-3m
NAME         STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-rwo-3m   Bound    pv-rwo-5m   5Mi        RWO                           10s

bigred@m1:~/0613$ kubectl get pv pv-rwo-5m
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
pv-rwo-5m   5Mi        RWO            Retain           Bound    default/pvc-rwo-3m                           55s
```

## 建立 Deployment

```bash=
$ echo 'apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: quay.io/flysangel/nginx
          volumeMounts:
            - name: html-storage
              mountPath: /usr/share/nginx/html
      volumes:
        - name: html-storage
          persistentVolumeClaim:
            claimName: pvc-rwo-3m' > deploy-nginx.yaml
```

## 建立 Pod 使用 PVC

```bash=
$ ssh w1 sudo mkdir -p /opt/pv/5m
$ ssh w1 "echo 'Hello My Pvc' | sudo tee /opt/pv/5m/index.html"
```


## 驗證 nginx 首頁

```bash=
$ kubectl apply -f deploy-nginx.yaml

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE   NOMINATED NODE   READINESS GATES
nginx-5bfb7cc7b7-2jkbl   1/1     Running   0          2m50s   10.244.1.116   w1     <none>           <none>
nginx-5bfb7cc7b7-9zrbt   1/1     Running   0          2m50s   10.244.1.115   w1     <none>           <none>
nginx-5bfb7cc7b7-g2glm   1/1     Running   0          2m50s   10.244.1.117   w1     <none>           <none>

$ curl http://10.244.1.116
Hello My Pvc

$ curl http://10.244.1.115
Hello My Pvc

$ curl http://10.244.1.117
Hello My Pvc
```

> 透過 Deployment/ReplicaSets 建立的 pods 後面會有一串代碼



---

## 認識 ReplicaSet

![](https://i.imgur.com/3sGeVno.png)

- 對於 ReplicaSet 所管理的 Pod, 雖然每個 Pod 都有 hostname, 但當 Pod 被移轉時, hostname 會重新分配, 對於指向 ReplicaSet 的 Service, 其他服務訪問 Service 時會被隨機分配到某個 Pod

- DNS 伺服器，幫我們把記不住的 IP 轉成記得住的網址
- Service 建立後，會跟 K8s core-dns 註冊


## 建立 Service Deployment

```bash=
$ echo 'apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None
  selector:
    app: nginx' > svc-nginx.yaml
```

上面這個 Service 是 Headless Service （ 無頭服務 ）。

```bash=
$ kubectl apply -f svc-nginx.yaml
service/nginx created

$ kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.98.0.1    <none>        443/TCP   13d
nginx        ClusterIP   None         <none>        <none>    14s
```

## 觀察 Service Deployment

```bash=
$ kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.98.0.10   <none>        53/UDP,53/TCP,9153/TCP   13d
```

透過 nslookup 來將

```bash=
$ nslookup
> server 10.98.0.10
Default server: 10.98.0.10
Address: 10.98.0.10#53

> 10.244.1.115
115.1.244.10.in-addr.arpa       name = 10-244-1-115.nginx.default.svc.k8s.org.

> 10.244.1.116
116.1.244.10.in-addr.arpa       name = 10-244-1-116.nginx.default.svc.k8s.org.

> 10.244.1.117
117.1.244.10.in-addr.arpa       name = 10-244-1-117.nginx.default.svc.k8s.org.

> nginx.default.svc.k8s.org
Server:         10.98.0.10
Address:        10.98.0.10#53

Name:   nginx.default.svc.k8s.org
Address: 10.244.1.116
Name:   nginx.default.svc.k8s.org
Address: 10.244.1.117
Name:   nginx.default.svc.k8s.org
Address: 10.244.1.115
```

## 練習

請問如何擴展 Deployment。
請觀察擴展 Deployment 時，pod 或 container 狀態為何。
請將 Local Persistent Volumes 換成 Local Path Provisioner。

練習完後請刪除練習環境
Pod -> PVC

```bash=
$ kubectl scale deploy/nginx --replicas=6



```

###### tags: `系統工程`



