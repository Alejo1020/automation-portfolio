# Real-Time Lead Notification via WhatsApp

🇪🇸 [Read in Spanish / Leer en español](README.es.md)

## Problem

Businesses that capture leads through web forms (real estate agencies, dealerships, clinics, law firms) often take hours — sometimes days — to follow up, because someone has to manually check an inbox or CRM. Response-time studies consistently show that contacting a lead within the first 5 minutes dramatically increases conversion rates, while response after 30 minutes causes conversion to drop sharply.

## Solution

An automation that receives any incoming lead (web form, landing page, Facebook Ads) via webhook and instantly notifies the assigned sales rep over WhatsApp — with the lead's name, phone number, message, and source — ready to call within seconds.

**Architecture:**

```
[Webhook] → [Edit Fields] → [HTTP Request: Twilio WhatsApp API]
   ↓               ↓                      ↓
Receives      Builds the           Sends the WhatsApp
lead data     WhatsApp text        notification
```

1. **Webhook** — captures the incoming lead payload (name, phone, message, source).
2. **Edit Fields** — formats the data into a single WhatsApp-ready message and isolates the destination phone number.
3. **HTTP Request** — calls the Twilio API to send the WhatsApp message to the sales rep in real time.

## Business Result

- Lead response time: hours → seconds
- Zero leads lost to "forgot to check the inbox"
- Scales from 1 to 100+ leads/day with zero added manual work

*"How many leads are you losing because no one reached out in time? This system alerts your sales team over WhatsApp the second a new lead comes in."*


## Tech Stack

- n8n (self-hosted, Docker)
- Twilio WhatsApp API (sandbox for testing / WhatsApp Business API for production)
- Webhook-based trigger (compatible with any form or landing page provider)

## Key Technical Learnings

- **WhatsApp sandbox restriction:** Twilio's sandbox can only deliver messages to numbers that have manually sent the `join <code>` message first. This is a testing-only limitation — production requires a verified WhatsApp Business API number to message any recipient freely.
- **Body Content Type matters:** Twilio's API expects `Form-Urlencoded`, not JSON. Sending JSON causes silent failures ("To phone number is required") even when the field is filled in the node UI, if "Send Body" is toggled off.
- **Intermittent DNS resolution (`ENOTFOUND api.twilio.com`):** resolved by flushing the local DNS cache and switching to Google's public DNS (8.8.8.8 / 8.8.4.4) — a local network issue, not an n8n or Twilio configuration problem.
