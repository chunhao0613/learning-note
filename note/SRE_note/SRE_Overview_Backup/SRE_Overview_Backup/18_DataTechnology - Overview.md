# DataTechnology - Overview

:::warning

:::spoiler 目錄 (  :mag_right: 點我，可展開 )

[TOC]

:::

## Information Technology

![](https://i.imgur.com/6gSEqDj.png)

- **OLTP｜線上交易處理｜Online transaction processing**
    - Transactional 代表 4 大動作
        - crud，新增,修改,刪除,查詢，多人同時去執行這幾個動作 create read update delete
    - lock
當有一個人在改資料內容，系統會鎖住那筆資料，等他改完，其他人才能看到那筆資料
    - rollback
    以銀行轉帳為例，當A帳戶轉錢給B帳戶的過程中，因為動物攻擊線路造成停電，Ａ轉錢轉到一半無法成功，這時候系統就會rollback倒退回去更改回去的狀態，A帳戶扣掉的錢會再次新增回來
    - commit
完成，當A轉戶成功轉錢給B帳，Ａ帳戶扣錢，Ｂ帳戶前增加，這兩個動作完成，就會顯示commit
    - OLTP 沒辦法同時處理很大量的資料，因為同時多人在做 Transaction 的 4 個動作，所以沒辦法同時處理很大量的資料
- 以上這些 Transaction 動作會由 Transcation Log Reader 紀錄。
- OLTP 應用程式 : MS SQL, MySQL, Oracle
- 企業買電腦為了 run application 會裝防火牆、防毒軟體，為的就是 run 的長長久久。
- 如何做備份？
    - Transaction Log Reader
Lod Reader 意思是說，Server本身會有做一個紀錄，全程紀錄，會有一個log的存放區
會有另一個程式，去讀取 Transaction log 檔，這個程式就叫 CDC ( Change data capture ) ，只要這個 log 檔一有變動，就會馬上把資料抓出來，把抓到的資料做整理，整理後再透過 Stream Loader 丟到 OLAP 進行重播，譬如在 OLTP 這個 DATABASE 新增一筆資料，CDC 抓到了這個 log 檔，會在 OLAP 這裡的資料庫也新增一筆資料，只要硬體設備夠強，可以做到即時作業
- Bulk Loader
批次備份，通常會在半夜進行。

OLAP｜線上分析處理｜Online Analytical Processing
- 在這台資料庫伺服器裡面，只會用到 read，不會做修改，主要的功能是用來做商業分析，用的命令像是 select...等
- Data Warehouse 
- Business Intelligence 商業智慧
- 在 select 之前，一定要先 create 
- schema on write
    - 先產生 Database 及 Table，才能做後續的資料儲存及分析


> Teradata ，一間公司，它的軟體和硬體拿來做 BI 很強，但很貴
「Business Intelligence」
現在的公司使用資料庫就是為了能夠
分析出有效、有價值的決策報告，
但是 teradata 天睿這家公司出版的 Teradata 資料庫，
不只軟體價格變高，還要求使用指定的硬體設備，
讓很多公司受不了

:::spoiler VMware 網路複習


![](https://i.imgur.com/UAMa2jC.png)!


VMware 網路模式
Host｜不可上網，僅限虛擬網路內部通訊 
Brigde｜可以上網，藉由橋接器領取與主機相同網段的 IP Address
NAT｜可以上網，透過封包轉送實現虛擬網路上網

>route r負責讓要送的封包，讓他傳到外面的網路
NAT 會把封包的 Source IP 位址進行更改，才能上網，所以和 router 不一樣
VMware player 的 3 種網路模式，事實上都有一個虛擬橋接器，NAT模式因為有開啟轉址功能才可以上網，而 Host only 沒有開啟轉址功能，所以不能上網

> 當整間教室的所有 windows 主機都用 VMware 啟動一台 bridge 網路的虛擬機，其實就等同於一個私有雲的架構，

> 公有雲，以 GCP 為例，他的實體電腦用的是什麼作業系統，用什麼虛擬技術建立虛擬機器？
> Google 絕對不會使用 windows 作業系統，因為他們可以自己建立作業系統，用 KVM 的虛擬技術起虛擬機器，不過他的虛擬技術不一定要叫 KVM

之後要教的 Hadoop, Spark 和 Hbase，可以讓我應徵資料管理師，資料分析師和資料科學家

--- 

:::


---


## DataTechnology

![](https://i.imgur.com/hX7YE5D.png)
++**新世代的資料科技｜DataTechnology (DT)**++
- 資料來源（data lake）： 
    - 傳統的 IT 系統的資料，例如 : MS SQL, MySQL, Orcale 的資料
    - OpenData ｜ 開放資料平台，不會有 license 問題
        - 缺點 : 資料不整齊，亂七八糟。
    - IoT - Internet of Things ｜ 萬物皆連線，用於環境監測、智慧居家、自動化工業... 
    - 企業內部資料｜同一間公司，不同部門的文本資料、營運資料... 
    - 爬蟲資料｜到網路上去抓取特定資料
    - 黑資料 | 私下用錢買賣資料
- 以上的資料都會上載到 Hadoop

Dara lake (資料湖) 和 資料倉儲 不一樣，資料倉儲是已經先做初步分類; 資料湖則是在資料先進到湖裡，在湖旁邊蓋很多工廠再做處理

第一個工廠
    Apache Hive，讓我們可以用 SQL command，對資料湖做分析

第二個工廠
    Spark，做深度分析，機器學習
    Spark Straming ， 分析 IOT 的資料

第三個工廠
    Hbase，可以承接 IoT 的資料，以及 nosql 的資料
    

資料庫的核心就是Hadoop，



以 Hadoop 為主 + Hive 就可以做出企業要的 BI

- 請問可以說是因為關聯式資料庫使用上成本較高，所以大家才開始使用 hadoop 嗎?
    - 答 : 大單位在用傳統關聯式資料庫的維運成本才會比較高，小單位的部分並不會有這部分的問題，故學 Hadoop 可以為有一定規模的公司解決維運傳統關聯式資料庫成本高的問題。




---


![](https://i.imgur.com/tOo19Un.png)  

- <font color=red>**Hadoop 被設計出來，用來處理 RAW DATA（原始資料）**</font>，像是氣象局的氣象資料或是火車的誤點資料，RAW DATA 的資料不能被修改
- Raw Data (原始資料) : 一但產生，即不可修改 (例: 火車時刻表、各區域排碳量、機場出入境人次、感測器所蒐集的資料)
- 4V
    - VELOCITY，這種 RAW DATA 是一種"串流資料"，一段時間就會有資料輸入，有分為高中低3種速度 高速:微秒，中速:秒，低速:分
    - Variety，企業 IT 系統的紀錄檔（ log 檔），舉例，醫院的病歷歷史資料, Open Data （政府資料開放平臺）,社交網站的資料（ Facebook, Line ）
    - Volune，台灣的大數據資料量單為是 Ｔ，
    - Veracity，資料真實性，老師翻成品質，因為資料越真，品質越高


---


## Hadoop Architecture

![](https://i.imgur.com/WaXUf3b.png)


-   **Hadoop Common**: The common utilities that support the other Hadoop modules.
    - 對於 hdfs, YARN, MapReduce 的管理命令和應用命令
- **Hadoop Distributed File System (HDFS™)**，分散式檔案系統，**這套檔案系統在運作一定要多台電腦**
    > windows 的檔案系統為：NTFS or FAT32 ，只能在一台電腦上面跑，所以檔案系統的大小是有限制的。
- **YARN ( Yet Another Resource Negotiator )**，運算系統，專門來跑資料處理引擎的程式，可以平行處理，意思就是說可以在多台電腦同時讓 Mapreduce 和 Spark 跑程式分析資料
    > Hadoop 算不算一個資料型的作業系統？
    > 是，因為它具備資料儲存、運算能力
    > windows 裡，執行程式的平台叫做 `.net`，其中有一個 msSQ
- **MapReduce**，內建資料處理引擎，用來做資料分析
- **Spark**，（可加裝）也是資料處理引擎
- **Hive**，會將 SQL Command 翻譯成 Map, Reduce 程式，最後得出分析結果


[Hadoop 官網連結](https://hadoop.apache.org/)


### Related projects (Hadoop的親戚)

- Hbase 資料庫，一定要使用 HDFS 檔案系統來儲存資料庫的 DATA
- Hive，內定使用 Mapreduce 資料處理引擎，來分析資料
- pig 豬小弟，內定會使用 Mapreduce 來做資料整理
- Spark 是 Hadoop 的朋友，不是親戚，他們是在一場比賽中認識的， Spark 他的強項在於 RDD 資料處理引擎，表現比 Mapreduce 強很多，但底層一樣要有存儲檔案系統和運算系統， Spark 跑到公有雲（GCP,AWS, Azure），可以使用公有雲的存儲檔案系統和運算系統; 在 Hadoop 上跑的話，需要有底層( HDFS、YARN ) support 才可運行


> 雅馬遜的檔案系統 S3


---


## Hadoop Cluser 架構

![](https://i.imgur.com/xlo4uIi.png)

- Hadoop 一定由多台電腦以及上面跑的 Java 程式組成。
- dta1，這台電腦給資料管理師，資料工程師，資料分析師，資料科學家用的
    > 叢集 cluster，多台電腦
- HDFS 檔案系統的組成：
    - dtm1 裡面有 Secondary Name Node 和 Namenode ( 兩個 Java程式 )
    - dtw1, dtw2, dtw3 三台電腦上面都有 run Data node（Java程式）
- YARN 由以下組成：
    - dtm2 裡面的 Resource Manager ( Java 程式 )
    - dtw1, dtw2, dtw3 三台電腦上跑的 Node Manager ( Java 程式 )
- Hadoop 叢集總共會有 7 台電腦，Client 一台，Secondary NameNode 一台，Active Namenode 一台，Data Node ＋ Node Manager 4 台
> Standby NameNode 是獨立的Active Name Node 備援機


> 在 Open Source 的領域要看功能與支援程度，不是一昧追最新的版本
Hadoop 2.10.1 這個版本在Apache裡面，跟很多頂級專案是朋友，所以功能會比較多

[Apache.org 裡面都是頂級專案](https://apache.org/)


---

{%hackmd K3xvoyRCRNS-b4-sPS1W_Q %}
{%hackmd zObYmETOSbS--aHj4Ycbaw %}
{%hackmd cVAzCcAfQE2919WvyX_1-w %}
{%hackmd 5O2EFLOOTlKRNc498z6JDQ %}
{%hackmd sH4pBq1fTiKKDPqM2xXjsg %}



###### tags: `資料科技平台`

