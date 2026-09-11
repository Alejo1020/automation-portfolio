# P19 — Automated Billing/ROI System

🇬🇧 English | [🇪🇸 Español](README.es.md)

## Problem

A service business running automation (lead capture, qualification, scheduling) generates operational results, but none of that data automatically translates into the one number that actually matters to the business owner: **how much money the system generated, and how much it cost to run.**

A client paying for automation doesn't want to hear "the chatbot answered 200 messages." They want to hear: "the system generated $X in closed deals, cost $Y to operate, ROI of Z%." That's the report that justifies the monthly fee — and the evidence that lets a freelancer raise prices with proof, not promises.

## Solution

An n8n workflow that turns raw closed-deal data into a plain-language financial report, delivered automatically on a schedule.

```
Schedule Trigger (weekly)
  → Get Rows (closed deals, logged in Google Sheets)
  → Code (sum revenue, subtract fixed operating cost, calculate ROI%)
  → HTTP Request → Claude Sonnet 5 (turns numbers into an executive summary)
  → Edit Fields (extracts Claude's text + rebuilds data via node reference)
  → Append Row (ROI history log)
  → Gmail (sends the HTML report to the business owner)
```

Deals are logged manually in a Google Sheet rather than pulled from a CRM — a deliberate scope decision to ship a working system now, with a clean path to plug in a CRM (e.g. HubSpot) later without touching the rest of the flow.

## Business Result

- Fully automated report once a deal is logged — zero manual reporting work after that
- Tested with real numbers: $1,650 in revenue, $50 in operating cost, 3,200% ROI, 2 deals
- Claude Sonnet flagged a real business risk on its own — 100% revenue concentration in a single client — something a cheaper model (Haiku) did not surface in the same test
- Builds a searchable, dated history of results in Sheets — a track record you can show a client at any time

*"On top of automating your lead capture, you get a weekly financial report: how much the system generated, how much it cost to run, and a plain-language recommendation — not a technical dashboard nobody reads. You can see at any moment whether the automation is paying for itself, and I have hard evidence to keep optimizing your account."*


## Tech Stack

n8n · Claude API (Sonnet 5) · Google Sheets · Gmail

## Key Technical Learnings

1. **`WEBHOOK_URL` also breaks OAuth, not just webhooks.** n8n uses that same environment variable to build the redirect for any public callback, including Google OAuth2 — even though OAuth doesn't need to be exposed to the internet. If the variable points to a tunnel URL and a credential's token expires, re-authentication fails with `redirect_uri_mismatch`, because that tunnel URL was never registered in Google Cloud Console. Permanent fix: set `N8N_EDITOR_BASE_URL=http://localhost:5678` on the container — this separates `WEBHOOK_URL` (external webhooks only, e.g. Twilio/Telegram) from the editor/OAuth flow (always localhost). Confirmed working when the startup log shows `Editor is now accessible via: http://localhost:5678`.
2. **Scope decision, not a shortcut:** shipping with manual Google Sheets entry instead of waiting to master a CRM integration was the right call — a simple system working today beats a perfect one still being learned.
3. Reconfirms a known n8n pattern: `$json` resets after any HTTP Request node. Recovered prior data here with `$('Code').item.json.field` after the call to Claude.
4. A direct Haiku-vs-Sonnet comparison on the same task confirmed the model-selection rule: Sonnet for client-facing content where insight matters, Haiku only for simple classification.
