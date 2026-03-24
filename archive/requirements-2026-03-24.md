# 需求文件

## 簡介

目前系統在使用者進入「微調行程」模式後，點選「換一個」按鈕時，僅從一個靜態的本地備用池（`swapFallbackPool`）隨機抽取假資料，無法呈現真實地點。

本功能旨在改善此體驗：在初次呼叫 Gemini API 時，一併請求額外的備選地點清單，並依類型（景點、餐廳、咖啡廳、購物、文化等）分類儲存於 `state` 中。當使用者點選「換一個」時，系統優先從預先載入的分類備選池中取出真實地點，無需再次呼叫 Gemini，達到即時替換且資料真實的效果。

---

## 詞彙表

- **系統（System）**：本旅遊規劃應用程式（Vanilla JS 單頁應用）。
- **Gemini API**：Google Generative Language API，用於生成旅遊行程與備選地點。
- **備選池（Swap Pool）**：由 Gemini API 在初次請求時一併回傳的額外地點清單，依 `type` 分類儲存於 `state.swapPool` 中。
- **活動（Activity）**：行程中的單一地點項目，具有 `instanceId`、`type`、`name`、`lat`、`lng` 等屬性。
- **換一個（Swap）**：使用者在微調模式下，將某個活動替換為備選池中同類型地點的操作。
- **類型（Type）**：活動的分類，允許值為 `sightseeing`、`dining`、`cafe`、`shopping`、`culture`、`nature`、`accommodation`、`event`。
- **本地備用資料（Fallback）**：當 Gemini API 呼叫失敗時，系統使用的靜態假資料。
- **isMock**：`state.currentTrip.isMock`，標示目前行程是否為備用假資料。

---

## 需求

### 需求 1：初次請求時一併載入備選池

**使用者故事：** 身為使用者，我希望系統在生成行程的同時預先載入備選地點，以便「換一個」功能能立即回應且呈現真實地點。

#### 驗收標準

1. WHEN 使用者提交表單並觸發 Gemini API 請求，THE 系統 SHALL 在同一次 Prompt 中要求 Gemini 額外回傳一個 `swapPool` 物件，其結構為以 `type` 為鍵、地點陣列為值的分類清單。
2. THE Prompt SHALL 要求每個 `type` 分類至少包含 3 個備選地點，每個地點須包含 `name`、`type`、`time`（建議時段）、`duration`、`rating`、`desc`、`lat`、`lng`、`tips` 欄位。
3. WHEN Gemini API 回傳成功，THE 系統 SHALL 將解析後的 `swapPool` 儲存至 `state.swapPool`，資料結構為 `{ [type: string]: Activity[] }`。
4. IF Gemini API 回傳的 JSON 中缺少 `swapPool` 欄位或解析失敗，THEN THE 系統 SHALL 將 `state.swapPool` 設為空物件 `{}`，並繼續正常顯示行程，不得中斷流程。
5. WHEN 系統使用本地備用資料（`isMock === true`），THE 系統 SHALL 以現有的 `swapFallbackPool` 陣列依 `type` 分類後填入 `state.swapPool`，確保備用模式下「換一個」功能仍可運作。

---

### 需求 2：「換一個」從備選池取得真實地點

**使用者故事：** 身為使用者，我希望點選「換一個」時能看到真實的備選地點，而不是無意義的假名稱。

#### 驗收標準

