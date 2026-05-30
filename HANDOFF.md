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
**Current phase:** Complete (v1.20 shipped)
**Last updated:** 2026-05-30
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that's Tom's home page across every browser/device. A top tab bar switches between "personas" (Work, Personal, Investments, etc.); each is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent. All data lives in one private GitHub gist, so the same URL shows the same synced view everywhere within seconds. Dark/modern by default; a light-mode toggle (v1.6) covers daytime use.

**Layout (v1.15):** the widget row and the categories pack vertically like Pinterest tiles (CSS multi-column "masonry") — each card is only as tall as its content.

**Rearranging cards (v1.18 — now reliable):** the precise tools are the always-visible ▲/▼ arrows on each card, "Move to front" / "Move to end" in the card's menu, and a "Reset layout" button in Settings. You can also **drag** a card by its grip — the drag no longer blinks (a glowing edge bar shows the drop spot, no layout reflow while moving) — but in masonry a dropped card lands **approximately**, not pixel-perfect. Arrows/Reset = precise; drag = quick-and-rough.

**Backgrounds:** 10 CSS-only dark presets (incl. **Starlink** as of v1.14, which auto-applies a matched electric-blue accent, overridable) or any image URL.

**Widgets** (per-persona; show/hide, reorder, optional glow color as of v1.10): Clock, Weather (Open-Meteo, no key — icon + label, high/low, "🌧 NN%" as of v1.12), To-do, Sticky notes, Crypto (CoinGecko), Markets (Finnhub, fixed SPY/QQQ/DIA), Watchlists (Finnhub, multiple/renamable, v1.10).

**Also an "Ask Gemini" AI chat panel (v1.11)** — a *global* panel, not a widget. A bottom-right bubble opens a slide-over chat for concise answers (free Gemini API), with an "Open full Gemini" link-out. Auto-retries Google's transient 503s with a Retry button (v1.13). Has a 🎤 voice-input button and renders replies as formatted markdown (v1.17).

---

# Where We Are

## Most recent session (2026-05-30 — v1.20)

Added an in-app **Help link to the User Experience guide** (full Design→Build→Document flow, as a feature). `index.html` + `docs/USER-EXPERIENCE.html` only, no new dependencies.

- **Two entry points, same target.** A **`?` icon in the top-bar header** (between the ☀ theme toggle and the ⚙ gear, `#open-help`) and a **"How to use StartDashboard" link in the Settings modal** (just above Close/Save) both open `docs/USER-EXPERIENCE.html` in a new tab.
- **Reuses the existing, already-deployed guide** — no new doc written. The guide is tracked in the repo, so GitHub Pages already serves it at `…/startdashboard/docs/USER-EXPERIENCE.html`.
- **Relative href** (`docs/USER-EXPERIENCE.html`), so it resolves correctly both on Pages and from a local file. The top-bar control is an `<a>` (not a button) so cmd/middle-click open-in-new-tab works; it carries `class="icon-btn"` and matches the other header icons (anchors already inherit color + no-underline).
- Bumped the guide's "last updated" stamp to **v1.20**. JS parses cleanly (`node --check` on the extracted script).

## Earlier sessions

### (2026-05-30 — v1.19)

Fixed a real bug from v1.18: **clicking the move arrows (▲/▼) on a category card made the whole card disappear**, and "Reset layout" wouldn't bring it back — only a full browser refresh would. `index.html` only, no new dependencies.

- **Root cause:** the shared `moveCardInOrder(orderArray, key, direction, keyOf)` helper removed the card with `splice(i,1)` but re-inserted **`key`** instead of the removed element. For widgets the element *is* the key string, so it was harmless; for categories the element is an object, so the category was overwritten by a bare id string → the card had no title/bookmarks → it vanished. "Reset layout" only rebuilds `widgetOrder`, never `categories`, so it couldn't repair it; a refresh reloaded the still-intact data from the gist. **No gist data was ever corrupted** (the bad value lived only in the in-memory state until reload).
- **Fix:** capture the removed element — `const [item] = orderArray.splice(i, 1); orderArray.splice(target, 0, item);` — correct for both widgets (string) and categories (object). Also fixes the category "Move to front/end" menu, which uses the same helper. JS parses cleanly (`node --check` on the extracted script). Pushed; Tom to smoke-test live.

### (2026-05-29 — v1.18)

Made rearranging cards reliable. `index.html` only, no new dependencies:

