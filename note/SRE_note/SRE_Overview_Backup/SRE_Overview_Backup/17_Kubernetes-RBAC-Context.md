---
title: Kubernetes-RBAC-Context
tags: 系統工程
---

# Kubernetes-RBAC-Context

:::warning

:::spoiler 目錄

[TOC]

:::

## 前言


![](https://i.imgur.com/TyAPgJl.png)

- Linux 系統可以讓多人同時協同作業，使用者登入後，工作目錄會在自己的家目錄。
- 在 K8S 的系統中，也能夠實現多使用者同時進入 K8S 的叢集做操作，登入 K8S 時，靠的是所在機器(Linux/Windows 作業系統)的使用者家目錄下`~/.kube/config` 這個放憑證的檔案。
- 這個憑證會決定使用者在 K8S 可以進入哪個 Namespace ，可以"建立"還是"刪除"還是"查詢"物件...等。


## K8S Multi-Tenancy 管理套件

下載老師準備好的環境

```bash!
$ cd; wget http://www.oc99.org/zip/k8suser.zip; unzip k8suser.zip; cd k8suser

$ ls -l
total 40
-rw-r--r-- 1 bigred bigred  328 Apr  8 22:03 clusterbind.yaml
-rw-r--r-- 1 bigred bigred  507 Apr  8 22:03 clusterole.yaml
-rw-r--r-- 1 bigred bigred  269 Nov  1  2021 context.temp
-rw-r--r-- 1 bigred bigred  277 Oct 30  2021 csr.yaml
-rwxr-xr-x 1 bigred bigred   74 Apr  6 16:00 deluser.sh
-rwxr-xr-x 1 bigred bigred  975 Apr  8 21:57 mkcontext.sh
-rwxr-xr-x 1 bigred bigred 1045 Apr  6 16:01 mkubeuser.sh
-rw-r--r-- 1 bigred bigred  986 Apr  7 17:30 role.temp
-rw-r--r-- 1 bigred bigred  214 Apr  7 17:20 role.yaml
-rw-r--r-- 1 bigred bigred  302 Oct 27  2021 rolebind.yaml
```

## 建立 K8S User 及 憑証

建立 bigboss 使用者

```bash!
$ ./mkubeuser.sh bigboss
K8S bigboss created
```

憑證放在 `kuser` 目錄

```bash!
$ ls -l kuser/
total 12
-rw-r--r-- 1 bigred bigred 1115 Sep 12 10:13 bigboss.crt
-rw-r--r-- 1 bigred bigred  911 Sep 12 10:13 bigboss.csr
-rw------- 1 bigred bigred 1675 Sep 12 10:13 bigboss.key
```

- `bigboss.key`，私鑰
- `bigboss.csr`，放的是使用者的資訊，包含私鑰，姓名和服務單位的資料
- `bigboss.crt`，K8S 根據你用 `bigboss.csr` 這個檔案申請認證後，產生的憑證檔
    - `crt` stand for `certificate`

檢查 K8S 的所有使用者

```bash!
$ kubectl config view | grep -A 10 -e '^users'
users:
- name: bigboss
  user:
    client-certificate: /home/bigred/k8suser/kuser/bigboss.crt
    client-key: /home/bigred/k8suser/kuser/bigboss.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

- `REDACTED` ，表示沒有自己特別指定目錄將使用者登入資訊放在另一個檔案
    - 放在 `~/.kube/config` 這個檔案裡面

看 bigred 使用者在 K8S 系統的憑證

```bash!
$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
  ...
    server: https://192.168.61.4:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: bigboss
  user:
    client-certificate: /home/bigred/k8suser/kuser/bigboss.crt
    client-key: /home/bigred/k8suser/kuser/bigboss.key
- name: kubernetes-admin
  user:
    client-certificate-data:
...
```

- `clusters`，設定 K8S 叢集 (結尾是 s ，代表 K8S 的 Cluster 可以有很多個)
    - `cluster.server`，登入 K8S 的 IP 位址 : `https://192.168.61.4:6443`
    - `clusters.name`，K8S 叢集的名字 : `kubernetes`
- `contexts.`，設定 K8S 的入口 (結尾是 s ，代表 K8S 的入口可以有很多個)
    - `context.cluster`，入口在哪一個 K8S 叢集 ( 名字就叫 `kubernetes` )
    - `context.user`，可以進入入口的使用者 : `kubernetes-admin`
    - `name`，入口的名字 : `kubernetes-admin@kubernetes`
- `current-context`，當前的入口 : `kubernetes-admin@kubernetes`
- `kind: Config`，設定使用者登入的資訊
- 入口憑證檔的重點： **當前的入口名字**、**哪位使用者可以進去**、**進到哪一個 K8S Cluster**


## 認識 Role-based Access Control 

1. Subjects: The set of users and processes that want to access the Kubernetes API.
2. Resources: The set of Kubernetes API Objects available in the cluster. Examples include Pods, Deployments, Services, Nodes, and PersistentVolumes, among others.
3. Verbs: The set of operations that can be executed to the resources above. Different verbs are available (examples: get, watch, create, delete, etc.), but ultimately all of them are Create, Read, Update or Delete (CRUD) operations.

- `Subject`，指的是 K8S 的 User (分為 User 和 ServiceAccount)
    - Users: These are global, and meant for humans or processes living outside the cluster.
        - 人和 K8S Cluster **外面**的程序
    - ServiceAccounts: These are namespaced and meant for intra-cluster processes running inside pods.
        - 在 K8S Cluster **裡面**的 pod 裡的程序
- `Resources`，指的是 K8S 的物件
- `Verbs`，指的是可以對 K8S 的物件做什麼動作
    - 像是可以看 pod 的資訊、新增 pod 或是 刪除 pod ...等動作
    - `watch`，是一直監控物件的狀態
    - `get`，是看單一物件的狀態
    - `list`，看單一種物件的狀態

## 產生 Role

```bash!
# 檢視 RBAC 的版本
$ kubectl api-versions | grep rbac
rbac.authorization.k8s.io/v1

# 建立 finance Namespace
$ kubectl create namespace finance 

# 編輯 Role 的 yaml 檔
$ echo $'kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: finance   
  name: finance-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods", "services"]
  verbs: ["get", "watch", "list"] ' > role.yaml

# 建立 Role
$ kubectl create -f role.yaml 
role.rbac.authorization.k8s.io/finance-reader created

# 檢視 Role 資訊
$ kubectl get roles --namespace=finance
NAME             CREATED AT
finance-reader   2020-10-21T13:48:50Z
```

- `Role` ，設定一個 Namespace 裡對什麼物件可以有什麼動作
- `rules.resources`，主要設定 `pods` 和 `services` 的物件
- `rules.verbs`，針對剛剛的物件，設定動作 `get` 、 `watch` 和 `list` 

## 產生 RoleBinding

```bash!
# 建立 RoleBinding
$ echo $'kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-read-access
  namespace: finance 
subjects:
- kind: User
  name: bigboss
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: finance-reader 
  apiGroup: rbac.authorization.k8s.io ' |  kubectl  apply  -f  -

# 檢視 Rolebinding 資訊
$ kubectl get rolebindings --namespace=finance
NAME                  ROLE                  AGE
finance-read-access   Role/finance-reader   8m13s
```

- `RoleBinding`，在單一 Namespace 中，將 Role 跟 K8S 的 User 綁在一起
- `subjects.kind`，設定 K8S User
    - `name: bigboss`，指定 K8S 的 bigboss User
- `roleRef`，設定 Role 或是 ClusterRole
    - `name: finance-reader`，設定 `finance-reader` 這個 Role


## Create Cluster Role


```bash!
# 建立 ClusterRole
$ echo  $'kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: cluster-pods-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"] ' | kubectl create -f  -

# 檢查 ClusterRole 狀態
$ kubectl get clusterroles | grep cluster-pods-reader
NAME                 CREATED AT
cluster-pods-reader  2022-09-12T03:41:26Z
```

- `Cluster Role`，設定在全部的 Namespace 中對什麼物件可以有什麼動作
- `apiGroups`，有設定這個才能設定要限制 resource
    - `kubectl api-resources`，可以看 resources 在哪個 `apiGroups` 下


## Cluster Role Binding

```bash!
# 建立 ClusterRoleBinding
$ echo $'kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-cluster-pods
subjects:
- kind: User
  name: bigboss  
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pods-reader
  apiGroup: rbac.authorization.k8s.io ' | kubectl create -f  - 

$ kubectl get clusterrolebindings | grep read-cluster-pods
NAME               ROLE                             AGE
read-cluster-pods  ClusterRole/cluster-pods-reader  16s

$ kubectl get pods --as=bigboss
No resources found in default namespace
```

- `clusterRolebinding`，設定 K8S User (包含 service account)與 role 綁定而擁有存取權限



# 切換 K8S Namespace

```bash!
# 執行以下命令, 可知目前使用的 K8S Context 的 namespace 沒有設定
# 用的是預設 default 的 Namespace
$ kubectl config get-contexts
CURRENT   NAME                                CLUSTER      AUTHINFO      NAMESPACE
*  kubernetes-admin@kubernetes   kubernetes   kubernetes-admin  

# 只要從這個路口登入，Namespace 就會直接切換到 finance namespace
$ kubectl config set-context $(kubectl config current-context) --namespace=finance
Context "kubernetes-admin@kubernetes" modified.

# 只會顯示 finance namespace 中的 POD
$ kubectl get pods 
NAME         READY   STATUS    RESTARTS   AGE
helloworld   1/1       Running   0          13h

# 因內定為 finance namespace, 所以 bigboss 可以讀取
$ kubectl get pods --as bigboss
NAME         READY   STATUS    RESTARTS   AGE
helloworld   1/1        Running   0         13h
```


## 切換回 K8S Default Namespace

```bash!
# 切換到 default namespace
$ kubectl config set-context $(kubectl config current-context) --namespace=
Context "kubernetes-admin@kubernetes" modified.

$ kubectl get pods
No resources found in default namespace.
```

# 建立 K8S Context

建立 finance-context Context


```bash!
$ kubectl config set-context finance-context \
--cluster=kubernetes --namespace=finance --user=bigboss
```

- `set-context  finance-context`，建立新的 K8S 的入口，後面接的是入口名稱 `finance-context`
- `--cluster`，設定入口所在的 K8S 叢集
- `--namespace`，設定進入入口後的 Namespace
- `--user`，設定哪一個 K8S 的使用者可以進入新建立的入口

檢視新的入口資訊

```bash!
$ kubectl config view | grep -A 10 contexts:
contexts:
- context:
    cluster: kubernetes
    namespace: finance
    user: bigboss
  name: finance-context
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
```

- `Contexts` 下面成功多了剛剛新建立的 `finance-context`

切換至 finance-context

```bash!
$ kubectl config use-context finance-context
```

- `use-context`，設定當前的入口

在 finance-context 入口操作

```bash!
# 檢視 nodes 資訊
$ k get nodes
Error from server (Forbidden): nodes is forbidden: User "bigboss" cannot list resource "nodes" in API group "" at the cluster scope

# 檢視 pods 資訊
$ k get pods
NAME         READY   STATUS    RESTARTS   AGE
helloworld   1/1     Running   0          124m

# 檢視在 kube-system 中 pods 的資訊
$ k get pods -n kube-system
NAME                         READY   STATUS    RESTARTS   AGE
coredns-565d847f94-rf5tn     1/1     Running   2          2d2h
coredns-565d847f94-t5px2     1/1     Running   2          2d2h
etcd-m1                      1/1     Running   2          2d2h
kube-apiserver-m1            1/1     Running   2          2d2h
kube-controller-manager-m1   1/1     Running   2          2d2h
kube-proxy-2pgpr             1/1     Running   2          2d2h
kube-proxy-fh9c2             1/1     Running   2          2d2h
kube-proxy-qblhl             1/1     Running   2          2d2h
kube-scheduler-m1            1/1     Running   2          2d2h
```


# K8S Multi-Tenancy 實作

## 前置動作

```bash!
$ mkdir tmp; cd tmp

$ unzip ~/k8suser.zip

$ cp k8suser/*.yaml ~/k8suser/
```


## 建立 K8S User 及 憑証

```bash!
$ ./mkubeuser.sh rbean
K8S rbean created

$ ls -lt kuser/
total 24
-rw-r--r-- 1 bigred bigred 1111 Sep 12 13:49 rbean.crt
-rw-r--r-- 1 bigred bigred  907 Sep 12 13:49 rbean.csr
-rw------- 1 bigred bigred 1679 Sep 12 13:49 rbean.key
...
```

檢查是否符合預期

```bash!
$ k config view | grep -A12 -e '^users'
users:
- name: bigboss
  user:
    client-certificate: /home/bigred/k8suser/kuser/bigboss.crt
    client-key: /home/bigred/k8suser/kuser/bigboss.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: rbean
  user:
    client-certificate: /home/bigred/k8suser/kuser/rbean.crt
    client-key: /home/bigred/k8suser/kuser/rbean.key
```

建立 rbean 使用者的入口

```bash!
$ ./mkcontext.sh rbean
namespace/rbean created
- context:
    cluster: default
    namespace: rbean
    user: rbean
  name: rbean-context
Warning: resource roles/finance-reader is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
role.rbac.authorization.k8s.io/finance-reader configured
rolebinding.rbac.authorization.k8s.io/rbean-rbind created
clusterrole.rbac.authorization.k8s.io/rbean-clusterole created
clusterrolebinding.rbac.authorization.k8s.io/rbean-clusterbind created
```

檢查

```bash!
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.61.4:6443
  name: kubernetes
contexts:
...
- context:
    cluster: default
    namespace: rbean
    user: rbean
  name: rbean-context
current-context: kubernetes-admin@kubernetes
...
```


檢視 `mkubeuser.sh`

```bash=
#!/bin/bash

export STU=${1}
export DIR_CSR=~/k8suser/kuser

[ "$#" != "1" ] && echo "mkubeuser.sh user" && exit 1
[ ! -d ${DIR_CSR} ] && mkdir -p ${DIR_CSR}

which envsubst &>/dev/null
[ $? = 1 ] && sudo apk update && sudo apk add gettext &>/dev/null

if [ -f ${DIR_CSR}/${STU}.key ]; then
   echo "K8S ${STU} existed"
else
   openssl genrsa -out ${DIR_CSR}/${STU}.key 2048 &>/dev/null
   openssl req -new -key ${DIR_CSR}/${STU}.key -out ${DIR_CSR}/${STU}.csr -subj "/CN=${STU}/O=${STU}"
   export BASE64_CSR=$(cat ${DIR_CSR}/${STU}.csr | base64 | tr -d '\n')
   cat ~/k8suser/csr.yaml | envsubst | kubectl apply -f - &>/dev/null
   kubectl certificate approve ${STU}-csr &>/dev/null
   sleep 5
   kubectl get csr ${STU}-csr -o jsonpath='{.status.certificate}' | base64 -d > ${DIR_CSR}/${STU}.crt
   kubectl config set-credentials ${STU} --client-certificate=${DIR_CSR}/${STU}.crt --client-key=${DIR_CSR}/${STU}.key &>/dev/null
   kubectl config view | grep "name: ${STU}" &>/dev/null
   [ "$?" == "0" ] && echo "K8S ${STU} created"
fi
```

## 檢視 K8S User Context

```bash!
$ cat kuser/rbean.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F
........
    server: https://192.168.61.4:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: rbean
    user: rbean
  name: rbean-context
current-context: rbean-context
........
kind: Config
preferences: {}
users:
- name: rbean
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F..........
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0........
```

- `client-certificate-data` 存放 公鑰 的憑証檔
- `client-key-data` ，存放使用者的 私鑰


## 檢視 Namespace RBAC 設定

```bash!
$ kubectl get role -n rbean
NAME         CREATED AT
rbean-role   2022-04-08T14:50:29Z

$ kubectl get clusterrole -n rbean | grep rbean
rbean-clusterole   2022-04-08T14:50:29Z

$ kubectl describe role rbean-role -n rbean
Name:         rbean-clusterole
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources                         Non-Resource URLs  Resource Names  Verbs
  ---------                         -----------------  --------------  -----
  csidrivers.storage.k8s.io         []                 []              [*]
  csinodes.storage.k8s.io           []                 []              [*]
  storageclasses.storage.k8s.io     []                 []              [*]
  volumeattachments.storage.k8s.io  []                 []              [*]
  nodes                             []                 []              [create delete deletecollection get list patch update watch]
  persistentvolumes                 []                 []              [create delete deletecollection get list patch update watch]
```

## 建立 Linux User 及 設定 K8S 憑証

```bash!
$ echo -e "rbean\nrbean" | sudo adduser rbean

$ sudo mkdir /home/rbean/.kube; sudo cp ~/k8suser/kuser/rbean.conf /home/rbean/.kube/config; sudo chown -R rbean:rbean /home/rbean/.kube

$ dir /home/rbean/.kube
total 16K
drwxr-sr-x 2 rbean rbean 4.0K Apr  7 10:00 .
drwxr-sr-x 3 rbean rbean 4.0K Apr  7 10:00 ..
-rw-r--r-- 1 rbean rbean 5.5K Apr  7 10:00 config

$ exit
```

- `/home/rbean/.kube/config`，這個檔案為路口憑證檔


## 檢視 K8S User 憑証

```bash!
在 Windows 系統的 cmd 視窗, 執行以下命令
$ ssh rbean@<alp.m1 IP>

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.61.4:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: rbean
    user: rbean
  name: rbean-context
current-context: rbean-context
kind: Config
preferences: {}
users:
- name: rbean
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

## 測試 K8S User 憑証

```bash!
$ kg all
No resources found in rbean namespace.
$ kg pv
No resources found
$ kg sc
No resources found

$ kubectl run a1 --rm -it --image=quay.io/cloudwalker/alpine
If you don't see a command prompt, try pressing enter.
/ # ping -c 1 www.hinet.net
PING www.hinet.net (163.28.83.113): 56 data bytes
ping: permission denied (are you root?)
/ # exit

$ exit
```


# bobo 網站應用系統

## 佈署 bobo 網站應用系統


在 bigred 終端機, 執行以下命令

```bash!
$ wget http://www.oc99.org/zip/bobowas.zip; unzip bobowas.zip

# 佈署 alp.mysql, alp.goweb 及 alp.myfbs image
$ cd ~/bobowas/k8simg/

$ ./go-k8s-images.sh
[images build]
alp.myfbs image ok
alp.mysql image ok
alp.goweb image ok

[images deploy]
w1: localhost/alp.myfbs image ok
w2: localhost/alp.myfbs image ok

w1: localhost/alp.mysql image ok
w2: localhost/alp.mysql image ok

w1: localhost/alp.goweb image ok
w2: localhost/alp.goweb image ok
```

在 rbean 終端機, 執行以下命令
```bash!
$ wget http://www.oc99.org/zip/bobowas.zip; unzip bobowas.zip

$ cd /home/bigred/bobowas/k8sobj/

$ ./go-k8s-object.sh
bobo.fbs(w1) pod created
bobo.mysql(w2) pod created
bobo.goweb(w1) pod created
bobo.gosvc service created

$ kg all
NAME                   READY   STATUS    RESTARTS   AGE
pod/bobo.fbs         1/1       Running    0               15s
pod/bobo.goweb    1/1       Running    0               15s
pod/bobo.mysql    1/1        Running    0               15s

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP    PORT(S)   AGE
service/svc-gosrv   ClusterIP  10.98.0.237  192.168.61.4   80/TCP   15s
```


# K8S Namespace Resource Quota

在 bigred 終端機, 執行以下命令


```bash!
$ nano ~/bobowas/k8sobj/nsrq.yaml
```

```yaml=
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: rbean
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 256Mi
    requests.nvidia.com/gpu: 1
    limits.cpu: "2"
    limits.memory: 1Gi
    limits.nvidia.com/gpu: 2
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota
  namespace: rbean
spec:
  hard:
    pods: "3"
    configmaps: "2"
    persistentvolumeclaims: "2"
    replicationcontrollers: "2"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```

產生 resourcequota

```bash!
$ ka -f nsrq.yaml
```

## 檢視 Resource Quota


在 rbean 終端機, 執行以下命令

```bash!
$ kubectl get resourcequotas
NAME            AGE     REQUEST                                                                                                                                            LIMIT
compute-quota   2m36s   requests.cpu: 0/1, requests.memory: 0/256Mi, requests.nvidia.com/gpu: 0/1                                                                          limits.cpu: 0/2, limits.memory: 0/1Gi, limits.nvidia.com/gpu: 0/2
object-quota    2m36s   configmaps: 1/2, persistentvolumeclaims: 0/2, pods: 3/3, replicationcontrollers: 0/2, secrets: 0/10, services: 1/10, services.loadbalancers: 0/2
```

- Resource Quota 建立後，並不會限制已經建立的 pod 。

產生 a1 的 pod 測試有沒有被限制

```bash!
$ kubectl run a1 --rm -it --image=quay.io/cloudwalker/alpine
Error from server (Forbidden): pods "a1" is forbidden: failed quota: compute-quota: must specify limits.cpu for: a1; limits.memory for: a1; requests.cpu for: a1; requests.memory for: a1
```

- 錯誤訊息說明 : 建立 pod 時，要設定使用的資源

重新編輯 yaml 檔

```bash!
$ kubectl run a1 --image=quay.io/cloudwalker/alpine --dry-run=client -o yaml > pod.yaml
```

編輯 yaml 檔

```bash!
$ nano pod.yaml
```

```yaml=
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: a1
  name: a1
spec:
  containers:
  - image: quay.io/cloudwalker/alpine
    name: a1
    resources:
      requests:
         cpu: 1.0
         memory: 256Mi
      limits:
         cpu: 2.0
         memory: 512Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

再重新產生 pod 

```bash!
$ kubectl apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "a1" is forbidden: exceeded quota: object-quota, requested: pods=1, used: pods=3, limited: pods=3
```

- 這時候噴的錯誤就是超過 resourcequota 的限制






