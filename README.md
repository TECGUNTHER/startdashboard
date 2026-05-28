# StartDashboard

> **Note:** This README is for the original author's own deployment under GitHub username `TECGUNTHER`. If you want to set up your own StartDashboard, see [INSTALL-FOR-OTHERS.md](INSTALL-FOR-OTHERS.md) instead — it's a self-contained walkthrough for any new user.

> Tom's personal home page across every browser. One URL, identical view, synchronized in seconds.

A single-file web app that becomes the home screen for every browser you use. Persona tabs along the top (Work / Personal / Investments / …), each its own self-contained dashboard with categories of bookmarks, widgets, background, and accent color. All data lives in one private GitHub gist on your account.

**Live URL (after deploy):** https://tecgunther.github.io/startdashboard/

---

## What it does

- **Personas (channels) at the top.** One click to switch between Work, Personal, anything you create. Each persona has its own bookmarks, widgets, background, and accent color.
- **Per-device default.** Your work Mac opens to Work, your iPhone opens to Personal. Configurable per browser.
- **One-click bookmark add.** Drag the bookmarklet to your bookmarks bar once. On any site, click it → pick where it goes → done.
- **Drag-drop from address bar.** Drag any URL onto any category card.
- **Per-persona widgets.** Multi-time-zone clock, weather (free Open-Meteo, no API key), to-do list, sticky notes.
- **8 built-in dark/modern backgrounds.** Plus paste-any-image-URL. Plus per-persona accent color.
- **No ads, no third-party services** — just GitHub and (for weather) the free Open-Meteo API.

---

## Setup — first time, end to end (~10 minutes)

### Step 1 — Deploy to GitHub Pages

You need a GitHub repo named `startdashboard` under your `tecgunther` account with `index.html` at the root, served via GitHub Pages.

```
# from inside this project folder:
git init
git add index.html README.md
git commit -m "Initial StartDashboard"
git branch -M main
git remote add origin https://github.com/TECGUNTHER/startdashboard.git
git push -u origin main
```

Then in GitHub:
1. Go to **Settings → Pages**.
2. Source: **Deploy from a branch**.
3. Branch: **main**, folder: **/ (root)**. Save.
4. Wait ~60 seconds. Your dashboard is now at `https://tecgunther.github.io/startdashboard/`.

### Step 2 — Generate a GitHub Personal Access Token (PAT)

The dashboard needs read/write access to one gist on your account. The token never leaves your browser (stored in localStorage, sent only to GitHub).

**Easiest path — fine-grained token:**

1. Visit [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta).
2. **Generate new token.** Name it `StartDashboard`.
3. Expiration: 1 year (or "No expiration" if you prefer).
4. **Account permissions → Gists → Read and write.**
5. Generate. **Copy the token now** — it only shows once.

Or — classic token, single scope:

- [github.com/settings/tokens/new?scopes=gist&description=StartDashboard](https://github.com/settings/tokens/new?scopes=gist&description=StartDashboard) — the link pre-fills the right scope.

### Step 3 — First-run setup in the browser

1. Open `https://tecgunther.github.io/startdashboard/` in your primary browser.
2. The setup screen appears. Paste your token.
3. Select **First browser — create a new gist**.
4. **Continue.** The dashboard creates a private gist and loads.

**On your other browsers / devices:** repeat Step 2 to make a token (each browser needs its own — never share tokens across browsers), then in the setup screen choose **I already set up another browser** and paste the gist ID. You can find the gist ID in **Settings → Sync → Gist ID** on the first browser.

### Step 4 — Install the bookmarklet

Inside the dashboard:

1. Open **Settings (⚙)** → click **Bookmarklet code**.
2. Drag the "+ Add to StartDashboard" button to your browser's bookmarks bar.
3. On any website, click the bookmark. A popup opens. Pick which persona and category. Done.

Install the bookmarklet on every browser where you want one-click adding.

---

## Day-to-day use

- **Switch personas** — click any tab in the top bar.
- **Add a bookmark** — three ways:
  - Click the bookmarklet on any site.
  - Drag a URL from the address bar onto a category card.
  - Click **+ Add bookmark** at the bottom of a category.
- **Rename anything** — double-click the dashboard title, a persona tab, a category title, or a bookmark name.
- **Reorder** — drag persona tabs, drag category titles, drag bookmarks between categories.
- **Move bookmark across personas** — hover a bookmark → click **↗** → pick destination.
- **Delete** — hover for the × button (bookmarks), use **⋯ menu** on categories, or right-click a persona tab.
- **Change persona theme** — click ⚙ in the header → adjust accent color and background for the current persona.
- **Web search** — type in the search bar (Google by default; switch engines in Settings).

---

## Troubleshooting

| If… | Try this |
|---|---|
| Setup screen says "Bad credentials" | Token wasn't pasted correctly, or its `gist` scope is missing. Re-generate. |
| Sync indicator turns red ⚠ | Click it to reload from the gist. If still failing, your token may have expired — regenerate and **Reset this browser** in Settings. |
| Changes not appearing in other browser | Reload the other browser (Cmd+R / F5). Sync polls every ~60s, but a reload is instant. |
| Bookmarklet popup blocked | Allow popups for `tecgunther.github.io` in your browser settings. |
| Weather widget says "Could not load" | Open-Meteo had a transient failure; refresh. Confirm lat/lon are valid numbers in Settings. |
| Lost token but still have the gist | Make a new token with the same `gist` scope. In the setup screen, choose "I already set up another browser" and paste the gist ID. |
| Want to wipe a browser's setup but keep the data | Settings → **Reset this browser**. Token + gist ID cleared from this browser only; the gist itself untouched. |

---

## Privacy & security

- The PAT lives in `localStorage` of the browser you set up. It's sent only to `api.github.com`, never to any other host.
- All bookmark/widget data lives in one **private** gist on your GitHub account.
- The dashboard makes outbound requests to two domains only:
  - `api.github.com` (for sync)
  - `api.open-meteo.com` (for weather, only if you've added cities)
- Favicons are loaded best-effort from `www.google.com/s2/favicons` so site logos appear next to bookmarks. If you don't want Google to see which sites you bookmark, you can patch this out in `index.html` (search for `s2/favicons`).
- If you lose a device, revoke just that browser's token in [GitHub → Settings → Tokens](https://github.com/settings/tokens). The other browsers keep working.

---

## Stack

- Single-file HTML — vanilla JS, no framework, no build step
- Google Fonts (Inter + JetBrains Mono) — only external CSS dependency
- GitHub REST API — sync (`/gists/:id` GET/PATCH)
- Open-Meteo — weather (free, no key)
- Hosted on GitHub Pages

No npm, no node_modules, no compile step. You can edit `index.html` in any text editor, commit, push, and the change is live.

---

## File map

```
projects/personal-dashboard/
├── index.html              ← the entire app
├── README.md               ← this file
├── PROJECT-HUB.html        ← project dashboard (Tom's first stop)
├── HANDOFF.md              ← drop-in context for any LLM
├── docs/
│   ├── TECHNICAL.html      ← architecture and decisions
│   └── USER-EXPERIENCE.html ← what the app does, plain English
└── plans/
    └── 2026-05-27-personal-dashboard-v1.md  ← approved build plan
```
