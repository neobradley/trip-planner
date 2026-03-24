Specification By Example (SBE) / BDD 驗證條件

本文件定義了系統必須無條件滿足的三大極端業務場景。任何程式碼的修改或重構，都必須通過以下測試標準。

情境一：大型團體的嚴格飲食限制 (Large Group + Strict Dietary)

假設 (Given) 使用者輸入的 partySize 為 "6+" 且 dietary 包含 "素食/蔬食" 或 "純素食"。

當 (When) 系統呼叫 API 規劃每日的 dining (午/晚餐) 行程時。

那麼 (Then) 系統生成的餐廳指南必須：

100% 提供素食選項或為純素餐廳。

在 tips 陣列中，必須包含 category: "success" 的提示，明確指出「已確認可容納團體/適合多人聚餐」。

tips 應包含 category: "warning" 提醒「需提前訂位」。

情境二：攝影愛好者的黃金時刻 (Photography + Golden Hour)

假設 (Given) 使用者的 interests 陣列中包含 "photography" (📸 攝影/取景)。

當 (When) 系統安排 type: "sightseeing" 或 type: "nature" 等戶外景點時。

那麼 (Then) 系統必須：

將最具視覺震撼力的戶外行程優先排定在晨間 (約 07:00 - 08:00) 或日落前 (約 16:00 - 17:30) 的黃金時刻。

該景點的 tips 必須包含說明，例如「絕佳日落拍攝點 (Golden Hour)」。

情境三：絕對錨點與不可變動事件 (Fixed Anchor Events)

假設 (Given) 使用者在 fixedEvents 中加入了特定事件，例如 { day: 2, time: "14:00", name: "海外婚禮" }。

當 (When) 系統進行全局排程演算時。

那麼 (Then) 系統必須：

在第 2 天的行程中，精準插入一個 type: "event" 的節點，時間為 14:00，名稱包含該事件。

系統必須自動為該事件前後預留交通與準備的緩衝時間（例如：12:30 到 16:30 之間不得安排其他消耗性行程）。

該事件的 tips 需標示為「已鎖定此事件」。
