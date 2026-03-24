# swapActivity 顯示錯誤 Bugfix 任務清單

## 任務

- [x] 1. 探索性測試：確認 Bug 根本原因
  - [x] 1.1 在 `index.html` 中找到 `swapActivity` 函式（約第 588 行），閱讀現有實作，確認其完全依賴 `swapFallbackPool` 且未讀取 `state.swapPool`
  - [x] 1.2 在瀏覽器主控台手動模擬：設定 `state.swapPool = { sightseeing: [{ name: '真實景點A', type: 'sightseeing', time: '10:00', duration: '2小時', rating: '4.9', desc: '測試', lat: 16.4, lng: 120.6, tips: [] }] }`，再呼叫 `swapActivity(0, 0)`，觀察替換結果是否仍為假資料
  - [x] 1.3 記錄觀察結果：確認反例（替換結果來自 `swapFallbackPool` 而非 `state.swapPool`），驗證根本原因為「未實作 swapPool 讀取邏輯」

- [x] 2. 實作修復：更新 `swapActivity` 函式
  - [x] 2.1 在 `index.html` 中修改 `swapActivity(dayIdx, actIdx)` 函式，加入讀取 `state.swapPool[oldAct.type]` 的邏輯
    - 取得當天行程中已有的地點名稱集合，用於排除重複
    - 從 `state.swapPool[oldAct.type]` 中找出第一個不在當天行程中的地點，並用 `splice` 從池中移除
  - [x] 2.2 實作退回邏輯：若 `state.swapPool[oldAct.type]` 為空或不存在，從 `swapFallbackPool` 中篩選相同 `type` 的地點並隨機抽取
  - [x] 2.3 實作完全耗盡處理：若兩個池均無可用地點，呼叫 `showExhaustedDialog()`（若函式不存在則先建立，顯示 `alert` 或簡單提示）
  - [x] 2.4 確保保留行為：新活動的 `time` 設為 `oldAct.time`，`instanceId` 格式為 `swap-{Date.now()}`，最後呼叫 `selectActivity(newAct.instanceId)`

- [x] 3. 修復驗證：確認 Bug 已修復（Property 1）
  - [x] 3.1 在瀏覽器主控台重新模擬步驟 1.2 的情境，確認替換結果現在來自 `state.swapPool.sightseeing`
  - [x] 3.2 驗證 `state.swapPool.sightseeing` 在替換後長度減少 1（地點已從池中移除）
  - [x] 3.3 驗證替換後的活動 `type` 與被替換活動相同（類型匹配）
  - [x] 3.4 連續點選「換一個」兩次，確認不會重複出現相同地點

- [x] 4. 保留驗證：確認現有行為未受影響（Property 2）
  - [x] 4.1 設定 `state.swapPool = {}`（模擬 API 失敗情境），呼叫 `swapActivity`，確認仍能從 `swapFallbackPool` 取得替換活動，功能不中斷
  - [x] 4.2 確認替換後 `newAct.time === oldAct.time`（時間欄位保留）
  - [x] 4.3 確認替換後 `state.selectedActivityId` 更新為新活動的 `instanceId`（`selectActivity` 被正確呼叫）
  - [x] 4.4 確認表單重新提交後 `state.swapPool` 被重置為 `{}`（`handleGenerate` 中的重置邏輯未受影響）
  - [x] 4.5 確認非「換一個」操作（點選活動卡片、刪除活動、地圖互動）行為完全不受影響
