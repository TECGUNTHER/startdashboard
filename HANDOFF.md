# Quick-Start Prompt

> Read this entire document before responding. It contains everything you need
> to know about a project I'm working on. After reading, summarize back to me
> in three sentences: (1) what this project is, (2) what state it's in,
> (3) what the immediate next step would be. Do not write code or take action
> until I confirm your summary is correct.

---

# About Tom (Your User)

- Senior Director of Pre-Sales Solutions Engineering (collaboration platforms). Not a professional developer; technically fluent but can't read raw stack traces or jargon without translation.
- Plain English only. Hates marketing language, filler, over-explanation. Prefers "added a Q4 toggle" over "implemented quarterly filtering."
- Default browser is Safari (Mac). Phone browsing is mostly inbound link-taps, not outbound surfing.
- When testing, he logs UX issues as Open Items and batch-fixes at the end — don't fix each one mid-test.
- For major changes he expects the real multi-agent flow: a Build Agent for code, a Documentation Agent for docs.

---

# The Project

**Name:** StartDashboard
**Folder:** `projects/personal-dashboard/` in Tom's `tom-workspace`
**Current status:** Complete (in active use; iterating on polish)
**Current phase:** Complete (v1.17 shipped)
**Last updated:** 2026-05-29
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that's Tom's home page across every browser/device. A top tab bar switches between "personas" (Work, Personal, Investments, etc.); each is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent. All data lives in one private GitHub gist, so the same URL shows the same synced view everywhere within seconds. Dark/modern by default; a light-mode toggle (v1.6) covers daytime use.

**Layout (v1.15):** the widget row and the categories pack vertically like Pinterest tiles (CSS multi-column "masonry") — each card is only as tall as its content. **Rearranging cards (v1.16):** widget and category cards drag with a custom pointer-drag — a faded clone follows the cursor and a highlighted "landing slot" shows where the card drops; works on touch (iPhone); each card also has hover-revealed ▲/▼ move arrows.

**Backgrounds:** 10 CSS-only dark presets (Midnight Mesh, Aurora, Synthwave, Carbon Grid, Deep Space, Liquid Glass, Tokyo Night, Terminal, Monochrome, **Starlink** as of v1.14), or any image URL. Starlink auto-applies a matched electric-blue accent (overridable).

**Widgets** (per-persona; show/hide, drag-to-reorder, optional glow color as of v1.10): Clock (multi-zone, 12/24hr), Weather (Open-Meteo, no key — each day shows icon + label, high/low, "🌧 NN%" as of v1.12), To-do, Sticky notes, Crypto (CoinGecko), Markets (Finnhub, fixed SPY/QQQ/DIA), Watchlists (Finnhub, multiple/renamable, v1.10).

**Also an "Ask Gemini" AI chat panel (v1.11)** — a *global* panel, not a widget. A bottom-right bubble opens a slide-over chat for concise answers, on the free Gemini API, with an "Open full Gemini" link-out. Auto-retries Google's transient "overloaded" 503s with a Retry button (v1.13). **As of v1.17 it has a 🎤 voice-input button** (free browser speech-to-text — speak your question into the box) **and renders Gemini's replies as formatted markdown** (bold, lists, code, clickable links).

---

# Where We Are

## Most recent session (2026-05-29 — v1.17)

Two upgrades to the Ask Gemini chat — `index.html` only, no new key/backend/library:

