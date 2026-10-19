# 活動幸運抽獎系統 (Google Apps Script + GitHub Pages)

一套無伺服器負擔、開箱即用的現代化活動抽獎系統。後端透過 Google Form 表單收集大學生名冊（系所、年級、姓名）、以 Google Sheet 維護獎品清單與配額，並由 Google Apps Script (GAS) 提供 JSON 讀取 API 與需管理權杖的資料寫入操作；前端採用原生 HTML5、CSS 與 JavaScript 實作，可直接免費託管於 GitHub Pages。

---

## ⚡ 快速設定與 API 介接指南 (Quick Setup & API Integration)

本系統支援「**本機展示模式 (Demo Mode)**」與「**正式 Google 試算表 API 連線**」兩種運行方式。

### 1. 實際介接 Google Apps Script API 設定方式

#### 介面操作設定（推薦，開箱即用）
1. 使用瀏覽器開啟 `index.html`（或您的 GitHub Pages 網址）。
2. 點擊右上角 **「⚙️ 設定 API」** 按鈕展開設定面板。
3. 在 **Google Apps Script Web App URL** 輸入框中貼上您部署好的 GAS Web App 網址：
   ```text
   https://script.google.com/macros/s/AKfycb.../exec
   ```
4. 點擊 **「連線載入資料」** 按鈕：
   - 系統將發起 HTTPS GET 請求，自動同步獎品配額與學生名冊。
   - 連線成功後，API 網址將自動儲存於瀏覽器 `localStorage`（Key: `lottery_gas_api_url`），後續重整或再次開啟網頁皆會自動套用，無需重複輸入。

#### 程式預設配置（可選）
若希望網頁初次開啟時即自動載入特定 API，亦可直接在瀏覽器開發者工具 (Console) 預先寫入：
```javascript
localStorage.setItem('lottery_gas_api_url', 'https://script.google.com/macros/s/YOUR_GAS_ID/exec');
```

> [!IMPORTANT]
> **GAS 發布權限關鍵注意事項**：
> 在 Google 試算表中發布 GAS 網路應用程式時，**「誰可以存取 (Who has access)」務必選擇「所有人 (Anyone)」**。若設定為「僅限自己」或「網域內」，瀏覽器跨網域（CORS）存取時會被導向 Google 登入頁面，導致前端無法成功解析 JSON 資料。

---

### 2. 資料結構範例 (app.js DEMO_DATA / API Schema)

系統內建於 `app.js` 的 `DEMO_DATA` 提供獎品與參加者示範資料；正式 GAS API 除這兩個欄位外，還可能回傳 `weights`、`claims` 與 `winners`。您可依此格式直接擴充或進行離線測試：

#### 標準 JSON 資料範本
```json
{
  "status": "success",
  "prizes": [
    {
      "id": "demo_p1",
      "name": "頭獎：Apple iPad Air (256GB)",
      "quota": 1,
      "order": 1,
      "description": "特獎校園高效學習首選"
    },
    {
      "id": "demo_p2",
      "name": "二獎：Sony 降噪藍牙耳機",
      "quota": 2,
      "order": 2,
      "description": "專注讀書聽歌神器"
    },
    {
      "id": "demo_p3",
      "name": "三獎：校園書局與百貨禮券 $2000",
      "quota": 5,
      "order": 3,
      "description": "購書文具實用好禮"
    },
    {
      "id": "demo_p4",
      "name": "幸運獎：星巴克飲品兌換券",
      "quota": 10,
      "order": 4,
      "description": "期中期末暖心咖啡"
    }
  ],
  "participants": [
    {
      "id": "B11902001",
      "name": "陳冠宇",
      "department": "資訊工程學系",
      "grade": "大三",
      "email": "b11902001@school.edu.tw"
    },
    {
      "id": "B12703015",
      "name": "林若晴",
      "department": "財務金融學系",
      "grade": "大二",
      "email": "b12703015@school.edu.tw"
    },
    {
      "id": "B10102033",
      "name": "張凱翔",
      "department": "外國語文學系",
      "grade": "大四",
      "email": "b10102033@school.edu.tw"
    },
    {
      "id": "B13501008",
      "name": "黃子庭",
      "department": "生物機電學系",
      "grade": "大一",
      "email": "b13501008@school.edu.tw"
    },
    {
      "id": "R12521019",
      "name": "吳家瑋",
      "department": "電機工程研究所",
      "grade": "碩二",
      "email": "r12521019@school.edu.tw"
    }
  ]
}
```

#### 資料欄位說明規範

