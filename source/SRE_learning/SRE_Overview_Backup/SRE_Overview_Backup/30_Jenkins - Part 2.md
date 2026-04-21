# Jenkins - Part 2

[TOC]

---

## 前置作業

若已完成簡報 904-Jenkins-Part1 請跳過此頁
請依簡報 904-Jenkins-Part1 完成以下事項
第 2~4 頁環境準備
第 10~13 頁部署 Jenkins
第 20 頁設定 SSH 免密碼
第 21~24 頁新增 Jenkins 作業「mycicd」
第 28~29 頁建立 post-commit

## Jenkinsfile 範例 06

:::success


```bash=
$ pwd
/home/bigred/wk/mycicd

下載 Jenkinsfile 範例 06
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/06"

檢視 Jenkinsfile 範例 06
$ cat Jenkinsfile
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
    options {
        skipDefaultCheckout true
    }
    environment {
        QUAY_ADMIN = credentials('quay-admin-id')
    }
    stages {
        stage ('QUAY_ADMIN') {
            steps {
                sh 'echo ${QUAY_ADMIN}'
                sh 'echo ${QUAY_ADMIN_USR}'
                sh 'echo ${QUAY_ADMIN_PSW}'
            }
        }
    }
}
```

`environment` 是 pipeline 的原生語法
`credentials` 是 要在 Jenkins 上安裝 Credentials Binding Plugin 這個插件，才能使用這個語法

我們部署的 Jenkins 的 Server 是老師先客製化過了，原生乾淨的 Jenkins Server 沒有這些設定。
QUAY_ADMIN = credentials('quay-admin-id') >> quay/Quay12345
使用 credentials 時會同時生成兩個變數 QUAY_ADMIN_USR、QUAY_ADMIN_PSW
密碼會被遮罩處理 >> ****


:::

## 動手修改範例 06

```bash=
$ cat Jenkinsfile
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
    environment {
        QUAY_ADMIN = credentials('quay-admin-id')
    }
    stages {
        stage ('QUAY_ADMIN') {
            steps {
                sh 'echo ${QUAY_ADMIN}'
                sh 'echo ${QUAY_ADMIN_USR}'
                sh 'echo ${QUAY_ADMIN_PSW}'
            }
        }
        stage ('podman login') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false quay.k8s.org -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW}'
                }
            }
        }
    }
}
```

## Jenkinsfile 範例 07

請至落地 Qauy 將 alpine 設定為 private
下載 Jenkinsfile 範例 07
```bash=
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/07"

## 檢視 Jenkinsfile 範例 07
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'p1'
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
    environment {
        QUAY_ADMIN = credentials('quay-admin-id')
        IMAGE_TAG = 'quay.k8s.org/quay/alpine.httpd:1.0.0'
    }
    stages {
        stage ('podman login') {
            steps {
                sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
            }
        }
        stage ('build image') {
            steps {
                sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                sh 'podman images'
            }
        }
    }
}

$ nano Dockerfile
FROM quay.k8s.org/quay/alpine:3.15.4
RUN \
  apk update && \
  apk add --no-cache nano sudo bash wget curl git tree grep && \
  wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
  chmod +x busybox-x86_64 && \
  mv busybox-x86_64 bin/busybox1.28 && \
  mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

ENTRYPOINT ["/bin/busybox1.28"]
CMD ["httpd", "-f", "-h", "/opt/www"]

$ git add Dockerfile Jenkinsfile

$ git commit -m 'test 07'
```





## 動手修改範例 07

修改 Jenkinsfile 範例 07
新增一個 stage 叫 push image
將 alpine.httpd:1.0.0 push 到落地 Quay
至落地 Quay 確認是否上傳成功


```bash=
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'p1'
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
    environment {
        QUAY_ADMIN = credentials('quay-admin-id')
        IMAGE_TAG = 'quay.k8s.org/quay/alpine.httpd:1.0.0'
    }
    stages {
        stage ('podman login') {
            steps {
                sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
            }
        }
        stage ('build image') {
            steps {
                sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                sh 'podman images'
            }
        }
        stage ('push image') {
            steps {
                sh 'podman push --tls-verify=false "${IMAGE_TAG}"'
            }
        }
    }
}

$ git commit -a -m "test-07-1"
```


