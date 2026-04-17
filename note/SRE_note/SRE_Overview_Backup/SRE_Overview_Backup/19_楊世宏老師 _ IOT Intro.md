# 楊世宏老師 : IOT Intro

[TOC]



---


## IOT 講義連結

[講義連結](https://drive.google.com/drive/folders/19P5pijLjiZPojj9m29feK_0qL5VQ3sii?usp=sharing)



## IoT Solution Architecture



![](https://i.imgur.com/NedQ3n8.png)






## 機房監控系統

![](https://i.imgur.com/9XVl94K.png)

- 電流感測器
	- 型號：PZEM-004T
	- 安裝位置：一條電源延長線中間剪一段連上去，另外抽一條 USB 線，這條 USB 線就是用來傳送 Sensor 收集到的數據。
	- 接法：將延長線的電源接到牆壁插座，延長線的插座接要偵測的電器，只要<font color=red>將想偵測的電器接到延長線的插座就可以收集到數據</font>
	- USB 線：傳送 Sensor 收集到的數據到 Windows or Linux 機器（都可以），這台機器收到以後，會將資料傳到雲端，這台機器就是扮演 IOT Gateway 的角色。
	
:::spoiler 電流感測器圖片

![](https://i.imgur.com/H3s7POR.jpg)

:::


:::spoiler 距離感測器圖片

![](https://i.imgur.com/JaDZGKZ.jpg)
![](https://i.imgur.com/nQPpnmJ.jpg)

:::

## IOT Gateway

定義：
- 能夠處理和蒐集 IoT Device 的資料
- 具備網際網路傳輸能力的機器

## IOT Device

- 又稱為 Sensor
- 通常不具有網際網路的傳輸能力。
- 有些人會直接在 Sensor 上面加工，讓它具有網際網路傳輸的能力，缺點是該 Sensor 就只能傳它自己的資料，不能同時接多個 Sensor 。

## 傳輸方法

![](https://i.imgur.com/PUdjzRg.png)

從 IOT Gateway 傳到 MQTT broker ，是透過 MQTT 通訊協定

:::spoiler MQTT 通訊協定

:::info
- 主要是用來傳輸資料，傳輸資料消耗的流量低。
- Client 端會與 Server 端建立通道，一旦通道建立成功，除非 Clinet 端主動結束通道，不然通道不會結束。
- Server 端可以主動傳資料到 Client 端，不需得到 Client 端同意。
- <font color=red>**Server 無須知道 Client 端的 IP 位址也能進行即時的雙向溝通**。</font>
- ![示意圖](https://i.imgur.com/LMrQeX0.png)

:::

- MQTT Broker
	- 它是 MQTT 的中心，允許 Clinet 端收集和傳送資料。
	- 做資料的中轉，所有設備會將資料傳到 broker 上面， broker 會根據 Subscriber 的需求，將資料轉送給 Subscriber (資料科學家)

- MQTT Client
	- Publisher ，代表 IOT Gaeway ，發布者
		- 可自定義 Topic 
	- Subscriber ，資料科學家，訂閱者
		- Topic 設為 `#` ，代表一次訂閱 Broker 的所有內容
- Topic 
	- 資料的分類
	- 例如：`iotdata/powermeter/{MQTT_Broker_hostname}`
- Payload
	- Topic 的內容

:::danger
- 重點整理
	- clinet（ 資料科學家 ） 與 client ( IOT Gayeway ) 溝通會需要透過 MQTT 的 Broker。
	- Topic 由發布者定義。
	- broker 本身不會儲存資料，他是中間的轉寄站。
	- Subscriber 一定要先訂閱 Topic 才會接收到 Publisher 發布的 Topic 資料。
	- MQTT Broker 收到 Publisher 發布的資料後，發現沒有 Subscriber 訂閱，則將資料丟掉。
:::



---

# MQTT Client 端命令

- Mosquitto 
	- 由 Eclipse 公司設計的 program
	- 讓我們可以扮演 MQTT client 的角色
	- Ubuntu Linux 安裝方法：`sudo apt install mosquitto-clients`
	- Alpine Linux 安裝方法：`sudo apk add mosquitto-clients`

:::success
## MQTT Subscribe 命令

```bash=
mosquitto_sub -v -t 'iotdata/powermeter/J509-12' \
-h 'pu230.oc99.org' \
-p 1883 \
-u 'bigred' \
-P 'bigred'
```


`-v` 詳細資訊，收到的資訊會看到資料是哪個 Topic
`-t` Puberlisher 端的 Topic 
`-h` MQTT broker 網址
`-p` port 號

以下為連接到 broker 使用的帳號密碼：
`-u` 帳號
`-P` 密碼

**執行完命令後，看到卡住是正常的，因為他正在等待 Puberlisher 傳資料給他。**
:::


:::success


## MQTT Publish 命令

```bash=
mosquitto_pub -t 'iotdata/powermeter/J509-12' \
 -h 'pu230.oc99.org' \
 -p 1883 \
 -u "bigred"\
 -P 'bigred' \
 -m 'Hello'
```

`-m`  Topic 內容
:::



---


# Install MQTT Broker on alpine

[文件連結](https://github.com/Oscar-Young/Mosquitto-alpine-podman/blob/main/README.md)

## prepare

```
$ sudo apk update && sudo apk add podman 

$ sudo  rc-update add cgroups

$ sudo rc-service cgroups start
```


## Create work directory

```
$ mkdir iot && cd iot 
```

## Setting mosquitto config file and password_file

Create Config file


```
$ nano mosquitto.conf

listener 1883
allow_anonymous false
password_file /mosquitto/data/password_file
```

Create password_file

```
$ touch password_file
```

Create mosquitto container

```
$ sudo podman run -itd \
 --restart=always \
 --name mosquitto \
 -p 1883:1883 \
 -v ./mosquitto.conf:/mosquitto/config/mosquitto.conf \
 -v ./password_file:/mosquitto/data/password_file \
 eclipse-mosquitto
```

## Create mosquttio user

Use mosquitto_passwd command to create user, please replace {USERNAME} and {PASSWORD} to you wanted.

```!
# Add user
$ sudo podman exec -it mosquitto mosquitto_passwd -b /mosquitto/data/password_file {USERNAME} {PASSWORD}

# Reload password_file
$ sudo podman exec -it mosquitto kill -SIGHUP 1
```


---

# Publisher Program

[SmartHost - 世宏老師的 Github 連結](https://github.com/Oscar-Young/SmartHost)




















###### tags: `IOT`





