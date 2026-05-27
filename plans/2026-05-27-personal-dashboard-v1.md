# Build Plan — Personal Dashboard / Bookmark Hub

**Date:** 2026-05-27
**Status:** Draft — awaiting Tom's approval
**Plan author:** Orchestrator (Design Agent flow, in plan mode)
**Project folder:** `projects/personal-dashboard/`
**Product name in the UI:** **StartDashboard**
**GitHub account:** `TECGUNTHER`
**GitHub repo name:** `startdashboard` (live URL: `https://tecgunther.github.io/startdashboard/`)
**Starter personas (seeded on first run):** **Work**, **Personal** — Tom can add more (e.g. Investments) anytime via the `+` button on the tab bar

---

## Context

Tom uses Chrome and Safari daily (plus other browsers occasionally), and bookmarks live trapped inside each browser. Over time he forgets what's bookmarked or even what each one referred to. He's tried StartMe but dislikes the ads, the clunky add flow, and the difficulty of keeping it organized. He also juggles distinct mental modes — Work, Personal, focused interests like Investments — that each pull in completely different sites and tools. He wants a self-owned alternative that:

- Lives at one URL, identical across every browser and device
- Synchronized — changes in one browser show up in the others within seconds
- Easy bookmark add (one-click bookmarklet + drag-drop from address bar)
- Quick rename and re-categorize (double-click, drag)
- Custom widgets pulling from his own data
- **Multiple "personas" or channels** (Work / Personal / Investments / …) — each its own self-contained dashboard
- **Customizable backgrounds** — dark, modern, tech aesthetic by default; easy to swap
- No ads, no third-party lock-in

This is **Project #1** in the workspace.

---

## The Idea, In Plain English

A single web page (**StartDashboard**) that becomes the home screen for every browser you use. A tab bar across the top lets you switch between personas — Work, Personal, Investments, whatever you create — and each persona is its own complete dashboard: its own categories, bookmarks, widgets, background, and accent color. Click a bookmarklet on any site to add it to the current persona; drag URLs from your address bar onto a category; double-click any name to rename it. Each persona has its own row of widgets — clock with time zones, weather, a to-do list, sticky notes. The dashboard's data lives in one private GitHub gist on your account, so opening the same URL in Chrome on your Mac and Safari on your iPhone gives you the same view, synchronized within seconds. Each device remembers its preferred default persona (your work Mac opens to Work; your iPhone opens to Personal), and you can flip between them with one click on the tab bar.

---

## What It Will Do (Behavior)

### Personas (the top-level concept)

- **Tab bar across the top of the page**: one tab per persona, plus a `+` to create new
- Each persona has: a name, an accent color, a background, its own categories+bookmarks, its own widgets+settings
- Switch personas: single click on a tab → entire dashboard re-renders for that persona
- Create a persona: `+` button → name, pick an accent color, pick a background → empty dashboard ready
- Rename a persona: double-click its tab
- Delete a persona: `...` menu on the tab → confirmation (must have ≥1 persona remaining)
- Reorder personas: drag tabs left/right
- **Per-device default**: each browser stores in `localStorage` which persona auto-loads when the dashboard opens. Your work Mac defaults to Work; your iPhone defaults to Personal. Override anytime.
- If too many personas to fit in the tab bar (responsive), overflow collapses into a "More ▾" dropdown

### Layout (per persona)

- **Top header**: dashboard title (editable), persona tab bar, sync status indicator, settings button
- **Search bar**: directly below header, hits Google by default (configurable per persona)
- **Widgets row**: 2×2 grid (or single row on wide screens) of widget cards — _this persona's_ widget instances
- **Categories grid below**: each category is a card with title + list of bookmarks
- **All on top of the persona's chosen background**, with the persona's accent color as the highlight throughout

### Bookmarks

- **Bookmarklet**: drag a "+ Add to Dashboard" button to your browser's bookmarks bar once. On any site, click it → small popup overlay asks "Add to which category? (in which persona?)" → click → done. Popup closes.
- **Drag from address bar**: drag a URL from any browser's address bar onto a category card → added to that category in the current persona, with the page's title as the label
- **Inline rename**: double-click any bookmark to rename; Enter to save
- **Move between categories**: drag a bookmark across categories
- **Move between personas**: right-click a bookmark → "Move to other persona ▸" → pick persona → pick category
- **Delete**: hover → small × appears

### Categories

- `+ New category` button at bottom of the grid
- Double-click category title to rename
- Drag a category by its title to reorder
- `...` menu on category to delete (confirmation if not empty)

