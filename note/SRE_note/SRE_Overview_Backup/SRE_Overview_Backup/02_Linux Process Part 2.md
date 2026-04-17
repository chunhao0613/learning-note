# Linux Process Part2

## 程序執行權限

程序與程式 ( process & program )

![](https://i.imgur.com/DIOvX41.png)

- 當我們在系統裡面執行程式，電腦會把程式載入記憶體，而這時候記憶體裡面會有程式的 PID 和執行者的權限屬性（UID 或 GID），還有程序的程式碼（machine code）和相關資料 (程式在執行時可能修改到的檔案、存取的硬體資源...等)
    - `PID` = Process identify
    - `UID` = User identify
- 程式在執行時所需要的資源(檔案目錄、網路、CPU、記憶體...等)，會以執行者的身分 ( UID ) 取得


Linux 程序執行權限

```bash!
# 建立工作目錄
$ cd; mkdir setuid; cd setuid

# 編輯 GO 語言的程式
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

$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a myfile.go

# 檢視執行檔有無編譯成功
$ dir myfile
-rwxrwxr-x 1 bigred bigred  2.0M  Jul 26 05:13 myfile

# -d ，列出目錄本身的資訊，而不是目錄下的內容
# 根目錄權限，只有擁有者可以讀取、編輯和進入，群組與其他人只能讀取和進入
$ ls -ld /
drwxr-xr-x 23 root root 4096 Jul 22 20:16 /

# bigred 帳號沒有權限在 根目錄 (/) 產生檔案
$ ./myfile
Unable to write file: open /mulan.txt: permission denied

# 開啟一個新的終端機, 執行以下命令
# 選擇要看 user,pid,ruid,euid,cmd 這幾個欄位的資訊
$ ps -eo user,pid,ruid,euid,cmd | grep myfile
bigred    50775  1000  1000  ./myfile
bigred     50825  1000   1000   grep myfile
```

### What is difference between RUID and EUID

- **Real UserID** ，Real UserId 只是啟動它的用戶的 UserID。
- **Effective UserID** ，真正執行程式的 UID ，它通常與 Real UserID 相同
- [詳細介紹文章連結](https://www.geeksforgeeks.org/real-effective-and-saved-userid-in-linux/)



## Setuid


Linux 檔案進階設定 - setuid


```bash!
# 回到原先終端機, 一定要先設定 owner, 才可設定 setuid
$ sudo chown root myfile; sudo chmod 4755 myfile

$ ls -al myfile
-rwsr-xr-x 1 root bigred 2.0M Jul 26 06:57 myfile

$ ./myfile 
/mulan.txt created

# 在新啟動的終端機, 執行以下命令
# 可以看到 euid 變成 root 的 UID
$ ps -eo user,pid,ruid,euid,cmd | grep myfile
root       50886  1000     0  ./myfile  
bigred    50898   1000  1000 grep myfile 

# setuid 無法套用到 shell script   
$ nano mkmoo.sh
#!/bin/bash
mkdir /moo

$ sudo chown root:root mkmoo.sh
$ sudo chmod 4755 mkmoo.sh

$ ls -al mkmoo.sh
-rwsr-xr-x 1 root root 25 Sep 27 17:33 mkmoo.sh

$ ./mkmoo.sh
mkdir: cannot create directory ‘/moo’: Permission denied

# 檢視 系統中有多少具有 setuid 功能的 命令
$ sudo find / -user root -perm -4000 2>/dev/null | grep -E '^/bin|^/usr/bin'
/usr/bin/sudo
/bin/mount
/bin/bbsuid
/bin/umount
/bin/ping
/bin/fusermount
```

- `setuid` : 使用者在執行程式時，可以讓檔案擁有者代為執行檔案，因此在查看 EUID 時，會顯示出檔案擁有者的 UID 。
- 目前所有的 Linux 作業系統，擁有 setuid 的功能應越少越好，Alpine 3.16.1 的版本目前剩 6 個


由以下命令得知 /etc/passwd 只能 root 讀寫, 為何 rbean 能自行重設密碼 ?

```bash!
$ sudo ls -al /etc/shadow
-rw-r----- 1 root shadow 1038 Aug  8 14:01 /etc/shadow
```





###### tags: `系統工程`