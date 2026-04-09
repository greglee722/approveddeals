# ApprovedDeals CRM — Project Overview

## What It Is
A single-file HTML/CSS/JS real estate deal management CRM built for a Boston-based rental brokerage. It tracks approved deals, clients, addresses, landlords, agents, payments, and paperwork (Notarized Cosigner Forms, signed leases).

## Live URL
https://greglee722.github.io/approveddeals

## Architecture
- **Frontend:** Single `index.html` file (~3,500 lines) — HTML, CSS, and JS all in one file
- **Database:** Google Sheets (OAuth via Google Identity Services)
- **Hosting:** GitHub Pages (static file hosting, free)
- **AI:** Anthropic Claude API (claude-sonnet-4-6) for natural language deal creation
- **Auth:** Google OAuth 2.0 (token-based, no backend)
- **File Storage:** Google Drive (for payment/paperwork photo attachments)

## Key Credentials (stored in the file)
- **OAuth Client ID:** `808344707268-kdj9p87cgkp36nk9k1ju9s5h6aap3te0.apps.googleusercontent.com`
- **Spreadsheet ID:** `1sJMskh19goukAZ_-lCgb7ufsZb31VywyLsa9zeGtI_s`
- **Anthropic API Key:** Stored in localStorage as `ai_api_key` (user-provided)

## Tech Constraints
- **No build system** — pure vanilla JS, no npm, no bundler
- **No backend server** — all logic runs in the browser
- **Single file** — everything in `index.html`. Never split into multiple files.
- **Column order in Google Sheets is critical** — never reorder columns; always append new columns to the right
- **JS syntax validation** — extract JS with Python regex, then run `node --check /tmp/check.js` (NOT directly on .html file)
- **Template literals** — be careful with backticks in Python string manipulation; use line-number replacement when needed
- **Nested template literals** — avoid backtick strings inside template literal expressions; use string concatenation instead

## Fonts
- Headings: `Playfair Display` (Google Fonts)
- Body: `IBM Plex Sans` (Google Fonts)

## Color Scheme
```css
--bg: #e3dacb;
--surface: #f0eee6;
--surface2: #e8e4da;
--border: #d4cfc6;
--accent: #000000;
--text: #000000;
--muted: #7a7570;
--danger: #c0392b;
--success: #1a6b3a;
```

## Current Version
v2.5 (displayed in header next to logo)

## Version History
| Version | Changes |
|---------|---------|
| v1.0 | Initial release |
| v1.1 | Inline SVG pencil/edit icons on detail card tile headers (admin-only); payment type dropdown fix on Add Row |
| v1.2 | Notes field added to payment system — captured per payment row, displayed in Payments & Paperwork Log Details column |
| v1.3 | Version bumping convention established; Notes field restored in payment modal (4-column layout: Client / Type / Amount / Notes); Details column correctly renders `p.notes` in payment log |
| v1.4 | Removed redundant Property detail card from deal detail view |
| v1.5 | (reserved) |
| v1.6 | (reserved) |
| v1.7 | Payment photo attachments — upload check/document images per payment row; stored in Google Drive; "View" link appears in payment log Details column; added `drive.file` OAuth scope |
| v1.8 | (reserved) |
| v1.9 | AI Deal Creator — delivered fields (`fmrDelivered`, `lmrDelivered`, etc.) included in AI JSON output schema; `applyAIDeal()` sets delivered dropdowns automatically |
| v2.0 | Dashboard column sorting — all six columns sortable (Lease Start, Name, Landlord, Agent, Stage, Payments); unified sort state with `sortCol`/`sortDir`; cycles asc → desc → none |
| v2.1 | Attachments tile — auto-appears on deal detail next to Clients tile when payment log entries have images; inline lightbox viewer (Escape/backdrop dismiss); version bumping system formalized |
| v2.2 | Paperwork tab parity — Notes field and photo upload button added to Notarized Cosigner Form rows; attachments from paperwork now appear in Attachments tile; `_uploadingRowType` flag routes uploads correctly |
| v2.3 | AI Deal Creator inline status flow — spinning progress bar with live steps; textarea auto-clears on success |
| v2.4 | Clickable landlord name in deal detail hero bar |
| v2.5 | Due Soon tile (within 15 days) + Overdue tile replace Drafts stat tile; Due Date filter with payment type selector (Any/App Fee/FMR/Broker/LMR/Sec Dep/Key Fee/NCF) and window (7/15/30 days/Overdue); clicking tiles auto-applies filter; hash-based page navigation (Back button, refresh restores position) |