| 區塊 | 欄位名稱 | 型別 | 必填 | 說明與呈現位置 |
| :--- | :--- | :---: | :---: | :--- |
| **prizes** | `id` | String | 是 | 獎品唯一識別碼（如 `demo_p1` 或 `prize_1`） |
| | `name` | String | 是 | 獎品完整名稱（呈現於下拉選單、看版與中獎名冊） |
| | `quota` | Number | 是 | 獎品總配額名額數量（抽完時按鈕自動鎖定） |
| | `order` | Number | 否 | 排序優先序（數字越小越優先） |
| | `description` | String | 否 | 獎品備註說明（呈現於獎品看版下方） |
| **participants** | `name` | String | 是 | 學生真實姓名（抽獎舞台中央大字體呈現） |
| | `department` | String | 是 | 就讀學系/研究所（抽獎舞台與名冊次標題） |
| | `grade` | String | 是 | 年級別，如大一、大二、大三、大四、碩一、碩二 |
| | `id` | String | 否 | 學生學號或身分編號（以 `#學號` 格式標註防同名同姓） |
| | `email` | String | 否 | 學生聯絡信箱（匯出 CSV 時提供中獎名單核對） |

---

### 3. 一鍵體驗展示模式 (Demo Mode)

若尚未建立 Google 試算表或處於離線環境，可直接點擊首頁右上角 **「📊 載入展示資料」** 按鈕，系統會立即注入上述 `DEMO_DATA`，包含 4 項獎品與 15 位各系所學生名冊，方便快速測試開獎動畫、獎品切換與 CSV 匯出功能。

---

## 🆕 三項 Issue 整合功能（#1、#8、#11）

本次更新保留原本的獎品切換、名額統計、抽獎動畫、避免重複中獎、展示模式與 CSV 匯出，另外加入下列功能。此整合版本仍屬**開發測試版**，尚未完成正式 GAS 部署與多人同時操作驗證。

### Issue #1：參加者權重設定與加權抽獎

- 在權重設定區依參加者 ID 調整權重，預設值為 `1`，可設定範圍為 `0–1000`。
- 抽獎時以**符合抽獎資格的報名紀錄**作為候選池，按其權重進行隨機抽選；權重 `0` 不會被抽中。
- 權重代表相對機率，不是保證中獎。例如兩筆候選紀錄的權重為 3 與 1，抽中機率分別為 75% 與 25%。
- 重複報名紀錄會保留多個抽取機會；但原有已中獎者排除規則仍須遵守。
- 按下儲存權重後會先保存在瀏覽器；若已設定 API 網址並輸入管理權杖，前端會嘗試將權重送往 GAS 的 `saveWeights` 操作。

### Issue #11：中獎紀錄搜尋與獎項篩選

- 在中獎紀錄區輸入姓名關鍵字，即可篩選姓名包含該文字的紀錄。
- 可透過獎項下拉選單只顯示特定獎項的中獎者；兩項條件可同時使用。
- 提供清除篩選及無結果提示。
- 搜尋只改變畫面顯示，不修改原始中獎紀錄、獎品配額或後續抽獎資格。

### Issue #8：領獎狀態管理

- 中獎紀錄新增領獎勾選欄位，用來區分「已領取」與「未領取」。
- CSV 匯出包含領獎狀態，方便現場核對及活動後整理。
- 領獎狀態先存於瀏覽器；正式連線時可嘗試透過 GAS `setClaim` 操作更新試算表。
- **中獎與已領獎是不同狀態**：勾選領獎不會重新開獎，也不應改變中獎資格。

### 新增的 GAS 資料與操作

| 項目 | 功能 | 儲存位置 |
| --- | --- | --- |
| `weights` | 參加者 ID 對應權重 | `抽獎權重` 工作表 |
| `claims` | 中獎紀錄 ID 對應領獎狀態 | `領獎狀態` 工作表 |
| `winners` | 中獎結果紀錄 | `中獎紀錄` 工作表 |
| `saveWeights` | 批次儲存權重 | GAS `doPost` |
| `setClaim` | 更新領獎狀態 | GAS `doPost` |
| `addWinner` | 新增中獎紀錄 | GAS `doPost` |
| `resetWinners` | 重置中獎紀錄與領獎狀態 | GAS `doPost`（高風險操作） |

**管理權杖設定：** 在 Apps Script「專案設定 → 指令碼屬性」建立 `ADMIN_TOKEN`，值應使用高強度隨機字串。於前端管理權杖輸入框輸入後，才可嘗試執行正式寫入。**不可把權杖寫進公開 GitHub 檔案。**

> [!WARNING]
> GAS 寫入採 `fetch(..., { mode: 'no-cors' })`，瀏覽器無法讀取寫入回應，因此畫面上的「已送出」**不代表後端已成功儲存**。操作後應重新載入 API，並檢查 Google Sheets 是否更新。正式環境也必須另外處理公開 API 的個資存取、多人抽獎時的配額一致性及重置權限，不能把本版視為已通過安全測試的正式系統。

---

## 🌟 系統亮點

