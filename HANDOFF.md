# Quick-Start Prompt

> Read this entire document before responding. It contains everything you need
> to know about a project I'm working on. After reading, summarize back to me
> in three sentences: (1) what this project is, (2) what state it's in,
> (3) what the immediate next step would be. Do not write code or take action
> until I confirm your summary is correct.

---

# About Tom (Your User)

- Senior Director of Pre-Sales Solutions Engineering (collaboration platforms). Not a professional developer; technically fluent but can't read raw stack traces or jargon without translation.
- Plain English only. Hates marketing language, filler, and over-explanation. Prefers "added a Q4 toggle" over "implemented quarterly filtering."
- Default browser is Safari (Mac). Phone browsing is mostly inbound link-taps, not outbound surfing.
- When testing, he logs UX issues as Open Items and batch-fixes at the end — don't stop to fix each one mid-test.
- For major changes he expects the real multi-agent flow: a Build Agent subagent for code, a Documentation Agent subagent for docs.

---

# The Project

**Name:** StartDashboard
**Folder:** `projects/personal-dashboard/` in Tom's `tom-workspace`
**Current status:** Complete (in active use; iterating on polish)
**Current phase:** Complete (v1.13 shipped)
**Last updated:** 2026-05-29
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that serves as Tom's home page across every browser and device. A top tab bar switches between "personas" (Work, Personal, Investments, etc.); each is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent color. All data lives in one private GitHub gist, so the same URL shows the same synced view everywhere within seconds. Dark/modern by default; a light-mode toggle (v1.6) covers daytime use.

**Widgets** (per-persona; show/hide, drag-to-reorder, optional border/glow color as of v1.10): Clock (multi-zone, 12/24hr), Weather (Open-Meteo, no key — each day shows icon + label, high/low, "🌧 NN%" as of v1.12), To-do, Sticky notes, Crypto (CoinGecko), Markets (Finnhub — fixed SPY/QQQ/DIA proxies), Watchlists (Finnhub — multiple, renamable, v1.10).

**Also: an "Ask Gemini" AI chat panel (v1.11)** — a *global* panel, not a widget. A bottom-right floating bubble opens a slide-over chat for concise answers from every persona, on the free Google Gemini API, with an "Open full Gemini" link-out. As of v1.13 it auto-retries Google's transient "overloaded" 503s, with a Retry button if retries run out.

---

# Where We Are

## Most recent session (2026-05-29 — v1.13)

Shipped a **reliability fix to the Ask Gemini chat** (only `index.html` changed; no new deps/keys): it now auto-retries Google's transient **503 "model overloaded"** responses instead of showing them as errors — up to 2x (3 attempts total) with short backoffs (~1.5s/3s + jitter), keeping the "thinking…" indicator up. If all retries fail, it shows a plain-English note ("Google's servers are briefly overloaded — on Google's side, not you. Try again in a moment.") with a **Retry** button + the link-out. Constants `CHAT_OVERLOAD_MAX_RETRIES` / `CHAT_OVERLOAD_BACKOFFS` tune it; the retry loop lives in `runChatRequest` (split out of `sendChatMessage` so Retry re-issues without re-pushing the user's turn). 429 and network/CORS handling unchanged. Details in `docs/TECHNICAL.html`.

Also: the v1.11 "confirm Gemini browser-direct CORS works" open question is **closed** — Tom confirmed the chat works in his browser. That open item was removed.

## Sessions before that

- **v1.12 (2026-05-28):** Weather forecast detail — each day shows a condition icon + label and a "🌧 NN%" rain line alongside high/low.
- **v1.11 (2026-05-28):** Built the "Ask Gemini" chat panel — browser-direct, no backend, history per-device in localStorage.
- **v1.10 (2026-05-28):** Biggest release since v1.0 — drag-to-reorder widget cards, multiple renamable watchlists, per-card border/glow color.
- **v1.9 (2026-05-28):** Migrated stock data Twelve Data → **Finnhub** (8→60 calls/min, fixing chronic 429s); shared 5-min quote cache.
- **v1.0–v1.8 (2026-05-27 → 28):** Initial build through earlier polish (deploy, finance widgets, light mode, hotfixes). Full history in PROJECT-HUB.

