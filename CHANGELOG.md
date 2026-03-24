# Changelog

本文件依照 [Keep a Changelog](https://keepachangelog.com/zh-TW/1.0.0/) 格式記錄所有重要變更。

---

## [Unreleased]

---

## [1.1.0] - 2026-03-24

### Fixed

- **[BUG] API 404 錯誤 — 廢棄的 preview 模型端點**
  - 根本原因：`callGeminiAPI()` 使用了已廢棄的 `gemini-2.5-flash-preview-09-2025` 端點，導致所有 API 呼叫回傳 HTTP 404。
  - 修復方式：將模型更新為正式 GA 版本 `gemini-2.5-flash`（2025-06-17 發布，有效至 2026-06-17）。
  - 影響範圍：`index.html` `callGeminiAPI()`、`GEMINI.md`、`README.md`。

- **[BUG] `swapFallbackPool` 未定義導致「換一個」功能崩潰**
  - 根本原因：`swapActivity()` 引用了從未宣告的 `swapFallbackPool` 陣列，點擊「換一個」直接拋出 `ReferenceError`。
  - 修復方式：新增包含 6 筆備用景點的 `swapFallbackPool` 常數陣列。

- **[BUG] `instanceId` 每次重新渲染都產生新值，導致選取狀態失效**
  - 根本原因：`formatTripData()` 無條件呼叫 `Math.random()` 產生 `instanceId`，重新渲染後 `state.selectedActivityId` 無法對應。
  - 修復方式：改為優先保留已存在的 `instanceId`，僅在不存在時才產生新值。

- **[BUG] PDF 匯出步驟進度永遠不顯示第三步完成狀態**
  - 根本原因：`updateStep()` 的判斷邏輯從 0 開始，但 `finally` 在顯示完成前就隱藏 modal。
  - 修復方式：步驟計數改從 1 開始，`save()` 完成後呼叫 `updateStep(3)` 再關閉 modal。

- **[BUG] 瀏覽器快取導致舊模型端點持續被使用**
  - 修復方式：新增 `Cache-Control: no-cache` meta tag，強制瀏覽器每次重新載入最新版本。

### Added

- `callGeminiAPI()` 加入防呆斷言：若模型名稱包含 `preview` 字串，立即拋出 `[Config Error]` 錯誤，防止誤用廢棄端點。
- 模型名稱改為獨立常數 `const MODEL`，方便日後統一維護。

### Changed

- `state.formData.days` 賦值時改為直接 `parseInt()`，確保型別為數字而非字串。
- `renderDashboard()` 中的 `updateMapMarkers()` 改為僅在地圖已初始化時執行，避免不必要的重複呼叫。

### Docs

- 所有 `.md` 文件修正為標準 Markdown 格式（標題層級、條列、程式碼區塊、表格）。
- `GEMINI.md` 退避重試延遲描述從 `1000ms` 更正為 `1500ms`，與實際代碼一致。
- `README.md` 技術棧改為 Markdown 表格呈現。

---

## [1.0.0] - 初始版本

### Added

- 表單輸入：目的地、天數、人數、住宿安排、興趣、飲食限制、強制錨點事件。
- Gemini API 整合：Google Search Grounding、Exponential Backoff 重試、Bulletproof JSON 解析。
- Fallback 機制：API 失敗時自動切換本地備用資料，UI 顯示「備用測試資料」標示。
- Leaflet.js 真實地圖：OpenStreetMap 圖資、自訂 HTML Marker、路徑繪製、`flyTo` 飛越動畫。
- 行程微調：支援單一景點「換一個 (Swap)」與「刪除 (Delete)」。
- PDF 匯出：整合 html2pdf.js，輸出 A4 格式行程表。
- Zero-Build-Step 架構：單一 `index.html`，無需任何編譯或安裝步驟。
