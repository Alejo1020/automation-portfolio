# P5 — Weekly Social Content Generator

🇪🇸 [Leer en español](./README.es.md)

## Problem

Small businesses know they need to stay active on social media, but rarely have the time or discipline to produce content consistently. The result: abandoned accounts, lost visibility, and inconsistent brand presence.

## Solution

An n8n workflow that automatically generates 30 days of ready-to-publish social content (2 posts/day: Instagram + LinkedIn) from a pre-loaded content calendar, with zero manual work once it's set up.

**Flow:**

```
Schedule Trigger (daily)
        │
        ▼
Get row(s) in sheet  ──> matches today's date against the content calendar
        │
        ▼
HTTP Request (Gemini) ──> generates IG + LinkedIn post drafts as JSON
        │
        ▼
Code (JS)             ──> parses the nested JSON string from Gemini
        │
        ▼
Update row in sheet   ──> writes both posts + status back, matched by date
```

1. **Schedule Trigger** — fires once a day at a fixed hour.
2. **Get row(s) in sheet** — looks up the row in Google Sheets where `Fecha` (date) matches today, using date-matching instead of row number so the flow stays in sync even if a day is skipped.
3. **HTTP Request → Gemini API** (`gemini-flash-latest`, free tier) — sends the day's topic and returns two drafts (casual IG post, professional LinkedIn post) as structured JSON.
4. **Code** — parses the nested JSON string Gemini returns and builds a clean object.
5. **Update row in sheet** — overwrites that day's row with `Post_IG`, `Post_LinkedIn`, and `Estado = Generado`, matched by date.

## Business Result

- 30 days of content across 2 platforms = 60 pieces generated with zero daily manual work.
- Estimated time saved: ~5–8 hours/month a small business would otherwise spend writing posts.
- No human intervention needed once the topic calendar is loaded.


## Tech Stack

n8n  · Google AI Studio · Google Sheets

## Key Technical Learnings

1. **Match by a real column (date), not row number** — same pattern used in the lead qualifier project (P4); prevents desync if the flow is paused or rows are reordered.
2. **Date format must match exactly** between the Sheet and `$now.format('yyyy-MM-dd')` — a mismatch silently returns zero rows instead of a clear error.
3. **"Latest" model aliases can resolve to versions you don't expect** — `gemini-flash-latest` resolved to `gemini-3.6-flash` in testing. Always check `modelVersion` in the response to confirm what's actually running.
4. **Gemini's response is a JSON string nested inside JSON** — `candidates[0].content.parts[0].text` needs a second `JSON.parse()` to extract the actual post content.
5. **"Using JSON" mode for headers/body** in the HTTP Request node is faster and less error-prone than field-by-field configuration when the payload has nested structure.