## Jenkinsfile 範例 08

下載 Jenkinsfile 範例 08

```bash=
$ wget -q -O Jenkinsfile \
"http://web.flymks.com/cicd/v1/jenkinsfile/08"

檢視 Jenkinsfile 範例 08
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'k1'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    options {
        skipDefaultCheckout true
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id')
    }
    stages {
        stage ('get node') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl get nodes -o wide'
            }
        }
    }
}

$ git add Jenkinsfile; git commit -m "test 08"

```


:::spoiler `jenkins.yaml`詳細設定

```bash=
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - pods/exec
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - pods/log
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - watch
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: jenkins
---
apiVersion: v1
data:
  credentials.yaml: |
    Y3JlZGVudGlhbHM6CiAgc3lzdGVtOgogICAgZG9tYWluQ3JlZGVudGlhbHM6CiAgICAgIC
    0gY3JlZGVudGlhbHM6CiAgICAgICAgICAtIHVzZXJuYW1lUGFzc3dvcmQ6CiAgICAgICAg
    ICAgICAgaWQ6ICJnaXRlYS1hZG1pbi1pZCIKICAgICAgICAgICAgICB1c2VybmFtZTogIm
    dpdGVhIgogICAgICAgICAgICAgIHBhc3N3b3JkOiAiR2l0ZWExMjM0NSIKICAgICAgICAg
    ICAgICBkZXNjcmlwdGlvbjogImdpdGVhIGFkbWluIgogICAgICAgICAgICAgIHNjb3BlOi
    BHTE9CQUwKICAgICAgICAgIC0gdXNlcm5hbWVQYXNzd29yZDoKICAgICAgICAgICAgICBp
    ZDogInF1YXktYWRtaW4taWQiCiAgICAgICAgICAgICAgdXNlcm5hbWU6ICJxdWF5IgogIC
    AgICAgICAgICAgIHBhc3N3b3JkOiAiUXVheTEyMzQ1IgogICAgICAgICAgICAgIGRlc2Ny
    aXB0aW9uOiAicXVheSBhZG1pbiIKICAgICAgICAgICAgICBzY29wZTogR0xPQkFMCiAgIC
    AgICAgICAtIGJhc2ljU1NIVXNlclByaXZhdGVLZXk6CiAgICAgICAgICAgICAgaWQ6ICJz
    c2gtcHJpdmF0ZS1rZXktaWQiCiAgICAgICAgICAgICAgdXNlcm5hbWU6ICJzc2gtcHJpdm
    F0ZS1rZXkiCiAgICAgICAgICAgICAgZGVzY3JpcHRpb246ICJzc2ggcHJpdmF0ZSBrZXki
    CiAgICAgICAgICAgICAgc2NvcGU6IEdMT0JBTAogICAgICAgICAgICAgIHByaXZhdGVLZX
    lTb3VyY2U6CiAgICAgICAgICAgICAgICBkaXJlY3RFbnRyeToKICAgICAgICAgICAgICAg
    ICAgcHJpdmF0ZUtleTogIiR7cmVhZEZpbGU6L3Zhci9qZW5raW5zX2hvbWUvLnNzaC9zc2
    hfaG9zdF9yc2Ffa2V5fSIKICAgICAgICAgIC0gZmlsZToKICAgICAgICAgICAgICBpZDog
    Imt1YmVjb25maWctaWQiCiAgICAgICAgICAgICAgZmlsZU5hbWU6ICJrdWJlY29uZmlnIg
    ogICAgICAgICAgICAgIHNlY3JldEJ5dGVzOiAiJHtyZWFkRmlsZUJhc2U2NDovcnVuL3Nl
    Y3JldHMva3ViZWNvbmZpZy9jb25maWd9IgogICAgICAgICAgICAgIGRlc2NyaXB0aW9uOi
    Aia3ViZWNvbmZpZyIKICAgICAgICAgICAgICBzY29wZTogR0xPQkFM
  jenkins.yaml: |
    dW5jbGFzc2lmaWVkOgogIGxvY2F0aW9uOgogICAgdXJsOiAiJHtKRU5LSU5TX1VSTDotaH
    R0cDovL2plbmtpbnMuazhzLm9yZy99IgogIGdpdGVhU2VydmVyczoKICAgIHNlcnZlcnM6
    CiAgICAgIC0gY3JlZGVudGlhbHNJZDogImdpdGVhLWFkbWluLWlkIgogICAgICAgIGRpc3
    BsYXlOYW1lOiAiZ2l0ZWEiCiAgICAgICAgbWFuYWdlSG9va3M6IHRydWUKICAgICAgICBz
    ZXJ2ZXJVcmw6ICJodHRwOi8vZ2l0ZWEuZ2l0ZWE6MzAwMC8iCmplbmtpbnM6CiAgZGlzYW
    JsZWRBZG1pbmlzdHJhdGl2ZU1vbml0b3JzOgogICAgLSAiamVua2lucy5zZWN1cml0eS5R
    dWV1ZUl0ZW1BdXRoZW50aWNhdG9yTW9uaXRvciIKICAgIC0gImplbmtpbnMuZGlhZ25vc3
    RpY3MuQ29udHJvbGxlckV4ZWN1dG9yc0FnZW50cyIKICBjbG91ZHM6CiAgICAtIGt1YmVy
    bmV0ZXM6CiAgICAgICAgbmFtZTogImt1YmVybmV0ZXMiCiAgICAgICAgbmFtZXNwYWNlOi
    AiamVua2lucyIKICAgICAgICBqZW5raW5zVXJsOiAiaHR0cDovL2plbmtpbnMiCiAgbGFi
    ZWxTdHJpbmc6ICJtYXN0ZXIiCiAgc2VjdXJpdHlSZWFsbToKICAgIGxvY2FsOgogICAgIC
    BhbGxvd3NTaWdudXA6IGZhbHNlCiAgICAgIHVzZXJzOgogICAgICAgIC0gaWQ6ICIke0pF
    TktJTlNfQURNSU5fSUQ6LWplbmtpbnN9IgogICAgICAgICAgcGFzc3dvcmQ6ICIke0pFTk
    tJTlNfQURNSU5fUFc6LUplbmtpbnMxMjM0NX0iCiAgYXV0aG9yaXphdGlvblN0cmF0ZWd5
    OgogICAgZ2xvYmFsTWF0cml4OgogICAgICBwZXJtaXNzaW9uczoKICAgICAgICAtICJVU0
    VSOk92ZXJhbGwvQWRtaW5pc3Rlcjoke0pFTktJTlNfQURNSU5fSUQ6LWplbmtpbnN9Igog
    ICAgICAgIC0gIkdST1VQOk92ZXJhbGwvUmVhZDphdXRoZW50aWNhdGVkIgogIHJlbW90aW
    5nU2VjdXJpdHk6CiAgICBlbmFibGVkOiB0cnVlCnNlY3VyaXR5OgogIHF1ZXVlSXRlbUF1
    dGhlbnRpY2F0b3I6CiAgICBhdXRoZW50aWNhdG9yczoKICAgICAgLSBnbG9iYWw6CiAgIC
    AgICAgICBzdHJhdGVneTogdHJpZ2dlcmluZ1VzZXJzQXV0aG9yaXphdGlvblN0cmF0ZWd5
    Cg==
  jobs.yaml: |
    am9iczoKICAtIHNjcmlwdDogPgogICAgICBwaXBlbGluZUpvYignZmlyc3Qtam9iJykgew
    ogICAgICAgIGRlZmluaXRpb24gewogICAgICAgICAgY3BzIHsKICAgICAgICAgICAgc2Fu
    ZGJveCgpCiAgICAgICAgICAgIHNjcmlwdCgiIiJcCiAgICAgICAgICAgICAgcGlwZWxpbm
    UgewogICAgICAgICAgICAgICAgYWdlbnQgeyBsYWJlbCAnbWFzdGVyJyB9CiAgICAgICAg
    ICAgICAgICBzdGFnZXMgewogICAgICAgICAgICAgICAgICBzdGFnZSAoJ2VjaG8nKSB7Ci
    AgICAgICAgICAgICAgICAgICAgc3RlcHMgewogICAgICAgICAgICAgICAgICAgICAgZWNo
    byAiSGVsbG8hIgogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICAgfQ
    ogICAgICAgICAgICAgICAgICBzdGFnZSAoJ3NoZWxsJykgewogICAgICAgICAgICAgICAg
    ICAgIHN0ZXBzIHsKICAgICAgICAgICAgICAgICAgICAgIHNoICdjYXQgL2V0Yy9vcy1yZW
    xlYXNlJwogICAgICAgICAgICAgICAgICAgICAgc2ggJ2hvc3RuYW1lJwogICAgICAgICAg
    ICAgICAgICAgICAgc2ggJ2hvc3RuYW1lIC1pJwogICAgICAgICAgICAgICAgICAgICAgc2
    ggJ3dob2FtaScKICAgICAgICAgICAgICAgICAgICAgIHNoICdwd2QnCiAgICAgICAgICAg
    ICAgICAgICAgICBzaCAnbHMgLWFsJwogICAgICAgICAgICAgICAgICAgIH0KICAgICAgIC
    AgICAgICAgICAgfQogICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgIH0iIiIuc3Ry
    aXBJbmRlbnQoKSkKICAgICAgICAgIH0KICAgICAgICB9CiAgICAgIH0K
kind: Secret
metadata:
  labels:
    app: jenkins
  name: jenkins-casc
  namespace: jenkins
type: Opaque
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
spec:
  ports:
  - name: jenkins-http
    port: 80
    targetPort: 8080
  - name: jenkins-agent
    port: 50000
    protocol: TCP
    targetPort: 50000
  selector:
    app: jenkins
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: local-path
  volumeMode: Filesystem
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins-ingress
  namespace: jenkins
spec:
  ingressClassName: ig1
  rules:
  - host: jenkins.k8s.org
    http:
      paths:
      - backend:
          service:
            name: jenkins
            port:
              number: 80
        path: /
        pathType: Prefix
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
spec:
  containers:
  - env:
    - name: LIMITS_MEMORY
      valueFrom:
        resourceFieldRef:
          divisor: 1Mi
          resource: limits.memory
    - name: JAVA_OPTS
      value: -Djenkins.install.runSetupWizard=false -Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Taipei
        -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0
        -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
    image: quay.io/grassknot/jenkins:jcasc
    imagePullPolicy: Always
    name: jenkins
    ports:
    - containerPort: 8080
    - containerPort: 50000
    resources:
      limits:
        cpu: 1
        memory: 2Gi
      requests:
        cpu: 0.5
        memory: 500Mi
    volumeMounts:
    - mountPath: /usr/jenkins_config/
      name: jenkins-config
    - mountPath: /var/jenkins_home
      name: jenkins-storage
    - mountPath: /run/secrets/kubeconfig
      name: kubeconfig
      readOnly: true
  securityContext:
    fsGroup: 1000
    runAsUser: 1000
  serviceAccountName: jenkins
  volumes:
  - name: jenkins-storage
    persistentVolumeClaim:
      claimName: jenkins-pvc
  - name: jenkins-config
    secret:
      secretName: jenkins-casc
  - name: kubeconfig
    secret:
      secretName: kubeconfig
```

