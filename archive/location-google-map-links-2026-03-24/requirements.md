# 需求文件

## 簡介

本功能為「Gemini 實境旅遊規劃系統」（index.html）中的每個行程地點，加入可開啟外部 Google Maps 的超連結。使用者在瀏覽行程時間軸或匯出 PDF 後，可直接點擊地點名稱旁的連結圖示，跳轉至 Google Maps APP 或網頁，查看該地點的詳細地圖資訊與導航路線。

---

## 詞彙表

- **系統（System）**：Gemini 實境旅遊規劃系統，即 index.html 單頁應用程式。
- **活動卡片（Activity_Card）**：時間軸中代表單一行程地點的 UI 元件，包含地點名稱、時間、描述等資訊。
- **Google Maps 連結（Maps_Link）**：以 `https://www.google.com/maps/search/?api=1&query=<地點名稱>` 或 `https://www.google.com/maps/search/?api=1&query=<緯度>,<經度>` 格式組成的外部超連結。
- **座標（Coordinates）**：活動資料中的 `lat`（緯度）與 `lng`（經度）浮點數欄位。
- **地點名稱（Location_Name）**：活動資料中的 `name` 字串欄位。
- **PDF 匯出（PDF_Export）**：透過 html2pdf.js 將行程時間軸轉換為 PDF 檔案的功能。
- **時間軸（Timeline）**：儀表板左側顯示每日行程活動的捲動清單區域（`#timeline-container`）。

---

## 需求

### 需求 1：為每個地點產生 Google Maps 連結

**使用者故事：** 身為旅遊規劃使用者，我希望每個行程地點都有對應的 Google Maps 連結，以便快速查看地點位置與周邊資訊。

#### 驗收標準

1. WHEN 活動資料同時包含有效的 `lat` 與 `lng` 數值，THE System SHALL 以 `https://www.google.com/maps/search/?api=1&query=<lat>,<lng>` 格式產生 Maps_Link。
2. WHEN 活動資料缺少有效的 `lat` 或 `lng` 數值，THE System SHALL 以 `https://www.google.com/maps/search/?api=1&query=<URL編碼後的地點名稱>` 格式產生 Maps_Link。
3. THE System SHALL 為行程中每一個活動（包含住宿類型 `accommodation` 與錨點類型 `event`）產生對應的 Maps_Link。
4. THE System SHALL 確保 Maps_Link 中的地點名稱經過 `encodeURIComponent` 編碼，以支援中文及特殊字元。

---

### 需求 2：在活動卡片上顯示 Google Maps 連結

**使用者故事：** 身為旅遊規劃使用者，我希望在時間軸的每個活動卡片上看到 Google Maps 連結按鈕，以便一鍵開啟地圖。

#### 驗收標準

1. WHEN 儀表板時間軸渲染完成，THE Activity_Card SHALL 在地點名稱旁顯示一個可點擊的地圖連結圖示（使用 Lucide `map-pin` 或 `external-link` 圖示）。
2. WHEN 使用者點擊 Maps_Link 圖示，THE System SHALL 以新分頁（`target="_blank"`）開啟對應的 Google Maps 頁面。
3. THE Activity_Card SHALL 為 Maps_Link 加上 `rel="noopener noreferrer"` 屬性，以確保安全性。
4. WHILE 系統處於編輯模式（`isEditing === true`），THE Activity_Card SHALL 持續顯示 Maps_Link 圖示，不受編輯模式影響。

---

### 需求 3：PDF 匯出時保留 Google Maps 超連結

**使用者故事：** 身為旅遊規劃使用者，我希望匯出的 PDF 中保留可點擊的 Google Maps 超連結，以便離線查閱時仍能開啟地圖。

#### 驗收標準

1. WHEN 使用者執行 PDF 匯出，THE PDF_Export SHALL 在輸出的 PDF 中保留每個地點的 Maps_Link 為可點擊的超連結。
2. THE PDF_Export SHALL 使用 html2pdf.js 的預設超連結保留行為，不需額外處理連結轉換。
3. IF html2pdf.js 無法保留超連結，THEN THE System SHALL 在地點名稱旁以純文字顯示完整的 Maps_Link URL，確保使用者仍可手動複製連結。

---

### 需求 4：連結格式的正確性與一致性

**使用者故事：** 身為旅遊規劃使用者，我希望所有 Google Maps 連結都能正確指向對應地點，以避免開啟錯誤的地圖位置。

#### 驗收標準

1. THE System SHALL 使用統一的輔助函式（helper function）`buildMapsLink(name, lat, lng)` 產生所有 Maps_Link，確保格式一致。
2. FOR ALL 活動資料，`buildMapsLink` 函式 SHALL 在輸入相同的 `name`、`lat`、`lng` 時，每次回傳完全相同的 URL（冪等性）。
3. FOR ALL 包含有效座標的活動，`buildMapsLink` 回傳的 URL SHALL 包含 `query=<lat>,<lng>` 參數，可被 Google Maps 正確解析為地圖座標。
4. FOR ALL 缺少有效座標的活動，`buildMapsLink` 回傳的 URL SHALL 包含 `query=<URL編碼地點名稱>` 參數，可被 Google Maps 正確解析為地點搜尋。
