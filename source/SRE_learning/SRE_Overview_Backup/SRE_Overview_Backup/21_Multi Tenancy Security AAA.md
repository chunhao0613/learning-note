# Multi Tenancy Security AAA

[TOC]


---

## Kubernetes 一個大樓標準藍圖

![](https://i.imgur.com/k8sLGAC.png)

把 Kubernetes 想成一棟大樓的標準藍圖，把自己想成要當地產大亨，首先要先確認多少土地，能蓋多少大樓和住戶...等，在 kubernetes 中，土地代表建立 k8s 的資源，以目前來說就是虛擬平台、三大公有雲 or 實體機，接著要開始蓋房子，以大樓來看，就是分住戶、公共建設和管理委員等，
公共建設是 K8s 啟動後須先安裝的


## Multi Tenancy

- Soft multi-tenancy
	- which doesn’t have very strict isolation between tenants, is aimed at preventing accidental interference, and is suitable for trusted tenants.
	- 多人共用 k8s ，透過內建軟體隔離。

- Hard multi-tenancy
	- which assumes tenants cannot be trusted, enforces stricter isolation to protect tenants from malicious interference, and is suitable for both trusted and untrusted tenants.
	- 一個用戶一個 k8s ，代表會啟動不同的 K8s Cluster 


## AAA

- Authentication 身分認證
	- 驗證你是誰
- Authorization 授權
	- 你有什麼權限，可以做什麼事情
- Accounting(Auditing) 統計
	- 使用者行為的紀錄



## Landlord 角色設計

- Gateway
	- 提供 SSH 登入的入口，透過 Deployment 分流，將連線轉向用戶專用的資源。
- Tenant
	- 用戶的專用資源，每個用戶家目錄都有 wk 資料夾，供用戶儲存個人資料，屬於永存資料區塊。
	- 透過 K8s 的 namespace 做環境的隔離
- Kuser
	- 建立 Tenant 對應的 K8S 認證及授權。
- Logger
	- 接收 Tenant 操作命令及檔案編輯歷程，送至資料庫儲存。
- Mariadb
	- 儲存 Tenant 操作命令及檔案編輯歷程


## Landlord 架構圖

![](https://i.imgur.com/5WKy3gp.png)



---

## Landlord 資料流

![](https://i.imgur.com/gUwZ5HK.png)


---


## Landlord 監控

- Prometheus：住戶社區整體監控中心
- Grafana：監控中心裡的超大型 LED 儀表板


## 問題討論

1. 為甚麼腳色大多使用 Deployment 設計
    - 當管理的 pod 毀損或被刪除時，會自動再產生同樣的 pod 出來
2. 為甚麼要有 Gateway 這個腳色
    - 當我們透過 `ssh -p 20000 tenant-0@Gateway_Headless_Service_ExternalIP` 連線時，
    - Gateway 裡面的 `/etc/profile` 會再放一段 `sshpass -p \$(whoami) ssh \$(whoami)@\$(whoami).${USERTAG}.${NS} -q 2>/dev/null` 連線到 Tenant 的程式，
    - Tenant 透過 StatefulSet 建出來的 pod 名字會有流水號，搭配無頭服務和 CoreDNS 來達到名稱解析
    - 當 Tenant 數量增加時，Gateway 也能知道連到哪一個 Tenant
    - `${USERTAG}` 是 StatefulSet 的名字
3. 為甚麼 Tenant 要用 StatefulSets 設計
    - pod 的名字是流水號
    - 當所在主機掛點時，可在另一台主機生出來
    - 多個 pod 可共用 PVC
5. Tenant 是透過何種方式達到永存資料
    - 使用 Local Path Provisioner 來達到永存
    - 當 PV 所在的 node 掛了，資料還是救不回來，因為 PVC 建不出來 pod 也會無法建立
    - 可使用 NFS Storage 把資料備份到其他主機
    - 最佳解決方案 Ceph
6. 可不可以沒有  Logger ，將資料直接放入 Mariadb
    - 不行，要保護 Mariadb 不被 Tanent 隨意破壞
    - Tanent 傳給 Logger 資料，再統一由 Logger 做資料傳送到 Mariadb



###### tags: `系統工程`







































































