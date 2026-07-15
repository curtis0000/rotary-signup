# PROGRESS.md — 目前進度

> 規則與架構看 `CLAUDE.md`，這裡只記「做到哪、接下來做什麼」。

## 現況（已上線）

- ✅ 報名板已正式發佈：**https://curtis0000.github.io/rotary-signup/**
- ✅ 後端從原本的 Claude Artifact 共享儲存，改接 **Firebase Firestore**（專案 `rotary-signup-f8ac9`，Spark 免費方案）。社員報名免帳號。
- ✅ 共享名單可用：多人看到同一份、重整不消失，已實測通過。
- ✅ 效能/防呆：改為本機快取繪製（互動瞬間反應）、寫入上鎖 + 「送出中…」防止連點重複報名，已實測順暢。
- ✅ 已部署到 GitHub Pages（repo `curtis0000/rotary-signup`，`main` 分支根目錄）。
- ✅ 清掉桌面多餘的原始 uuid 檔，資料夾現只剩 `index.html`、`CLAUDE.md`、`PROGRESS.md`、`.gitignore`。
- ✅ 報名人數可填 **0**：用來登記「不參加」——名字與備註照樣顯示，但不計入總人數（`normParty`／`totalHeads` 處理）。
- ✅ 修「名單全部顯示 0 人」：`refresh()` 改用單一 collection 查詢一趟抓齊（原本逐場 19 趟，行動網路易中途斷線）；改用 `getDocsFromServer` 讓斷線真的丟錯誤，失敗時保留舊快取並顯示「⚠ 未能更新」，不再錯誤歸零。背景同步 6 秒 → 15 秒，降低免費額度消耗。已用 headless Chrome 實測正常／斷線兩條路徑。

## 待處理 / 注意

- ⚠️ **清測試資料**：先前本機測試新增的場次/報名是寫進正式雲端的，正式給社員用前，用「管理場次」把測試場次刪掉。
- ⬜ `CLAUDE.md`、`PROGRESS.md` 尚未 commit/push（等開發者確認後再上）。

## 之後可能要做（未定案）

- 短網址 / QR code，方便社員掃碼進入。
- 真正的權限控管（Firebase Auth + 收緊 Firestore 規則），讓只有管理者能開/刪場次。
- 名額額滿時硬性阻擋報名（目前僅標示候補）。