1. **零伺服器維護成本**：完全依託 Google Apps Script 與 GitHub Pages，不需購買主機或架設資料庫。
2. **表單即時同步**：人員填寫 Google Form 表單後，隨時點擊重新同步即可載入最新名單。
3. **獎品彈性切換**：下拉選單即時切換抽獎品項，動態呈現各獎項之總額、已抽出與剩餘名額。
4. **炫麗抽獎舞台**：高對比慶祝視覺、流暢隨機滾輪動畫，並確保不重複中獎。
5. **中獎名單防遺失**：中獎結果先存入瀏覽器 `localStorage`，一般重新整理可保留本機進度；跨裝置同步需另外確認 GAS 寫入成功。
6. **一鍵匯出 CSV**：活動結束後可一鍵下載含獎項、學生姓名、系所、年級、學號、中獎時間與領獎狀態的名冊。
7. **開箱即測 Demo 模式**：內建大學生展示假資料，未連線前即可點擊「載入展示資料」體驗完整流程。
8. **加權抽獎**：可依參加者設定權重，調整相對中獎機率（Issue #1）。
9. **中獎紀錄查詢**：支援姓名與獎項雙條件篩選（Issue #11）。
10. **領獎管理**：可勾選已領獎狀態並匯出核對（Issue #8）。

---

## 📁 專案目錄結構

```text
.
├── index.html       # 前端抽獎系統主頁面 (GitHub Pages 入口)
├── style.css        # 前端視覺、舞台動畫與響應式排版樣式
├── app.js           # 抽獎邏輯、API 串接、LocalStorage 與 CSV 匯出
├── presentation.md  # 系統架構、實作剖析與正式上板操作展示簡報 (Markdown 格式)
├── gas/
│   ├── Code.gs      # Google Apps Script 資料讀取與受權杖保護的寫入 API
│   ├── README.md    # Google 表單、試算表欄位設定與 GAS 部署詳細教學
│   └── INTEGRATION_NOTES.md # 三個 Issue 整合與測試限制
└── README.md        # 專案總覽、API 介接設定指引與部署指南
```

---

## 🚀 完整部署與正式上線指南

### 步驟 1：部署 Google Apps Script 後端
詳細設定請參閱 [gas/README.md](gas/README.md) 與 [gas/INTEGRATION_NOTES.md](gas/INTEGRATION_NOTES.md)：
1. 於 Google 試算表中建立「表單回應 1」與「獎品」分頁。
2. 進入擴充功能 -> Apps Script，貼上 `gas/Code.gs`。
3. 在指令碼屬性新增 `ADMIN_TOKEN`，並依活動需求評估 Web App 存取權限；若採公開讀取，請先排除不應公開的個資。
4. 發布為網路應用程式並取得 Web App URL；正式寫入操作須使用管理權杖。

### 步驟 2：本地預覽測試
直接雙擊開啟 [index.html](index.html) 或使用任意靜態伺服器（如 VS Code Live Server）：
* 點擊右上角 **「📊 載入展示資料」** 即可立即開始模擬抽獎。
* 點擊 **「⚙️ 設定 API」**，貼上您於步驟 1 取得的 GAS 網址並點擊連線，即可同步真實名單與獎項。
* 測試權重設定、加權抽獎、姓名／獎項搜尋、領獎勾選與 CSV 匯出。
* 測試 GAS 寫入後，務必重新讀取 API 並核對試算表，不能僅憑前端提示判斷寫入成功。

### 步驟 3：部署至 GitHub Pages
1. 將本專案整包目錄提交並推播至您的 GitHub Repository（例如 `main` 分支）：
   ```bash
   git add .
   git commit -m "feat: initial lottery system"
   git push origin main
   ```
2. 進入該 GitHub 儲存庫的 **Settings** -> **Pages**。
3. 在 **Build and deployment** > **Source** 選擇 **Deploy from a branch**。
4. Branch 選擇 **`main`**，資料夾選擇 **`/ (root)`**，點擊 **Save**。
5. 等待 1~2 分鐘後，即可透過 GitHub 提供之靜態網址（如 `https://<username>.github.io/<repo>/`）直接開啟線上抽獎系統。

---

## 🧪 整合版本測試清單

- [ ] 展示模式可正常載入、抽獎、切換獎項及匯出 CSV。
- [ ] 權重 `0` 的候選紀錄不會中獎；其他權重影響抽取比例。
- [ ] 同名不同 ID、重複報名及已中獎者排除規則符合需求。
- [ ] 姓名搜尋、獎項篩選、組合條件與清除條件正常。
- [ ] 領獎狀態切換後，CSV 顯示正確。
- [ ] GAS 寫入後重新讀取，確認 Google Sheets 資料一致。
- [ ] 無權杖／錯誤權杖不能修改正式資料。
- [ ] 多裝置同時抽獎、配額與重置權限已完成安全驗證（正式上線前必須處理）。

---

## 🎯 現場操作小技巧

* **防止人員重複中獎**：系統自動記錄所有已中獎人員，後續抽出其他獎項時將自動過濾已得獎者。
* **分輪抽獎**：可隨時在下拉選單中切換欲抽出的獎品，各獎項的「已抽出」與「剩餘名額」會自動計算。
* **緊急備份**：每抽完一個階段，可隨時點擊「匯出 CSV」保存目前已抽出的中獎名冊。
