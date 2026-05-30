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
**Current phase:** Complete (v1.16 shipped)
**Last updated:** 2026-05-29
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that's Tom's home page across every browser/device. A top tab bar switches between "personas" (Work, Personal, Investments, etc.); each is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent. All data lives in one private GitHub gist, so the same URL shows the same synced view everywhere within seconds. Dark/modern by default; a light-mode toggle (v1.6) covers daytime use.

**Layout (v1.15):** the widget row and the categories pack vertically like Pinterest tiles (CSS multi-column "masonry") — each card is only as tall as its content. **Rearranging cards (v1.16):** widget and category cards drag with a custom pointer-drag — a faded clone follows the cursor and a highlighted "landing slot" shows where the card drops; works on touch (iPhone); each card also has hover-revealed ▲/▼ move arrows.

**Backgrounds:** 10 CSS-only dark presets (Midnight Mesh, Aurora, Synthwave, Carbon Grid, Deep Space, Liquid Glass, Tokyo Night, Terminal, Monochrome, **Starlink** as of v1.14), or any image URL. Starlink auto-applies a matched electric-blue accent (overridable).

**Widgets** (per-persona; show/hide, drag-to-reorder, optional glow color as of v1.10): Clock (multi-zone, 12/24hr), Weather (Open-Meteo, no key — each day shows icon + label, high/low, "🌧 NN%" as of v1.12), To-do, Sticky notes, Crypto (CoinGecko), Markets (Finnhub, fixed SPY/QQQ/DIA), Watchlists (Finnhub, multiple/renamable, v1.10).

**Also an "Ask Gemini" AI chat panel (v1.11)** — a *global* panel, not a widget. A bottom-right bubble opens a slide-over chat for concise answers, on the free Gemini API, with an "Open full Gemini" link-out. As of v1.13 it auto-retries Google's transient "overloaded" 503s, with a Retry button if retries run out.

---

# Where We Are

## Most recent session (2026-05-29 — v1.16)

Rebuilt how widget and category cards are rearranged — `index.html` only, no new deps:

- Replaced the flaky native HTML5 drag (which also guessed the wrong direction after v1.15's masonry change) with **one shared vanilla pointer-drag engine**, `initCardDrag({ grip, card, container, cardSelector, onDrop })`. On grip `pointerdown` it builds a fixed `.drag-clone` that follows the cursor and a `.drop-placeholder` "landing slot"; on `pointermove` it picks the nearest card by 2D distance and decides before/after by the **dominant axis**; on `pointerup` it computes the index and fires `onDrop`. Pointer events → **mouse and touch (iPhone)**. Widget drop = anchor-key splice of `widgetOrder` (hidden widgets don't skew the index); category drop = splice `persona.categories` at the index.
- Added **hover-revealed ▲/▼ move arrows** on every widget + category card (`moveCardInOrder` / `buildMoveControls`) for a no-drag nudge. New CSS: `.drag-clone`, `.drop-placeholder`, `.card-dragging`, `.move-controls`/`.move-btn`; kept `.drop-before`/`.drop-after` (bookmark rows still use them).
- **Native drag for bookmarks, persona tabs, and config lists is intentionally untouched.** `moveWidget` is now unused (left with a note).
- **Not verified live:** sandbox can't fully load the app (needs gist auth); `node --check` passes, index math traced. Tom should smoke-test.

## Sessions before that

- **v1.15 (2026-05-29):** Masonry layout — widget row and categories switched from equal-height grids to CSS multi-column (vertical packing), so a tall card no longer stretches neighbors or pushes categories down. CSS-only.
- **v1.14 (2026-05-29):** "Starlink" CSS-only deep-space background preset (10th); selecting it auto-applies a matched electric-blue accent (`#4F9CFF`, overridable) via a new optional `accent` field on the preset.
- **v1.13 (2026-05-29):** Ask Gemini chat auto-retries Google's transient 503 "overloaded" (2x) with a Retry button; confirmed the browser-direct call isn't CORS-blocked.
- **v1.10–v1.12 (2026-05-28):** Drag-to-reorder widget cards, multiple renamable watchlists, per-card glow color (v1.10); the "Ask Gemini" chat panel (v1.11); richer weather forecast (v1.12).
- **v1.0–v1.9 (2026-05-27 → 28):** Initial build through earlier polish (deploy, finance widgets, Finnhub migration, light mode, hotfixes). Full history in PROJECT-HUB.

## Immediate next step

1. **Push v1.16** (single-file `index.html` commit), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Smoke-test the drag:** drag a widget and a category by the ⠿ grip — confirm the highlighted landing slot shows where it'll go and the card drops there reliably. Try the ▲/▼ arrows.
3. **Check light + dark mode, and iPhone** (touch dragging is new).
4. **Possible future build (not started):** a backup/export of dashboard data.

---

# Key Decisions Made (and Why)

