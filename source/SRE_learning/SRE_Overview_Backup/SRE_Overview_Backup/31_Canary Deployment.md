# Canary Deployment

[TOC]


---


## 前置作業 Part1

請依簡報 904-Jenkins-Part1 完成以下事項
- 第 2~4 頁環境準備
- 第 10~13 頁部署 Jenkins
- 第 20 頁設定 SSH 免密碼

## 前置作業 Part2

- K8S 建立 namespace 命名為 prod
- K8S 建立 namespace 命名為 dev
```bash=
$ kubectl create ns prod
namespace/prod created
bigred@m1:~/wk/mp3$ kubectl create ns dev
namespace/dev created
```


參考簡報 905-Jenkins-Part2 第 17~ 20 頁
- 在 prod 建立 imagePullSecrets 
- 在 dev 建立 imagePullSecrets 

```bash=
$ kubectl create secret generic regcred \
> --from-file=.dockerconfigjson=/home/bigred/auth.json \
> --type=kubernetes.io/dockerconfigjson \
> -n prod

$ kubectl create secret generic regcred \
> --from-file=.dockerconfigjson=/home/bigred/auth.json \
> --type=kubernetes.io/dockerconfigjson \
> -n dev
```


## 下載 canary-demo

```bash=
$ pwd
/home/bigred/wk

$ wget http://web.flymks.com/cicd/v1/canary-demo.zip

$ unzip canary-demo.zip && cd canary-demo

$ tree .
.
├── Dockerfile
├── Jenkinsfile
├── b1-canary.yaml
├── b1-dev.yaml
├── b1-prod.yaml
├── busybox-x86_64
├── info.cgi
└── service.yaml

0 directories, 8 files
```


## 建立 post-commit

```bash=
$ nano .git/hooks/post-commit
#!/bin/bash

jenkinsURL="http://jenkins.k8s.org/git/notifyCommit?url="
repo="ssh://bigred@192.168.61.4/home/bigred/wk/canary-demo"
branchName=$(git rev-parse --abbrev-ref HEAD)

curl -s "${jenkinsURL}${repo}&branches=${branchName}"

$ chmod +x .git/hooks/post-commit
```

## 建立 post-merge

用途一樣是通知 jenkins 

```bash=
$ cp .git/hooks/post-commit .git/hooks/post-merge
$ chmod +x .git/hooks/post-merge
```

## Git & Jenkins 設定

- 新增 Jenkins multibranch pipeline 作業
- 作業命名「canary-demo」
	- 連接到 canary-demo git 儲存庫
	- `ssh://bigred@192.168.61.4/home/bigred/wk/canary-demo`

```bash=
$ git add .
$ git commit -m "Initial commit"
```

檢視執行結果

```bash=
$ kubectl get all -n prod
NAME                           READY   STATUS    RESTARTS   AGE
pod/b1-prod-7d7ff6687c-mdszn   1/1     Running   0          92s

NAME         TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)        AGE
service/b1   LoadBalancer   10.98.0.21   192.168.61.222   80:32302/TCP   92s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/b1-prod   1/1     1            1           92s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/b1-prod-7d7ff6687c   1         1         1       92s
```


檢視 Jenkinsfile

```bash=
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'canary-demo'
            defaultContainer 'p1'
            yaml """
apiVersion: v1
kind: Pod
spec:
  hostAliases:
  - ip: "192.168.61.220"
    hostnames:
    - "quay.k8s.org"
  imagePullSecrets:
    - name: regcred
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    environment {
        QUAY_ADMIN = credentials('quay-admin-id')
        KUBECONFIG = credentials('kubeconfig-id')
        IMAGE_TAG = "quay.k8s.org/quay/alpine.httpd:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    }
    stages {
        stage ('build and push image') {
            steps {
                sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
                sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                sh 'podman images'
                sh 'podman push --tls-verify=false "${IMAGE_TAG}"'
            }
        }
        stage ('deploy canary') {
            when { branch 'canary' }
            steps {
                container('k1') {
                    sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                    sh "sed -i.bak 's#quay.k8s.org/quay/alpine.httpd:1.0.0#${IMAGE_TAG}#' b1-canary.yaml"
                    sh 'kubectl apply -f service.yaml -n prod'
                    sh 'kubectl apply -f b1-canary.yaml -n prod'
                }
            }
        }
        stage ('deploy prod') {
            when { branch 'master' }
            steps {
                container('k1') {
                    sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                    sh "sed -i.bak 's#quay.k8s.org/quay/alpine.httpd:1.0.0#${IMAGE_TAG}#' b1-prod.yaml"
                    sh 'kubectl apply -f service.yaml -n prod'
                    sh 'kubectl apply -f b1-prod.yaml -n prod'
                }
            }
        }
        stage ('deploy dev') {
            when {
                not { branch 'master' }
                not { branch 'canary' }
            }
            steps {
                container('k1') {
                    sh 'kubectl get ns ${BRANCH_NAME} || kubectl create ns ${BRANCH_NAME}'
                    sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                    sh "sed -i.bak 's#quay.k8s.org/quay/alpine.httpd:1.0.0#${IMAGE_TAG}#' b1-dev.yaml"
                    sh "sed -i.bak 's#LoadBalancer#ClusterIP#' service.yaml"
                    sh 'kubectl apply -f service.yaml -n ${BRANCH_NAME}'
                    sh 'kubectl apply -f b1-dev.yaml -n ${BRANCH_NAME}'
                }
            }
        }
    }
```

