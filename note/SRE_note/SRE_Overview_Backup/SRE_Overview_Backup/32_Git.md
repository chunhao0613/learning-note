# Git

:::spoiler **目錄**

[TOC]

---

:::

## Git 介紹
<font color=red>Git was created by Linus Torvalds in 2005 for development of the Linux kernel </font>, with other kernel developers contributing to its initial development. Since 2005, Junio Hamano has been the core maintainer. As with most other <font color=red>**distributed version control** </font>systems, and unlike most client–server systems, every Git directory on every computer is a full-fledged repository with complete history and full version-tracking abilities, independent of network access or a central server. Git is free and open-source software distributed under GNU General Public License Version 2.

- Git 屬於 Distributed Version Control 分散式版控軟體
    - 最早的版本由 Linus Torvalds 於 2005 開發完成
    - 具備離線功能，有可以線上發佈
    - 功能的核心在於備份程式內容

by 程威

## 情境說明

在網路書店擔任程式設計師
負責書目庫存查詢系統開發
由兩個子系統組成
書目管理系統(已完成)
庫存查詢系統(開發中)  <----------- <font color=red> 搭配 Git 進行系統開發 </font>


## 系統規格與目標

使用 httpd + CGI 完成一個 REST API
資料庫使用 SQLite 3

Table Schema

| inventory  |
| ---------- |
| isbn       |
| quantity   |


搭配 Git 完成源碼備份


---

# 建立開發環境

```bash=
$ docker run -itd --rm --name dev quay.io/flysangel/alpine:3.15.4

$ docker ps -a

$ docker exec -it dev /bin/sh
/ # 

/ # apk update && apk upgrade

/ # apk add nano wget curl tree bash grep

/ # exit

$ docker exec -it dev /bin/bash

bash-5.1# cd

bash-5.1# pwd

bash-5.1# ls -al

```

## 開發環境準備

```
## 下載 busybox 1.28 版

bash-5.1# wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64
bash-5.1# chmod +x busybox-x86_64
bash-5.1# mv busybox-x86_64 /bin/busybox1.28

## 使用 busybox1.28 httpd 啟動網站伺服器
bash-5.1# mkdir -p /opt/www && echo "let me go" > /opt/www/index.html
bash-5.1# busybox1.28 httpd -h /opt/www

## 安裝sqlite
bash-5.1# apk add sqlite
```

## 安裝 git

```
$ apk update && apk add git

確認 git 版本
$ git version
git version 2.34.2
```

## git 必要設定

```bash=
設定 git user (請自己替換名稱)
$ git config --global user.name "danny"
$ git config --global user.email "danny@example.com"
> --global 全域設定

git 預設使用 vim，修改設定使用 nano
$ git config --global core.editor "nano"

檢查 git 設定
$ git config --list
user.name=danny
user.email=danny@example.com
core.editor=nano

```

## 建立專案資料夾

```bash=
bash-5.1# mkdir bookstore-inventory
bash-5.1# cd bookstore-inventory/
bash-5.1# pwd
/root/bookstore-inventory

## git 初始化
bash-5.1# git init
...
## 等等備份的資料都會存到/root/bookstore-inventory/.git/
Initialized empty Git repository in /root/bookstore-inventory/.git/

bash-5.1# ls -al
total 12
drwxr-xr-x    3 root     root          4096 May 27 06:32 .
drwx------    1 root     root          4096 May 27 06:32 ..
drwxr-xr-x    7 root     root          4096 May 27 06:32 .git
```

## 庫存管理系統開發
撰寫建立系統資料庫的腳本
```
$ nano invdb.sh
#!/bin/bash

DB_NAME="inventory.db"

## 建立資料庫
create() {
  sqlite3 ${DB_NAME} <<EOF               
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    isbn TEXT NOT NULL,
    quantity INTEGER NOT NULL
);
EOF
}

create
```
> AUTOINCREMENT，意思是等等的ID會自動增加

執行程式
```
$ chmod +x invdb.sh
$ ./invdb.sh
```

確認是否符合預期
```
$ echo '.schema' | sqlite3 inventory.db
```

## git status

命令
```bash=
$ git status
```

結果：
```
On branch master

No commits yet

## Untracked，不做備份與檢視
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        invdb.sh
        inventory.db

nothing added to commit but untracked files present (use "git add" to track)
```

## 讓 git 追蹤檔案

```
$ git add invdb.sh
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   invdb.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory.db

```

## git 備份檔案

```
$ git commit 
	::
Add invdb.sh   <------------- 在最底下新增這行字串
```

