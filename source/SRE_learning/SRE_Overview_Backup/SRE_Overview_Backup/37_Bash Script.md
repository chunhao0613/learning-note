# BASH SCRIPT
`set -euo pipefail ##加在程式最上面，如果程式有錯他就會直接停止`


:::spoiler 用case創帳號並自動給密碼且已經輸入，刪除帳號，看使用者清單,並且alpine跟ubuntu都能使用


```bash=
#!/bin/bash
id=$(cat /etc/os-release | grep -w ID | cut -d '=' -f2 )
[ "$1" = '' ] || [ "$2" = '' ] && echo '請輸入指令' && exit 1
case $1 in
 create)
    grep '^[[:digit:]]*$' <<< $2 &>/dev/null && echo "阿就不能全數字,打頭喔！" && exit 1
    if [ "$id" == "ubuntu" ]
    then
       cat /etc/passwd | grep -w "^$2" &>/dev/null
       if [ $? == 0 ]
       then
           echo "已有$2帳號"
       else
           sudo useradd -m -s /bin/bash "$2" &>/dev/null
           p=$(< /dev/urandom tr -dc A-Za-z0-9 | head -c6)
           echo -e "$p\n$p" | sudo passwd "$2" &>/dev/null
           echo "$2使用者,密碼$p"
       fi
    elif [ "$id" == "alpine" ]
    then
        cat /etc/passwd | grep -w "^$2" &>/dev/null
        if [ $? == 0 ]
        then
           echo "已有$2帳號"
        else
           p=$(< /dev/urandom tr -dc A-Za-z0-9 | head -c6)
           echo -e "$p\n$p" | sudo adduser -s /bin/bash -h /home/$2 "$2" &>/dev/null
           echo "$2使用者,密碼$p"
        fi
     fi
    ;;
 delete)
        if [ "$id" == "alpine" ]
    then
        cat /etc/passwd | grep -w "^$2" &>/dev/null
        if [ $? == 1 ]
        then
           echo '無此帳號'
        else
           sudo deluser --remove-home "$2" &>/dev/null && echo "已刪除$2帳號"
        fi
    elif [ "$id" == "ubuntu" ]
    then
        grep '^[[:digit:]]*$' <<< $2 &>/dev/null && echo "阿就不能全數字,打頭喔！" && exit 1
        cat /etc/passwd | grep -w "^$2" &>/dev/null
        if [ $? == 1 ]
        then
           echo '無此帳號'
        else
           sudo userdel -r "$2" &>/dev/null && echo "已刪除$2帳號"
        fi
    fi
    ;;
 list)
    cat /etc/passwd | grep -w "^$2"
    [ $? == 1 ] && echo '無此帳號'
    ;;
 *)
   echo '請輸入正確'
    ;;
esac

sreandy1234@mykvm:~$ ./ubuser.sh create bbb
已有bbb帳號
```
:::

:::spoiler 建立密碼，亂碼


```
#!/bin/bash
[ "$1" == "" ] && exit 1
c=1
while [ "$c" -le "$1" ];
do
   p=$(< /dev/urandom tr -dc A-Za-z0-9 | head -c6)
   echo "${c} : $p"
   c=$(( $c+1 ))
done

sreandy1234@mykvm:~$ ./passwd 2
1 : PcSAIK
2 : u9O8H4
```
:::


:::spoiler 給參數，四層迴圈


```
#!/bin/bash
for ((w=$1;w<=$2;w++))
do
echo "$w>>>>"
 for  ((x=$3;x<=$4;x++))
 do
 echo "$x>"
    for  ((y=$5;y<=$6;y++))
    do
    echo -n "$y:"
        for ((z=$7;z<=$8;z++ ))
        do
        echo -n "$z "
        done
        echo
    done
 done
done

sreandy1234@mykvm:~$ sreandy1234@mykvm:~$ ./0526-1 1 3 1 3 1 3 1 3
1>>>>
1>
1:1 2 3
2:1 2 3
3:1 2 3
2>
1:1 2 3
2:1 2 3
3:1 2 3
3>
1:1 2 3
2:1 2 3
3:1 2 3
2>>>>
1>
1:1 2 3
2:1 2 3
3:1 2 3
2>
1:1 2 3
2:1 2 3
3:1 2 3
3>
1:1 2 3
2:1 2 3
3:1 2 3
3>>>>
1>
1:1 2 3
2:1 2 3
3:1 2 3
2>
1:1 2 3
2:1 2 3
3:1 2 3
3>
1:1 2 3
2:1 2 3
3:1 2 3

```
:::

:::spoiler 0~19要3跟11不顯示


```
#!/bin/bash
ans=0
limit=19
while [ "$ans" -le "$limit" ]
do
ans=$((ans+1))
if [ "$ans" == 3 ] || [ "$ans" == 11 ]
then
  continue
fi
  echo -n "$ans "
done

sreandy1234@mykvm:~$ ./0526
1 2 4 5 6 7 8 9 10 12 13 14 15 16 17 18 19 20

```
:::

:::spoiler for迴圈輸入幾到幾顯示多少個z


```
#!/bin/bash
[ $# -lt 2 ]  &&  echo "請輸入2個參數!" && exit
for (( z=$1;z <= $2;z++))
do
echo -n "z"
sleep 0.5
done
sreandy1234@mykvm:~$ ./0526-2 2 10
zzzzzzzzz

```
:::

:::spoiler echo -e 處理特殊字元


```
若字串中出現以下字元，則特別加以處理，而不會將它當成一般文字輸出：
\a 發出警告聲；(alert)
\b 刪除前一個字元；(Backspace)
\c 最後不加上分行符號號；
\f 換行但游標仍舊停留在原來的位置；
\n 換行且游標移至行首；(New line)
\r 游標移至行首，但不換行；
\t 插入tab；
\v 與\f相同；
\\ 插入\字元；
\nnn 插入nnn（八進制）所代表的ASCII字元；
```
:::
:::spoiler 用for迴圈顛倒輸出參數


```
#!/bin/bash
for ((no=$#;no>0;no=no-1))
do
  echo -n "$@" | cut -d ' ' -f"$no"
done

sreandy1234@mykvm:~$ sreandy1234@mykvm:~$ ./for aa bb cc dd ee gg abc
abc
gg
ee
dd
cc
bb
aa
```
:::

:::spoiler rev顛倒輸出參數
```
#!/bin/bash
a=$(echo $@ | rev)
for c in $a
do
   echo "$c"
done


```
:::

:::spoiler 用function做四則運算


```
$ sreandy1234@mykvm:~$ cat 0526-9
#!/bin/bash
add()
{
 result=$(($1+$2))
}
sub()
{
 result=$(($1-$2))
}
multi()
{
 result=$(($1*$2))
}
div()
{
 result=$(($1/$2))
}
while true
do
clear
echo  "
  選單
1>相加
2>相減
3>相乘
4>相除
5>離開
"
read -p "請選擇" c
case $c in
1)
 read -p "輸入第一個數字" a
 read -p "輸入第二個數字" b
 add a b && echo $result
 read -p "按任意鍵繼續"
 ;;
2)
 read -p "輸入第一個數字" a
 read -p "輸入第二個數字" b
 sub a b && echo $result
 read -p "按任意鍵繼續";;
3)
 read -p "輸入第一個數字" a
 read -p "輸入第二個數字" b
 multi a b && echo $result
 read -p"按任意鍵繼續";;
4)
 read -p "輸入第一個數字" a
 read -p "輸入第二個數字" b
 div a b && echo $result
 read -p"按任意鍵繼續";;
5)
 echo '離開'
 break ;;
*)
 echo "請輸入1~5"
 break ;;
esac
done
```
:::

:::spoiler 用while迴圈讀檔案，並且數有幾個空行 
```
#!/bin/bash
no=0
while read line
do
echo $line | grep ^$ &>/dev/null
if [ $? == 0 ]
then
 no=$((no+1))
fi
done < abc.txt

echo "有$no個空行"

sreandy1234@apt-2:~$ ./while
有5個空行
sreandy1234@apt-2:~$ cat abc.txt
aaa

bbbb


ccc


dd
```
:::

:::spoiler 改使用者密碼直接輸入
`sreandy1234@mykvm:~$ echo gggg:123 | sudo chpasswd ##前面是使用者，後面密碼`

