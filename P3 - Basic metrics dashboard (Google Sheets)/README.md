# P3 — Basic Metrics Dashboard (Google Sheets)

🇪🇸 [Leer en español](README.es.md)

## Problem

Small and mid-sized service businesses (agencies, dealerships, consultancies) often track sales activity manually in spreadsheets or not at all. Owners have no daily visibility into leads, closed sales, revenue, or conversion rate without asking someone to pull a report together — usually 15-20 minutes of manual work, repeated over and over.

## Solution

An n8n workflow that captures daily activity data and automatically consolidates it into a live Google Sheet — a "Resumen" (Summary) view that always reflects current totals, with zero manual math.

**Architecture:**

```
[Manual Trigger]
      ↓
[Generate Sample Data]  → simulates leads, sales, revenue (Code node)
      ↓
[Append Row]  → logs the record in "Metricas_Diarias" (raw historical log)
      ↓
[Get Rows]  → reads the full historical log back
      ↓
[Calculate Totals]  → sums/averages leads, sales, revenue, conversion rate (Code node)
      ↓
[Update Row]  → overwrites row 2 of "Resumen" (always-current dashboard snapshot)
```

Two Google Sheets tabs:
- **Metricas_Diarias** — append-only historical log, one row per run
- **Resumen** — single live row, always overwritten, acts as the "dashboard"

## Business Result

Removes the manual process of consolidating daily metrics entirely. The owner opens one sheet and sees total leads, closed sales, accumulated revenue, and conversion % — always current, with no admin time spent compiling it.


## Tech Stack

n8n (self-hosted, Docker) · Google Sheets API · JavaScript (Code nodes)

## Key Technical Learnings

- **Fixed vs. Expression mapping bug:** when manually mapping "Values to Update" fields in the Google Sheets Update node, each field defaults to a literal/fixed value. Copy-pasting or typing a value directly (e.g. "2") locks it as a hardcoded constant instead of a dynamic reference — every field must be explicitly set to Expression mode (`{{ $json.field }}`) to pull the real upstream data.
- **`row_number` as a match key:** using a fixed `row_number: 2` (not an expression) is intentional here — it's what turns the sheet into a "live dashboard" (always overwritten) instead of a historical log. This is a deliberate architecture choice, not a bug.
- **Two-sheet pattern:** separating raw log (append-only) from summary (always-overwritten) is a reusable pattern for any dashboard-style deliverable — keeps history intact while giving the client a single glanceable view.
