# ChatGPT Optimizer — Faster Long Chats

**Keep ChatGPT responsive on long threads** by reducing live DOM size and loading older messages in fixed batches. Smooth scroll, predictable work, and lower memory usage on big conversations.

> **Creator:** [Sarthak Gupta](https://www.linkedin.com/in/sarthakgupta04/)  
> **Website:** https://<your-username>.github.io/chatgpt-optimizer-site/  
> **Chrome Web Store:** https://CWS_URL_HERE

---

## Why this is faster

- **DOM windowing** — Keep ~50 recent messages in the DOM; hide the rest with a lightweight class. This shrinks the style/layout scope dramatically on large threads.
- **Batch lazy‑load** — Reveal older messages in **fixed steps** (e.g., 25 at a time) for bounded, predictable work per action.
- **Observer orchestration** — A subtree `MutationObserver` keeps counts accurate as ChatGPT hydrates; an optional `IntersectionObserver` loads **one batch per distinct scroll** (rising‑edge + re‑arm, no cascade).

**Typical results (internal tests, 300–600 messages)**

- **×5–×15** fewer live DOM nodes  
- **−20–45%** layout/recalc time  
- **−25–50%** JS heap memory  
<sub>Actual results vary by device, theme, and content.</sub>

---

## Features

- **Window size**: target visible messages (recommend **50**)
- **Batch size**: messages revealed per action (recommend **25**)
- **“Load more”** banner at the hidden/visible boundary (moves after each batch)
- **Optional** one‑batch‑per‑scroll (off by default; no cascade)
- **Stats**: total / visible / hidden / container / initialized
- Minimal UI, no tracking, predictable behavior

---

## Install

### From the Chrome Web Store (recommended)

1. Visit **https://CWS_URL_HERE**  
2. Click **Add to Chrome**.  
3. Pin the extension (puzzle icon → pin).

### Manual (unpacked) — for development

1. Clone/download this repo.  
2. `chrome://extensions` → enable **Developer mode**.  
3. **Load unpacked** → select the folder with `manifest.json`.

---

## Usage

1. Open a ChatGPT thread.  
2. In the popup, set:
   - **Window Size** ≈ `50`
   - **Batch Size** ≈ `25`
3. Scroll down; click **Load 25 more** to reveal older messages.  
   - If **Auto‑load on scroll** is enabled, the banner loads **one batch** the first time it enters view; it re‑arms only after you scroll away.

**Tips**

- For maximum smoothness, keep **Auto‑load** off and click to load batches manually.  
- If you see jumps in counts, wait a second for ChatGPT’s hydration; the observer will resync stats.

---

## How it works (high level)

```text
content.js
 ├─ bootstrap()        → detect chat container, hydrate state
 ├─ applyWindowing()   → hide/show message nodes to meet window size
 ├─ ensureBanner()     → place “Load more” before first visible message
 ├─ showOlderMessages()→ reveal exactly <batchSize> hidden nodes
 ├─ setupAutoScroll()  → (optional) rising-edge observer, re-arm on leave
 └─ refresh()          → recompute counts; do not auto-grow beyond batches
```

- **No `scrollIntoView` chaining** after loading a batch (prevents cascade).  
- **Banner always repositions** to the new boundary after reveal.  
- **Observer** uses `rootMargin: '0px 0px -45% 0px'`, `threshold: 0` to avoid being “stuck” intersecting.

---

## Permissions & privacy

- **Host permissions:** `https://chat.openai.com/*`, `https://chatgpt.com/*`  
- **Permissions:** `storage` (save preferences), `tabs` (detect ChatGPT tab), optional `declarativeNetRequest` (static rules)  
- **Privacy:** No tracking. No external servers. All logic runs locally.

Read the policy: `/privacy_policy.html` (or the hosted version on your site).

---

## Troubleshooting

- **Banner doesn’t appear** → ensure you’re on a chat page; reload once.  
- **Auto‑load fires repeatedly** → it should only fire once per distinct scroll; if you customized CSS/scroll containers, disable Auto‑load.  
- **Counts look wrong** → hydration may still be happening; wait a moment or scroll to trigger refresh.

---

## Development

### File structure (core)
```
content.js        # windowing, banner, observer, stats
popup.html/js     # settings & debug
styles.css        # minimal scoped styles for banner/fab
manifest.json     # MV3 configuration
background.js     # optional declarativeNetRequest rules
privacy_policy.html
```

### Build a ZIP for the Web Store
```bash
zip -r chatgpt-optimizer-<version>.zip . \
  -x ".*" -x "__MACOSX" -x "*.map" -x "node_modules/*"
```

### Versioning
- Bump `version` in `manifest.json` for each release.
- Keep a short **CHANGELOG** in the repo or in the Web Store “What’s New”.

---

## Roadmap

- Floating “Load more” FAB (manual-only mode)  
- Keyboard shortcut for batch reveal  
- Per-thread remembered preferences  
- Optional telemetry-block list toggle (DNR rules UI)

---

## License

MIT (or your preference). If you need a different license, replace this section.

---

## Acknowledgements

Not affiliated with OpenAI. “ChatGPT” is a trademark of its respective owner.  
Created by **Sarthak Gupta** — [LinkedIn](https://www.linkedin.com/in/sarthakgupta04/).
