# StickerBoard — 技術總結

> 一份單一 HTML 檔案如何長出一個可用的印刷工具

---

## 起點

這個專案不是從「我要寫一個工具」開始的，而是從「/sticker 不寄台灣，能不能繞過去」這個問題開始的。從 Claude 畫了幾張貼紙草圖，到最後輸出一個單檔 `index.html`（初版 820 行，目前約 970 行），整個過程是三個 AI（Claude、ChatGPT、Gemini）跟一個人（Doe）透過對話把產品規格一輪一輪收斂出來的。

這份文件記錄幾個比較值得留下來的決策。

---

## 架構：為什麼是單一 HTML 檔案

最早有討論過 React + Vite 的選項，但目標使用者是「想把 AI 圖印成貼紙的人」，不是開發者。單一 HTML 的好處是：

- 雙擊就能跑，不需要任何環境
- GitHub Pages、Netlify Drop、本地都能用，零部署成本
- 不需要 node_modules，不需要 build step
- 任何人都能看到全部原始碼，改一行馬上生效

代價是狀態管理沒有框架，全靠一個全域 `S` 物件 + 手動 DOM 操作。820 行內這還算可控。

---

## 雙畫布渲染

這是整個工具最重要的架構決策，由 GPT 在規格討論時提出：**預覽層和輸出層分離**。

```
使用者操作 → 修改 S 物件 → scheduleUpdate()（debounce 90ms）
                              ↓
                       renderCanvas(previewCanvas, C, previewScale)
                       // 縮小版，立即回饋

下載時 → renderCanvas(document.createElement('canvas'), C, 1)
         // 全解析度，只在需要時才算
```

如果直接把全解析度的 1200×1800 畫布做為預覽，A4 + 多張大圖的情境下瀏覽器會明顯卡頓。分層之後預覽跑縮圖，輸出才跑完整版，兩者共用同一套 `renderCanvas()` 邏輯，scale 參數控制。

---

## 版面計算引擎

整個排版邏輯集中在一個 `compute()` 函數，輸入 `S`，輸出所有像素座標。兩種模式：

**固定貼紙尺寸模式**（預設）
```
cellW = stickerWcm × 10 × pxPerMM
cols  = floor((canvasW + spacing) / (cellW + spacing))
offsetX = (canvasW − cols × cellW − (cols−1) × spacing) / 2
```
自動置中，不管設幾 cm 都不會超出畫布。

**固定格數模式**
```
cellW = (canvasW − (cols+1) × spacing) / cols
offsetX = spacing
```
外框間距跟格間距相同，視覺上最均勻。

兩種模式的格子座標計算邏輯完全一樣，只有 `cellW / cellH / offset` 的來源不同。

---

## Canvas 圓角裁切

貼紙圓角用 `ctx.clip()` + 手刻的 `roundedRectPath()`，而不是 `ctx.roundRect()`，原因是跨瀏覽器相容性。路徑用四段 `arcTo()` 拼出來：

```javascript
function rrPath(ctx, x, y, w, h, r) {
  r = Math.min(r, Math.min(w, h) / 2);
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + w - r, y);
  ctx.arcTo(x + w, y, x + w, y + r, r);
  // ... 四個角
  ctx.closePath();
}
```

`r = 0` 時退化成直角矩形，`r = min(w,h)/2` 時是正圓。圓角半徑 slider 的 50% 直接對應這個公式的最大值。

---

## AI 生圖助手的設計思路

最初 Gemini 建議的是一組通用咒語（white background + sticker outline + vector style），但「vector style」會把水彩風、寫實風的圖完全改掉。最後的設計是：

- **固定部分**（所有工具通用）：`die-cut sticker design, isolated on pure white background, centered composition, clear solid outlines`
- **可變部分**（風格選擇）：7 種，各自有獨立的關鍵詞組
- **工具版本**（比例/語法差異）：ChatGPT 說 `square format`，Midjourney 加 `--ar 1:1 --v 6`，Gemini 不需要，以此類推

📸 **參考照片模式**是後來加的。這個功能不涉及任何圖片上傳或處理，純粹是在 prompt 最前面插入 `Please analyze the attached photo and generate...` 這樣的句子，告訴 AI 去看使用者自己在 AI 介面上傳的照片。Midjourney 版本改用 `--cref [網址]`，並附上取得網址的說明（上傳到 Discord → 右鍵複製）。

---

## Typography Sticker：資料與函數分離的 Prompt Engine

文字貼紙（2026-07-19）是 StickerBoard 的第二條創作軸。定位刻意不是「模板機」，而是「一句話 → AI 理解 → 設計感貼紙」，所以架構上做了三件事：

**1. 三個維度解耦**

```
Emotion（情緒）    控制心情，不控制角色
Aesthetic（風格）  控制視覺，不控制角色
Route（路線）      控制生成策略，不控制內容
```

情緒和風格是自由組合的（5 × 4），不會出現「梗圖就一定厭世」這種綁死。

**2. 純資料 + 純函數，為抽檔預留**

所有可調內容集中在 `TY_DATA`（emotions / aesthetics / overrides）和 `TY_TAIL`（各工具的尾綴），組裝邏輯是一個沒有 DOM 依賴的純函數：

```javascript
tyAssemble(recipe, tool) → { route:'A', a } | { route:'B', b1, b2 }
// recipe = { text, emotion, aesthetic, route }
```

