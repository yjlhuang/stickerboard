# StickerBoard Architecture Notes

## 專案定位

StickerBoard 是：

> AI 生圖 → 貼紙排版 → 印刷輸出

工具。

核心目標：

讓一般使用者不懂排版也能快速做出可列印貼紙。

---

# Product Lines

目前分成兩條產品線：

## Product A

StickerBoard

用途：

* 實體貼紙
* 手帳
* 筆電貼紙
* ibon 列印

目前已上線。

---

## Product B（Future）

LINE Helper

用途：

* LINE Creators Market
* 貼圖包整理
* 尺寸轉換
* ZIP 打包

未開發。

獨立 HTML。

不與 StickerBoard UI 混合。

---

# 核心原則

## Principle 1

不要把 StickerBoard 與 LINE Helper 合併。

原因：

需求完全不同。

---

StickerBoard 重視：

```text
創作自由
驚喜感
印刷品質
```

---

LINE Helper 重視：

```text
一致性
批量輸出
官方規格
```

---

兩者共用：

```text
Prompt Engine
```

即可。

---

# Prompt Engine

抽成：

```text
shared/
 ├─ recipe-data.js
 └─ recipe.js
```

---

目的：

未來兩條產品線共用。

---

# Typography Sticker

新增於 StickerBoard。

定位：

不是模板機。

而是：

```text
一句話
↓
AI理解
↓
設計感貼紙
```

---

# Prompt Architecture

四層結構：

## Layer 1

Text

例如：

```text
還要繼續拚
```

---

## Layer 2

Emotion

例如：

```text
厭世
崩潰
熱血
治癒
文青
```

---

作用：

控制情緒。

不控制角色。

---

## Layer 3

Aesthetic

例如：

```text
粗體梗圖
漫畫標語
東方水墨
可愛貼紙
```

---

作用：

控制視覺風格。

不控制角色。

---

## Layer 4（Future）

Character Preference

選填：

```text
自動
動物
人物
怪獸
食物
物件
```

---

預設：

```text
自動
```

---

# Route System

Prompt Engine 必須支援兩種路由。

---

## Route A

圖文融合

適用：

* 單張貼紙
* 梗圖貼紙
* 文字主導貼紙

流程：

```text
文字直接進 Prompt
```

---

優點：

融合度最高。

---

建議模型：

```text
GPT Image
```

---

## Route B

圖字分離

適用：

* 文青風
* 長句子
* 成套貼圖

流程：

```text
Step1
生底圖

Step2
後加文字
```

---

優點：

字不容易翻車。

---

建議模型：

```text
Gemini
GPT
皆可
```

---

# Model Philosophy

不要把模型寫死。

不要：

```text
GPT模式
Gemini模式
```

---

要：

```text
Route A
Route B
```

---

模型只是執行者。

不是產品架構。

---

# Preset Philosophy

Preset 決定：

```text
流程
```

---

不是：

```text
情緒
```

---

錯誤：

```js
梗圖 = 厭世
```

---

正確：

```js
梗圖 = Route A
```

---

使用者自己選：

```text
厭世
熱血
崩潰
```

---

# Typography v1 範圍

包含：

✅ 主文字

✅ Emotion

✅ Aesthetic

✅ Route A/B

✅ Prompt Generator

---

不包含：

❌ LINE

❌ ZIP

❌ 去背

❌ IndexedDB

❌ 角色一致性

❌ 成套管理

---

# Typography v2

可能加入：

* Character Preference
* Pack Mode
* Series Generator

---

# Typography v3

可能加入：

* LINE Helper 整合
* 批量貼圖
* ZIP Export

---


> 請以「Typography Sticker 是 StickerBoard 的第二創作軸」為核心思考。重點不是新增模板，而是建立一套可重複使用的 Prompt Engine。Emotion 負責情緒、Aesthetic 負責風格、Route 負責生成策略，三者保持解耦。優先完成可擴充架構，而非增加更多預設角色或主題。

