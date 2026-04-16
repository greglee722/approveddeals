# Development Guide

## How to Make Changes

### The Golden Rule
This is a single `index.html` file. Every change is a surgical edit to that file. Never:
- Split into multiple files
- Add a build system
- Use npm packages
- Create a backend

### Workflow
1. Read the relevant section of `index.html` using `view` tool with line range
2. Make the targeted change using `str_replace` (exact match) or Python line replacement
3. Validate JS syntax — extract JS with Python regex, then run `node --check /tmp/check.js`
4. Bump the version number in the header logo (see Version Bumping below)
5. Present file with `present_files`
6. User uploads to GitHub Pages and replaces `/mnt/project/index.html`

### Adding a New Feature Checklist
- [ ] Does it need a new Google Sheet column? → append to end of `SHEET_HEADERS[tab]` array, note user must add header to sheet manually
- [ ] Does it need a new modal? → place OUTSIDE `#app` div, use `.mini-overlay` pattern
- [ ] Does it need a new page/tab? → add `.page` div inside `#main`, add nav button, add `showPage` handler
- [ ] Does it save data? → function must be `async`, use `await saveRecord()`
- [ ] Does it involve JS template literals? → verify braces balance after editing
- [ ] **Always bump the version number** before delivering any output file

### JS Syntax Validation (Always Do This)
```python
import re
content = open('/home/claude/index.html').read()
scripts = re.findall(r'<script>(.*?)</script>', content, re.DOTALL)
js = '\n'.join(scripts)
open('/tmp/check.js','w').write(js)
```
Then: `node --check /tmp/check.js`

Note: `node --check index.html` does NOT work — must extract JS first.

### Python String Replacement Gotchas
When using Python to insert JavaScript containing template literals:
```python
# BAD - backticks get escaped
content = content.replace(old, f"const x = `hello ${name}`")

# GOOD - use line number replacement for JS with template literals
lines[1234] = "  const x = `hello ${name}`;\n"
```

**Nested template literal warning:** Never put a backtick string inside another backtick string inside a JS template literal embedded in Python. This causes `SyntaxError: Invalid regular expression`. Use string concatenation instead:
```js
// BAD (inside template literal)
${p.imageUrl ? `<a href="${p.imageUrl}">View</a>` : ''}

// GOOD
${p.imageUrl ? '<a href="' + p.imageUrl + '">View</a>' : ''}
```

### Common Patterns

#### Adding a field to the deal form
1. Add HTML input to deal modal (inside `.form-section`)
2. Add field to `SHEET_HEADERS.deals` array (end)
3. Clear field in `openDealModal()` reset block
4. Populate field in `openDealModal()` edit block (`setV('df-fieldname', d.fieldname)`)
5. Save field in `saveDeal()` deal object
6. Display field in `openDealDetail()`

#### Adding a new tab
1. Add nav button to `.header-nav`
2. Add `.page` div with `id="page-{name}"` inside `#main`
3. Add `if (p==='{name}') renderXTable();` in `showPage()`
4. Write `renderXTable()` function

#### Making a field searchable in filter panel
Add to `FILTER_FIELDS` object and handle in `applyFilters()` and `renderValueInput()`.

#### Admin-only UI elements
Wrap in `${isAdmin() ? '...' : ''}` in template literals, or check `if (isAdmin())` in JS.
Edit pencil icons on detail card headers follow this pattern.

#### Google Drive file upload (v1.7+)
The app uses `uploadFileToDrive(file)` — uploads via multipart to Drive API, then sets `reader/anyone` permission so the link is publicly viewable. Returns a `https://drive.google.com/file/d/{id}/view` URL. OAuth scope must include `https://www.googleapis.com/auth/drive.file`. Access token is stored in `accessToken` variable (also in `sessionStorage` as `crm_access_token`).

