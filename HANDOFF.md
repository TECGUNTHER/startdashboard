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
**Current phase:** Complete (with v1.10 shipped)
**Last updated:** 2026-05-28
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that becomes Tom's home page across every browser and device. A tab bar at the top lets him switch between "personas" (Work, Personal, Investments, anything he creates). Each persona is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent color. Data lives in one private GitHub gist on his account, so opening the same URL in Chrome on his Mac and Safari on his iPhone shows the same view, synced within seconds. Default aesthetic is dark and modern (Linear/Vercel/Raycast as references); a v1.6 light-mode toggle covers daytime / bright-light use.

**Available widgets (per-persona, can be shown/hidden and reordered individually):**
- Clock (multi time zone, 12/24hr toggle)
- Weather (Open-Meteo, no key)
- To-do list
- Sticky notes
- Crypto (CoinGecko, no key — shows price + dollar + percent change)
- Markets (Finnhub — defaults to SPY/QQQ/DIA for S&P/Nasdaq/Dow; a single, fixed index-proxy card)
- Watchlists (Finnhub — Tom's custom stocks). **As of v1.10 there can be more than one, each with its own custom title.**

As of v1.10, the widget cards can be dragged to reorder, and any category or widget card can be given a custom border/glow color.

---

# Where We Are

## Most recent session (2026-05-28 — v1.10)

The biggest release since v1.0. Three new features, all per-persona and synced to the gist:

- **Drag widget cards to reorder them.** Each widget card header has a ⠿ grip; dragging is horizontal and position-aware. Order is stored in a new per-persona `widgetOrder` array and persists across reloads/browsers. Helpers: `buildWidgetCard`, `moveWidget`. The widget row, previously static HTML, is now built dynamically by `renderWidgets` from `widgetOrder` (this is the structural change that makes reorder and multiple watchlists possible).
- **Multiple, renamable watchlists.** The old single `widgets.watchlist` became `persona.watchlists` — an array of `{ id, title, symbols[] }`, each rendered as its own card. A "+ Add watchlist" button in Settings (below the widget show/hide toggles) creates more. Each watchlist's ⚙ config gained a title rename (live as you type), the existing symbol management, and a delete-this-watchlist button (confirm-gated). Each watchlist card is keyed by a **composite key** `watchlist:<id>` used everywhere a widget is referenced. Markets stays a single, un-renamable index-proxy card. Helpers: `addWatchlist`, `deleteWatchlist`, `renderStockWidgetFor`.
- **Per-card border/glow color.** Any category or widget card can carry a custom hex color (`category.color` for categories, a new `persona.widgetColors` map for widgets; absent/null = inherit the persona accent). A colored card gets a colored border + soft static glow. Categories: "..." menu → "Card color…" → popover. Widgets: each ⚙ gear has a "Card color" section; To-do and Notes (no config before) got a new color-only gear. The picker has preset swatches, a custom color picker, and a "Default (theme)" reset. Helpers: `applyCardColor`, `buildCardColorControl`, `openCategoryColorPopover`.

Schema migration is invisible: `ensurePersonaSchema` + `migrateWidgetOrder` convert the legacy singleton watchlist to one titled "Watchlist" (symbols preserved), give new personas one empty "Watchlist," remap any legacy bare `'watchlist'` key to the composite key(s), append missing built-ins/new watchlists to `widgetOrder`, and drop stale keys. Verified against pre-widget-order, mid-version, and empty personas; an old-shape persona never crashes. No new dependencies; only `index.html` changed (~4,120 → ~4,610 lines).

## Sessions before that

- **v1.9 (2026-05-28, later evening):** Migrated stock data from Twelve Data to **Finnhub** (Twelve Data's free tier was 8 calls/min; Tom's ~12 symbols hit chronic HTTP 429s — Finnhub is 60/min, also free). Added a shared 5-minute localStorage stock-quote cache (`sd_quote_cache`) across all tabs/reloads. On a rate limit, the widget backs off 60s instead of retrying immediately. Overhauled drag-to-reorder with visible ⠿ grip handles on categories, bookmarks, and config lists. **Tom must paste a free Finnhub key (finnhub.io/register) into Settings.**
- **v1.8 hotfix (2026-05-28, evening):** Empty Settings key field no longer wipes the stored API key. Finance widgets gained a one-shot cold-start retry at 2s. Active persona tab restyled to a soft-accent fill + accent outline + bold text. New `INSTALL-FOR-OTHERS.md` guide. (v1.9 dropped the stock-widget retry; crypto kept it.)
- **v1.7 hotfix (2026-05-28, late afternoon):** Negative finance prices fixed to 2 decimals (`formatPrice()` branches on `Math.abs()` and re-applies sign). Bookmark description popover spacing bumped to 28px to clear Safari's native ellipsis tooltip. Active persona tab made solid-accent to fix "no tab selected on first frame" (softened in v1.8).

## Immediate next step

1. **Push v1.10 to GitHub** (single-file commit of `index.html`), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Verify all three features:** (a) drag a widget card by its ⠿ grip and confirm the order sticks after reload; (b) add a second watchlist from Settings, rename it, add a couple of stocks, then delete it; (c) set a custom card color on a category and a widget, then reset one to "Default (theme)." Also confirm the existing single watchlist migrated with its symbols intact and Markets still works.
3. **(If a Finnhub key isn't already set)** generate a free key at finnhub.io/register and paste it into Settings — Markets/Watchlists need it.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend** — durable, free, no third-party service to outlive Tom's GitHub account. Chosen over Supabase / jsonbin / iCloud-folder sync (Safari can't write back to local files).
- **Per-browser Personal Access Tokens stored in localStorage** — avoids OAuth in a static page. Each browser gets its own token; losing a device means revoking one token, not all.
- **Per-persona widgets, backgrounds, accent colors, visibility, order, and per-card colors** — the whole point of personas is mental separation. Each persona is fully independent.
- **Per-device default persona** (localStorage) — Tom's work Mac opens to Work, iPhone opens to Personal. Not synced through the gist.
- **Per-device theme choice** (localStorage, v1.6) — light vs dark is a per-environment preference, not a per-account one.
- **Multiple watchlists via an array + composite `watchlist:<id>` keys (v1.10)** — the single `widgets.watchlist` became `persona.watchlists` (an array). Each watchlist is addressed everywhere by a composite key so it participates in the same reorder/hide/color/config machinery as a built-in widget, without special-casing it throughout the code.
- **Widget row converted from static HTML to fully dynamic rendering (v1.10)** — `renderWidgets` rebuilds every card from `persona.widgetOrder`. Required to support both card reorder and multiple watchlist instances. Side effects: the Notes textarea is regenerated each render (keeps its `notes-textarea` id), and closing a widget config now does a full `render()` so title/color/symbol edits propagate at once.
- **Markets kept as a singleton, not folded into watchlists (v1.10)** — Markets is the fixed index-proxy view (SPY/QQQ/DIA); watchlists are user-curated lists. Different purposes; combining them would muddy both.
- **Per-card color stored in `persona.widgetColors` map + `category.color` (v1.10)** — all widget colors live in one per-persona map keyed by widget key (including composite watchlist keys); categories use `category.color`. Absent/null = inherit accent. One predictable location for the picker, the reset, and migration. Glow is a static `box-shadow` — deliberately no animation.
- **Finnhub for stocks, replacing Twelve Data (v1.9)** — Twelve Data's free tier was 8 calls/min; Tom's ~12 symbols blew past it and hit chronic HTTP 429s. Finnhub's free tier is 60/min (also free, no credit card). Both REST APIs, so this was a swap with no new dependency. CoinGecko (crypto, no key) is unchanged.
- **Shared 5-minute localStorage quote cache (v1.9)** — replaced the old per-tab in-memory cache. Multiple tabs/reloads were multiplying API calls toward the rate limit; a shared 5-min cache means only stale/missing symbols hit the network. (Covers stock quotes only, not crypto.)
- **On a 429, back off 60 seconds instead of retrying immediately (v1.9)** — v1.8's 2-second retry made rate-limiting worse. One delayed refresh recovers cleanly. Per-symbol failures degrade to "—" instead of retrying.
- **Visible ⠿ grip handles for drag-to-reorder (v1.9)** — bare HTML5 drag had no affordance; Tom couldn't find what to grab. An explicit grip on categories, bookmarks, and config-list items (and, in v1.10, widget card headers) makes the target obvious; a drop-indicator line shows where it'll land.
- **Light mode replaces the persona background with a single clean light surface** (v1.6) — not per-persona light gradients (would double config burden). Persona accent color is preserved in both themes.
- **CSS palette tokenized for theming** (v1.6) — the whole `body.light-mode` skin is one override block. Future themes are cheap.
- **Bookmarklet over a browser extension** — one JS snippet works everywhere; no per-browser build burden.
- **Backgrounds: CSS-only presets + image URL only (no upload)** — image upload would bloat the gist with base64 on every save.
- **API key per-browser in localStorage (parallel to GitHub PAT)** — rate limits are per-key; don't sync credentials through a gist.
- **Markets uses ETF proxies (SPY, QQQ, DIA)** for major indices since the actual indices need higher API tiers.
- **Clock zones stored as {name, tz} objects (since v1.3)** — allows multiple cities in the same timezone to render as separate rows.
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

## Data model notes (v1.10)

Each persona now carries: `categories` (each with an optional `color`), `widgets` (clock, weather, todos, notes, crypto, markets — the old `watchlist` key is gone), `watchlists` (array of `{ id, title, symbols[] }`), `widgetOrder` (array of widget keys incl. composite `watchlist:<id>` keys), `widgetColors` (map of widgetKey → hex), and `hiddenWidgets` (now also holds composite `watchlist:<id>` keys).

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (HTML + CSS + JS in one file, ~4,610 lines)
├── README.md                — Tom's own TECGUNTHER-specific setup walkthrough
├── INSTALL-FOR-OTHERS.md    — clean install guide for someone else to fork and run their own
├── PROJECT-HUB.html         — project dashboard for Tom
├── HANDOFF.md               — this file
├── docs/
│   ├── TECHNICAL.html       — architecture, decisions, file map, how-to-modify
│   └── USER-EXPERIENCE.html — features, screens, workflows, scope
└── plans/
    ├── 2026-05-27-personal-dashboard-v1.md            — original v1.0 build plan
    └── 2026-05-28-v1.10-widgets-watchlists-colors.md  — the v1.10 plan (this session)
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
- **Markets/Watchlists say "Add your Finnhub API key" (v1.9)** → Stock widgets use Finnhub now, not Twelve Data. Generate a free key at finnhub.io/register and paste it into Settings. The old Twelve Data key is ignored.
- **"Rate limit reached — prices refresh shortly" (v1.9)** → Every tracked symbol got rate-limited with nothing cached. The widget backs off and auto-refreshes once after 60s. No action needed (Finnhub free tier = 60 calls/min).
- **A single stock row shows "—" (v1.9)** → That one symbol's quote call failed (bad ticker or transient error). The rest of the widget still renders; the dash means "no data for this symbol," not a widget-wide failure.
- **Crypto widget hangs on "Loading…" for ~2 seconds (v1.8)** → CoinGecko's cold-start retry doing its job (one retry at 2s before a red error). Stocks no longer do this immediate retry as of v1.9.
- **An old single watchlist seems to have "moved" after the v1.10 update** → expected. v1.10 turned the one watchlist into a list of watchlists; your old one is now the first entry (titled "Watchlist") with its symbols intact. Add more via Settings → "+ Add watchlist."

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's own setup walkthrough (TECGUNTHER-specific). The top points third-party users at `INSTALL-FOR-OTHERS.md`.
- `INSTALL-FOR-OTHERS.md` — clean install guide for anyone Tom shares the project with: fork → enable GitHub Pages → generate their own PAT → run their own copy with their own private gist.
- `PROJECT-HUB.html` — visual dashboard for this project (Tom's home base)
- `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture, decisions, file map, how-to-modify
- `docs/USER-EXPERIENCE.html` — what the app does, plain English
- `plans/2026-05-27-personal-dashboard-v1.md` — the approved v1.0 build plan
- `plans/2026-05-28-v1.10-widgets-watchlists-colors.md` — the approved v1.10 plan (widget reorder, multiple watchlists, per-card colors)

---

# Open Items

- **(Feature)** Cache the Crypto widget's price fetches — it re-fetches CoinGecko on every full render, and v1.10's drag-to-reorder triggers more renders. Add a shared cache like the stock widgets' 5-minute one if rate-limiting appears. (added 2026-05-28)
- **(Low priority)** iPhone slim mode — deferred indefinitely (Tom's phone is inbound-link-driven, not outbound surfing). (added 2026-05-27)
- **(Question)** Mobile Safari drag-drop behavior on iPhone — only matters if Tom starts managing bookmarks on phone. (added 2026-05-27)
- **(Future feature)** Custom embed widget for pasting iframes (calendar, RSS, etc.) — only when Tom has a specific use case. (added 2026-05-27)

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
- Markets is intentionally a single, fixed index-proxy card (SPY/QQQ/DIA) — don't make it renamable or multiple. Watchlists are the renamable/multiple concept (v1.10).

## Multi-agent flow (as of v1.6)

- For major changes Tom expects the actual multi-agent flow: **Design Agent** plans, Tom approves, **Build Agent** subagent writes code and returns a standard Agent Report, **Documentation Agent** subagent updates docs from that report. v1.1–v1.5 were done by one Claude playing all roles; v1.6 onward use real subagents. v1.10 followed this flow. Tom wants this to be the pattern going forward.

## What he's likely to want next

- More polish based on actual use (he iterates fast when he finds friction).
- Possibly the Crypto cache (see Open Items) if reorder-driven re-fetching gets noticeable.
- Eventually: a Custom embed widget for arbitrary iframes (calendar, RSS).
- Maybe: more financial widgets (international markets, futures, FX) if he leans further into Investments.

---

# Deeper Reference

- **Architecture, dependencies, technical decisions:** `docs/TECHNICAL.html`
- **What the app does, screens, user workflows:** `docs/USER-EXPERIENCE.html`
- **Project state and history:** `PROJECT-HUB.html`
- **Build plans:** `plans/`
