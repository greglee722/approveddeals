# ApprovedDeals CRM

Single-file HTML/CSS/JS deal-tracking CRM for Boston Linked RE. Tracks approved rental deals, clients, addresses, landlords, agents, payments, and paperwork (Notarized Cosigner Forms, signed leases).

## Live

https://greglee722.github.io/approveddeals

## Architecture

- **Frontend:** single `index.html` — HTML + CSS + JS in one file
- **Database:** Google Sheets (Sheets API v4)
- **Hosting:** GitHub Pages (static)
- **Auth:** Google OAuth 2.0 (Google Identity Services)
- **File storage:** Google Drive (payment / paperwork photo attachments)
- **AI:** Anthropic Claude (`claude-sonnet-4-6`) for natural-language deal creation

## Deployment

Any push to `main` automatically publishes to GitHub Pages within ~30 seconds.

## Version

The current app version is shown in the header logo (`vX.X`). Full history is in `git log`.
