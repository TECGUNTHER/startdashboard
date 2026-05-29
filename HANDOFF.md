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
**Current phase:** Complete (v1.15 shipped)
**Last updated:** 2026-05-29
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that's Tom's home page across every browser/device. A top tab bar switches between "personas" (Work, Personal, Investments, etc.); each is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent. All data lives in one private GitHub gist, so the same URL shows the same synced view everywhere within seconds. Dark/modern by default; a light-mode toggle (v1.6) covers daytime use.

**Layout (v1.15):** the widget row and the categories pack vertically like Pinterest tiles (CSS multi-column "masonry") — each card is only as tall as its content, so a tall card no longer stretches its neighbors or pushes the categories down.

**Backgrounds:** 10 CSS-only dark presets (Midnight Mesh, Aurora, Synthwave, Carbon Grid, Deep Space, Liquid Glass, Tokyo Night, Terminal, Monochrome, **Starlink** as of v1.14), or any image URL. Starlink also auto-applies a matched electric-blue accent (overridable) when selected.

**Widgets** (per-persona; show/hide, drag-to-reorder, optional glow color as of v1.10): Clock (multi-zone, 12/24hr), Weather (Open-Meteo, no key — each day shows icon + label, high/low, "🌧 NN%" as of v1.12), To-do, Sticky notes, Crypto (CoinGecko), Markets (Finnhub, fixed SPY/QQQ/DIA), Watchlists (Finnhub, multiple/renamable, v1.10).

**Also an "Ask Gemini" AI chat panel (v1.11)** — a *global* panel, not a widget. A bottom-right bubble opens a slide-over chat for concise answers from every persona, on the free Gemini API, with an "Open full Gemini" link-out. As of v1.13 it auto-retries Google's transient "overloaded" 503s, with a Retry button if retries run out.

---

# Where We Are

## Most recent session (2026-05-29 — v1.15)

Switched the **widget row and the categories to a masonry (vertical-packing) layout** — CSS only, `index.html` only, no JS logic change, no new deps:

- `.widgets-row`: grid `repeat(4,1fr)` → `column-count:4` (with `column-count:2` ≤1100px, `1` ≤600px). `.categories-grid`: grid `auto-fill minmax(280px,1fr)` → `column-width:280px` (+ a `column-count:1` rule ≤700px). Each `.widget`/`.category`/`.new-category-card` got `break-inside:avoid` (+ webkit/page-break for Safari), `width:100%`, `margin-bottom:16px`. (Exact CSS in PROJECT-HUB / TECHNICAL.)
- **Drag-reorder needed no change** — `moveWidget` splices `widgetOrder`, category drop splices `persona.categories`, both read per-card `getBoundingClientRect` (not grid geometry), so they're layout-agnostic. `.drop-before`/`.drop-after` kept.
- **Not verified live:** the Build Agent's sandbox had no browser, so static checks only (JS parses clean); it asked Tom to eyeball the packing and grip-drag one widget + one category once live.

## Sessions before that

- **v1.14 (2026-05-29):** "Starlink" CSS-only deep-space background preset (10th); selecting it auto-applies a matched electric-blue accent (`#4F9CFF`, overridable) via a new optional `accent` field on the preset.
- **v1.13 (2026-05-29):** Ask Gemini chat auto-retries Google's transient 503 "overloaded" (2x) instead of surfacing it, with a Retry button; confirmed the browser-direct call isn't CORS-blocked.
- **v1.12 (2026-05-28):** Weather forecast — each day shows a condition icon + label and a "🌧 NN%" rain line.
- **v1.10–v1.11 (2026-05-28):** Drag-to-reorder widget cards, multiple renamable watchlists, per-card glow color (v1.10); the "Ask Gemini" chat panel (v1.11).
- **v1.0–v1.9 (2026-05-27 → 28):** Initial build through earlier polish (deploy, finance widgets, Finnhub migration, light mode, hotfixes). Full history in PROJECT-HUB.

## Immediate next step

