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
- For major changes, Tom now expects the **multi-agent flow** to run for real: a Build Agent subagent for code, a Documentation Agent subagent for docs. Not one chat playing every role.

---

# The Project

**Name:** StartDashboard
**Folder:** `projects/personal-dashboard/` in Tom's `tom-workspace`
**Current status:** Complete (in active use; iterating on polish)
**Current phase:** Complete (with v1.6 polish round shipped)
**Last updated:** 2026-05-28
**Live URL:** https://tecgunther.github.io/startdashboard/
**Code repo:** https://github.com/TECGUNTHER/startdashboard
**In use on:** Safari (Mac), Chrome (Mac), Safari (iPhone)

**What it does (plain English):**
A single-file web app that becomes Tom's home page across every browser and device. A tab bar at the top lets him switch between "personas" (Work, Personal, Investments, anything he creates). Each persona is its own complete dashboard with its own bookmarks (in categories), widgets, background, and accent color. Data lives in one private GitHub gist on his account, so opening the same URL in Chrome on his Mac and Safari on his iPhone shows the same view, synced within seconds. Default aesthetic is dark and modern (Linear/Vercel/Raycast as references); v1.6 added a light-mode toggle for eye comfort.

**Available widgets (per-persona, can be shown or hidden individually):**
- Clock (multi time zone, 12/24hr toggle)
- Weather (Open-Meteo, no key)
- To-do list
- Sticky notes
- Crypto (CoinGecko, no key — shows price + dollar + percent change)
- Markets (Twelve Data — defaults to SPY/QQQ/DIA for S&P/Nasdaq/Dow)
- Watchlist (Twelve Data — Tom's custom stocks)

---

# Where We Are

## Most recent session (2026-05-28 — v1.6)

A polish round driven by real-use friction Tom logged after a day of v1.5 use. First build to run the actual multi-agent flow: a Build Agent subagent did the code, a Documentation Agent subagent (me) did the docs.

- **Light/dark theme toggle** added to the header. Sun ☀ in dark mode, moon 🌙 in light mode. Click flips the whole UI; persists in `localStorage.sd_theme` per browser. Light mode replaces the persona's gradient background with a clean light surface; the persona's accent color is preserved.
- **Finance prices trimmed to 2 decimals.** Crypto, Markets, and Watchlist all show 2-decimal prices for anything ≥ $0.01. Sub-cent altcoins still show 4 decimals so they don't display as `$0.00`.
- **Bookmark description popover no longer fights the browser URL tooltip.** Removed the `title` attribute from bookmark links; the URL now lives inside the popover on a small monospace footer line below the description. Hovering a bookmark with no description still shows a tiny popover with just the URL.
- **"Default persona on this browser/device"** surfaced as a focal block at the top of Settings (with a 🖥 icon and explanatory subtext). The per-device default has always existed; Tom just never saw it.
- **CSS palette tokenized.** Introduced `--modal-bg`, `--input-bg`, `--popover-bg`, `--on-accent` tokens. The entire light-mode skin is one `body.light-mode` override block — no per-component overrides.

## Sessions before that

- **v1.5 (earlier 2026-05-28):** Bookmark description redesign (hover popover, multi-line edit modal), dollar + percent change on Crypto/Markets/Watchlist, Twelve Data API key inline walkthrough, full doc refresh (HANDOFF/TECHNICAL/UX were stale at v1.0).
- **v1.4 (2026-05-28):** Per-persona 12hr/24hr clock toggle, bookmark descriptions (always-visible inline, later superseded in v1.5), Settings "Danger zone" block.
- **v1.3 (2026-05-28):** Background CSS specificity hotfix (gradients never rendered from v1.0 through v1.2), clock zones changed from string[] to {name, tz}[], removed duplicate Settings UI, `saveSettings` flush-immediately.

## Immediate next step

Tom pushes v1.6 to GitHub, hard-reloads Safari + Chrome, smoke-tests the four user-facing changes (2-decimal prices → popover URL footer → Settings default-persona block at top → theme toggle persists per browser). Then uses it for a few days before the next iteration.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend** — durable, free, no third-party service to outlive Tom's GitHub account. Chosen over Supabase / jsonbin / iCloud-folder sync (Safari can't write back to local files).
- **Per-browser Personal Access Tokens stored in localStorage** — avoids OAuth in a static page. Each browser gets its own token; losing a device means revoking one token, not all.
- **Per-persona widgets, backgrounds, accent colors, and visibility** — the whole point of personas is mental separation. Each persona is fully independent.
- **Per-device default persona** (localStorage) — Tom's work Mac opens to Work, iPhone opens to Personal. Not synced through the gist.
- **Per-device theme choice** (localStorage, v1.6) — same pattern as the default persona and the GitHub PAT. Light vs dark is a per-environment preference (laptop in bright sun vs dark study), not a per-account one.
- **Light mode replaces the persona background with a single clean light surface** (v1.6) — not per-persona light gradients. The per-persona gradient system was designed for dark; making each persona also pick a light gradient would double Tom's config burden for marginal benefit. The persona accent color is preserved in both themes.
- **CSS palette tokenized for theming** (v1.6) — added `--modal-bg`, `--input-bg`, `--popover-bg`, `--on-accent` tokens. The whole `body.light-mode` skin is one override block instead of dozens of per-component overrides. Future themes are cheap to add.
- **Light palette pulled from `agent-system/agents/briefs/hub-design-brief.md`** (v1.6) — keeps the dashboard's light mode visually consistent with the rest of Tom's workspace documents (PROJECT-HUB, MASTER-HUB, TECHNICAL, UX).
- **Bookmarklet over a browser extension** — one JavaScript snippet that works everywhere; no per-browser build burden.
- **Backgrounds: CSS-only presets + image URL only (no upload)** — image upload would bloat the gist with base64 on every save.
- **Twelve Data for stocks (free tier with API key)**, **CoinGecko for crypto (free, no key)** — picked over Yahoo Finance because Yahoo's chart endpoint blocks browser requests via CORS.
- **API key per-browser in localStorage (parallel to GitHub PAT)** — rate limits are per-key; don't sync credentials through a gist.
- **Markets uses ETF proxies (SPY, QQQ, DIA)** for major indices since the actual indices (^GSPC etc.) need higher Twelve Data tiers.
- **Clock zones stored as {name, tz} objects (since v1.3)** — allows multiple cities in the same timezone (Atlanta + NYC both EST) to render as separate rows.
- **Bookmark descriptions as hover popover (since v1.5)** — keeps the row clean at rest; full text appears with a 0.5s delay so it doesn't pop on every mouseover.
- **Bookmark URL shown inside the description popover, not in a `title` tooltip** (v1.6) — the browser's native URL tooltip was overlapping/blocking the description popover. Removing `title` and showing the URL on a small monospace footer line below the description gives one cohesive hover artifact instead of two competing ones.
- **`formatPrice` shows 2 decimals everywhere ≥ $0.01, 4 decimals below** (v1.6) — Tom wants prices that look like prices, not scientific notation. Sub-cent visibility is preserved for tiny altcoins.
- **Last-write-wins on sync conflicts** — single-user app; full conflict resolution is overkill. A toast warns when another browser has updated.
- **iPhone slim mode deferred indefinitely** — Tom's phone is inbound-link-driven; he doesn't browse outbound on it.

