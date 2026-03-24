# 實作計畫：gemini-itinerary-swap

## 概覽

在 `index.html` 的 `<script>` 區塊中，依序修改與新增函式，使「換一個」功能從 Gemini API 預先載入的真實備選池取得地點，並在備選耗盡時提供恢復原始建議的選項。

## 任務

- [x] 1. 擴充 `state` 物件並重置備選池
  - 在 `state` 初始化物件中新增 `swapPool: {}` 與 `originalTrip: null` 兩個欄位
  - 在 `handleGenerate` 函式呼叫 `callGeminiAPI` 之前，加入 `state.swapPool = {}; state.originalTrip = null;`
  - _需求：4.1、4.3_

- [x] 2. 修改 Gemini Prompt 以請求備選池
  - [x] 2.1 在 `callGeminiAPI` 的 `promptText` 中加入 `swapPool` 輸出要求段落
    - 要求 Gemini 在 JSON 根層級回傳 `swapPool` 物件，結構為以 `type` 為鍵、地點陣列為值
    - 每個 type 分類至少 3 個備選地點，每個地點須含 `name`、`type`、`time`、`duration`、`rating`、`desc`、`lat`、`lng`、`tips` 欄位
    - _需求：1.1、1.2_

- [x] 3. 新增 `validateSwapPool` 函式
  - [x] 3.1 實作 `validateSwapPool(rawSwapPool, baseLat, baseLng)` 函式
    - 過濾掉缺少 `name` 欄位（或 `name` 為空字串）的項目
    - 對缺少有效 `lat`/`lng`（非數字）的項目，補上 `baseLat ± 0.05` 與 `baseLng ± 0.05` 的隨機偏移座標
    - 對每個通過驗證的項目執行與 `formatTripData` 相同的欄位補全邏輯（`instanceId`、`icon`、`mappedTips`）
    - 回傳 `{ [type: string]: Activity[] }` 結構
    - _需求：3.1、3.2、3.3、3.4_

  - [x] 3.2 為 `validateSwapPool` 撰寫屬性測試
    - **屬性 4：無名稱地點被排除**
    - **驗證：需求 3.3**

  - [x] 3.3 為 `validateSwapPool` 撰寫屬性測試
    - **屬性 5：無效座標補全後為數字**
    - **驗證：需求 3.2**

- [x] 4. 修改 `finishGenerate` 以儲存備選池與原始行程
  - [~] 4.1 在 `finishGenerate(tripData)` 中加入 `state.swapPool` 與 `state.originalTrip` 的賦值邏輯
    - 取得目的地中心點座標（`baseLat`、`baseLng`）
    - 呼叫 `validateSwapPool(tripData.swapPool || {}, baseLat, baseLng)` 並將結果存入 `state.swapPool`
    - 將 `deepClone(tripData)` 存入 `state.originalTrip`
    - _需求：1.3、1.4、5.6_

  - [~] 4.2 為 `finishGenerate` 撰寫屬性測試
    - **屬性 6：swapPool 永遠為物件**
    - **驗證：需求 4.3**

  - [~] 4.3 為 `finishGenerate` 撰寫屬性測試
    - **屬性 8：恢復原始建議為深層複本**
    - **驗證：需求 5.6**

- [~] 5. 修改 `generateFallback` 以建構分類備選池
  - 在 `generateFallback` 回傳前，將 `swapFallbackPool` 依 `type` 分類後填入 `raw.swapPool`
  - _需求：1.5_

  - [~] 5.1 為 `generateFallback` 撰寫單元測試
    - **屬性 7：備用模式備選池可用**
    - **驗證：需求 1.5**

- [~] 6. 檢查點 — 確認所有測試通過，如有疑問請詢問使用者

- [~] 7. 新增 `showExhaustedDialog` 與 `restoreOriginalActivity` 函式
  - [~] 7.1 實作 `showExhaustedDialog(dayIdx, actIdx)` 函式
    - 在 DOM 中動態插入 Modal，包含提示文字「目前沒有更多備選地點」
    - 提供「恢復原始建議」按鈕（呼叫 `restoreOriginalActivity(dayIdx, actIdx)`）與「取消」按鈕（關閉 Modal）
    - _需求：5.1、5.2、5.5_

  - [~] 7.2 實作 `restoreOriginalActivity(dayIdx, actIdx)` 函式
    - 取得 `currentAct = state.currentTrip.schedule[dayIdx].activities[actIdx]`
    - 在 `state.originalTrip` 中搜尋對應 `instanceId` 的原始地點
    - 若找到：替換 `state.currentTrip.schedule[dayIdx].activities[actIdx]`，呼叫 `selectActivity`
    - 若未找到：顯示錯誤提示「無法取得原始建議資料」，維持現狀
    - _需求：5.3、5.4_

  - [~] 7.3 為 `restoreOriginalActivity` 撰寫單元測試
    - 測試找到對應 instanceId 時正確還原活動
    - 測試找不到 instanceId 時顯示錯誤提示且不修改現有活動
    - _需求：5.3、5.4_

- [~] 8. 修改 `swapActivity` 以使用備選池
  - [~] 8.1 重寫 `swapActivity(dayIdx, actIdx)` 的核心邏輯
    - 取得 `oldAct` 與當天已有地點名稱集合
    - 從 `state.swapPool[oldAct.type]` 找出第一個不在當天行程中的地點
    - 若找到：取出、從池中移除、組裝 `newAct`（保留 `time`，產生 `swap-{timestamp}` instanceId）、呼叫 `selectActivity`
    - 若未找到：嘗試從 `swapFallbackPool` 中找同 `type` 的項目
    - 若仍未找到：呼叫 `showExhaustedDialog(dayIdx, actIdx)`
    - _需求：2.1、2.2、2.3、2.4、2.5、2.6_

  - [~] 8.2 為 `swapActivity` 撰寫屬性測試
    - **屬性 1：備選池取出後縮減**
    - **驗證：需求 2.4**

  - [~] 8.3 為 `swapActivity` 撰寫屬性測試
    - **屬性 2：替換保留原始時間**
    - **驗證：需求 2.5**

  - [~] 8.4 為 `swapActivity` 撰寫屬性測試
    - **屬性 3：替換後 instanceId 唯一**
    - **驗證：需求 2.5**

- [~] 9. 最終檢查點 — 確認所有測試通過，如有疑問請詢問使用者

## 備註

- 標記 `*` 的子任務為選用，可跳過以加速 MVP 開發
- 屬性測試使用 fast-check 函式庫，每個屬性最少執行 100 次迭代
- 每個屬性測試須以 `// Feature: gemini-itinerary-swap, Property {N}: {描述}` 格式標記
- 所有修改均在 `index.html` 的 `<script>` 區塊內完成，不引入外部模組或建置工具
