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
**Current phase:** Complete (with v1.12 shipped)
**Last updated:** 2026-05-28
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that becomes Tom's home page across every browser and device. A tab bar at the top lets him switch between "personas" (Work, Personal, Investments, anything he creates). Each persona is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent color. Data lives in one private GitHub gist on his account, so opening the same URL in Chrome on his Mac and Safari on his iPhone shows the same view, synced within seconds. Default aesthetic is dark and modern (Linear/Vercel/Raycast as references); a v1.6 light-mode toggle covers daytime / bright-light use.

**Available widgets (per-persona, can be shown/hidden and reordered individually):**
- Clock (multi time zone, 12/24hr toggle)
- Weather (Open-Meteo, no key). **As of v1.12, each forecast day shows a condition icon + short label and a chance-of-rain percentage ("🌧 NN%"), alongside the high/low it already showed; the current-conditions line shows a matching icon.**
- To-do list
- Sticky notes
- Crypto (CoinGecko, no key — shows price + dollar + percent change)
- Markets (Finnhub — defaults to SPY/QQQ/DIA for S&P/Nasdaq/Dow; a single, fixed index-proxy card)
- Watchlists (Finnhub — Tom's custom stocks). **As of v1.10 there can be more than one, each with its own custom title.**

As of v1.10, the widget cards can be dragged to reorder, and any category or widget card can be given a custom border/glow color.

**Also: an "Ask Gemini" AI chat panel (v1.11).** This is a **global panel, not a per-persona widget**. A floating chat bubble in the bottom-right corner opens a slide-over panel where Tom can ask a question and get a concise, search-style answer without leaving the dashboard. It's available from every persona, is powered by the free Google Gemini API, and has an "Open full Gemini" link-out to his real account.

---

# Where We Are

## Most recent session (2026-05-28 — v1.12)

Shipped a small **weather forecast detail** enhancement.

- **Per-day condition icon + label.** Each forecast day now leads with a weather emoji from a new `weatherCodeIcon(code)` function (maps each WMO code → an icon: ☀️ clear, ⛅ partly cloudy, ☁️ overcast, 🌫️ fog, 🌧️ rain, ❄️ snow, ⛈️ thunder, etc.), plus the short text label it already used (`.d-cond`, still from `weatherCodeText`). The current-conditions line gained the matching icon (`.w-icon`).
- **Chance of rain per day.** The Open-Meteo daily request now also asks for `precipitation_probability_max` (alongside the existing `temperature_2m_max`/`temperature_2m_min`/`weather_code`), and each day shows a "🌧 NN%" line (`.d-rain`). Still no API key, still a 4-day forecast in Fahrenheit.
- **Null handling.** If a day has no rain-chance value, the rain line is simply omitted — never "null%".
- **Light, theme-safe styling.** New CSS for the icon and the two new lines; the forecast row now wraps and each day has a minimum width so the extra detail lays out cleanly in both light and dark.

No new dependencies; only `index.html` changed (~5,190 → ~5,235 lines). Only the Weather widget was touched.

## Sessions before that

- **v1.11 (2026-05-28, night):** Shipped a built-in **"Ask Gemini" AI chat panel** inside the dashboard — a bottom-right floating bubble opens a slide-over chat for concise, search-style answers. Powered by the free Google Gemini API, called **browser-direct with no backend** (`sendChatMessage` POSTs to `generativelanguage.googleapis.com/v1beta/models/<model>:generateContent` with the key in an `x-goog-api-key` header; default model `gemini-2.5-flash`, overridable in Settings). A `GEMINI_SYSTEM_PROMPT` primes short answers; the last ~10 turns are sent for context. Chat history is per-device in `sd_chat_history` (localStorage, capped at 50), **not** synced through the gist. Settings gained a Gemini key + optional model field; errors render as friendly in-panel notes (never stack traces) with an "Open full Gemini" link-out that works even with no key. **One empirical unknown:** the Build Agent couldn't fully confirm the browser-direct call avoids a CORS block — the fallback makes the worst case graceful, and Tom's first real query settles it.
- **v1.10 (2026-05-28, late evening):** Biggest release since v1.0. Three per-persona, gist-synced features: (1) **drag widget cards to reorder** (each card header has a ⠿ grip; order stored in a new `widgetOrder` array); (2) **multiple, renamable watchlists** (the old single `widgets.watchlist` became `persona.watchlists`, an array of `{id, title, symbols[]}`, each keyed by a composite `watchlist:<id>`; "+ Add watchlist" in Settings); (3) **per-card border/glow color** (`category.color` + a `persona.widgetColors` map; picker with swatches + custom color + "Default (theme)" reset). The widget row is now built dynamically by `renderWidgets`. Migration is invisible (`ensurePersonaSchema` + `migrateWidgetOrder`). Only `index.html` changed.
- **v1.9 (2026-05-28, later evening):** Migrated stock data from Twelve Data to **Finnhub** (Twelve Data's free tier was 8 calls/min; Tom's ~12 symbols hit chronic HTTP 429s — Finnhub is 60/min, also free). Added a shared 5-minute localStorage stock-quote cache (`sd_quote_cache`). On a rate limit the widget backs off 60s. Overhauled drag-to-reorder with visible ⠿ grip handles. **Tom must paste a free Finnhub key (finnhub.io/register) into Settings.**

## Immediate next step

1. **Push v1.12 to GitHub** (single-file commit of `index.html`), wait ~60s for Pages, hard-reload Safari (Opt+Cmd+R) and Chrome (Cmd+Shift+R).
2. **Verify the weather detail:** open a persona with the Weather widget; each forecast day should show a condition icon + short label + high/low + a "🌧 NN%" chance-of-rain line, and the current line should show a matching icon.
3. **Still pending from v1.11:** if not done yet, generate a free Gemini key at https://aistudio.google.com/app/apikey (no credit card), paste it into **Settings → Gemini API key**, click the floating bubble, and ask a question to confirm the chat works. That first real query also confirms the browser-direct CORS path; if blocked, the friendly "Couldn't reach Gemini" note + "Open full Gemini" link covers it (no fix needed).

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend** — durable, free, no third-party service to outlive Tom's GitHub account. Chosen over Supabase / jsonbin / iCloud-folder sync (Safari can't write back to local files).
- **Per-browser Personal Access Tokens stored in localStorage** — avoids OAuth in a static page. Each browser gets its own token; losing a device means revoking one token, not all.
- **Per-persona widgets, backgrounds, accent colors, visibility, order, and per-card colors** — the whole point of personas is mental separation. Each persona is fully independent.
- **Per-device default persona** (localStorage) — Tom's work Mac opens to Work, iPhone opens to Personal. Not synced through the gist.
- **Per-device theme choice** (localStorage, v1.6) — light vs dark is a per-environment preference, not a per-account one.
- **Weather widget enhanced incrementally, not redesigned (v1.12)** — added one new Open-Meteo daily field (`precipitation_probability_max`) plus a `weatherCodeIcon` map, rendered into the existing per-day layout. Null rain values drop the line rather than show "null%". Kept entirely theme-token-styled so light and dark both work, and touched only the Weather widget.
- **Ask Gemini uses Gemini Flash, not Claude or OpenAI (v1.11)** — it's free with no credit card, and it's the same model family behind the Google Search AI answers Tom already reaches for, so it fits the "fast quick answer" use case. Claude/OpenAI have no free tier. The model name is a Settings override field defaulting to `gemini-2.5-flash`, so a Google rename doesn't require a code change.
- **Browser-direct Gemini call with a graceful link-out fallback, not a backend (v1.11)** — preserves the zero-backend, single-file architecture. If the browser blocks the call (CORS), the panel shows a friendly message plus the "Open full Gemini" link rather than breaking. CORS on the native `:generateContent` endpoint is unconfirmed until Tom's first real query; the fallback makes the worst case graceful.
- **Chat history per-device in localStorage, not synced to the gist (v1.11)** — transcripts would bloat the synced data on every save, and the in-progress conversation is naturally a per-device thing (same reasoning as the per-device theme and default persona).
- **Quick-answer system prompt (v1.11)** — `GEMINI_SYSTEM_PROMPT` tunes Gemini toward short, direct, search-style replies; only the last ~10 turns are sent for follow-up context.
- **Multiple watchlists via an array + composite `watchlist:<id>` keys (v1.10)** — the single `widgets.watchlist` became `persona.watchlists`. Each watchlist is addressed everywhere by a composite key so it participates in the same reorder/hide/color/config machinery as a built-in widget, without special-casing throughout the code.
- **Widget row converted from static HTML to fully dynamic rendering (v1.10)** — `renderWidgets` rebuilds every card from `persona.widgetOrder`. Required to support both card reorder and multiple watchlist instances. Side effects: the Notes textarea is regenerated each render (keeps its `notes-textarea` id), and closing a widget config does a full `render()` so edits propagate at once.
- **Markets kept as a singleton, not folded into watchlists (v1.10)** — Markets is the fixed index-proxy view (SPY/QQQ/DIA); watchlists are user-curated. Different purposes; combining them would muddy both.
- **Per-card color stored in `persona.widgetColors` map + `category.color` (v1.10)** — all widget colors in one per-persona map keyed by widget key (incl. composite watchlist keys); categories use `category.color`. Absent/null = inherit accent. Glow is a static `box-shadow` — no animation.
- **Finnhub for stocks, replacing Twelve Data (v1.9)** — Twelve Data's free tier was 8 calls/min; Tom's ~12 symbols blew past it (chronic HTTP 429s). Finnhub's free tier is 60/min (also free, no card). Both REST APIs, so a swap with no new dependency. CoinGecko (crypto, no key) unchanged.
- **Shared 5-minute localStorage quote cache (v1.9)** — replaced the old per-tab in-memory cache. Multiple tabs/reloads were multiplying API calls; a shared 5-min cache means only stale/missing symbols hit the network. (Stock quotes only, not crypto.)
- **On a 429, back off 60 seconds instead of retrying immediately (v1.9)** — v1.8's 2-second retry made rate-limiting worse. Per-symbol failures degrade to "—".
- **Visible ⠿ grip handles for drag-to-reorder (v1.9)** — bare HTML5 drag had no affordance. Explicit grips on categories, bookmarks, config-list items, and (v1.10) widget card headers; a drop-indicator line shows where it'll land.
- **Light mode replaces the persona background with one clean light surface (v1.6)** — not per-persona light gradients (would double config burden). Accent color preserved in both themes.
- **CSS palette tokenized for theming (v1.6)** — the whole `body.light-mode` skin is one override block.
- **Bookmarklet over a browser extension** — one JS snippet works everywhere; no per-browser build burden.
- **Backgrounds: CSS-only presets + image URL only (no upload)** — image upload would bloat the gist with base64 on every save.
- **API key per-browser in localStorage (parallel to GitHub PAT)** — rate limits are per-key; don't sync credentials through a gist. Applies to the Finnhub and Gemini keys too.
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
  - Open-Meteo (free weather + geocoding, no key) — `api.open-meteo.com` + `geocoding-api.open-meteo.com`. The Weather widget's daily request asks for `temperature_2m_max`, `temperature_2m_min`, `weather_code`, and (as of v1.12) `precipitation_probability_max`.
  - CoinGecko (free crypto prices + search, no key) — `api.coingecko.com`
  - **Finnhub** (stock quotes + symbol search, free tier with key, 60 calls/min) — `finnhub.io/api/v1/quote` + `finnhub.io/api/v1/search` *(replaced Twelve Data in v1.9)*
  - **Google Gemini** (AI chat, free tier with key, browser-direct, no backend) — `generativelanguage.googleapis.com/v1beta/models/<model>:generateContent`, default model `gemini-2.5-flash` *(since v1.11)*
  - Google Favicon service (best-effort bookmark icons) — `www.google.com/s2/favicons`
