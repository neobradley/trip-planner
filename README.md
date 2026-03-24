客製化旅遊行程系統 (AI-Powered Travel Itinerary Planner)

專案概述 (Overview)

這是一款結合 Generative AI 與實時搜尋能力的客製化旅遊行程推薦系統。系統透過收集使用者的具體條件（天數、預算、住宿、特殊飲食、興趣、絕對錨點事件），利用 Gemini API 進行地理與情境演算，並輸出帶有真實地圖座標與注意事項的結構化行程表。

核心功能 (Core Features)

實時資料接地 (Google Search Grounding)：串接 Gemini 2.5 Flash，獲取全世界真實存在的景點與餐廳評價。

BDD 約束演算 (Constraint Satisfaction)：嚴格遵守使用者的極端條件（如：6人以上的純素食餐廳、攝影愛好者的黃金時刻排程）。

實境地圖互動 (Real-Map Integration)：整合 Leaflet.js 與 OpenStreetMap，提供沉浸式的 flyTo 座標飛越體驗與路線繪製。

行程微調 (Itinerary Micro-adjustment)：支援單一景點的「換一個 (Swap)」與「刪除 (Delete)」，並動態重繪地圖。

所見即所得匯出 (WYSIWYG PDF Export)：利用 html2pdf.js 將前端行程時間軸完美轉譯為 A4 尺寸的文件下載。

技術棧 (Tech Stack)

前端介面: HTML5, Vanilla JavaScript (ES6+), Tailwind CSS (CDN)

圖示系統: Lucide Icons (CDN)

地圖引擎: Leaflet.js (OpenStreetMap Tiles)

文件匯出: html2pdf.js

AI 引擎: Google Gemini API (gemini-2.5-flash-preview-09-2025)

快速啟動 (Quick Start)

本專案目前採用 Zero-Build-Step (零編譯) 架構。

將程式碼儲存為 index.html。

直接使用任何現代瀏覽器開啟該檔案。

在畫面頂部輸入您的 Gemini API Key 即可啟用實時 AI 搜尋功能。
