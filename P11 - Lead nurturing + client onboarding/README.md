🇬🇧 English | [🇪🇸 Español](README.es.md)

# P11 — Automated Lead Nurturing System

## Problem

A lead gets scored hot, warm, or cold (see P4) — but if nobody reaches out within the next few hours, it goes cold and gets lost. Most small businesses (dealerships, clinics, real estate, law firms) don't have the manpower to systematically follow up on every lead. Follow-up depends on a salesperson remembering — and in practice, they don't. Hot leads not contacted within the first few hours drop sharply in conversion probability, and warm/cold leads simply get forgotten.

## Solution

Automated system that runs differentiated nurture sequences by lead category, generating personalized WhatsApp messages via AI based on context (name, category, sequence stage):

- **Hot:** immediate contact (0h), follow-up at 4h, then 24h — urgency tone, pushes toward closing.
- **Warm:** 24h, 72h, 7 days — educational tone, builds trust without pressure.
- **Cold:** 7, 21, and 45 days — soft presence, keeps contact alive without being pushy.

**Architecture (n8n):**

```
Schedule Trigger (hourly)
  → Get rows in sheet (reads leads with estado = active)
  → Code: evaluates category + stage vs. hours elapsed
  → Claude API (generates personalized message per lead)
  → Code: extracts message text
  → Twilio (sends WhatsApp message)
  → Edit Fields (rescues row_number and next stage)
  → Update row in sheet (advances stage, marks completed if applicable)
```

**Database:** dedicated Google Sheet (`Leads_Nutricion`) with columns: name, phone, category, entry_date, nurture_stage, status, last_updated.

## Business Result

System tested end-to-end with 10 test leads across all 3 categories and various stages. Results:
- Correctly identified who was due for a message based on elapsed time (8 of 10 active leads triggered a message; 2 were correctly excluded — one paused, one not yet at threshold).
- Generated messages with clearly differentiated tone per category.
- Sent all 8 messages via WhatsApp through Twilio with zero errors.
- Updated all 8 corresponding rows, advancing stage and marking `completed` for leads that finished their sequence (3 leads).

Zero manual intervention in the evaluate → generate → send → update cycle.


## Tech Stack

n8n · Claude Sonnet 5 (Anthropic API) · Twilio (WhatsApp sandbox) · Google Sheets

## Key Technical Learnings

1. **Twilio + Body Content Type:** must be explicitly set to "Form Urlencoded," not JSON — otherwise Twilio ignores the parameters and throws "To phone number is required" even when the field is correctly filled.
2. **JSON reset across chained HTTP Requests:** when two HTTP Request nodes run in sequence (Claude → Twilio), any data needed from the first is lost by the second. Fix: an intermediate Edit Fields node that explicitly rescues data via `$('NodeName').item.json.field` before it's needed — not reactively after the error.
3. **Matching by row_number in Google Sheets Update:** more reliable than columns with possible duplicates (e.g. phone number in test data), but row_number must be explicitly mapped as a column with its value, not just selected as the matching column — otherwise the node doesn't know which row to touch.
4. **Confirmed model:** `claude-sonnet-5` via Header Auth (`anthropic-version: 2023-06-01`, `content-type: application/json`) — not to be confused with `claude-sonnet-4-6`, the previous generation.