:::

> Jenkins 在 Kubernetes 裡面的權限只能對 pod 做管理，其他的 k8s 物件是沒有權限的，所以才要要讓 Jenkins 能使用 Kubectl 來對整個 Kubernetes 來做管理。 

![](https://i.imgur.com/dYgwTQO.png)


---


## 動手修改範例 08

```bash=
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'k1'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    options {
        skipDefaultCheckout true
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id')
    }
    stages {
        stage ('get node') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl get nodes -o wide'
            }
        }
        stage ('get jenkins') {
            steps {
                sh 'kubectl get all -n Jenkins'
            }
        }
        stage ('get quay') {
            steps {
                sh 'kubectl get all -n quay'
            }
        }
        stage ('get default') {
            steps {
                sh 'kubectl get all'
            }
        }
    }
}


```


# 實作練習

請先在 K8S 叢集建立一個 namespace 命名為 cicd
首先撰寫 pod-a1.yaml
a1 pod 部署在 cicd namespace
image 使用 `quay.k8s.org/quay/alpine.httpd`
imagePullPolicy 設定 Always


```bash=
$ kubectl create ns cicd

$ nano Jenkinsfile
$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'k1'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: quay.io/flysangel/inbound-agent:4.13-2-jdk11
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
        KUBECONFIG = credentials('kubeconfig-id')
    }
    stages {
        stage ('deploy a1') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl delete -f pod-a1.yml || exit 0'
                sh 'kubectl apply -f pod-a1.yml'
            }
        }
        stage ('get cicd') {
            steps {
                sh 'kubectl get pod -n cicd'
            }
        }
    }
}

$ nano Dockerfile
FROM quay.k8s.org/quay/alpine:3.15.4
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




## 總練習 1

請把落地 Quay 中的 alpine.httpd 刪除
參考前面的 Jenkinsfile 範例 07 08 與練習
撰寫四個 stage
podman login
build image
push image
deploy a1

```bash=
$ nano Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mycicd'
            defaultContainer 'k1'
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
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id')
        QUAY_ADMIN = credentials('quay-admin-id')
        IMAGE_TAG = 'quay.k8s.org/quay/alpine.httpd:1.0.0'
    }
    stages {
        stage ('podman login') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
                }
            }
        }
        stage ('build image') {
            steps {
                container('p1') {
                    sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                    sh 'podman images'
                }
            }
        }
        stage ('push image') {
            steps {
                container('p1') {
                    sh 'podman push --tls-verify=false "${IMAGE_TAG}"'
                }
            }
        }
        stage ('deploy a1') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl delete -f pod-a1.yml || exit 0'
                sh 'kubectl apply -f pod-a1.yml'
            }
        }
    }
}
```


---

## imagePullSecrets

在 m1 主機登入落地 Quay

```
$ sudo podman login --tls-verify=false quay.k8s.org -u quay -p Quay12345
```

檢視 auth.json
```bash=
$ sudo cat /run/containers/0/auth.json
{
        "auths": {
                "quay.k8s.org": {
                        "auth": "cXVheTpRdWF5MTIzNDU="
                }
        }
}
```


複製 auth.json 到家目錄並變更擁有者
```
$ sudo cp /run/containers/0/auth.json ~
$ sudo chown bigred:bigred ~/auth.json
```

建立 K8S Secret
```bash=
$ kubectl create secret generic regcred \
--from-file=.dockerconfigjson=/home/bigred/auth.json \
--type=kubernetes.io/dockerconfigjson \
-n cicd
```

regcred = registry + credential (都是名字，可以自取)


```bash=
spec:
  containers:
    - name: a1
      image: quay.k8s.org/quay/alpine.httpd:1.0.0
      imagePullPolicy: Always
  imagePullSecrets:
    - name: regcred
