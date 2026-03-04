# Player Card Technical Architecture

Reference implementation: `/home/dso/playercards/openrouter-2026/`

## System Overview

Each player card is a Cloudflare Worker that serves a password-protected, single-page HTML application with an embedded resume PDF and email alerts on access.

## File Roles

### src/worker.js (Template)
- Contains `{{HTML_CONTENT}}` and `{{RESUME_PDF_BASE64}}` placeholders replaced at build time
- Three routes:
  1. `GET /resume` or `/resume.pdf` — serves embedded PDF (NO auth required)
  2. `POST /auth` — password form submission, sets cookie, sends email alert
  3. `GET /` — serves login page (no cookie) or player card (valid cookie)
- `BASE_PATH` constant for URL routing (default `/player-card`)
- Auth: HTTP-only cookie `player_card_auth=authenticated`, 24hr expiry, Secure, SameSite=Strict
- Email alerts via Resend API using `ctx.waitUntil()` (non-blocking)
- Visitor info captured: IP, geolocation, user agent, ASN/org, referer, timezone
- Environment secrets: `PASSWORD`, `RESEND_API_KEY`, `ALERT_EMAIL`

### src/index.html
- Self-contained HTML with embedded CSS (no external dependencies)
- Dark theme using CSS custom properties
- Responsive design (breakpoints at 600px and 768px)
- Print styles for PDF export
- Contains both login screen and player card content
- Client-side JS for session handling (backup to server-side auth)

### build.js
- Node.js script (only Node built-ins required)
- Reads src/index.html and resume PDF
- Modifies HTML: hides login screen, shows player card by default
- Escapes backticks and template literals in HTML
- Replaces `{{HTML_CONTENT}}` and `{{RESUME_PDF_BASE64}}` placeholders
- Outputs: `dist/worker.js` (deployable) and `public/index.html` (preview)

### wrangler.toml
```toml
name = "adam-pridgen-playercard-[company]"
main = "dist/worker.js"
compatibility_date = "2024-01-01"

routes = [
  { pattern = "[company].sp98.org/player-card*", zone_name = "sp98.org" }
]
```

### package.json scripts
```json
{
  "build": "node build.js",
  "deploy": "npm run build && wrangler deploy",
  "dev": "npm run build && wrangler dev",
  "preview": "npm run build && npx serve public"
}
```

## CSS Design System

```css
:root {
    --primary: #6366f1;        /* Indigo - CTAs, badges, section borders */
    --primary-dark: #4f46e5;   /* Darker indigo - hover states */
    --success: #10b981;        /* Green - match score, positive items, impact metrics */
    --warning: #f59e0b;        /* Amber - caution, threats SWOT */
    --danger: #ef4444;         /* Red - errors, weakness SWOT */
    --bg-dark: #0f172a;        /* Very dark slate - page background */
    --bg-card: #1e293b;        /* Dark slate - card backgrounds */
    --bg-card-hover: #334155;  /* Lighter slate - hover, table headers */
    --text-primary: #f1f5f9;   /* Light slate - main text */
    --text-secondary: #94a3b8; /* Medium slate - secondary text, labels */
    --border: #334155;         /* Medium slate - borders */
}
```

## HTML Section Structure (in order)

1. **Login Screen** — centered login box, hidden when authenticated
2. **Header** — name, subtitle, match score badge (green pill), resume download link, role target box
3. **TL;DR Section** — gradient background box (primary→success), narrative pitch, "Case in point" callout
4. **Impact Metrics** — `profile-grid` with large stat numbers in green
5. **SWOT Analysis** — `swot-grid` (2x2), colored left borders (green/blue/red/amber)
6. **JD Requirements** — `profile-grid` with two `metric-card` columns (core + bonus)
7. **Why [Company] Specifically** — `metrics-grid` with bullet lists demonstrating company knowledge
8. **Technical Credibility** — `metrics-grid` (recent work + research/patents)
9. **Career Timeline** — vertical timeline with dots, `.timeline` class
10. **Questions for Company** — `metrics-grid` with role and company questions
11. **Contact** — `profile-grid` with links + large resume download CTA
12. **Footer** — generation date and target

## Key CSS Components

- `.profile-grid` — `grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))`
- `.swot-grid` — `grid-template-columns: repeat(2, 1fr)` (1fr on mobile)
- `.metrics-grid` — `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`
- `.swot-card` — `.strengths` (green), `.opportunities` (primary), `.weaknesses` (red), `.threats` (amber)
- `.metric-card` — card with uppercase h4 header, bullet list
- `.profile-stat` — centered stat with large `.value` and small `.label`
- `.timeline` — vertical left border with positioned dots
- `.match-score` — green gradient pill badge

## Login Page (Server-Side Rendered)

The worker serves a separate lightweight login page HTML via `getLoginHTML()` function. This is NOT the same as the login screen in index.html. Key details:
- Form action: `POST /auth` (relative to base path)
- Input: `name="password"`, required, autofocus
- Error state: passed as boolean, shows/hides error div
- Same CSS variables as main card
- Candidate name and company in description text

## Email Alert HTML

Subject: `Player Card Viewed - {city}, {country}`
From: `Player Card Alert <onboarding@resend.dev>`
Body: H2 title mentioning the company, visitor details list (IP, location, timezone, ISP, referrer, user agent)
