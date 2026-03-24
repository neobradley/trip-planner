# swapActivity 顯示錯誤 Bugfix 設計文件

## 概覽

`swapActivity` 函式在執行「換一個」時，完全忽略 `state.swapPool`（Gemini API 已回傳的真實備選地點），直接從靜態的 `swapFallbackPool` 隨機抽取假資料。修復策略是讓 `swapActivity` 優先讀取 `state.swapPool[activity.type]`，僅在池耗盡時才退回 `swapFallbackPool`，並在完全無可用地點時呼叫 `showExhaustedDialog`。

所有修改均在 `index.html` 的 `<script>` 區塊內完成。

---

## 詞彙表

- **Bug_Condition (C)**：觸發錯誤的條件——`swapActivity` 被呼叫，且 `state.swapPool[activity.type]` 存在可用地點，但函式卻從 `swapFallbackPool` 隨機抽取假資料
- **Property (P)**：正確行為——替換後的活動應來自 `state.swapPool[activity.type]`，且該地點應從池中移除
- **Preservation**：不受修復影響的現有行為——保留原活動 `time`、產生唯一 `instanceId`、呼叫 `selectActivity`、`swapPool` 為空時退回 `swapFallbackPool`
- **swapActivity(dayIdx, actIdx)**：`index.html` 中負責替換指定活動的函式，目前有缺陷
- **state.swapPool**：以 `type` 為鍵的備選地點物件，由 `finishGenerate` 從 Gemini API 回傳資料填入
- **swapFallbackPool**：靜態假資料陣列，僅作為最後備援使用

---

## Bug 詳情

### Bug 條件

當使用者點選「換一個」時，`swapActivity` 函式不讀取 `state.swapPool`，而是直接從 `swapFallbackPool` 隨機抽取任意類型的假資料，導致即使 Gemini API 已回傳真實備選地點，使用者看到的仍是通用假資料（如「網紅打卡景點」、「社群媒體熱門拍照地點」）。

**形式化規格：**
```
FUNCTION isBugCondition(dayIdx, actIdx)
  INPUT: dayIdx（天索引）, actIdx（活動索引）
  OUTPUT: boolean

  oldAct := state.currentTrip.schedule[dayIdx].activities[actIdx]
  pool   := state.swapPool[oldAct.type]

  RETURN pool 存在 AND pool.length > 0
         AND 函式實際使用的是 swapFallbackPool 而非 pool
END FUNCTION
```

### 具體範例

- **範例 1（核心缺陷）**：`state.swapPool.sightseeing` 有 3 個真實景點，使用者點選一個 `sightseeing` 活動的「換一個」，預期顯示真實景點，實際顯示「網紅打卡景點」（假資料）
- **範例 2（類型不符）**：使用者點選 `dining` 活動的「換一個」，預期顯示同類型餐廳，實際可能顯示 `cafe` 或 `shopping` 類型的假資料
- **範例 3（池耗盡應退回）**：`state.swapPool.cafe` 為空，使用者點選 `cafe` 活動的「換一個」，預期退回 `swapFallbackPool` 中的 `cafe` 項目，實際行為與預期相同（但這是偶然，因為目前完全依賴 `swapFallbackPool`）
- **邊界情況**：`state.swapPool` 為空物件 `{}`（API 失敗後），預期退回 `swapFallbackPool`，此行為應繼續保留

---

## 預期行為

### 保留需求

**不應改變的行為：**
- 替換完成後，新活動的 `time` 欄位應與被替換活動相同
- 替換完成後，應呼叫 `selectActivity(newAct.instanceId)` 更新選取狀態並重新渲染地圖
- `state.swapPool[activity.type]` 為空或不存在時，應退回 `swapFallbackPool` 中相同 `type` 的地點
- `state.swapPool` 為空物件時（API 失敗情境），功能不中斷