### Widgets — per persona

Each persona owns its own instance of each widget with its own state.

- **Clock (multi time zone)**: shows current time across any number of user-added zones
- **Weather**: current conditions + 3-day forecast for one or more cities (Open-Meteo API, no key required)
- **To-do list**: simple checklist; add, check off, clear completed
- **Sticky notes**: freeform multi-line scratchpad; auto-saves as you type

Future widgets to be added in later builds (out of scope for v1): custom embed, recently added bookmarks, quick links.

### Backgrounds & Visual Customization

Each persona has its own background and accent color.

- **Built-in background library** (8–10 dark/modern presets) — examples: "Midnight Mesh" (subtle animated gradient mesh), "Aurora" (slow shifting auroral gradient), "Synthwave" (grid + sunset), "Carbon Grid" (fixed dark grid), "Deep Space" (starfield), "Liquid Glass" (soft dark blur), "Tokyo Night" (urban dark gradient), "Terminal" (matrix-style minimal). All implemented in CSS — no images required.
- **Image URL**: paste any URL (Unsplash, your own hosted image, etc.) — the page will use it as the background with a subtle dark overlay so text stays readable
- **Solid color**: pick any color
- **Per-persona accent color**: chosen at create time, shown throughout that persona's UI (buttons, hover states, current-tab highlight)
- Change anytime: persona settings panel → background + accent

### Visual Direction (Default)

- **Dark, modern, tech.** Reference points: Linear dark mode, Vercel dashboard, Raycast, Arc browser.
- Deep near-black surfaces (`#0A0A0F` base) with subtle elevation via tinted gradients
- Soft glass effect on category cards and widget cards (subtle background blur + translucent fill)
- Vibrant per-persona accent color used sparingly — borders on hover, current-tab indicator, focus rings
- Typography: **Inter** for UI, **JetBrains Mono** for timestamps, accents, the clock widget, and the search bar input
- No animation theater — animations only on legitimate state changes (tab switch fade, sync indicator transitions)
- Fully responsive 380px → 1600px; mobile collapses the widgets row above the categories grid

### Sync

- Data lives in one private GitHub gist on Tom's GitHub account
- Page reads the gist on load; shows "Loaded HH:MM" timestamp
- Edits are debounced and written back ~2 seconds after you stop changing things
- **Sync status indicator** in the header: ● Synced / ◐ Saving… / ⚠ Out of date — reload to pull latest
- If a conflict is detected (gist updated by another browser while you've been editing), banner appears — you decide whether to reload (loses your unsaved changes) or keep working (overwrites the other browser's changes)

### First-time setup (per browser, once)

