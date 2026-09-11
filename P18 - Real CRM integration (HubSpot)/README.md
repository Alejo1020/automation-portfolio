# P18 — Real CRM Integration (HubSpot)

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

By project 17, the lead pipeline still lived in Google Sheets. That's fine for a demo, but no serious client is going to move their sales operation out of their real CRM (HubSpot, Salesforce, Pipedrive). If an automation can't talk to the CRM a business already runs on, the pitch dies right there.

## Solution

**Flow:** Webhook → Claude Haiku qualifies the lead → parse/sanitize response → search contact in HubSpot by email → create or update contact → if the lead is hot, create a Deal in a custom pipeline stage, associated to the contact → respond to the webhook with the outcome.

```
[Webhook: new lead]
        ↓
[Claude Haiku 4.5] → scores lead (1-10), classifies cold/warm/hot
        ↓
[Parse response] → sanitizes Claude's JSON, merges with original lead data
        ↓
[HubSpot: Search Contact by email]
        ↓
   ┌────┴────┐
 exists?   new?
   ↓          ↓
[Update]   [Create]
   └────┬────┘
     [Merge]
        ↓
[IF: is lead hot?]
   ┌────┴────┐
  yes         no
   ↓          ↓
[Create Deal] │
associated to  │
the contact    │
   └────┬─────┘
        ↓
[Respond to Webhook]
status, category, score, deal_created
```

A lead enters through a webhook (WhatsApp, a form, any external source), gets scored by Claude, and lands in HubSpot as a Contact — updated if it already existed, created if not. If the lead is qualified as "hot," a Deal is automatically created in a custom pipeline stage ("Hot Lead - New"), already linked to the contact, ready for a sales rep to work without touching a spreadsheet.

## Business Result

100% of incoming leads land in the CRM with zero manual entry. Hot leads (score 8-9+ in testing) generate a sales opportunity in the pipeline within seconds of the lead coming in, instead of hours or days of manual triage and data entry.

*"I connect your lead capture system — WhatsApp, a form, whatever you use — directly to your CRM, with zero copy-pasting. AI qualifies the lead instantly, and if it's a real opportunity, it shows up in your sales pipeline ready to work."*

## Tech Stack

n8n · Claude API (Haiku 4.5) · HubSpot API (Private App) · Cloudflare Tunnel (webhook exposure)

## Key Technical Learnings

- **HubSpot pipeline stage IDs are not the visible label.** They're generated when the stage is created — fetch them via `GET /crm/v3/pipelines/deals` or read them off the settings UI after saving.
- **Deal↔Contact association via API** uses `associations: [{ to: {id}, types: [{associationCategory: "HUBSPOT_DEFINED", associationTypeId: 3}] }]` inside the same POST that creates the Deal. Note: the `num_associated_contacts` field in the API response can show `0` even when the association was saved correctly — verify in the HubSpot UI, not just the response body.
- **Claude sometimes wraps JSON in markdown** (```json fences) despite an explicit instruction not to. Always sanitize with `.replace(/```json/g, '').replace(/```/g, '').trim()` before `JSON.parse()` — never fully trust the prompt instruction alone.
- **Merge node in "Append" mode** (not "Choose Branch") is the correct way to unify two branches of an IF where only one branch executes per run.
- **After a Merge with converging conditional branches**, prefer explicit node references (`$('NodeName')`) over generic `$json` before a final response node — `$json` there depends on which branch actually ran last, which is easy to get wrong.