1. WHEN 使用者在微調模式下點選某活動的「換一個」按鈕，THE 系統 SHALL 優先從 `state.swapPool[activity.type]` 中取出尚未在當天行程中出現的第一個地點作為替換項目。
2. IF `state.swapPool[activity.type]` 為空陣列或不存在，THEN THE 系統 SHALL 退回使用現有的 `swapFallbackPool` 隨機抽取，確保功能不中斷。
3. IF `state.swapPool[activity.type]` 為空陣列或不存在，且 `swapFallbackPool` 中亦無符合相同 `type` 的可用地點，THEN THE 系統 SHALL 通知使用者目前沒有更多備選地點，並詢問是否恢復為 Gemini 最初建議的原始地點。
4. WHEN 備選池中的某個地點被使用後，THE 系統 SHALL 將該地點從 `state.swapPool[activity.type]` 中移除，避免同一地點被重複替換至同一行程。
5. WHEN 替換完成，THE 系統 SHALL 保留原活動的 `time` 欄位，並為新活動產生唯一的 `instanceId`（格式：`swap-{timestamp}`）。
6. WHEN 替換完成，THE 系統 SHALL 呼叫 `selectActivity(newAct.instanceId)` 以更新選取狀態並重新渲染地圖標記。

---

### 需求 5：備選池耗盡時通知使用者並提供恢復原始建議選項

**使用者故事：** 身為使用者，當所有備選地點都已用完時，我希望系統告知我目前狀況，並讓我選擇是否恢復為 Gemini 最初建議的原始地點，以便繼續調整行程。

#### 驗收標準

1. WHEN 使用者點選「換一個」且 `state.swapPool[activity.type]` 與 `swapFallbackPool` 中均無可用地點，THE 系統 SHALL 顯示提示訊息，告知使用者「目前沒有更多備選地點」。
2. WHEN 系統顯示備選耗盡提示，THE 系統 SHALL 同時提供「恢復原始建議」與「取消」兩個操作選項供使用者選擇。
3. WHEN 使用者選擇「恢復原始建議」，THE 系統 SHALL 將該活動的內容還原為 `state.originalTrip` 中對應 `instanceId` 的原始地點資料。
4. IF `state.originalTrip` 中找不到對應的原始地點資料，THEN THE 系統 SHALL 顯示錯誤提示「無法取得原始建議資料」，並維持目前活動內容不變。
5. WHEN 使用者選擇「取消」，THE 系統 SHALL 關閉提示訊息，並維持目前活動內容不變。
6. THE 系統 SHALL 在初次生成行程成功後，將完整行程資料的深層複本儲存至 `state.originalTrip`，以供後續恢復使用。

---

### 需求 3：備選池的資料格式驗證

**使用者故事：** 身為開發者，我希望系統能防禦性地處理 Gemini 回傳的備選池資料，避免格式錯誤導致功能異常。

#### 驗收標準

1. WHEN 系統解析 Gemini 回傳的 `swapPool`，THE 系統 SHALL 驗證每個地點項目是否包含 `name`（字串）、`lat`（數字）、`lng`（數字）欄位。
2. IF 某個地點項目缺少 `lat` 或 `lng` 欄位，或其值不為數字，THEN THE 系統 SHALL 為該項目補上基於目的地中心點的隨機偏移座標（偏移範圍：±0.05 度）。
3. IF 某個地點項目缺少 `name` 欄位，THEN THE 系統 SHALL 將該項目從備選池中排除，不得將無名稱地點加入 `state.swapPool`。
4. THE 系統 SHALL 對每個通過驗證的備選地點執行 `formatTripData` 相同的欄位補全邏輯（補全 `instanceId`、`icon`、`mappedTips`）。

---

### 需求 4：備選池狀態的生命週期管理

**使用者故事：** 身為使用者，我希望每次重新規劃行程時，備選池都能正確重置，不會殘留上次的資料。

#### 驗收標準

1. WHEN 使用者點選「重新規劃」並提交新的表單，THE 系統 SHALL 在呼叫 Gemini API 之前將 `state.swapPool` 重置為空物件 `{}`。
2. WHEN `finishGenerate` 被呼叫，THE 系統 SHALL 以新行程對應的備選池資料覆寫 `state.swapPool`。
3. THE 系統 SHALL 確保 `state.swapPool` 在任何時刻都是一個物件（不為 `null` 或 `undefined`），以防止 `swapActivity` 函式存取時發生型別錯誤。
