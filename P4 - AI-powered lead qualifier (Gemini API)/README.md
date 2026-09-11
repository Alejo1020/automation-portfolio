# AI-Powered Lead Qualifier (Gemini API)

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

Sales teams waste time manually reviewing every inbound lead from WhatsApp or web forms to decide who to call first. Hot leads go cold while sitting in the same queue as cold, low-intent ones.

## Solution

An n8n workflow that receives a lead via webhook, uses AI to analyze the message and classify it (score 1-10, cold/warm/hot, with a reason), logs every lead to Google Sheets, immediately returns the qualification result to the webhook caller, and sends a WhatsApp alert only when the lead is hot.

**Architecture:**

```
Webhook (receive lead)
  → HTTP Request (Gemini API qualifies the lead)
  → Edit Fields (strip markdown from AI response)
  → Code (parse JSON, structure lead + score data)
  → Append row in sheet (log to Google Sheets)
     ├─→ Respond to Webhook (return result to caller)
     └─→ If (categoria == "caliente")
           └─→ HTTP Request (Twilio WhatsApp alert)
```

The webhook always responds immediately with the qualification result, regardless of lead temperature. The WhatsApp notification runs in parallel and only fires for hot leads — this avoids blocking the response while also avoiding unnecessary notification costs.

## Business Result

- Response time to hot leads: hours → minutes
- 100% of leads qualified with zero manual review
- Zero hot leads lost to slow follow-up
- Zero wasted WhatsApp notifications on cold/warm leads

## How This Sells

- One-time setup fee to build and integrate the system with the client's existing CRM/WhatsApp/form.
- Monthly maintenance fee for prompt tuning, monitoring, and support.
- Pitch: *"Your sales team stops reviewing leads one by one — the system pings you on WhatsApp the moment a hot lead comes in, with the exact reason why."*
- Target clients: real estate agencies, car dealerships, and any Meta Ads-driven business with high lead volume.

## Tech Stack

n8n (self-hosted) · Gemini API (free tier, testing phase) · Google Sheets · Twilio WhatsApp Sandbox

## Key Technical Learnings

- AI model names change without notice — always verify the current model is still available before assuming a documented name works.
- The order of the "Respond to Webhook" node matters. Connecting it too early in the flow returns raw/unprocessed data to the caller. It must sit after the data is clean, and before any conditional branch (like an IF leading to WhatsApp) that doesn't always execute — otherwise the webhook can time out for leads that don't take that branch.
- A single node can fan out to multiple nodes in parallel — "Append row in sheet" connects simultaneously to "Respond to Webhook" and "If", so the response to the caller never waits on the conditional WhatsApp branch.
- Twilio requires "From" and "To" to share the same channel prefix (`whatsapp:`) — mismatched channels return error 21910.
- "Retry On Fail" is essential for external AI APIs — Gemini's free tier saturates under load, and without automatic retry, any demand spike breaks the flow in production.
- Testing both branches of a conditional (not just the "happy path") matters — the webhook timeout bug only showed up with cold leads, not hot ones.