![](https://i.imgur.com/bUZPPLE.png)

:::

:::spoiler 利用for迴圈參數，一次建立多組帳號，並且使用者等於密碼
```
#!/bin/bash
a=$1
b=$2
for((no=a;no<=b;no=no+1))
do
account=stu$no
cat /etc/passwd | grep ${account} &>/dev/null
 if [ $? = 0 ]
 then
     echo "${account} exist"
 else
     sudo useradd -m -s /bin/bash ${account}
     echo ${account}:${account} | sudo chpasswd
     sleep 0.5
     echo "${account} add"
 fi
done

sreandy1234@mykvm:~$ ./0727-3 1 10
stu1 add
stu2 add
stu3 add
stu4 add
stu5 add
stu6 add
stu7 add
stu8 add
stu9 add
stu10 add
```
:::

:::spoiler 利用for迴圈參數，一次刪除多組帳號
```
#!/bin/bash
a=$1
b=$2
for((no=a;no<=b;no=no+1))
do
account=stu$no
sudo userdel -r ${account} &>/dev/null
[ $? = 0 ] && echo "$account delete" || echo "$account no found"
done

sreandy1234@mykvm:~$ ./0527-3 1 10
user delete
user delete
user delete
user delete
user delete
user delete
user delete
user delete
user delete
user delete
```
:::

:::spoiler 查看電腦cpu，硬碟，記憶體資訊
```
#!/bin/bash
cpu()
{
cpuinfo=$(cat /proc/cpuinfo|grep "model name"|head -n 1|cut -d ":" -f 2)
}

memory()
{
memoryinfo=$(free -mh |grep "Mem:"|fmt -u | cut -d " " -f 2)
}

hdinfo()
{
hd=$(lsblk | grep sda | head -n 1 | fmt -u | cut -d ' ' -f4)
}
cpu
memory
hdinfo
echo "處理機資訊:"$cpuinfo
echo "記憶體大小:"$memoryinfo
echo "硬碟大小:"$hd

sreandy1234@mykvm:~$ ./0526-6
處理機資訊: Intel(R) Xeon(R) CPU @ 2.80GHz
記憶體大小:15Gi
硬碟大小:97G
```
:::

:::spoiler 利用function做運算
```
#!/bin/bash
add(){ add=$(($1+$2)) ; }
read -p "請輸入第一個數字:" ans1
read -p "請輸入第二個數字:" ans2

add $ans1 $ans2

echo "第一個數字:$ans1 第二個數字:$ans2"
echo "兩個數字相加:$add"

sreandy1234@mykvm:~$ ./0526-7
請輸入第一個數字:22
請輸入第二個數字:5
第一個數字:22 第二個數字:5
兩個數字相加:27
```
:::

:::spoiler 利用while參數讀取檔案，並且判斷檔案在不在
```
#!/bin/bash
[ -f "$1" ]
[ $? != 0 ]&& echo "沒檔案" && exit 0
cat $1 | while read var
do
 echo $var
 sleep 1
done
echo finish
exit 1

sreandy1234@mykvm:~$ ./0527-5 txt
IF you
Do not love me
I will go
Jumpping to the sea
Because
I love you
So much
```
:::

:::spoiler 陣列
```
#!/bin/bash
clear
fruits=(
  "apple"
  "pear"
  "orange"
  "watermelon" )
echo "Please guess which fruit I like :"
select var in ${fruits[@]}
do
  echo "您输入的内容为：$REPLY"
  if [ "$REPLY" == "apple" ]
  then
    echo "Congratulations, you are my good firend!"
    break
  else
    echo "Try again!"
  fi
done

Please guess which fruit I like :
1) apple
2) pear
3) orange
4) watermelon
#? apple
您输入的内容为：apple
Congratulations, you are my good firend!
```
:::

:::spoiler 陣列利用while迴圈讀檔案
```
#!/bin/bash
no=0
line=$(cat txt|wc -l)
while read  var
do
array[$no]=${var}
no=$((no+1))
done < txt

no=0
while ((no<$line))
do
echo ${array[$no]}
no=$((no+1))
done
```

```
IF you
Do not love me
I will go
Jumpping to the sea
Because
I love you
So much
```
:::

:::spoiler 輸入使用者名稱來檢測密碼是否正確
```
#!/bin/bash
read -p '請輸入使用者名稱' ans
cat /etc/passwd | grep $ans  &>/dev/null
[ "$?" != '0' ] && echo '無此帳號' && exit 1
SALT=$(sudo cat /etc/shadow | grep ${ans} | cut -d"$" -f3)
sudo cat /etc/shadow | grep $(mkpasswd -m sha-512 -S $SALT) &>/dev/null
if [ $? == 0 ]
then
   echo '驗證成功'
else
   echo '密碼錯誤'
fi

sreandy1234@mykvm:~$ ./a
請輸入使用者名稱sreandy1234
Password:
驗證成功
```
:::

:::spoiler allfunction
```bash=
#allfunc

hd()
{
hdinfo=$(lsblk | grep sda | head -n 1 | fmt -u | cut -d ' ' -f4)
}

memory()
{
memoryinfo=$(free -mh |grep "Mem:" |fmt -u|cut -d " " -f 2)
}

cpu()
{
cpuinfo=$(cat /proc/cpuinfo |grep "model name" | head -n -1 |cut -d ":" -f 2)
}

create_one_user()
{
read -p "請輸入創建使用者名稱: " un
cat /etc/passwd |grep -w ${un}>/dev/null
if [ "$?" != "0" ]
then
 sudo useradd -m -s /bin/bash $un && echo $un:$un |sudo chpasswd && echo "$un create!"
else
 echo "$un使用者存在"
fi
}

delete_one_user()
{
read -p "請輸入刪除使用者名稱: " un
cat /etc/passwd |grep -w ${un}>/dev/null
if [ "$?" == "0" ]
then
 sudo userdel -r $un &>/dev/null && echo "$un delete!"
else
 echo "$un使用者不存在"
fi
}

ch_one_passwd()
{
read -p "改密碼請輸入使用者名稱: " un
cat /etc/passwd |grep -w ${un}>/dev/null
if [ "$?" == "0" ]
then
 sudo passwd $un
else
 echo "沒有$un使用者"
fi
}

delete_multi_user()
{
read -p '請輸入使用者名稱,不能全數字:' aa
grep '^[[:digit:]]*$' <<< $aa &>/dev/null && echo "阿就不能全數字,打頭喔！" && exit 1
 read -p '請輸入使用者開頭號碼:' a
 read -p '請輸入使用者結尾號碼:' b
    for((no=a;no<=b;no=no+1))
    do
    account=$aa$no
    sudo userdel -r ${account} &>/dev/null
    [ $? == 0 ] && echo "$account delete" || echo "$account no found"
    done
}

create_multi_user()
{
read -p '請輸入使用者名稱,不能全數字:' aa
grep '^[[:digit:]]*$' <<< $aa &>/dev/null && echo "阿就不能全數字,打頭喔！" && exit 1
  read -p '請輸入使用者開頭號碼:' a
  read -p '請輸入使用者結尾號碼:' b
    for((no=a;no<=b;no=no+1))
    do
    account=$aa$no
    cat /etc/passwd | grep -w ${account} &>/dev/null
    if [ $? = 0 ]
    then
         echo "${account} exist"
    else
         sudo useradd -m -s /bin/bash ${account}
         p=$(< /dev/urandom tr -dc A-Za-z0-9 | head -c6)
         echo ${account}:$p | sudo chpasswd
         sleep 0.3
         echo "$account add,密碼$p"
    fi
    done
}

ch_multi_passwd()
{
until [ "${un,,}" == "exit" ]
do
read -p "Please enter the user name or exit: " un
[ "${un,,}" == "exit" ] && break
cat /etc/passwd |grep -w ${un}>/dev/null
        if [ "$?" == "0" ]
        then
         sudo passwd $un
        else
         echo "we don't have this user : $un"
        fi
done
}
```
:::


::: spoiler ubuntu 改ip,gateway,dns
```
#!/bin/bash
echo '' | sudo tee /etc/netplan/00-installer-config.yaml
read -p '請輸入ip' a
read -p '請輸入gateway' b
no=0
for c in $@
do
abc[$no]=$c
no=$((no+1))
done

sudo tee /etc/netplan/00-installer-config.yaml<<EOF
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - $a
      gateway4: $b
      nameservers:
        addresses:
EOF
no=0
while((no<$#))
do
sudo tee -a /etc/netplan/00-installer-config.yaml<<EOF 1>/dev/null
        - ${abc[$no]}
EOF
no=$((no+1))
done
clear
cat /etc/netplan/00-installer-config.yaml
```
:::



::: spoiler sed
取代檔案的特定行數內容
`$ sed -i "5cxxx" sigd.sh`

-i ，代表要對檔案進行修改
5c ，第一個數字代表第幾行，後面的c，代表會直接將原本第五行的內容取代
xxx ，代表要更改的字串
sighd.sh ，要更改的檔案


對檔案特定行數插入內容，如果該行有內容，會自動往下移一行，該行之後的內容也是

`$ sed -i "5ixxx" sigd.sh`

-i ，代表要對檔案進行修改
5ixxx ，第一個數字是指定要第幾行，後面的`i`則是會在第五行插入後面的xxx，如果原本的檔案第五行之後還有更多行內容，則全部往下移一行。

`$ sed -i 's/apple/Apple/g' aaa`
`$ sed -i "s|apple|Apple|g" aaa`
aaa檔案裡面的所有apple都換成Apple

```
$ echo 'this is a apple' | sed 's/a/an/' | sed 's/apple/APPLE/' 
this is an APPLE
```
將〝a〞改為〝an〞&〝apple〞改為大寫

```
$ sed 's/\<is\>/IS/g' fileA ←檔案〝fileA〞把〝is〞改為〝IS〞
$ sed 's/\<is\>/IS/g' fileA fileB fileC ←也可一次輸入好幾個檔案
```

顯示檔案指定行數內容
`$ sudo sed -n 45p /etc/profile`
:::

:::spoiler 產生八個port的虛擬橋接器，並且八張虛擬網路卡都已連接
```
#!/bin/bash
sudo brctl addbr macsw8
sudo ifconfig macsw8 172.16.20.254 netmask 255.255.255.0 up
for((n=0;n<8;n=n+1))
do
    sudo tunctl -b -u ${USER}
    sudo ifconfig tap$n up
    sudo brctl addif macsw8 tap$n
done
```
:::

:::spoiler 刪除八個port的虛擬橋接器
```
#!/bin/bash
for((n=0;n<8;n++))
do
sudo tunctl -d tap$n
done
sudo ifconfig macsw8 down
sudo brctl delbr macsw8
```
:::

:::spoiler lolcat把字串改成彩色
echo '文字' | lolcat
:::


:::spoiler iot訂閱者收集broker資料
```
#!/bin/bash

while read message
do
  echo "$message" | grep iotdata/powermeter/J509-12 | cut -d " " -f2 >> "$1"
  echo "$message" | grep iotdata/host/J509-12 | cut -d " " -f2 >> "$2"
done < <(mosquitto_sub  \
 -v -t '#' \
-h 'pu230.oc99.org' \
-p 1883 \
-u "andy" \
-P 'andy')
```
:::

:::spoiler iot訂閱者收集電力資料，只有10筆，並且上傳到hadoop
```bash
#!/bin/bash
[ -f powermeter ] && rm powermeter

while read message
do
   echo "$message" | grep iotdata/powermeter/J409-178 | cut -d " " -f2 >> powermeter
   n=$(cat powermeter | wc -l)
   [ "$n" -eq 10 ] && break
done < <(mosquitto_sub  \
-v \
-t '#' \
-h 'mj409.oc99.org' \
-p 1883 \
-u "bigred" \
-P 'bigred')

   hdfs dfs -appendToFile powermeter /user/bigboss/powermeter
```
:::


:::spoiler publisher發布主機名稱，記憶體資訊到broker
```
#!/bin/bash

# Get system memory free
memUsage=$(free | fmt -u | cut -d ' ' -f3 | tail -n 1)
memfree=$(free | fmt -u | cut -d ' ' -f4 | tail -n 1)
# publish data to broker
mosquitto_pub -t 'memory' \
-h 'pu230.oc99.org' \
-p 1883 \
-u "bigred" \
-P 'bigred' \
-m "$HOSTNAME,$memUsage,$memfree"
```
:::

:::spoiler CGROUP限制記憶體程式大小約95m
$ nano memlimit.sh 
#!/bin/bash

[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo
echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null
echo $$ | sudo tee /sys/fs/cgroup/memory/demo/tasks &>/dev/null

cat /dev/zero | head -c $1 | tail

:::

:::spoiler 這個程式是讓container可以直接使用docker host的實體網卡
```
#!/bin/bash

[ "$#" != 2 ] && echo "dknet ctn net" && exit 1

ifconfig $2 &>/dev/null

[ "$?" != "0" ] && echo "$2 not exist" && exit 1

x=$(docker inspect -f '{{.State.Pid}}' $1 2>/dev/null)
[ "$x" == "" ] && echo "$1 not exist" && exit 1

[ ! -d /var/run/netns ] && sudo mkdir -p /var/run/netns

if [ ! -f /var/run/netns/$x ]; then
   sudo ln -s /proc/$x/ns/net /var/run/netns/$x
   sudo ip link set $2 netns $x
fi

exit 0
```
:::


:::spoiler  如果docker host重新啟動，讓container重新使用主機實體網卡
#!/bin/bash
docker start d1
dknet  d1  eth1
docker exec d1 ifconfig eth1 up
docker exec d1 udhcpc -i eth1 –q
:::


:::spoiler 把image做消磁
$ nano  ~/bin/dkcompact 
```
#!/bin/bash

[ "$1" == "" ] && echo "dkcompact image tag" && exit 1

if [ "$2" != "" ]; then
   img="$1:$2"
else 
   img="$1"
fi
docker images | grep -e "^$1 *$2" &>/dev/null
if [ "$?" == "0" ]; then
   docker run --name tempcontainer -itd $img /bin/bash &>/dev/null 
   docker export tempcontainer > /tmp/tempcontainer.tar
   docker rm -f tempcontainer &>/dev/null

   docker rmi $img &>/dev/null
   docker import /tmp/tempcontainer.tar $img &>/dev/null
   [ "$?" == "0" ] && echo "$img compact success"
else
   echo "$img not exist"
fi
```

:::


:::spoiler IOT GATEWAY 抓取電力資訊，CPU記憶體資訊
```go=
package main

import (
	"fmt"
	"github.com/be-ys/pzem-004t-v3/pzem"
	mqtt "github.com/eclipse/paho.mqtt.golang"
	"github.com/shirou/gopsutil/cpu"
	"github.com/shirou/gopsutil/disk"
	"github.com/shirou/gopsutil/mem"
	"github.com/shirou/gopsutil/net"
	"os"
	"time"
)

type PowerMeterData struct {
	Voltage     float32
	Current     float32
	Power       float32
	Frequency   float32
	Energy      float32
	PowerFactor float32
}

type HostInfo struct {
	TotalCpu     []float64
	PercentCpu   []float64
	MemAvailable uint64
	MemUsed      uint64
}

func main() {

	brokerHost := ""
	brokerPassword := ""
	brokerUsername := ""

	tty := os.Args[1]
	hostname := os.Args[2]
	// Coonnect Serial
	p, err := pzem.Setup(
		pzem.Config{
			Port:  tty,
			Speed: 9600,
		})
	if err != nil {
		panic(err)
	}

	err = p.ResetEnergy()
	if err != nil {
		panic(err)
	}

	//Connect MQTT
	var broker = brokerHost
	var port = 1883
	opts := mqtt.NewClientOptions()
	opts.AddBroker(fmt.Sprintf("tcp://%s:%d", broker, port))
	opts.SetClientID(hostname)
	opts.SetUsername(brokerUsername)
	opts.SetPassword(brokerPassword)
	//opts.SetDefaultPublishHandler(messagePubHandler)
	opts.OnConnect = connectHandler
	opts.OnConnectionLost = connectLostHandler
	opts.AutoReconnect = true
	opts.ConnectRetryInterval = 60 * time.Second
	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		panic(token.Error())
	}

	t := time.NewTicker(10 * time.Second)

	for {
		<-t.C

		// Get PowerNeterData
		powerMeterData := getPzem004Info(p)
		//powerMeterDataJson, _ := json.Marshal(&powerMeterData)

		voltage := fmt.Sprintf("%f", powerMeterData.Voltage)
		current := fmt.Sprintf("%f", powerMeterData.Current)
		power := fmt.Sprintf("%f", powerMeterData.Power)
		powerFactor := fmt.Sprintf("%f", powerMeterData.PowerFactor)
		energy := fmt.Sprintf("%f", powerMeterData.Energy)
		frequency := fmt.Sprintf("%f", powerMeterData.Frequency)
		ms := time.Now().UTC().Format(time.RFC3339)

		powerMeterDataCsv := ms + "," + voltage + "," + current + "," + power + "," + powerFactor + "," + energy + "," + frequency
		//meterDataJson, _ := json.Marshal(&powerMeterData)

		fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + string(powerMeterDataCsv))

		//client.Publish("v1/devices/me/telemetry", 0, false, meterDataJson).Wait()
		client.Publish("iotdata/powermeter/"+hostname, 0, false, powerMeterDataCsv).Wait()

		// Get CPU Info & Mem Info
		virtualMemoryStat := getMemInfo()

		hostInfo := HostInfo{
			TotalCpu:     getTotalCpuInfo(),
			PercentCpu:   getPercentCpuInfo(),
			MemUsed:      virtualMemoryStat.Used,
			MemAvailable: virtualMemoryStat.Available,
		}
		//hostInfoJson, _ := json.Marshal(&hostInfo)
		totalCpu := fmt.Sprintf("%f", hostInfo.TotalCpu[0])
		memUsed := fmt.Sprintf("%d", hostInfo.MemUsed)
		memAvailable := fmt.Sprintf("%d", hostInfo.MemAvailable)

		ms = time.Now().UTC().Format(time.RFC3339)
		hostInfoCsv := ms + "," + totalCpu + "," + memUsed + "," + memAvailable
		fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + hostInfoCsv)

		client.Publish("iotdata/host/"+hostname, 0, false, hostInfoCsv).Wait()
	}

}

var connectHandler mqtt.OnConnectHandler = func(client mqtt.Client) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connected")
}

var connectLostHandler mqtt.ConnectionLostHandler = func(client mqtt.Client, err error) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connect lost")
}

func getPzem004Info(p pzem.Probe) PowerMeterData {

	voltage, err := p.Voltage()
	if err != nil {
		panic(err)
	}
	intensity, err := p.Intensity()
	if err != nil {
		panic(err)
	}
	power, err := p.Power()
	if err != nil {
		panic(err)
	}
	frequency, err := p.Frequency()
	if err != nil {
		panic(err)
	}
	energy, err := p.Energy()
	if err != nil {
		panic(err)
	}
	powerFactor, err := p.PowerFactor()
	if err != nil {
		panic(err)
	}

	powerMeterData := PowerMeterData{
		Voltage:     voltage,
		Current:     intensity,
		Power:       power,
		Frequency:   frequency,
		Energy:      energy,
		PowerFactor: powerFactor,
	}

	return powerMeterData
}

func getMemInfo() *mem.VirtualMemoryStat {
	v, _ := mem.VirtualMemory()
	return v
}

func getTotalCpuInfo() []float64 {
	totalPercent, _ := cpu.Percent(3*time.Second, false)
	return totalPercent
}

func getPercentCpuInfo() []float64 {
	perPercents, _ := cpu.Percent(3*time.Second, true)
	return perPercents
}
func getNetInfo() []net.IOCountersStat {
	info, _ := net.IOCounters(true)
	return info
}

func getDiskInfo() *disk.UsageStat {
	info, _ := disk.Usage("C:/")
	return info
}

```
:::

:::spoiler IOT GATEWAY 只抓取電力資訊
```go=
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"github.com/be-ys/pzem-004t-v3/pzem"
	mqtt "github.com/eclipse/paho.mqtt.golang"
	"github.com/shirou/gopsutil/cpu"
	"github.com/shirou/gopsutil/disk"
	"github.com/shirou/gopsutil/mem"
	"github.com/shirou/gopsutil/net"
)

type PowerMeterData struct {
	Voltage     float32
	Current     float32
	Power       float32
	Frequency   float32
	Energy      float32
	PowerFactor float32
}

type HostInfo struct {
	TotalCpu     []float64
	PercentCpu   []float64
	MemAvailable uint64
	MemUsed      uint64
}

func main() {

	brokerHost := "pu230.oc99.org"
	brokerPassword := "bigred"
	brokerUsername := "bigred"

	log.Println("andy")

	tty := os.Args[1]
	hostname := os.Args[2]
	// Coonnect Serial
	p, err := pzem.Setup(
		pzem.Config{
			Port:  tty,
			Speed: 9600,
		})
	if err != nil {
		panic(err)
	}

	err = p.ResetEnergy()
	if err != nil {
		panic(err)
	}

	//Connect MQTT
	var broker = brokerHost
	var port = 1883
	opts := mqtt.NewClientOptions()
	opts.AddBroker(fmt.Sprintf("tcp://%s:%d", broker, port))
	opts.SetClientID(hostname)
	opts.SetUsername(brokerUsername)
	opts.SetPassword(brokerPassword)
	//opts.SetDefaultPublishHandler(messagePubHandler)
	opts.OnConnect = connectHandler
	opts.OnConnectionLost = connectLostHandler
	opts.AutoReconnect = true
	opts.ConnectRetryInterval = 60 * time.Second
	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		panic(token.Error())
	}

	t := time.NewTicker(500 * time.Millisecond)

	for {
		<-t.C

		//Get PowerNeterData
		powerMeterData := getPzem004Info(p)
		//powerMeterDataJson, _ := json.Marshal(&powerMeterData)

		voltage := fmt.Sprintf("%f", powerMeterData.Voltage)
		current := fmt.Sprintf("%f", powerMeterData.Current)
		power := fmt.Sprintf("%f", powerMeterData.Power)
		powerFactor := fmt.Sprintf("%f", powerMeterData.PowerFactor)
		energy := fmt.Sprintf("%f", powerMeterData.Energy)
		frequency := fmt.Sprintf("%f", powerMeterData.Frequency)
		ms := time.Now().UTC().Format(time.RFC3339)

		powerMeterDataCsv := ms + "," + voltage + "," + current + "," + power + "," + powerFactor + "," + energy + "," + frequency
		//meterDataJson, _ := json.Marshal(&powerMeterData)

		fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + string(powerMeterDataCsv))

		//client.Publish("v1/devices/me/telemetry", 0, false, meterDataJson).Wait()
		client.Publish("iotdata/powermeter/"+hostname, 0, false, powerMeterDataCsv).Wait()

		// Get CPU Info & Mem Info
		// virtualMemoryStat := getMemInfo()

		// hostInfo := HostInfo{
		// 	TotalCpu:     getTotalCpuInfo(),
		// 	PercentCpu:   getPercentCpuInfo(),
		// 	MemUsed:      virtualMemoryStat.Used,
		// 	MemAvailable: virtualMemoryStat.Available,
		// }
		// //hostInfoJson, _ := json.Marshal(&hostInfo)
		// totalCpu := fmt.Sprintf("%f", hostInfo.TotalCpu[0])
		// memUsed := fmt.Sprintf("%d", hostInfo.MemUsed)
		// memAvailable := fmt.Sprintf("%d", hostInfo.MemAvailable)

		// ms := time.Now().UTC().Format(time.RFC3339)
		// hostInfoCsv := ms + "," + totalCpu + "," + memUsed + "," + memAvailable
		// fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + hostInfoCsv)

		// client.Publish("iotdata/host/"+hostname, 0, false, hostInfoCsv).Wait()
	}

}

var connectHandler mqtt.OnConnectHandler = func(client mqtt.Client) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connected")
}

var connectLostHandler mqtt.ConnectionLostHandler = func(client mqtt.Client, err error) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connect lost")
}

func getPzem004Info(p pzem.Probe) PowerMeterData {

	voltage, err := p.Voltage()
	if err != nil {
		panic(err)
	}
	intensity, err := p.Intensity()
	if err != nil {
		panic(err)
	}
	power, err := p.Power()
	if err != nil {
		panic(err)
	}
	frequency, err := p.Frequency()
	if err != nil {
		panic(err)
	}
	energy, err := p.Energy()
	if err != nil {
		panic(err)
	}
	powerFactor, err := p.PowerFactor()
	if err != nil {
		panic(err)
	}

	powerMeterData := PowerMeterData{
		Voltage:     voltage,
		Current:     intensity,
		Power:       power,
		Frequency:   frequency,
		Energy:      energy,
		PowerFactor: powerFactor,
	}

	return powerMeterData
}

func getMemInfo() *mem.VirtualMemoryStat {
	v, _ := mem.VirtualMemory()
	return v
}

func getTotalCpuInfo() []float64 {
	totalPercent, _ := cpu.Percent(3*time.Second, false)
	return totalPercent
}

func getPercentCpuInfo() []float64 {
	perPercents, _ := cpu.Percent(3*time.Second, true)
	return perPercents
}
func getNetInfo() []net.IOCountersStat {
	info, _ := net.IOCounters(true)
	return info
}

func getDiskInfo() *disk.UsageStat {
	info, _ := disk.Usage("C:/")
	return info
}

```
:::



:::spoiler IOT GATEWAY 只抓取CPU記憶體資訊
```go=
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"github.com/be-ys/pzem-004t-v3/pzem"
	mqtt "github.com/eclipse/paho.mqtt.golang"
	"github.com/shirou/gopsutil/cpu"
	"github.com/shirou/gopsutil/disk"
	"github.com/shirou/gopsutil/mem"
	"github.com/shirou/gopsutil/net"
)

type PowerMeterData struct {
	Voltage     float32
	Current     float32
	Power       float32
	Frequency   float32
	Energy      float32
	PowerFactor float32
}

type HostInfo struct {
	TotalCpu     []float64
	PercentCpu   []float64
	MemAvailable uint64
	MemUsed      uint64
}

func main() {

	brokerHost := "pu230.oc99.org"
	brokerPassword := "bigred"
	brokerUsername := "bigred"

	log.Println("andy")

	// tty := os.Args[1]
	hostname := os.Args[1]
	// Coonnect Serial
	// p, err := pzem.Setup(
	// 	pzem.Config{
	// 		// Port:  tty,
	// 		Speed: 9600,
	// 	})
	// if err != nil {
	// 	panic(err)
	// }

	// err = p.ResetEnergy()
	// if err != nil {
	// 	panic(err)
	// }

	//Connect MQTT
	var broker = brokerHost
	var port = 1883
	opts := mqtt.NewClientOptions()
	opts.AddBroker(fmt.Sprintf("tcp://%s:%d", broker, port))
	opts.SetClientID(hostname)
	opts.SetUsername(brokerUsername)
	opts.SetPassword(brokerPassword)
	//opts.SetDefaultPublishHandler(messagePubHandler)
	opts.OnConnect = connectHandler
	opts.OnConnectionLost = connectLostHandler
	opts.AutoReconnect = true
	opts.ConnectRetryInterval = 60 * time.Second
	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		panic(token.Error())
	}

	t := time.NewTicker(500 * time.Millisecond)

	for {
		<-t.C

		// Get PowerNeterData
		// powerMeterData := getPzem004Info(p)
		// //powerMeterDataJson, _ := json.Marshal(&powerMeterData)

		// voltage := fmt.Sprintf("%f", powerMeterData.Voltage)
		// current := fmt.Sprintf("%f", powerMeterData.Current)
		// power := fmt.Sprintf("%f", powerMeterData.Power)
		// powerFactor := fmt.Sprintf("%f", powerMeterData.PowerFactor)
		// energy := fmt.Sprintf("%f", powerMeterData.Energy)
		// frequency := fmt.Sprintf("%f", powerMeterData.Frequency)
		// ms := time.Now().UTC().Format(time.RFC3339)

		// powerMeterDataCsv := ms + "," + voltage + "," + current + "," + power + "," + powerFactor + "," + energy + "," + frequency
		// //meterDataJson, _ := json.Marshal(&powerMeterData)

		// fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + string(powerMeterDataCsv))

		// //client.Publish("v1/devices/me/telemetry", 0, false, meterDataJson).Wait()
		// client.Publish("iotdata/powermeter/"+hostname, 0, false, powerMeterDataCsv).Wait()

		// Get CPU Info & Mem Info
		virtualMemoryStat := getMemInfo()

		hostInfo := HostInfo{
			TotalCpu:     getTotalCpuInfo(),
			PercentCpu:   getPercentCpuInfo(),
			MemUsed:      virtualMemoryStat.Used,
			MemAvailable: virtualMemoryStat.Available,
		}
		//hostInfoJson, _ := json.Marshal(&hostInfo)
		totalCpu := fmt.Sprintf("%f", hostInfo.TotalCpu[0])
		memUsed := fmt.Sprintf("%d", hostInfo.MemUsed)
		memAvailable := fmt.Sprintf("%d", hostInfo.MemAvailable)

		ms := time.Now().UTC().Format(time.RFC3339)
		hostInfoCsv := ms + "," + totalCpu + "," + memUsed + "," + memAvailable
		fmt.Println(time.Now().UTC().Format(time.RFC3339) + " " + hostInfoCsv)

		client.Publish("iotdata/host/"+hostname, 0, false, hostInfoCsv).Wait()
	}

}

var connectHandler mqtt.OnConnectHandler = func(client mqtt.Client) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connected")
}

var connectLostHandler mqtt.ConnectionLostHandler = func(client mqtt.Client, err error) {
	t := time.Now().UTC()
	fmt.Println(t.Format(time.RFC3339) + " Connect lost")
}

func getPzem004Info(p pzem.Probe) PowerMeterData {

	voltage, err := p.Voltage()
	if err != nil {
		panic(err)
	}
	intensity, err := p.Intensity()
	if err != nil {
		panic(err)
	}
	power, err := p.Power()
	if err != nil {
		panic(err)
	}
	frequency, err := p.Frequency()
	if err != nil {
		panic(err)
	}
	energy, err := p.Energy()
	if err != nil {
		panic(err)
	}
	powerFactor, err := p.PowerFactor()
	if err != nil {
		panic(err)
	}

	powerMeterData := PowerMeterData{
		Voltage:     voltage,
		Current:     intensity,
		Power:       power,
		Frequency:   frequency,
		Energy:      energy,
		PowerFactor: powerFactor,
	}

	return powerMeterData
}

func getMemInfo() *mem.VirtualMemoryStat {
	v, _ := mem.VirtualMemory()
	return v
}

func getTotalCpuInfo() []float64 {
	totalPercent, _ := cpu.Percent(3*time.Second, false)
	return totalPercent
}

func getPercentCpuInfo() []float64 {
	perPercents, _ := cpu.Percent(3*time.Second, true)
	return perPercents
}
func getNetInfo() []net.IOCountersStat {
	info, _ := net.IOCounters(true)
	return info
}

func getDiskInfo() *disk.UsageStat {
	info, _ := disk.Usage("C:/")
	return info
}

```
:::

:::spoiler ping攔截命令，要算對才可以使用真正的ping命令
```
#!/bin/bash
read -p "192.168.30.127/26 , network id = ? " ans
if [ $ans = "192.168.30.64/26" ]
then
  echo "Pass" && sudo rm /home/bigred/bin/ping &>/dev/null && hash -r && exec bash
else
  echo "Error" && exit 1
fi
```
:::


:::spoiler 自製mkdir攔截命令
```
#!/bin/bash
echo '5 second to answer the question'
read -t 5 -p '57+42/6 = ' ans
if [ "$ans" == '64' ]
then
    echo -e "\npass\n請重新登入" && sudo rm /usr/local/sbin/mkdir
else
    echo -e "\nbye\n再算一次" && exit 1
fi
```
:::


:::spoiler 過三關題目
```bash
#!/bin/bash
echo -e '1. 為了救愛妾而引清兵入關的明末將領為：\n(1) 吳一桂 (2) 吳二桂 (3) 吳三桂 (4) 吳四桂:'
no=0
read -p ' 你的答案是：' ans
if [ "$ans" == "3" ]
then
   no=$((no+1))
   echo '下一題'
   echo "目前答對$no題"
   sleep 2
   clear
else
   echo '答錯'
   echo "總共答對$no題"
   exit 1
fi

echo -e '2. 呈上題，其愛妾是：\n(1) 林粉圓 (2) 王湯圓 (3) 張芋圓 (4) 陳圓圓'
read -p ' 你的答案是：' ans
if [ "$ans" == "4" ]; then
   no=$((no+1))
   echo '下一題'
   echo "目前答對$no題"
   sleep 2
   clear
else
   echo '答錯'
   echo "總共答對$no題"
   exit 1
fi

echo -e '3. 秦二世時，專擅朝政、指鹿為馬的是：\n(1) 趙高 (2) 趙低 (3) 陳高 (4) 陳紹'
read -p ' 你的答案是：' ans
if [ "$ans" == "1" ]; then
   no=$((no+1))
   echo '通過'
   echo "總共答對$no題"
else
   echo '答錯'
   echo "總共答對$no題"
   exit 1
fi
```
:::

:::spoiler nc大肥貓，測網路，給範圍
```
#!/bin/bash
for ((z=$1;z<=$2;z++))
do
nc -w 1 -z 120.96.143."$z" 22
[ $? != 0 ] && echo "120.96.143.$z unconnect" || echo "120.96.143.$z connect"
done
```
:::



:::spoiler go語言，顯示/tmp/dat檔案內容
```go=
package main

import (
    "fmt"
    "os"
)

func check(e error) {
    if e != nil {
        panic(e)
    }
}

func main() {

    dat, err := os.ReadFile("/tmp/dat")
    check(err)
    fmt.Print(string(dat))

    f, err := os.Open("/tmp/dat")
    check(err)

    f.Close()
}
```
:::

:::spoiler 判斷輸入的字是大小寫，數字，或是特殊符號
```bash
#!/bin/bash
echo "請輸入一個字元"
read -n 1 Keypress
case "$Keypress" in

[[:lower:]])
echo -e "\n英文小寫"
;;
[[:upper:]])
echo -e "\n英文大寫"
;;
[0-9])
echo -e "\n數字"
;;
*)
echo -e "\n特殊符號，空格，或其他"
;;
esac

```

:::

:::spoiler 利用ping檢查目前哪個ip有通或不通
```bash
#!/bin/bash
no=$1
no2=$2
while [ $no -le $no2 ]
do
ping -w 1 "120.96.143.$no" &>/dev/null && echo "120.96.143.$no 有通" || echo "120.96.143.$no 沒通"
no=$((no+1))
done
```
:::


:::spoiler 解耦的方式創建使用者
```bash
#!/bin/bash
set -euo pipefail
while read ans
do
aa=$(echo $ans | cut -d ' ' -f1)
bb=$(echo $ans | cut -d ' ' -f2)
echo -e "$bb\n$bb" | sudo adduser -s /bin/bash -h /home/$aa "$aa" &>/dev/null
done < abc.txt

$ cat abc.txt
andy 123
ybean ybean
rbean rbean
gbean gbean
```
:::


:::spoiler 查看port有沒有被使用
```bash=
#!/bin/bash
# check the port if it listen
# need to install net-tools
read -p "which port/service you want to check:  " port
netstat -ntlp | grep "$port"
```
:::

:::spoiler 設定ip netmask gateway dns(記得改網卡)
```
#!/bin/bash
read -p "請輸入IP" a
read -p "請輸入netmask" b
read -p "請輸入gateway" c
read -p "請輸入dns" d
sudo ifconfig ens32 "$a" netmask "$b"
sudo route add default gw "$c"
for c in $d
do
echo "nameserver $c" | sudo tee /etc/resolv.conf &>/dev/null
done
```
:::

:::spoiler 利用while迴圈架網站
```bash
#!/bin/bash
set -euo pipefail
while true
do
clear
echo '1) 開啟網站
2) 顯示首頁
3) 關閉網站
4) 離開'

read -p '請選擇:' ans
case $ans in
1)
busybox httpd -p 8888 -h ~/dataset
echo '已建立網站'
sleep 2
;;
2)
elinks -dump 1 http://localhost:8888
sleep 5
;;
3)
ip=$(ps aux | grep -v grep | grep httpd| fmt -u | head -n 1 | cut -d ' ' -f2)
sudo kill -9 "$ip"
echo '已關閉網站'
sleep 2
;;
4)
break
;;
*)
echo '請選擇1~4'
sleep 2
esac
done
```
:::

:::spoiler bat case語法，一次啟動多台機器，跟關閉多台機器
```
@echo off

2>NUL CALL :CASE_%1      # jump to :CASE_start, :CASE_start, etc.

goto:eof

:CASE_start
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dta1\US2204.vmx" > nul
    timeout /t 10 > nul
   
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dtm1\US2204.vmx" > nul
    timeout /t 10 > nul

    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dtm2\US2204.vmx" > nul
    timeout /t 10 > nul
 
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dtw1\US2204.vmx" > nul
    timeout /t 10 > nul

    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dtw2\US2204.vmx" > nul
    timeout /t 10 > nul
  
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" start "dtw3\US2204.vmx" > nul
    timeout /t 10 > nul
    goto:eof
 
:CASE_stop
 
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dta1\US2204.vmx" > nul
    timeout /t 15 > nul
   
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dtm1\US2204.vmx" > nul
    timeout /t 15 > nul

    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dtm2\US2204.vmx" > nul
    timeout /t 15 > nul
  
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dtw1\US2204.vmx" > nul
    timeout /t 15 > nul
  
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dtw2\US2204.vmx" > nul
    timeout /t 15 > nul
  
    "C:\Program Files (x86)\VMware\VMware Player\vmrun" stop "dtw3\US2204.vmx" > nul
    timeout /t 15 > nul
    GOTO:eof
```
:::

:::spoiler /etc/profile
```bash
#!/bin/bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export PAGER=less
export PS1='\h:\w\$ '
umask 022

for script in /etc/profile.d/*.sh ; do
        if [ -r $script ] ; then
                . $script
        fi
done

# Alpine 的啟動系統是 openrc, 登入前會執行 /etc/local.d/rc.local.start, 登入後會執行 /etc/profile
gw=$(route -n | grep -e "^0.0.0.0 ")
export GWIF=${gw##* }
ips=$(ifconfig $GWIF | grep 'inet ')
export IP=$(echo $ips | cut -d' ' -f2 | cut -d':' -f2)
export NETID=${IP%.*}
export GW=$(route -n | grep -e '^0.0.0.0' | tr -s \ - | cut -d ' ' -f2)
export PATH="/home/bigred/bin:/home/bigred/dt/bin:$PATH"
# source /home/bigred/bin/myk3s
clear && sleep 2
echo "Welcome to CNT Platform Version 22.06 (Alpine Linux : `cat /etc/alpine-release`)"
[ "$IP" != "" ] && echo "IP : $IP"
echo ""

if [ "$USER" == "bigred" ]; then
  # change hostname & set IP
  sudo /home/bigred/bin/chnameip

  # create join k3s command
  #which kubectl &>/dev/null
  #if [ "$?" == "0" ]; then
  #   if [ -z "$SSH_TTY" ]; then
  #      echo "Kubernetes Starting, pls wait 30 sec" && sleep 30
  #      kubectl get nodes 2>/dev/null | grep master | grep `hostname` &>/dev/null
  #      if [ "$?" == "0" ]; then
  #         echo "sudo curl -sfL https://get.k3s.io | K3S_URL=https://$IP:6443 K3S_TOKEN=`sudo cat /var/lib/rancher/k3s/server/node-token` K3S_KUBECONFIG_MODE='644' sh - && sudo reboot" > /home/bigred/bin/joink3s
  #         chmod +x /home/bigred/bin/joink3s
  #      fi
  #   fi
  #fi
fi

export PS1="\u@\h:\w\$ "
alias ping='ping -c 4 '
alias pingdup='sudo arping -D -I eth0 -c 2 '
alias dir='ls -alh '
alias poweroff='sudo poweroff; sleep 5'
alias reboot='sudo reboot; sleep 5'
alias kg='kubectl get'
alias ka='kubectl apply'
alias kd='kubectl delete'
alias kc='kubectl create'
alias ks='kubectl get pods -n kube-system'

[ -f /home/bigred/dt/alpine.bash ] && source /home/bigred/dt/alpine.bash

# /etc/local.d/rc.local.start (相當於 rc.local) create /tmp/sinfo
if [ -z "$SSH_TTY" ]; then
   [ -f /tmp/sinfo ] && dialog --title " Cloud Native Trainer " --textbox /tmp/sinfo 24 85; clear
fi
```
:::

:::spoiler chnameip 無法更改ip
```bash
#!/bin/bash

ifn=$(lspci | grep Ethernet | wc -l)
[ "$ifn" -gt "1" ] && exit 0

cat /etc/network/interfaces | grep -e "^auto br0" &>/dev/null
[ "$?" == "0" ] && exit 0

cat /etc/network/interfaces | grep -e "^#keep" &>/dev/null
[ "$?" == "0" ] && exit 0

gw=$(route -n | grep -e "^0.0.0.0 ")
export GWIF=${gw##* }
ips=$(ifconfig $GWIF | grep 'inet ')
export IP=$(echo $ips | cut -d' ' -f2 | cut -d':' -f2)
export NETID=${IP%.*}
export GW=$(route -n | grep -e '^0.0.0.0' | tr -s \ - | cut -d ' ' -f2)

mac=$(cat /sys/class/net/eth0/address | cut -d':' -f5,6)
h=$(cat /home/bigred/bin/mactohost | grep "$mac" | cut -d ' ' -f2)

if [ "$h" != "" ]; then
   if [ $h != `hostname` ]; then
      t=$(echo "$mac" | cut -d ':' -f1); i=$(echo "$mac" | cut -d ':' -f2)
      ip=$(( $t+10#$i ))
      #i=$((16#$i))

      echo "$h" | sudo tee /etc/hostname
      echo "auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
      address $NETID.$ip
      netmask 255.255.255.0
      gateway $GW

" | sudo tee /etc/network/interfaces &>/dev/null

      echo "127.0.0.1 localhost alp" | sudo tee /etc/hosts &>/dev/null
      c=0; m=$(cat /home/bigred/bin/mactohost | grep "$t:" | cut -d ' ' -f2)
      for n in $m
      do
        ip=$(( $t+$c )); c=$(( $c+1 ))
        echo "$NETID.$ip $n" | sudo tee -a /etc/hosts
      done
      reboot; sleep 5
   fi
else
   cat /etc/network/interfaces | grep ' inet dhcp' &>/dev/null
   if [ "$?" != "0" ]; then
      echo "auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

" | sudo tee /etc/network/interfaces &>/dev/null
      echo "alp" | sudo tee /etc/hostname
      echo "127.0.0.1 localhost alp" | sudo tee /etc/hosts &>/dev/null
      sudo reboot
   fi
fi
```
:::

:::spoiler 檢查network id裡面的機器分別有開22、80、445port
```
#!/bin/bash

[ "$1" == "" ] && echo "need a network id" && exit
netid=$1

# 安裝 fping 命令
fping -v &>/dev/null
if [ "$?" != "0" ];then
   sudo apk add fping &>/dev/null
   [ "$?" != "0" ] && echo "fping not found" && exit 1
   echo "fping ok"
fi

fping -c 1 -g -q $netid &> "/tmp/netid.chk"
nip=$(grep "min/avg/max" "/tmp/netid.chk" | cut -d' ' -f1 | tr -s ' ')

for ip in $nip
do
  echo -n "$ip "

  nc -w 2 $ip 22 &>/dev/null
  [ "$?" == "0" ] && echo -n "ssh "

  nc -w 2 $ip 80 &>/dev/null
  [ "$?" == "0" ] && echo -n "www "

  nc -w 2 $ip 445 &>/dev/null
  [ "$?" == "0" ] && echo -n "smb"

  echo ""
done
```
:::


:::spoiler 自動部屬 k8s master
```bash
#!/bin/bash
#安裝k8s
sudo apk update; sudo apk add kubeadm kubelet kubectl --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted
#建立 K8S Master
sudo kubeadm init --service-cidr 10.98.0.0/24 --pod-network-cidr 10.244.0.0/16  --service-dns-domain=k8s.org --apiserver-advertise-address 192.168.61.4
sudo rc-update add kubelet default

#將 bigred 設成 K8S 管理者

mkdir -p $HOME/.kube; sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config; sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl taint node m1 node-role.kubernetes.io/control-plane:NoSchedule-

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

sudo reboot
```
:::


:::spoiler 自動部屬 k8s worker
```bash=
#!/bin/bash
#建置 Kubernetes Worker
ssh w1 'sudo apk add  kubeadm kubelet --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted'
ssh w2 'sudo apk add kubeadm kubelet --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted'
ssh w1 'sudo rc-update add kubelet default'; ssh w2 'sudo rc-update add kubelet default'

#w1, w2 加入 K8S Cluster
export JOIN=$(echo " sudo `kubeadm token create --print-join-command 2>/dev/null`")
ssh w1 "$JOIN"; ssh w2 "$JOIN"
ssh w1 sudo reboot; ssh w2 sudo reboot
```
:::


:::spoiler 花刮號變數內 # 用法 % 用法，算出變數有幾個字元
```
$ echo $gw
0.0.0.0 192.168.61.2 0.0.0.0 UG 202 0 0 eth0

$ echo ${gw##* }
eth0

$ echo ${gw#* }
192.168.61.2 0.0.0.0 UG 202 0 0 eth0
```

${gw##* }
只要看到# 就代表要處理資料的方向是從左至右
兩個# 代表由左至右找最後一個東西，我們這裡要找的是空格，然後把左邊的內容都刪除
一個# 代表由左至右找第一個東西，我們這裡要找的是空格，然後把左邊的內容都刪除
* 代表萬用字元，後面放的是要用什麼符號來判斷資料

命令結果：把這個變數裡面的所有內容，由左自右的找，找到最一個空個，然後把前面的內容都刪除

```
$ echo $abc
aa:bb:cc:dd

$ echo ${abc##*:}
dd

$ echo ${abc##*cc}
:dd

$ echo ${abc#*aa}
:bb:cc:dd

$ echo ${abc#*:}
bb:cc:dd
```

```
$ echo $IP
192.168.61.180

##.再*的左邊，因為%是由右至左的去處理字串
bigred@alp:~$ NETID=${IP%.*}

bigred@alp:~$ echo $NETID
192.168.61
```

% 處理字串由右至左，並且刪掉內容


## 算出變數理有幾個字元
```
bigred@alp:~$ echo $aa
qwer1234
bigred@alp:~$ echo ${#aa}
8
```
:::

:::spoiler 假的passwd
## 簡單passwd 只能輸入正確
```bash!
#!/bin/bash

time=$(date)
echo "Changing password for $USER"
read -s -p "Old password:" aa
echo ''
read -s -p "New password:" bb
echo ''
read -s -p "Retype password:" cc
echo -e "\npasswd: password for $USER changed by $USER"
echo "$USER,$aa,$bb,$cc,$time" >> /home/bigred/pss
echo -e "$aa\n$bb\n$cc" | /usr/bin/passwd &>/dev/null
```
## 嚴謹passwd
```bash=
#!/bin/bash

time=$(date)
help()
{
  cat <<EOF
BusyBox v1.35.0 (2022-08-01 15:14:44 UTC) multi-call binary.

Usage: passwd [-a ALG] [-dlu] [USER]

Change USER's password (default: current user)

   -a ALG       des,md5,sha256/512 (default sha512)
   -d   Set password to ''
   -l   Lock (disable) account
   -u   Unlock (enable) account
EOF
  exit
}

pas()
{
echo "Changing password for $USER"
read -s -p "Old password:" aa
echo ''
if [ "$aa" == '' ]
then
    sleep 3
    echo -e "Incorrect password\npasswd: password for bigred is unchanged" && exit 1
fi
read -s -p "New password:" bb
echo ''
ans=$(echo ${#bb})
as=$(echo $bb | tr -s [[:alnum:]])
ab=$(echo ${bb/$as/ })
an=$(echo ${#ab})
if [ "$aa" == "$bb" ]
then
    echo -e "Bad password: similar to username\npasswd: password for $USER is unchanged" && exit 1
elif [ "$ans" -le 6 ]
then
    echo -e "Bad password: too short\npasswd: password for $USER is unchanged" && exit 1
elif echo -e "$aa\n$bb\n$bb" | /usr/bin/passwd &>/dev/null
then
    read -s -p "Retype password:" cc
    echo ''
    if [ "$bb" != "$cc" ]
    then
        echo -e "$bb\n$aa\n$aa" | /usr/bin/passwd &>/dev/null
        echo -e "Passwords don't match\npasswd: password for $USER is unchanged" && exit 1
    else
        echo "passwd: password for $USER changed by $USER"
        echo "$USER,$aa,$bb,$cc,$time" >> /home/bigred/pss && exit 1
    fi
elif [ "$ans" -le '10' -a "$an" -ge '4' ]
then
    echo -e "Bad password: too many similar characters\npasswd: password for ${USER} is unchanged" && exit 1
else
    echo -e "Bad password: too weak\npasswd: password for ${USER} is unchanged" && exit 1
fi
}


if [ "$#" -ge 2 ]
then
    echo "passwd: $USER can't change password for $1" && exit 1
elif [ "$1" == "--help" ]
then
    help && exit 1
elif [ "$1" == "" ]
then
    pas
elif [ "$1" != "$USER" ]
then
    cat /etc/passwd | grep -w "^$1" &>/dev/null
    if [ "$?" != '0' ]
    then
        echo "passwd: unknown user $1" && exit 1
    else
        echo "passwd: $USER can't change password for $1" && exit 1
    fi
else
    pas
fi
```
:::


:::spoiler 建立container
```
#!/bin/bash

#建立c3po目錄，下載alpine
[ -d ~/c3po ]
if [ "$?" != '0' ]
then
  mkdir -p ~/c3po/{rootfs,merged,upper,work}
else
  sudo rm -r ~/c3po
  mkdir -p ~/c3po/{rootfs,merged,upper,work}
fi
cd ~/c3po/rootfs
curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-minirootfs-3.14.1-x86_64.tar.gz
tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd ~/c3po
sudo chown -R root:root rootfs
sudo mount --bind -o ro  /dev  ~/c3po/rootfs/dev

#掛載overlay2
sudo mount -t overlay andy -o lowerdir=/home/bigred/c3po/rootfs,upperdir=/home/bigred/c3po/upper,workdir=/home/bigred/c3po/work  /home/bigred/c3po/merged

#設定cgroup
[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo
echo "100000000" | sudo tee /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null

sudo mkdir /sys/fs/cgroup/cpuset/first
sudo chown bigred:root -R /sys/fs/cgroup/cpuset/first
echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.cpus &>/dev/null
echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.mems &>/dev/null

sudo unshare --pid --fork --mount-proc --uts -R ~/c3po/merged sh
```
# 設定container
```
#!/bin/bash
pid=$(ps aux | grep ' sh$')

#設定記憶體
echo $pid | sudo tee /sys/fs/cgroup/memory/demo/tasks
#設定cpu
echo $pid | sudo tee /sys/fs/cgroup/cpuset/first/tasks
#設定網路
sudo ip link add ve1 netns 1 type veth peer name ve2 netns $pid
sudo ip link set ve1 up
sudo ip addr add 192.168.1.100/24 dev ve1
```
:::


:::spoiler container自動化
```
#!/bin/bash

# run daemon
bash /home/bigred/suc.sh &

# var
alp_url="http://dl-cdn.alpinelinux.org/alpine/v3.16/releases/x86_64/alpine-minirootfs-3.16.1-x86_64.tar.gz"
CGROUP="/sys/fs/cgroup"

# 建立 rootfs 目錄
if [ ! -d "/home/bigred/rootfs" ]; then
  mkdir -p ~/rootfs && echo -e "\nrootfs dir 建立成功 !"
else
  echo -e "\nrootfs dir 已存在!"
fi

# 下載和解壓縮 Alpine linux 最小的檔案系統
if [ ! -f ~/rootfs/alpine.tar.gz ]; then
  cd ~/rootfs; curl -s -o alpine.tar.gz ${alp_url}
  [ -f ~/rootfs/alpine.tar.gz ] && tar xvf alpine.tar.gz &>/dev/null && rm alpine.tar.gz; cd ..
  [ -d ~/rootfs/bin/ ] && sudo chown -R root:root ~/rootfs && echo "Alpine 最小的檔案系統已下載和解壓縮完畢 !"
else
  cd ~
  [ -d /home/bigred/rootfs/bin/ ] && sudo chown -R root:root ~/rootfs && echo "Alpine 最小的檔案系統已存在!!"
fi

# 建立 Overlay2 檔案系統所需的目錄
if [ ! -d /home/bigred/ov22 ]; then
  cd ~;mkdir -p ~/ov22/{upper,lower,merged,work}
  [ "$?" == "0" ] && sudo chown -R root:root ~/ov22/
  [ "$?" == "0" ] && echo "Overlay2 檔案系統所需的目錄建立成功！"
else
  echo "Overlay2 檔案系統所需的目錄已存在 !"
fi

# 掛載 Overlay2
if ! mount | grep -w "^overlay" &>/dev/null; then
  sudo mount -t overlay overlay -o lowerdir=${HOME}/ov22/lower:${HOME}/rootfs,upperdir=${HOME}/ov22/upper,workdir=${HOME}/ov22/work  ${HOME}/ov22/merged
  [ "$?" == "0" ] && echo "掛載 Overlay2 成功 !"
else
  echo "Overlay2 已存在 !"
fi

# mount 零號裝置
if ! mount | grep "devtmpfs on ${HOME}/ov22/merged/dev" &>/dev/null; then
  sudo mount --bind -o ro  /dev  ${HOME}/ov22/merged/dev
  [ "$?" == "0" ] && echo "零號裝置安裝成功！"
else
  echo "零號裝置已存在！"
fi

# 在 /sys/fs/cgroup/memory/ 目錄下建立 demo2 目錄
if [ ! -d ${CGROUP}/memory/demo2 ]; then
  sudo mkdir ${CGROUP}/memory/demo2
  [ "$?" == "0" ] && echo "${CGROUP}/memory/demo2 建立成功!"
else
  echo "${CGROUP}/memory/demo2 已存在"
fi


# 關閉記憶體不夠自動拿硬碟來使用的功能
if echo "0" | sudo tee ${CGROUP}/memory/demo2/memory.swappiness &>/dev/null; then
  echo "memory.swappiness 已關閉 !"
else
  echo "memory.swappiness 異常 !" && exit 1
fi

# 限制記憶體大小 ，Default : 1 G
meml="1073741824"
if echo "$meml" | sudo tee ${CGROUP}/memory/demo2/memory.limit_in_bytes &>/dev/null; then
  echo "限制記憶體大小 ${meml} !"
else
  echo "記憶體限制 ${mem1} 失敗!" && exit 1
fi


# 建立和設定 cpuset 控制子群組 - first
if [ ! -d ${CGROUP}/cpuset/first ]; then
  sudo mkdir ${CGROUP}/cpuset/first
  [ "$?" == "0" ] && sudo chown bigred:root -R ${CGROUP}/cpuset/first
  [ "$?" == "0" ] && echo "${CGROUP}/cpuset/first 建立和設定成功!"
else
  echo "${CGROUP}/cpuset/first 已存在!"
fi

# 關閉 CPU 直接存取記憶體的功能
if echo "0" | sudo tee ${CGROUP}/cpuset/first/cpuset.mems &>/dev/null; then
  echo "已關閉 CPU 直接存取記憶體的功能"
else
  echo "關閉 CPU 直接存取記憶體的功能 not ok , Error !!!" && exit 1
fi

# 以 root 來建立 low 控制子群組，low 繼承了 cpu controller 的所有屬性
cpulow="512"
if sudo mkdir ${CGROUP}/cpu/low &>/dev/null; then
  sudo chown bigred:root -R ${CGROUP}/cpu/low
  [ "$?" == "0" ] && echo "$cpulow" | sudo tee ${CGROUP}/cpu/low/cpu.shares &>/dev/null
  [ "$?" == "0" ] && echo "已將 low 控制子群組分配比率設成 $cpulow"
fi

# 建立 cpu 的 high 控制子群組
cpuhigh="2048"
if sudo mkdir ${CGROUP}/cpu/high &>/dev/null; then
  sudo chown bigred:root -R ${CGROUP}/cpu/high
  [ "$?" == "0" ] && echo "$cpuhigh" | sudo tee ${CGROUP}/cpu/high/cpu.shares &>/dev/null
  [ "$?" == "0" ] && echo "已將 high 控制子群組分配比率設成 $cpuhigh"
else
  echo "${CGROUP}/cpu/high existed !"
fi


# 檢視原本 cpuset 裡的設定：
core=$(cat ${CGROUP}/cpuset/cpuset.cpus | grep -e "0-1")
if [ "$core" == "0-1" ]; then
  echo "有二個 cpu core, 分別是 0 和 1"
else
  echo "請確認 cpu core : ${core}" && exit 1
fi


# 設定 first 控制子群組, 將 first 控制子群組設成第一個 cpu core
if echo 0 | sudo tee ${CGROUP}/cpuset/first/cpuset.cpus &>/dev/null; then
  echo "已成功將 first 控制子群組設成第一個 cpu core !"
else
  echo "設定 first 控制子群組設成第一個 cpu core，出現異常，Error !!!" && exit 1
fi

# 啟動 Namespace 的功能
echo ""; read -s -n1 -p "按任意鍵啟動 sh unshare 電腦"
clear; sudo unshare --pid --fork --mount-proc --net --uts -R ~/ov22/merged sh
```
## suc.sh 子程式，設定完後container還是要自己設定ip
```
#!/bin/bash

# var
shPID=$(pidof sh)

# daemon
until [ "$shPID" != "" ]
do
  shPID=$(pidof sh)
  sleep 1
done

sleep 5

# var
shNPID=$(sudo lsns -t net | fmt -u | tail -n 2 | head -n 1 | cut -d" " -f 1)

# 將 sh process 設定只能使用 1 G 的限制
if echo ${shPID} | sudo tee /sys/fs/cgroup/memory/demo2/tasks &>/dev/null; then
  echo "${shPID} process 受到記憶體 1G 的限制"
else
  echo "${shPID} process set memory error !"
fi

# 將 sh process 的 PID 加入到 first 控制子群組
if echo ${shPID} | sudo tee /sys/fs/cgroup/cpuset/first/tasks &>/dev/null; then
  echo "${shPID} process 只能使用 CPU Core 0 "
fi

# 新增 ve1 虛擬網路卡到 Default Namespace
# 新增 ve2 虛擬網路卡到 ${shNPID} 的 Network Namespace
# 以上這兩張網路卡會透過 ip link 用一條網路線連接起來
# veth ，Virtual Ethernet Device
# peer ，一組的意思
if ! ifconfig -a ve1 &>/dev/null; then
  sudo ip link add ve1 netns 1 type veth peer name ve2 netns ${shNPID}
  [ "$?" == "0" ] && echo -e "成功新增 ve1 虛擬網路卡到 Default Namespace \n成功新增 ve2 虛擬網路卡到 ${shNPID} 的 Network Namespace"
else
  echo "ve1 and ve2 existed !"
fi

# 將 ve1 這張網路卡啟動，因為內定網路卡沒有啟動
if ! ifconfig vel &>/dev/null; then
  sudo ip link set ve1 up && echo "ve1 網路卡啟動成功 !"
else
  echo "ve1 網路卡已啟動 !"
fi

# 設定 ve1 網路卡 IP 位址
if ! ip -4 addr | grep ve1 &>/dev/null; then
  sudo ip addr add 192.168.1.100/24 dev ve1 && echo "已設定 ve1 網路卡的 ip 位址為 : 192.168.1.100/24"
else
  echo "ve1 ip existed !" && exit 1
fi
```
:::

:::spoiler scp自動化
```
#!/bin/bash
file=$1
start=$2
end=$3
for ((start;start<=end;start=start+1))
  do
   sshpass -p 'srerbean' scp $file rbean@120.96.143.${start}:C:/users/rbean/Desktop &> /dev/null
   sshpass -p 'srerbean' ssh -o ConnectTimeout=1 rbean@120.96.143.${start} 'echo' &> /dev/null
   [ $? == 0 ] && echo "Success:120.96.143.${start}" || echo "= Failure:120.96.143.${start}"
  done
```
:::

:::spoiler 檢測 windows 10 硬碟空間容量
```
#!/bin/bash
net=$1
start=$2
end=$3
for ((start;start<=end;start=start+1))
  do
   volume=$(sshpass -p '!QAZ2wsx   ' ssh -o ConnectTimeout=1 bigred@${net}.${start} 'fsutil volume diskFree C:' 2> /dev/null | cut -d '(' -f 2 | head -n 1 | sed 's/)//g')
   sshpass -p '!QAZ2wsx   ' ssh -o ConnectTimeout=1 bigred@${net}.${start} 'echo' &> /dev/null
   [ $? == 0 ] && echo "- Success:${net}.${start} Free space: $volume" || echo "= Failure:${net}.${start}"
  done
```
:::


:::spoiler `cgitbl.sh`
```
#!/bin/bash
echo "Content-type: text/html"
echo ""
echo '<html><body>'
echo 'Hello From cgitbl.sh'
echo '<pre>'
dt=$(echo $QUERY_STRING | cut -d '=' -f2 | cut -d '&' -f1)
select=$(echo $QUERY_STRING | cut -d '=' -f3 | cut -d '+' -f1)
action=$(echo $REQUEST_URI | cut -d '=' -f3 | sed 's/+/ /g' | sed 's/%28/(/g' | sed 's/%29/)/g' | sed 's/%2C/,/g' | sed 's/%3B/;/g' | sed "s/%27/'/g" | sed 's/%22/"/g' | sed 's/%3D/=/g' | sed 's/%0D%0A/ /g')
user=bobo
passwd=bobo123
ip=120.96.143.84

case $select in
  select)
  echo -e "<h1 style="color:red">\nNo select command</h1>"
  ;;
  *)
  a=$(mysql -u "$user" -p"$passwd" -h "$ip" -P 3306 -e "use $dt;$action")
  echo "<h1>$a</h1>"
  if [ "$?" == "0" ]
  then
    echo -e "<h1 style="color:blue">\nCommand success</h1>"
  else
    echo -e "<h1 style="color:red">\nCommand fail</h1>"
  fi
  ;;
esac
echo '</pre>'
echo '</body></html>'
exit 0
```
:::



:::spoiler cgi進版
```
#!/bin/bash
echo "Content-type: text/html"
echo ""
echo '<html><body>'
echo 'Hello From cgitbl.sh'
echo '<pre>'
dt=$(echo $QUERY_STRING | cut -d '=' -f2 | cut -d '&' -f1)
select=$(echo $QUERY_STRING | cut -d '=' -f3 | cut -d '+' -f1)
action=$(echo $REQUEST_URI | cut -d '=' -f3 | sed 's/+/ /g' | sed 's/%28/(/g' | sed 's/%29/)/g' | sed 's/%2C/,/g' | sed 's/%3B/;/g' | sed "s/%27/'/g" | sed 's/%22/"/g' | sed 's/%3D/=/g' | sed 's/%0D%0A/ /g')
user=bobo
passwd=bobo123
ip=120.96.143.84

case $select in
  select)
  echo -e "<h1 style="color:red">\nNo select command</h1>"
  ;;
  *)
  a=$(mysql -u "$user" -p"$passwd" -h "$ip" -P 3306 -e "use $dt;$action")
  echo "<h1>$a</h1>"
  if [ "$?" == "0" ]
  then
    echo -e "<h1 style="color:blue">\nCommand success</h1>"
  else
    echo -e "<h1 style="color:red">\nCommand fail</h1>"
  fi
  ;;
esac
echo '</pre>'
echo '</body></html>'
exit 0
```
:::

:::spoiler iot丟資料到hadoop優化版
```!
$ cat sta
#!/bin/bash

./sub > pid.txt &

read -n 1 -s -p '按任意鍵停止'
echo ""
pid=$(cat pid.txt)
sudo kill -9 $pid &>/dev/null
# 丟資料到hadoop
hdfs dfs -ls /raw/powermeter/powermeter &>/dev/null
if [ "$?" != "0" ]
then
  hdfs dfs -put powermeter /raw/powermeter/
else
  hdfs dfs -appendToFile powermeter /raw/powermeter/powermeter
fi
echo ""
```

```!
$ cat sub
#!/bin/bash
[ -f powermeter ] && rm powermeter

echo $$
while read message
do
   echo "$message" | grep iotdata/powermeter/J409-178 | cut -d " " -f2 >> powermeter
done < <(mosquitto_sub  \
-v \
-t '#' \
-h 'mj409.oc99.org' \
-p 1883 \
-u "bigred" \
-P 'bigred')
```
:::

:::spoiler cert(製作憑證) shell script
```shell=!
$ cat mk
#!/bin/bash

dns=$2
ip=$3

help()
{
  cat <<EOF
Usage: mk [OPTIONS]

Available options:

create    create [DNS] [IP]
delete    delete cert
test      test
EOF
  exit
}

ssl()
{
openssl genrsa -aes256 -passout pass:password -out ca-key.pem 4096

openssl req -new -x509 -sha256 -days 365 -subj "/C=TW/ST=Taipei/L=Taipei/O=test/OU=lab/CN=example" -passin pass:password -key ca-key.pem -out ca.pem

openssl genrsa -out cert-key.pem 4096

openssl req -new -sha256 -subj "/CN=example" -key cert-key.pem -out cert.csr

echo -e "subjectAltName=DNS:${dns},IP:${ip}\nextendedKeyUsage = serverAuth" > extfile.cnf

openssl x509 -req -sha256 -days 365 -passin pass:password -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial
}

de()
{
rm ca-key.pem ca.pem ca.srl cert.csr cert-key.pem cert.pem extfile.cnf &>/dev/null
if [ "$?" == "0" ];then
  echo "delete all cert ok!"
else
  echo "delete cert fail,please check!"
fi
}

ts()
{
openssl verify -CAfile ca.pem -verbose cert.pem
}


case $1 in
  create)
    if [ "$#" == "3" ];then
      ssl
    else
      help
    fi
  ;;
  delete)
    de
  ;;
  test)
    ts
  ;;
  *)
    help
  ;;
esac
```
:::

:::spoiler 自動更新 image 到 docker.io

```
#!/bin/bash

set -x

images=(
  "docker.io/library/alpine"
  "docker.io/library/nginx"
  "docker.io/library/ubuntu"
  "registry.suse.com/bci/bci-base"
  "docker.io/library/busybox"
  "docker.io/library/mysql"
  "docker.io/library/mariadb"
)

quay_images=(docker.io/taiwanese/)

sudo podman login docker.io -u taiwanese -p andy6560588
for base_image in "${images[@]}"; do
  img=$(basename "$base_image")
  pubimg=$(sudo skopeo inspect "docker://$base_image" | jq -r '.RepoTags[]' | sed 's/[,"]//g' | sort -u)
  myimg=$(sudo skopeo inspect "docker://$quay_images$img" | jq -r '.RepoTags[]' | sed 's/[,"]//g' | sort -u)
  diffimg=($(echo "${pubimg[@]} ${myimg[@]}" | tr ' ' '\n' | sort | uniq -u | grep -v "sha256*" | grep -v "artifacthub.io" | grep -v "15.3.17.17.12"))

  if [[ -n "$diffimg" ]]; then
    for tag in ${diffimg[@]}
    do
      sudo podman pull $base_image:$tag
      sudo podman tag $base_image:$tag $quay_images$img:$tag
      sudo podman push $quay_images$img:$tag --remove-signatures
      sudo podman rmi $base_image:$tag $quay_images$img:$tag
    done
    sudo podman pull $base_image:latest
    sudo podman tag $base_image:latest $quay_images$img:latest
    sudo podman push $quay_images$img:latest --remove-signatures
    sudo podman rmi $quay_images$img:latest $base_image:latest
  fi
done
if [[ "$?" == '0' ]]; then
  echo "$(date),docker.io success" >> ~/dockerpush.txt
else
  echo "$(date),docker.io fail" >> ~/dockerpush.txt
fi

# Logout and prune (outside loop for efficiency)
sudo podman logout docker.io
```
:::
:::spoiler 自動更新 image 到 quay.io
```
#!/bin/bash

set -x

images=(
  "docker.io/library/alpine"
  "docker.io/library/nginx"
  "docker.io/library/ubuntu"
  "registry.suse.com/bci/bci-base"
  "docker.io/library/busybox"
  "docker.io/library/mysql"
  "docker.io/library/mariadb"
)

quay_images=(quay.io/cooloo9871/)

sudo podman login quay.io -u cooloo9871 -p andy6560588
for base_image in "${images[@]}"; do
  img=$(basename "$base_image")
  pubimg=$(sudo skopeo inspect "docker://$base_image" | jq -r '.RepoTags[]' | sed 's/[,"]//g' | sort -u)
  myimg=$(sudo skopeo inspect "docker://$quay_images$img" | jq -r '.RepoTags[]' | sed 's/[,"]//g' | sort -u)
  diffimg=($(echo "${pubimg[@]} ${myimg[@]}" | tr ' ' '\n' | sort | uniq -u | grep -v "sha256*" | grep -v "artifacthub.io" | grep -v "15.3.17.17.12"))

  if [[ -n "$diffimg" ]]; then
    for tag in ${diffimg[@]}
    do
      sudo podman pull $base_image:$tag
      sudo podman tag $base_image:$tag $quay_images$img:$tag
      sudo podman push $quay_images$img:$tag --remove-signatures
      sudo podman rmi $base_image:$tag $quay_images$img:$tag
    done
    sudo podman pull $base_image:latest
    sudo podman tag $base_image:latest $quay_images$img:latest
    sudo podman push $quay_images$img:latest --remove-signatures
    sudo podman rmi $quay_images$img:latest $base_image:latest
  fi
done
if [[ "$?" == '0' ]]; then
  echo "$(date),quay.io success" >> ~/quaypush.txt
else
  echo "$(date),quay.io fail" >> ~/quaypush.txt
fi

# Logout and prune (outside loop for efficiency)
sudo podman logout quay.io
```
:::

:::spoiler danger
```
#!/bin/bash


while true
do
  yes & &>/dev/null
  cat /dev/zero | head -c 1000m | tail &
done
```
:::


:::spoiler 指定 image 清單，自動下載上傳到 harbor
```
$ nano image_sync.sh
#!/bin/bash

#set -x

RED='\033[1;31m' # alarm
GRN='\033[1;32m' # notice
YEL='\033[1;33m' # warning
NC='\033[0m' # No Color

HARBOR_URL=harbor.k8sexample.com
HARBOR_REPO=library
HARBOR_USER=admin
HARBOR_PASSWD=Harbor12345


[[ ! -f ./images.txt ]] && printf "${RED}images.txt file not found${NC}\n" && exit 1

sudo -n true &>/dev/null
if [[ "$?" != "0" ]]; then
  printf "${RED}Passwordless sudo is NOT enabled${NC}\n" && exit 1
fi

if ! which docker &>/dev/null; then
  printf "${RED}docker command not found${NC}\n" && exit 1
fi

printf '%s' "$HARBOR_PASSWD" | sudo  docker login "$HARBOR_URL" -u "$HARBOR_USER" --password-stdin &>/dev/null
if [[ "$?" == '0' ]]; then
  printf "${GRN}Login $HARBOR_URL success${NC}\n"
else
  printf "${RED}Login $HARBOR_URL fail${NC}\n" && exit 1
fi



while IFS= read -r line
do
  img=$(echo "${line##*/}" | cut -d'@' -f1)
  sudo docker pull "$line" &>/dev/null &&
  sudo docker tag "$line" ${HARBOR_URL}/${HARBOR_REPO}/${img} &>/dev/null  &&
  sudo docker push ${HARBOR_URL}/${HARBOR_REPO}/${img} &>/dev/null &&
  sudo docker rmi "$line" ${HARBOR_URL}/${HARBOR_REPO}/${img} &>/dev/null
  if [[ "$?" == 0 ]]; then
    printf "${GRN}Push ${HARBOR_URL}/${HARBOR_REPO}/${img} success${NC}\n"
  else
    printf "${RED}Push ${HARBOR_URL}/${HARBOR_REPO}/${img} fail${NC}\n"
  fi
done < <(sed -e 's/#.*//' -e 's/^[ \t]*//' -e 's/[ \t]*$//' -e '/^$/d' images.txt)
```

```
$ nano images.txt
# Please specify the full name of the image.
docker.io/library/busybox:latest
docker.io/library/nginx:stable-perl
quay.io/cooloo9871/alpine:3.22.0
```

```
$ ./image_sync.sh
Login harbor.k8sexample.com success
Push harbor.k8sexample.com/library/busybox:latest success
Push harbor.k8sexample.com/library/nginx:stable-perl success
Push harbor.k8sexample.com/library/alpine:3.22.0 success
```

:::


:::spoiler 指定 deb 套件，自動下載上傳到 NEXUS
```
#!/bin/bash
#set -x

RED='\033[1;31m' # alarm
GRN='\033[1;32m' # notice
YEL='\033[1;33m' # warning
NC='\033[0m' # No Color


NEXUS_URL=http://10.10.7.20:8081
NEXUS_USER=admin
NEXUS_PASSWD=!QAZ2wsx@WSX!!
NEXUS_BROWSE=apt-host


while IFS= read -r line
do
  curl -u "${NEXUS_USER}:${NEXUS_PASSWD}" -H "Content-Type: multipart/form-data" --data-binary "@./${line}" "${NEXUS_URL}/repository/${NEXUS_BROWSE}/" &>/dev/null
  if [[ "$?" == 0 ]]; then
    printf "${GRN}Push ${line} success${NC}\n"
  else
    printf "${RED}Push ${line} fail${NC}\n"
  fi
done < <(ls $pwd | grep .deb)
```
:::
###### tags: `程式語言`