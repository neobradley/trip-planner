# Changelog

本文件依照 [Keep a Changelog](https://keepachangelog.com/zh-TW/1.0.0/) 格式記錄所有重要變更，版本號遵循 [Semantic Versioning](https://semver.org/)。

---

## [Unreleased]

---

## [1.4.0] - 2026-03-24

### Added

- **地點 Google Maps 外部連結**：行程時間軸中每個活動卡片的地點名稱旁，新增可點擊的 `map-pin` 圖示連結，點擊後以新分頁開啟 Google Maps。
- 新增 `buildMapsLink(name, lat, lng)` 輔助函式：有效座標時產生 `query=lat,lng` 格式 URL，否則退回 `encodeURIComponent(name)` 名稱搜尋，支援中文及特殊字元。
- PDF 匯出相容性：由於 html2pdf.js 以 html2canvas 點陣化，超連結無法保留為可點擊連結；改於圖示旁以極小字顯示完整 Maps URL，確保 PDF 中仍可手動複製。
- 新增 `tests/location-google-map-links.test.html`：5 個 fast-check 屬性測試（各 100 次迭代），驗證座標 URL 格式、名稱退回邏輯、卡片連結屬性、編輯模式不影響連結、冪等性。

---

## [1.3.1] - 2026-03-24

### Fixed

- **[BUG] `swapActivity` 忽略 `state.swapPool`，「換一個」永遠顯示假資料**
  - 根本原因：`swapActivity` 從未實作 `state.swapPool` 讀取邏輯，直接從整個 `swapFallbackPool` 隨機抽取，無類型過濾、無重複排除。
  - 修復方式：重寫 `swapActivity` 核心邏輯——優先從 `state.swapPool[activity.type]` 取出第一個不在當天行程中的地點並以 `splice` 移除；池為空時退回 `swapFallbackPool`（依 `type` 過濾）；兩池均耗盡時呼叫 `showExhaustedDialog()`。
  - 保留行為：`time` 欄位繼承、`instanceId` 格式 `swap-{timestamp}`、`selectActivity` 呼叫、`handleGenerate` 重置邏輯均不受影響。

---

## [1.3.0] - 2026-03-24

### Added

- **Gemini 備選池（Swap Pool）**：初次呼叫 Gemini API 時，於同一 Prompt 中一併請求分類備選地點，依 `type` 儲存於 `state.swapPool`，「換一個」功能改為優先從真實備選池取出地點，無需額外 API 呼叫。
- 新增 `validateSwapPool(rawSwapPool, baseLat, baseLng)` 函式，對 Gemini 回傳的備選池進行防禦性驗證：過濾無名稱項目、補全缺失座標（目的地中心點 ±0.05°）、執行 `instanceId`／`icon`／`mappedTips` 欄位補全。
- 新增 `showExhaustedDialog(dayIdx, actIdx)` 函式，備選池耗盡時顯示 Modal，提供「恢復原始建議」與「取消」選項。
- 新增 `restoreOriginalActivity(dayIdx, actIdx)` 函式，從 `state.originalTrip` 深層複本還原原始活動；找不到對應 `instanceId` 時顯示錯誤提示。
- `state` 新增 `swapPool: {}` 與 `originalTrip: null` 欄位；每次重新規劃前自動重置。
- `generateFallback` 備用模式下自動將 `swapFallbackPool` 依 `type` 分類填入 `state.swapPool`，確保離線模式「換一個」功能可用。
- 屬性測試（fast-check）：Property 4（無名稱地點被排除）、Property 5（無效座標補全後為數字）、Property 6（swapPool 永遠為物件）、Property 8（originalTrip 為深層複本），各執行 20 次迭代。

### Changed

- `swapActivity` 核心邏輯重寫：優先從 `state.swapPool[type]` 取出真實地點並移除已用項目，退回 `swapFallbackPool`，最終耗盡時呼叫 `showExhaustedDialog`。
- `finishGenerate` 新增 `state.swapPool` 與 `state.originalTrip` 賦值邏輯。
- `callGeminiAPI` Prompt 加入 `swapPool` 輸出要求段落。

---

## [1.2.0] - 2026-03-24
> git: `95e7f40`, `a22faf7`, `5b70fef`, `d05f2ce`

### Added

- GitHub Actions 自動部署工作流程，推送至 `main` 分支時自動部署靜態內容至 GitHub Pages（`5b70fef`）。
- Telegram 通知：部署完成後自動發送 Telegram 通知（`a22faf7`）。

### Removed

- 移除未使用的 `jekyll-docker.yml` 工作流程（`95e7f40`）。

---

## [1.1.2] - 2026-03-24
> git: `3382ff1`, `ef35349`

### Fixed

- **[BUG] `google_search` tool 與 `responseMimeType: "application/json"` 不相容（HTTP 400）**
  - 根本原因：Gemini API 不允許同時使用 `google_search` tool 與 `responseMimeType`，回傳 `INVALID_ARGUMENT`。
  - 修復方式：移除 `generationConfig` 中的 `responseMimeType`，改為在 Prompt 中強化純 JSON 輸出約束指令（`3382ff1`）。
  - 同步更新 `GEMINI.md` 文件，標註 `responseMimeType` 不可與 `google_search` 併用。

- **[BUG] 瀏覽器快取導致舊 JS 持續執行，`Ctrl+Shift+R` 無效**
  - 修復方式：新增版本守衛 inline script，於頁面最早期比對 `sessionStorage` 版本號，若不符立即執行 `window.location.reload(true)`（`ef35349`）。
  - 新增 `<html data-version="1.1.1">` 與 `<meta name="app-version">` 供版本追蹤。
  - 模型名稱改為從 `<meta name="gemini-model">` 動態讀取，切斷 JS 快取對模型端點的影響。

---

## [1.1.1] - 2026-03-24
> git: `e62460d`

### Fixed

- **[BUG] API 仍呼叫廢棄端點 `gemini-2.5-flash-preview-09-2025`（HTTP 404）**
  - 根本原因：`file://` 協議下 `Cache-Control` meta tag 無效，瀏覽器仍執行快取的舊版 JS。
  - 修復方式：將模型名稱從 JS hardcode 改為從 `<meta name="gemini-model">` 動態讀取。

---

## [1.1.0] - 2026-03-24
> git: `0bcca83`

### Fixed

- **[BUG] `swapFallbackPool` 未定義導致「換一個」功能崩潰**
  - 新增包含 6 筆備用景點的 `swapFallbackPool` 常數陣列。

- **[BUG] `instanceId` 每次重新渲染都產生新值，導致選取狀態失效**
  - 改為優先保留已存在的 `instanceId`，僅在不存在時才產生新值。

- **[BUG] PDF 匯出步驟進度永遠不顯示第三步完成狀態**
  - 步驟計數改從 1 開始，`save()` 完成後呼叫 `updateStep(3)` 再關閉 modal。

- **[BUG] 瀏覽器快取導致舊模型端點持續被使用**
  - 新增 `Cache-Control: no-cache` meta tag。

### Added

- `callGeminiAPI()` 加入防呆斷言：若模型名稱包含 `preview` 字串，立即拋出 `[Config Error]` 錯誤。
- 新增 `CHANGELOG.md`。

### Changed

- `state.formData.days` 賦值時改為直接 `parseInt()`，確保型別為數字。
- `renderDashboard()` 中的 `updateMapMarkers()` 改為僅在地圖已初始化時執行。

---

## [1.0.1] - 2026-03-24
> git: `343f547`

### Changed

- 所有 `.md` 文件修正為標準 Markdown 格式（標題層級、條列、程式碼區塊、表格）。
- `GEMINI.md` 退避重試延遲描述從 `1000ms` 更正為 `1500ms`，與實際代碼一致。
- `README.md` 技術棧改為 Markdown 表格呈現。
- 新增 `.vscode/settings.json`。

---

## [1.0.0] - 2026-03-24
> git: `4389450`

### Added

- 表單輸入：目的地、天數、人數、住宿安排、興趣、飲食限制、強制錨點事件。
- Gemini API 整合：Google Search Grounding、Exponential Backoff 重試、Bulletproof JSON 解析。
- Fallback 機制：API 失敗時自動切換本地備用資料，UI 顯示「備用測試資料」標示。
- Leaflet.js 真實地圖：OpenStreetMap 圖資、自訂 HTML Marker、路徑繪製、`flyTo` 飛越動畫。
- 行程微調：支援單一景點「換一個 (Swap)」與「刪除 (Delete)」。
- PDF 匯出：整合 html2pdf.js，輸出 A4 格式行程表。
- Zero-Build-Step 架構：單一 `index.html`，無需任何編譯或安裝步驟。
- 專案文件：`AGENT.md`、`GEMINI.md`、`README.md`、`SBE.md`、`skills.md`。