```

---


## 總練習 2 前置作業


```bash=
$ pwd
/home/bigred/wk

$ wget http://web.flymks.com/cicd/v1/bi2.zip
$ unzip bi2.zip
$ cd bi2
$ ls
Dockerfile  invdb.sh  inventory
```


## 總練習 2-1

題目：

- 首先將 bi2 專案資料夾完成 git 初始化
- Build Image Tag 如下
  `quay.k8s.org/quay/bookstore-inventory:1.0.0`
- 撰寫 K8S YAML 部署庫存管理系統 (pod-b1.yaml)
- b1 pod 部署在 cicd namespace
- imagePullPolicy 設定 Always
- 新增 Jenkins multibranch pipeline 作業，作業命名「bi2」並連接到 bi2 git 儲存庫
- 記得建立 git hook post-commit
- 參考總練習 1 撰寫 Jenkinsfile

:::success


答案：

```bash=
## git 初始化
$ git init

$ ls -al
total 24
drwxr-sr-x 3 bigred bigred 4096 Jun 15 09:26 .
drwxr-sr-x 9 bigred bigred 4096 Jun 15 09:20 ..
drwxr-sr-x 7 bigred bigred 4096 Jun 15 09:26 .git
-rw-r--r-- 1 bigred bigred  616 Jun  9 23:41 Dockerfile
-rwxr-xr-x 1 bigred bigred  711 Jun  9 23:39 invdb.sh
-rwxr-xr-x 1 bigred bigred  476 Jun  9 23:41 inventory

