# CLAUDE.md — 扶輪社活動報名板（取代 Line 接龍的線上報名表）

## 專案是什麼

這是給扶輪社用的**線上活動報名板**，用來取代過去在 Line 群組裡「接龍」報名的做法。
社員打開一個網址，就能看到所有活動場次、按下「我要報名」填名字，所有人看到的是**同一份即時名單**。報名的人**不需要任何帳號**，打開網址就能用。

- 給誰用：扶輪社社員（報名）、少數管理者（開/改/刪場次）。
- 解決什麼：Line 接龍難統計、訊息洗版、人數算不準。
- 線上網址：https://curtis0000.github.io/rotary-signup/

## 技術棧

- **前端**：單一 `index.html`（原生 HTML/CSS/JS，無框架、無建置步驟）。整個 App 就這一個檔案。
- **後端**：Firebase Firestore（免費 Spark 方案），透過 CDN 載入 `firebase-app` / `firebase-firestore` v12.4.0 的 ES module。
- **部署**：GitHub Pages，repo `curtis0000/rotary-signup`，發佈分支 `main`、根目錄。
- Firebase 專案：`rotary-signup-f8ac9`。

## 核心架構原則（不可違反）

1. **維持單檔。**（必遵守）整個 App 就是 `index.html`，不要拆檔、不要引入打包工具或框架。改了之後要 push 到 `main` 才會更新線上版。
2. **畫面一律從本機快取繪製，雲端只在讀寫時連線。**（不可違反）這是刻意設計：若每個互動都去 Firestore 重抓資料，每次點擊都會疊上網路往返而卡頓。`render()` 只讀記憶體中的 `ACTS` / `SIGNUPS` 快取；`refresh()` 才連雲端把最新資料抓回快取（背景每 6 秒一次、手動刷新、初次載入時呼叫）。
3. **所有寫入動作必須上 `busy` 鎖。**（不可違反）送出/建立/刪除進行中時 `busy=true`，擋掉重複觸發，避免社員連點送出好幾筆重複報名。送出按鈕同時要 disable 並顯示「送出中…」。
4. **寫入報名前先重抓該場次雲端最新名單再 append。**（必遵守）降低兩人同時報名互相覆蓋的機率（採「寫前重讀單一 key」而非全量鎖，因為這是輕量報名板、衝突極少）。
5. **Firebase 設定值（apiKey 等）可公開。** 它本來就是前端值，安全靠 Firestore 規則控管，不是靠藏 key。不要把它當機密處理。

## 資料模型

Firestore 用一個 `store` 集合當 key-value 儲存，每筆文件 id 就是 key、欄位 `value` 存陣列：

```
store / "activities"        → { value: [ {id, title, date, time, location, capacity, note} ] }   // 所有場次
store / "signups:<活動id>"  → { value: [ {id, name, party, note, ts} ] }                          // 該場次的報名,每場一筆文件
```

- `capacity`：名額上限，`0` = 不限。
- `party`：同行人數含本人（帶眷屬就填 2）。
- 文件 id 含冒號（`signups:xxx`）是 Firestore 允許的，不要改成別的分隔符。

瀏覽器 `localStorage` 另存一筆**個人標記**（不上雲、每台裝置各自記）：

```
localStorage "rotary_mine" → [報名id, ...]   // 只用來標示「哪些是我自己報的」
```

## 身分與權限

目前**沒有真正的登入驗證**，是刻意的取捨（公開報名板、優先簡單可用）。權限現況如下：

| 角色 | 看名單 | 報名/改/刪自己 | 開/改/刪場次 |
|---|---|---|---|
| 一般社員 | ✅ | ✅ | ❌（畫面隱藏） |
| 管理者 | ✅ | ✅ | ✅（開「管理場次」開關後出現） |

- 「管理者」只是**前端開關**（adminMode），不是真權限：Firestore 規則目前是 `allow read, write: if true`，任何人都能透過 API 讀寫。
- 這代表懂技術的人可繞過畫面直接改資料。對社內報名板通常可接受；若要真權限需另做登入（見第二階段）。

## 開發慣例

- **語言**：所有 UI 文案、註解一律繁體中文。
- **改完要上線**：直接改 `index.html` → commit → push 到 `main`，GitHub Pages 會自動重建（約 1～2 分鐘）。**不要自動 commit / push，等開發者確認。**
- **本機測試**：`python3 -m http.server` 後開 `http://localhost:<port>/index.html`（一定要用 http server，ES module 不能用 `file://` 開）。
- **進度記錄**：現況與待辦寫在 `PROGRESS.md`，保持精簡、只朝前看。

## 第二階段預留（現在不做，但設計別擋路）

- **真正的權限控管**：若要讓只有管理者能開/刪場次，需接 Firebase Auth 並改寫 Firestore 規則（管理動作限特定帳號）。目前資料模型不需為此改動。
- **名額額滿自動鎖報名**：目前額滿只顯示「候補」仍可報，未硬性阻擋。
- **匯出備份**：管理模式已有「匯出 JSON」功能，作為搬到正式系統前的備份手段。
