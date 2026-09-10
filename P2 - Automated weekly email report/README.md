🇬🇧 English | [🇪🇸 Español](./README.es.md)

# P2 — Automated Weekly Lead Report by Email

## Problem

Business owners collecting leads (via forms, WhatsApp, ads) rarely review them consistently. Manual spreadsheet checks are inconsistent, time-consuming, and leads fall through the cracks without anyone noticing.

## Solution

A fully automated weekly report that lands in the owner's inbox every Monday morning — no manual work required.

**Flow:** `Schedule Trigger (weekly, Mon 8am) → Google Sheets (read leads) → Code (format as HTML table) → Gmail (send report)`

1. **Schedule Trigger** fires every Monday at 8:00 AM.
2. **Google Sheets** node reads all lead rows from the tracking sheet.
3. **Code node** (JavaScript) transforms the raw rows into a clean HTML table with a lead count summary.
4. **Gmail node** sends the formatted report to the configured recipient(s).

## Business Result

Estimated time saved: ~45 min/week of manual review (~3 hours/month) per business, plus fewer leads lost to inconsistent follow-up. Report generation and delivery is fully hands-off once configured.

## How This Sells

- **Target client:** small service businesses (clinics, agencies, real estate, local commerce) with a lead-capture channel already in place but no reporting discipline.
- **Price range:** $50–100 USD setup fee as a standalone deliverable.
- **Upsell:** bundles naturally with P1 (WhatsApp lead notifier) into a "Lead Follow-up System" package — higher ticket, same core stack.

## Tech Stack

n8n (self-hosted, Docker/Ubuntu) · Google Sheets API · Gmail API · JavaScript (Code node)

## Key Technical Learnings

- Google Sheets tab names can be renamed after workflow setup — the node's cached `sheetName` reference (by `gid`) still resolves correctly, but always re-verify the displayed tab name matches what's expected before running in production.
- HTML email body must be built entirely in the Code node and passed as a single string (`$json.htmlReport`) to the Gmail node's Message field — inline styles are needed since Gmail strips external CSS.
- OAuth2 credential setup (Google Cloud Console → enable API → create OAuth client → authorize in n8n) is the same flow reused across all Google-based nodes (Sheets, Gmail, Drive) — one credential setup unlocks the pattern for future projects.

## Next Iterations (planned)

- Filter to only leads created in the last 7 days, instead of the full sheet on every run.
- Improve HTML table styling for a more polished, branded look.
