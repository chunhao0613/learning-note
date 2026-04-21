# Kubernetes Resource Storage by 明城老師

[TOC]

---


## 學習目標

- EmptyDir （ k8s 原生 ）
- HostPath  （ k8s 原生 ）
- Local Persistent Volumes （ k8s 原生 ）
- Local Path Provisioner
	- 需額外安裝


## Kubernetes Volume

![](https://i.imgur.com/q65JTtm.png)


- emptydir 讓兩個 Container 共用
- HostPath, Local Persistent Volumes 可以讓多個 pod 一起使用
- 當 pod 沒有宣告儲存的地方， pod 掛了 = 資料遺失
- Storage 在叢集裡面或外面


---

## EmptyDir 應用


```bash=
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: time-emptydir
spec:
  containers:
  - image: quay.io/flysangel/busybox
    name: c1
    command: ["/bin/sh", "-c", "while true; do echo $(date) > /cache/time; sleep 1; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  - image: quay.io/flysangel/busybox
    name: c2
    command: ["/bin/sh", "-c", "while true; do cat /cache/time; sleep 5; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
  - name: cache
    emptyDir: {}' > time-pod-emptydir.yaml
```

```bash=

## 建立並啟動 pod
$ kubectl apply -f time-pod-emptydir.yaml
pod/time-emptydir created

## 檢查 pod 的運作狀態
$ kubectl get pod time-emptydir -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
time-emptydir   2/2     Running   0          12m   10.244.1.9   w1     <none>           <none>

# 看 Container 的運作狀況
$ kubectl logs time-emptydir -c c2
...
Sun Nov 7 01:55:49 UTC 2021
Sun Nov 7 01:55:54 UTC 2021

```

### 檢視 Emptydir

```
$ ssh w1 sudo tree /var/lib/kubelet/pods/ | grep -B 10 cache
│               └── token -> ..data/token
├── 4795b226-d3c9-40f4-9941-d6dfe1ada41c
│   ├── containers
│   │   ├── c1
│   │   │   └── c16cc904
│   │   └── c2
│   │       └── 4fe3441b
│   ├── etc-hosts
│   ├── plugins
│   │   └── kubernetes.io~empty-dir
│   │       ├── cache
│   │       │   └── ready
│   │       └── wrapped_kube-api-access-jfsdk
│   │           └── ready
│   └── volumes
│       ├── kubernetes.io~empty-dir
│       │   └── cache
```


```bash=
## 把 pod 宰了
$ kubectl delete pod time-emptydir
pod "time-emptydir" deleted

## 再檢視一次 Emptydir
$ ssh w1 sudo tree /var/lib/kubelet/pods/ | grep -B 10 cache

```

### 請再次產生 emptyDir 應用，判斷 emptyDir 的生命週期是否與 Pod 一致。

```
$ kubectl apply -f time-pod-emptydir.yaml
pod/time-emptydir created

$ ssh w2 sudo tree /var/lib/kubelet/pods/ | grep -B 10 cache
│               └── token -> ..data/token
└── dfbc42a8-6576-4f4b-b9fa-0e2553e98d04
    ├── containers
    │   ├── c1
    │   │   └── a2548b80
    │   └── c2
    │       └── c22897a2
    ├── etc-hosts
    ├── plugins
    │   └── kubernetes.io~empty-dir
    │       ├── cache
    │       │   └── ready
    │       └── wrapped_kube-api-access-qtk78
    │           └── ready
    └── volumes
        ├── kubernetes.io~empty-dir
        │   └── cache
		
$ kubectl delete pod time-emptydir
pod "time-emptydir" deleted

$ ssh w2 sudo tree /var/lib/kubelet/pods/ | grep -B 10 cache
```

<font color=red>**答：一致，但是 Container 的 Overlay2 生命週期可能更短，單獨把 Container 宰了 Overlay2 就沒了。**</font>


---


## HostPath 應用

```bash=
## 宣告 pod yaml 檔
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: time-hostpath
spec:
  containers:
  - image: quay.io/flysangel/busybox
    name: c1
    command: ["/bin/sh", "-c", "while true; do echo $(date) > /cache/time; sleep 1; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
    - name: cache
      hostPath:
        path: /opt/pv/cache' > time-pod-hostpath.yaml

## 建立並啟動 pod
$ kubectl apply -f time-pod-hostpath.yaml
pod/time-hostpath created

## 檢視 pod 運作狀況
bigred@m1:~/0608$ kubectl get pod time-hostpath -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
time-hostpath   1/1     Running   0          9s    10.244.1.10   w1     <none>           <none>

## 查看 pod 掛載的目錄區
bigred@m1:~/0608$ ssh w1 sudo tree /opt/pv/cache
/opt/pv/cache
└── time

0 directories, 1 file

## 透過 cat 檢視 pod 掛載目錄區裡的檔案內容
bigred@m1:~/0608$ ssh w1 sudo cat /opt/pv/cache/time
Wed Jun 8 01:56:39 UTC 2022

bigred@m1:~/0608$ ssh w1 sudo cat /opt/pv/cache/time
Wed Jun 8 01:56:48 UTC 2022

```

### 刪除 pod 檢視 Hostpath 是否存在

```bash=
$ kubectl delete -f time-pod-hostpath.yaml
pod "time-hostpath" deleted

$ ssh w1 sudo tree /opt/pv/cache
/opt/pv/cache
└── time

0 directories, 1 file

$ ssh w1 sudo cat /opt/pv/cache/time
Wed Jun 8 02:03:39 UTC 2022
```

答： 當 pod 被刪除時， Hostpath 還會存在。


### 反覆建立 emptydir、hostPath 應用，判斷 hostPath Pod 是否固定於同一個節點上。

當 k8s 在建立 pod 時， Scheduler 會判斷 node 的悠閒度，來建立 pod。

:::success


在 yml 檔內宣告 pod 要在 哪台 node 上產生
```bash=
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: time-hostpath
spec:
  containers:
  - image: quay.io/flysangel/busybox
    name: c1
    command: ["/bin/sh", "-c", "while true; do echo $(date) > /cache/time; sleep 1; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
    - name: cache
      hostPath:
        path: /opt/pv/cache
  nodeSelector:
    kubernetes.io/hostname: w1' > time-pod-hostpath.yaml
```

下面這兩行可以指定 pod 要在哪一個 node 產生

```
  nodeSelector:
    kubernetes.io/hostname: w1
```
:::

### 練習

題目：建立兩個 hostPath 應用，一個 pod 名稱為 hostPath-1 固定在 w1 的 node 上，另一個 pod 名稱為 hostPath-2 固定在 w2 的 node 上。

答：

```bash=
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: hostpath1
spec:
  containers:
  - image: quay.io/flysangel/busybox
    name: c1
    command: ["/bin/sh", "-c", "while true; do echo $(date) > /cache/time; sleep 1; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
    - name: cache
      hostPath:
        path: /opt/pv/cache
  nodeSelector:
    kubernetes.io/hostname: w1' > pod-hostpath-1.yaml
	
$ echo 'apiVersion: v1
kind: Pod
metadata:
  name: hostpath2
spec:
  containers:
  - image: quay.io/flysangel/busybox
    name: c1
    command: ["/bin/sh", "-c", "while true; do echo $(date) > /cache/time; sleep 1; done"]
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
    - name: cache
      hostPath:
        path: /opt/pv/cache
  nodeSelector:
    kubernetes.io/hostname: w2' > pod-hostpath-2.yaml

$ kubectl apply -f pod-hostpath-1.yaml

$ kubectl apply -f pod-hostpath-2.yaml

$ kg pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
hostpath1   1/1     Running   0          71s   10.244.1.15   w1     <none>           <none>
hostpath2   1/1     Running   0          32s   10.244.2.22   w2     <none>           <none>
```

---

## 認識 Persistent Volumes

- PersistentVolume (PV) 
	- a piece of storage in the cluster 
	- <font color=red>**provisioned by an administrator or dynamically provisioned using Storage Classes.**</font> 
	- It is a resource in the cluster just like a node is a cluster resource. 

- PersistentVolumeClaim (PVC) 
	- request for storage by a user. 
	- Pods consume node resources and PVCs consume PV resources.
	- Pods can request specific levels of resources (CPU and Memory).
	- Claims can request specific size and access modes (ReadWriteOnce, ReadOnlyMany or ReadWriteMany).

PV 與 PVC 的 Access modes
- ReadWriteOnce(RWO)
	- the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node.
	- <font color=red>在同一個主機，可以提供多個 pod 一起使用同一個 pvc</font>
- ReadOnlyMany(ROX)
	- the volume can be mounted as read-only by many nodes.
	- 在多台主機掛載，但只能讀取
- ReadWriteMany(RWX)
	- the volume can be mounted as read-write by many nodes.


---

- PV Persistent Volumes & PVC Persistent Volume Claim
    - PVC｜Persistent Volume Claim（PV 的取用宣告）
    - PV 與 PVC 是一對一關係，只要 PV 與 PVC 建立關係後（Access mode 也必須要符合），PV 就沒辦法被其他 PVC 取用了
    - 取用 PV 只能向上擴充，不能低於 PVC 的需求（要五毛可以給一塊，但不能只給一毛，但多給就是浪費資源）
    - Persistent Volumes Access mode
        - ReadWriteOnce（RWO）
            - 提供同一個節點的多個 pod 存取
        - ReadOnlyMany（ROX）
            - 提供跨節點的多個 pod 存取，唯讀
        - ReadWriteMany（RWX）
            - 提供跨節點的多個 pod 存取，可讀可寫

![](https://i.imgur.com/38qkXr1.png)


![](https://i.imgur.com/ONd2sk6.png)

當 PVC 這邊


## **認識 Local Persistent Volumes**


- What is a Local Persistent Volume?
	- A local persistent volume represents a local disk directly-attached to a single Kubernetes Node.

- How is it different from a HostPath Volume?
	- To better understand the benefits of a Local Persistent Volume, it is useful to compare it to a HostPath volume. HostPath volumes mount a file or directory from the host node’s filesystem into a Pod. Similarly a Local Persistent Volume mounts a local disk or partition into a Pod.

- How is it different from a HostPath Volume?
	- The biggest difference is that the Kubernetes scheduler understands which node a Local Persistent Volume belongs to. With HostPath volumes, a pod referencing a HostPath volume may be moved by the scheduler to a different node resulting in data loss. But with Local Persistent Volumes, the Kubernetes scheduler ensures that a pod using a Local Persistent Volume is always scheduled to the same node.



    - Local Persistent Volumes
        - 基礎與 hostpath 相同，但是可以單獨掛載硬碟來使用
        - 當 PCV 與 LPV 綁定後，pod 就會被限制在 LPV 的所在節點上



---


### 建立 Local Persistent Volume

```bash=
$ echo 'kind: PersistentVolume
apiVersion: v1
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
```

### 建立 Local PersistentVolumeClaim


```bash=
echo 'apiVersion: v1
kind: Pod
metadata:
  name: nginx-1
spec:
  containers:
    - name: nginx-1
      image: quay.io/flysangel/nginx
      volumeMounts:
        - name: html-storage
          mountPath: /usr/share/nginx/html
  volumes:
    - name: html-storage
      persistentVolumeClaim:
        claimName: pvc-rwo-3m' > nginx-pod-1.yaml

$ kubectl apply -f nginx-pod-1.yaml
pod/nginx-1 created

$ kubectl get pod nginx-1 -o wide 
NAME      READY   STATUS              RESTARTS   AGE   IP       NODE   NOMINATED NODE   READINESS GATES
nginx-1   0/1     ContainerCreating   0          17s   <none>   w1     <none>           <none>

$ kubectl describe pod nginx-1
...
Warning  FailedMount  12s (x6 over 28s)  kubelet
MountVolume.NewMounter initialization failed for volume "pv-rwo-5m" : 
path "/opt/pv/5m" does not exist

$ ssh w1 sudo mkdir –p /opt/pv/5m

$ kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP             NODE   NOMINATED NODE   READINESS GATES
nginx-1   1/1     Running   0          11m   10.199.3.207   w1     <none>           <none>
```

<font color=red>**注意！HostPath 如果給的目錄不存在，系統會自動幫你建立，但是 PV 本身在建立的時候，要自己建好目錄區，否則就算 PVC 與 PV bound 在一起，一樣會無法建立 pod**</font>


### 建立 nginx 首頁

```bash=
$ kubectl exec -it nginx-1 -- bash
root@nginx-1:/# echo 'Hello My Pvc' > /usr/share/nginx/html/index.html

root@nginx-1:/# exit
exit

bigred@m1:~/0608$ ssh w1 cat /opt/pv/5m/index.html
Hello My Pvc

bigred@m1:~/0608$ curl 10.244.1.16
Hello My Pvc
```

### 移除 Pod 與 PVC 重新建立

```bash=
$ kubectl delete -f nginx-pod-1.yaml
pod "nginx-1" deleted

$ kubectl delete -f pvc-rwo-3m.yaml
persistentvolumeclaim "pvc-rwo-3m" deleted

$ kubectl get pv pv-rwo-5m
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                STORAGECLASS   REASON   AGE
pv-rwo-5m   5Mi        RWO            Retain           Released   default/pvc-rwo-3m                           18m

$ kubectl apply -f pvc-rwo-3m.yaml
persistentvolumeclaim/pvc-rwo-3m created

$ kubectl get pvc pvc-rwo-3m
NAME         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-rwo-3m   Pending         
2s

$ export KUBE_EDITOR="nano"

$ kubectl edit pv pv-rwo-5m
## 刪除以下這段
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: pvc-rwo-3m
    namespace: default
    resourceVersion: "602201"
    uid: 79c392da-3cc3-4088-876e-e83799c22441
```

當 pvc 與 pv Bound 過一次後， pv 會一直記著當前的 pvc ，當我們把 pvc 刪掉後，去檢查 pv 的狀態會顯示 Released ，於是我們建了一個一模一樣的 pvc 後發現他的狀態是 Pending ，發現 pv 其實還是記著被刪掉的 pvc ，所以如果要讓 pv 忘記舊的 pvc 要透過 `kubectl edit pv $pv_name`，進去把紀錄的 pvc 資訊都刪除。

注意！ pv 刪除後，pv 對應的目錄也要刪除，資料才會清空。



---

# 練習

:::success

題目：
建立兩個 nginx Pod 掛載相同的 PVC，Access Modes 為 ReadWriteOnce，觀察兩個 Pod 是否在同一台節點上。

答案：

```bash=
## 編輯 nginx-pod-1.yaml檔
$ nano nginx-pod-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-1
spec:
  containers:
    - name: nginx-1
      image: quay.io/flysangel/nginx
      volumeMounts:
        - name: html-storage
          mountPath: /usr/share/nginx/html
  volumes:
    - name: html-storage
      persistentVolumeClaim:
        claimName: pvc-rwo-3m

## 編輯 nginx-pod-2.yaml檔
$ nano nginx-pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
spec:
  containers:
    - name: nginx-2
      image: quay.io/flysangel/nginx
      volumeMounts:
        - name: html-storage
          mountPath: /usr/share/nginx/html
  volumes:
    - name: html-storage
      persistentVolumeClaim:
        claimName: pvc-rwo-3m

## 編輯 pv 的 yml 檔
$ nano pv-rwo-5m.yaml
kind: PersistentVolume
apiVersion: v1
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
          - w1

## 編輯 pvc 的 yml 檔
$ nano pvc-rwo-3m.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwo-3m
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Mi

## 建立 pv 
$ kubectl apply -f pv-rwo-5m.yaml

## 檢查 pv 狀態
$ kg pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
pv-rwo-5m   5Mi        RWO            Retain           Bound    default/pvc-rwo-3m                           17s

## 建立 pvc
$ kubectl apply -f pvc-rwo-3m.yaml

## 檢查 pvc 狀態
$ kg pvc
NAME         STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-rwo-3m   Bound    pv-rwo-5m   5Mi        RWO                           13s

## 建立 nginx-pod-1
$ kubectl apply -f nginx-pod-1.yaml
pod/nginx-1 created

## 建立 nginx-pod-2
$ kubectl apply -f nginx-pod-2.yaml
pod/nginx-2 created

## 檢視結果是否符合預期
$ kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx-1   1/1     Running   0          2m28s   10.244.1.17   w1     <none>           <none>
nginx-2   1/1     Running   0          2m25s   10.244.1.18   w1     <none>           <none>
```

<font color=red>結論：兩個 pod 會在同一個 node</font>

:::




:::success

題目：
建立兩個 nginx Pod 掛載相同的 PVC，Access Modes 為 ReadWriteMany，將一個 Pod 鎖定在 W1，另一個 Pod 鎖定在 W2，觀察兩個 Pod 的狀態。

答案：
```bash=
## 清理空間
$ kubectl delete -f nginx-pod-1.yaml
$ kubectl delete -f nginx-pod-2.yaml
$ kubectl delete -f pvc-rwo-3m.yaml
$ kubectl delete -f pv-rwo-5m.yaml

## 

677  nano pv-rwo-5m.yaml
  678  nano pvc-rwo-3m.yaml
  679  kubectl apply -f pvc-rwo-3m.yaml
  680  kubectl apply -f pv-rwo-5m.yaml
  681  kg pv
  682  kg pvc
  683  kg pv
  684  nano nginx-pod-1.yaml
  685  kubectl apply -f nginx-pod-1.yaml
  686  kubectl apply -f nginx-pod-2.yaml
$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
nginx-1   1/1     Running   0          9s    10.244.1.20   w1       <none>           <none>
nginx-2   0/1     Pending   0          5s    <none>        <none>   <none>           <none>
```

:::

:::success

題目：
建立一個 nginx pod 掛載 PVC，Access Modes 為 ReadOnlyMany，並產生一個 10MB 的檔案，觀察是否可以正常儲存。

答案：
```bash=
$ nano pvc-rwo-3m.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwo-3m
spec:
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 3Mi

$ nano pv-rwo-5m.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-rwo-5m
spec:
  capacity:
    storage: 5Mi
  accessModes:
    - ReadOnlyMany
  local:
    path: "/opt/pv/5m"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - w1

$ kubectl apply -f pv-rwo-5m.yaml

$ kubectl apply -f pvc-rwo-3m.yaml

$ kg pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
pv-rwo-5m   5Mi        ROX            Retain           Bound    default/pvc-rwo-3m                           22m

$ kg pvc
NAME         STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-rwo-3m   Bound    pv-rwo-5m   5Mi        ROX                           22m

$ kubectl apply -f nginx-pod-1.yaml

$ kubectl get pod nginx-1 -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
nginx-1   1/1     Running   0          20m   10.244.1.21   w1     <none>           <none>

$ kubectl exec -it nginx-1 -- bash

root@nginx-1:/usr/share/nginx/html# dd if=/dev/zero of=test bs=1M count=10
10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00454216 s, 2.3 GB/s

```



:::


依序建立 3 個 ROX PV(3g, 5g, 10g)，再依序建立 3 個 PVC(4g, 8g, 5g)，觀察 PVC 與 PV 之關係。



---


# Local Path Provisioner

Local Path Provisioner provides a way for the Kubernetes users to utilize the local storage in each node. Based on the user configuration, the Local Path Provisioner will create hostPath based persistent volume on the node automatically. It utilizes the features introduced by Kubernetes Local Persistent Volume feature, but make it a simpler solution than the built-in local volume feature in Kubernetes.


- 只要 pod 宣告新增 pvc ，他會自動新增 pv
    - 單純建立 pvc 的話，不會自動新增 pv
- 刪除 pvc ，他也會自動幫我們刪除 pv
    - pv 的目錄也會被刪除，不會殘留
    - 刪除有使用 pvc 的 pod ，pvc 還會在
- StorageClass "local-path": Only support ReadWriteOnce access mode
- capacitiy 也不會限制

## 安裝 Local Path Provisioner

```bash=
$ kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.22/deploy/local-path-storage.yaml
namespace/local-path-storage created
serviceaccount/local-path-provisioner-service-account created
clusterrole.rbac.authorization.k8s.io/local-path-provisioner-role created
clusterrolebinding.rbac.authorization.k8s.io/local-path-provisioner-bind created
deployment.apps/local-path-provisioner created
storageclass.storage.k8s.io/local-path created
configmap/local-path-config created
```

```bash=
$ kubectl get pod -n local-path-storage
NAME                                      READY   STATUS    RESTARTS   AGE
local-path-provisioner-7fdb4745c6-njprx   1/1     Running   0          4m47s
```

## 建立 PersistentVolumeClaim

```bash=
$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: html-storage
spec:
  storageClassName: local-path
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Mi' > nginx-pvc.yaml
	  
$ kubectl apply -f nginx-pvc.yaml
persistentvolumeclaim/html-storage created

$ kubectl get pvc html-storage 
NAME           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
html-storage   Pending                                      local-path     29s

$ kubectl get pv
No resources found

$ kubectl get pod nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m11s   10.244.1.23   w1     <none>           <none>

```


## 刪除 Pod 檢查 PV PVC

```bash=
$ kubectl delete -f nginx-pod.yaml
pod "nginx" deleted

$ kg pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
html-storage   Bound    pvc-4a8cd7cc-bcd9-4aec-8d6f-171877bdc51b   3Mi        RWO            local-path     15m


$ kubectl delete -f nginx-pvc.yaml
persistentvolumeclaim "html-storage" deleted

$ kubectl get pvc
No resources found in default namespace.

$ kubectl get pv
No resources found

$ ssh w1 ls -al /opt/local-path-provisioner
total 8
drwxr-xr-x 2 root root 4096 Jun  8 14:47 .
drwxr-xr-x 5 root root 4096 Jun  8 14:32 ..
```

---


# 練習

:::success

題目：
建立兩個 nginx Pod 掛載相同的 PVC，Access Modes 為 ReadWriteOnce，觀察兩個 Pod 是否在同一台節點上。

答案：
```bash=
$ nano nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-1
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
        claimName: html-storage

$ nano nginx-pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
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
        claimName: html-storage

$ kubectl apply -f nginx-pod.yaml
pod/nginx-1 created

bigred@m1:~/0608$ kubectl apply -f nginx-pod2.yaml
pod/nginx-2 created

bigred@m1:~/0608$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
nginx-1   1/1     Running   0          31s   10.244.1.29   w1     <none>           <none>
nginx-2   1/1     Running   0          28s   10.244.1.28   w1     <none>           <none>
```

:::


建立兩個 nginx Pod 掛載相同的 PVC，Access Modes 為 ReadWriteMany，將一個 Pod 鎖定在 W1，另一個 Pod 鎖定在 W2，觀察兩個 Pod 的狀態。
```bash=
$ kubectl delete -f nginx-pod.yaml

$ kubectl delete -f nginx-pod2.yaml

$ kubectl delete -f nginx-pvc.yaml

$ nano nginx-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: html-storage
spec:
  storageClassName: local-path
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 3Mi

$ 
```




建立一個 nginx pod 掛載 PVC，Access Modes 為 ReadOnlyMany，並產生一個 10MB 的檔案，觀察是否可以正常儲存。









###### tags: `系統工程`




