- **🎤 Voice input.** A `#chat-mic` button in `.chat-input-row` uses the browser-native **Web Speech API** (`SpeechRecognition`/`webkitSpeechRecognition`) for free speech-to-text. Feature-detected — hidden (`display:none`) if unsupported. The transcript is **appended** to the input (never wipes typed text), the box auto-grows, and it **never auto-sends**. Permission denial → friendly note. Needs HTTPS (GitHub Pages is fine; a local file isn't); works on iOS Safari.
- **Formatted replies via `renderMarkdownSafe(text)`** (~110 lines, no library). **Escape-first + tag whitelist**: escapes the whole input, then only inserts `strong, em, code, pre, ul, ol, li, h3, a, br, p`; links become real only if `^https?://`. Applied to model bubbles only via the existing `isHtml` hook; user/note/thinking stay plain text. **`state.chatHistory` stores raw text**, so re-renders re-parse cleanly.
- **Not verified live:** sandbox can't load the app (needs gist auth) or grant a mic; `node --check` passes, security traced. Tom should smoke-test.

## Sessions before that

- **v1.16 (2026-05-29):** Replaced the flaky native HTML5 drag for widget + category cards with one shared pointer-drag engine (`initCardDrag`) — clone, highlighted `.drop-placeholder` landing slot, dominant-axis before/after, touch-capable (iPhone) — plus hover-revealed ▲/▼ move arrows. Bookmarks/tabs/config-lists still on native drag.
- **v1.15 (2026-05-29):** Masonry layout — widget row + categories moved from equal-height grids to CSS multi-column. CSS-only.
- **v1.13–v1.14 (2026-05-29):** "Starlink" CSS background preset (10th, auto-applies a matched electric-blue accent); Ask Gemini chat auto-retries Google's transient 503 "overloaded" with a Retry button.
- **v1.0–v1.12 (2026-05-27 → 28):** Initial build through earlier polish — deploy, finance widgets, Finnhub migration, light mode, drag-to-reorder/multiple watchlists/per-card color (v1.10), the Ask Gemini panel (v1.11), richer weather (v1.12). Full history in PROJECT-HUB.

## Immediate next step

1. **Push v1.17** (single-file `index.html` commit), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Test voice:** open Ask Gemini, click 🎤, allow the mic, speak — confirm the words appear (and typed text isn't wiped), then Send. (Needs the live HTTPS URL, not a local file.)
3. **Confirm formatting:** ask for a list / code snippet / link and check it renders, not raw markdown. Check **light + dark**.
4. **Check iPhone**, and confirm the 🎤 button is hidden on any browser without speech support.
5. **Possible future builds (not started):** read-aloud / TTS for replies (deferred this round), a backup/export of dashboard data.

---

# Key Decisions Made (and Why)

- **Voice input via the free browser Web Speech API; markdown replies via a safe in-house renderer (v1.17)** — `SpeechRecognition`/`webkitSpeechRecognition` gives free speech-to-text with no key/backend/library, hidden where unsupported (graceful, not broken); voice never auto-sends and never overwrites typed text. Replies use a ~110-line **escape-first, whitelist-only** `renderMarkdownSafe` instead of a markdown library — keeps the zero-dependency design and is auditably safe against HTML/script injection from model output (links limited to `https?://`). Read-aloud/TTS deliberately deferred.
- **Custom vanilla pointer-drag for cards, replacing native HTML5 DnD (v1.16)** — native drag was unreliable and, after v1.15's masonry, computed the wrong drop axis. One shared `initCardDrag` engine fixes reliability + masonry correctness (dominant-axis before/after), adds touch support for free (iPhone), pulls in no library, shows a real landing-slot highlight. Bookmarks/tabs/config-lists kept on native drag.
- **Masonry via CSS multi-column, not experimental `grid-template-rows:masonry` (v1.15)** — fixes "tall card stretches neighbors / pushes categories down"; multi-column is well-supported today. Drag-reorder reads per-card geometry, so it's layout-agnostic.
- **GitHub gist as the data backend; no backend, browser-direct, single file** — durable, free, zero-dependency; chosen over Supabase/jsonbin/iCloud. Per-browser PATs in localStorage avoid OAuth in a static page.
- **Per-persona everything** (widgets, backgrounds, accent, order, colors). **Per-device, not synced:** default persona, theme, credentials, Gemini chat history — syncing keys/transcripts through a gist would be wrong.
- **Starlink background auto-applies a matched accent but doesn't lock it (v1.14)** — an optional `accent` field on the preset; overridable. CSS-only, no light variant.
- **Finnhub over Twelve Data (v1.9)** — 60 calls/min vs 8; Tom's ~12 symbols hit chronic 429s. Both free, same REST shape.
- **Ask Gemini = Gemini Flash, browser-direct, no backend (v1.11)** — free, fits "fast quick answer." Model is a Settings override (default `gemini-2.5-flash`). Native `:generateContent` confirmed not CORS-blocked; a note + "Open full Gemini" link-out covers failures. **Auto-retries the transient 503 "overloaded" (v1.13)**; a 429 is never retried.
- **Multiple watchlists via array + composite `watchlist:<id>` keys (v1.10)** — each uses the same reorder/hide/color/config machinery as a built-in widget. **Markets stays a singleton index-proxy card (SPY/QQQ/DIA).**
- **Light mode = one light surface + tokenized CSS (v1.6); bookmarklet over an extension; CSS-only backgrounds + image URL, no upload** (bloats the gist). **Empty Settings credential field = "no change" (v1.8)** — saving is never destructive; explicit clear is Danger Zone → Reset browser. **Last-write-wins on sync. iPhone slim mode deferred indefinitely.**

Minor decisions live in `docs/TECHNICAL.html`.

---

# The Stack

- **Languages:** HTML, CSS, JavaScript (vanilla — no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs (all browser-direct, no backend):** GitHub REST (gist read/write); Open-Meteo (weather + geocoding, no key); CoinGecko (crypto, no key); Finnhub (stock quotes + search, free key, 60/min — replaced Twelve Data in v1.9); Google Gemini (AI chat, free key — `:generateContent`, default `gemini-2.5-flash`, auto-retries 503s as of v1.13); Google Favicon (bookmark icons, best-effort).
- **Browser APIs:** Web Speech API (`SpeechRecognition`/`webkitSpeechRecognition`) for the chat 🎤 voice input (v1.17, feature-detected, HTTPS-only).
- **Storage:** One private GitHub gist (account `TECGUNTHER`) holds all persona data. Per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub key, Gemini key/model/history, theme, and a shared 5-min quote cache. (A legacy Twelve Data key is read but unused.)
- **Hosting:** GitHub Pages (free static), public repo `github.com/TECGUNTHER/startdashboard`.

**Layout + drag:** `.widgets-row` / `.categories-grid` use CSS multi-column masonry; widget + category cards drag via `initCardDrag` (clone + `.drop-placeholder` landing highlight + dominant-axis + anchor-key splice; touch-capable). **Chat (v1.17):** `#chat-mic` drives the Web Speech API; `renderMarkdownSafe` renders model replies (escape-first whitelist). **Data model (v1.10):** each persona has `categories` (optional `color`), `widgets`, `watchlists` (`[{id, title, symbols[]}]`), `widgetOrder` (incl. composite `watchlist:<id>` keys), `widgetColors`, `hiddenWidgets`. `BG_PRESETS` = 10, each `{ id, name }` + optional `accent` (only Starlink). Gemini chat is global, history (raw text) outside the gist. Full schema in `docs/TECHNICAL.html`.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (~5,560 lines)
├── README.md / INSTALL-FOR-OTHERS.md  — Tom's setup / fork-and-run guides
├── PROJECT-HUB.html         — project dashboard
├── HANDOFF.md               — this file
├── docs/TECHNICAL.html      — architecture, decisions, how-to-modify
├── docs/USER-EXPERIENCE.html— features, screens, workflows
└── plans/                   — approved build plans
```

---

# How to Run It

## Port: none (static page served by GitHub Pages)

## Key URLs
- Live: https://tecgunther.github.io/startdashboard/ · Repo: https://github.com/TECGUNTHER/startdashboard
- GitHub tokens: github.com/settings/tokens · Finnhub key: finnhub.io/register · Gemini key (free, no card): aistudio.google.com/app/apikey
- Gist ID: dashboard → Settings (⚙) → Sync → Gist ID

## One-time setup (already done)
Deployed and set up. To repeat on a fresh machine: push to the repo, enable Pages (main / root), add a PAT with `gist` scope, then free Finnhub + Gemini keys. Full walkthrough in `README.md`.

## To launch
```
open https://tecgunther.github.io/startdashboard/
```
Local-only preview (no sync): double-click `index.html`. Note: the chat 🎤 voice button needs HTTPS, so it won't work from a local file — use the live URL.

## Iteration loop (when changing code)
```
git add index.html && git commit -m "vX.Y: ..." && git push
```
Then hard-reload (Safari Opt+Cmd+R, Chrome Cmd+Shift+R). Pages takes ~30–60s.

## Troubleshooting (known issues)
- **"Bad credentials" on first run** → PAT wrong or missing `gist` scope; regenerate.
- **Sync indicator red ⚠** → Click to reload; if still failing the token expired — reset this browser in Settings → Danger Zone.
- **Cross-browser changes not showing** → Reload the other browser (poll is 60s; manual reload is instant).
- **Markets/Watchlists say "Add your Finnhub API key"** → Add a free key from finnhub.io/register; the old Twelve Data key is ignored.
- **"Rate limit reached"** → Nothing cached + rate-limited; the widget backs off and auto-refreshes after 60s. No action.
- **Ask Gemini says "Add your free Gemini API key"** → Add one from aistudio.google.com/app/apikey; the link-out works without a key.
- **Chat says "Google's servers are briefly overloaded" + Retry** → A Google-side 503 blip; already auto-retried twice. Click Retry or ask again shortly.
- **Chat 🎤 button missing** → That browser doesn't support speech-to-text (button is hidden by design) — just type. If it's there but doesn't work, you're likely on a local file (needs HTTPS) or denied the mic permission.

(Full troubleshooting list lives in `PROJECT-HUB.html`.)

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's setup walkthrough · `INSTALL-FOR-OTHERS.md` — fork-and-run guide for anyone he shares with
- `PROJECT-HUB.html` — Tom's home base · `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture/decisions/how-to-modify · `docs/USER-EXPERIENCE.html` — features and workflows
- `plans/` — approved build plans (through v1.17), read-only history

---

# Open Items

- **(Feature)** Cache the Crypto widget's price fetches — it re-fetches CoinGecko on every full render, and v1.10's reorder triggers more renders. Add a shared cache like the stock widgets' 5-min one if rate-limiting appears. (added 2026-05-28)
- **(UX Fix, low priority)** iPhone slim mode — deferred indefinitely (Tom's phone is inbound-link-driven). (added 2026-05-27)
- **(Question)** Mobile Safari drag-drop behavior on iPhone — only matters if Tom starts managing bookmarks on phone. (added 2026-05-27)
- **(Feature)** Custom embed widget for pasting iframes (calendar, RSS, etc.) — only when Tom has a specific use case. (added 2026-05-27)

---

# Working with Tom (Behavioral Notes)

## Communication norms
- Plain English, no jargon. One question at a time. Lead with the answer. No filler. Be honest about limits.

## Project-specific quirks
- Single-file HTML by design — don't suggest splitting into modules without a real reason. The *app* is dark/modern (v1.6 light option); the workspace *docs* are light/restrained.
- Tom is the only user — no multi-user/sharing. Don't add third-party services without checking (Gemini is the one beyond data APIs). Per-persona by default; per-device globals are theme, credentials, Gemini chat + history.
- Markets is a single fixed index-proxy card — don't make it renamable/multiple; Watchlists are the multiple concept.
- Layout is CSS multi-column masonry (v1.15) — keep natural card heights; don't reintroduce equal-height grids.
- Widget + category card drag is the custom pointer engine `initCardDrag` (v1.16) — touch-capable, `.drop-placeholder` landing highlight, ▲/▼ arrows. Bookmarks/tabs/config-lists still use native HTML5 drag; don't assume they share the engine, and don't remove `.drop-before`/`.drop-after` (bookmarks use them). `moveWidget` is dead code, left with a note.
- Backgrounds are CSS-only presets (10) or an image URL — no uploads; a preset may carry an optional `accent` (only Starlink) that auto-applies but stays overridable.
- Ask Gemini is a global panel, not a widget; a fresh assistant with no link to Tom's real Gemini history. Auto-retries 503; a 429 is not retried. **Voice input (v1.17)** is the free Web Speech API (`#chat-mic`), hidden where unsupported, never auto-sends. **Replies render via `renderMarkdownSafe`** — escape-first + tag whitelist; to add a tag you must add it in *both* the transform and the whitelist, and keep links restricted to `https?://`. History stores raw text.

## What he's likely to want next
- More polish from real use. Near-term: verify v1.17; possibly read-aloud/TTS for replies, the Crypto cache, a data backup/export, or extending the pointer-drag engine to bookmarks/tabs/config lists. Eventually a Custom embed widget or more financial widgets.

---

# Deeper Reference

For more than this doc provides: `docs/TECHNICAL.html` (architecture, decisions), `docs/USER-EXPERIENCE.html` (screens, workflows), `PROJECT-HUB.html` (full state + session history), `plans/` (build plans).
