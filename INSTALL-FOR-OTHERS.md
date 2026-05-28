# StartDashboard — Setup Guide for Your Own Copy

This guide walks you through forking StartDashboard and running it as your own personal home page. About 10 minutes from start to finish. No coding required.

---

## What this is

StartDashboard is a single-file web app that becomes the start page across every browser and device you use. A tab bar at the top lets you switch between **personas** — Work, Personal, Investments, anything you create. Each persona is its own complete dashboard: its own bookmarks (organized into categories), its own widgets, its own background, its own accent color. The default look is dark and modern; a sun/moon button in the header flips the whole UI to a clean light mode for daytime use.

Seven widgets are available per persona, each show-or-hide-able: **Clock** (multi time zone, 12 or 24 hour), **Weather** (free Open-Meteo, no key), **To-do list**, **Sticky notes**, **Crypto** (free CoinGecko, no key), **Markets** (S&P / Nasdaq / Dow via Twelve Data — free key needed), and **Watchlist** (your custom stocks).

Your data lives in one private GitHub gist on your own GitHub account. Open the same dashboard URL in Safari on your Mac and Chrome on your laptop and they show the same view, synced within seconds.

---

## What you need

- A **GitHub account** (free).
- About **10 minutes** for the first-time setup.
- *(Optional)* a **Twelve Data API key** (free, no credit card) if you want the Markets and Watchlist widgets to work. You can skip this and add it later.

That's it. No terminal beyond a couple of git commands (and you can do those through the GitHub web UI if you prefer). No npm, no Docker, no compile step.

---

## Files you need on your computer

Practically just one: `index.html`. That single file is the entire app — HTML, CSS, and JavaScript in one place. When you fork the repo, this is what gets served by GitHub Pages.

The rest of what you see in the repo (`PROJECT-HUB.html`, `HANDOFF.md`, `docs/`, `plans/`, `reports/`) is project-tracking documentation from the original author's authoring workflow. You don't need any of it to run the app. Feel free to delete those files from your fork if you want a leaner repo — or keep them as reference. They don't affect anything.

The two reference docs in `docs/` are still useful reading if you ever want to:

- Understand what the app does, screen by screen — `docs/USER-EXPERIENCE.html`
- Modify the code (add a widget, change colors, etc.) — `docs/TECHNICAL.html`

---

## How it's designed to be shared

StartDashboard is built so that sharing it works out of the box, with no privacy compromise:

- **It's a single static HTML file.** No backend, no shared database, no central server.
- **Every user runs their own copy.** When you fork the repo, you get your own deployment under your own GitHub Pages URL.
- **Every user's data goes into their own private gist.** On their own GitHub account. No one else can read it.
- **No shared accounts.** Your GitHub Personal Access Token never leaves your browser. The original author never sees your data; other forkers never see your data; you never see theirs.

The fork model means each user fully controls their own deployment. If the original repo disappeared tomorrow, your fork would keep working.

---

## Step-by-step setup

### Step 1 — Fork the repo