**範圍：**
所有不涉及「換一個」按鈕的操作均不受此修復影響，包含：
- 使用者直接點選活動卡片（`selectActivity`）
- 刪除活動（`deleteActivity`）
- 地圖標記互動
- 表單提交與行程重新規劃

---

## 假設根本原因

根據 Bug 描述與程式碼分析，最可能的原因如下：

1. **未實作 swapPool 讀取邏輯**：`swapActivity` 函式在原始設計中可能是佔位實作，直接使用 `swapFallbackPool` 作為臨時方案，後來 `state.swapPool` 的填入邏輯（`finishGenerate`、`validateSwapPool`）雖已完成，但 `swapActivity` 本身從未被更新以讀取它

2. **缺少類型過濾**：現有程式碼使用 `Math.random()` 從整個 `swapFallbackPool` 抽取，未依 `oldAct.type` 過濾，導致類型不符問題

3. **缺少重複地點排除**：現有程式碼未檢查當天行程中已有的地點，可能產生重複

4. **缺少池縮減邏輯**：現有程式碼未將已使用的地點從 `state.swapPool` 中移除，即使修復後也需要加入此邏輯

---

## 正確性屬性

Property 1: Bug Condition - swapPool 優先替換

_For any_ 呼叫 `swapActivity(dayIdx, actIdx)` 的情境，其中 `state.swapPool[oldAct.type]` 存在且長度大於 0，修復後的函式 SHALL 從 `state.swapPool[oldAct.type]` 取出第一個可用地點作為替換活動，並將該地點從池中移除。

**Validates: Requirements 2.1, 2.4**

Property 2: Preservation - 非 swapPool 情境行為不變

_For any_ 呼叫 `swapActivity(dayIdx, actIdx)` 的情境，其中 `state.swapPool[oldAct.type]` 為空或不存在（isBugCondition 返回 false），修復後的函式 SHALL 產生與原始函式相同的結果——退回 `swapFallbackPool` 或呼叫 `showExhaustedDialog`，保留原有的 `time` 欄位、`instanceId` 產生邏輯與 `selectActivity` 呼叫。

**Validates: Requirements 3.1, 3.2, 3.3, 3.4**

---

## 修復實作

### 所需變更

假設根本原因分析正確：

**檔案**：`index.html`

**函式**：`swapActivity(dayIdx, actIdx)`

**具體變更**：

1. **讀取 swapPool**：在函式開頭取得 `oldAct.type`，並查找 `state.swapPool[oldAct.type]`
   - 若池存在且有可用地點，取出第一個地點（`pool.shift()` 或 `splice(0,1)`）
   - 排除當天行程中已存在的地點（依 `name` 比對）

2. **退回邏輯**：若 `state.swapPool[oldAct.type]` 為空或不存在，從 `swapFallbackPool` 中篩選相同 `type` 的地點
   - 使用 `swapFallbackPool.filter(item => item.type === oldAct.type)` 取得候選清單
   - 若有候選，隨機抽取一個

3. **完全耗盡處理**：若 `swapPool` 與 `swapFallbackPool` 均無可用地點，呼叫 `showExhaustedDialog()`

4. **保留欄位**：將 `newAct.time` 設為 `oldAct.time`，並產生新的 `instanceId`（格式：`swap-{Date.now()}`）

5. **更新狀態**：替換 `state.currentTrip.schedule[dayIdx].activities[actIdx]`，並呼叫 `selectActivity(newAct.instanceId)` 重新渲染

---

## 測試策略

### 驗證方法

測試策略分兩階段：先在未修復的程式碼上觀察反例以確認根本原因，再驗證修復後的正確性與保留行為。

### 探索性 Bug 條件檢查

**目標**：在實作修復前，先找出能展示 Bug 的反例，確認或推翻根本原因分析。

**測試計畫**：模擬 `state.swapPool` 已填入真實備選地點的情境，呼叫 `swapActivity`，斷言替換結果來自 `swapPool` 而非 `swapFallbackPool`。在**未修復**的程式碼上執行，預期測試失敗。