## Canary Deployment

```bash=
## 部署金絲雀 Deployment
$ kubectl apply -f b1-canary.yaml -n prod

## scale "b1-prod" pod 的數量至 4
$ kubectl -n prod scale deployment b1-prod --replicas=4
deployment.apps/b1-prod scaled

## 檢視部署狀態
$ kubectl get all -n prod
NAME                             READY   STATUS    RESTARTS   AGE
pod/b1-canary-6587596f4b-kd5px   1/1     Running   0          79s
pod/b1-prod-7d7ff6687c-c756t     1/1     Running   0          8s
pod/b1-prod-7d7ff6687c-fgs9l     1/1     Running   0          8s
pod/b1-prod-7d7ff6687c-mdszn     1/1     Running   0          31m
pod/b1-prod-7d7ff6687c-zk95m     1/1     Running   0          8s

NAME         TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)        AGE
service/b1   LoadBalancer   10.98.0.21   192.168.61.222   80:32302/TCP   31m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/b1-canary   1/1     1            1           79s
deployment.apps/b1-prod     4/4     4            4           31m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/b1-canary-6587596f4b   1         1         1       79s
replicaset.apps/b1-prod-7d7ff6687c     4         4         4       31m
```

## 輪詢 /info URL

開啟一個新的 cmd 視窗登入 m1 主機

```
$ while true; do curl 192.168.61.221/cgi-bin/info; sleep 3; done
```

結果：

```bash=
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
<HTML><HEAD><TITLE>404 Not Found</TITLE></HEAD>
<BODY><H1>404 Not Found</H1>
The requested URL was not found
</BODY></HTML>
version: v1.0.0, IP: 10.244.2.114, hostname: b1-prod-7d7ff6687c-c756t
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
version: v1.0.0, IP: 10.244.2.114, hostname: b1-prod-7d7ff6687c-c756t
version: v1.0.0, IP: 10.244.0.39, hostname: b1-prod-7d7ff6687c-zk95m
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
version: v1.0.0, IP: 10.244.1.178, hostname: b1-prod-7d7ff6687c-mdszn
version: v1.0.0, IP: 10.244.2.114, hostname: b1-prod-7d7ff6687c-c756t
<HTML><HEAD><TITLE>404 Not Found</TITLE></HEAD>
<BODY><H1>404 Not Found</H1>
The requested URL was not found
</BODY></HTML>
```




## 建立 canary branch

```
$ git checkout -b canary
Switched to a new branch 'canary'
```


修改 info.cgi

```
$ nano info.cgi
	::
version="v2.0.0"
	::
```

讓 git 追蹤

```bash=
$ git add info.cgi

$ git commit -m 'Version 2'
```


## 確認輪詢狀態

回到輪詢 /info URL 的 cmd 視窗

```bash=
version: v1.0.0, IP: 10.244.1.178, hostname: b1-prod-7d7ff6687c-mdszn
version: v2.0.0, IP: 10.244.1.180, hostname: b1-canary-6f586475bd-w9hrj
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
version: v1.0.0, IP: 10.244.0.39, hostname: b1-prod-7d7ff6687c-zk95m
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
version: v1.0.0, IP: 10.244.1.179, hostname: b1-prod-7d7ff6687c-fgs9l
version: v2.0.0, IP: 10.244.1.180, hostname: b1-canary-6f586475bd-w9hrj
```

## 部署 V2 到正式環境

```bash=
$ git checkout master
$ git log --oneline --all
e8557fa (canary) Version 2
5ee1aa1 (HEAD -> master) Initial commit

$ git merge canary
```
請至 Jenkins 網頁確認執行結果 !

