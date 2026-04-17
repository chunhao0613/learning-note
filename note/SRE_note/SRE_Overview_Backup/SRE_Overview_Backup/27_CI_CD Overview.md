# CI/CD Overview

**目錄**
[TOC]


---

## 單人開發流程

1. 攥寫Bash Script
2. 執行程式  -- > 成功
3. 攥寫Dockerfile
4. Build Image Run Container  -- > 成功
5. Push Image
6. 攥寫K8s Yaml檔
7. Kubectl apply  -- > 符合預期
8. 維運應用程式  ----------------- > 如有新需求 --- > 回到第一步



:::spoiler 流程圖
![](https://lh4.googleusercontent.com/qxRu6gFxQP9A6OkCdSuePmoS1PHIxkrmQ0qA45CLYpxHSxSifgjBggPuqXu3-DVNROliByeOlvfDO37HB6f9KDo_AbQVj727WHq-5zaXcP-vmjeSJP6nUcsVjEuT0Kxt8uDbQonfJTxmT2KzhA)
:::

### **程式設計師的痛點**
- 哪些東西需要備份?
- 如何備份?
- 程式碼版本差異如何得知?
- 時常出現的人為失誤如何減少?
- 人工手動的反覆步驟如何簡化?




### **夢想與現實**
:::spoiler 示意圖
![](https://i.imgur.com/i4yZnzt.png)
:::

三位工程師各自開發三個不同的新功能 (現實上很容易超過三個功能)
一個月後另一位工程師會來整合
發生 - > 整合地獄
最後測試人員 - > 測試 -- > 部署上線
![](https://i.imgur.com/y3dlKpg.png)

------------------------------------------------- <font color=red> 為了解決以上痛點 </font>

---

## 持續整合 Continuous integration (CI)

持續小規模小規模的(縮短時間週期)整合加測試 --- > 快樂部署上線

![](https://i.imgur.com/sp6GjHb.png)

## 持續交付 vs 持續部署(CD)

- Continuous delivery (手動)
	- <font color=red> 持續交付：將整合＋測試完成後的成果交付行為，減小規模、縮短週期，經過人工審查後交付（業界主流） </font>
- Continuous deployment (自動)
	- 持續部署：將整合＋測試完成後的成果交付行為，減小規模、縮短週期，並且自動執行無人工審查（心臟要夠大顆）

![](https://i.imgur.com/3bBKpXo.png)


---

## 導入Git
[Git 筆記連結](https://hackmd.io/@QI-AN/r1RJ810Dc)

解決程式設計師的痛














###### tags: `CI/CD`