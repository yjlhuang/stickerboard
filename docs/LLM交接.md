# 給下一個 LLM 的交接文件

> 假設你是全新開機、零上下文。讀完這份就能安全動工。
> 快照：2026-07-22 · v0.3.1 · commit `8242de5`

---

## 0. 開工必讀（依序，全部都在 repo 內）

**這個專案沒有 CLAUDE.md**，不要去找。實際順序是：

1. **本檔**（`docs/LLM交接.md`）——地雷區在第 4 節，先看
2. [`TECH.md`](../TECH.md)——架構決策與為什麼這樣做
3. [`stickerboard architecture notes.md`](../stickerboard%20architecture%20notes.md)——擴充藍圖（雙產品線、Prompt Engine、Route A/B）
4. [`CHANGELOG.md`](../CHANGELOG.md)——最近改了什麼
5. [`docs/進度與展望.md`](進度與展望.md)——待辦與構想

使用者另有跨專案知識庫（Obsidian vault）在
`G:\我的雲端硬碟\☆Obsidian\the_vault_of_doe\`，
其中 `ai/專案卡/stickerboard.md`（一頁摘要）和 `ai/lessons/stickerboard.md`（翻車史）跟本專案有關。
vault 內的 `me/` 和 `nook/` 是私人內容，**不要主動讀取或引用**。

---

## 1. 這是什麼

**一句話**：把 AI 生成的圖片排版成印刷用貼紙圖檔的單檔網頁工具。
目標使用者是「想把 AI 圖印成貼紙的人」，不是開發者。

**技術形態**：
- **單一 `index.html`**（約 978 行），HTML + CSS + JS 全部在裡面
- **零外部依賴、零 build step、零 node_modules**
- 部署 = **push 到 GitHub，GitHub Pages 自動發布**（不是 Vercel，沒有 pinned alias 問題）
- 狀態管理靠一個全域 `S` 物件 + 手動 DOM 操作，沒有框架

**線上**：https://yjlhuang.github.io/stickerboard/ ｜ **Repo**：https://github.com/yjlhuang/stickerboard（public）

---

## 2. 現狀快照

| 項目 | 值 |
|---|---|
| 最新 commit | `8242de5`（2026-07-22） |
| 線上版本 | v0.3.1（標題列右側徽章可見） |
| 分支 | `main`，與 origin 同步 |
| 進行中的開發 | 無 |
| 已知 bug | 無 |

**功能範圍**：排版引擎（固定貼紙尺寸 / 固定格數兩模式）、圓角與圓形貼紙、
多圖上傳與循環填格、列印品質指示、AI 生圖助手（圖像貼紙 7 風格 × 5 工具 ＋
文字貼紙 5 情緒 × 4 風格 × Route A/B）、PNG 全解析度輸出。

---

## 3. 程式碼地圖（`index.html` 內，行號會漂移，用關鍵字找）

| 區塊 | 找什麼 |
|---|---|
| 全域狀態 | `const S=` |
| 版面計算引擎 | `function compute()` — 輸入 `S`，輸出所有像素座標 |
| 雙畫布渲染 | `renderCanvas()` — 預覽走縮圖 scale，下載走 scale=1 全解析度 |
| 圓角裁切 | `rrPath()` — 手刻 `arcTo()`，不用 `ctx.roundRect()`（相容性） |
| 圖像貼紙 prompt | 風格與工具的關鍵詞組 |
| **文字貼紙 Prompt Engine** | `TY_DATA`（純資料）、`tyAssemble()`（純函數）、`TY_TAIL` |
| 文字貼紙 UI glue | `buildTy()` 及其事件綁定 |
| 三欄版面搬移 | `placeAI()` + `mqWide` |
| 歡迎彈窗 | `SEEN_KEY` / `welcomeSeen()` / `openModal()` |

---

## 4. ⚠️ 地雷區（這個專案特有，踩過的）

1. **不能用 ES module。** `file://` 下 `<script type="module">` 會被 CORS 擋掉，
   破壞「雙擊 index.html 就能跑」這個核心賣點。所有 JS 都是傳統 script。

2. **Clipboard API 在 `file://` 下會失效**（已修，別改回去）。
   `navigator.clipboard` 需要 secure context；備援的 `execCommand('copy')`
   **必須在點擊事件的同步路徑裡呼叫**——放進 Promise 的 `.catch()` 就脫離使用者手勢、一樣被擋。
   正確寫法是 `if (window.isSecureContext) {現代 API} else {fallback()}`，不是 `.catch(fallback)`。
   詳見 vault 的 `ai/lessons/stickerboard.md`。

