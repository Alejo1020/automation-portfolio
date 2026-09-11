🇬🇧 English | [🇪🇸 Español](./README.es.md)

# P6 — Meta Ads Campaign Analyzer

n8n workflow that reads Meta Ads campaign metrics, calculates real efficiency indicators (CTR, CPA), and uses AI to generate a comparative analysis with actionable recommendations — delivered automatically by email.

## Problem

Business owners running Meta Ads campaigns track total spend and lead count, but rarely calculate CPA or CTR per campaign — and almost never compare campaigns side by side on a weekly basis. The result: budget keeps flowing into underperforming campaigns simply because nobody has the time (or the analytical habit) to catch it.

## Solution

**Architecture:**

```
Manual Trigger (→ Schedule Trigger in production)
        ↓
Google Sheets — Read campaign data (simulates Meta Ads API)
        ↓
Edit Fields — Calculate CTR and CPA per campaign
        ↓
Aggregate — Merge all campaigns into one array
        ↓
HTTP Request — Gemini analyzes all campaigns together
        ↓
Edit Fields — Extract clean analysis text + date
        ↓
Google Sheets — Append to history log
        ↓
Gmail — Send formatted HTML report
```

Each week, the workflow pulls campaign data, computes the efficiency metrics that actually matter (not just raw spend), and sends everything to an AI model in a single batched call so it can reason across campaigns — identifying which one is winning, which one is burning budget, and why. The analysis is logged to a running history sheet and delivered as a styled HTML email, ready to forward to a client.

## Business Result

In the test scenario (5 simulated campaigns for a car dealership), the analysis flagged a **4.6x difference in cost-per-acquisition** between the best and worst performing campaigns ($14,545 vs $67,778). Reallocating that budget toward the efficient campaign represents the potential to roughly double lead volume without any additional spend.


## Tech Stack

n8n (self-hosted) · Gemini API (free tier, testing only) · Google Sheets · Gmail

## Key Technical Learnings

- **Aggregate before comparison:** to get the AI to compare items against each other (not analyze them one by one), they need to be merged into a single array first. Skipping this step means the model loses all cross-campaign context.
- **Nested JSON.stringify in expressions:** mixing literal JSON quotes with `{{ }}` expressions breaks the parser. The robust fix is building the entire body as a JS object and serializing it once at the end (`JSON.stringify({...})`) instead of hand-assembling JSON as text.
- **Free-tier rate limits are real and low:** 20 requests/day on Gemini Flash. Every syntax error still counts as a used request — precision before execution matters more than trial-and-error iteration.
- **Model naming changes fast:** always verify the current model version before hardcoding it; free-tier model aliases get deprecated or renamed without much notice.
- **Keep credentials out of the JSON:** use Header Auth credentials instead of embedding an API key in the URL. Any JSON you share (portfolio, GitHub, client) should be safe to distribute without leaking account access.
- **The derived metric is the product, not the raw data.** Anyone can export spend and conversions from Meta Ads. The value sold here is CTR/CPA calculation + reasoned comparison + actionable recommendation — the part a business owner doesn't have the time or analytical skill to do themselves.