**測試案例**：
1. **swapPool 有資料時應使用 swapPool**：設定 `state.swapPool.sightseeing` 有 2 個地點，呼叫 `swapActivity`，斷言結果來自 `swapPool`（未修復時失敗）
2. **類型應匹配**：設定 `dining` 活動，斷言替換結果的 `type` 為 `dining`（未修復時可能失敗）
3. **已使用地點應從池中移除**：呼叫兩次 `swapActivity`，斷言第二次不重複第一次的地點（未修復時失敗）
4. **邊界情況：池耗盡**：設定 `swapPool.cafe` 為空，斷言退回 `swapFallbackPool`（未修復時偶然通過）

**預期反例**：
- 替換結果的 `name` 為 `swapFallbackPool` 中的假資料（如「網紅打卡景點」）而非 `swapPool` 中的真實地點
- 可能原因：`swapActivity` 完全未讀取 `state.swapPool`

### 修復檢查

**目標**：驗證所有 Bug 條件成立的輸入，修復後函式均產生預期行為。

**偽代碼：**
```
FOR ALL (dayIdx, actIdx) WHERE isBugCondition(dayIdx, actIdx) DO
  result := swapActivity_fixed(dayIdx, actIdx)
  ASSERT result 來自 state.swapPool[oldAct.type]
  ASSERT state.swapPool[oldAct.type] 長度減少 1
  ASSERT result.time = oldAct.time
  ASSERT result.instanceId 以 "swap-" 開頭
END FOR
```

### 保留檢查

**目標**：驗證所有 Bug 條件不成立的輸入，修復後函式與原始函式行為相同。

**偽代碼：**
```
FOR ALL (dayIdx, actIdx) WHERE NOT isBugCondition(dayIdx, actIdx) DO
  ASSERT swapActivity_original(dayIdx, actIdx) 行為 = swapActivity_fixed(dayIdx, actIdx) 行為
END FOR
```

**測試方法**：建議使用屬性測試（Property-Based Testing），因為：
- 可自動產生大量 `swapPool` 為空的情境組合
- 能捕捉手動測試可能遺漏的邊界情況
- 對「所有非 Bug 輸入行為不變」提供強力保證

**測試案例**：
1. **swapPool 為空物件時退回 swapFallbackPool**：觀察未修復程式碼的行為，再驗證修復後相同
2. **time 欄位保留**：驗證替換後 `newAct.time === oldAct.time`
3. **selectActivity 被呼叫**：驗證替換後 `state.selectedActivityId` 更新為新活動的 `instanceId`
4. **重新規劃時 swapPool 重置**：驗證表單提交後 `state.swapPool` 為 `{}`

### 單元測試

- 測試 `swapPool` 有資料時，`swapActivity` 正確取出並移除第一個地點
- 測試 `swapPool` 為空時，退回 `swapFallbackPool` 中相同 `type` 的地點
- 測試 `swapPool` 與 `swapFallbackPool` 均無可用地點時，呼叫 `showExhaustedDialog`
- 測試替換後 `time` 欄位保留、`instanceId` 格式正確

### 屬性測試

- 產生隨機 `swapPool` 狀態，驗證替換結果的 `type` 始終與被替換活動相同
- 產生隨機行程配置，驗證 `swapPool` 有資料時不使用 `swapFallbackPool`
- 多次呼叫 `swapActivity`，驗證 `swapPool` 中的地點不重複出現

### 整合測試

- 完整流程：API 成功回傳 `swapPool` → 進入微調模式 → 點選「換一個」→ 驗證顯示真實地點
- 完整流程：API 失敗使用備用資料 → 進入微調模式 → 點選「換一個」→ 驗證退回 `swapFallbackPool`
- 驗證替換後地圖標記正確更新至新活動位置