- **Custom vanilla pointer-drag for cards, replacing native HTML5 DnD (v1.16)** — native drag was unreliable and, after v1.15's masonry switch, computed the wrong drop axis (treated cards as a left-to-right row). One shared `initCardDrag` engine fixes reliability + masonry correctness (dominant-axis before/after), adds touch support for free (iPhone), pulls in no library, and shows a real landing-slot highlight. Bookmarks/tabs/config-lists kept on native drag this round.
- **Masonry via CSS multi-column, not experimental `grid-template-rows:masonry` (v1.15)** — fixes "tall card stretches neighbors / pushes categories down"; multi-column is well-supported today. Drag-reorder reads per-card geometry, so it's layout-agnostic.
- **GitHub gist as the data backend; no backend, browser-direct, single file** — durable, free, zero-dependency; chosen over Supabase/jsonbin/iCloud. Per-browser PATs in localStorage avoid OAuth in a static page.
- **Per-persona everything** (widgets, backgrounds, accent, order, colors). **Per-device, not synced:** default persona, theme, credentials, Gemini chat history — syncing keys/transcripts through a gist would be wrong.
- **Starlink background auto-applies a matched accent but doesn't lock it (v1.14)** — an optional `accent` field on the preset; overridable. CSS-only, no light variant.
- **Finnhub over Twelve Data (v1.9)** — 60 calls/min vs 8; Tom's ~12 symbols hit chronic 429s. Both free, same REST shape.
- **Ask Gemini = Gemini Flash, browser-direct, no backend (v1.11)** — free, fits "fast quick answer." Model is a Settings override (default `gemini-2.5-flash`). Native `:generateContent` confirmed not CORS-blocked; a note + "Open full Gemini" link-out covers failures. **Auto-retries the transient 503 "overloaded" (v1.13)**; a 429 is never retried.
- **Multiple watchlists via array + composite `watchlist:<id>` keys (v1.10)** — each uses the same reorder/hide/color/config machinery as a built-in widget. **Markets stays a singleton index-proxy card (SPY/QQQ/DIA).**
- **Light mode = one light surface + tokenized CSS (v1.6); bookmarklet over an extension; CSS-only backgrounds + image URL, no upload** (upload would bloat the gist).
- **Empty Settings credential field = "no change" (v1.8)** — saving is never destructive; explicit clear is Danger Zone → Reset browser. **Last-write-wins on sync.** **iPhone slim mode deferred indefinitely.**

Minor decisions live in `docs/TECHNICAL.html`.

---

# The Stack

- **Languages:** HTML, CSS, JavaScript (vanilla — no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs (all browser-direct, no backend):** GitHub REST (gist read/write); Open-Meteo (weather + geocoding, no key); CoinGecko (crypto, no key); Finnhub (stock quotes + search, free key, 60/min — replaced Twelve Data in v1.9); Google Gemini (AI chat, free key — `:generateContent`, default `gemini-2.5-flash`, auto-retries 503s as of v1.13); Google Favicon (bookmark icons, best-effort).
- **Storage:** One private GitHub gist (account `TECGUNTHER`) holds all persona data. Per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub key, Gemini key/model/history, theme, and a shared 5-min quote cache. (A legacy Twelve Data key is read but unused.)
- **Hosting:** GitHub Pages (free static), public repo `github.com/TECGUNTHER/startdashboard`.

**Layout + card drag:** `.widgets-row` and `.categories-grid` use CSS multi-column masonry (knobs `column-count` / `column-width`; cards use `break-inside:avoid`). Widget + category cards drag via `initCardDrag` (clone + `.drop-placeholder` landing highlight + dominant-axis nearest-slot + anchor-key splice; pointer events = touch-capable). **Data model (v1.10):** each persona has `categories` (optional `color`), `widgets`, `watchlists` (`[{id, title, symbols[]}]`), `widgetOrder` (incl. composite `watchlist:<id>` keys), `widgetColors`, `hiddenWidgets`. `BG_PRESETS` = 10 entries, each `{ id, name }` + optional `accent` (only Starlink). Gemini chat is global, history outside the gist. Full schema in `docs/TECHNICAL.html`.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (~5,450 lines)
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
Local-only preview (no sync): double-click `index.html`.

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

(Full troubleshooting list lives in `PROJECT-HUB.html`.)

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's setup walkthrough · `INSTALL-FOR-OTHERS.md` — fork-and-run guide for anyone he shares with
- `PROJECT-HUB.html` — Tom's home base · `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture/decisions/how-to-modify · `docs/USER-EXPERIENCE.html` — features and workflows
- `plans/` — approved build plans (through v1.16), read-only history

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
- Ask Gemini is a global panel, not a widget; a fresh assistant with no link to Tom's real Gemini history. Auto-retries 503; a 429 is not retried.

## What he's likely to want next
- More polish from real use. Near-term: verify v1.16; possibly the Crypto cache, a data backup/export, or extending the new pointer-drag engine to bookmarks/tabs/config lists. Eventually a Custom embed widget or more financial widgets.

---

# Deeper Reference

For more than this doc provides: `docs/TECHNICAL.html` (architecture, decisions), `docs/USER-EXPERIENCE.html` (screens, workflows), `PROJECT-HUB.html` (full state + session history), `plans/` (build plans).