#### AI Deal Creator — delivered fields (v1.9+)
The AI system prompt includes `fmrDelivered`, `lmrDelivered`, `secdepDelivered`, `keyfeeDelivered`, `ncfDelivered`, `leaseDelivered` in its JSON output schema. These map directly to the `df-{type}-delivered` select dropdowns via `applyAIDeal()`. Broker and App Fee have no delivered fields in the form. The `deliverablePayments` array in `applyAIDeal()` controls which payments get the delivered dropdown set: `['fmr','lmr','secdep','keyfee']`.

#### Dashboard Column Sorting (v2.0+)
All six dashboard columns are sortable. The unified sort state is managed by two variables:
```js
let sortCol = null; // 'leaseStart' | 'name' | 'landlord' | 'agent' | 'stage' | 'payments'
let sortDir = null; // 'asc' | 'desc' | null
```
Each `<th>` calls `toggleSort(col)`. Clicking a new column sets it to `asc`. Clicking the same column cycles `asc → desc → null`. The `SORT_COLS` array lists all sortable column keys. Sort icons are referenced by id `sort-icon-{col}`. The sort logic lives inside `applyFilters()` after the filter pass.

To add a new sortable column:
1. Add `id="sort-icon-{col}"` span to the `<th>`
2. Add `col` to `SORT_COLS` array
3. Add `else if (sortCol==='{col}')` branch in `applyFilters()` sort block

#### Paperwork Row Attachments (v2.2+)
The Paperwork tab (Notarized Cosigner Form rows) now mirrors the Payment tab with a **Notes** field and **photo upload button** per row.

Key implementation details:
- `_uploadingRowType` flag (`'payment'` or `'paperwork'`) routes the shared file input handler to the correct row array
- `triggerPaperworkImageUpload(rowId)` sets `_uploadingRowType = 'paperwork'` before clicking the hidden file input
- `handlePaymentImageFile()` checks `_uploadingRowType` to find the row in `paperworkRows` vs `paymentRows` and calls the correct render function after upload
- Saved entries include `notes` and `imageUrl` fields, so they appear automatically in the **Attachments tile** on the deal detail view

#### Attachments Tile (v2.1+)
The Attachments tile appears automatically in the deal detail view, next to the Clients tile, whenever any `paymentLog` entries have a non-empty `imageUrl` field.

- Rendered inline in `openDealDetail()` using an IIFE that filters `d.paymentLog` for entries with `imageUrl`
- Each row shows the payment `type` as the label, `client` as a subtitle, and a **View** button
- Clicking **View** calls `openLightbox(url, title)` which opens the `#attachment-lightbox` modal
- Lightbox can be closed via x, backdrop click, or `Escape` key (`closeLightbox()`)
- The lightbox also includes an "Open in new tab" link pointing to the Drive URL

To add support for attachments on a new entry type, ensure the entry has an `imageUrl` property populated — the tile renders automatically.

#### Toast Notifications (v2.7+)
A lightweight toast system is available globally via `showToast(msg)`.

- `#toast` div is placed just after `</div><!-- /app -->` in HTML
- CSS: fixed bottom-center, black bg, fades in/out via `.show` class toggle
- Auto-dismisses after 2.5 seconds; calling `showToast()` again resets the timer
- Use for any lightweight one-off confirmations (clipboard copy, quick saves, etc.)

#### Copy All Client Emails — Deal Detail (v3.0+, updated v3.8)
A single clipboard icon in the Clients card header copies all client emails for the deal.

- Button placed far-right in the card header, next to the edit pencil
- Function: `copyDealClientEmails(dealId)` — collects all client emails from the deal, joins with `, `, copies to clipboard, shows toast
- Uses `navigator.clipboard.writeText()` with `execCommand('copy')` fallback
- **v3.0:** Button in card header. **v3.7:** Moved inline per-client chip. **v3.8:** Moved back to card header as single copy-all icon; per-chip copy icons removed.

#### Welcome Email Draft (v3.3+)
A **"Generate Welcome Email"** button appears in the deal detail header action row for both admin and agent roles. Clicking it calls `openEmailDraftModal(dealId)`.

