# Changelog

本文件依照 [Keep a Changelog](https://keepachangelog.com/zh-TW/1.0.0/) 格式記錄所有重要變更，版本號遵循 [Semantic Versioning](https://semver.org/)。

---

## [Unreleased]

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