1. **Push v1.15** (single-file `index.html` commit), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Eyeball the packing:** on a persona with a tall widget (a long Watchlist), confirm the short cards beside it are no longer stretched and the categories sit higher up.
3. **Grip-drag one widget and one category** (the ⠿ handle) to confirm reorder still works and the order persists — the one thing the Build Agent couldn't auto-verify.
4. **Possible future build (not started):** a backup/export of dashboard data.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend; no backend, browser-direct, single file** — durable, free, zero-dependency; chosen over Supabase/jsonbin/iCloud.
- **Per-browser Personal Access Tokens in localStorage** — avoids OAuth in a static page; losing a device means revoking one token, not all.
- **Per-persona everything** (widgets, backgrounds, accent, visibility, order, colors) — personas are for mental separation, so each is fully independent. **Per-device, not synced:** default persona, theme, credentials (GitHub/Finnhub/Gemini keys), Gemini chat history — per-environment by nature; syncing keys/transcripts through a gist would be wrong.
- **Masonry via CSS multi-column, not the experimental `grid-template-rows:masonry` (v1.15)** — vertical packing fixes "tall card stretches its neighbors / pushes categories down"; multi-column is well-supported today, native CSS masonry isn't. No internal-scroll height caps or per-card width spanning this round (both deferred). Drag-reorder reads per-card geometry, so it's layout-agnostic.
- **Starlink background auto-applies a matched accent, but doesn't lock it (v1.14)** — complete out of the box yet overridable; an **optional `accent` field on the preset** (not a separate map). CSS-only, no light variant.
- **Finnhub over Twelve Data (v1.9)** — 60 calls/min vs 8; Tom's ~12 symbols hit chronic 429s. Both free, same REST shape.
- **Ask Gemini = Gemini Flash, browser-direct, no backend (v1.11)** — free with no card, fits "fast quick answer." Model is a Settings override (default `gemini-2.5-flash`). The native `:generateContent` endpoint is confirmed working (not CORS-blocked); a note + "Open full Gemini" link-out covers any connection failure.
- **Auto-retry Gemini's transient 503 "overloaded" (v1.13)** — a Google-side blip that clears in a second or two, so retry up to 2x (short backoff) rather than surface it. A 429 is never retried.
- **Multiple watchlists via an array + composite `watchlist:<id>` keys (v1.10)** — lets each use the same reorder/hide/color/config machinery as a built-in widget. **Markets stays a singleton index-proxy card (SPY/QQQ/DIA)** — different purpose from user watchlists.
- **Light mode = one clean light surface + tokenized CSS (v1.6); bookmarklet over an extension; CSS-only backgrounds + image URL, no upload** (upload would bloat the gist).
- **Empty Settings credential field = "no change" (v1.8)** — saving must never be destructive; explicit clear is Danger Zone → Reset browser. **Last-write-wins on sync** (single-user; a toast warns on conflict). **iPhone slim mode deferred indefinitely.**

Minor decisions live in `docs/TECHNICAL.html`.

---

# The Stack

- **Languages:** HTML, CSS, JavaScript (vanilla — no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs (all browser-direct, no backend):** GitHub REST (gist read/write); Open-Meteo (weather + geocoding, no key); CoinGecko (crypto, no key); Finnhub (stock quotes + search, free key, 60/min — replaced Twelve Data in v1.9); Google Gemini (AI chat, free key — `:generateContent`, default `gemini-2.5-flash`, auto-retries 503s as of v1.13); Google Favicon (bookmark icons, best-effort).
- **Storage:** One private GitHub gist (account `TECGUNTHER`) holds all persona data. Per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub key, Gemini key/model/history, theme, and a shared 5-min quote cache. (A legacy Twelve Data key is read but unused.)
- **Hosting:** GitHub Pages (free static), public repo `github.com/TECGUNTHER/startdashboard`.

**Layout (v1.15):** both `.widgets-row` and `.categories-grid` use CSS multi-column masonry (natural card heights). Knobs: `column-count` (widgets), `column-width` (categories); `.widget`/`.category`/`.new-category-card` use `break-inside:avoid` so cards don't split across columns. **Backgrounds:** 10 presets in `BG_PRESETS`, each `{ id, name }` + optional `accent` (only Starlink, `#4F9CFF`). **Data model (v1.10):** each persona has `categories` (optional `color`), `widgets`, `watchlists` (`[{id, title, symbols[]}]`), `widgetOrder` (incl. composite `watchlist:<id>` keys), `widgetColors`, `hiddenWidgets`. Gemini chat is global, history outside the gist. Full schema in `docs/TECHNICAL.html`.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (~5,390 lines)
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
- `plans/` — approved build plans (through v1.15), read-only history

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
- Single-file HTML by design — don't suggest splitting into modules without a real reason.
- The *app* (`index.html`) is dark/modern (v1.6 light option); the workspace *docs* are light/restrained — don't mix the two.
- Tom is the only user — no multi-user/sharing. Data lives on his GitHub; don't add third-party services without checking (Gemini is the one beyond data APIs).
- Per-persona by default. Per-device globals: theme, credentials, Gemini chat + history.
- Markets is a single fixed index-proxy card — don't make it renamable/multiple; Watchlists are the multiple concept.
- Layout is CSS multi-column masonry (v1.15) — cards keep natural height; don't reintroduce equal-height grids. Drag-reorder is layout-agnostic.
- Backgrounds are CSS-only presets (10) or an image URL — no uploads. A preset may carry an optional `accent` (only Starlink) that auto-applies but stays overridable.
- Ask Gemini is a global panel, not a widget; a fresh assistant with no link to Tom's real Gemini history. Auto-retries 503; a 429 is not retried.

## What he's likely to want next
- More polish from real use. Near-term: verify v1.15; possibly the Crypto cache or a data backup/export; eventually a Custom embed widget or more financial widgets.

---

# Deeper Reference

For more than this doc provides: `docs/TECHNICAL.html` (architecture, decisions), `docs/USER-EXPERIENCE.html` (screens, workflows), `PROJECT-HUB.html` (full state + session history), `plans/` (build plans).
