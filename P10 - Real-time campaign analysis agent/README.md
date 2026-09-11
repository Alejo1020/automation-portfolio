# P10 — Real-Time Campaign Analysis Agent

🇬🇧 English | [🇪🇸 Español](README.es.md)

## Problem

Business owners running ad campaigns don't have time to open dashboards or wait for a weekly report. They need answers the moment a question comes to mind — "how's campaign X doing?", "where am I wasting money?" — not on Monday when the report lands.

## Solution

A conversational AI agent that receives natural-language questions via webhook, autonomously decides whether it needs to query real campaign data, and replies in plain text ready for WhatsApp.

**Architecture:**

```
Webhook (POST /analisis-campanas)
        │
        ▼
   AI Agent ── Chat Model: Claude Sonnet 5
        │  └── Tool: Google Sheets (read campaign data)
        ▼
Respond to Webhook (plain text)
```

The agent is not a fixed linear flow: it reasons over the question and decides on its own whether to call the Google Sheets tool or answer directly, based only on the tool's description.

## Business Result

Validated with two different question types, no reconfiguration needed:

- **Ranking question** ("which campaign has the best CTR?") — agent computed CTR per campaign from raw clicks/impressions and ranked them correctly.
- **Optimization question** ("which campaign is wasting the most budget?") — agent derived cost-per-conversion (a metric not present in the raw data) and gave a concrete reallocation recommendation.

This demonstrates the agent generalizes across question types rather than answering a single hardcoded case.


## Tech Stack

n8n · Claude API (Sonnet 5) · Google Sheets · Cloudflare Tunnel

## Key Technical Learnings

1. The n8n AI Agent node (LangChain) always returns its final answer in the fixed `$json.output` field — not configurable.
2. Without explicit formatting instructions, Claude defaults to markdown (tables, bold) and sometimes exposes its intermediate reasoning ("wait, let me recalculate") in the final answer — both had to be explicitly restricted in the System Message for a WhatsApp-ready output.
3. The Tool Description is the actual control mechanism: the agent decides when to call a tool based on that text, not on the user's prompt — it needs to be precise and specific.
4. Claude Sonnet 5 is the right cost/reasoning tradeoff for tool-calling with multi-step decisions — confirmed in testing via visible self-correction on a CTR calculation.
5. With `Respond to Webhook` set to `Text`, the Webhook trigger node can later be swapped for a Twilio WhatsApp trigger without touching the rest of the flow — the agent doesn't know or care about the origin channel.
