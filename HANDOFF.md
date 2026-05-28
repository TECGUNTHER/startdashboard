# Quick-Start Prompt

> Read this entire document before responding. It contains everything you need
> to know about a project I'm working on. After reading, summarize back to me
> in three sentences: (1) what this project is, (2) what state it's in,
> (3) what the immediate next step would be. Do not write code or take action
> until I confirm your summary is correct.

---

# About Tom (Your User)

- Senior Director of Pre-Sales Solutions Engineering. Technology sales, collaboration platforms (UCaaS, CCaaS, Webex, contact center).
- Not a professional developer. Technically fluent. Cannot interpret raw stack traces or developer jargon without translation.
- Communicates in plain English. Hates marketing language, filler phrases, and over-explanation. Prefers concrete language ("added a Q4 toggle") to corporate language ("implemented quarterly filtering").
- Default browser on his Mac is Safari (not Chrome). His phone browsing is mostly inbound link-taps from text/email/social — not outbound surfing.
- When testing a freshly built thing, his strong preference is: log UX issues as Open Items, continue testing, batch-fix at the end. Don't stop to fix each one individually mid-test.
- For major changes, Tom expects the **multi-agent flow** to run for real: a Build Agent subagent for code, a Documentation Agent subagent for docs. Not one chat playing every role.

---

# The Project

**Name:** StartDashboard
**Folder:** `projects/personal-dashboard/` in Tom's `tom-workspace`
**Current status:** Complete (in active use; iterating on polish)
**Current phase:** Complete (with v1.9 shipped)
**Last updated:** 2026-05-28
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that becomes Tom's home page across every browser and device. A tab bar at the top lets him switch between "personas" (Work, Personal, Investments, anything he creates). Each persona is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent color. Data lives in one private GitHub gist on his account, so opening the same URL in Chrome on his Mac and Safari on his iPhone shows the same view, synced within seconds. Default aesthetic is dark and modern (Linear/Vercel/Raycast as references); a v1.6 light-mode toggle covers daytime / bright-light use.

