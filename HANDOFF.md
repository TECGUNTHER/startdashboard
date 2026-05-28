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
**Current phase:** Complete (with v1.8 hotfix shipped)
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
- Markets (Twelve Data — defaults to SPY/QQQ/DIA for S&P/Nasdaq/Dow)
- Watchlist (Twelve Data — Tom's custom stocks)

---

# Where We Are

## Most recent session (2026-05-28 — v1.8 hotfix)

A three-bug hotfix on top of v1.7, surfaced by Tom after a day of real use.

- **Settings no longer wipes the Twelve Data API key when the field is left empty.** v1.7's `saveSettings` treated an empty trimmed input as "clear the stored key" — meaning every time Tom opened Settings for an unrelated change and hit Save, the API key got nuked (Safari's `type="password"` field can't reliably re-populate from autofill, so the field reads empty even when the key is stored). v1.8 deletes that destructive `else` branch: empty input means "no change." Only a non-empty trimmed value updates the stored key. To explicitly clear the key, go to Danger Zone → Reset browser. A code comment in `saveSettings` documents the Safari rationale.
- **Cold-start retry on Crypto, Markets, and Watchlist.** Refactored `renderCryptoWidget` and `renderTwelveDataWidget` to wrap the fetch + render body in an internal `doFetch(isRetry)` async helper. On first failure, the widget restores its "Loading…" placeholder and schedules `setTimeout(() => doFetch(true), 2000)`. On second failure, it falls through to the existing `fin-error` red message. Exactly one retry per widget per load, at 2000ms. Both Markets and Watchlist inherit this since they both route through `renderTwelveDataWidget`. The existing `_tdCache` / `TD_CACHE_MS` logic in `fetchTwelveDataQuotes` is untouched — retry sits above the cache layer.
- **Active persona tab restyled from solid slab to soft fill.** v1.7's solid `var(--accent)` with `var(--on-accent)` text was high-contrast but visually loud. v1.8 keeps `font-weight: 600` but softens the colors: background `var(--accent-soft)`, text `var(--text-primary)`, border `var(--accent)` (unchanged) for an accent outline. The active-tab dot reverted from `--on-accent` back to `--accent`. Same discoverability win, calmer look.

## Sessions before that

- **v1.7 hotfix (2026-05-28, late afternoon):** Negative finance prices were rendering with 4 decimals (`-89.4463 → -$89.4463`) — `formatPrice()` rewritten to branch on `Math.abs()` and re-apply the sign at output. Bookmark description popover spacing bumped to 28px to leave room for Safari/Chromium's native ellipsis-truncation tooltip. Active persona tab styled solid `var(--accent)` to fix the "no tab selected on first frame" bug (the visual was right but loud — see v1.8 for the softening pass).
- **v1.6 (earlier 2026-05-28):** Light/dark theme toggle in the header (☀ ↔ 🌙, persists per browser), all finance prices trimmed to 2 decimals (sub-cent altcoins keep 4), bookmark hover popover restructured with URL on a small monospace footer line, "Default persona on this browser/device" surfaced as a focal block at the top of Settings, CSS palette tokenized (`--modal-bg`, `--input-bg`, `--popover-bg`, `--on-accent`) so the whole light skin is one override block. First build to run the real subagent flow.
- **v1.5 (2026-05-28):** Bookmark description redesign (hover popover, multi-line edit modal), dollar + percent change on Crypto/Markets/Watchlist, Twelve Data API key inline walkthrough, full doc refresh (HANDOFF/TECHNICAL/UX had been stale at v1.0).

## Immediate next step

Tom pushes v1.8 to GitHub, hard-reloads Safari + Chrome, and smoke-tests the three fixes:

1. Open Settings, leave the Twelve Data key field empty, save — reopen Settings and confirm the key dots are still there; finance quotes still load.
2. Open the page on a cold tab and confirm Crypto resolves cleanly even if the first CoinGecko call hits a network hiccup — at most a brief "Loading prices…" for ~2 seconds, then real data; no red error. Same for Markets/Watchlist.
3. Confirm the active persona tab now reads as a soft fill (soft-accent background + bold primary text + accent outline), not the solid-accent slab from v1.7.

Then he uses it for a few days before the next iteration.

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
- **Bookmark URL shown inside the description popover, not in a `title` tooltip** (v1.6) — the browser's native URL tooltip was overlapping the description popover. Removing `title` and showing the URL on a small monospace footer line below the description gives one cohesive hover artifact instead of two competing ones.
- **`formatPrice` shows 2 decimals everywhere ≥ $0.01, 4 decimals below** (v1.6) — Tom wants prices that look like prices, not scientific notation. Sub-cent visibility is preserved for tiny altcoins.
- **Negative-aware formatters branch on `Math.abs()` and re-apply the sign at output** (v1.7) — comparing the raw value to a positive threshold lets negatives fall through to the wrong branch. Any future number formatter that handles negatives should follow the same pattern.
- **Accommodate the browser's native truncation tooltip, don't try to suppress it** (v1.7) — Safari and some Chromium builds show a native tooltip for ellipsis-truncated text that can't be reliably suppressed cross-browser. Popovers near truncated text leave a 28px gap so the native tooltip doesn't overlap.
- **Active persona tab uses soft-accent fill + accent outline + bold primary text** (v1.8) — v1.7's solid-accent slab solved the discoverability bug ("no tab selected on first frame") but was visually loud. Soft fill + bold text + the existing accent border keeps the discoverability win without overshooting. `font-weight: 600` does as much of the lifting as color now.
- **Empty Settings input means "no change" for credentials** (v1.8) — saving Settings should never be accidentally destructive. Safari's `type="password"` fields can't reliably re-populate from autofill, so an empty read is ambiguous; treating it as "clear" punished Tom every time he opened Settings for an unrelated change. The only way to explicitly clear the Twelve Data key is now Danger Zone → Reset browser, where destructive intent is unambiguous. Reusable principle for any future credential UI.
- **Finance widgets retry once after 2 seconds on initial-load failure** (v1.8) — Tom kept seeing red "Could not load" states on cold tab open that resolved on manual reload — a classic transient-flake symptom. One retry at 2s is the sweet spot: two would mask real outages too long; zero leaves manual-reload friction in place. Retry sits above the `_tdCache` layer, so cached quotes are still respected.
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
├── index.html               — the entire app (HTML + CSS + JS in one file)
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
- **(v1.7) Negative price renders with 4 decimals** → `formatPrice()` must branch on `Math.abs(n)`, not raw `n`. If the bug ever returns, it's because someone reverted the sign-stripping pattern in `formatPrice()` or re-introduced a positive-only threshold comparison.
- **(v1.7) Description popover sits behind a translucent tooltip on long bookmark titles** → that's the browser's native ellipsis tooltip. Don't try to suppress it; the popover gap (`.bookmark .bk-desc-popover { margin-top: 28px }`) is sized to leave room. If the popover is hugging the row again, the margin got shrunk.
- **(v1.7) Active persona tab not highlighted on first frame** → check `render()`: `const p = currentPersona()` must be called before `renderPersonaTabs()`. `currentPersona()` lazily initializes `state.currentPersonaId`, so reversing the order leaves the first frame without an active tab.
- **(v1.8) Twelve Data API key got wiped after saving Settings** → shouldn't happen anymore. `saveSettings` no longer treats an empty trimmed key field as "clear." If it ever does happen again, someone re-introduced the destructive `else` branch in `saveSettings` — restore the v1.8 behavior (empty means "no change," only a non-empty value writes).
- **(v1.8) A finance widget shows "Loading…" for ~2 seconds before resolving** → that's the new cold-start retry doing its job. The first fetch failed transiently; the widget is auto-retrying once at 2s before falling through to the existing red error. Not a hang.
- **(v1.8) Need to explicitly clear the Twelve Data API key** → Settings → Danger Zone → Reset browser. The Settings field can no longer remove it (empty means "no change") — specifically because Safari's password-type field can't reliably re-populate from autofill, so an empty read used to be ambiguous and accidentally destructive.

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — Tom's own setup walkthrough (TECGUNTHER-specific). The top of the README points third-party users at `INSTALL-FOR-OTHERS.md` instead.
- `INSTALL-FOR-OTHERS.md` — clean install guide for anyone Tom shares the project with: fork → enable GitHub Pages → generate their own PAT → run their own copy with their own private gist. Self-contained; no Tom-specific references.
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

- For major changes Tom now expects the actual multi-agent flow: **Design Agent** plans, Tom approves, **Build Agent** subagent writes code and returns a standard Agent Report, **Documentation Agent** subagent updates docs from that report. v1.1 through v1.5 were done by one Claude playing all roles; v1.6 was the first build to use real subagents, and v1.7's and v1.8's hotfixes followed the same pattern. Tom called this out explicitly and wants it to be the pattern going forward.

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