- On first load, dashboard shows a "Setup" screen
- Step 1: link to GitHub's "Create personal access token" page with exact instructions (only `gist` scope needed)
- Step 2: paste the token into a field, click Save
- Step 3: dashboard either creates a fresh gist (if this is your first browser) or you paste the gist ID (if you've already set it up in another browser)
- Step 4: pick your default persona for this browser → done
- Token stored in this browser's `localStorage` only — never transmitted anywhere except the GitHub API
- Reset/change: settings panel → "Re-setup this browser"

---

## What It Will NOT Do (Scope Boundary)

- No multi-user / shared dashboards. This is Tom's, period.
- No real-time live collaboration. Sync is "every few seconds when you save"; conflicts surface a banner, last save wins.
- No browser extension. Bookmarklet does the job without per-browser-build overhead.
- No automatic bookmark import from Chrome/Safari in v1. (Easy to add JSON import later.)
- No widget marketplace or plugin system in v1. Four baked-in widgets per persona.
- No mobile-native app. The web page is responsive; mobile Safari handles it.
- No image upload in v1. Backgrounds support preset + image URL + solid color (upload can be added later if Tom wants it).
- No analytics, no tracking, no third-party scripts (only Google Fonts + Open-Meteo weather API).

---

## Approach

**Architecture** is intentionally boring and durable. The page is a single HTML file — one `<style>` block, vanilla JS, no framework. It's hosted as a static page on GitHub Pages at `https://tecgunther.github.io/startdashboard/`. The dashboard data — all personas, their categories+bookmarks+widget state+background+accent — lives as a single JSON file in one private GitHub gist on your account. Reads and writes go through the GitHub REST API, authenticated with a personal access token you generate once and paste in per browser.

**Why this stack:** Single HTML matches your stated preference. No build step, no framework churn. GitHub provides durable free hosting + durable free storage with no third-party service to outlive you. Sync is "every browser opens the same URL, fetches the latest JSON" — true sync without us writing a server. Per-browser personal access tokens mean if you lose a device, you revoke that token on GitHub and the rest keep working.

**Data shape (rough):**
```
{
  "personas": [
    { "id": "work", "name": "Work", "accent": "#22D3EE", "background": "midnight-mesh",
      "categories": [ { "name": "Daily", "bookmarks": [ ... ] }, ... ],
      "widgets": { "clock": {...}, "weather": {...}, "todos": [...], "notes": "..." } },
    { "id": "personal", "name": "Personal", "accent": "#F472B6", ... },
    { "id": "investments", "name": "Investments", "accent": "#34D399", ... }
  ],
  "personaOrder": ["work", "personal", "investments"]
}
```

Per-device localStorage holds: GitHub PAT, gist ID, this device's default persona ID.

**Bookmarklet flow:** Tiny JavaScript snippet opens a small popup window pointed at your dashboard URL with `?add=<url>&title=<title>` params. The dashboard sees those params, shows a compact "Pick persona + category" overlay, writes to the gist, closes the popup. No third-party server.

---

## Port Number

**No port needed** — static HTML page served by GitHub Pages. For local preview while developing, just double-click the HTML file (works file:// because there's no backend).

MASTER-HUB Ports & Commands entry: `Project: personal-dashboard · No port · Live URL: https://tecgunther.github.io/startdashboard/`

---

## Files to Be Created

| File | What it does | New or modified |
|------|--------------|-----------------|
| `projects/personal-dashboard/index.html` | The entire dashboard — HTML, CSS, JS in one file | New |
| `projects/personal-dashboard/PROJECT-HUB.html` | Project dashboard (built from template) | New |
| `projects/personal-dashboard/HANDOFF.md` | Drop-in context for other LLMs | New |
| `projects/personal-dashboard/docs/TECHNICAL.html` | Implementation reference | New |
| `projects/personal-dashboard/docs/USER-EXPERIENCE.html` | What the app does, plain English | New |
| `projects/personal-dashboard/plans/2026-05-27-personal-dashboard-v1.md` | This plan (moved from temp location) | New |
| `projects/personal-dashboard/README.md` | Quick-start: GitHub Pages deploy, PAT generation, bookmarklet install | New |
| `MASTER-HUB.html` | Workspace portfolio dashboard — first card added | New (first project) |
| `agent-system/templates/*.html` | Templates need to exist before Doc Agent runs | New (Design Agent builds these on first project) |

---

## Dependencies

- **Google Fonts** — Inter + JetBrains Mono
- **Open-Meteo API** — free weather data, no API key required
- **GitHub account** — Tom already has one
- **GitHub personal access token** with `gist` scope — generated once, pasted once per browser
- **No npm, no build tools, no framework, no install step**

---

## Open Questions for Tom

None — all answered.

- Product name in UI: **StartDashboard**
- GitHub account: `TECGUNTHER`
- Repo name: `startdashboard` (live URL `https://tecgunther.github.io/startdashboard/`)
- Starter personas: **Work** and **Personal**; "+" button on the tab bar to add more (Investments, etc.) anytime

---

## Unknowns (for Research Agent)

None — no research needed.

---

## Definition of Done

1. `index.html` opens cleanly in Chrome, Safari, Firefox on macOS, plus mobile Safari on iPhone — single file, no broken layout.
2. First-time setup flow works: paste a GitHub PAT, choose to create or join a gist, pick default persona, transition to dashboard.
3. Persona tab bar renders, switching personas re-renders the entire dashboard in under a second.
4. Creating, renaming, reordering, and deleting personas works.
5. Per-device default persona is honored on reload (different defaults on different browsers).
6. Each persona has independent categories, bookmarks, widget state, background, and accent color.
7. Background library shows ≥8 dark/modern presets; image URL field works; per-persona accent color picker works.
8. Bookmarklet adds a bookmark end-to-end (click on any site → category picker → persists to gist → visible).
9. Drag-drop from address bar onto a category works in Chrome and Safari.
10. All four widgets render per-persona and persist their state.
11. Editing in one browser shows up in another browser within 5 seconds of reload.
12. Sync status indicator correctly reflects synced / saving / out-of-date states; conflict banner appears when applicable.
13. Deployed to GitHub Pages at a stable URL Tom bookmarks in every browser.
14. README explains GitHub PAT setup, gist setup, bookmarklet install, in 5 steps or fewer per topic.
15. Documentation Agent has run and `PROJECT-HUB.html`, `HANDOFF.md`, and `MASTER-HUB.html` all reflect this build.
