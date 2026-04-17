# Jenkins

[TOC]

---


## 環境準備


CNT.2022.v4 K8S 叢集建置完成
以下 K8S 公設完成部署
1. MetalLB
2. Ingress NGINX Controller
3. Local Path Provisioner

在 Windows 系統的 cmd 視窗登入 m1 主機
修改 `/etc/hosts`
```
$ sudo nano /etc/hosts
	::
192.168.124.220 quay.k8s.org jenkins.k8s.org
```

參考 902 簡報完成 git 必要設定
```
$ git config --list
user.name=danny
user.email=danny@example.com
core.editor=nano
```

## Jenkins 參考資料

```bash=
https://www.jenkins.io/

https://github.com/jenkinsci/jenkins
https://www.jenkins.io/doc/book/using/
```



## Jenkins 介紹

Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.


## Jenkins Plugins

[Jenkins Plugins 網站連結](https://plugins.jenkins.io/)

There are over a thousand different plugins which can be installed on a Jenkins controller and to integrate various build tools, cloud providers, analysis tools, and much more.


## JCasC

Jenkins Configuration as Code 
https://plugins.jenkins.io/configuration-as-code/

Jenkins Configuration as Code provides the ability to define this whole configuration as a simple, human-friendly, plain text yaml syntax.

## Kubernetes plugin

Kubernetes plugin for Jenkins
https://plugins.jenkins.io/kubernetes/

Jenkins plugin to run <font color=red>dynamic agents</font> in a Kubernetes cluster.

<font color=red>**The plugin creates a Kubernetes Pod for each agent started, and stops it after each build.**</font>

It is not required to run the Jenkins controller inside Kubernetes.


---

# 部署前置作業

```bash=
$ pwd
/home/bigred/wk

$ kubectl create ns jenkins
namespace/jenkins created

## kubeconfig 為 secret 物件的名字
$ kubectl create secret generic kubeconfig --from-file=/home/bigred/.kube/config -n jenkins
secret/kubeconfig created


$ kubectl apply -f http://web.flymks.com/cicd/v1/jenkins.yaml

$ kubectl get all -n jenkins
NAME          READY   STATUS    RESTARTS   AGE
pod/jenkins   1/1     Running   0          89s

NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)            AGE
service/jenkins   ClusterIP   10.98.0.124   <none>        80/TCP,50000/TCP   89s

$ kubectl get ingress -n jenkins
NAME              CLASS   HOSTS             ADDRESS          PORTS   AGE
jenkins-ingress   ig1     jenkins.k8s.org   192.168.61.220   80      109s
```





## 登入 Jenkins

打開瀏覽器，網址 `http://jenkins.k8s.org`

```
帳號：jenkins
密碼：Jenkins12345
```

## Jenkins 主畫面

![](https://i.imgur.com/52uWm5Q.png)

## 執行第一個 Job

左側按下「馬上建置」

![](https://i.imgur.com/eLyiUAH.png)

## 檢視執行結果

滑鼠移至綠色區塊可確認 Log
左側下方「建置歷程」  點選 `#1`
選擇 Console Output 查看終端機輸出

![](https://lh4.googleusercontent.com/TYtpep0Urqf_LMYN-9rXtfAZOdgDD6kbmoRyvV_o0J0emuK4zGSSQJZkSCxA9J7mW95DnnSg7boVA4O1_L7vzVrV-KPKA0CFq8Wr-1l5o5IJvF91KfWHyS_UOM8KAbQxhN6d4NyR7AlDLDC1AA)


## 進入 Blue Ocean

左側按下「Open Blue Ocean」
進入 Blue Ocean 畫面後，點選中間第一列
如下圖紅框處


![](https://i.imgur.com/0cZv1j2.png)

## Blue Ocean 介面

![](https://i.imgur.com/OdyQFj0.png)

## 離開 Blue Ocean

右上點選 X 先離開執行結果畫面

![](https://i.imgur.com/Dwux0dn.png)

右上點選回到經典界面的圖示

![](https://i.imgur.com/h2kab1q.png)


## 設定 SSH 免密碼

Jenkins 透過 SSH 連接 Git 
Jenkins 主機中已預先產生 SSH 公私鑰
將公鑰給 M1 主機

```
$ kubectl -n jenkins exec -it jenkins -- cat /var/jenkins_home/.ssh/ssh_host_rsa_key.pub | tee -a ~/.ssh/authorized_keys
```

## mycicd 專案資料夾

於 m1 主機建立 mycicd 專案資料夾
建立一個檔案並完成 git 備份

```
~/wk$ mkdir mycicd; cd mycicd
$ git init
$ echo "Hi! Jenkins!" > readme
$ git add readme; git commit -m 'Init repo'
```

## 新增 Jenkins Job

回到 Jenkins Dashboard
點選左側新增作業
item name 輸入 mycicd
下方選擇 Multibranch Pipeline
按下 OK

![](https://i.imgur.com/15vEjYm.png)


點選 Branch Sources 頁籤 🡪 點選 Add source 下拉選 Git
Project Repository 輸入(紅字請替換為 m1 主機 IP)

```
ssh://bigred@192.168.61.4/home/bigred/wk/mycicd
```

Credentials 選擇 ssh-private-key
最後按下 Save


![](https://i.imgur.com/aB3cVql.png)


## Jenkins 掃描儲存庫

新增作業完成後 Jenkins 會自動掃描 Git 儲存庫

![](https://i.imgur.com/Mxo9ufd.png)



## 撰寫 Jenkinsfile

:::success


Jenkinsfile 使用 echo 語法輸出字串
```
$ nano Jenkinsfile
pipeline {
    agent { label 'master' }
    stages {
        stage ('echo') {
            steps {
                echo "Hello!"
            }
        }
    }
}
```

`steps` ，步驟
注意！`echo "Hello!"` 的 `echo` 不是 linux 的 `echo`

:::

## 手動通知 Jenkins

必需先將 Jenkinsfile 備份至 Git 儲存庫
```
$ git add Jenkinsfile
$ git commit -m "Add Jenkinsfile"
```

手動通知 Jenkins
```
$ curl http://jenkins.k8s.org/git/notifyCommit?url=ssh://bigred@192.168.61.4/home/bigred/wk/mycicd
No git jobs found
Scheduled indexing of mycicd
```

確認執行結果

Jenkins 網頁  Dashboard
點選 mycicd 點選 master
確認 Stage View 並檢視 Logs
至 Blue Ocean 界面確認結果

![](https://i.imgur.com/9Yuug5R.png)

# 搭配 Git Hooks

搭配 Git Hooks 達到 commit 後自動通知 Jenkins

## 使用 post-commit

```bash=
建立 post-commit
$ cat << EOF > .git/hooks/post-commit
#!/bin/bash

curl -s http://jenkins.k8s.org/git/notifyCommit?url=ssh://bigred@192.168.61.4/home/bigred/wk/mycicd 
EOF

務必賦予執行權限
$ chmod +x .git/hooks/post-commit
```

## 修改 Jenkinsfile

:::success


Jenkinsfile 使用 sh 語法執行指令

```bash=
$ nano Jenkinsfile
pipeline {
    agent { label 'master' }
    stages {
        stage ('shell') {
            steps {
                sh "whoami"
                sh "pwd"
                sh "ls -al"
            }
        }
    }
}

```

stage 取名為 shell
steps 步驟
裡面的 sh 代表要執行我們的 shell 程式，可以幫我們執行 linux 的 command

:::

## 觸發 post-commit

```
$ git add Jenkinsfile ; git commit -m "Update Jenkinsfile"
```

Jenkins 網頁 Dashboard
點選 mycicd 點選 master
確認 Stage View 並檢視 Logs
至 Blue Ocean 界面確認結果


# Jenkinsfile 練習


:::success


修改 Jenkinsfile
Pipeline 有兩個 stage，分別是 echo 與 shell
echo stage，使用 echo 語法輸出 Hello!
shell stage 依序執行以下命令
```
cat /etc/os-release
hostname
hostname -i
whoami
pwd
ls -al
```

答案：

```bash=
$ nano Jenkinsfile
pipeline {
    agent { label 'master' }
    stages {
        stage ('echo') {
            steps {
                echo "Hello"
            }
        }
        stage ('shell') {
            steps {
                sh "cat /etc/os-release"
                sh "hostname"
                sh "hostname -i"
                sh "whoami"
                sh "pwd"
                sh "ls -al"
            }
        }
    }
}

$ git add Jenkinsfile ; git commit -m "Update Jenkinsfile 2"
```

![](https://i.imgur.com/x7wTm1x.png)

:::



---


# dynamic agents



Jenkins 透過 Kubernetes plugin 可以動態產生 Pod
Agent 概念等同 Worker
Master 與 Agent 透過 JNLP 協定溝通

![](https://i.imgur.com/OIiQjLt.png)


## Jenkinsfile 範例 01


下載 Jenkinsfile 範例 01
```
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/01"
```


檢視 Jenkinsfile 範例 01

```bash=
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
"""
        }
    }
    stages {
        stage ('shell') {
            steps {
                sh 'whoami'
                sh 'pwd'
                sh 'ls -al'
                sh 'cat readme'
            }
        }
    }
}
```

如果不宣告 pod 的 Container 的 image 為 `qua.io` 來源的話，預設是 ducker.hub 的 image 中心。

## 確認範例 01 結果

```
$ git add Jenkinsfile; git commit -m "test 01"
```

觀察作業用 Pod 是否有產生

```
$ watch kubectl get pod -n jenkins
```

請至 Jenkins 網頁確認執行結果 !


---


## Multiple containers

![](https://i.imgur.com/FH40xL1.png)

可以透過 Container 的技術達成不同的效果

```bash=
## 下載 Jenkinsfile 範例 02
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/02"

## 檢視 Jenkinsfile 範例 02
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    options {
        skipDefaultCheckout true
    }
    stages {
        stage ('podman') {
            steps {
                sh 'podman version || echo "command not found"'
                container('p1') {
                    sh 'podman version'
                }
            }
        }
    }
}
```




如果要更進階的學習 Jenkins 的 pipeline ，請點[連結](https://www.jenkins.io/doc/book/pipeline/)

以上的語言最後都會翻譯成 [Groovy 語言](https://zh.wikipedia.org/zh-tw/Groovy)

```bash=
## 下載 Jenkinsfile 範例 05
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/05"

$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    options {
        skipDefaultCheckout true
    }
    stages {
        stage ('podman login') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false -u=quay -p=Quay12345 quay.k8s.org'
                }
            }
        }
    }
}
```
stage 階段
steps 步驟

Jenkins 如果沒有特定宣告哪一個 Contianer 的話，預設是 jnlp 那台 Container

結果報錯

```bash=
$ git add Jenkinsfile
$ git commit -m "test05"
+ podman login --tls-verify=false -u=quay -p=Quay12345 quay.k8s.org

Error: authenticating creds for "quay.k8s.org": pinging container registry quay.k8s.org: Get "http://quay.k8s.org/v2/": dial tcp: lookup quay.k8s.org: no such host

script returned exit code 125
```

```bash=
## Jenkinsfile 05-1
$ nano Jenkinsfile
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  hostAliases:
  - ip: "192.168.61.220"
    hostnames:
    - "quay.k8s.org"
  containers:
  - name: jnlp
--- 以下省略 ---

```





```bash=
## Jenkinsfile 05-2
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            yaml """
apiVersion: v1
kind: Pod
spec:
  hostAliases:
  - ip: "192.168.61.220"
    hostnames:
    - "quay.k8s.org"
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    options {
        skipDefaultCheckout true
    }
    stages {
        stage ('podman login') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false -u=quay -p=Quay12345 quay.k8s.org'
                }
            }
        }
        stage ('hostAliases') {
            steps {
                sh 'cat /etc/hosts'
                container('p1') {
                    sh 'cat /etc/hosts'
                }
            }
        }
    }
}
```




###### tags: `CI/CD`