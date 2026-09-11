# P12 — AI-Powered Business Dashboard That Interprets Metrics and Recommends Actions

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

Service business owners (clinics, dealerships, real estate agencies) collect weekly metrics — leads, sales, revenue, marketing spend, customer satisfaction, support tickets — but most don't have the time or analytical expertise to interpret them. A spreadsheet full of numbers doesn't tell them what to do. The result: slow decisions based on gut feeling instead of data, and problems that go unnoticed until they've already hurt revenue.

## Solution

An automated weekly system that reads historical business metrics, has an LLM reason over the full trend (not just the latest data point), and delivers a plain-language diagnosis with concrete next actions — straight to the owner's inbox.

**Architecture:**

```
Schedule Trigger (Mon 8am)
      │
      ▼
Google Sheets — Get 8 weeks of metrics
      │
      ▼
Aggregate — merge all rows into one array
      │
      ▼
HTTP Request — Claude API (Sonnet)
  Analyzes the full trend, returns structured JSON:
  { diagnostico, alerta_principal, recomendaciones[] }
      │
      ▼
Edit Fields — extract text from Claude's response
      │
      ▼
Code — parse JSON string into separate fields
      │
      ▼
Google Sheets — append to analysis history log
      │
      ▼
Gmail — send formatted HTML report to owner
```

**How it works:**
1. Every Monday at 8am, the workflow pulls the last 8 weeks of business metrics from Google Sheets.
2. All rows are aggregated into a single array so the AI can detect *trends*, not analyze isolated weeks.
3. Claude API receives the full dataset and returns a structured diagnosis: overall trend, the single most urgent issue, and 3 concrete actions — as clean JSON, no filler text.
4. The response is parsed and logged to a running history sheet, building a record of AI-driven consulting over time.
5. A formatted HTML email delivers the analysis directly to the business owner — no dashboard to open, no numbers to interpret.

## Business Result

Using 8 weeks of realistic service-business data, the system identified a non-obvious correlation that a standard dashboard would miss: drops in customer satisfaction and spikes in support tickets (weeks 5 and 8) consistently preceded sales dips the following week. This kind of cross-variable, trend-based insight requires reasoning across multiple metrics simultaneously — exactly what a static chart or table cannot surface on its own.

*"This isn't a dashboard — it's a business analyst that never sleeps. Every week, without lifting a finger, the owner gets what's happening in their business, what's most urgent, and what to do about it — the same caliber of reasoning they'd pay a consultant for, delivered automatically and consistently."*

## Tech Stack

- n8n (workflow orchestration)
- Claude API (Claude Sonnet — analysis and recommendations)
- Google Sheets (data source + analysis history log)
- Gmail (report delivery)

## Key Technical Learnings

- **HTTP Request body with a JSON expression:** when the request body is built with `JSON.stringify()` inside an n8n expression, `Specify Body` must be set to **"Using JSON"**, never "Using Fields Below." The latter treats the entire stringified expression as a single field *name*, silently breaking the request with no clear error.
- **Header Auth credentials only support one header pair.** Additional required headers (`anthropic-version`, `content-type` for the Anthropic API) must be added manually in the node's "Send Headers" section — they can't live inside the credential itself.
- **`$json` resets after an HTTP Request node.** Data from upstream nodes isn't directly accessible after the call returns; the response must be extracted from the current `$json` (e.g., `$json.content[0].text` for Claude's response format), not referenced via `$('Previous Node')`.
- Claude's response format (`content[0].text`) differs structurally from Gemini's (`candidates[0].content.parts[0].text`) — worth keeping a reference note when switching between providers across projects.
