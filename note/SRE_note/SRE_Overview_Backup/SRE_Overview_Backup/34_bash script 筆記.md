# bash script 筆記

:::warning

:::spoiler 目錄

[TOC]

:::

# 初學

## bash script 開始下達指令
```
bigred@m1:~$ command [-option] parameter1 parameter2
                指令      選項        參數1       參數2
```
上述指令詳細說明如下：

1. **一行指令中第一個輸入的部分絕對是『指令(command)』或『可執行檔案(例如批次腳本,script)』**
2. command 為指令的名稱，例如變換工作目錄的指令為 cd 等等；
3. 中刮號[]並不存在於實際的指令中，而加入選項設定時，通常選項前會帶 - 號，例如 -h；有時候會使用選項的完整全名，則選項前帶有 -- 符號，例如 --help；
4. parameter1 parameter2.. 為依附在選項後面的參數，或者是 command 的參數；
5. 指令, 選項, 參數等這幾個咚咚中間以空格來區分，**不論空幾格 shell 都視為一格**。**所以空格是很重要的特殊字元！**；
6. 按下[Enter]按鍵後，該指令就立即執行。**[Enter]按鍵代表著一行指令的開始啟動**。
7. 指令太長的時候，可以使用反斜線 (\) 來跳脫[Enter]符號，使指令連續到下一行。注意！反斜線後就立刻接特殊字符，才能跳脫！
8. 其他：
    *  在 Linux 系統中，**英文大小寫字母是不一樣的**。舉例來說， cd 與 CD 並不同。
--- 
## 查看環境變數 PATH

- `echo $PATH`  ➡️  linux 輸入法
- `echo %PATH%` ➡️  windows 輸入法
- 環境變數PATH：
    - 記錄所有可執行命令所在的目錄路徑，因此若命令的儲存路徑沒有記錄在PATH中，則無法執行。
    - `$PATH` 告知 OS 哪裡會存放可執行的檔案，會依照先後順序找檔名，並且執行。若要執行的檔案不放在 $PATH 的目錄裡的話則需要使用相對/絕對路徑來執行


--- 


## Linux重要的幾個熱鍵

### [Tab]按鍵
[Tab]按鍵算是Linux的Bash shell最棒的功能之一了！他具有『命令補全』與『檔案補齊』的功能喔！ 重點是，可以避免我們打錯指令或檔案名稱呢！很棒吧！但是[Tab]按鍵在不同的地方輸入，會有不一樣的結果喔！ 我們舉下面的例子來說明。上一小節我們不是提到 cal 這個指令嗎？如果我在指令列輸入 ca 再按兩次 [tab] 按鍵， 會出現什麼訊息？
```
bigred@m1:~$ ca[tab][tab]   <==[tab]按鍵是緊接在 a 字母後面！
cal     caller  capsh   case    cat
# 上面的 [tab] 指的是『按下那個tab鍵』，不是輸入中括號內的tab
```
以ca為開頭的指令都被顯示出來了，如果輸入『ls -al ~/.bash』再加兩個[tab]會出現什麼？
```
[ dmtsai@study ~]$ ls -al ~/.bash[tab][tab]
.bash_history  .bash_logout   .bash_profile  .bashrc
```
該目錄下面所有以 .bash 為開頭的檔案名稱都會被顯示出來了，我們按[tab]按鍵的地方如果是在command(第一個輸入的資料)後面時，他就代表著 『命令補全』，如果是接在第二個字以後的，就會變成『檔案補齊』的功能！但是在某些特殊的指令底下，檔案補齊的功能可能會變成『參數/選項補齊』！ 我們同樣使用 date 這個指令來查一下：
```
[dmtsai@study ~]$ date --[tab][tab]  <==[tab]按鍵是緊接在 -- 後面！
--date        --help        --reference=  --rfc-3339=   --universal
--date=       --iso-8601    --rfc-2822    --set=        --version
# 系統會列出來 date 這個指令可以使用的選項有哪些喔～包括未來會用到的 --date 等項目
```
總結:
* [Tab] 接在一串指令的第一個字的後面，則為『命令補全』
* [Tab] 接在一串指令的第二個字以後時，則為『檔案補齊』
* 若安裝 bash-completion 軟體，則在某些指令後面使用 [tab] 按鍵時，可以進行『選項/參數的補齊』功能
**善用 [tab] 按鍵真的是個很好的習慣！可以避免掉很多輸入錯誤的機會！**