檢查
```
$ git status
```

結果：
```bash=
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory.db     <-----------------只剩 inventory.db 未備份

nothing added to commit but untracked files present (use "git add" to track)

```

# git 狀態流程圖解1

![](https://i.imgur.com/COI1NHJ.png)

**名詞解釋**
- Untracked ，未被git追蹤的狀態
- Unmodified ，program被git備份到Repository後的狀態，在這個狀態下，只要有修改program，git都會追蹤，下`git status`命令，會看到狀態是Modified。
- Modified ，program在Unmodified狀態下，又被修改完的狀態
- Staged ，git的檔案暫存區，功能在於把檔案分開備份
- Repository ， git備份檔案的儲存區，位於專案目錄中的`.git`目錄裡面

**命令解釋**
- `git add <file name>` ，將Untracked或Modified狀態中的檔案，加到Staged（暫存區）
- `git commit <file name>` ，將Staged 中的檔案提交給Repository
	- 如果後面不加`<file name>`，則staged狀態的所有檔案都會備份

**流程解釋**

在git的工作目錄下，寫完了一支新的program，可以下`git status`檢查狀態，會發現當前的狀態屬於Untracked，代表未被git追蹤與備份。

當我們下命令`git add < file name >`之後，再下`git status`檢查狀態，可以看到program會被加到Staged（暫存區）。
> **注意，不管git的工作目錄下有幾個未被追蹤的檔案，就算只有一個，`git add`後面也要指定檔案，該檔案才會被git追蹤**

最後我們下命令`git commit < file name >`之後，git會把檔案，從Staged搬到Repository，此時檔案的狀態會回到Unmodified。

> `.git`目錄內容先不討論

問題：如果git commit後面不加檔名，git會把所有存在Staged的檔案都備份到Repository
但是commit後的名稱對應多個檔案要怎麼分辨？？？

答：執行`git log -p`，可以看到commit 中有修改那些內容


---


## 修改 `invdb.sh`

先刪除`inventory.db`

```
$ rm inventory.db
```

編輯`invdb.sh`

```
$ nano invdb.sh

DB_NAME="inventory.db"

create() {
  sqlite3 ${DB_NAME} <<EOF
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    isbn TEXT NOT NULL,
    quantity INTEGER NOT NULL
);

## 新增下面這行
insert into inventory (isbn,quantity) values ("1492090719","17");
EOF
}

create
```

執行

```
$ ./invdb.sh
```

檢查程式
```
$ echo 'select * from inventory' | sqlite3 inventory.db
1|1492090719|17
```

檢查git 狀態
```bash=
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   invdb.sh
```

將`invdb.sh`加入Staged

```
$ git add invdb.sh && git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   invdb.sh
```

將invdb.sh從Staged加到Repository

```
$ git commit -m 'Update create function'
[master cf2d9e1] Update create function
 1 file changed, 1 insertion(+)
```
> -m ，message，代表可以直接在命令後面幫備份檔命名。

最後檢查git狀態

```
$ git status
```



---

# git 狀態流程圖解2

![](https://i.imgur.com/ZSh63vs.png)

將Unmodified狀態的程式進行修改，因為此時git已經在追蹤該程式，所以修改後狀態會變成Modified，此時可以下`git add`命令，將檔案加入Staged（暫存區），再下`git commit`，將檔案完成備份，之後狀態會回到Unmodified。

## `invdb.sh` 新增功能

```
$ nano invdb.sh
刪除最後一行
create


新增以下程式碼
rmdb() {
  rm ${DB_NAME}
}

if [ "${1}" == "create" ]; then
  create
elif [ "${1}" == "rm"  ]; then
  rmdb
fi
```

:::spoiler 修改後的`invdb.sh`
```
#!/bin/bash

DB_NAME="inventory.db"

## 建立資料庫
create() {
  sqlite3 ${DB_NAME} <<EOF
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    isbn TEXT NOT NULL,
    quantity INTEGER NOT NULL
);

insert into inventory (isbn,quantity) values ("1492090719","17");
EOF
}

rmdb() {
  rm ${DB_NAME}
}

if [ "${1}" == "create" ]; then
  create
elif [ "${1}" == "rm"  ]; then
  rmdb
fi
```
:::

修改後，刪除inventory.db

```
$ rm inventory.db
```

執行程式並確認是否符合預期
```
$ ./invdb.sh create
$ echo 'select * from inventory' | sqlite3 inventory.db
1|1492090719|17

$ ./invdb.sh rm
$ ls -al
```

## 檢視與備份檔的差異

命令：

```
$ git diff <file name>
```

> 如果命令後面不加檔案名稱，會看到每個modified狀態的檔案與對應備份檔的差異。

結果：

```
...以上省略
-create
+rmdb() {
+  rm ${DB_NAME}
+}
+
+if [ "${1}" == "create" ]; then
+  create
+elif [ "${1}" == "rm"  ]; then
+  rmdb
+fi
...以下省略
```

# 跳過暫存區直接備份檔案

命令：

```
$ git commit -a <file name>
```
> 如果命令後面不指定檔案，git會直接將Staged的所有檔案通通備份

```
$ git commit -a -m "Add rmdb function"
[master 0383843] Add rmdb function
 1 file changed, 10 insertions(+), 1 deletion(-)
```

檢查git的狀態

```
$ git status
On branch master
nothing to commit, working tree clean
```

## git 狀態流程圖解3

![](https://i.imgur.com/QLWwrRn.png)

**流程解釋**

將Modified狀態的所有檔案，跳過暫存區，直接完成備份。

如果Staged裡已經存有上次`git add`的檔案，此時又修改程式，然後直接`git commit -a`，staged的檔案會存在嗎？

答：staged裡的檔案，經過修改後，會回到Modified的狀態，所以直接打`git commit -a`，一樣會跳過暫存區，將檔案與暫存區的所有檔案直接備份到Repository。


---


# 實作練習 1

請重構 `invdb.sh`
使用 case 語法替換 if/elif
新增執行 SQL 命令的功能

預期結果

```
$ ./invdb.sh create
$ ./invdb.sh "select * from inventory"
1|1492090719|17
```

完成重構後，請使用 git 備份!
Commit 內容為：`Refactor invdb.sh`


答案：

```bash=
#!/bin/bash

DB_NAME="inventory.db"

## 建立資料庫
create() {
  sqlite3 ${DB_NAME} <<EOF
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    isbn TEXT NOT NULL,
    quantity INTEGER NOT NULL
);

insert into inventory (isbn,quantity) values ("1492090719","17");
EOF
}

rmdb() {
  rm ${DB_NAME}
}

[ "${1}" == "" ] && echo "請輸入參數，謝謝！" && exit 1
case $1 in
  create)
                            create ;;
  rm)
                            rmdb ;;
  "select * from inventory")
                            echo "${1}" | sqlite3 ${DB_NAME} ;;
  *)
                            echo "請輸入（ create / rm / select * from inventory ）其中一個參數，謝謝！";;
esac
```


---


# git log

命令：

```
$ git log
```
功能：顯示commit後的資訊。

用法：按鍵盤的上下鍵移動，按Q離開。

:::spoiler 結果：

```bash=
commit fbd420d1652b3211c7fba65b9eb85b910064c879 (HEAD -> master)
Author: antony <antony@example.com>
Date:   Sat May 28 09:01:28 2022 +0000

    Refactor invdb.sh

commit 61432f393fb5f4da0b9ef7604619ad14bf30e1ac
Author: antony <antony@example.com>
Date:   Sat May 28 08:29:38 2022 +0000

    Add rmdb function

commit bb26a89b0d49e5976fb385222a5459acda9d97f3
Author: antony <antony@example.com>
Date:   Sat May 28 07:25:11 2022 +0000

    Update create function

commit 46a27e28c331b9fb1ad753c14a0f6febe2c9cb68
Author: antony <antony@example.com>
Date:   Sat May 28 04:16:52 2022 +0000

    test aaa
    inventory1

commit c7f5a8a64e2833b44030e8771b5e49ff0546df09
Author: antony <antony@example.com>
Date:   Sat May 28 04:07:56 2022 +0000

    echo "abc"

commit 9549e5f88f915d9f97c65182ee7da4c5b6c975c2
Author: antony <antony@example.com>
Date:   Sat May 28 03:49:44 2022 +0000

    Add invdb.sh

~
```

:::

加參數 `--oneline` ，縮短commit後產生的版本（取前七碼）、作者與時間
和 `--graph`，在同一個開發路線前面加符號。
```bash=
$ git log --oneline --graph

* fbd420d (HEAD -> master) Refactor invdb.sh
* 61432f3 Add rmdb function
* bb26a89 Update create function
* 46a27e2 test aaa inventory1
* c7f5a8a echo "abc"
* 9549e5f Add invdb.sh

...以下省略
```

<font color=red>**超級注意！關於commit後的訊息，不能隨便亂寫!**</font>

加參數：`--grep`，git內建搜尋commit的訊息

```
$ git log --grep Update

commit bb26a89b0d49e5976fb385222a5459acda9d97f3
Author: antony <antony@example.com>
Date:   Sat May 28 07:25:11 2022 +0000

    Update create function
	
...以下符號省略
```


---

# 撰寫API程式

[REST API 筆記連結](https://hackmd.io/@QI-AN/HkP73iavc)


```
$ nano inventory
#!/bin/bash

## 對於CGI而言以下兩行是必要的程式碼
echo "Content-type: application/json"
echo ""

## 主程式碼
DB_PATH="/root/bookstore-inventory/inventory.db"

sqlite3 ${DB_PATH} "select * from inventory"
```


執行程式並確認是否符合預期

```
$ chmod +x inventory
$ ./inventory
Content-type: application/json

1|1492090719|17
```
> 程式說明：這支程式等等會交由Httpd網站伺服器來執行，他使用的技術是CGI，可以讓網站伺服器執行Bash script。


## httpd CGI 測試

```
$ mkdir /opt/www/cgi-bin
```
> cgi-bin 是指定的，Httpd 網站伺服器只認這個目錄

拷貝我們的程式碼
```
$ cp inventory /opt/www/cgi-bin
```

curl檢查能不能成功
```
$ curl localhost/cgi-bin/inventory
1|1492090719|17
```

<font color=red>**為何程式名稱後面不加`.sh`，因為這樣就不符合REST API的要求**</font>
此時已經初步完成REST API程式了，剩回傳的json 格式




使用 git 備份 inventory

```
$ git add inventory
$ git commit -m 'Add API script "inventory"'
```



---

## 修改 `invdb.sh`

資料只有一筆太少，請修改 `invdb.sh` 新增多筆資料
請輸出以下資料
```
$ ./invdb.sh rm && ./invdb.sh create

$ ./invdb.sh "select * from inventory"
1|1492090719|17
2|9865024918|11
3|1492092304|16
4|9864760785|10
5|9865028042|12
```


答案：

```bash=
#!/bin/bash

DB_NAME="inventory.db"

## 建立資料庫
create() {
  sqlite3 ${DB_NAME} <<EOF
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    isbn TEXT NOT NULL,
    quantity INTEGER NOT NULL
);

insert into inventory (isbn,quantity) values ("1492090719","17");
insert into inventory (isbn,quantity) values ("9865024918","11");
insert into inventory (isbn,quantity) values ("1492092304","16");
insert into inventory (isbn,quantity) values ("9864760785","10");
insert into inventory (isbn,quantity) values ("9865028042","12");

EOF
}
...以下省略
```

## API 程式修改

修改 API 程式，請在最後一行 sqlite3 加入 -json 參數

```
$ nano inventory
	::
sqlite3 -json ${DB_PATH} "select * from inventory"
```

```
$ cp inventory /opt/www/cgi-bin
$ curl localhost/cgi-bin/inventory
[{"id":1,"isbn":"1492090719","quantity":17},
{"id":2,"isbn":"9865024918","quantity":11},
{"id":3,"isbn":"1492092304","quantity":16},
{"id":4,"isbn":"9864760785","quantity":10},
{"id":5,"isbn":"9865028042","quantity":12}]
```

# 實作練習 2

先刪除inventory.db
```
$ ./invdb.sh rm 
```

查看git狀態

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   invdb.sh
        modified:   inventory


no changes added to commit (use "git add" and/or "git commit -a")
```


備份 `invdb.sh`
commit 資訊：`Update invdb.sh, insert multiple record`

```
$ git commit invdb.sh
...以上省略
Update invdb.sh, insert multiple record
```

檢查git狀態
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   inventory
```

看git最後一筆log檔資訊

```
commit e1c3580841b9f06e786439d63c61830c577ac8e2 (HEAD -> master)
Author: antony <antony@example.com>
Date:   Sat May 28 12:25:21 2022 +0000

    Update invdb.sh, insert multiple record
```

## API 程式修改

```bash=
$ nano inventory
#!/bin/bash

echo "Content-type: application/json"
echo ""

DB_PATH="/root/bookstore-inventory/inventory.db"

saveIFS=$IFS
IFS='=&'
parm=($QUERY_STRING)
IFS=$saveIFS

if [ "${parm[0]}" == "id" ]; then
  sqlite3 -json ${DB_PATH} "select * from inventory where id = ${parm[1]}"
else
  sqlite3 -json ${DB_PATH} "select * from inventory"
fi
```

> QUERY_STRING 就是網址後面要搜尋的字串，例如:
> `localhost/cgi-bin/inventory?xxx=123&yyy=456&zzz=789`
> $QUERY_STRING="xxx=123&yyy=456&zzz=789"

> 在bash裡，有一個預設的變數，$IFS，它代表分隔符號，用來規定系統內的分隔符，如果在linux系統裡直接echo $IFS，可以看到是空白

> `parm=($QUERY_STRING)`，在bash裡面，`$QUERY_STRING`兩側加小括號，會被視為陣列處理，但是分隔符這裡已經變更為=&，在系統裡會認為是：`xxx 123 yyy 456 zzz 789`
> 舉例：`parm[0]=xxx` ; `parm[1]=123`

> 做完陣列以後，分隔符號會再變回預先儲存的saveIFS 

## API 程式測試

```
$ ./invdb.sh create
$ ./inventory

$ cp inventory /opt/www/cgi-bin
$ curl localhost/cgi-bin/inventory
[{"id":1,"isbn":"1492090719","quantity":17},
{"id":2,"isbn":"9865024918","quantity":11},
{"id":3,"isbn":"1492092304","quantity":16},
{"id":4,"isbn":"9864760785","quantity":10},
{"id":5,"isbn":"9865028042","quantity":12}]

$ curl localhost/cgi-bin/inventory?id=1
[{"id":1,"isbn":"1492090719","quantity":17}]

```

# 實作練習3

請完成 API 程式，預期結果如下

```
$ curl localhost/cgi-bin/inventory
[{"id":1,"isbn":"1492090719","quantity":17},
{"id":2,"isbn":"9865024918","quantity":11},
{"id":3,"isbn":"1492092304","quantity":16},
{"id":4,"isbn":"9864760785","quantity":10},
{"id":5,"isbn":"9865028042","quantity":12}]

$ curl localhost/cgi-bin/inventory?id=1
[{"id":1,"isbn":"1492090719","quantity":17}]

$ curl localhost/cgi-bin/inventory?isbn=9865028042
[{"id":5,"isbn":"9865028042","quantity":12}]
```

答：

```
$ nano inventory
#!/bin/bash

echo "Content-type: application/json"
echo ""

DB_PATH="/root/bookstore-inventory/inventory.db"

saveIFS=$IFS
IFS='=&'
parm=($QUERY_STRING)
IFS=$saveIFS

if [ "${parm[0]}" == "id" ]; then
  sqlite3 -json ${DB_PATH} "select * from inventory where id = ${parm[1]}"
elif [ "${parm[0]}" == "isbn" ];then
  sqlite3 -json ${DB_PATH} "select * from inventory where isbn = ${parm[1]}"
else
  sqlite3 -json ${DB_PATH} "select * from inventory"
fi
```

拷貝API

```
$ cp inventory /opt/www/cgi-bin
```

最後檢查

```
$ curl localhost/cgi-bin/inventory?isbn=9864760785
[{"id":4,"isbn":"9864760785","quantity":10}]
```

git備份

```
$ git commit -a -m 'Update API script "inventory"'
```



## 檢視 git log


命令：

```
$ git log --oneline --graph
```

結果：

```
7bc247 (HEAD -> master) Update API script "inventory"
* e1c3580 Update invdb.sh, insert multiple record
* fbd420d Refactor invdb.sh
* 61432f3 Add rmdb function
* bb26a89 Update create function
* 46a27e2 test aaa inventory1
* c7f5a8a echo "abc"
* 9549e5f Add invdb.sh
```

## git restore


如果不小心把程式給宰了，怎麼辦？
```
$ rm invdb.sh
$ ls -al
drwxr-xr-x    8 root     root          4096 May 26 16:35 .git
-rwxr-xr-x    1 root     root           488 May 26 16:25 inventory
```

檢查git 的狀態

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    invdb.sh

no changes added to commit (use "git add" and/or "git commit -a")
```
> 可以看到此時狀態為: deleted

還原我們的檔案
```
$ git restore invdb.sh
```

檢查
```
$ ls -al
drwxr-xr-x    8 root     root          4096 May 26 16:38 .git
-rwxr-xr-x    1 root     root           714 May 26 16:38 invdb.sh
-rwxr-xr-x    1 root     root           488 May 26 16:25 inventory
```

# 退出開發環境

離開Container

```
$ exit
```

複製 bookstore-inventory

```
~/wk$ docker cp dev:/root/bookstore-inventory .

$ ls -al bookstore-inventory
$ docker stop dev
$ docker ps -a
```
























###### tags: `CI/CD`