Key implementation details:
- `generateWelcomeEmail(deal)` builds the email body from a fixed template, dynamically omitting payment sections where the `needed` amount is 0 or empty (Broker Fee, Security Deposit, LMR, Key Deposit)
- Client first names are pulled from `db.clients` via the deal's `clients` array
- Landlord name is used as the payable-to value for all cashier's check payments
- Due dates are formatted via `formatEmailDate(dateStr)` (Long month format, e.g. "June 1, 2025")
- Amounts are formatted via `formatCurrency(val)` (dollar sign, 2 decimal places)
- The modal (`#email-draft-overlay`) uses the `.mini-overlay` / `.mini-modal` pattern
- "Copy Body" uses `navigator.clipboard.writeText()` with `execCommand('copy')` fallback
- "Open in Mail Client" builds a `mailto:` URI and assigns it to `window.location.href`
- Subject is always: `Approval for [address]` — address resolved via `resolveDealAddress(deal)`
- `_emailDraftDealId` holds the current deal id while the modal is open
- **v3.4:** `resolveDealAddress()` falls back to `deal.name` if `addressDisplay` and `addressId` are both empty
- **v3.5:** RE: line in the mailing address block always uses `deal.name` (not the address)

#### Notarized Cosigner Email Draft (v3.6+, updated v3.8)
A **"Generate Notarized Cosigner"** button appears in the deal detail header action row for both admin and agent roles. Clicking it calls `openNCFDraftModal(dealId)`.

Key implementation details:
- Reuses the same `#email-draft-overlay` modal as the welcome email
- **Subject**: `Notarized Cosigner Form for [deal.name]`
- Due date pulled from `deal.ncfDue` via `formatEmailDate()`; shows `___________` if not set
- Address in body uses `deal.name` as primary value

**Per-cosigner To/CC (v3.8+):**
- Modal has two new elements: `#email-draft-cosigner-row` (dropdown, hidden by default) and `#email-draft-cc-row` (CC field, hidden by default)
- `openEmailDraftModal()` (Welcome Email) explicitly hides both rows
- 1 cosigner: rows show without dropdown; To = cosigner email, CC = matched client email via `cosigner.clientId` → `db.clients`
- 2+ cosigners: dropdown renders in `#email-draft-cosigner-select`; `selectNCFCosigner(idx)` updates To and CC on selection
- `openEmailInMailClient()` reads `#email-draft-cc` and appends `&cc=` to the `mailto:` URI if non-empty

**Client name grammar (v3.8+):**
- 1 name: `Name`
- 2 names: `Name1 and Name2`
- 3+ names: `Name1, Name2, and Name3` (Oxford comma via `.slice(0,-1).join(', ') + ', and ' + last`)

#### lastUpdated Persistence (v2.8+)
`lastUpdated` is now included in `SHEET_HEADERS.deals` as the last field, mapping to column **BH** in the Google Sheet.

- Header `lastUpdated` must exist in cell **BH1** of the deals tab in Google Sheets
- The field is a plain ISO timestamp string — no special `rowToObj` parsing needed
- It is stamped in: `saveDeal()`, `savePaymentModal()`, `deletePaymentEntry()`, `deletePaymentLog()`
- The dashboard **Last Updated** column reads `d.lastUpdated` and formats it via `relativeDate(iso)`
- Existing deals will show `—` until next edit; once edited the value persists across sessions

## Deploying
1. Save `index.html` to `/mnt/user-data/outputs/index.html`
2. User goes to `github.com/greglee722/approveddeals`
3. Click `Add file → Upload files`
4. Upload `index.html` (replaces existing)
5. Also replace `/mnt/project/index.html` with the new file to keep Claude's project in sync
6. Commit changes
7. Live at `https://greglee722.github.io/approveddeals` in ~30 seconds

## Version Bumping
**Every new output file must increment the version.** This lets you confirm the correct file is live after uploading.

The version string is in the header logo:
```html
<div class="logo">Approved<span>Deals</span> <span style="font-size:11px;font-weight:400;color:rgba(255,255,255,0.5);font-family:var(--font-body);letter-spacing:0.5px;">v3.0</span></div>
```