**Available widgets (per-persona, can be shown or hidden individually):**
- Clock (multi time zone, 12/24hr toggle)
- Weather (Open-Meteo, no key)
- To-do list
- Sticky notes
- Crypto (CoinGecko, no key — shows price + dollar + percent change)
- Markets (Finnhub — defaults to SPY/QQQ/DIA for S&P/Nasdaq/Dow)
- Watchlist (Finnhub — Tom's custom stocks)

---

# Where We Are

## Most recent session (2026-05-28 — v1.9)

A significant release with three changes:

- **Migrated stock data from Twelve Data to Finnhub.** Twelve Data's free tier was 8 price lookups/min, and Tom's ~12 tracked symbols routinely exceeded that, producing chronic HTTP 429 ("too many requests") errors. Finnhub's free tier is 60 calls/min (no credit card). The old `fetchTwelveDataQuotes` was replaced with `fetchFinnhubQuotes(symbols)` — one `/quote` call per symbol fanned out via `Promise.all`, normalized from Finnhub's `c`/`d`/`dp` fields into the existing `{price, dollar, percent}` shape. Symbol search moved to Finnhub's `/search` endpoint. A failed symbol degrades to "—" instead of killing the widget. The shared stock render function was renamed `renderTwelveDataWidget` → `renderStockWidget`. The old Twelve Data key is still read for backward-compat but unused. **Tom must paste a free Finnhub key (finnhub.io/register) into Settings.**
- **Added a shared 5-minute price cache.** Replaced the old per-tab in-memory cache (`_tdCache`) with a localStorage-backed cache (`sd_quote_cache`, 5-min TTL) shared across all tabs and reloads, so opening multiple tabs no longer multiplies API calls. Only stale/missing symbols hit the network. On a rate limit with nothing cached, the widget shows "Rate limit reached — prices refresh shortly" and refreshes once 60s later (the v1.8 immediate 2s retry was dropped for stocks because it made rate-limiting worse).
- **Overhauled drag-to-reorder with visible grip handles.** The old HTML5 drag was undiscoverable — Tom couldn't tell what to grab. Now categories, bookmarks, and every config list (stock tickers, weather cities, clock zones) show a draggable ⠿ grip handle; an accent line shows where the item will drop. Bookmark drops are position-aware (top/bottom half of a row) via a new `moveBookmark()` helper handling within- and cross-category moves. Config lists use a shared `makeCfgItemDraggable()` helper.

## Sessions before that

- **v1.8 hotfix (2026-05-28, evening):** Empty Settings key field no longer wipes the stored API key ("no change," not "clear"). Finance widgets gained a one-shot cold-start retry at 2s. Active persona tab restyled from a solid-accent slab to a calmer soft-accent fill + accent outline + bold text. New `INSTALL-FOR-OTHERS.md` install guide. (Note: v1.9 dropped the stock-widget retry; crypto kept it.)
- **v1.7 hotfix (2026-05-28, late afternoon):** Negative finance prices fixed to 2 decimals (`formatPrice()` branches on `Math.abs()` and re-applies the sign). Bookmark description popover spacing bumped to 28px to clear Safari's native ellipsis tooltip. Active persona tab made solid-accent to fix "no tab selected on first frame" (softened in v1.8).
- **v1.6 (earlier 2026-05-28):** Light/dark theme toggle (persists per browser), finance prices trimmed to 2 decimals, bookmark URL moved into the hover popover, "Default persona on this device" surfaced at top of Settings, CSS palette tokenized. First build to run the real subagent flow.

## Immediate next step

1. **Generate a free Finnhub API key** at finnhub.io/register (no credit card) and paste it into Settings — the field is now labeled Finnhub. Without it, Markets/Watchlist show an "add your Finnhub key" prompt.
2. **Push v1.9 to GitHub** (single-file commit of `index.html`), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
3. **Smoke-test:** (a) Markets/Watchlist load with no 429/rate-limit errors; (b) the ⠿ grip handles are visible on categories, bookmarks, and config lists, and dragging reorders and sticks after reload; (c) a bookmark dragged into another category lands where dropped.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend** — durable, free, no third-party service to outlive Tom's GitHub account. Chosen over Supabase / jsonbin / iCloud-folder sync (Safari can't write back to local files).
- **Per-browser Personal Access Tokens stored in localStorage** — avoids OAuth in a static page. Each browser gets its own token; losing a device means revoking one token, not all.
- **Per-persona widgets, backgrounds, accent colors, and visibility** — the whole point of personas is mental separation. Each persona is fully independent.
- **Per-device default persona** (localStorage) — Tom's work Mac opens to Work, iPhone opens to Personal. Not synced through the gist.
- **Per-device theme choice** (localStorage, v1.6) — light vs dark is a per-environment preference, not a per-account one.
- **Light mode replaces the persona background with a single clean light surface** (v1.6) — not per-persona light gradients (would double config burden). Persona accent color is preserved in both themes.
- **CSS palette tokenized for theming** (v1.6) — the whole `body.light-mode` skin is one override block. Future themes are cheap.
- **Bookmarklet over a browser extension** — one JavaScript snippet that works everywhere; no per-browser build burden.
- **Backgrounds: CSS-only presets + image URL only (no upload)** — image upload would bloat the gist with base64 on every save.
- **Finnhub for stocks, replacing Twelve Data (v1.9)** — Twelve Data's free tier was 8 calls/min; Tom's ~12 symbols blew past it and hit chronic HTTP 429s. Finnhub's free tier is 60/min (also free, no credit card). Both are REST APIs, so this was a swap with no new dependency. CoinGecko (crypto, no key) is unchanged.
- **Shared 5-minute localStorage quote cache (v1.9)** — replaced the old per-tab in-memory cache. Multiple tabs/reloads were multiplying API calls toward the rate limit; a shared 5-min cache means only stale/missing symbols hit the network.
- **On a 429, back off 60 seconds instead of retrying immediately (v1.9)** — v1.8's 2-second retry made rate-limiting worse by piling on more calls under the limit. One delayed refresh recovers cleanly. Per-symbol failures degrade to "—" instead of retrying.
- **Visible ⠿ grip handles for drag-to-reorder (v1.9)** — bare HTML5 drag had no affordance; Tom literally couldn't find what to grab. An explicit grip on categories, bookmarks, and config-list items makes the drag target obvious; a drop-indicator line shows where it'll land.
- **API key per-browser in localStorage (parallel to GitHub PAT)** — rate limits are per-key; don't sync credentials through a gist.
- **Markets uses ETF proxies (SPY, QQQ, DIA)** for major indices since the actual indices need higher API tiers.
- **Clock zones stored as {name, tz} objects (since v1.3)** — allows multiple cities in the same timezone to render as separate rows.
- **Bookmark descriptions as hover popover (since v1.5)** — keeps the row clean; full text appears after a 0.5s delay.
- **`formatPrice` shows 2 decimals ≥ $0.01, 4 below; negative-aware formatters branch on `Math.abs()` and re-apply the sign (v1.6/v1.7)** — prices that look like prices; negatives don't fall through to the wrong branch.
- **Active persona tab uses soft-accent fill + accent outline + bold text (v1.8)** — v1.7's solid slab fixed discoverability but was visually loud; the soft fill keeps the win without overshooting.
- **Empty Settings input means "no change" for credentials (v1.8)** — saving Settings should never be accidentally destructive. Safari's `type="password"` fields can't reliably re-populate from autofill, so an empty read is ambiguous. Explicit clear is Danger Zone → Reset browser.
- **Last-write-wins on sync conflicts** — single-user app; a toast warns when another browser has updated.
- **iPhone slim mode deferred indefinitely** — Tom's phone is inbound-link-driven.