- **Storage:** One private GitHub gist on Tom's account (`TECGUNTHER`); per-browser localStorage holds the GitHub PAT, gist ID, default persona, Finnhub API key, Gemini API key + model + chat history, theme choice, and a shared 5-minute stock-quote cache. (The legacy Twelve Data key is still read for backward-compat but unused.)
- **Hosting:** GitHub Pages (free static), public repo at `github.com/TECGUNTHER/startdashboard`

## Data model notes (v1.10)

Each persona carries: `categories` (each with an optional `color`), `widgets` (clock, weather, todos, notes, crypto, markets — the old `watchlist` key is gone), `watchlists` (array of `{ id, title, symbols[] }`), `widgetOrder` (array of widget keys incl. composite `watchlist:<id>` keys), `widgetColors` (map of widgetKey → hex), and `hiddenWidgets` (also holds composite `watchlist:<id>` keys). The Gemini chat is global, not per-persona, and its history lives outside the gist in localStorage.

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html               — the entire app (HTML + CSS + JS in one file, ~5,235 lines)
├── README.md                — Tom's own TECGUNTHER-specific setup walkthrough
├── INSTALL-FOR-OTHERS.md    — clean install guide for someone else to fork and run their own
├── PROJECT-HUB.html         — project dashboard for Tom
├── HANDOFF.md               — this file
├── docs/
│   ├── TECHNICAL.html       — architecture, decisions, file map, how-to-modify
│   └── USER-EXPERIENCE.html — features, screens, workflows, scope
└── plans/
    ├── 2026-05-27-personal-dashboard-v1.md            — original v1.0 build plan
    ├── 2026-05-28-v1.10-widgets-watchlists-colors.md  — the v1.10 plan
    ├── 2026-05-28-v1.11-gemini-chat-panel.md          — the v1.11 plan
    └── 2026-05-28-v1.12-weather-detail.md             — the v1.12 plan (this session)
