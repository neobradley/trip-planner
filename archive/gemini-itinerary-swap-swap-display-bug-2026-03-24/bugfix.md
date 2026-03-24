# Bugfix 需求文件

## 簡介

在「微調行程」功能中，使用者點選「換一個」按鈕時，系統應從 Gemini API 預先回傳的 `swapPool` 中取得對應類型的備選活動並顯示。然而，實際行為是 `swapActivity` 函式完全忽略 `state.swapPool`，直接從靜態的本地備用池（`swapFallbackPool`）隨機抽取假資料（如「網紅打卡景點」、「社群媒體熱門拍照地點」等通用描述），導致即使 Gemini API 已成功回傳真實備選地點，使用者看到的仍是無意義的假資料。

---

## Bug 分析

### 當前行為（缺陷）

1.1 WHEN 使用者在微調模式下點選某活動的「換一個」按鈕，THEN 系統直接從 `swapFallbackPool` 隨機抽取假資料作為替換活動，完全不讀取 `state.swapPool`

1.2 WHEN `state.swapPool` 已包含 Gemini API 回傳的真實備選地點，THEN 系統仍顯示 `swapFallbackPool` 中的通用假資料（如「網紅打卡景點」、「社群媒體熱門拍照地點」），而非真實地點

1.3 WHEN 替換活動時，THEN 系統不考慮被替換活動的 `type`，從 `swapFallbackPool` 中隨機抽取任意類型的假資料，可能導致類型不符（例如將餐廳替換為景點）

### 預期行為（正確）

2.1 WHEN 使用者在微調模式下點選某活動的「換一個」按鈕，THEN 系統 SHALL 優先從 `state.swapPool[activity.type]` 中取出尚未在當天行程中出現的第一個地點作為替換活動

2.2 WHEN `state.swapPool[activity.type]` 為空陣列或不存在，THEN 系統 SHALL 退回使用 `swapFallbackPool` 中相同 `type` 的地點，確保功能不中斷

2.3 WHEN `state.swapPool[activity.type]` 與 `swapFallbackPool` 中均無可用地點，THEN 系統 SHALL 呼叫 `showExhaustedDialog` 通知使用者目前沒有更多備選地點

2.4 WHEN 備選池中的某個地點被使用後，THEN 系統 SHALL 將該地點從 `state.swapPool[activity.type]` 中移除，避免重複替換

### 未變更行為（迴歸防護）

3.1 WHEN 替換完成，THEN 系統 SHALL CONTINUE TO 保留原活動的 `time` 欄位，並為新活動產生唯一的 `instanceId`（格式：`swap-{timestamp}`）

3.2 WHEN 替換完成，THEN 系統 SHALL CONTINUE TO 呼叫 `selectActivity(newAct.instanceId)` 以更新選取狀態並重新渲染地圖標記

3.3 WHEN 使用者點選「換一個」且 `state.swapPool` 為空物件（例如 API 失敗後使用備用資料），THEN 系統 SHALL CONTINUE TO 從 `swapFallbackPool` 抽取資料作為備援，確保功能不中斷

3.4 WHEN 使用者重新規劃行程並提交表單，THEN 系統 SHALL CONTINUE TO 在呼叫 Gemini API 之前將 `state.swapPool` 重置為空物件 `{}`
