# P8 — Weekly Content Pipeline for Law Firm (Fictional Client)

🇬🇧 English | [🇪🇸 Español](README.es.md)

## Problem
Professional service firms (legal, accounting, consulting) need consistent social + blog presence to generate organic leads, but producing quality content weekly eats hours from a partner or requires hiring a writer. Result: firms either don't publish, or publish generic content with no strategy.

## Solution
n8n pipeline that automates weekly content generation from an editorial calendar:
- Schedule Trigger fires every Monday 8am
- Reads pending topics from a Google Sheet (Estado column empty)
- For each topic, a single Gemini call generates 3 content pieces: Instagram post, LinkedIn post, and an 800-1000 word SEO article
- Parses and cleans the AI response
- Updates the Sheet with generated content, marking Estado = "Generado"

**Architecture:**
```
Schedule Trigger → Get rows (pending topics) → Prepare Topic (Code)
→ Gemini API (HTTP Request) → Clean response (Set) → Parse + build row (Code)
→ Update Sheet
```

## Business Result
- 12 topics loaded → 12 weeks of content (36 pieces) generated from 12 API calls
- Live test: 2/12 rows processed with zero errors, coherent and publish-ready output
- Estimated time saved: manual generation of this content would take 1-2 hours per topic; the pipeline does it in seconds

"Load your topics once a month — the system delivers 3 ready-to-review content pieces per topic, every week, without anyone on your team writing from scratch."


## Tech Stack
n8n · Google Gemini API (Flash) · Google Sheets · JavaScript (Code nodes)

## Key Technical Learnings
- Requesting all 3 content pieces in a single Gemini prompt (one JSON response) instead of 3 separate calls saves API quota and keeps messaging consistent across channels.
- n8n's Google Sheets filter has no native "is empty" operator — leaving the filter Value field blank achieves the same result.
- With multiple Code nodes in one workflow, explicit renaming (e.g. "Preparar Tema") is required to reference a specific node's output unambiguously via `$('NodeName')`.
- Matching rows by a business key (Fecha) rather than row_number is more resilient to row insertions/deletions — same pattern used in P3/P5.
