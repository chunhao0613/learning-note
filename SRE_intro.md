```dataviewjs
// 1. 在這裡設定你的資料夾路徑 (例如: "Books/PDFs" 或 "")
const folderPath = "source/SRE_learning/可觀測性、監控與日誌";

// 2. 取得 Vault 中所有的檔案
const pdfFiles = app.vault.getFiles()
    // 3. 過濾出符合路徑且副檔名為 pdf 的檔案
    .filter(file => file.path.startsWith(folderPath) && file.extension === "pdf");

// 4. 將結果轉換為 Obsidian 連結並以清單顯示
dv.list(pdfFiles.map(file => dv.fileLink(file.path)));
```

```dataviewjs
// 1. 在這裡設定你的資料夾路徑 (例如: "Books/PDFs" 或 "")
const folderPath = "source/SRE_learning/雲原生部署與容器化";

// 2. 取得 Vault 中所有的檔案
const pdfFiles = app.vault.getFiles()
    // 3. 過濾出符合路徑且副檔名為 pdf 的檔案
    .filter(file => file.path.startsWith(folderPath) && file.extension === "pdf");

// 4. 將結果轉換為 Obsidian 連結並以清單顯示
dv.list(pdfFiles.map(file => dv.fileLink(file.path)));
```

AIOps 研究與前瞻
```dataviewjs
// 1. 在這裡設定你的資料夾路徑 (例如: "Books/PDFs" 或 "")
const folderPath = "source/SRE_learning/AIOps 研究與前瞻";

// 2. 取得 Vault 中所有的檔案
const pdfFiles = app.vault.getFiles()
    // 3. 過濾出符合路徑且副檔名為 pdf 的檔案
    .filter(file => file.path.startsWith(folderPath) && file.extension === "pdf");

// 4. 將結果轉換為 Obsidian 連結並以清單顯示
dv.list(pdfFiles.map(file => dv.fileLink(file.path)));
```

```dataviewjs
// 1. 在這裡設定你的資料夾路徑 (例如: "Books/PDFs" 或 "")
const folderPath = "source/SRE_learning/CI_CD_DevOps";

// 2. 取得 Vault 中所有的檔案
const pdfFiles = app.vault.getFiles()
    // 3. 過濾出符合路徑且副檔名為 pdf 的檔案
    .filter(file => file.path.startsWith(folderPath) && file.extension === "pdf");

// 4. 將結果轉換為 Obsidian 連結並以清單顯示
dv.list(pdfFiles.map(file => dv.fileLink(file.path)));
```
