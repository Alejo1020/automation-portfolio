🇪🇸 [Leer en español](README.es.md)

# Clínica Renova — End-to-End Service Business Automation

**Type:** AI-powered automation system (n8n + Claude Sonnet 5)
**Client type:** Aesthetic/medical clinics, dental practices, service-based businesses
**Complexity:** High — 6 integrated blocks, full lead lifecycle

---

## Problem

Service businesses like aesthetic clinics lose revenue in the gaps between systems: leads come in through WhatsApp or a web form and nobody qualifies them in time, appointments get booked incorrectly or double-booked, warm leads go cold from lack of follow-up, and the owner only finds out about the real state of the business — revenue, ROI, system failures — after it's too late, with no daily visibility or automatic alerts.

## Solution

A single system with 6 integrated blocks that orchestrates the entire lead lifecycle end-to-end, without manual intervention:

1. **Lead Entry + Qualification** — Two channels (WhatsApp via Twilio + web form) converge into a normalized flow that Claude Sonnet 5 automatically scores (score, category, reasoning).
2. **Scheduling** — Hot/warm leads are routed to an AI Agent with its own tools (check calendar, book appointment) that proposes and confirms a time slot without double-booking, respecting real business hours.
3. **Nurturing** — Cold leads receive an AI-generated value message (no hard sell) via WhatsApp, with follow-up tracked.
4. **Daily Reporting** — Every morning at 8am, the owner receives an email with yesterday's metrics and an AI executive summary that flags what needs attention (e.g., hot leads with no appointment booked).
5. **Weekly Billing/ROI** — Every Monday, the system calculates revenue vs. operating cost, and Claude interprets the result in plain business language, with a probable cause and recommended action if ROI is negative.
6. **Error Handling** — An independent workflow automatically detects any failure in any block and sends an immediate email alert, so no lead falls through the cracks due to a silent error.

## Architecture

```
Block 1: Entry + Qualification
  WhatsApp Webhook ─┐
                     ├─> Merge → Normalize → Claude scores lead → Save to Sheet
  Web Form Webhook ─┘

Block 2: Scheduling (hot/warm leads)
  AI Agent (Claude Sonnet 5 + tools: check calendar, book appointment)

Block 3: Nurturing (cold leads)
  Claude generates value message → WhatsApp send → Log to Sheet

Block 4: Daily Reporting (Schedule Trigger, 8am)
  Read 3 Sheets → Merge → Consolidate metrics → Claude summary → Email

Block 5: Weekly Billing/ROI (Schedule Trigger, Monday 8am)
  Read appointments → Calculate ROI → Claude interprets → Email

Block 6: Error Handling (independent workflow)
  Error Trigger → Build alert message → Email
```

## Business Result

- Zero manual intervention between "a lead arrives" and "the owner gets an actionable report about their business"
- Daily and weekly visibility that most service businesses this size don't have today
- Self-monitoring system: if anything breaks, the owner knows within minutes, not days
- Reusable architecture: the same skeleton (qualify → schedule/nurture → report → bill → monitor) fits a law firm, real estate agency, dental clinic, spa — any appointment-based service business

## Sales Pitch

> "I'm not selling you a chatbot. I'm installing the nervous system of your business: every lead that messages you gets qualified automatically, gets scheduled if they're ready or nurtured if they're not, and every week a summary lands in your inbox telling you exactly what's working and what needs your attention — without you ever opening a spreadsheet."

Fits directly into a mid-to-high pricing tier ($1,200–2,500+/mo): it's the 4 core service modules (content/messaging/lead capture/dashboard) working as one integrated system, not sold piecemeal.

## Tech Stack

- **Orchestration:** n8n (self-hosted, Docker)
- **AI:** Claude Sonnet 5 (Anthropic API) — used for lead scoring, appointment scheduling reasoning, nurture message generation, executive reporting, and ROI interpretation
- **Channels:** Twilio (WhatsApp), Web form (webhook)
- **Data:** Google Sheets (leads, appointments, nurturing log, ROI history)
- **Notifications:** Gmail (daily report, weekly ROI report, error alerts)

## Key Technical Learnings

1. **Fixed vs. Expression toggle** — In any n8n text field (HTTP body, AI Agent system message), both the `fx` editor and the "Fixed | Expression" toggle must independently be set to Expression mode. Having one without the other silently sends literal, unevaluated strings.
2. **Empty sheet reads halt execution** — A Google Sheets "Get row(s)" node on an empty sheet returns 0 items and, by default, n8n stops the entire workflow there. Fix: enable "Always Output Data" in the node's Settings.
3. **AI Agents run once per input item** — Never feed a multi-row node (e.g., a full appointment list) directly into an AI Agent's main input — it triggers duplicate agent runs (and duplicate API costs) once per row. Data the agent needs to look up belongs in a Tool it calls on demand, not in the main chain.
4. **Exact node names matter** — n8n auto-numbers duplicate node names (e.g., "Edit Fields" vs. "Edit Fields1"). Referencing the wrong one causes a silent `undefined` with no visible error.
5. **Merge before consolidating parallel branches** — When 3+ parallel branches (e.g., multiple Sheets read in parallel) feed into a single Code node, always insert a Merge node (Append mode) first. Referencing each parallel node directly by name in the Code node is unreliable — only one branch may register as "executed."
6. **Prompts are iterative** — An AI prompt rarely works perfectly on the first try. Two to three refinement rounds after seeing real output (e.g., removing a follow-up question Claude added that doesn't make sense in an automated email) is normal, not a sign of failure.

---

*Built as part of a 29-project automation portfolio roadmap, moving from n8n fundamentals to multi-agent systems with real MCP protocol integration.*