3. **凡是需要使用者手勢或 secure context 的瀏覽器 API，都要包 try/catch 並想過 `file://`。**
   localStorage 也一樣（歡迎彈窗那段就是這樣寫的）。
   **只在 GitHub Pages 上測是不夠的，一定要另外用 `file://` 開一次。**

4. **改版面要同時想寬窄兩種。** AI 助手是 `placeAI()` **搬移同一個 DOM 節點**
   （≥1101px 進右欄、否則搬回左欄），不是複製兩份 HTML。
   所以：所有 `id` 唯一、事件綁定不用重綁；但你**不能**假設某個元素永遠在同一個父節點底下。

5. **標題列很擠。** header 是 `display:flex` 且 `.ver` / 按鈕都 `flex-shrink:0`。
   加任何東西進 header 前，先算 360px 手機放不放得下（曾經算出需要 363px，差點溢出）。
   目前解法是窄螢幕隱藏副標與版本號的日期部分。

6. **預覽要套 DPR 縮放**，否則 Retina 螢幕上會糊。改動 `renderCanvas` 時別把這段拿掉。

7. **`index.html` 是靠多輪字串取代長出來的**，曾經發生過 `<div>` 開頭被吃掉、
   DOM 樹錯亂、`getElementById` 拿到 null 的事故。**大範圍編輯後要驗證標籤配對與關鍵 id 存在。**

8. **版本號徽章是部署驗證機制**，不是裝飾。改動 UI 後記得 bump
   （`index.html` 的 `.ver` + CHANGELOG 要一致），這樣「線上是不是最新版」才能一眼裁決。
   純文件改動不用 bump。

---

## 5. Prompt Engine 的設計意圖（動它之前務必讀）

文字貼紙的架構刻意做成「**純資料 + 純函數**」，因為未來要抽成 `shared/recipe-data.js` +
`shared/recipe.js`，給第二條產品線（LINE Helper）共用。所以：

- **不要把 DOM 操作寫進 `tyAssemble()`**，它必須維持純函數（recipe + tool 進，字串出）
- **不要把模型寫死成「GPT 模式 / Gemini 模式」**。架構層只有 **Route A（圖文融合）**
  和 **Route B（圖字分離）**——模型只是執行者，不是產品架構
- **Preset 決定的是流程，不是情緒**。風格會帶入建議 route，但情緒和風格必須保持自由組合
- 某個「情緒|風格」組合出圖不好時，用 `TY_DATA.overrides['emotion|aesthetic']` 覆蓋，**不要改架構**

---

## 6. 使用者偏好與溝通方式

- **一律繁體中文**（使用者看得懂英文，但預設中文；不要混入簡體字）
- 使用者是高中英文老師，**不寫程式**。解釋要白話，術語出現時附一句人話
- **回報「完成」前要真的驗證**：看實際渲染結果，不是只看程式碼；
  並確認線上部署的就是最新版（這就是版本號徽章存在的原因）
- **視覺微調要主動說明差異**（例如「深色換深色、差異很小、長這樣」），
  否則會被當成「沒部署成功」
- 這個專案**允許**自起瀏覽器做線上驗證（有些別的專案禁止，不要弄混）
- Windows 環境：用 PowerShell 寫檔要注意編碼，明確指定 UTF-8
- 有 `/commit`（只 commit）和 `/deploy`（commit + push + 驗證線上）兩個 skill，收尾走它們

---

## 7. 沒做完的事與下一步建議

完整清單見 [`docs/進度與展望.md`](進度與展望.md)。摘要：

- 文字貼紙 v1 還沒經過大量實測，出圖不好的組合用 `overrides` 微調即可
- 付款區塊只有版位，沒串金流
- Typography Layer 4（角色偏好）、Pack Mode 等在藍圖裡，**尚未承諾**

**接手第一件該做的事**：不是寫程式，是打開 https://yjlhuang.github.io/stickerboard/
實際跑一次「文字貼紙 → 複製 prompt → 貼到 AI 生圖 → 上傳排版 → 下載」的完整流程。
這個專案的價值全在使用體驗上，光讀 code 看不出哪裡卡。

---

*相關文件：[小白版使用指南](使用指南.md)　·　[目前進度與未來展望](進度與展望.md)　·　[TECH.md](../TECH.md)*