```

---

# How to Run It

## Key URLs

- **Live dashboard:** https://tecgunther.github.io/startdashboard/
- **Code repo:** https://github.com/TECGUNTHER/startdashboard
- **GitHub Pages settings:** https://github.com/TECGUNTHER/startdashboard/settings/pages
- **GitHub tokens** (to revoke/regenerate): https://github.com/settings/tokens
- **Finnhub signup:** https://finnhub.io/register
- **Gemini API key (free, no card):** https://aistudio.google.com/app/apikey
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

Generate a Personal Access Token at `github.com/settings/tokens` with `gist` read/write scope, and paste it into the dashboard's setup screen. For the stock widgets, generate a free Finnhub key at `finnhub.io/register` and paste it into Settings. For the Ask Gemini chat, generate a free Gemini key at `aistudio.google.com/app/apikey` and paste it into Settings → Gemini API key.

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
- **A weather day shows no chance-of-rain line (v1.12)** → Expected. Open-Meteo returned no `precipitation_probability_max` for that day, so the line is omitted rather than showing "null%".
- **Markets/Watchlists say "Add your Finnhub API key" (v1.9)** → Stock widgets use Finnhub now, not Twelve Data. Generate a free key at finnhub.io/register and paste it into Settings. The old Twelve Data key is ignored.
- **"Rate limit reached — prices refresh shortly" (v1.9)** → Every tracked symbol got rate-limited with nothing cached. The widget backs off and auto-refreshes once after 60s. No action needed (Finnhub free tier = 60 calls/min).
- **A single stock row shows "—" (v1.9)** → That one symbol's quote call failed (bad ticker or transient error). The rest of the widget still renders; the dash means "no data for this symbol," not a widget-wide failure.
- **Crypto widget hangs on "Loading…" for ~2 seconds (v1.8)** → CoinGecko's cold-start retry doing its job (one retry at 2s before a red error). Stocks no longer do this immediate retry as of v1.9.
- **An old single watchlist seems to have "moved" after the v1.10 update** → expected. v1.10 turned the one watchlist into a list of watchlists; your old one is now the first entry (titled "Watchlist") with its symbols intact. Add more via Settings → "+ Add watchlist."
- **Ask Gemini chat says "Add your free Gemini API key" (v1.11)** → No key set. Generate one free at aistudio.google.com/app/apikey and paste it into Settings → Gemini API key. The "Open full Gemini" link-out works even without a key.
- **Chat says "Couldn't reach Gemini" (v1.11)** → A network hiccup, a rate limit, or the browser blocked the direct call (CORS). Use the "Open full Gemini" link in the note to continue in your real Gemini account.

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
- `plans/2026-05-28-v1.10-widgets-watchlists-colors.md` — the approved v1.10 plan
- `plans/2026-05-28-v1.11-gemini-chat-panel.md` — the approved v1.11 plan
- `plans/2026-05-28-v1.12-weather-detail.md` — the approved v1.12 plan (weather forecast detail)

---

# Open Items

- **(Question)** Confirm the Gemini browser-direct call works on the first real query (v1.11). If the browser blocks it (CORS), the "Open full Gemini" link-out fallback already covers it; no backend planned. (added 2026-05-28)
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
- All data lives on Tom's GitHub account. Don't add third-party services without checking first. (v1.11's Gemini chat adds one external service — Google's free Gemini API — called browser-direct with no backend; the key is per-browser like the others.)
- He prefers per-persona settings over global. **Exceptions:** theme (light/dark), credentials (GitHub PAT, Finnhub key, Gemini key), and the Ask Gemini chat + its history are per-browser/per-device globals stored in localStorage, not synced through the gist.
- Markets is intentionally a single, fixed index-proxy card (SPY/QQQ/DIA) — don't make it renamable or multiple. Watchlists are the renamable/multiple concept (v1.10).
- The Ask Gemini chat is a global panel, not a widget — it doesn't go through the persona/widget order/visibility machinery, and it's a fresh assistant with no link to Tom's personal Gemini account history.

## Multi-agent flow (as of v1.6)

- For major changes Tom expects the actual multi-agent flow: **Design Agent** plans, Tom approves, **Build Agent** subagent writes code and returns a standard Agent Report, **Documentation Agent** subagent updates docs from that report. v1.1–v1.5 were done by one Claude playing all roles; v1.6 onward use real subagents. v1.10, v1.11, and v1.12 followed this flow. Tom wants this to be the pattern going forward.

## What he's likely to want next

- More polish based on actual use (he iterates fast when he finds friction).
- Pushing and verifying v1.12's weather detail; confirming the Gemini chat works on first use (the one open empirical question).
- Possibly the Crypto cache (see Open Items) if reorder-driven re-fetching gets noticeable.
- Eventually: a Custom embed widget for arbitrary iframes (calendar, RSS).
- Maybe: more financial widgets (international markets, futures, FX) if he leans further into Investments.

---

# Deeper Reference

- **Architecture, dependencies, technical decisions:** `docs/TECHNICAL.html`
- **What the app does, screens, user workflows:** `docs/USER-EXPERIENCE.html`
- **Project state and history:** `PROJECT-HUB.html`
- **Build plans:** `plans/`