![](https://i.imgur.com/Lr9QgNA.png)


請確認 V2 部署狀態
```
$ watch kubectl get pod -n prod
```

確認輪詢狀態

回到輪詢 /info URL 的 cmd 視窗

```bash=
version: v2.0.0, IP: 10.244.1.180, hostname: b1-canary-6f586475bd-w9hrj
version: v2.0.0, IP: 10.244.1.180, hostname: b1-canary-6f586475bd-w9hrj
version: v2.0.0, IP: 10.244.1.180, hostname: b1-canary-6f586475bd-w9hrj
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
```


---


# 新功能開發流程模擬

1. PM 與客戶討論新功能需求
2. PG 接到新需求，建立開發用 branch
3. PG 開發搭配 Jenkins 部署到測試環境
4. 經過一番測試與修改後，新功能確認沒問題
5. 驗測 App 採用金絲雀部署策略
6. 確認新功能運作沒問題後，部署至正式環境


## 建立開發用 branch

```bash=
$ git checkout -b dev

修改 info.cgi
$ nano info.cgi
	::
version="v3.0.0"

$ git add info.cgi
$ git commit -m 'Test Version 3'
```

### Jenkins 執行結果

![](https://i.imgur.com/wYerEj3.png)

### 確認部署狀態

```bash=
$ kubectl get all -n prod
NAME                             READY   STATUS    RESTARTS   AGE
pod/b1-canary-6f586475bd-w9hrj   1/1     Running   0          23m
pod/b1-prod-75466968fd-x7nnw     1/1     Running   0          13m

NAME         TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)        AGE
service/b1   LoadBalancer   10.98.0.21   192.168.61.222   80:32302/TCP   63m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/b1-canary   1/1     1            1           33m
deployment.apps/b1-prod     1/1     1            1           63m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/b1-canary-6587596f4b   0         0         0       33m
replicaset.apps/b1-canary-6f586475bd   1         1         1       23m
replicaset.apps/b1-prod-75466968fd     1         1         1       13m
replicaset.apps/b1-prod-7d7ff6687c     0         0         0       63m

bigred@m1:~/wk/canary-demo$ kubectl get pod -n dev -o wide
NAME                      READY   STATUS    RESTARTS   AGE    IP             NODE   NOMINATED NODE   READINESS GATES
b1-dev-66ddcfd76d-tgbnc   1/1     Running   0          2m2s   10.244.1.181   w1     <none>           <none>

bigred@m1:~/wk/canary-demo$ curl 10.244.1.181
let me go

bigred@m1:~/wk/canary-demo$ curl 10.244.1.181/cgi-bin/info
version: v3.0.0, IP: 10.244.1.181, hostname: b1-dev-66ddcfd76d-tgbnc
```

程式設計師測試 App 沒問題
使用金絲雀部署策略

## 金絲雀部署 Part1

部署金絲雀前，先擴展 "b1-prod" pod 的數量至 4

```
$ kubectl -n prod scale deployment b1-prod --replicas=4
```

切換至 canary branch

```
$ git checkout canary
Switched to branch 'canary'
```

## 金絲雀部署 Part2

```
合併開發環境的程式碼
$ git merge dev
```

### 確認金絲雀部署狀態

至 Jenkins 網頁確認執行結果

![](https://i.imgur.com/y5SvCJf.png)


### 確認輪詢狀態

回到輪詢 /info URL 的 cmd 視窗

```
version: v2.0.0, IP: 10.244.1.182, hostname: b1-prod-75466968fd-gngbz
version: v3.0.0, IP: 10.244.0.41, hostname: b1-canary-c6fb868c-xnxs7
version: v2.0.0, IP: 10.244.2.118, hostname: b1-prod-75466968fd-5pml7
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
version: v3.0.0, IP: 10.244.0.41, hostname: b1-canary-c6fb868c-xnxs7
version: v2.0.0, IP: 10.244.0.40, hostname: b1-prod-75466968fd-x7nnw
version: v3.0.0, IP: 10.244.0.41, hostname: b1-canary-c6fb868c-xnxs7
...
```

## 金絲雀部署 Part3

切換至 master branch
```
$ git checkout master
Switched to branch 'master'
```


合併 canary branch
```
$ git merge canary
```

## 確認新版本上線

請至 Jenkins 網頁確認執行結果

![](https://i.imgur.com/1sHlEuO.png)


回到輪詢 /info URL 的 cmd 視窗確認結果

```
version: v3.0.0, IP: 10.244.0.41, hostname: b1-canary-c6fb868c-xnxs7
version: v3.0.0, IP: 10.244.0.41, hostname: b1-canary-c6fb868c-xnxs7
version: v3.0.0, IP: 10.244.1.183, hostname: b1-prod-6cbd5bd499-jl8rl
version: v3.0.0, IP: 10.244.1.183, hostname: b1-prod-6cbd5bd499-jl8rl
version: v3.0.0, IP: 10.244.1.183, hostname: b1-prod-6cbd5bd499-jl8rl
version: v3.0.0, IP: 10.244.1.183, hostname: b1-prod-6cbd5bd499-jl8rl
...
```

###### tags: `CI/CD`




















