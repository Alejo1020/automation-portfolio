# P17 — Multi-Agent System with Memory & Handoff

🇬🇧 English | [🇪🇸 Español](./README.es.md)

## Problem

A service business (clinic, dealership, law firm) gets messages that need very different kinds of attention: pricing questions, appointment scheduling, or complaints. A single generic chatbot handles this poorly — it doesn't remember context across topics and can't take real action (book, modify, cancel).

Manual handling doesn't scale and is expensive. A multi-agent system solves this the same way a human team would: a "receptionist" that routes the conversation, and specialists that handle each type of request with their own judgment and tools.

## Solution

**Architecture:**
```
Telegram Trigger → Normalize Input → Orchestrator (Claude Haiku 4.5, with memory)
    ↓
Switch (handoff based on classification)
    ├── Sales Agent (Claude Sonnet 5, memory)
    ├── Scheduling Agent (Claude Sonnet 5, memory + 3 tools: read / create / update appointment in Sheets)
    └── Support Agent (Claude Haiku 4.5, memory, escalates to human)
    ↓
Telegram Send Message (unified response to client)
```

**Key pieces:**
- **Orchestrator** classifies each message into `ventas` / `agendamiento` / `soporte` using conversation history, without acting as a conversational assistant itself (explicit output restriction in the system prompt).
- **Scheduling Agent** with 3 Google Sheets tools: it first checks whether the client already has an appointment (Get Rows), then decides between creating (Append) or updating (Update Row) — it never assumes.
- **Per-user memory**: each of the 4 agents uses `Simple Memory` with Session Key = `usuario_id` (Telegram chat.id), so every client has an isolated context thread.
- **Mixed model strategy by cost/complexity**: Haiku 4.5 for classification and support (simple tasks), Sonnet 5 for sales and scheduling (reasoning and tool use).

## Business Result

- A client can book, change their mind, and modify their appointment **within the same conversation**, without repeating data — the system remembers an existing appointment and updates it instead of duplicating it.
- The bot never fabricates a false confirmation: if a tool call fails, it admits it and offers an alternative (verified behavior during testing).
- Channel-agnostic architecture: it runs on Telegram today; migrating to WhatsApp/Twilio is a swap of a single input/output node — the agent logic doesn't change.


*"A WhatsApp/Telegram attention system with 3 AI specialists (sales, scheduling, support) that hand off the conversation to each other without the client repeating anything — and that books or modifies appointments directly in your system, not just answers questions."*


## Tech Stack

n8n · Claude API (Sonnet 5 + Haiku 4.5) · Telegram Bot API · Google Sheets · Cloudflare Tunnel

## Key Technical Learnings

- **`.item` vs `.first()`:** after a `Switch` node (or any node that branches/breaks the linear flow), `.item` position-based linking can fail. Use `$('NodeName').first().json.field` in any node downstream of a Switch — including Memory Session Keys and Agent prompts.
- **Memory nodes are sub-nodes**, not part of the main data flow — they are especially sensitive to the `.item` issue above.
- **An orchestrator with memory can start "acting" instead of classifying** if its context includes conversational turns from other agents — requires an explicit, hard output restriction in the system prompt, not just a single instruction.
- **A tool-using agent can hallucinate a successful action** (e.g. claim "I replaced your appointment") if it doesn't actually have the right tool available — the real fix is providing the missing tool and enforcing a strict read → decide → act flow, not just prompt tweaking.
- **Google Sheets Tool — Update Row** requires `columns.matchingColumns` to be explicitly set — selecting the column in the visual picker alone isn't always enough.
- **Telegram Trigger requires `WEBHOOK_URL`** as an environment variable on the Docker container, pointing to the active public tunnel URL — every time the Cloudflare Tunnel URL changes, the container needs to be recreated with the new URL.
- **Set `appendAttribution: false`** on the Telegram Send Message node to remove the "sent automatically with n8n" footer.

## Known Limitation (documented, not hidden)

At the time of this write-up, the Sales Agent branch still has a couple of unresolved issues from the original build — its memory Session Key expression and prompt text weren't updated with the same `.item` → `.first()` fix applied to the other branches. This is called out here deliberately, as part of demonstrating debugging depth rather than presenting a "perfect" system.
