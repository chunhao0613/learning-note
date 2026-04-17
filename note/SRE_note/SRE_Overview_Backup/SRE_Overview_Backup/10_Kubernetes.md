---
title: Kubernetes
tags: 系統工程
---

# 雲原生世代最佳舵手 Kubernetes


## Kubernetes-Oerview

:::warning

:::spoiler 目錄

[TOC]


---


:::

![](https://i.imgur.com/FZgie9K.png)

Kubernetes 新一代的雲端作業系統
- 管理實體電腦或虛擬電腦裡面的多台 Container
- 由 Google 公司設計

## 單純使用 Podman/Docker 會面臨的挑戰

1. 單點毀損
Podman/Docker 啟動的 App Container 只會在一個機器上，如果該主機掛了，這個 Application 怎麼辦?
我們希望 Application 擁有容錯率，他能夠在另外一台 Podman/Docker 主機啟動一樣的 App Contianer，但 Podman/Docker 沒辦法自動做到這件事，只能靠工程師手動在另外一台機器上去做部署。
2. 高可用性(High Availability)
High Availability 的意思是，同一個 Applcation 在不同機器上同時都在運作，可以想成本尊跟分身的概念，如果一台電腦故障了，另外一台電腦上的 App 還是能持續提供服務。
3. 自動橫向擴充
假設當前的 Application 只有一個本尊跟分身，但萬一今天下午發生了一些事件，譬如應用系統是一個售票網站，在下午三點開賣，一堆流量灌進來，這時候我們就會希望有一個系統來會自動幫我們做到橫向擴充，英文專有名詞叫 Auto-scaling，也就是自動多長幾個分身出來一起提供服務。
4. Rolling updates
為了要讓企業的 Application 能達到長長久久穩穩當當，就還會需要 Rolling updates，因為任何一個企業開發的應用系統，一定會有第一代、第二代、第三代，會新增功能或刪除功能，也就會產生不同的版本，這時候就會需要系統要可以自動幫我們做進版跟退版。
5. 資料儲存(Storage)
Application 在運作的時候，一定會有資料的產生，所以還要幫 app 設計資料永存的解決方案，要讓 app 的資料能夠存的穩穩當當。

## **Kubernetes 最高戰略目標**

:::danger
**幫企業 run Application ( 應用系統 ) run 的長長久久、穩穩當當**
:::

## Kubernetes 的能力介紹

Containerisation (貨櫃化) has brought a lot of flexibility for developers in terms of managing the deployment of the applications. However, the more granular the application is, the more components it consists of and hence requires some sort of management for those.

One still needs to take care of scheduling the deployment of a certain number of containers to a specific node, managing networking between the containers, following the resource allocation, moving them around as they grow and much more.

Nearly all applications nowadays need to have answers for things like

- **Replication of components (Pod)**
    - 同樣的 pod 可以複製很多台出來 (本尊與分身的概念，分身也會做事)
    - 叢集，同一個服務會有好幾個 pod 來提供服務，避免單點損毀的情況，提高可用性 (HA) 與容錯
- **Auto-scaling** 
    - 自動橫向擴充，可以依照暴漲的流量擴充 pod
    - 一開始只做一個本尊和一個分身的 pod (兩個相同的應用系統再跑)，當偵測到同一時間有大量連線時，會自動長出一樣的應用系統出來一起作業，連線恢復正常時，多的分身會消失
- **Load balancing** 
    - 平衡負載，讓每個 pod 都能平均分工
- Replication of components + Auto-scaling + Load balancing 能達到**隨需擴增**
- **Rolling updates**
    - 滾動式更新，專門給程式設計師使用，因增加或修改功能會更新程式版本，也就是<font color=red>**進版或退版**</font>
    - 當 Application 部屬到測試環境的 K8S Cluster 時，可檢查程式碼有無死邏輯，當服務發現有異常時，程式會退版，確認都無問題後，再部屬到 Production 環境
    - Example: 藍綠部屬、金絲雀部屬
- **Logging across components**
    - 每台 pod 運作的 log 檔都會記錄下來
    - grafana ( 發音 : 寡發哪 ) ： 透過儀錶板來監控所有的 Container 
- **Monitoring and health checking**
    - 透過 log 來監控 pod 的狀態和對 pod 進行健康檢查
- 由於在整個 K8S Cluster 上面會跑多個應用系統，所以需要 Logging across components + Monitoring and health checking 來幫助我們維運
- **Service discovery (DNS Server)**
    - DNS Server 的功能 : 將人看得懂的網站網址 (URL)，變成 IP 位址
    - 由 CoreDNS System 得知，你的 Service 會對到哪一個 pod，會將 Service 的 Domain name 轉成綁在 Service 上的 pod 的 IP 位址。
    - [Domain name wiki link](https://zh.wikipedia.org/zh-tw/%E5%9F%9F%E5%90%8D)
- **Authentication**
    - 身分認證，透過憑證 ( 公鑰 and 私鑰 ) 作業
    - 在 K8s 中沒人在打帳號密碼，全靠憑證作業。

Google has given a combined solution for that which is Kubernetes, or how it’s shortly called – K8s.

**甚麼時候會需要用到隨需擴充?**
- 網站伺服器，遇到多人大量同時連線時會需要，例如 : 演唱會訂票系統, 台鐵在節慶時的訂票系統
- 擴充的方式 : 金絲雀 ( 之後會學的 )...等

## 雲原生作業系統 - Kubernetes 大架構

![](https://i.imgur.com/FuRjMvt.png)


這張圖中，有3台虛擬電腦
- kubernetes master 的 hostname : m1 
- Worker node 的 hostname : w1 和 w2
	- 可以視 Application 的複雜程度新增 Worker node 主機
- <font color=blue>**kubernetes 最小的運作單位為 pod ( 豆莢 )**</font>
	- pod 由一台或以上的 Container 組成
	- 同一個 pod 裡的多台 Container 會共用同一張網路卡，網路卡對內透過記憶體溝通（localhost），對外有同一個 Pod 的 IP 位址 。
	- 一個 pod 裡面 3 台 Container 跑的 Application（舉例，Container 1 跑 nginx 的網站; Container 2 跑 MYSQL 資料庫; Container 3 跑 log Server ）= > 可以理解為 3 個 Container 跑企業用的 Application 在一個 pod 裡面。


### Master node 

- Master node 裡一定會有 4 個 pod 分別是 `API Server` 、`Secheduler` 、`Controller-Manager` 和 `etcd`
- `API Server`
    - 負責接受前端管理者透過 CLI ( Command Line Interface, 命令模式 ) 下達的命令，例如 : `kubectl` 、`kubeadm`...等
    - 也接受 Web UI ，在瀏覽器的網頁上點點選選，對 `API Server` 下達命令，例如 : [Rancher](https://rancher.com/) 
        > Rancher 這家公司還有推出 K3s ( 無肥肉、身材好、精實型的 K8s )
    - Authentication 身分認證，你是誰
    - Authorization 授權，你有什麼權限，可以做什麼事情
- `Secheduler`
    - 負責安排 pod ( 企業的 application ) 要在哪一台工作主機上 run
    - 安排的方式不只看記憶體，會有更複雜的演算法，溝通的對象為該工作主機的 `kubelet`
- `Controller-Manager` 
    - 會負責照顧 node 裡面的 pod ，如果一台 node 裡面的 pod 掛了，它會去請 `Scheduler` 幫我們把 pod 建立回來，但是不一定會在原來的 node 把 pod 建立回來。 ( Kubernetes 內建的容錯功能 )
    - 會監控 pod 的狀態，如果發現很多 connection 連到 pod ，它就會請 `Sechedular`， 幫忙把 pod 複製出來，也就是隨需擴增。
- `etcd` 
    - 一個資料庫。
    - 存放整個 K8S 系統運作的重要資料，e.g., node 裡面會存/跑什麼 application, pod 叫什麼名字 ...等。


### Worker node

- `kubelet` 
    - 在背景執行的程序
    - 把 Container-Runtime-Engine (Podman, Docker, CRI-O) 做出來的一個或多個 Container 包成一個 pod
    - 實際上 Podman, Docker, CRI-O 會透過 runc/crun (有人根據 OCI 組織制定的 Runtime spec 寫出來的程式) 來將 Container 建立出來。
    - Docker 用的是 runc 來建立 Container，Podman 用的是 crun ，CRI-O 預設是用 runc
    - runc/crun 在建立 Container 時，一定會需要 bundle ( runtime 設定檔 + rootfs 檔案系統目錄 )
- `Kube-proxy` ( 哭 ㄅ ㄆㄨㄚ seeˇ )
    - 一個 pod
    - 讓不同 node 中 pod 的網路能互相連接 
    - 在我們的環境中，`kube-proxy` 會透過 Flannel 網路套件，來讓 Pods 跨主機互通有無

> <font color=blue>m1 只要執行 `kubelet` ，其實也可以扮演 worker 的角色</font>


### 情境敘述

- 我們 ( 前端管理者 ) 打 `kubectl` 這個命令告訴 `API Server`，我有一個 pod 要 run， `API Server` 接收到後，會去找 `Scheduler`，請他安排，`Secheduler` 會去找看哪一台 node 最閒，然後會去通知那台 node 裡面的 `kubelet` ，叫他去請 Container-Runtime-Engine 幫我們產生 Container 。

- K8s 的 pod 在外面工作，我們稱它為 Solution ( 解決方案 ) ，注意 ! 它不能稱為一台電腦，因為 pod 是由一台或多台 Container 組成， Container 是一台電腦，pod 就是很多台電腦堆疊起來的一整個機櫃，舉例 : 應用系統由一個或多個 process 組成，而 pod 就是一個企業所需的應用系統，pod 裡面的 Container 就是 process 和 process 的相依檔。


---

## Kubernetes CRI 的架構演進圖

![](https://i.imgur.com/Ub04GNj.png)

- 最上面是第一代的流程，因為 K8S 不紅，所以自己寫了一個 `Dockershim` 來跟 `Docker Engine` 來溝通，建立 Container ，透過 `Docker Engine` 來做到內部網路，所以速度慢，效率差。
- 第二代：把 `Docker` 給砍掉了，並且 `K8S` 自己寫了 `CRI-Cotainerd` 來跟 `Containerd` 做溝通，不依賴 `Docker Engine`
- 第三代：`Containerd` 討好 K8S ，直接提供 `CRI plugin` ( Container Runtime Interface, 內建一個介面，根據 CRI 標準寫的程式 ) 給 K8S，讓 K8S 可以直接使用 `Containerd`，但因為 `Containerd` 本身還有給 `Docker` 平台使用，所以制定 image 的標準也有 `Docker` 的，跟下面最新的架構比，效率還是差了點。
- 目前：使用 CRI-O 這個專案，特別為 K8S 量身訂做，直接翻譯 `kubelet` 的話，效率最優。
  - 而且直接對 runc 溝通，不用再經過好多流程才產生 container  
  - CRI-O 是 open source 的專案，老師目前使用最下面的架構，kubernetes 會直接透過 CRI-O 來跟 rnc 溝通產生 Container
<font color=red>
- 結論
    - K8S 不需要 `Docker` 就可以建立和跑 Container 了
    - 目前 Google Cloud 、 AWS 和 Microsoft Azure 三大公有雲使用的是第三代的架構，Containerd 直接和 kubernetes 做溝通請 runc 產生 Container。
    - `Docker` 並沒有遵循 CRI 的標準
</font>

---

# 選擇 Kubernetes Cluster

- Cluster 叢集
- kubernetes Cluster 就是 kubernetes 會用到的電腦，代表 K8S 是由多台電腦組成的作業系統


## Managed Kubernetes Services

意思是有人已經幫你設好了 Kubernetes
- **雲端商的 Kubernetes**  (<font color=red>**資料安全上會有問題**</font>)
	- Amazon Elastic Kubernetes Service | Amazon 做出來的 K8S ，簡稱 : EKS/Amazon EKS
	- Google Kubernetes Engine | Google 做出來的 K8S ，簡稱 : GKE
	- Azure Kubernetes Service | Microsoft 做出來的 K8S ，簡稱 : AKS
    - 服務供應商也會有像是 `quay.io` 的 image 管理中心，供使用者 `Pull`、`Push image`，還會有 `CI/CD` ，讓程式設計師有健身房可以用，達到敏捷開發。
    - 不同雲端供應商都追加了不同的系統功能，但其核心都是 Kubernetes Cluster
- **K8S 商軟（distribution (“distro”) of Kubernetes）**
    - 由大公司幫我們做好 K8S Cluster ，我們只需要付錢錢，就可已買回來自己管理
	- RedHat OpenShift | RedHat 做的 Kubernetes
	- VMware Tanzu ｜ VMware 做的 Kubernetes 
	- Rancher | K8S (前端用的是 Web UI，價格較親民) + K3S（比 K8S 更輕量，可用於物聯網。）
	- 還有很多 K8S 商軟 ...等
- **Self-Hosting Kubernetes**
	- 自己建置 Kubernetes。



---

# Kubernetes Cluster 核心運作

## Open Interfaces in Kubernetes

![](https://i.imgur.com/hG5rPqX.png)


- CRI Container Runtime Interface ， Kubernetes 建置 Container 的標準
    - 選用 CRI-O 的原因是因為，CRI-O 為了 K8S 而活，**效能最好**，支援多種 OCI Runtime , 不只有 runC/Crun ，還可以選用其他的，相容度很高
- CNI Container Network Interface ， 網路的標準
    - 讓不同 node 中的 pod 網路可以互通有無，還有讓別人可以透過網路連接到我們建立對外的 Service （ Routing Table and NAT and Iptables ）
    - 例如 : Flannel 網路套件
- CSI Container Storage Interface ， 儲存系統的標準，
    - 其實講的就是 Volume ， bypass overlay2，讓 pod 的資料可以達到永存。e.g., Rook、ceph、NFS (Net Work Filesystem)
- 陳老師建置出來的 K8S CRI 用的是 CRI-O 這個專案，CNI 用的是 Flannel 網路套件，CSI 用的是 NFS
以上3種標準會衍生出很多不同的專案，但這些專案都是根據這3種標準做出來的
Kuberentes 可以根據不同標準做出不同架構的 Kubernetes

> k8s 的 scale out ，自動擴充出分身的 pod ，硬體設備要夠強，不然會有災難。


---

## Kubernetes Pods (CRI 的規矩)

![](https://i.imgur.com/CqaHh8u.png)


- pod 不會存在兩個 node 或以上，只會在一個 node 上
- pods 的總記憶體用量為所在的 node 記憶體的總量乘上 80%，保留 20% 給 node 主機作業要使用。



---

## Kubernetes Pods (CNI)

![](https://i.imgur.com/f9hSk3v.png)


- 不同 pod 裡的 Container 之間的溝通無法使用 localhost
- 一個 pod 會有一個 IP 位址和一片網路卡，裡面的多個 Container 會共用 pod 的網路卡和 IP 位址
- pod 對外有 IP 位址，對內多個 Container 是用 localhost 這個名稱來互通有無 (裡面是記憶體作業)


---

## Kubernetes Pods (CSI)

![](https://i.imgur.com/kqBN7pH.png)

- 上圖為 NFS 專案的架構，CSI 還有很多種專案
- Container 使用的 overlay2 檔案系統，如果大量的新增、修改和刪除檔案，檔案系統會掛點，所以他不適合儲存大量的資料。
- pod 可以透過 `bypass` 跳過 overlay2 檔案系統， 使用 K8S 的 volume 來做儲存，這個 volume 會透過網路連到 NFS 。
- NFS（共享資料夾） 是一台機器(網路儲存設備，e.g., qnap)，把一堆資料夾，透過網路分享給不同的 pod 來使用，如果 NFS 使用的是 linux 系統，那麼 pod 就會用 ext4 檔案系統做儲存
- pod 的 volume 可以連到我們的 big data 的平台 -- > ==ROOK==，
    - ROOK ，類似 Hadoop ，他的底層運作和 Hadoop 很像


---


## Kubernetes Pods (Scheduling)

![](https://i.imgur.com/FzVQ9Et.png)

- Scheduler 專門安排 pod 在哪一個 node run，靠的是 Labels (標籤)
- 我們的 pod 可以貼 Label ( 標籤 ) ，標籤儲存在資料庫裡面
- Pod 可指定如何佈署到帶有特定 Labels 的  Node



---


## Kubernetes Pods (High Availability)


![](https://i.imgur.com/xR2zoMU.png)

- pod 可以有很多本尊和分身，當他們同時在運作時，只要資料的儲存系統做的好（ 像是透過 ROOK ），多個 pod 可以並行運作，可以解決突然在短時間大量連線，造成系統當機的狀況。
- 結論：<font color=red>**Pod 可輕易做到高可用性 ( HA )**</font>



---

## Kubernetes Master 容錯

![](https://i.imgur.com/G4ySwDN.png)


- 在 Kubernetes Master 裡面的 etcd 儲存整個 K8S 的 meta data ， 只要 Master 這台主機掛了， 整個 K8S 再見，所以要做容錯
- 多台 Master 主機的 etcd 會自動同步
- 根據 Raft 演算法可以算出如何做到 Master 主機的容錯。
    - etcd: 計算故障容許節點數為 (N-1)/2，其中 N 為 K8S Master 數量, 而由 (N/2)+1 計算出 Majority, 只要滿足這數量, 代表 K8S 可以新增 Service, Deployment 等功能。
- 當一個 K8S Cluster 只存在 1 台 或 2 台 Master 無法做到容錯。
- 3 台 Master 可以允許壞一台 Master 主機， 如果再壞一台，會進入鎖死狀態，代表 Application 可以繼續 run ，但無法新增刪除修改資料。
- 如果要壞 2 台 Matser ，必須要有 5 台 Master 主機
- 如果只有 3 台實體主機，要如何做到 K8S 的容錯 ?
  將 3 台實體主機都做成 Master 主機，透過以下命令

```bash!
# 設定 K8S Master 可以執行 Pod
$ kubectl taint node m1 node-role.kubernetes.io/control-plane:NoSchedule-

output : 
node/m1 untainted
```

- 讓 master 主機也能 run pod ，但如果要這樣做，一開始在規劃主機的硬體規格時，要盡量規劃強一點。


{%hackmd 6-wDy9uiQNKd6G23w78uog %}
{%hackmd ISud-e3CSsO6Uw5UL8adSQ %}
{%hackmd Yfx_m5QMQWezA08KN2_BPw %}