## Immediate next step

1. **Push v1.13** (single-file `index.html` commit), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Verify chat recovers from a 503:** use the Ask Gemini chat normally; on a Google "high demand" moment it should keep "thinking…" and recover on its own, or show the overloaded note with a working Retry button.
3. **Possible future build (not started):** a backup/export of dashboard data.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend; no backend, browser-direct, single file** — durable, free, zero-dependency; chosen over Supabase/jsonbin/iCloud.
- **Per-browser Personal Access Tokens in localStorage** — avoids OAuth in a static page; losing a device means revoking one token, not all.
- **Per-persona everything (widgets, backgrounds, accent, visibility, order, colors)** — personas exist for mental separation, so each is fully independent.
- **Per-device, not synced: default persona, theme, credentials (GitHub/Finnhub/Gemini keys), Gemini chat history** — per-environment by nature; syncing keys/transcripts through a gist would be wrong.
- **Finnhub over Twelve Data (v1.9)** — Twelve Data was 8 calls/min and Tom's ~12 symbols hit chronic 429s. Finnhub is 60/min, also free, same REST shape.
- **Ask Gemini uses Gemini Flash, not Claude/OpenAI (v1.11)** — free with no card, fits "fast quick answer." Model is a Settings override (default `gemini-2.5-flash`) so a rename needs no code change.
- **Gemini browser-direct with a graceful link-out fallback (v1.11)** — keeps the no-backend design; the native `:generateContent` endpoint is confirmed working (not CORS-blocked); the note + link-out covers any future connection failure.
- **Auto-retry Gemini's transient 503 "overloaded" (v1.13)** — a 503 is a Google-side capacity blip that usually clears in a second or two, so the chat retries up to 2x (short backoff) rather than surfacing it. No 30s+ backoff, no model-switching. Distinct from a 429, which is never retried.
- **Multiple watchlists via an array + composite `watchlist:<id>` keys (v1.10)** — lets each watchlist use the same reorder/hide/color/config machinery as a built-in widget without special-casing.
- **Markets stays a singleton index-proxy card (SPY/QQQ/DIA)** — different purpose from user-curated watchlists.
- **Light mode = one clean light surface + tokenized CSS (v1.6)**; **bookmarklet over an extension**; **CSS-only backgrounds + image URL, no upload** (upload would bloat the gist with base64).
- **Empty Settings credential field = "no change" (v1.8)** — saving must never be accidentally destructive. Explicit clear is Danger Zone → Reset browser.
- **Last-write-wins on sync conflicts** — single-user app; a toast warns when another browser updated. **iPhone slim mode deferred indefinitely** — phone is inbound-link-driven.

Minor decisions live in `docs/TECHNICAL.html`.

---

# The Stack