---

# The Stack

- **Language(s):** HTML, CSS, JavaScript (vanilla, no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs:**
  - GitHub REST API (gist read/write) — `api.github.com`
  - Open-Meteo (free weather + geocoding, no key) — `api.open-meteo.com` + `geocoding-api.open-meteo.com`
  - CoinGecko (free crypto prices + search, no key) — `api.coingecko.com`
  - **Finnhub** (stock quotes + symbol search, free tier with key, 60 calls/min) — `finnhub.io/api/v1/quote` + `finnhub.io/api/v1/search` *(replaced Twelve Data in v1.9)*
  - Google Favicon service (best-effort bookmark icons) — `www.google.com/s2/favicons`
- **Storage:** One private GitHub gist on Tom's account (`TECGUNTHER`); per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub API key, theme choice, and a shared 5-minute stock-quote cache. (The legacy Twelve Data key is still read for backward-compat but unused.)
- **Hosting:** GitHub Pages (free static), public repo at `github.com/TECGUNTHER/startdashboard`

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (HTML + CSS + JS in one file, ~4,120 lines)
├── README.md                — Tom's own TECGUNTHER-specific setup walkthrough
├── INSTALL-FOR-OTHERS.md    — clean install guide for someone else to fork and run their own
├── PROJECT-HUB.html         — project dashboard for Tom
├── HANDOFF.md               — this file
├── docs/
│   ├── TECHNICAL.html       — architecture, decisions, file map, how-to-modify
│   └── USER-EXPERIENCE.html — features, screens, workflows, scope
└── plans/
    └── 2026-05-27-personal-dashboard-v1.md  — approved build plan (v1.0 scope)
```

---

# How to Run It

## Key URLs

- **Live dashboard:** https://tecgunther.github.io/startdashboard/
- **Code repo:** https://github.com/TECGUNTHER/startdashboard
- **GitHub Pages settings:** https://github.com/TECGUNTHER/startdashboard/settings/pages
- **GitHub tokens** (to revoke/regenerate): https://github.com/settings/tokens
- **Finnhub signup:** https://finnhub.io/register
- **Find your gist ID:** open the dashboard → Settings (⚙) → Sync → Gist ID

## Port: none (static page served by GitHub Pages)

## One-time setup (already done, but for reference)

```
cd projects/personal-dashboard
git init && git add . && git commit -m "Initial StartDashboard"
git branch -M main
git remote add origin https://github.com/TECGUNTHER/startdashboard.git
git push -u origin main
```

Then in GitHub: **Settings → Pages → Deploy from a branch → main / (root)**.

Generate a Personal Access Token at `github.com/settings/tokens` with `gist` read/write scope, and paste it into the dashboard's setup screen. For the stock widgets, generate a free Finnhub key at `finnhub.io/register` and paste it into Settings.

## To launch

```
open https://tecgunther.github.io/startdashboard/
```

For local-only preview, double-click `index.html`.

## Iteration loop (when making code changes)

```
cd ~/Documents/tom-workspace/projects/personal-dashboard
git add index.html
git commit -m "vX.Y: short description"
git push
```

Then hard-reload the dashboard (Safari: Option+Cmd+R, Chrome: Cmd+Shift+R). GitHub Pages takes ~30–60s to update.

## Troubleshooting (known issues)

- **"Bad credentials" on first run** → PAT wrong or missing `gist` scope. Regenerate.
- **Sync indicator turns red ⚠** → Click it to reload from gist. If still failing, token may have expired; reset this browser in Settings → Danger Zone.
- **Cross-browser changes not showing** → Reload the other browser. Background poll is 60s; manual reload is instant.
- **Bookmarklet popup blocked** → Allow popups for `tecgunther.github.io` in browser settings.
- **Weather "Could not load"** → Transient Open-Meteo issue, or invalid lat/lon coords.
- **Markets/Watchlist say "Add your Finnhub API key" (v1.9)** → Stock widgets use Finnhub now, not Twelve Data. Generate a free key at finnhub.io/register and paste it into Settings. The old Twelve Data key is ignored.
- **"Rate limit reached — prices refresh shortly" (v1.9)** → Every tracked symbol got rate-limited with nothing cached. The widget backs off and auto-refreshes once after 60s. No action needed (Finnhub free tier = 60 calls/min).
- **A single stock row shows "—" (v1.9)** → That one symbol's quote call failed (bad ticker or transient error). The rest of the widget still renders; the dash means "no data for this symbol," not a widget-wide failure.
- **Crypto widget hangs on "Loading…" for ~2 seconds (v1.8)** → CoinGecko's cold-start retry doing its job (one retry at 2s before a red error). Stocks no longer do this immediate retry as of v1.9.
- **Twelve Data key got wiped after saving Settings (v1.8)** → shouldn't happen; `saveSettings` no longer treats an empty key field as "clear." (Now applies to the Finnhub key field.)
- **(v1.6) Persona gradient still shows behind the light surface after toggling** → the persona-class wasn't removed before `bg-light-surface` was added. Fix is in `applyPersonaTheme()`.
- **(v1.7) Negative price renders with 4 decimals** → `formatPrice()` must branch on `Math.abs(n)`, not raw `n`.

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's own setup walkthrough (TECGUNTHER-specific). The top points third-party users at `INSTALL-FOR-OTHERS.md`.
- `INSTALL-FOR-OTHERS.md` — clean install guide for anyone Tom shares the project with: fork → enable GitHub Pages → generate their own PAT → run their own copy with their own private gist.
- `PROJECT-HUB.html` — visual dashboard for this project (Tom's home base)
- `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture, decisions, file map, how-to-modify
- `docs/USER-EXPERIENCE.html` — what the app does, plain English
- `plans/2026-05-27-personal-dashboard-v1.md` — the approved build plan (v1.0 scope; later versions tracked via session log)

---

# Open Items

- **(Low priority)** iPhone slim mode — deferred indefinitely (Tom's phone is inbound-link-driven, not outbound surfing).
- **(Question)** Mobile Safari drag-drop behavior on iPhone — only matters if Tom starts managing bookmarks on phone.
- **(Future feature)** Custom embed widget for pasting iframes (calendar, RSS, etc.) — only when Tom has a specific use case.

---

# Working with Tom (Behavioral Notes)

## Communication norms

- Plain English at all times. No jargon without translation.
- One question at a time. Don't stack three at once.
- Lead with the answer. He reads executive briefs all day.
- No filler enthusiasm. Skip "Great question!", "Happy to help!"
- Be honest about limits. If something isn't working or isn't built, say so directly.

## Project-specific quirks

- Single-file HTML by design — don't suggest splitting into modules unless there's a real reason.
- Workspace's persistent documents (PROJECT-HUB, MASTER-HUB, TECHNICAL, UX) use a light/restrained aesthetic. The dashboard *app itself* (`index.html`) is dark/modern/tech by default, with a v1.6 light-mode option that pulls from the same workspace palette. Don't mix the visual systems in either direction.
- Tom is the only user. No multi-user features, no sharing model.
- All data lives on Tom's GitHub account. Don't add third-party services without checking first.
- He prefers per-persona settings over global. **Exceptions:** theme (light/dark) and credentials (GitHub PAT, Finnhub key) are per-browser globals stored in localStorage, not synced through the gist.

## Multi-agent flow (as of v1.6)

- For major changes Tom expects the actual multi-agent flow: **Design Agent** plans, Tom approves, **Build Agent** subagent writes code and returns a standard Agent Report, **Documentation Agent** subagent updates docs from that report. v1.1–v1.5 were done by one Claude playing all roles; v1.6 onward use real subagents. Tom wants this to be the pattern going forward.

## What he's likely to want next

- More polish based on actual use (he iterates fast when he finds friction).
- Eventually: a Custom embed widget for arbitrary iframes (calendar, RSS).
- Maybe: more financial widgets (international markets, futures, FX) if he leans further into Investments.

---

# Deeper Reference

- **Architecture, dependencies, technical decisions:** `docs/TECHNICAL.html`
- **What the app does, screens, user workflows:** `docs/USER-EXPERIENCE.html`
- **Project state and history:** `PROJECT-HUB.html`
- **Build plans:** `plans/`
