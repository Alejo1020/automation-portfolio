# P20 — Voice Agent with Twilio + Claude

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

Businesses lose leads and sales every time a phone call goes unanswered — after hours, during peak times, or simply because the team is busy. Every missed call is a potential customer who likely calls a competitor next. Hiring 24/7 reception isn't viable for most small businesses.

## Solution

A voice agent that answers real phone calls around the clock:

- **Holds a natural conversation** in Spanish, powered by Claude Haiku 4.5 (chosen for low latency, critical in real-time voice).
- **Remembers context** within the same call (caller's name, what they already asked) using Google Sheets as a lightweight history store, keyed by `CallSid`.
- **Decides its own next action** each turn: keep talking, say goodbye and hang up, or transfer to a human — all from a single Claude call that returns structured JSON (`{texto, accion}`).
- **Built entirely on n8n + Twilio + Claude**, no extra telephony infrastructure required.

### Architecture

```
Call comes in → Webhook 1 → Say + Gather (greets, listens)
     ↓ (caller speaks, Twilio transcribes)
Webhook 2 → Read history (Sheets) → Build messages → Claude Haiku
     ↓
Parse {texto, accion} → Build TwiML per action → Save turn to Sheets → Respond to Twilio
     ↓
continue → loops back to listening | say goodbye → hangs up | transfer → dials a human
```

**Why Twilio + n8n instead of a dedicated voice AI platform:** full control over the conversation logic, no per-minute platform markup beyond Twilio's own rates, and it plugs directly into the same n8n/Claude stack used across the rest of this portfolio.

## Business Result

Live test call verified end-to-end: the agent greeted the caller, understood a spoken question about business hours, and — in a later turn — correctly recalled the caller's name without being told again, confirmed in the conversation log saved to Google Sheets.

## How This Sells

*"Your business never misses a call again. This agent answers 24/7, understands what callers are asking, remembers the context of the conversation, and knows when to hand the call to you because the customer is ready to close — or when to wrap up on its own because the question's been answered. It's not a numeric IVR menu: it's a real conversation."*

**Target client:** service businesses with call volume that can't justify full-time reception — clinics, auto shops, real estate agencies, small law firms.

**Pricing model:** setup fee + monthly rate that includes Twilio call minutes (passed through to the client, not absorbed into margin).

## Tech Stack

n8n (self-hosted) · Claude API (Haiku 4.5) · Twilio Programmable Voice · Google Sheets · Cloudflare Tunnel

## Key Technical Learnings

1. **Twilio expects TwiML (XML), not JSON** as the webhook response — set `Respond With: Text` (never `JSON`) plus an explicit `Content-Type: text/xml` header, or Twilio throws `Invalid JSON in Response Body`.
2. **Conversation loop architecture:** two webhooks (call entry + transcription) ping-ponging until a hangup or transfer action ends the loop.
3. **Per-call memory without a Memory node:** Google Sheets filtered by `CallSid` as the session key — same conceptual pattern as an n8n Memory node, built manually for full control over what gets stored.
4. **Empty Google Sheets reads don't propagate by default** in n8n — `Always Output Data` must be enabled on the read node, and the resulting empty item filtered out downstream, or the next HTTP Request to Claude fails with `messages.0.role: Field required`.
5. **Escaping quotes in a system prompt embedded in a JSON body:** internal double quotes must be escaped with `\"` or the node's own JSON parsing breaks before the request ever reaches the API.
6. **Twilio trial account constraints:** free, no credit card, but limited to 5 verified destination numbers — this restricts testing real `<Dial>` transfers to a third party without a second verified number.