## 建立 post-commit
$ cat << EOF > .git/hooks/post-commit
#!/bin/bash

curl -s http://jenkins.k8s.org/git/notifyCommit?url=ssh://bigred@192.168.61.4/home/bigred/wk/bi2/
EOF

## 務必賦予執行權限
$ chmod +x .git/hooks/post-commit
```

新增 Jenkins Job

回到 Jenkins Dashboard
- 點選左側新增作業
	- item name 輸入 bi2
	- 下方選擇 Multibranch Pipeline
	- 按下 OK

點選 Branch Sources 頁籤 
- 點選 Add source 下拉選 Git
- Project Repository 輸入(紅字請替換為 m1 主機 IP)
	- `ssh://bigred@192.168.61.4/home/bigred/wk/bi2`
	- Credentials 選擇 ssh-private-key
	- 最後按下 Save



編輯 pod-b1.yml

```bash=
$ nano pod-b1.yml
apiVersion: v1
kind: Pod
metadata:
  name: b1
  namespace: cicd
spec:
  containers:
    - name: b1
      image: quay.k8s.org/quay/bookstore-inventory:1.0.0
      imagePullPolicy: Always
  imagePullSecrets:
    - name: regcred
```

編輯 Jenkinsfile

```bash=
$ nano Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'bi2'
            defaultContainer 'k1'
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
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id')
        QUAY_ADMIN = credentials('quay-admin-id')
        IMAGE_TAG = 'quay.k8s.org/quay/bookstore-inventory:1.0.0'
    }
    stages {
        stage ('podman login') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
                }
            }
        }
        stage ('build image') {
            steps {
                container('p1') {
                    sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                    sh 'podman images'
                }
            }
        }
        stage ('push image') {
            steps {
                container('p1') {
                    sh 'podman push --tls-verify=false "${IMAGE_TAG}"'
                }
            }
        }
        stage ('deploy b1') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl delete -f pod-b1.yml || exit 0'
                sh 'kubectl apply -f pod-b1.yml'
            }
        }
    }
}
```
```bash=
git add *
git status
git commit -m 'Build Project test'
```
![](https://i.imgur.com/aJnH03D.png)


```bash=
$ kubectl get pod -n cicd -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP             NODE   NOMINATED NODE   READINESS GATES
b1     1/1     Running   0          4m25s   10.244.1.169   w1     <none>           <none>

$ curl 10.244.1.169/cgi-bin/inventory
[{"id":1,"isbn":"1492090719","quantity":17},
{"id":2,"isbn":"9865024918","quantity":11},
{"id":3,"isbn":"1492092304","quantity":16},
{"id":4,"isbn":"9864760785","quantity":10},
{"id":5,"isbn":"9865028042","quantity":12}]
```

:::


---


## 總練習 2-2

請先完成總練習 2-1
- 將 podman login 指令移至 build image stage 中
- 將 podman login stage 刪除
- 在 build image stage 之前新增兩個 stage
	- 第一個 stage 命名為 shellcheck
		- 負責檢查 `invdb.sh` 以及 `inventory`
		- image 使用 `quay.io/grassknot/shellcheck-alpine:stable`
	- 第二個 stage 命名為 Test
		- image 使用 `quay.k8s.org/quay/alpine.httpd:1.0.0`
		- 使用 curl 將 API 結果輸出，執行結果可參考下圖


答案：

```bash=
$ kubectl create secret generic regcred \
> --from-file=.dockerconfigjson=/home/bigred/auth.json \
> --type=kubernetes.io/dockerconfigjson \
> -n jenkins

$ kubectl get secret -n jenkins
NAME           TYPE                             DATA   AGE
jenkins-casc   Opaque                           3      5d22h
kubeconfig     Opaque                           1      5d22h
regcred        kubernetes.io/dockerconfigjson   1      2s

$ nano Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'bi2'
            defaultContainer 'k1'
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
  - name: s1
    image: quay.io/grassknot/shellcheck-alpine:stable
    securityContext:
      privileged: true
    command: ["sleep"]
    args: ["infinity"]
  - name: t1
    image: quay.k8s.org/quay/alpine.httpd:1.0.0
    securityContext:
      privileged: true
  - name: k1
    securityContext:
      privileged: true
    image: quay.io/grassknot/kubectl:1.24.1
    command: ["sleep"]
    args: ["infinity"]
  - name: p1
    securityContext:
      privileged: true
    image: quay.io/podman/stable:v3.4.7
    command: ["sleep"]
    args: ["infinity"]
"""
        }
    }
    environment {
        KUBECONFIG = credentials('kubeconfig-id')
        QUAY_ADMIN = credentials('quay-admin-id')
        IMAGE_TAG = 'quay.k8s.org/quay/bookstore-inventory:1.0.0'
    }
    stages {
        stage ('shellcheck') {
            steps {
                container('s1') {
                    sh 'shellcheck invdb.sh'
                    sh 'shellcheck inventory'
                }
            }
        }
        stage ('test') {
            steps {
                container('t1') {
                    sh '''
                       #!/bin/bash
                       apk update && apk add sqlite
                       mkdir -p /root/bookstore-inventory/ && mkdir -p /opt/www/cgi-bin
                       cp invdb.sh /root/bookstore-inventory/ && cp inventory /opt/www/cgi-bin
                       cd /root/bookstore-inventory/
                       bash invdb.sh create
                       '''
                    sh 'curl -s localhost'
                    sh 'curl -s localhost/cgi-bin/inventory?id=1'
                    sh 'curl -s localhost/cgi-bin/inventory?isbn=1492090719'
                }
            }
        }
        stage ('build image') {
            steps {
                container('p1') {
                    sh 'podman login --tls-verify=false -u=${QUAY_ADMIN_USR} -p=${QUAY_ADMIN_PSW} quay.k8s.org'
                    sh 'podman build --tls-verify=false -t "${IMAGE_TAG}" .'
                    sh 'podman images'
                }
            }
        }
        stage ('push image') {
            steps {
                container('p1') {
                    sh 'podman push --tls-verify=false "${IMAGE_TAG}"'
                }
            }
        }
        stage ('deploy b1') {
            steps {
                sh 'mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config'
                sh 'kubectl delete -f pod-b1.yml || exit 0'
                sh 'kubectl apply -f pod-b1.yml'
            }
        }
    }
}

$ git add Jenkinsfile ; git commit -m "t2"
```

![](https://i.imgur.com/tj2gniM.png)



---

# 範例 09
## 前置作業 1

建立 mp3 專案資料夾 (請先不要建立 post-commit)

```bash=
~/wk$ mkdir mp3; cd mp3
$ git init

建立 Jenkinsfile，內容從備忘稿複製
$ nano Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mp3'
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
    options {
        skipDefaultCheckout true
    }
    stages {
        stage ('echo') {
            steps {
                echo 'Hello World!'
            }
        }
    }
}

$ git add Jenkinsfile; git commit -m 'Init repo'
```

## git branch

```bash=
$ git branch
* master

建立 branch
$ git branch production
$ git branch dev

$ git branch
  dev
* master
  production
```


## .git/refs/heads

```bash=
$ git log
commit 36961d234c7f676f422bfbdda8536ed528055337 (HEAD -> master, prod, dev)
Author: danny <danny@example.com>
Date:   Wed Jun 15 13:35:19 2022 +0800

    Init repo
```

可以看到 commit 的代碼：`36961d234c7f676f422bfbdda8536ed528055337`

檢查我們建立的 branch 
```
$ tree .git/refs/heads
.git/refs/heads
├── dev
├── master
└── prod

0 directories, 3 files
```

在同一個 git 專案目錄中新增的 branch 都是共用相同的 commit ID

```bash=
$ cat .git/refs/heads/master
36961d234c7f676f422bfbdda8536ed528055337
$ cat .git/refs/heads/production
36961d234c7f676f422bfbdda8536ed528055337
$ cat .git/refs/heads/dev
36961d234c7f676f422bfbdda8536ed528055337
```


## 手動刪除 branch

```
$ rm .git/refs/heads/production
$ git branch
  dev
* master
```

重新建立 branch，命名為 prod

```
$ git branch prod
```


---

## 前置作業 2

- 新增 Jenkins multibranch pipeline 作業
- 作業命名「mp3」並連接到 mp3 git 儲存庫
	- `ssh://bigred@192.168.61.4/home/bigred/wk/mp3`
- 完成後至 Blue Ocean 網頁查看分支

不同的 branch ，Jenkins 會用不同的 pod 來作業

## Jenkinsfile 範例 09

```bash=
$ wget -q -O Jenkinsfile \
> "http://web.flymks.com/cicd/v1/jenkinsfile/09"

$ cat Jenkinsfile
pipeline {
    agent {
        kubernetes {
            inheritFrom 'mp3'
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
    options {
        skipDefaultCheckout true
    }
    stages {
        stage ('echo') {
            steps {
                echo 'Hello World!'
            }
        }
        stage ('Deploy Production') {
            when {
                branch 'prod'
            }
            steps {
                echo 'Hello Prod!'
            }
        }
    }
}
```

```bash=
$ git add Jenkinsfile; git commit -m "test 09"

$ git log
commit a154b98d53d31aefd40beb213fe2580241b5bfcf (HEAD -> master)
Author: danny <danny@example.com>
Date:   Wed Jun 15 14:13:47 2022 +0800

    test 09

commit 36961d234c7f676f422bfbdda8536ed528055337 (prod, dev)
Author: danny <danny@example.com>
Date:   Wed Jun 15 13:35:19 2022 +0800

    Init repo
```

手動執行作業

![](https://i.imgur.com/uM6DSzj.png)


## 動手修改範例 09

在 Deploy Production stage 之前新增一個 stage
stage 命名為 Deploy Dev
when 宣告如下
```
when {
    not { branch 'master' }
}
```
steps 只需要一個 echo  'Hello Dev!'
請參考簡報 904 建立 git hook post-commit

```bash=
$ git checkout dev
$ cat Jenkinsfile
$ git pull . master
$ git log --oneline
```



實作練習 Part1
```bash=
$ git checkout prod
Switched to branch 'prod'

bigred@m1:~/wk/mp3$ git pull . master
From .
 * branch            master     -> FETCH_HEAD
Updating 36961d2..978df0c
Fast-forward
 Jenkinsfile | 18 +++++++++++++++++-
 1 file changed, 17 insertions(+), 1 deletion(-)
 
$ git log --oneline
```



```bash=
## 建立 post-commit
$ cat << EOF > .git/hooks/post-commit
#!/bin/bash

curl -s http://jenkins.k8s.org/git/notifyCommit?url=ssh://bigred@192.168.61.4/home/bigred/wk/mp3/
EOF

## 務必賦予執行權限
$ chmod +x .git/hooks/post-commit
```













































































































































###### tags: `CI/CD`















































































