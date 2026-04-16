# Feature Reference

## Navigation Tabs
| Tab | Page ID | Render Function | Who Can See |
|-----|---------|-----------------|-------------|
| Dashboard | `page-dashboard` | `applyFilters()` + `renderStats()` | All |
| Payments & Paperwork | `page-payments` | `renderPaymentsTable()` | All |
| Delivery Needed | `page-delivery` | `renderDeliveryTable()` | All |
| Clients | `page-clients` | `renderObjTable('client')` | All |
| Addresses | `page-addresses` | `renderObjTable('address')` | All |
| Landlords | `page-landlords` | `renderObjTable('landlord')` | All |
| Agents | `page-agents` | `renderObjTable('agent')` | All |
| Archived *(hamburger)* | `page-archived` | `renderArchivedTable()` | All |
| Drafts *(hamburger)* | `page-drafts` | `renderDraftsTable()` | All |
| Edit Requests *(hamburger)* | `page-requests` | `renderRequestsTable()` | Admin only |

---

## Deal Stages
| Value | Name | How Set |
|-------|------|---------|
| 0 | Approved Deal | Default |
| 1 | Ready For Keys | Auto (all payments received + delivered) |
| 2 | Closed | Manual only |

---

## Payment Types
| Field Prefix | Label |
|-------------|-------|
| `appfee` | App Fee |
| `fmr` | FMR (First Month's Rent) |
| `broker` | Broker Fee |
| `lmr` | LMR (Last Month's Rent) |
| `secdep` | Security Deposit |
| `keyfee` | Key Fee |

Each has: `Needed`, `Received`, `Due`, `Notes`, and (except appfee/broker) `Delivered?`

---

## Paperwork Types
| Type | Fields |
|------|--------|
| Notarized Cosigner Form (NCF) | Needed, Due, Received, Notes, Delivered, Partial Notes |
| Signed Lease | Received?, Delivered?, Notes |

---

## "Payments & Paperwork Needed" Tab Logic
Shows active deals where ANY of:
- Payment received < needed (App Fee, FMR, Broker, LMR, Sec Dep, Key Fee)
- NCF needed > NCF received
- Lease received = "No"

---

## "Delivery Needed" Tab Logic
Shows active deals where ANY of:
- Payment **fully collected** (received >= needed > 0) AND delivered != "Yes"
- NCF fully collected AND delivered != "Yes"
- Lease received = "Yes" AND delivered != "Yes"

---

## Filter Panel
Fields: Lease Start Date, Landlord, Agent, Stage, Payment Status, Due Date
Operators: equals, does not equal, before (date), after (date)
Active filters shown as chips in dashboard header.

---

## AI Deal Builder
- Button: "AI Fill" on dashboard
- API: `claude-sonnet-4-6` via Anthropic API
- Key: stored in `localStorage` as `ai_api_key`
- Flow: natural language → Claude extracts JSON → `applyAIDeal()` pre-fills deal modal
- Auto-creates new clients and landlords if not found in existing db
- Shows inline status steps with spinner

---

## Role-Based Permissions
| Feature | Admin | Agent |
|---------|-------|-------|
| See all deals | Yes | Own deals only |
| See all clients | Yes | Own clients only |
| See all addresses | Yes | Own deal addresses only |
| See all landlords | Yes | Yes |
| Edit active deals | Yes | Request only |
| See Edit Requests tab | Yes | No |
| Archive/Delete deals | Yes | No |
| Edit pencil icons on detail cards | Yes | No |

---

## CSV Import (Clients)
- Supported fields: first, last, email, phone, notes
- Auto-matches column names (fuzzy)
- Updates existing clients matched by email
- Creates new clients for unmatched rows

---

## Welcome Email Draft (v3.3+)
- Button: **Generate Welcome Email** in the deal detail header action row (visible to admin and agent)
- Opens a modal with pre-filled **To** (client emails), **Subject** (`Approval for [address]`), and editable **Body**
- Body is generated from deal data: client first names, address, landlord name, payment amounts and due dates
- Payment sections (Broker Fee, Security Deposit, LMR, Key Deposit) are omitted if the `needed` amount is 0 or blank
- Address resolved via `resolveDealAddress()`: tries `addressDisplay` → `db.addresses` lookup → `deal.name` → placeholder
- RE: line in the mailing address block always uses `deal.name`
- **Copy Body** — copies the textarea content to clipboard; confirms via toast
- **Open in Mail Client** — opens a `mailto:` link pre-filled with To, Subject, and Body
- Functions: `generateWelcomeEmail(deal)`, `openEmailDraftModal(dealId)`, `resolveDealAddress(deal)`

---

## Notarized Cosigner Email Draft (v3.6+, updated v3.8)
- Button: **Generate Notarized Cosigner** in the deal detail header action row (visible to admin and agent)
- Reuses the same email draft modal as the welcome email
- **Subject**: `Notarized Cosigner Form for [deal name]`
- Body includes client first names and NCF due date
- Due date pulled from `deal.ncfDue`; shows `___________` if not set
- Function: `openNCFDraftModal(dealId)`

**Per-cosigner To/CC (v3.8+):**
- **1 cosigner**: To is auto-filled with the cosigner's email; CC is auto-filled with the linked client's email (looked up via `cosigner.clientId` → `db.clients`)
- **2+ cosigners**: A "Cosigner" dropdown appears above the To field; selecting a cosigner updates To and CC dynamically via `selectNCFCosigner(idx)`
- "Open in Mail Client" includes the CC address in the `mailto:` URI (`&cc=`)

**Client name grammar (v3.8+):**
- 1 name: `Name`
- 2 names: `Name1 and Name2`
- 3+ names: `Name1, Name2, and Name3` (Oxford comma)

---

## Copy All Client Emails — Deal Detail (v3.0, updated v3.8)
- A single clipboard icon appears in the **Clients card header** (far right, next to the edit pencil)
- Clicking it copies all client emails for the deal as a comma-separated list; confirms via toast
- Function: `copyDealClientEmails(dealId)`
- **v3.0:** Button was in the card header. **v3.7:** Moved to inline per-client chip. **v3.8:** Moved back to card header as a single copy-all button; per-chip icons removed.

---

## Client Detail Page
Accessed by clicking client name on Clients tab or on deal page.
Shows: Contact info, Notarized Cosigners (from client record), Associated Deals
Clicking a deal navigates to dashboard and opens that deal.

---

## Payment/Paperwork Modal
Two tabs:
1. **Payment** — Client, Payment Type (only shows where received < needed), Amount, Notes, photo upload. Multi-row. 4-column layout.
2. **Paperwork** — Client, Paperwork type (NCF only), Cosigner (auto-populated), Notes, photo upload. Multi-row.

On save:
- Payment tab: increments `{type}Received` on deal, appends to `paymentLog` with `notes` and `imageUrl`
- Paperwork tab: increments `ncfReceived`, appends to `paymentLog`

### paymentLog entry shape
```javascript
{ client: '', type: '', amount: '', date: '', cosigner: '', notes: '', imageUrl: '' }
```

---

## Deal Detail View — Cards
- **Clients** — pill/chip list of all attached clients (clickable → client detail); copy-all emails icon in card header (v3.8+); edit pencil top-right (admin only)
- **Attachments** — auto-renders when any paymentLog entry has an imageUrl; inline lightbox viewer
- **Additional Information** — free text notes (shown only if populated)
- **Payment tiles** — one card per payment type that has `needed > 0`
- **Payments & Paperwork Log** — full log table with columns: Client, Type, Details, Amount, Date

> **Note:** The Property card was removed in v1.4 — the address is already shown in the deal header.

Admin-only pencil icons appear on card headers to trigger `editDeal()` or `openPaymentModal()`.

---

## Payments & Paperwork Log — Details Column
- For paperwork entries: `Cosigner: [name]`
- For payment entries with notes: `[note text]`
- For entries with both cosigner and notes: `Cosigner: [name] · [note text]`
- For entries with imageUrl: 📎 View button linking to Drive file
- Empty otherwise

---

## Last Updated Column (v2.6+, persisted v2.8+)
- Sortable column on dashboard showing relative time of last change
- Formats: Today / Yesterday / X days ago / Mon DD
- Stored in Google Sheets column BH as ISO timestamp
- Stamped whenever a deal is saved, payment added, or log entry deleted

---

## Edit Request Flow
1. Agent clicks "Request Edit" on approved deal
2. Modal asks for change description
3. Request saved to `edit_requests` sheet with status `pending`
4. Admin sees badge on hamburger menu
5. Admin opens Edit Requests tab, clicks Approve → deal modal opens for editing
6. Or clicks Reject with optional note

---

## Toast Notifications (v2.7+)
Global `showToast(msg)` utility — fixed bottom-center, auto-dismisses after 2.5s.
Used for clipboard confirmations; available for any future lightweight feedback.

---

## Sync Indicator
Bottom-right fixed bar shows: Saving... / Saved / error message
Colors: black (loading), green for success, red for error
