# StickerBoard

> **From sticker, to sticker.**

Turn AI-generated images into print-ready sticker sheets — no design software, no install, no build step.

[![License: MIT](https://img.shields.io/badge/License-MIT-5655F5.svg)](LICENSE)
[![Single file](https://img.shields.io/badge/single%20file-HTML-10B981.svg)]()
[![Live Demo](https://img.shields.io/badge/demo-GitHub%20Pages-5655F5.svg)](https://yjlhuang.github.io/stickerboard)

---

## What it does

Upload your AI-generated images, pick a print format, and download a PNG ready to send to a print shop — or your local 7-11 ibon.

**Layout**
- Two modes: *fixed sticker size* (auto-calculates how many fit) or *fixed grid* (auto-calculates sticker size)
- Multi-image upload — images cycle to fill all slots automatically
- Reorder images with ↑↓ buttons

**Style**
- Corner radius slider: square → rounded → circle (50% = perfect circle)
- Sticker padding (safe margin from cut line): 0 / 2 / 3 / 5 mm
- Background color per cell (white, transparent, or custom)
- Fill mode: Cover (crop to fill) or Contain (show whole image)

**AI 生圖助手 (Prompt Helper) — 圖像貼紙 tab**
- 7 style presets: cartoon, watercolor, Ghibli, minimal, realistic, pixel, stamp
- 5 tool templates: Universal / ChatGPT / Gemini / Midjourney / Flux·SD
- 📸 Photo reference mode: generates prompts that ask the AI to reference an uploaded photo (for avatar/pet stickers)
- Dynamic recommended image size based on your current sticker settings
- One-click copy with `isSecureContext`-aware clipboard fallback

**文字貼紙 (Typography Sticker v1) — 第二個 tab**
- Say it in one sentence, get a designed text sticker prompt — not a template filler
- Emotion (厭世 / 崩潰 / 熱血 / 治癒 / 文青) × Aesthetic (粗體梗圖 / 漫畫標語 / 東方水墨 / 可愛貼紙), fully decoupled
- Two generation routes: **Route A** (text baked into the image, one prompt) or **Route B** (background first, text added in a second step) — picking a style suggests a route, you can still override
- Per-tool notes, incl. Midjourney's CJK-text and follow-up-edit limitations

**Print quality**
- Image quality indicator 🟢🟡🔴 based on print resolution
- HEIC format detection with step-by-step conversion instructions

**Output**
- Canvas presets: 7-11 ibon 4×6", A4, or custom (mm)
- Cut marks: dashed border or corner marks
- Downloads at full print resolution (300 DPI equivalent)

---

## Usage

### Quickest path

1. Open `index.html` in any modern browser
2. Upload your AI-generated images (JPG · PNG · WebP · GIF)
3. Choose a print format and adjust the layout
4. Hit **Download PNG**

### For 7-11 ibon (Taiwan)

1. Select preset **7-11 ibon 4×6"**
2. Set sticker size to 3×3 cm → 15 stickers per sheet
3. Download PNG → open ibon App → 相片列印 → 貼紙相紙 → 4×6 英吋
4. Cut along the dashed lines after printing ✂️

### Circle stickers

Set corner radius to **50%** — square cells become perfect circles. Works especially well with AI-generated images that have transparent or white backgrounds.

---

## Deploy

**GitHub Pages** — fork this repo, go to Settings → Pages → deploy from `main`. Done.

**Netlify Drop** — drag `index.html` to [app.netlify.com/drop](https://app.netlify.com/drop). 30 seconds.

**Locally** — double-click `index.html`. No server needed, works offline.

---

## Origin story

This project started from a very specific question:

> *"Can Claude's `/sticker` command send stickers to Taiwan?"*

The answer was no. But that conversation about workarounds became a Claude-generated sticker designer, which became a layout tool, which became a full print-ready generator — spec'd across three AI conversations and iterated into a single HTML file (~970 lines today).

**From sticker, to sticker.**

---

## About

Created by **Doe**.

Product design and feature spec were collaboratively discussed with Claude (Anthropic), ChatGPT (OpenAI), and Gemini (Google) — each bringing a different angle to the same problem. The final implementation lives in one `index.html`.

See [TECH.md](TECH.md) for architecture notes and design decisions, and
[stickerboard architecture notes.md](stickerboard%20architecture%20notes.md) for the forward-looking blueprint (product lines, Prompt Engine, Route A/B).

---

## License

[MIT](LICENSE) © 2026 Doe
