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
- Often returns to projects after long gaps. Needs orientation, not assumed context.

---

# The Project

**Name:** StartDashboard
**Folder:** `projects/personal-dashboard/` in Tom's `tom-workspace`
**Current status:** In Progress (built; deployment pending)
**Current phase:** Documented
**Last updated:** 2026-05-27

**What it does (plain English):**
A single-file web app that becomes Tom's home page across every browser and device. A tab bar at the top lets him switch between "personas" (Work, Personal, Investments, anything he creates). Each persona is its own complete dashboard with its own bookmarks, widgets, background, and accent color. Data lives in one private GitHub gist on his account, so opening the same URL in Chrome on his Mac and Safari on his iPhone shows the same view, synced within seconds. The aesthetic is dark and modern (Linear/Vercel/Raycast as reference points), and he can swap backgrounds from a library of 8 CSS-only presets or paste any image URL.

---

# Where We Are

## Most recent session (2026-05-27)

- Approved the build plan after two rounds of design input (personas and customizable backgrounds were added during planning, on Tom's prompting).
- Built `index.html` — the single-file dashboard with persona tabs, GitHub gist sync, four widgets (clock, weather, todos, notes), 8 CSS-only background presets, drag-drop bookmark adds, and a bookmarklet popup flow.
- Built the four workspace-level templates (`PROJECT-HUB`, `MASTER-HUB`, `TECHNICAL`, `UX`) since this is the first project in Tom's workspace.
- Generated `PROJECT-HUB.html`, `docs/TECHNICAL.html`, `docs/USER-EXPERIENCE.html`, and this `HANDOFF.md`.
- Created `MASTER-HUB.html` at the workspace root with this project as the first card.

## Sessions before that

- None — this is the first session on this project.

## Immediate next step

Tom needs to deploy the repo to GitHub Pages and run first-time setup in his primary browser. Specifically:
1. `git init` in `projects/personal-dashboard/`, push to `github.com/TECGUNTHER/startdashboard`, enable GitHub Pages from `main` / root.
2. Generate a Personal Access Token with `gist` scope at `github.com/settings/tokens`.
3. Visit `https://tecgunther.github.io/startdashboard/`, paste the token, choose "First browser — create a new gist."
4. Test the bookmarklet on a real site to confirm the round-trip works.
5. Repeat steps 2 and 3 on Safari and on iPhone, choosing "I already set up another browser" and pasting the gist ID from the first browser.

---

# Key Decisions Made (and Why)

- **GitHub gist as the data backend** — durable, free, no third-party service to outlive Tom's GitHub account. Chosen over Supabase, jsonbin, and iCloud-folder-based sync. The latter failed because Safari can't write back to local files.
- **Personal Access Token per browser, stored in localStorage** — avoids OAuth complexity in a static page. Each browser gets its own token; losing a device means revoking one token, not all of them.
- **Per-persona widgets, backgrounds, and accent colors** — the whole point of personas is mental separation. Sharing widgets across personas would blur the lines between Work and Personal. Each persona is fully independent.
- **Per-device default persona via localStorage** — Tom wants his work Mac to open to Work and his iPhone to open to Personal. The default is a per-device preference, not a synced setting.
- **Bookmarklet over a browser extension** — one piece of JavaScript that works in every browser, forever. A browser extension means three separate builds and ongoing maintenance.
- **Backgrounds: 8 CSS-only presets + image URL only** — image upload would bloat the gist with base64 on every save. URL-based images work well and keep the gist small.
- **Last-write-wins on sync conflicts** — real conflict resolution is overkill for a single-user app. A toast warns Tom if another browser has updated since he loaded, and the most recent save wins.
- **No automatic bookmark import from existing browser bookmarks** — bookmarks come over manually as Tom uses the bookmarklet. Adding JSON import is straightforward later if needed.

---

# The Stack

- **Language(s):** HTML, CSS, JavaScript (vanilla, no framework)
- **Frameworks / Libraries:** None
- **Data sources:** GitHub REST API (private gist holds the JSON data)
- **External APIs:** Open-Meteo (free weather, no key required); Google's favicon service (best-effort icons for bookmark rows)
- **Storage:** One private GitHub gist on Tom's account (`TECGUNTHER`); per-browser localStorage holds the PAT, gist ID, and default persona

## File structure (key files only)

```
projects/personal-dashboard/
├── index.html              — the entire app (HTML + CSS + JS in one file, ~1,800 lines)
├── README.md               — setup walkthrough
├── PROJECT-HUB.html        — project dashboard for Tom
├── HANDOFF.md              — this file
├── docs/
│   ├── TECHNICAL.html      — architecture, decisions, file map
│   └── USER-EXPERIENCE.html — what the app does, plain English
└── plans/
    └── 2026-05-27-personal-dashboard-v1.md  — approved build plan
```

---

# How to Run It

## Port: none (static page served by GitHub Pages)

## One-time setup

Deploy `index.html` to GitHub Pages under `github.com/TECGUNTHER/startdashboard`:

```
cd projects/personal-dashboard
git init
git add .
git commit -m "Initial StartDashboard"
git branch -M main
git remote add origin https://github.com/TECGUNTHER/startdashboard.git
git push -u origin main
```

Then in GitHub: **Settings → Pages → Deploy from a branch → main / (root)**.

Generate a Personal Access Token at `github.com/settings/tokens?type=beta` with `gist` read/write scope.

## To launch

```
open https://tecgunther.github.io/startdashboard/
```

For local-only preview (no sync), just double-click `index.html`.

## Troubleshooting (known issues)

- **"Bad credentials" on first run** → PAT was pasted wrong or is missing the `gist` scope. Regenerate.
- **Sync indicator turns red ⚠** → Click it to reload from the gist. If still failing, the token may have expired; reset this browser in Settings.
- **Bookmarklet popup blocked** → Allow popups for `tecgunther.github.io` in browser settings.
- **Changes not appearing across browsers** → Reload the other browser (Cmd+R). Background poll is 60s, but reload is instant.
- **Weather "Could not load"** → Transient Open-Meteo issue, or invalid lat/lon coordinates in Settings.

---

# What's in the Folder

- `index.html` — the entire app
- `README.md` — setup walkthrough (PAT, GitHub Pages deploy, bookmarklet install)
- `PROJECT-HUB.html` — visual dashboard for this project (Tom's home base for the project)
- `HANDOFF.md` — this file
- `docs/TECHNICAL.html` — architecture, dependencies, decisions, file map, how-to-modify
- `docs/USER-EXPERIENCE.html` — what the app does, features, workflows, scope
- `plans/2026-05-27-personal-dashboard-v1.md` — the approved build plan (read-only history)

---

# Open Items

- **Question** — Does mobile Safari's drag-drop from address bar behave the same as desktop? Worth a quick check on iPhone after first deploy. (added 2026-05-27)
- **Feature** — Optionally add a "Custom embed" widget for pasting iframes (calendar, RSS, etc.) once the basics are working. (added 2026-05-27)

---

# Working with Tom (Behavioral Notes)

## Communication norms

- Plain English at all times. No jargon without translation.
- One question at a time. Don't stack three at once.
- Lead with the answer. He reads executive briefs all day.
- No filler enthusiasm. Skip "Great question!", "Happy to help!"

## Project-specific quirks

- Tom prefers single-file HTML when possible. This whole app is one file by design.
- The workspace's persistent documents (PROJECT-HUB, MASTER-HUB, TECHNICAL, UX) follow a light, restrained aesthetic. The dashboard *app itself* is dark/modern/tech. Don't mix the two — keep the dark theme inside `index.html` and the light theme in the hub/doc files.
- Tom is the only user. No multi-user features, no sharing model, no admin layer.
- All data lives on his GitHub account. Don't add third-party services without checking first.

## What he's likely to want next

After deploying and running through the first-time setup, he's most likely to:
1. Hit something that doesn't work in practice (a bookmarklet popup, a sync edge case) and ask for help diagnosing.
2. Want to tune the visual look — different backgrounds, accent colors, a specific widget layout.
3. Add the "Investments" persona he mentioned during planning.
4. Eventually request the deferred features: custom embed widget, image upload for backgrounds, recently-added widget, bookmark-import flow.

---

# Deeper Reference

If you need more detail than this document provides:

- **Architecture, dependencies, technical decisions:** see `docs/TECHNICAL.html`
- **What the app does, screens, user workflows:** see `docs/USER-EXPERIENCE.html`
- **Project state and history:** see `PROJECT-HUB.html`
- **Build plans (history of what was planned and built):** see `plans/`

All files are in the project root or its subfolders.
