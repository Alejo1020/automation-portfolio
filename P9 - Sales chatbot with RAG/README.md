# P9 — Sales Chatbot with RAG

🇪🇸 [Leer en español](./README.es.md)

## Problem

Service businesses with high volumes of repetitive customer questions (prices, hours, treatments, location) lose staff time answering the same questions over and over on WhatsApp — and slow response times cost leads, especially outside business hours.

## Solution

A WhatsApp chatbot that answers customer questions using Retrieval-Augmented Generation (RAG) grounded in a live knowledge base, so the business owner can update pricing/hours without touching any code.

**Architecture:**

```
WhatsApp message (Twilio)
        │
        ▼
   [ Webhook ]
        │
        ▼
[ Google Sheets ]  ← knowledge base (treatments, prices, hours, location, FAQ)
        │
        ▼
   [ Aggregate ]  ← bundles all rows into one context block
        │
        ▼
[ Gemini / LLM ]  ← answers ONLY from the provided context; defers to a
        │            human advisor if the answer isn't in the data
        ▼
  [ Edit Fields ]  ← extracts answer + restores sender's phone number
        │
        ▼
[ Twilio API ]  ← sends the reply back on WhatsApp
```

The knowledge base lives in a single Google Sheet tab. No vector database is used — for a small, static catalog (10–20 rows), passing the full dataset as context on every request is simpler, cheaper, and easier for a non-technical business owner to maintain than managing embeddings.

## Business Result

- Instant response, 24/7, no staff time spent on repetitive questions
- Zero hallucinated prices or treatments — the model is constrained to the knowledge base and explicitly told to hand off to a human when it doesn't know
- Knowledge base is editable by the business owner directly in Sheets, no developer needed for updates


## Tech Stack

n8n · Twilio WhatsApp API · Google Sheets · Gemini API (Google AI Studio) · Cloudflare Tunnel

## Key Technical Learnings

- **RAG without a vector database:** for small, static catalogs, bundling all rows as one context block (via an Aggregate node) is simpler and cheaper than embeddings/vector search — the added complexity isn't justified until the dataset grows significantly.
- **Webhook auth mismatch:** the inbound Webhook node must have Authentication set to `None` when the caller is Twilio — Twilio doesn't send credentials on inbound requests, so Basic Auth silently rejects every message.
- **`+` gets lost in URL-decoding:** Twilio sends the sender's number as `x-www-form-urlencoded`, where `+` is interpreted as a space during decoding. Fix: `$json.body.From.replace(' ', '+')` before using the number downstream.
- **Every HTTP Request node needs its own credential explicitly selected** — it isn't inherited from other nodes even with the same auth type, which caused a silent 401 on the outbound Twilio call.
- **Stale webhook registration in self-hosted n8n:** an actively-published workflow can stop receiving real webhook traffic ("unknown webhook" in logs) after repeated node edits, even though the CLI reports it as active. Fix: delete and recreate the Webhook node from scratch, save, reactivate, and restart the container — a plain toggle or CLI reactivation isn't always enough.