---

# The Stack

- **Language(s):** HTML, CSS, JavaScript (vanilla, no framework, no build step)
- **Frameworks / Libraries:** None
- **External APIs:**
  - GitHub REST API (gist read/write) — `api.github.com`
  - Open-Meteo (free weather + geocoding, no key) — `api.open-meteo.com` + `geocoding-api.open-meteo.com`
  - CoinGecko (free crypto prices + search, no key) — `api.coingecko.com`
  - Twelve Data (stock quotes + symbol search, free tier with key) — `api.twelvedata.com`
  - Google Favicon service (best-effort bookmark icons) — `www.google.com/s2/favicons`
- **Storage:** One private GitHub gist on Tom's account (`TECGUNTHER`); per-browser localStorage holds the GitHub PAT, gist ID, default persona, Twelve Data API key, and theme choice.
- **Hosting:** GitHub Pages (free static), public repo at `github.com/TECGUNTHER/startdashboard`

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html              — the entire app (HTML + CSS + JS in one file)
├── README.md               — setup walkthrough (PAT, GitHub Pages deploy, bookmarklet)
├── PROJECT-HUB.html        — project dashboard for Tom
├── HANDOFF.md              — this file
├── docs/
│   ├── TECHNICAL.html      — architecture, decisions, file map, how-to-modify
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
- **Twelve Data signup:** https://twelvedata.com/register
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

Generate a Personal Access Token at `github.com/settings/tokens?type=beta` with `gist` read/write scope. Paste into the dashboard's setup screen.

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
- **Markets/Watchlist say "Add Twelve Data API key"** → Open Settings ⚙ → see walkthrough → paste key.
- **(v1.6) Persona gradient still shows behind the light surface after toggling** → indicates the persona-class wasn't removed before `bg-light-surface` was added. Fix is in `applyPersonaTheme()`.
- **(v1.6) Light mode looks unreadable on one specific element** → most likely a hardcoded color was missed during the tokenization pass. Search for `#11111A`, `#0A0A12`, `#0A0A0F`, or `rgba(0, 0, 0` and check whether that element needs to become a token.

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — setup walkthrough
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
- He prefers per-persona settings over global settings — matches the persona-as-channel mental model. **Exception:** theme (light/dark) is a per-browser global, not per-persona — flipping themes in one persona affects all of them in that browser. Same pattern as the default persona and the GitHub PAT.

## Multi-agent flow (as of v1.6)

- For major changes Tom now expects the actual multi-agent flow: **Design Agent** plans, Tom approves, **Build Agent** subagent writes code and returns a standard Agent Report, **Documentation Agent** subagent updates docs from that report. v1.1 through v1.5 were done by one Claude playing all roles; v1.6 was the first build to use real subagents. Tom called this out explicitly and wants it to be the pattern going forward.

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
