# Git - Part 2

[TOC]


---

#  環境準備
CNT.2022.v4 K8S 叢集建置完成
在 Windows 系統的 cmd 視窗登入 m1 主機
下載 bookstore-inventory 程式碼

```bash=
bigred@m1:~/wk$ pwd
/home/bigred/wk

$ wget http://web.flymks.com/cicd/v1/bi.zip

$ unzip bi.zip

$ cd bookstore-inventory

## 設定 git user (請自己替換名稱)
$ git config --global user.name "danny"
$ git config --global user.email "danny@example.com"
> --global 全域設定

## git 預設使用 vim，修改設定使用 nano
$ git config --global core.editor "nano"

## 檢查 git 設定
$ git config --list
user.name=danny
user.email=danny@example.com
core.editor=nano

## git 初始化
$ git init

## git 檢查檔案狀態
$ git status
```


---

# Git Hooks 介紹

![](https://i.imgur.com/9lzrktL.png)

git 的儲存庫分為 Local （本地）和 Remote （遠端）

```bash=
$ tree .git/hooks/
.git/hooks/
├── applypatch-msg.sample
├── commit-msg.sample
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample   ------------------> 今天主要會用到的
├── pre-merge-commit.sample
├── pre-push.sample
├── pre-rebase.sample
├── pre-receive.sample
├── prepare-commit-msg.sample
├── push-to-checkout.sample
└── update.sample
```

```bash=
$ $ cat .git/hooks/pre-commit.sample
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".
```


---

## ShellCheck 介紹

<font color=red>**A shell script static analysis tool**</font>
https://github.com/koalaman/shellcheck

ShellCheck is a GPLv3 tool that gives warnings and suggestions for bash/sh shell scripts.

安裝 shellcheck
```
$ sudo apk add shellcheck
```

## 使用 ShellCheck

```bash=
$ shellcheck invdb.sh

In invdb.sh line 32:
    if [ ! -z "${1}" ]; then
         ^-- SC2236 (style): Use -n instead of ! -z.

For more information:
  https://www.shellcheck.net/wiki/SC2236 -- Use -n instead of ! -z.
```

> -z 為判斷檔案不存在，前面加驚嘆號為雙重否定


建立 pre-commit
```bash=
$ cat << EOF > .git/hooks/pre-commit
#!/bin/bash

shellcheck invdb.sh
EOF
```

務必賦予執行權限
```
$ chmod +x .git/hooks/pre-commit
```

## 觸發 pre-commit

```bash=
bigred@m1:~/wk/bookstore-inventory$ ls -al
total 20
drwxr-sr-x 3 bigred bigred 4096 Jun  9 09:11 .
drwxr-sr-x 7 bigred bigred 4096 Jun  9 09:08 ..
drwxr-sr-x 7 bigred bigred 4096 Jun  9 09:36 .git
-rwxr-xr-x 1 bigred bigred  713 Jun  5 22:05 invdb.sh
-rwxr-xr-x 1 bigred bigred  487 Jun  5 22:05 inventory
```

在 .git 的專案目錄中，除了 `.git` 目錄外，其他都為 git 的工作目錄區

```bash=
##  invdb.sh 加入 git 追蹤
$ git add invdb.sh

## 檢查狀態
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   invdb.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory


## git 備份我們的檔案
$ git commit -m 'Add invdb.sh'

In invdb.sh line 32:
    if [ ! -z "${1}" ]; then
         ^-- SC2236 (style): Use -n instead of ! -z.

For more information:
  https://www.shellcheck.net/wiki/SC2236 -- Use -n instead of ! -z.

## 觸發 git-precommit
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   invdb.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory
```

## 修改 `invdb.sh`

請依 shellcheck 建議修改 `invdb.sh`

```bash=
$ nano invdb.sh
    if [ -n "${1}" ]; then  -----> 將 "! -z" 改為 "-n"
```


完成後執行 `git status`

```bash=
$ git status
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   invdb.sh

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   invdb.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory
```

<font color=red>**注意，如果在 `git add` 檔案後又修改檔案，此時直接 `commit` ，git 只會備份舊版的檔案，所以要再 `git add` 一次**</font>

```bash=
## 將 invdb.sh 再次加入追蹤
$ git add invdb.sh

## 檢查狀態
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   invdb.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory

## 將 invdb.sh 給 git 備份
$ git commit -m 'Add invdb.sh'
[master (root-commit) ad0bb75] Add invdb.sh
 1 file changed, 36 insertions(+)
 create mode 100755 invdb.sh
 
$ shellcheck invdb.sh

$ echo $?
0
```

## 實作練習 1

修改 pre-commit，使用 shellcheck 檢查 inventory
使用 git 備份 inventory
觀察 pre-commit 執行結果
依 shellcheck 建議修改 inventory
完成 git 備份 inventory


```bash=
$ shellcheck inventory

In inventory line 10:
parm=($QUERY_STRING)
      ^-----------^ SC2206 (warning): Quote to prevent word splitting/globbing, or split robustly with mapfile or read -a.

For more information:
  https://www.shellcheck.net/wiki/SC2206 -- Quote to prevent word splitting/g...
  

$ nano inventory
echo "Content-type: application/json"
echo ""

DB_PATH="/root/bookstore-inventory/inventory.db"

IFS="=&" read -r -a parm <<< "$QUERY_STRING"  ---- > 修改這行

sql="sqlite3 -json ${DB_PATH} "

[ "$QUERY_STRING" == "" ] && $sql "select * from inventory" && exit 0

case "${parm[0]}" in
  id)
    $sql "select * from inventory where id = ${parm[1]}"
    ;;
  isbn)
    $sql "select * from inventory where isbn = ${parm[1]}"
    ;;
  *)
    $sql "select * from inventory"
    ;;
esac  

$ cat << EOF > .git/hooks/pre-commit
#!/bin/bash

shellcheck inventory
EOF

$ git commit inventory -m 'Add inventory'
```


---

# 部署 Quay on K8S

環境準備
CNT.2022.v4 K8S 叢集建置完成
以下 K8S 公設完成部署
MetalLB
Ingress NGINX Controller
Local Path Provisioner
在 Windows 系統的 cmd 視窗登入 m1 主機

```bash=
$ kubectl create ns quay
namespace/quay created

$ kubectl apply -f http://web.flymks.com/cicd/v1/quay.yaml
```

```bash=
$ sudo podman run -it --rm quay.io/flysangel/busybox:1.35.0 echo 'Hello Quay'
Hello Quay

bigred@m1:~/quay-podman$ sudo podman images | grep 1.35.0
192.168.61.4/myquay/busybox                               1.35.0            af4897967787  2 weeks ago   1.47 MB
quay.io/flysangel/busybox                                 1.35.0            af4897967787  2 weeks ago   1.47 MB

bigred@m1:~/quay-podman$ sudo podman tag quay.io/flysangel/busybox:1.35.0 quay.k8s.org/quay/busybox:1.35.0

bigred@m1:~/quay-podman$ sudo podman login --tls-verify=false quay.k8s.org -u quay -p Quay12345
Login Succeeded!

$ sudo podman push --tls-verify=false quay.k8s.org/quay/busybox:1.35.0
$ sudo podman pull quay.io/flysangel/alpine:3.15.4
$ sudo podman tag quay.io/flysangel/alpine:3.15.4 quay.k8s.org/quay/alpine:3.15.4
$ sudo podman images
$ sudo podman images | grep k8s
$ sudo podman push --tls-verify=false quay.k8s.org/quay/alpine:3.15.4
$ sudo podman pull quay.io/flysangel/nginx:1.22.0
$ sudo podman tag quay.io/flysangel/nginx:1.22.0 quay.k8s.org/quay/nginx:1.22.0
$ sudo podman push --tls-verify=false quay.k8s.org/quay/nginx:1.22.0
```


---

## 實作練習 2

撰寫 bookstore-inventory Dockerfile
可參考備註 alpine httpd Dockerfile
Dockerfile 請指定工作目錄
`WORKDIR /root/bookstore-inventory`
Build Image Tag 如下
`quay.k8s.org/quay/bookstore-inventory:1.0.0`
container 測試沒問題後，完成 git 備份
Push Image 到落地 Quay


```bash=
$ nano bookstore-inventory/Dockerfile
FROM quay.k8s.org/quay/alpine:3.15.4
WORKDIR /root/bookstore-inventory
COPY invdb.sh /root/bookstore-inventory
COPY inventory /root/bookstore-inventory
RUN \
  apk update && \
  apk add --no-cache nano sudo bash wget curl tree grep sqlite && \
  wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
  chmod +x busybox-x86_64 && \
  mv busybox-x86_64 /bin/busybox1.28 && \
  bash invdb.sh create && \
  mkdir -p /opt/www/cgi-bin && echo "let me go" > /opt/www/index.html && \
  cp inventory /opt/www/cgi-bin/

ENTRYPOINT ["/bin/busybox1.28"]
CMD ["httpd", "-f", "-h", "/opt/www"]

$ sudo podman build --tls-verify=false -t quay.k8s.org/quay/bookstore-inventory:1.0.0 bookstore-inventory/

$ sudo podman run --rm --name b1 -d -p 8080:80 quay.k8s.org/quay/bookstore-inventory:1.0.0

$ curl localhost:8080

$ curl localhost:8080/cgi-bin/inventory

$ curl localhost:8080/cgi-bin/inventory?id=1

$ curl localhost:8080/cgi-bin/inventory?isbn=1492090719

$ git add Dockerfile

$ git status

$ git commit -m "quay.k8s.org/quay/bookstore-inventory:1.0.0"

$ git status

$ sudo podman push --tls-verify=false quay.k8s.org/quay/bookstore-inventory:1.0.0
```


###### tags: `CI/CD`


































