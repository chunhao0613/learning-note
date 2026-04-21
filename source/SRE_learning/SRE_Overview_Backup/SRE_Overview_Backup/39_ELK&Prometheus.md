### **ELK Stack (Elasticsearch, Logstash, Kibana)**

- **定位：** **Logs（日誌）** 處理與全文檢索。
    
- **核心功能：** 收集、過濾並存儲系統產生的每一行 Log（例如：Nginx 訪問記錄、Java 例外堆棧）。
    
- **與 Hadoop 的關係：** ELK 擅長「近期、即時」的檢索（找問題發生的原因）；Hadoop 擅長「長期、海量」的存檔與分析。
    

### **Prometheus**

- **定位：** **Metrics（指標）** 監控與告警。
    
- **核心功能：** 收集「數值型資料」（例如：CPU 使用率 80%、每秒請求數 500）。它不存具體的 Log 文字。
    
- **與 Spark 的關係：** Prometheus 負責即時告警（現在壞了沒？）；Spark 負責事後深度分析（為什麼這三個月效能越來越差？）。