### [Ctrl]-c 按鍵
如果你在Linux底下輸入了錯誤的指令或參數，有的時候這個指令或程式會在系統底下『跑不停』這個時候怎麼辦？別擔心， 如果你想讓當前的程式『停掉』的話，可以輸入：[Ctrl]與c按鍵(先按著[Ctrl]不放，且再按下c按鍵，是組合按鍵)， 那就是中斷目前程式的按鍵！舉例來說，如果你輸入了『find /』這個指令時，系統會開始跑一些東西(先不要理會這個指令串的意義)，此時你給他按下 [Ctrl]-c 組合按鍵，嘿嘿！是否立刻發現這個指令串被終止
```
[bigred@m1 ~]$ find /
....(一堆東西都省略)....
# 此時螢幕會很花，你看不到命令提示字元的！直接按下[ctrl]-c即可！
[bigred@m1 ~]$ <==此時提示字元就會回來了！find程式就被中斷！
```

### [Ctrl]-e 按鍵

游標 (cursor) 移到行首

### [Ctrl]-a 按鍵

游標 (cursor) 移到行尾

--- 
## Linux文書編輯器： `nano`
nano的使用，可以直接加上檔名就能夠開啟一個舊檔或新檔
```
[dmtsai@study ~]$ nano text.txt
# 不管text.txt存不存在都沒有關係！存在就開啟舊檔，不存在就開啟新檔

  GNU nano 2.3.1                        File: text.txt                                 

   <==這個是游標所在處




                                  [ New File ]
^G Get Help   ^O WriteOut   ^R Read File  ^Y Prev Page  ^K Cut Text   ^C Cur Pos
^X Exit       ^J Justify    ^W Where Is   ^V Next Page  ^U UnCut Te   ^T To Spell
# 上面兩行是指令說明列，其中^代表的是[ctrl]的意思
```
如上所示，可以看到第一行反白的部分，那僅是在宣告nano的版本與檔名(File: text.txt)而已。 之後你會看到最底下的三行，分別是檔案的狀態(New File)與兩行指令說明列。指令說明列反白的部分就是組合鍵， 接的則是該組合鍵的功能。那個指數符號(^)代表的是鍵盤的[Ctrl]按鍵啦！底下先來說說比較重要的幾個組合按鍵：

[ctrl]-G：取得線上說明(help)
[ctrl]-X：離開nano軟體，若有修改過檔案會提示是否需要儲存
[ctrl]-O：儲存檔案，若有權限的話就能夠儲存檔案了
[ctrl]-R：從其他檔案讀入資料，可以將某個檔案的內容貼在本檔案中
[ctrl]-W：搜尋字串
[ctrl]-C：說明目前游標所在處的行數與列數等資訊
~~[ctrl]-_：可以直接輸入行號，讓游標快速移動到該行；~~   #m1沒有
~~[alt]-Y：校正語法功能開啟或關閉(按一下開、再按一下關)~~ #m1沒有
~~[alt]-M：可以支援滑鼠來移動游標的功能~~          #m1沒有


## 以上資料參考自[鳥哥的私房菜第四篇](https://linux.vbird.org/linux_basic/centos7/0160startlinux.php)





--- 
## Linux查看記憶體指令—`free`:
顯示系統內部實體記憶體及Swap的使用情況，初步會以KB為單位
改進單位轉換成Bytes，MB和GB，分別是-b，-m和-g，加上 -t 參數，會顯示實體記憶體加上 Swap 的合共記憶體:
```
g06220565@cloudshell:~ (peppy-amplifier-324003)$ free -g -t
              total        used        free      shared  buff/cache   available
Mem:             15           1          13           0           1          14
Swap:             0           0           0
Total:           15           1          13
```
加上 -s 參數會在特定秒數自動重新執行 free 指令, 例如下面會以 MB 為單位, 並會每 5 秒印出一次新資料:
```
g06220565@cloudshell:~$ free -gt -s 5
              total        used        free      shared  buff/cache   available
Mem:             15           1          13           0           1          13
Swap:             0           0           0
Total:           15           1          13

              total        used        free      shared  buff/cache   available
Mem:             15           1          13           0           1          13
Swap:             0           0           0
Total:           15           1          13
```
要終止執行按 Ctrl + C 。

--- 
mem：實體記憶體統計。
swap：硬碟上交換分割槽的使用情況。只有mem被當前程序實際佔用完,即沒有了buffers和cache時，才會使用到swap。

mem 行（第一行）資料說明：
total：實體記憶體總大小。
used：總計分配給快取（包含buffers 與cache ）使用的數量，但其中可能部分快取並未實際使用。
free：未被分配的記憶體。
shared：共享記憶體，一般系統不會用到。
buffers(緩衝區)：系統分配但未被使用的buffers 數量。
cached(緩存)：系統分配但未被使用的cache 數量。

補充:
緩衝區(buffers)是尚未被"寫入"磁盤的東西。
緩存(cached)是指已經從磁盤上"讀"出並存儲起來供以後使用的東西。

更詳細的解釋參考：Difference Between Buffer and Cache

Buffer：緩衝區，一個用於存儲速度不同步的設備或優先級不同的設備之間傳輸數據的區域。通過緩衝區，可以使進程之間的相互等待變少，從而使從速度慢的設備讀入數據時，速度快的設備的操作進程不發生間斷。