1. Go to **[https://github.com/TECGUNTHER/startdashboard](https://github.com/TECGUNTHER/startdashboard)**.
2. Click the **Fork** button in the top-right of the page.
3. Confirm the fork. Default settings are fine — you'll get a copy at `https://github.com/YOUR-USERNAME/startdashboard`. (Replace `YOUR-USERNAME` with your actual GitHub username throughout this guide.)

### Step 2 — Enable GitHub Pages on your fork

1. In your forked repo, go to **Settings** (top tab) → **Pages** (left sidebar).
2. Under **Source**, pick **Deploy from a branch**.
3. Branch: **main**. Folder: **/ (root)**. Click **Save**.
4. Wait about 60 seconds for the first deploy.

### Step 3 — Your dashboard is live

Your StartDashboard is now at:

```
https://YOUR-USERNAME.github.io/startdashboard/
```

Open that URL in your primary browser. You'll see a setup screen asking for a GitHub token — that's the next step.

### Step 4 — Generate a GitHub Personal Access Token (PAT)

The dashboard needs permission to read and write **one** gist on your account. The token stays in your browser only — it's sent only to `api.github.com`, never anywhere else.

**Easiest path — fine-grained token:**

1. Go to **[github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)**.
2. Click **Generate new token**. Name it `StartDashboard`.
3. Expiration: **1 year** (or "No expiration" if you prefer).
4. **Account permissions → Gists → Read and write.**
5. Click **Generate token**. **Copy it now** — GitHub only shows it once.

**Or — classic token, one scope:**

- Use **[github.com/settings/tokens/new?scopes=gist&description=StartDashboard](https://github.com/settings/tokens/new?scopes=gist&description=StartDashboard)**. The link pre-fills the right scope.

Either token type works.

### Step 5 — First-run setup in the browser

1. Open `https://YOUR-USERNAME.github.io/startdashboard/`.
2. Paste your token into the setup screen.
3. Select **First browser — create a new gist**.
4. Click **Continue**. The dashboard creates a private gist on your account and loads.

You'll see two default personas (Work and Personal) with seeded Daily/Tools categories, a dark Midnight Mesh background, and clock + weather + to-do + notes widgets up top. From here, you can rename things, add bookmarks, swap backgrounds, etc.

### Step 6 — *(Optional)* Add a Twelve Data API key

Only needed if you want the Markets or Watchlist widgets to show real stock prices. Crypto works without it (CoinGecko is free and key-less).

1. In the dashboard, click **Settings ⚙** in the header.
2. Find the **Twelve Data API key** section. Follow the inline four-step walkthrough: sign up at twelvedata.com/register (free, no credit card), verify your email, copy your key, paste it in.
3. Save changes. Markets and Watchlist start working within a few seconds.

Each browser stores its own Twelve Data key in localStorage — it's not synced through the gist, because rate limits are per-key. If you set the dashboard up in five browsers, you can either use the same key in all of them or generate separate keys for higher combined throughput.

### Step 7 — Install the bookmarklet

The bookmarklet lets you add a bookmark to your dashboard with one click from any website.

1. In the dashboard, open **Settings ⚙** → click **Bookmarklet code**.
2. Drag the **"+ Add to StartDashboard"** button to your browser's bookmarks bar.
3. Done. On any website, click the bookmark in your bar. A popup opens. Pick which persona and category. Click. The bookmark lands on your dashboard within seconds.

Install the bookmarklet on every browser where you want one-click adding.

### Step 8 — Set up additional browsers / devices

On every other browser or device:

1. Open `https://YOUR-USERNAME.github.io/startdashboard/`.
2. Generate a **new** GitHub PAT for that browser (Step 4 again — never share tokens across browsers).
3. Paste the new token into the setup screen.
4. Choose **I already set up another browser**.
5. Paste your **gist ID** (find it in **Settings → Sync → Gist ID** on the first browser).
6. Continue. The new browser loads with all your existing data.
7. Optionally pick a different default persona for this browser (in Settings, near the top: "Default persona on this browser/device") — useful if you want your work laptop to open to Work while your iPhone opens to Personal.

---

## Customization (optional)

You're free to edit `index.html` directly to make the app yours:

- **Change the default personas.** Search for `Work` and `Personal` in `index.html` to find the seed values.
- **Add or rename background presets.** Add a CSS rule `.bg-<name>` and a matching entry in the `BG_PRESETS` array (search for `BG_PRESETS`). All nine current backgrounds are CSS-only — no images to load.
- **Change colors.** The CSS palette lives in the `:root` block near the top of the file. Dark mode and light mode both reference the same token names; the light overrides are in a single `body.light-mode` block.
- **Add a new widget.** Outlined in detail in `docs/TECHNICAL.html` under **How to Modify**. Roughly: add a `<div class="widget" data-widget="..."`, append to `ALL_WIDGETS` and `WIDGET_LABELS`, and write a `render<Name>Widget(persona)` function.
- **Change the search engine list.** The `SEARCH_ENGINES` object near the top of the script. Defaults to Google with DuckDuckGo, Bing, and Kagi as options.

For deeper code edits, read `docs/TECHNICAL.html` — the **Key Decisions** and **How to Modify** sections explain why the code is shaped the way it is.

---

## Privacy and security

- Your **GitHub PAT** lives in `localStorage` on the browser you set up. It's sent only to `api.github.com`, never to any other host.
- Your **data** (bookmarks, widget config, notes, to-dos) lives in one **private** gist on your GitHub account. Only you can read it.
- The dashboard makes outbound requests to:
  - `api.github.com` — for sync (always)
  - `api.open-meteo.com` — for weather and city/timezone lookups (only if you use the Weather or Clock widgets)
  - `api.coingecko.com` — for crypto prices (only if you use the Crypto widget)
  - `api.twelvedata.com` — for stock prices (only if you've added a Twelve Data key)
  - `www.google.com/s2/favicons` — for the small site logos next to bookmarks (best-effort)
- You can revoke any browser's token at any time from **[github.com/settings/tokens](https://github.com/settings/tokens)** without affecting the others.
- If you don't want favicons routed through Google, search for `s2/favicons` in `index.html` and remove the block or swap in a different favicon source.

---

## Troubleshooting

| If… | Try this |
|---|---|
| Setup screen says "Bad credentials" | Token wasn't pasted correctly, or it's missing the `gist` scope. Regenerate it. |
| Sync indicator turns red ⚠ | Click it to reload from the gist. If still failing, your token may have expired — regenerate, then **Reset this browser** in Settings → Danger Zone. |
| Changes not appearing in another browser | Reload the other browser (Cmd+R / F5). Sync polls every ~60s; a manual reload is instant. |
| Bookmarklet popup blocked | Allow popups for `YOUR-USERNAME.github.io` in your browser settings. |
| Weather widget says "Could not load" | Transient Open-Meteo failure; refresh. Or you haven't added a city yet — open the Weather widget's ⚙ to add one. |
| Markets / Watchlist say "Add Twelve Data API key" | Open Settings ⚙ → see the inline walkthrough → paste a key. |
| Finance widget shows "Loading…" for ~2 seconds before resolving | That's the cold-start retry doing its job — the first fetch failed transiently and the widget is auto-retrying once before falling through to an error. Not a hang. |
| Twelve Data key seems to "disappear" | It hasn't — saving Settings with the field empty does **not** clear the key (intentional, since some browsers can't reliably re-display password-type values). The key is still stored. To explicitly clear it, use **Settings → Danger Zone → Reset browser**. |
| Lost a token but still have the gist | Make a new token with the `gist` scope. In the setup screen, choose **I already set up another browser** and paste your gist ID. |
| Want to wipe a browser's setup but keep the data | **Settings → Danger Zone → Reset this browser.** Token and gist ID are cleared from this browser only; the gist itself is untouched. |

---

## What's NOT shared between users

Even though many people may be running the same app code, **no data is shared between users**. The fork model gives each person their own complete, independent deployment:

- **Your bookmarks are yours alone.** They live in your private gist, on your GitHub account.
- **Your widgets and their config are yours alone.** Your cities, your stocks, your to-dos, your notes — all in your gist.
- **Your tokens are yours alone.** Each browser you set up has its own PAT in its own localStorage. Tokens are never synced.
- **Your URL is yours alone.** `YOUR-USERNAME.github.io/startdashboard/` serves your fork's `index.html` and points at your gist. Nobody else sees what you see when they hit their own URL.

If you fork the repo and someone else forks it independently, you'll both run the same app code but you'll have completely separate dashboards. No interaction between them. No way for either of you to see the other's data.

---

## Going further

- **What the app does, screen by screen, plain English** → `docs/USER-EXPERIENCE.html`
- **Architecture, decisions, and how to modify the code** → `docs/TECHNICAL.html`
- **Original author's project tracking and version history** → `PROJECT-HUB.html` (not needed for use, but interesting if you're curious about the build history)

If you build a new widget, fix a bug, or add a feature that would be useful to others, feel free to open a pull request on the original repo. The app is intentionally single-file and dependency-free — keep contributions in that spirit.

Enjoy.