- **De-flickered the drag.** Root cause of the old blinking: the v1.16 drag inserted a real placeholder card into the flow on every mouse move, forcing the CSS masonry to **re-pack on every move** — that reflow was the flicker. The new `initCardDrag` drops the in-flow placeholder. On `pointermove`, `findNearest(x,y)` returns the nearest card + before/after side, and toggles `.drop-target-before`/`.drop-target-after` on it — a **box-shadow edge bar, no layout change** (no reflow → no blink). On `pointerup` the index is `pos + (after?1:0)` among the real cards, handed to the existing `onDrop`. Escape/`pointercancel` cancel.
- **Arrows promoted to primary.** `.move-controls` default opacity `0`→`0.45` (+ `:focus-within`), so ▲/▼ are always visible on every card, brightening on hover.
- **Move to front/end.** Widget config modal's "Layout" row and the category "..." menu both call `moveCardInOrder(orderArray, key, 'front'|'end'[, keyOf])` + `setState`.
- **Reset layout.** New `defaultWidgetOrder(p)` = `[...BUILTIN_WIDGETS, ...watchlistKeys]` (order only; doesn't touch `hiddenWidgets`); `ensurePersonaSchema` uses it. A Settings "Reset layout" button resets `p.widgetOrder` + toasts.
- **Key tradeoff:** masonry packs by height, so drag is **intentionally approximate**; arrows + Reset are precise. Not verified live (sandbox can't load the app); `node --check` passes. Tom should smoke-test.

## Sessions before that

- **v1.17 (2026-05-29):** Ask Gemini chat got a 🎤 voice-input button (free Web Speech API, `#chat-mic`, hidden where unsupported, never auto-sends) and renders replies as formatted markdown via a safe escape-first, whitelist-only `renderMarkdownSafe`.
- **v1.16 (2026-05-29):** First custom pointer-drag engine for widget + category cards (replaced flaky native HTML5 drag, added touch). v1.18 de-flickered it.
- **v1.15 (2026-05-29):** Masonry layout — widget row + categories moved from equal-height grids to CSS multi-column. CSS-only.
- **v1.0–v1.14 (2026-05-27 → 29):** Initial build through polish — deploy, finance widgets, Finnhub migration, light mode, drag-to-reorder/multiple watchlists/per-card color (v1.10), the Ask Gemini panel (v1.11), richer weather (v1.12), 503 auto-retry (v1.13), Starlink background (v1.14). Full history in PROJECT-HUB.

## Immediate next step

1. **Verify the v1.20 Help link (live).** Hard-reload Safari (Opt+Cmd+R). Click the **`?`** icon in the top bar (next to the ⚙ gear) → the User Experience guide should open in a new tab. Also check the **"How to use StartDashboard"** link inside Settings.
2. **Verify the v1.19 fix** (still untested by Tom if he hasn't yet): click the ▲/▼ arrows on a **category** card → it should move one slot and **stay visible**. Repeat with "Move to front" / "Move to end" in a category "..." menu.
3. **Re-confirm the v1.18 items:** widget ▲/▼ arrows, widget Move to front/end, drag (no blink, approximate landing), Settings → Reset layout.
4. **Possible future builds (not started):** read-aloud / TTS for chat replies; a backup/export of dashboard data.

---

# Key Decisions Made (and Why)

- **Masonry and live drag are fundamentally at odds → arrows + Reset are the reliable mechanism (v1.18).** Because CSS multi-column packs cards by height, a dragged card can only ever land *approximately* where you aim. So the always-visible ▲/▼ arrows, Move to front/end, and Reset layout are the precise, dependable controls; the drag is de-flickered but rough. The blink was caused by an in-flow placeholder re-packing the masonry on every move — replaced with a **no-reflow overlay edge bar** (`.drop-target-before`/`.drop-target-after`).
- **Voice input via the free Web Speech API; markdown replies via a safe in-house renderer (v1.17)** — free speech-to-text, no key/backend/library, hidden where unsupported. Replies use an escape-first, whitelist-only `renderMarkdownSafe` (links limited to `https?://`) — keeps the zero-dependency design and is auditably injection-safe. Read-aloud/TTS deferred.
- **Custom vanilla pointer-drag for cards, replacing native HTML5 DnD (v1.16)** — native drag was unreliable and computed the wrong drop axis after masonry. One shared `initCardDrag` engine, touch-capable. Bookmarks/tabs/config-lists kept on native drag.
- **Masonry via CSS multi-column, not experimental `grid-template-rows:masonry` (v1.15)** — multi-column is well-supported today; fixes "tall card stretches neighbors."
- **GitHub gist as the data backend; no backend, browser-direct, single file** — durable, free, zero-dependency; over Supabase/jsonbin/iCloud. Per-browser PATs in localStorage avoid OAuth in a static page.
- **Per-persona everything** (widgets, backgrounds, accent, order, colors). **Per-device, not synced:** default persona, theme, credentials, Gemini chat history.
- **Starlink background auto-applies a matched accent but doesn't lock it (v1.14)** — optional `accent` field on the preset; overridable.
- **Finnhub over Twelve Data (v1.9)** — 60 calls/min vs 8; Tom's ~12 symbols hit chronic 429s.
- **Ask Gemini = Gemini Flash, browser-direct, no backend (v1.11)** — free, fits "fast quick answer." Model is a Settings override (default `gemini-2.5-flash`); native `:generateContent` not CORS-blocked. **Auto-retries the transient 503 (v1.13)**; a 429 is never retried.
- **Multiple watchlists via array + composite `watchlist:<id>` keys (v1.10); Markets stays a singleton index-proxy card (SPY/QQQ/DIA).**
- **Light mode = one light surface + tokenized CSS (v1.6); bookmarklet over an extension; CSS-only backgrounds + image URL, no upload. Empty credential field = "no change" (v1.8). Last-write-wins on sync. iPhone slim mode deferred indefinitely.**

Minor decisions live in `docs/TECHNICAL.html`.

---

# The Stack

- **Languages:** HTML, CSS, JavaScript (vanilla — no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs (all browser-direct, no backend):** GitHub REST (gist); Open-Meteo (weather, no key); CoinGecko (crypto, no key); Finnhub (stocks, free key, 60/min); Google Gemini (chat, free key — `:generateContent`, default `gemini-2.5-flash`, auto-retries 503s); Google Favicon (icons).
- **Browser APIs:** Web Speech API (`SpeechRecognition`/`webkitSpeechRecognition`) for the chat 🎤 voice input (v1.17, feature-detected, HTTPS-only).
- **Storage:** One private GitHub gist (account `TECGUNTHER`) holds all persona data. Per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub/Gemini keys + model + chat history, theme, and a 5-min quote cache.
- **Hosting:** GitHub Pages (free static), public repo `github.com/TECGUNTHER/startdashboard`.

**Layout + drag:** `.widgets-row` / `.categories-grid` use CSS multi-column masonry. Widget + category cards drag via `initCardDrag` (v1.18: no in-flow placeholder — `findNearest` + `.drop-target-before`/`.drop-target-after` overlay edge bar, no reflow; `onUp` index = `pos + (after?1:0)`; touch-capable). Always-visible ▲/▼ via `.move-controls`; Move-to-front/end + Reset via `moveCardInOrder` and `defaultWidgetOrder(p)`. **Chat (v1.17):** `#chat-mic` drives the Web Speech API; `renderMarkdownSafe` renders model replies (escape-first whitelist). **Data model (v1.10):** each persona has `categories` (optional `color`), `widgets`, `watchlists`, `widgetOrder` (incl. composite `watchlist:<id>` keys), `widgetColors`, `hiddenWidgets`. `BG_PRESETS` = 10, each `{ id, name }` + optional `accent` (only Starlink). Full schema in `docs/TECHNICAL.html`.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (~5,600 lines)
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
- **Chat 🎤 button missing** → That browser doesn't support speech-to-text (hidden by design) — just type. If present but it doesn't work, you're likely on a local file (needs HTTPS) or denied the mic permission.
- **Dragged card didn't land exactly where I aimed** → Expected — masonry packs by height, so drag is approximate. Use the ▲/▼ arrows, Move to front/end, or Settings → Reset layout for precise control.

(Full troubleshooting list lives in `PROJECT-HUB.html`.)

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's setup walkthrough · `INSTALL-FOR-OTHERS.md` — fork-and-run guide for anyone he shares with
- `PROJECT-HUB.html` — Tom's home base · `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture/decisions/how-to-modify · `docs/USER-EXPERIENCE.html` — features and workflows
- `plans/` — approved build plans (through v1.18), read-only history

---

# Open Items

- **(Feature)** Cache the Crypto widget's price fetches — it re-fetches CoinGecko on every full render, and reorder triggers more renders. Add a shared cache like the stock widgets' 5-min one if rate-limiting appears. (added 2026-05-28)
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
- **Rearranging (v1.18):** widget + category card drag is `initCardDrag`, now **de-flickered** via a no-reflow overlay edge bar (`.drop-target-before`/`.drop-target-after`), **not** an in-flow placeholder (re-introducing one brings back the masonry-repack flicker). Drag is **intentionally approximate** in masonry; the always-visible ▲/▼ arrows, Move to front/end, and Settings → Reset layout (`defaultWidgetOrder`) are the precise tools. **`moveCardInOrder` must re-insert the *removed element* (`const [item]=splice(i,1); splice(target,0,item)`), not the `key` — re-inserting the key destroys category objects (the v1.19 bug). Reset layout only rebuilds `widgetOrder`, not `categories`.** Bookmarks/tabs/config-lists still use native HTML5 drag and `.drop-before`/`.drop-after` — don't remove those. Two harmless `.drop-placeholder` *comment* refs remain.
- Backgrounds are CSS-only presets (10) or an image URL — no uploads; a preset may carry an optional `accent` (only Starlink) that auto-applies but stays overridable.
- Ask Gemini is a global panel, not a widget; a fresh assistant with no link to Tom's real Gemini history. Auto-retries 503, not 429. Voice input is the free Web Speech API (`#chat-mic`), hidden where unsupported, never auto-sends. Replies render via `renderMarkdownSafe` — escape-first + tag whitelist; to add a tag, add it in *both* the transform and the whitelist, keep links restricted to `https?://`.

## What he's likely to want next
- More polish from real use. Near-term: verify v1.18; possibly read-aloud/TTS for replies, the Crypto cache, a data backup/export, or extending the pointer-drag engine to bookmarks/tabs/config lists. Eventually a Custom embed widget or more financial widgets.

---

# Deeper Reference

For more than this doc provides: `docs/TECHNICAL.html` (architecture, decisions), `docs/USER-EXPERIENCE.html` (screens, workflows), `PROJECT-HUB.html` (full state + session history), `plans/` (build plans).