- **Languages:** HTML, CSS, JavaScript (vanilla — no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs (all browser-direct, no backend):** GitHub REST (gist read/write); Open-Meteo (weather + geocoding, no key); CoinGecko (crypto, no key); Finnhub (stock quotes + search, free key, 60/min — replaced Twelve Data in v1.9); Google Gemini (AI chat, free key — `:generateContent`, default `gemini-2.5-flash`, auto-retries transient 503s as of v1.13); Google Favicon service (bookmark icons, best-effort).
- **Storage:** One private GitHub gist (account `TECGUNTHER`) holds all persona data. Per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub key, Gemini key + model + chat history, theme, and a shared 5-min quote cache. (A legacy Twelve Data key is read for backward-compat but unused.)
- **Hosting:** GitHub Pages (free static), public repo `github.com/TECGUNTHER/startdashboard`.

**Data model (v1.10):** each persona has `categories` (optional `color`), `widgets`, `watchlists` (array of `{id, title, symbols[]}`), `widgetOrder` (incl. composite `watchlist:<id>` keys), `widgetColors`, `hiddenWidgets`. Gemini chat is global, history outside the gist. Full schema in `docs/TECHNICAL.html`.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (~5,357 lines)
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
- GitHub tokens: https://github.com/settings/tokens · Finnhub key: https://finnhub.io/register · Gemini key (free, no card): https://aistudio.google.com/app/apikey
- Gist ID: dashboard → Settings (⚙) → Sync → Gist ID

## One-time setup (already done)
Deployed and set up. To repeat on a fresh machine: push to the repo, enable Pages (Settings → Pages → main / root), generate a PAT with `gist` scope and paste into the setup screen, then add free Finnhub + Gemini keys. Full walkthrough in `README.md`.

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
- **"Bad credentials" on first run** → PAT wrong or missing `gist` scope. Regenerate.
- **Sync indicator red ⚠** → Click to reload; if still failing the token may have expired — reset this browser in Settings → Danger Zone.
- **Cross-browser changes not showing** → Reload the other browser (poll is 60s; manual reload is instant).
- **Markets/Watchlists say "Add your Finnhub API key" (v1.9)** → Add a free key from finnhub.io/register; the old Twelve Data key is ignored.
- **"Rate limit reached" (v1.9)** → Rate-limited with nothing cached; the widget backs off and auto-refreshes after 60s. No action.
- **Ask Gemini says "Add your free Gemini API key" (v1.11)** → Add one from aistudio.google.com/app/apikey. The link-out works without a key.
- **Chat says "Google's servers are briefly overloaded" with a Retry button (v1.13)** → A Google-side 503 blip; the chat already auto-retried twice. Click Retry or ask again in a moment.

(Full troubleshooting list lives in `PROJECT-HUB.html`.)

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's setup walkthrough · `INSTALL-FOR-OTHERS.md` — fork-and-run guide for anyone he shares with
- `PROJECT-HUB.html` — Tom's home base · `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture/decisions/how-to-modify · `docs/USER-EXPERIENCE.html` — features and workflows
- `plans/` — approved build plans (through v1.13), read-only history

---

# Open Items

- **(Feature)** Cache the Crypto widget's price fetches — it re-fetches CoinGecko on every full render, and v1.10's reorder triggers more renders. Add a shared cache like the stock widgets' 5-min one if rate-limiting appears. (added 2026-05-28)
- **(UX Fix, low priority)** iPhone slim mode — deferred indefinitely (Tom's phone is inbound-link-driven). (added 2026-05-27)
- **(Question)** Mobile Safari drag-drop behavior on iPhone — only matters if Tom starts managing bookmarks on phone. (added 2026-05-27)
- **(Feature)** Custom embed widget for pasting iframes (calendar, RSS, etc.) — only when Tom has a specific use case. (added 2026-05-27)

---

# Working with Tom (Behavioral Notes)

## Communication norms
- Plain English, no jargon without translation. One question at a time. Lead with the answer. No filler enthusiasm. Be honest about limits.

## Project-specific quirks
- Single-file HTML by design — don't suggest splitting into modules without a real reason.
- The *app* (`index.html`) is dark/modern with a v1.6 light option; the workspace *docs* use a light/restrained aesthetic — don't mix the two.
- Tom is the only user — no multi-user or sharing model. All data lives on his GitHub account; don't add third-party services without checking (Gemini is the one beyond data APIs).
- Per-persona settings by default. Exceptions (per-device globals): theme, credentials, Gemini chat + history.
- Markets is intentionally a single fixed index-proxy card — don't make it renamable or multiple. Watchlists are the renamable/multiple concept.
- The Ask Gemini chat is a global panel, not a widget; a fresh assistant with no link to Tom's personal Gemini history. Auto-retries 503 "overloaded" (v1.13); a 429 is not retried.

## What he's likely to want next
- More polish from real use (he iterates fast on friction). Near-term: verify v1.13; possibly the Crypto cache or a data backup/export; eventually a Custom embed widget or more financial widgets.

---

# Deeper Reference

For more than this doc provides: `docs/TECHNICAL.html` (architecture, decisions), `docs/USER-EXPERIENCE.html` (screens, workflows), `PROJECT-HUB.html` (full state and session history), `plans/` (build plans).
