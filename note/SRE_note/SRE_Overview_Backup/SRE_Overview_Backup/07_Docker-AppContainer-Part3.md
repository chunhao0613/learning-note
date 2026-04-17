# Docker-AppContainer-Part3

[TOC]



---

## process (程序) 運作

![](https://i.imgur.com/aSNHGbF.png)



- program 程式
人寫好的一堆程式碼，變成一個檔案，儲存在檔案系統中。
- process 程序
執行一個程式或指令，就會產生一個程序，這個程序一定在記憶體中執行，而在記憶體中程序會產生 PID 、執行者的權限參數 ( UID 、 GID )，和程式所需的程式碼以及相關資料

```bash!
$ ps -eo user,pid,ruid,euid,cmd | grep bigred
root       3408     0     0 /bin/login -f bigred
bigred     3418  1000  1000 -bash
bigred     3473  1000  1000 dialog --title  Cloud Native Trainer  --textbox /tmp/sinfo 24 85
root       3682     0     0 sshd: bigred [priv]
bigred     3686  1000  1000 sshd: bigred@pts/0
bigred     3687  1000  1000 -bash
bigred     3784  1000  1000 ps -eo user,pid,ruid,euid,cmd
bigred     3785  1000  1000 grep bigred
```

 -e 參數是代表輸出所有程序的資訊
 -o 參數則是用來指定輸出欄位用的，後面接著所有想要輸出的欄位名稱
 > 在 Alpine Linux 系統中 , 內定 ps 命令是由 busybox 執行, 如要執行全功能 ps 命令, 請執行 sudo apk add procps

1. Real UserID : For a process, Real UserId is simply the UserID of the user that has started it. It defines which files that this process has access to. 

2. Effective UserID : It is normally the same as Real UserID, but sometimes it is changed to enable a non-privileged user to access files that can only be accessed by a privileged user like root.

3. Saved UserID : It is used when a process is running with elevated privileges (generally root) needs to do some under-privileged work, this can be achieved by temporarily switching to a non-privileged account. 


以 go 語言為例 : 


```go=
$ echo  'package main
import (
    "fmt"
    "io/ioutil"
    "os/exec"
)
func main() {
    cmd := exec.Command("sleep", "60")
    cmd.Run()
    err := ioutil.WriteFile("/mulan.txt", []byte("Hello"), 0755)
    if err != nil {
        fmt.Printf("Unable to write file: %v\n", err)
    } else {
        fmt.Printf("/mulan.txt created\n")
    }
} ' > myfile.go
```
程式功能 : 

```
先睡 60 秒後，在根目錄上產生一個檔案，如果成功噴訊息，如果失敗也噴訊息
```

翻譯為 2 進位檔

```
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a myfile.go
```




```
取得 ALP 的 bigred 帳號 ID
$ echo $UID
1000

確認執行權限
$ dir myfile
-rwxrwxr-x 1 bigred bigred  2.0M  Jul 26 05:13 myfile

bigred 帳號沒有權限在 根目錄 (/) 產生檔案
$ ./myfile &
[1] 4698

$ ps -eo user,pid,ruid,euid,cmd | grep myfile
bigred     4698  1000  1000 ./myfile
bigred     4715  1000  1000 grep myfile
```

## Setuid

回到原先終端機, <font color=red>**一定要先設定 owner, 才可設定 setuid**</font>

```
$ sudo chown root myfile; sudo chmod 4755 myfile
```

檢查是否符合預期

```
$ dir myfile
-rwsr-xr-x 1 root bigred 2.0M Jul 26 06:57 myfile
```

推到背景執行

```
$ ./myfile  &
/mulan.txt created
```

查看 process 資訊

```
$ ps -eo user,pid,ruid,euid,cmd | grep myfile
root       4983  1000     0 ./myfile
bigred     4994  1000  1000 grep myfile
```

檢視 系統中有多少具有 setuid 功能的 命令

```bash=
$ sudo find / -user root -perm -4000 2>/dev/null | grep -E '^/bin|^/usr/bin'
/usr/bin/sudo
/bin/mount
/bin/bbsuid
/bin/umount
/bin/ping
/bin/fusermount
```

查看 /bin/ping

```
bigred@alp:~/setuid$ ls -al /bin/ping
-rwsr-xr-x 1 root root 64120 Apr 16 06:51 /bin/ping
```

<font color=red>**setuid 的檔案容易被 hacker 鎖定，故這種檔案越少越好。**</font>

下面這隻程式 5 顆星 攻擊首要目標，因為只要攻陷，所有使用者的帳號密碼都有啦，目前只能透過演算法，檢查 2 進位碼有沒有被更改。

```bash=
$ which passwd
/usr/bin/passwd

$ ls -al /usr/bin/passwd
lrwxrwxrwx 1 root root 11 Jul  5 10:01 /usr/bin/passwd -> /bin/bbsuid

$ ls -al /bin/bbsuid
---s--x--x 1 root root 14144 Jun 12 00:59 /bin/bbsuid
```

## 指定 Container 執行的帳號

```bash=
$ docker run --rm -it -d --name u1 quay.io/cloudwalker/alpine sleep infinite
776c951a8670186e8a01a72e555beefe03cfa0604badfe8177b3717cbab567fb

$ ps -eo user,pid,ruid,euid,cmd | grep "sleep infinity"
bigred     7686  1000  1000 grep sleep infinity

$ docker exec u1 whoami
root

$ docker stop u1
u1
```



```!
$ docker run --rm -it -d --name u1 --user bigred quay.io/cloudwalker/alpine
5e3d1ac1776ad3215289fc26a13ea76e0065c11fb5b11d908f530e6aa80f68a0
docker: Error response from daemon: unable to find user bigred: no matching entries in passwd file.

--user bigred ，表示在 Container 裡面要指定 bigred 使用者
```

指定用 UID 1001 帳號執行以下 Container

```
$ docker run --rm -it -d --name u1 --user 1001 quay.io/cloudwalker/alpine
```

檢查 docker host 主機的 uid, pid, ruid, euid, cmd

```
$ ps -eo user,pid,ruid,euid,cmd | grep "/bin/sh$"
1001       5925  1001  1001 /bin/sh
```

Container 裡面雖然沒有 uid = 1001 的帳號名稱，但在建立 Container 時，允許這樣透過沒有對應帳號名稱的 uid 來建立 Container，但注意這位使用者可能無法執行系統命令。

```
$ docker exec u1 whoami
Error: No such container: u1
```

停止 Container

$ docker stop u1


---

## Dockerfile with a defined user


```bash=
# 編輯 image 內容
$ echo 'FROM quay.io/cloudwalker/alpine
RUN apk update && apk add tree nano
RUN addgroup appgroup
RUN adduser -u 1002 -G appgroup -D appuser
USER appuser
ENTRYPOINT ["sleep", "infinity"]' > myuser

# build image
$ docker build -t myuser  -  < myuser

# run Container
$ docker run --rm --name u1 -d myuser

$ ps -eo user,pid,ruid,euid,cmd | grep "sleep infinity"
1002       8129  1002  1002 sleep infinity
bigred      8180  1000  1000 grep sleep infinity

$ docker exec -it u1 sh
/ $ rm -rf /* &>/dev/null; echo $?
1
/ $ whoami
appuser
/ $ tree /dev | wc -l
19
/ $  rm -rf /dev &>/dev/null; echo $?
1
/ $ exit

$ docker stop u1; docker rmi myuser
```


---


## Container Volume Permission

```bash=
$ id -u bigred; id -g bigred
1000
1000

$ mkdir ~/zzz; ls -lsd ~/zzz
4 drwxr-sr-x 2 bigred bigred 4096 Jul 10 10:42 /home/bigred/zzz

$ docker run --rm  -v ~/zzz:/opt --name u1 -d myuser
c3e9b054e02a997be0a938eb0f9b6aeefd141ba58c515392fb3bf307712734e9

$ docker exec u1 ls -al /opt
total 8
drwxr-sr-x    2 1000     appgroup      4096 Jul 10 02:42 .
drwxr-xr-x    1 root     root          4096 Jul 10 02:43 ..

$ docker exec u1 cat /etc/passwd | grep appuser
appuser:x:1002:1000:Linux User,,,:/home/appuser:/bin/ash

$ docker exec u1 cat /etc/group | grep appuser
appgroup:x:1000:appuser

$ docker exec u1 id
uid=1002(appuser) gid=1000(appgroup) groups=1000(appgroup)
```


<font color=red>**Container 中的 /opt 權限是套用 Docker Host ~/zzz 目錄權限，Container 只會抓 uid 與 gid 來套用，uid 找不到對應的使用者帳號，只好顯示代號，而 gid 有找到對應的 group 名稱，則顯示 appgroup**</font>



```bash=
$ docker exec -it u1 sh
/ $ whoami
appuser
/ $ mkdir /opt/vvv
mkdir: cannot create directory '/opt/vvv': Permission denied
/ $ exit

$ chmod -R 777 ~/zzz

$ docker exec u1 mkdir /opt/vvv

$ ls -al ~/zzz
total 12
drwxrwsrwx  3 bigred bigred 4096 Jul  8 21:21 .
drwxr-sr-x 11 bigred bigred 4096 Jul  8 21:00 ..
drwxr-sr-x  2   1002 bigred 4096 Jul  8 21:21 vvv
```

如果在 Container 中建立目錄 /opt/vvv 到剛剛掛載在 Docker Host 主機上的 /opt 目錄，則以 Container 裡的那位 User 的 Uid 及 gid 為主


---


# Linux System Call & Seccomp 

## Linux Process 運作架構

![](https://i.imgur.com/LWJSdWG.png)


- Program 放在檔案系統時，
- User program 被執行時，一定會呼叫 system library
- Seccomp 管控 User program 可用那些 system library
- 當 程式進入 kernel Mode 後，會由 CGroup 、 Capabilities 和 SElinux/AppArmor 來控管 process

:::spoiler 提問 :　以上這些與 Container 有何關係 ?

Container 會共用 Host OS Kernel, 所以可以透過 CGroup 、 Capabilities 和 SElinux/AppArmor 來管控 Container run 的 process ，來避免單一 Container 影響到其他一起共用共用 Host OS Kernel 的 Container。

:::

## 檢視 mkdir System Call


安裝命令 : 
```
$ sudo apk add strace
```


mkdir 會用到 3 個 System Call
```bash=
$ rm -r /tmp/x; strace -qcf -e trace=file mkdir /tmp/x
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         8         3 open
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           mkdir
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000           0        10         3 total

$ rm -r /tmp/x; strace -e trace=file mkdir /tmp/x
execve("/bin/mkdir", ["mkdir", "/tmp/x"], 0x7ffe34279858 /* 22 vars */) = 0
open("/etc/ld-musl-x86_64.path", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = -1 ENOENT (No such file or directory)
open("/lib/libacl.so.1", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 3
open("/lib/libattr.so.1", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 3
open("/lib/libgmp.so.10", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = -1 ENOENT (No such file or directory)
open("/usr/local/lib/libgmp.so.10", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = -1 ENOENT (No such file or directory)
open("/usr/lib/libgmp.so.10", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 3
open("/lib/libutmps.so.0.1", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 3
open("/lib/libskarnet.so.2.11", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 3
mkdir("/tmp/x", 0777)                   = 0
+++ exited with 0 +++

```

## 認識 Seccomp

Seccomp (short for security computing mode) is a useful feature provided by the Linux kernel since 2.6.12 and is used to control the syscalls made by a process. Seccomp has been implemented by numerous projects such as Docker, Android, OpenSSH and Firefox to name a few.

1. Searchable Linux Syscall Table for x86 and x86_64
https://filippo.io/linux-syscall-table/


## Seccomp 實作範例

建立工作目錄

```
$ cd; mkdir syscall; cd syscall
```

編輯 go 語言程式

```go=
$ nano main.go 
package main

import (
  "fmt"
  "syscall"
)

func main() {
    var syscalls = []string{
     "rt_sigaction", "mkdirat", "clone", "mmap", "readlinkat", "futex",
     "rt_sigprocmask","exit_group","mprotect", "write", "sigaltstack", "gettid", "read",
     "open", "close", "fstat", "munmap","brk", "access", "execve", "getrlimit",
     "arch_prctl", "sched_getaffinity", "set_tid_address", "set_robust_list"}
    whiteList(syscalls)

    err := syscall.Mkdir("/tmp/moo", 0755)
    if err != nil {
        panic(err)
    } else {
        fmt.Printf("I just created a file\n")
    }
}

```


```bash=
$ wget http://www.oc99.org/ctn/whitelist.go

# 安裝 alpine linux 要用的 seccomp
$ sudo apk add libseccomp-dev
(1/2) Installing linux-headers (5.16.7-r1)
(2/2) Installing libseccomp-dev (2.5.2-r1)
OK: 1053 MiB in 255 packages

$ go mod init mygo

# 安裝 Golang 所需的 Seccomp 套件，他會去呼叫  alpine linux 要用的 seccomp
$ go get github.com/seccomp/libseccomp-golang
go: downloading github.com/seccomp/libseccomp-golang v0.9.1
go: added github.com/seccomp/libseccomp-golang v0.9.1

$ go build -o test

$ ./test
...........
I just created a file

$ ls -al /tmp | grep moo
drwxr-xr-x  2 bigred bigred 4096 Jul 21 12:51 moo

```


```go=
$ nano main.go
package main

import (
  "fmt"
  "syscall"
  "os"
)

func main() {
    ........
    whiteList(syscalls)

    err := syscall.Mkdir("/tmp/moo", 0755)
    if err != nil {
        panic(err)
    } else {
        fmt.Printf("I just created a file\n")
    }

    fmt.Printf("pid: %d\n", os.Getpid())
}
```

```bash=
$ go build -o test

$ ./test
[+] Whitelisting: rt_sigaction
[+] Whitelisting: mkdirat
[+] Whitelisting: clone
[+] Whitelisting: mmap
[+] Whitelisting: readlinkat
[+] Whitelisting: futex
[+] Whitelisting: rt_sigprocmask
[+] Whitelisting: exit_group
[+] Whitelisting: mprotect
[+] Whitelisting: write
[+] Whitelisting: sigaltstack
[+] Whitelisting: gettid
[+] Whitelisting: read
[+] Whitelisting: open
[+] Whitelisting: close
[+] Whitelisting: fstat
[+] Whitelisting: munmap
[+] Whitelisting: brk
[+] Whitelisting: access
[+] Whitelisting: execve
[+] Whitelisting: getrlimit
[+] Whitelisting: arch_prctl
[+] Whitelisting: sched_getaffinity
[+] Whitelisting: set_tid_address
[+] Whitelisting: set_robust_list
I just created a file
pid: -1
```


```go=
$ nano main.go
package main

import (
  "fmt"
  "syscall"
  "os"
)

func main() {
    var syscalls = []string{
     "rt_sigaction", "mkdirat", "clone", "mmap", "readlinkat", "futex",
     "rt_sigprocmask","exit_group","mprotect", "write", "sigaltstack", "gettid", "read",
     "open", "close", "fstat", "munmap","brk", "access", "execve", "getrlimit","getpid",
     "arch_prctl", "sched_getaffinity", "set_tid_address", "set_robust_list"}
    whiteList(syscalls)

    err := syscall.Mkdir("/tmp/moo", 0755)
    if err != nil {
        panic(err)
    } else {
        fmt.Printf("I just created a file\n")
    }

    fmt.Printf("pid: %d\n",os.Getpid())
}
```

```bash=
$ go build -o test; rm -r /tmp/moo

$ ./test
[+] Whitelisting: rt_sigaction
[+] Whitelisting: mkdirat
[+] Whitelisting: clone
[+] Whitelisting: mmap
[+] Whitelisting: readlinkat
[+] Whitelisting: futex
[+] Whitelisting: rt_sigprocmask
[+] Whitelisting: exit_group
[+] Whitelisting: mprotect
[+] Whitelisting: write
[+] Whitelisting: sigaltstack
[+] Whitelisting: gettid
[+] Whitelisting: read
[+] Whitelisting: open
[+] Whitelisting: close
[+] Whitelisting: fstat
[+] Whitelisting: munmap
[+] Whitelisting: brk
[+] Whitelisting: access
[+] Whitelisting: execve
[+] Whitelisting: getrlimit
[+] Whitelisting: getpid
[+] Whitelisting: arch_prctl
[+] Whitelisting: sched_getaffinity
[+] Whitelisting: set_tid_address
[+] Whitelisting: set_robust_list
I just created a file
pid: 19923
```

## Docker Seccomp 實作範例


建立工作目錄 : 

```
$ mkdir ~/seccomp; cd ~/seccomp
```

編輯 chmod.json

```bash=
$ nano chmod.json
{
    "defaultAction": "SCMP_ACT_ALLOW",
    "architectures": [
        "SCMP_ARCH_X86_64",
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
    ],
    "syscalls": [
        {
            "name": "chmod",
            "action": "SCMP_ACT_ERRNO",
            "args": []
        },
        {
            "name": "chown",
            "action": "SCMP_ACT_ERRNO",
            "args": []
        },
        {
            "name": "chown32",
            "action": "SCMP_ACT_ERRNO",
            "args": []
        }
    ]
}

```

docker run Container 

```!
即使是 root ，也會被阻擋
$ docker run --rm -it --security-opt seccomp=chmod.json quay.io/cloudwalker/alpine chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted

$ docker run --rm -it quay.io/cloudwalker/alpine chmod 400 /etc/hosts
```

[Docker 內定的 Secomp Profile 連結](
https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**如果企業內部有規定哪個 syscall 要拿掉，可以拷貝上面連結的內容，再參考上面的作法。**


---


## Linux Capabilities & Privilege Escalation

```bash=
$ grep Cap /proc/$$/status
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 000001ffffffffff
CapAmb: 0000000000000000
```

`$$` 指的程式為 `/bin/bash`，執行的使用者為 bigred

```!
$ capsh --decode=0000001fffffffff
0x0000001fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend
```

##  設定 Capabilities

```go=
$ echo 'package main
import (
    "syscall"
    "log"
    "fmt"
)
func main() {
    fmt.Print("Press any key to continue ")
    var input string
    fmt.Scanln(&input)

    err := syscall.Reboot(syscall.LINUX_REBOOT_CMD_POWER_OFF)
    if err != nil {
       log.Printf("power off failed: %v", err)
    }
}' > mypoff.go
```

Compile mypoff.go
```
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a mypoff.go
```

執行 mypoff

```
./mypoff
Press any key to continue
```

在第二個 Bigred 終端機執行以下命令
```
$ pidof mypoff
22413

$ grep Cap /proc/22413/status
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000  ---> 重點是這行，要有數字程式才能跑
CapBnd: 000001ffffffffff
CapAmb: 0000000000000000
```

再回去第一個終端機

```bash=
$ ./mypoff
Press any key to continue (按個按鍵)
2022/07/10 13:47:52 power off failed: operation not permitted

$ sudo setcap cap_sys_boot+ep /home/bigred/seccomp/mypoff

$ getcap /home/bigred/seccomp/mypoff
```


在第二個 Bigred 終端機執行以下命令

```bash=
$ pidof mypoff
22995

$ grep Cap /proc/22995/status
CapInh: 0000000000000000
CapPrm: 0000000000400000
CapEff: 0000000000400000
CapBnd: 000001ffffffffff
CapAmb: 0000000000000000
```

## Linux Capabilities - cap_setuid 

```bash=
$ echo -e "ybean\nybean" | sudo adduser -s /bin/sh -h /home/ybean ybean
Changing password for ybean
New password:
Bad password: too short
Retype password:
passwd: password for ybean changed by root

# 設定 rbean 家目錄中的 python3 命令檔, 具有 Capabilities setuid 權限
# 代表 python3 所執行的 程式可設定 setuid 功能
$ sudo cp /usr/bin/python3  /home/ybean 

$ sudo setcap  cap_setuid+ep  /home/ybean/python3

$ ls -al  /home/ybean/python3
-rwxr-xr-x 1 root root 13976 Jul 21 14:15 /home/ybean/python3

$ exit
```

## Linux Privilege Escalation

```bash=
$ ssh ybean@<ALP.Podman IP>
ybean@<ALP.Docker IP>s password: ybean

ybean@alp:~$ ./python3 -c 'import os,time;os.setuid(1002);time.sleep(120)'

# ybean 的 UID 是 1002, 上面命令透過 python3 的 setuid Capabilities 動態改變 python3 自己的 UID 為 ybean

ybean@alp:~$ exit

$ ssh bigred@<ALP.Podman IP>
bigred@<ALP.Docker IP>s password: bigred

$ ps -eo user,pid,ruid,euid,cmd | grep "python"
ybean      5254  1002  1002 ./python3 -c import os,time;os.setuid(1002);time.sleep(120)
bigred     5258  1000  1000 grep python

# 開啟另一個終端機, 使用 ybean 登入
$ ssh ybean@<ALP.Docker IP>
ybean@<ALP.Docker IP>s password: ybean

# 列出有設定 Linux capabilities 的所有命令
$ getcap -r / 2>/dev/null
/usr/sbin/fping cap_net_raw=ep
/home/bigred/seccomp/mypoff cap_sys_boot=ep
/home/ybean/python3 cap_setuid=ep

# 由以下命令得知 python3 命令檔並沒有設定 setuid 功能
$ ls -al python3
-rwxr-xr-x 1 root root 13976 Jul 28 04:56 python3

#在以下 python3 命令所執行的 程式可執行 os.setuid(0) 這行命令, 提升 /bin/bash 這命令為 root 權限
ybean@alp:~$ ./python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'

root@alp:~$ id
uid=0(root) gid=1002(ybean) groups=1002(ybean)

root@alp:~$ ps aux | grep '/bin/bash'
root      14981  0.0  0.2   5844  4808 pts/0    S    23:27   0:00 ./python3 -c import os;os.setuid(0);os.system("/bin/bash")
root      14982  0.0  0.1   2584  2288 pts/0    S    23:27   0:00 /bin/bash
root      15004  0.0  0.0   1592   852 pts/0    S+   23:28   0:00 grep /bin/bash

# 上面的 ./python3 -c import os;os.setuid(0);os.system("/bin/bash") 命令的 UID 也被改變

root@alp:~$ exit
```


## Docker Capabilities 

```bash
$ docker run --rm -it quay.io/cloudwalker/alpine
/ # apk add libcap
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/community/x86_64/APKINDEX.tar.gz
(1/1) Installing libcap (2.46-r0)
Executing busybox-1.32.1-r6.trigger
OK: 6 MiB in 15 packages
```

```!
/ # capsh --print
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep    
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Ambient set =
Current IAB: !cap_dac_read_search,!cap_linux_immutable,!cap_net_broadcast,!cap_net_admin,!cap_ipc_lock,!cap_ipc_owner,!cap_sys_module,!cap_sys_rawio,!cap_sys_ptrace,!cap_sys_pacct,!cap_sys_admin,!cap_sys_boot,!cap_sys_nice,!cap_sys_resource,!cap_sys_time,!cap_sys_tty_config,!cap_lease,!cap_audit_control,!cap_mac_override,!cap_mac_admin,!cap_syslog,!cap_wake_alarm,!cap_block_suspend,!cap_audit_read,!cap_perfmon,!cap_bpf,!cap_checkpoint_restore
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=0(root) euid=0(root)
gid=0(root)
groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
Guessed mode: UNCERTAIN (0)
```

 Current: (docker 內定開啟的 Capabilities)
 Docker 內定無開啟可以修改時間的 Capabilities


```bash=
$ docker build -t capimg capimg/

$ docker run --rm -u 1000 capimg capsh --print

$ docker run --rm capimg chown nobody /

$ docker run --rm -it --cap-drop ALL --cap-add CHOWN capimg chown nobody /

$ docker run --rm -it --cap-drop CHOWN capimg chown nobody /
chown: /: Operation not permitted

$ docker run --rm -it --cap-add chown -u nobody capimg chown nobody /
chown: /: Operation not permitted

The above command fails because Docker does not yet support adding capabilities to non-root users.
```

Container 用 root 跑，會有內定的 Capabilities ，但如果用一般使用者則不給加 Capabilities 



---


# Google gVisor

## 認識 gVisor

gVisor is a light weight <font color=red>**user-space kernel**</font>, written in Go, that implements a substantial portion of the Linux system surface. By implementing Linux system surface,it provides isolation between host and application.Also it includes an Open Container Initiative (OCI) runtime called <font color=red>**runsc**</font> so that isolation boundary between the application and the host kernel is maintained.

It intercepts all application system calls and acts as the guest kernel, without the need for translation through virtualized hardware. Also, gVisor does not simply redirect application system calls through to the host kernel. Instead, gVisor implements most kernel primitives (like signals, file systems, futexes, pipes, mm, etc.) and has complete system call handlers built on top of these primitives.

![](https://i.imgur.com/LBaGJFA.png)

- 透過 runsc 命令 run 的 Container ，會多一層 user-space kernel
- user-space kernel 模擬 Container 會呼叫的 system call，與虛擬化技術不同，他只模擬約 50 or 60 % 的 kernel，所以啟動 Container 的速度會比傳統的 Container 慢


## 安裝 gVisor

```bash!
$ ARCH=$(uname -m); URL=https://storage.googleapis.com/gvisor/releases/release/latest/${ARCH}

$ wget ${URL}/runsc ${URL}/runsc.sha512 ${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512

# 檢核 runsc 及 containerd-shim-runsc-v1 這二個執行檔
$ sha512sum -c runsc.sha512 -c containerd-shim-runsc-v1.sha512
runsc: OK
containerd-shim-runsc-v1: OK

$ rm -f *.sha512

# 因為在 PATH 環境變數裡面有 /usr/local/bin 這個目錄
# 所以把 runsc 和 containerd-shim-runsc-v1 放到這個目錄下
# 我們就可以直接 run 這兩支程式
$ chmod a+rx runsc containerd-shim-runsc-v1; sudo mv runsc containerd-shim-runsc-v1 /usr/local/bin

$ echo $PATH
/home/bigred/bin:/home/bigred/vmalpdt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

## 設定 gVisor

```bash=
$ sudo nano /etc/docker/daemon.json
{
   "runtimes": {
      "runsc": {
         "path": "/usr/local/bin/runsc"
      }
   }
}

$ sudo reboot

在 Windows 系統的 CMD 視窗, 執行以下命令
$ ssh bigred@<ALP.Docker IP>
```

> 注意，上面的設定是讓 docker 可以透過 runsc run Container ，但 docker 預設的還是 runc ，所以執行 `docker run` 時，要加參數` --runtime=runsc`

## 執行 gVisor

```bash!
$ docker run --rm --runtime=runsc -d -p 8080:80 --name n1 quay.io/cloudwalker/nginx

$ curl -I http://localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.17.10
Date: Sat, 21 May 2022 11:50:28 GMT
........

$ ps aux | grep runsc
root       4359  0.0  0.9 736384 18972 ?  Sl 21:53  0:00 runsc-gofer --root=/var/run/docker/runtime-ru ..........

$ $ docker stop n1
```

注意，透過 runsc 產生出來的一台 Container ，會有一個 runsc-gofer 以及 一個 runsc-sandbox 這兩支 daemon 去罩這台 Container，所以兩台 Container 就會有 4 個 daemon ，要安全就會需要硬體上的支援。

:::spoiler 提問 : 有無比 gVisor 功能更完整的專案?
如果完全模擬 kernel ，不就跟虛擬化技術一樣?
使用 [katacontainers 這個專案](https://katacontainers.io/)，他完全模擬 系統的 kernel，給一台 Container 一個專屬的 kernel，會動用到 QEMU 和 KVM 這兩項技術，這種 Container 在標準開機程序裡面，不會有 Boot Loader ，所以開機速度比 VM 快，又有屬於自己的 kernel 可以玩耍。
:::


###### tags: `系統工程`