Cache：高速緩存，是位於CPU與主記憶體間的一種容量較小但速度很高的記憶體。由於CPU的速度遠高於主記憶體，CPU直接從記憶體中存取數據要等待一定時間週期，Cache中保存著CPU剛用過或迴圈使用的一部分數據，當CPU再次使用該部分數據時可從Cache中直接調用,這樣就減少了CPU的等待時間,提高了系統的效率。Cache又分為一級Cache(L1 Cache)和二級Cache(L2 Cache)，L 1 C ache集成在CPU內部，L 2 C ache早期一般是焊在主板上,現在也都集成在CPU內部，常見的容量有256KB或512KB L2 Cache。

Free中的buffer和cache（它們都是佔用記憶體）：
　　buffer : 作為buffer cache的記憶體，是塊設備的讀寫緩衝區
　　cache : 作為page cache的記憶體, 文件系統的cache

　　如果 cache 的值很大，說明cache住的文件數很多。如果頻繁訪問到的文件都能被cache住，那麼磁片的讀IO bi會非常小。

---

## bash script 練習
1. 請撰寫 mysys 程式, 執行功能如下 :

$ ./mysys
./mysys [add|del|start|stop|list] object

$ ./mysys add xx
add xx

$ ./mysys start
start 

$ ./mysys test
./mysys [add|del|start|stop|list] object

答案:
```
g06220565@cloudshell:~$ nano mysys
#!/bin/bash
case $1 in
add)
  echo $0
;;
del)
  echo $@
;;
start)
  echo $@
;;
stop)
  echo $@
;;
list)
  echo $@
;;
*)
  echo "./nysys [add|del|start|stop|list] odject"
;;

esac

g06220565@cloudshell:~$ ./mysys ## "."代表的是g06220565的家目錄
```
改裝
```
#!/bin/bash
case $1 in
add)
  echo -e "$2\n$2" | sudo adduser -s /bin/bash $2
;;
del)
  sudo deluser --remove-home $2
;;
start)
  echo "Oops"
;;
stop)
  echo "Oops"
;;
list)
  echo "Oops"
;;
*)
  echo "./nysys [add|del|start|stop|list] odject"
;;

esac
```


---

# 進階

## sed ( stream editor )

:::success

取代檔案的特定行數內容
```
$ sed -i "5cxxx" sigd.sh
```

`-i` ，代表要對檔案進行修改
`5c` ，第一個數字代表第幾行，後面的c，代表會直接將原本第五行的內容取代
`xxx` ，代表要更改的字串
`sighd.sh` ，要更改的檔案
:::

:::success

對檔案特定行數插入內容，如果該行有內容，會自動往下移一行，該行之後的內容也是

```
$ sed -i "5ixxx" sigd.sh
```

`-i` ，代表要對檔案進行修改
`5ixxx` ，第一個數字是指定要第幾行，後面的`i`則是會在第五行插入後面的xxx，如果原本的檔案第五行之後還有更多行內容，則全部往下移一行。


:::

