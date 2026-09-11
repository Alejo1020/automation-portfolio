#  P7 — Full Lead Gen System for AutoPremium Medellín (Fictional Dealership)

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

A car dealership receives leads through multiple channels (web form + WhatsApp) with no prioritization. Sales reps waste time manually reviewing every lead one by one instead of acting first on the ones ready to buy. Leads that aren't ready yet (warm/cold) get contacted once and then forgotten — no follow-up, no re-engagement. The owner has no daily visibility into lead volume or quality without digging through a spreadsheet.

## Solution

Two connected sub-systems, built as separate n8n workflows sharing the same Google Sheets backend.

### P7a — Capture + Qualification
```
Webhook (Form) ──┐
                  ├─→ Merge (Append) ─→ Gemini 3.6 Flash (score + category)
Webhook (WhatsApp)┘        ↓
                    Clean JSON → Parse → Save to Sheets
                            ↓
                    IF category = "hot" → WhatsApp alert to sales rep
```
Captures leads from two channels with different native formats (JSON from a web form, `x-www-form-urlencoded` from Twilio), normalizes both into a common schema, sends the message to Gemini for scoring (1–10) and categorization (cold/warm/hot), logs everything to Sheets, and fires an instant WhatsApp alert only for hot leads.

### P7b — Follow-up + Reporting
```
Schedule (9am daily) → Sheets → Filter (not hot, not yet contacted)
   → Gemini drafts a short follow-up message → WhatsApp → mark "contacted" in Sheets

Schedule (daily) → Sheets → Filter (today's leads)
   → Code node calculates metrics (totals, by category, by channel) → Email report to owner
```
Automatically re-engages cold/warm leads that were never followed up, and sends the dealership owner a daily HTML email summary without anyone opening the spreadsheet.

## Business Result

- Zero warm/cold leads abandoned without a follow-up attempt.
- Instant triage: sales rep is notified the moment a hot lead comes in, on either channel.
- Owner gets daily visibility (volume, quality breakdown, channel split) with zero manual reporting effort.
- Fully demonstrable end-to-end: capture → AI qualification → nurture → reporting.

## How This Sells

**Target client**: businesses with high lead volume and a small sales team, in verticals with high ticket value where losing a hot lead to slow response is expensive — car dealerships, real estate, legal services, clinics. Best fit: already using WhatsApp Business and some kind of web form.

**Pricing**:
- Setup: $300–500 USD — channel integration, qualification prompt tuned to the client's business, connection to their Sheets/CRM.
- Recurring: $80–150 USD/month — maintenance, prompt tuning, uptime monitoring, support.
- API cost note: production version uses Claude API (not the Gemini free tier used during development), priced separately or passed through.

**Gap to sell today**: needs a stable domain + Named Tunnel (not a rotating Quick Tunnel URL) before demoing to a real client.

## Tech Stack

n8n · Gemini 3.6 Flash API (dev/testing) · Claude API (production target) · Twilio WhatsApp · Google Sheets · Gmail · Cloudflare Tunnel

## Key Technical Learnings

1. **Multi-channel ≠ same format.** Each source sends data however it wants: your own form → JSON (you control it), Twilio → `x-www-form-urlencoded` with fixed fields (you adapt to their standard). Rule: *if you own the emitter, you choose the format; if it's a third party, you match theirs.*
2. **Normalize before merging.** Two different payload shapes can't go straight into a `Merge` node — each channel needs its own `Edit Fields` step translating it into a common schema before converging.
3. **`Merge` with Append, not index-based combination**, when inputs arrive unsynchronized in time — Append simply stacks items regardless of arrival order.
4. **Real production bug, not a lab one**: a duplicated `+57` prefix (`+57+573205020526`) happened because each channel left the phone number in a different format before reaching the same downstream node. Fix format inconsistencies at the normalization step, not with a patch at the final node.
5. **A disabled node silently passes all input through** — it doesn't block anything. A disabled `If` looks like a working filter until you check its actual output; always verify node status when logic that "should" filter isn't filtering.
6. **LLM output is text, not data.** Cleaning markdown backticks + `JSON.parse()` is what turns model output into usable fields — but when you just need prose (like a follow-up message), skip the parsing step entirely and ask for plain text.
7. **`$('NodeName').item.json`** lets you pull fields from any earlier node in the flow, not just the one directly upstream — useful once a chain includes an AI call in between.
8. **Deterministic math doesn't need an LLM.** Daily metrics (counts, breakdowns) are calculated with a plain `Code` node — no reason to spend AI tokens summing numbers.
9. **A single workflow can host two independent `Schedule Trigger` branches** that never interact — clean way to keep related automations (follow-up + reporting) in one JSON without them interfering.