DOM 那一層（`buildTy()` 和事件綁定）另外放。等 Product B（LINE Helper）開工時，`TY_DATA` + `tyAssemble` 可以原封不動搬進 `shared/recipe-data.js` / `shared/recipe.js` 兩條產品線共用，`index.html` 只留 UI glue。

`TY_DATA.overrides` 是逃生口：某個「情緒|風格」組合出圖不佳時，用 `'meltdown|ink': { a, b1, b2 }` 覆蓋掉預設模板，不必動架構。

**3. Route A / Route B，而不是「GPT 模式 / Gemini 模式」**

模型只是執行者，不該寫進產品架構。所以路線是按生成策略分的：

- **Route A 圖文融合** — 文字直接進 prompt，AI 一次畫完。融合度最高，適合單張梗圖貼紙。
- **Route B 圖字分離** — Step 1 生「中間留白、絕對不要有文字」的底圖，Step 2 在同一段對話裡加字。字比較不容易翻車，適合長句和文青風。

風格會帶入建議路線（`aesthetics[x].route`，例如水墨預設走 B），但使用者仍可手動切換——Preset 決定的是流程，不是情緒。

已知限制寫在 UI 提示裡：Midjourney 中文字支援不佳，而且無法對同一張圖續改，Route B 的 Step 2 會提示改用 ChatGPT / Gemini。

完整藍圖（含 Layer 4 Character Preference、Typography v2/v3 範圍）見 `stickerboard architecture notes.md`。

---

## 三欄式版面：搬 DOM 而不是複製 DOM

寬螢幕上左欄要同時塞排版設定和 AI 助手，捲動距離太長。解法是加一個 300px 的右欄 `<aside id="rpanel">`，寬螢幕時把 AI 助手整塊搬過去：

```javascript
const mqWide = matchMedia('(min-width:1101px)');
function placeAI(){
  const ai = document.getElementById('aiSec');
  if (mqWide.matches) document.getElementById('rpanel').appendChild(ai);
  else { const imgSec = document.getElementById('imgSec');
         imgSec.parentNode.insertBefore(ai, imgSec); }   // 搬回左欄原位
}
mqWide.addEventListener('change', placeAI);
placeAI();
```

關鍵是 `appendChild` 移動的是同一個節點，不是兩份 HTML 各寫一次。所有 `id` 唯一、事件綁定不需要重綁，也不會有「窄螢幕改了設定、寬螢幕看到舊值」的雙份狀態問題。<1100px 時搬回左欄，手機版直排行為完全不變。

---

## Clipboard API 修正

第一版的複製邏輯在本地 `file://` 環境下失效，原因是：

1. `navigator.clipboard.writeText()` 要求 secure context（HTTPS 或 localhost）
2. 備用的 `document.execCommand('copy')` 如果放在 `.catch()` 的 callback 裡，瀏覽器認為不是使用者直接觸發，同樣會擋

修正方式：先用 `window.isSecureContext` 判斷，是的話走現代 API，不是的話在點擊事件的同步路徑裡直接呼叫 `execCommand`：

```javascript
if (navigator.clipboard && window.isSecureContext) {
  navigator.clipboard.writeText(text).then(ok).catch(fallback);
} else {
  fallback(); // 同步執行，不進 Promise
}
```

---

## HTML 維護的教訓

整個 `index.html` 是透過多輪 `str_replace` 迭代出來的。中間出現過一次 `<div class="sec">` 開頭連同標題被比對字串一起吃掉的情況，導致 DOM 樹錯亂，`document.getElementById('clearBtn')` 拿到 null，整個右側預覽無法渲染。

之後每輪修改後都加了一個 Python 驗證腳本：

```python
class DivChecker(HTMLParser):
    # 追蹤 depth，depth != 0 或 depth 變負數都是錯誤
```

加上對所有關鍵 `id` 的存在性檢查，讓每次 str_replace 後都能立刻確認結構正確。

---

## 協作模式回顧

這個專案的開發過程大致是：

- **Doe**：提需求、做使用者決策、定義「小白友善」的邊界
- **Claude**：主要實作者，負責評估其他 AI 的提案並落地成 code
- **ChatGPT**：產品規格討論，提出「預覽/輸出分離」和「AI 助手定位不應是 prompt 工具」兩個關鍵觀點
- **Gemini**：功能提案，提出 `die-cut sticker design` 關鍵詞升級、Gemini 工具支援、參考照片功能，以及診斷出 Clipboard API 的根本原因

每個 AI 有不同的提案風格，最後由 Doe 帶回 Claude 做最終評估和整合。

---

## 現況（2026-07-22 · v0.3.1）

| 項目 | 數值 |
|------|------|
| 檔案數 | 10（index.html / README.md / TECH.md / CHANGELOG.md / stickerboard architecture notes.md / LICENSE / .gitignore ＋ docs/ 三份交接文件） |
| index.html 行數 | 978 |
| 外部依賴 | 0 |
| 使用的瀏覽器 API | Canvas 2D、File API、Clipboard API、ResizeObserver、URL.createObjectURL |
| 支援格式輸入 | JPG / PNG / WebP / GIF |
| 輸出格式 | PNG |
| 測試環境 | Chrome / Safari / Firefox，HTTPS 和 file:// 皆可 |

---

*From sticker, to sticker.*