Increment the minor version (v3.0 → v3.1) for each new output, regardless of how small the change.

## Version History
| Version | Changes |
|---------|---------|
| v1.0 | Initial release |
| v1.1 | Inline SVG pencil/edit icons on detail card tile headers (admin-only); payment type dropdown fix on Add Row |
| v1.2 | Notes field added to payment system |
| v1.3 | Version bumping convention established; Notes field restored in payment modal |
| v1.4 | Removed redundant Property detail card from deal detail view |
| v1.7 | Payment photo attachments via Google Drive |
| v1.9 | AI Deal Creator delivered fields |
| v2.0 | Multi-column sort system |
| v2.1 | Attachments tile + inline lightbox on deal detail |
| v2.2 | Paperwork tab Notes + image upload; paymentLog entries save `notes` and `imageUrl` |
| v2.3 | AI Deal Creator inline status flow with spinner steps |
| v2.5 | Due Soon/Overdue stat tiles; Due Date filter; hash-based page navigation |
| v2.6 | Last Updated sortable column on dashboard; Created date in deal hero bar |
| v2.7 | Toast notification system |
| v2.8 | `lastUpdated` added to SHEET_HEADERS (persists to Google Sheets col BH) |
| v2.9 | Fixed notes input losing focus in Paperwork tab; split payment vs. paperwork badges on deal detail hero |
| v3.0 | Copy Emails functionality implemented on individual deal Clients tiles only; removed global Copy All Emails button from main Clients page |
| v3.3 | Welcome Email Draft — `generateWelcomeEmail(deal)` + `openEmailDraftModal(dealId)`; "Generate Welcome Email" button on deal detail for admin and agent views |
| v3.4 | `resolveDealAddress()` falls back to `deal.name` before placeholder; same fallback applied to email subject line |
| v3.5 | AI deal creator auto-creates address records via expanded JSON schema (addressStreet1/2/City/State/Zip/Rent/Beds/Baths); fuzzy match prevents duplicates; RE: line uses `deal.name` |
| v3.6 | Notarized Cosigner email draft — `openNCFDraftModal(dealId)`; reuses email draft modal; populates cosigner emails, client first names, deal name, NCF due date |
| v3.7 | Inline copy icon per client chip — `copyClientEmail(clientId)`; copy button removed from Clients card header |
| v3.8 | Client tile: single copy-all icon in card header; NCF name grammar fix (Oxford comma); NCF modal per-cosigner To/CC selector + CC field in mailto |

## Google Sheets Column Map (deals tab)
| Column | Header | Notes |
|--------|--------|-------|
| A–BC | id → leaseNotes | See SHEET_HEADERS array for full order |
| BC | archiveReason | |
| BD | archivedOn | |
| BE | created | ISO timestamp |
| BF | paymentLog | JSON array — do not edit manually |
| BG | (legacy empty column) | Artifact from early version — ignore, leave blank |
| BH | lastUpdated | Added v2.8 — ISO timestamp |

## Google Sheets Manual Editing Rules
- Never change column order
- Never delete row 1 (headers)
- Never edit `clients`, `cosigners`, or `paymentLog` columns directly (JSON format)
- After manual edits: sign out and back in to reload data
- IDs must be filled for rows to load (column A)
- ID formats: `d_` (deals), `cli_` (clients), `a_` (addresses), `lnd_` (landlords), `agt_` (agents), `req_` (edit requests)

## Adding a New Google Sheets Column
1. Add field name to end of relevant `SHEET_HEADERS[tab]` array in code
2. Manually add the column header to the Google Sheet (row 1, next empty column)
3. The `ensureHeaders()` function will NOT auto-add it (it only writes headers on empty sheets)
4. Handle the field in `rowToObj()` if it needs special parsing (JSON, int, etc.)

## OAuth Scopes (v1.7+)
```js
const SCOPES = 'https://www.googleapis.com/auth/spreadsheets https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/drive.file';
```
If a new scope is added, users must sign out and re-authorize to grant it.
