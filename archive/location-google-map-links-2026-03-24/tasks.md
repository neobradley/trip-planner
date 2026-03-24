# 實作計畫：地點 Google Maps 外部連結

## 概覽

在 `index.html` 的行程時間軸中，為每個活動卡片加入 Google Maps 外部連結圖示。實作分為三個步驟：新增輔助函式、修改渲染邏輯、撰寫屬性測試。

## 任務清單

- [x] 1. 新增 `buildMapsLink` 輔助函式
  - 在 `index.html` 的 `<script>` 區塊中，緊接在 `tipStyleMap` 常數定義之後，新增 `buildMapsLink(name, lat, lng)` 函式
  - 當 `lat` 與 `lng` 皆為有效數值（`typeof === 'number'` 且非 `NaN`）時，回傳 `https://www.google.com/maps/search/?api=1&query=<lat>,<lng>`
  - 否則回傳 `https://www.google.com/maps/search/?api=1&query=<encodeURIComponent(name)>`
  - _需求：1.1、1.2、1.4、4.1、4.2、4.3、4.4_

  - [ ]* 1.1 為 `buildMapsLink` 撰寫屬性測試（屬性 5：冪等性）
    - **屬性 5：buildMapsLink 的冪等性**
    - 對任意相同的 `name`、`lat`、`lng` 輸入，多次呼叫應回傳完全相同的 URL
    - **驗證需求：4.2**

  - [ ]* 1.2 為 `buildMapsLink` 撰寫屬性測試（屬性 1：有效座標產生座標格式 URL）
    - **屬性 1：有效座標產生座標格式 URL**
    - 對任意有效的 `name`、`lat`（-90~90）、`lng`（-180~180），回傳 URL 應包含 `query=<lat>,<lng>`
    - **驗證需求：1.1、4.3**

  - [ ]* 1.3 為 `buildMapsLink` 撰寫屬性測試（屬性 2：缺少座標時退回名稱搜尋）
    - **屬性 2：缺少座標時退回名稱搜尋**
    - 對任意有效的 `name`，當 `lat` 或 `lng` 為 `undefined`、`null`、`NaN`、非數值字串時，回傳 URL 應包含 `query=<encodeURIComponent(name)>`
    - **驗證需求：1.2、1.4、4.4**

- [x] 2. 修改 `renderDashboard()` 中的活動卡片 HTML
  - 找到 `renderDashboard()` 函式內組裝活動卡片的 `<h4>` 標籤
  - 將 `<h4>` 的 `onclick` 移至內部 `<span>` 上，並在旁邊加入 `<a>` 標籤
  - `<a>` 標籤的 `href` 呼叫 `buildMapsLink(act.name, act.lat, act.lng)`
  - `<a>` 標籤須包含 `target="_blank"` 與 `rel="noopener noreferrer"` 屬性
  - 使用 Lucide `map-pin` 圖示（`<i data-lucide="map-pin" class="w-4 h-4"></i>`）
  - `<h4>` 改用 `flex items-center gap-2` 排版，保留原有的顏色 class
  - _需求：2.1、2.2、2.3、2.4、1.3_

  - [ ]* 2.1 撰寫屬性測試（屬性 3：所有活動卡片均包含正確的 Maps 連結標籤）
    - **屬性 3：所有活動卡片均包含正確的 Maps 連結標籤**
    - 對任意活動類型（含 `accommodation`、`event`），渲染後每個卡片應包含 `href` 指向 `https://www.google.com/maps/search/` 的 `<a>` 標籤，且具備 `target="_blank"` 與 `rel="noopener noreferrer"`
    - **驗證需求：2.1、2.2、2.3、1.3**

  - [ ]* 2.2 撰寫屬性測試（屬性 4：編輯模式不影響 Maps 連結顯示）
    - **屬性 4：編輯模式不影響 Maps 連結顯示**
    - 切換 `state.isEditing` 為 `true` 或 `false`，渲染後每個活動卡片均應包含 Maps 連結 `<a>` 標籤
    - **驗證需求：2.4**

- [x] 3. 建立瀏覽器端屬性測試 HTML 檔案
  - 建立 `tests/location-google-map-links.test.html`，可直接在瀏覽器開啟執行
  - 透過 CDN 載入 `fast-check`（`https://cdn.jsdelivr.net/npm/fast-check/lib/bundle/fast-check.min.js`）
  - 將任務 1.1、1.2、1.3、2.1、2.2 的屬性測試整合至此檔案
  - 每個屬性測試以 `// Feature: location-google-map-links, Property <N>: <描述>` 標記
  - 每個屬性測試最少執行 100 次迭代（`numRuns: 100`）
  - 測試結果以可讀格式顯示於頁面上（通過/失敗、反例資訊）
  - _需求：4.1、4.2、4.3、4.4_

- [x] 4. 檢查點：確認所有測試通過
  - 在瀏覽器開啟 `tests/location-google-map-links.test.html`，確認所有屬性測試通過
  - 確認時間軸中每個活動卡片旁均顯示 `map-pin` 圖示連結
  - 確認點擊圖示可在新分頁開啟 Google Maps
  - 如有問題，請向使用者提問後再繼續

- [x] 5. 驗證 PDF 匯出相容性
  - [x] 5.1 確認 `<a>` 標籤不破壞 `html2pdf.js` 的匯出流程
    - 檢查 `handleExportPDF()` 函式，確認無需額外處理 `<a>` 標籤
    - 確認 `html2pdf.js` 預設行為可保留超連結（需求 3.2）
    - 若無法保留，確認 `href` URL 文字在 PDF 中仍可見（需求 3.3）
    - _需求：3.1、3.2、3.3_

- [x] 6. 最終檢查點：確認所有測試通過
  - 確認所有屬性測試通過，如有問題請向使用者提問後再繼續

## 備註

- 標記 `*` 的子任務為選用項目，可跳過以加速 MVP 開發
- 每個任務均對應具體需求，確保可追溯性
- 屬性測試驗證通用正確性，單元測試驗證具體範例與邊界條件
- 測試檔案為獨立 HTML，無需建置工具或 npm，直接在瀏覽器執行
