專案技能樹與相依性指南 (Skills & Dependencies)

AI Agent 在維護與開發此專案時，需具備以下技術棧的深度知識：

1. 原生前端開發 (Vanilla Web Development)

DOM Manipulation: 熟練使用 document.getElementById, innerHTML, classList 進行動態視圖切換與陣列渲染。

Event Handling: 熟悉內聯事件綁定 (onclick, onchange) 以及表單的 onsubmit 阻擋預設行為 (e.preventDefault())。

2. 樣式與視覺 (Styling & Visuals)

Tailwind CSS (CDN): 透過 Utility classes 進行快速排版、RWD 響應式設計與基礎動畫 (animate-pulse, animate-spin)。

Lucide Icons: 熟悉 lucide.createIcons() 的呼叫時機（在任何 DOM 動態更新後必須重新呼叫，以渲染 <i data-lucide="..."></i>）。

3. 地圖引擎 (Leaflet.js)

熟悉核心 API：L.map(), L.tileLayer(), L.marker(), L.polyline(), L.divIcon()。

掌握地圖視角控制：leafletMap.flyTo([lat, lng], zoom) 與 leafletMap.fitBounds()。

注意：每次重新規劃行程時，必須先 leafletMap.removeLayer() 清除舊的 marker 與 polyline，避免記憶體洩漏與畫面重疊。

4. 文件處理 (html2pdf.js)

了解非同步的 Promise-based PDF 生成流程。

熟悉設定參數：margin, html2canvas: { scale: 2, useCORS: true }, jsPDF: { unit: 'mm', format: 'a4' }。

注意：確保要匯出的 DOM 節點（如 #pdf-content）在匯出時必須處於可見狀態（非 display: none），否則套件會截取到空白畫面。
