# StickerBoard

> **From sticker, to sticker.**

Turn AI-generated images into print-ready sticker sheets — no design software, no install, no build step.

[![License: MIT](https://img.shields.io/badge/License-MIT-5655F5.svg)](LICENSE)
[![Single file](https://img.shields.io/badge/single%20file-HTML-10B981.svg)]()
[![Deploy to Netlify](https://img.shields.io/badge/deploy-Netlify-00C7B7.svg)](https://app.netlify.com/drop)

---

## What it does

Upload your AI-generated images, pick a print format, and download a PNG ready to send to a print shop — or your local 7-11 ibon.

**Layout**
- Two modes: *fixed sticker size* (auto-calculates how many fit) or *fixed grid* (auto-calculates sticker size)
- Multi-image upload — images cycle to fill all slots automatically
- Reorder images with ↑↓ buttons

**Style**
- Corner radius slider: square → rounded → circle
- Sticker padding (safe margin from cut line): 0 / 2 / 3 / 5 mm
- Background color per cell (white, transparent, or custom)
- Fill mode: Cover (crop to fill) or Contain (show whole image)

**Print quality**
- Image quality indicator 🟢🟡🔴 based on print resolution
- Recommended source image size shown dynamically
- HEIC format detection with conversion instructions

**Output**
- Canvas presets: 7-11 ibon 4×6", A4, or custom (mm)
- Cut marks: dashed border or corner marks
- Downloads at full print resolution (300 DPI equivalent)

---

## Usage

### Quickest path

1. Open `index.html` in any modern browser
2. Upload your AI-generated images (JPG / PNG / WebP / GIF)
3. Choose a print format and adjust the layout
4. Hit **Download PNG**

### For 7-11 ibon (Taiwan)

1. Select preset **7-11 ibon 4×6"**
2. Set sticker size to 3×3 cm (fits 15 per sheet)
3. Download PNG → open ibon App → 相片列印 → 貼紙相紙 → 4×6 英吋
4. Cut along the dashed lines after printing ✂️

---

## Deploy

**GitHub Pages** — fork this repo, go to Settings → Pages → deploy from `main`. Done.

**Netlify Drop** — drag `index.html` to [app.netlify.com/drop](https://app.netlify.com/drop). 30 seconds.

**Locally** — double-click `index.html`. No server needed, works offline.

---

## Origin story

This project started from a very specific question:

> *"Can Claude's `/sticker` command send stickers to Taiwan?"*

The answer was no. But that conversation about workarounds became a Claude-generated sticker designer, which became a layout tool, which became a full print-ready generator — spec'd across three AI conversations over two days and shipped as a single 537-line HTML file.

**From sticker, to sticker.**

---

## About

Created by **Doe**.

Product design and feature spec were collaboratively discussed with Claude (Anthropic), ChatGPT (OpenAI), and Gemini (Google) — each bringing a different angle to the same problem. The final implementation lives in one `index.html`.

---

## License

[MIT](LICENSE) © 2026 Doe