[sed 參考文件 ( 初學 )](https://terryl.in/zh/linux-sed-command/)
[sed 參考文件 ( 原文 )](https://www.gnu.org/software/sed/manual/sed.html)

---

## `sigh.sh`

:::success

功能：自動設定 ip , gateway , dns 的 ip 位址 （ Alpine Linux  版本 ）
使用：dns 的 ip 透過參數給定， ip 與 gateway 執行程式後再給

```bash=
#!/bin/bash
read -p "ip / gateway" ans1 ans2

sudo tee /etc/network/interfaces<<EOF
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
      address $ans1
      gateway $ans2
EOF

if [ "$#" -ge "1" ];then
  echo "" | sudo tee /etc/resolv.conf 1>/dev/null
  echo "search localdomain" | sudo tee /etc/resolv.conf 1>/dev/null
    no=0
    for c in $@
      do
      abc[$no]=$c
      no=$((no+1))
    done

    no=0
    while((no<$#))
    do
      echo "nameserver ${abc[$no]}" | sudo tee -a /etc/resolv.conf
      no=$((no+1))
    done
else
  echo "請給 DNS 的參數，謝謝！"
fi
```

:::

## 變數處理

```bash=
name="John"
echo ${name}
echo ${name/J/j}    #=> "john" (substitution)
echo ${name:0:2}    #=> "Jo" (slicing)
echo ${name::2}     #=> "Jo" (slicing)
echo ${name::-1}    #=> "Joh" (slicing)
echo ${name:(-1)}   #=> "n" (slicing from right)
echo ${name:(-2):1} #=> "h" (slicing from right)
echo ${food:-Cake}  #=> $food or "Cake"
```

```bash=
STR="/path/to/foo.cpp"
echo ${STR%.cpp}    # /path/to/foo
echo ${STR%.cpp}.o  # /path/to/foo.o
echo ${STR%/*}      # /path/to

echo ${STR##*.}     # cpp (extension)
echo ${STR##*/}     # foo.cpp (basepath)

echo ${STR#*/}      # path/to/foo.cpp
echo ${STR##*/}     # foo.cpp

echo ${STR/foo/bar} # /path/to/bar.cpp
STR="Hello world"
echo ${STR:6:5}   # "world"
echo ${STR: -5:5}  # "world"
SRC="/path/to/foo.cpp"
BASE=${SRC##*/}   #=> "foo.cpp" (basepath)
DIR=${SRC%$BASE}  #=> "/path/to/" (dirpath)
```

[補充連結](https://devhints.io/bash#conditionals)

[Bash Hackers 連結](https://wiki.bash-hackers.org/syntax/pe)



## 可以埋進開機登入後的第一支程式

```bash=
#!/bin/bash
#從不解析名字的路由表，找目的地為本機的路由資訊
gw=$(route -n | grep -e "^0.0.0.0 ")

#宣告$gw的網卡資訊為環境變數
export GWIF=${gw##* }

#從ifconfig找本機的IP位址、網路遮罩、廣播位址
ips=$(ifconfig $GWIF | grep 'inet ')

#宣告本機IP為環境變數
export IP=$(echo $ips | cut -d' ' -f2 | cut -d':' -f2)

#宣告本機之Class C網段位址為環境變數
export NETID=${IP%.*}

#宣告本機的路由Gateway為環境變數
export GW=$(route -n | grep -e '^0.0.0.0' | tr -s \ - | cut -d ' ' -f2)

#將hostname寫入/tmp/sinfo
echo "[System]" > /tmp/sinfo
echo "Hostname : `hostname`" >> /tmp/sinfo

#將記憶體大小寫入/tmp/sinfo
m=$(free -mh | grep Mem: | tr -s ' ' | cut -d' ' -f2)
echo "Memory : ${m}M" >> /tmp/sinfo

#將CPU的型號及核心數寫入/tmp/sinfo
cname=$(cat /proc/cpuinfo | grep 'model name' | head -n 1 | cut -d ':' -f2)
cnumber=$(cat /proc/cpuinfo | grep 'model name' | wc -l)
echo "CPU : $cname (core: $cnumber)" >> /tmp/sinfo

#將硬碟大小資訊寫入/tmp/sinfo
m=$(df -h | grep /dev/sda)
ds=$(echo $m | cut -d ' ' -f2)
echo "Disk : $ds" >> /tmp/sinfo

echo "" >> /tmp/sinfo

#將IP位址、Gateway資訊寫入/tmp/sinfo
echo "[Network]" >> /tmp/sinfo
echo "IP : $IP" >> /tmp/sinfo
echo "Gateway : $GW" >> /tmp/sinfo

#將域名的名稱伺服器寫入/tmp/sinfo
cat /etc/resolv.conf | grep 'nameserver' | head -n 1 >> /tmp/sinfo

#ping外網，若有通則將"Internet OK"寫入/tmp/sinfo
/bin/ping -c 1 www.hinet.net
[ "$?" == "0" ] && echo "Internet OK" >> /tmp/sinfo
```

## Alpine 開機自動執行程式

```bash!
$ sudo nano /etc/local.d/*.start
```

- Alpine 開機時會自動執行在 /etc/local.d/ 目錄下 `*.start` 的檔案


```bash!
$ sudo chmod +x /etc/local.d/*.start
```

- 記得給權限

## 設定當前使用者執行 sudo 免密碼

```
$ vi init.sh
```

檔案內容 : 

```!
#!/bin/bash

# Debug mode
 set -x

# test sudo, run as root
if ! command -v sudo &>/dev/null; then
  echo "pls install sudo command: zypper in -y sudo, or you can run as root"
fi

# var
line_base_a=$(sudo cat /etc/sudoers | grep -n 'NOPASSWD: ALL')
line_base_b=$(cat /etc/group | grep -nw "^wheel")
user_name=$(id -un)

# check var
if [[ -z "${line_base_a%%:*}" ]] || [[ -z "${line_base_b%%:*}" ]];then
  echo "var has error" && exit 0
fi

# 設定 wheel 群組執行 sudo 命令時免密碼
sudo sed -i "${line_base_a%%:*}s|#||" /etc/sudoers

# 將當前使用者加入 wheel 群組
sudo sed -i "${line_base_b%%:*}s|$|${user_name}|" /etc/group
```


```!
$ curl -s https://raw.githubusercontent.com/braveantony/bash-script/bbc34c7729a9b4164ab9dcc282cb771cb9d94e83/init.sh | bash
```


###### tags: `Shell Script`
