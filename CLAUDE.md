# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal academic dashboard for Zeming Wang at RWTH Aachen University, deployed at `zemingwang.de`. No build system — this is a pure static HTML site served directly via a web server on an Ubuntu VPS. GitHub Actions handles deployment.

## Project Goals

Long-term vision for this site beyond the current exam dashboard:

- **Social media aggregation hub** — let followers from Instagram, TikTok, Bilibili, and other platforms cross-discover each other through a single landing point
- **Academic portfolio** — showcase research, projects, and achievements to strengthen PhD applications
- **Personal interests showcase** — fitness, cooking, art, and other hobbies presented as a personal brand
- **Future monetization** — subscriptions, merchandise, and paid content mechanisms

## Development

No build, lint, or test commands. To preview locally, serve the files with any static HTTP server:

```bash
python3 -m http.server 8080
# or
npx serve .
```

The frontend expects a backend API running at the same origin (e.g., `/api/login`, `/api/me`, `/api/health`, `/api/states`). Without the VPS backend running, auth redirects will loop to `login.html`.

## Architecture

### Pages

| File | Purpose | Auth required |
|---|---|---|
| `index.html` | Main dashboard with module grid | Yes |
| `login.html` | Login form | No |
| `404.html` | Custom 404 error page | No |
| `rwth_aachen_2026.html` | RWTH exam schedule & study tracker | Yes |

### Frontend Stack (all CDN-loaded, no npm)

- **Vue 3** (`vue.global.js`) — reactive UI on each page
- **Tailwind CSS** — utility-first styling via CDN
- **Phosphor Icons** — icon library via CDN
- **MathJax 3** — LaTeX math rendering (exam page only)

### Auth Flow

Every protected page runs an inline auth guard script **before** Vue mounts to prevent content flash:

```js
(function () {
    const token = localStorage.getItem('auth_token') || sessionStorage.getItem('auth_token');
    if (!token) window.location.replace('/login.html');
})();
```

After Vue mounts, each page calls `GET /api/me` to verify the token is still valid server-side.

### Global Auth Helpers (defined in `index.html`, exposed on `window`)

`index.html` defines and exports four helpers for use across pages:

- `window.apiCall(path, options)` — centralized fetch wrapper; auto-attaches `Authorization: Bearer <token>` header, redirects to `/login.html` on 401
- `window.getAuthToken()` — reads token from localStorage or sessionStorage
- `window.getAuthUser()` — reads and parses the cached user object
- `window.clearAuth()` — removes all auth keys from both storages

### User Roles

Two roles enforced by the backend; the frontend reflects them visually:

- `admin` — full read/write access to exam states and file uploads
- `viewer` — read-only; write endpoints return 403

"Remember me" checked → token stored in `localStorage` (7-day JWT). Unchecked → `sessionStorage` (cleared when tab closes).

### Backend API (on VPS, not in this repo)

Node.js + Express + SQLite (better-sqlite3, WAL mode), managed by PM2.

| Method | Endpoint | Auth |
|---|---|---|
| POST | `/api/login` | Public |
| GET | `/api/me` | Any authenticated |
| GET | `/api/health` | Public |
| GET | `/api/states` | Any authenticated |
| POST | `/api/states/:lv_nummer` | Admin only |
| POST | `/api/files/upload` | Admin only |
| GET | `/api/files/:lv_nummer` | Any authenticated |
| GET | `/api/files/download/:id` | Any authenticated |
| DELETE | `/api/files/:id` | Admin only |
| GET | `/api/global-notes` | Any authenticated |
| POST | `/api/global-notes` | Admin only |

### RWTH Exam Page (`rwth_aachen_2026.html`)

The largest file (~100 KB). Key features: per-exam state tracking synced to the backend API, contenteditable notes with MathJax rendering, ECTS progress analysis, file attachment display, and print layout support (`@media print`).

## Design System

RWTH Aachen brand colors defined as CSS variables:

```css
--primary-blue: #00549F;
--dark-blue:    #002F5F;
--accent-green: #00875A;
--accent-red:   #E30066;
```

Glassmorphism card style (`.glass-card`) is reused across all pages. Admin badge is green; viewer badge is slate.

## Known TODOs

Planned features not yet implemented:

- Viewer account management UI (create/delete viewer accounts from the dashboard)
- Email subscription / newsletter mechanism
- Shopping cart / monetization mechanism (paid content, merchandise)
- Modularize inline JS and CSS out of HTML files (currently everything is inline)
- Consider migrating to Vite + single-file Vue components once complexity grows

## Instructions for Claude

- **Read the full file before editing.** HTML files contain inline JS that frequently exceeds 500 lines — always read the entire file before making changes to avoid missing context or breaking logic elsewhere.
- **Do not introduce new CDN dependencies** without discussing it first. All external libraries are currently loaded via CDN; any addition needs deliberate consideration.
- **Maintain RWTH brand color consistency.** The two primary accent colors are `#00549F` (blue) and `#E30066` (magenta/red). Do not substitute or approximate them.
- **Variable and function names in English; Chinese comments are fine.**
- **After completing a code change, suggest a git commit message** following the format already used in this repo (e.g., `feat: ...`, `fix: ...`).
- **Flag when an HTML file approaches 1500 lines** and recommend extracting JS or CSS into a separate file or splitting the page into a module.
