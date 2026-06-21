# BUILD_GUIDE — AI Ops Assistant (agentic capstone)

Single source of truth. Build **in order**; each step has an acceptance check. The AI Agent uses
version-sensitive LangChain sub-node connections — verify node structure on import rather than
assuming.

---

## 1. What we're building

A hosted chatbot that answers questions about the business by calling tools against the live data
from the other three projects.

```
Chat Trigger (hosted webchat URL)
   -> AI Agent (Groq llama-3.3-70b)
        + memory (window buffer, keyed by chat session)
        + tools:
            get_leads      -> Supabase leads_public   (CRM: name, score, temperature, stage)
            get_content    -> Supabase content_public  (content packs + status)
            get_invoices   -> Google Sheet Invoice Ledger  (stretch)
            calculator     -> arithmetic
   -> grounded, multi-turn answer back to the chat
```

Example questions: "How many hot leads do we have?" · "What's our highest-scored lead?" ·
"How many content packs are pending approval?" · "Any invoices flagged for review?"

**Why this design (capstone):** it demonstrates the missing pillar — **agentic, tool-using,
memory-backed AI** — and ties the whole portfolio together: one assistant over three systems.

---

## 2. Stack (100% free, no card)

| Concern        | Tool                                                  |
|----------------|------------------------------------------------------|
| Chat UI        | n8n Chat Trigger (hosted webchat)                    |
| Agent + reason | n8n AI Agent + Groq `llama-3.3-70b-versatile`        |
| Memory         | Window Buffer Memory (per chat session)             |
| Data tools     | Supabase sanitized views (anon, read-only) · Sheets  |

Everything reuses existing accounts. The agent reads **only** sanitized views (`leads_public`,
`content_public`) — never raw PII, never write paths.

---

## 3. Tools (the contract that matters most)

Each tool needs a precise **description** (the model reads it to decide when/how to call it):

| Tool | Backed by | Description given to the agent |
|---|---|---|
| `get_leads` | `GET /rest/v1/leads_public` (anon) | "Get qualified sales leads with first name, score (0-100), temperature (hot/warm/cold), stage, date. Use for any question about leads/pipeline. Supports filtering by temperature." |
| `get_content` | `GET /rest/v1/content_public` (anon) | "Get generated content packs with topic, source, status (pending/ready/rejected), date. Use for questions about content/marketing output." |
| `get_invoices` (stretch) | Google Sheets read of Invoice Ledger | "Get processed invoices with vendor, total, status (ok/review), issues. Use for questions about invoices/spend." |
| `calculator` | built-in | "Do exact arithmetic. Use whenever a question needs math." |

System prompt (essence): identity (Xenogliph Ops Assistant); "use ONLY data returned by tools —
never invent numbers; if a tool returns nothing, say so"; scope = leads, content, invoices; concise.

---

## 4. Build order (do in sequence)

> ✅ done · ⏳ in progress · ⬜ not started

1. ⬜ **Repo + docs scaffold** — CLAUDE.md, this guide, `.env.example`, `.gitignore`, README.
   *Accept:* repo initialized + pushed, `.env` ignored.
2. ⬜ **Slice 1 — agent skeleton:** Chat Trigger → AI Agent + Groq model + window memory + ONE
   tool (`get_leads`). *Accept:* in the n8n chat, "how many hot leads?" calls the tool and answers
   from live data; a follow-up ("and how many cold?") shows memory works.
3. ⬜ **Slice 2 — more tools:** add `get_content` + `calculator`; tighten the system prompt + tool
   descriptions so the agent routes correctly. *Accept:* a content question hits `get_content`; a
   math question uses the calculator; an out-of-scope question is declined gracefully.
4. ⬜ **Slice 3 (stretch) — invoices tool:** add a Google Sheets read tool over the Invoice Ledger.
   *Accept:* "any invoices in review?" returns real rows.
5. ⬜ **Polish:** README (pitch, architecture diagram, **live chat link**, demo GIF), exported
   workflow JSON, repo About/topics/pin, and a short note framing the 4-project portfolio.

Each slice: smallest working path → add failure handling → test with real questions → export +
document → report honestly.

---

## 5. Notes / open decisions

- **Channel:** n8n **Chat Trigger** (hosted webchat) for a live, recruiter-tryable demo with no
  frontend. Telegram is the fallback if a gated channel is preferred.
- **LangChain node uncertainty:** exact types/typeVersions for n8n 2.26.3
  (`@n8n/n8n-nodes-langchain.*` for agent / chatTrigger / lmChatGroq / memory / tools) and the
  `ai_*` connection wiring are verified on import. If a generated node mismatches, assemble/confirm
  the sub-nodes in the n8n UI (the AI Agent's + buttons make this reliable) and re-export.
- **Security:** read-only tools on sanitized views only; input cap on the chat; no write tools.
- **Memory:** window buffer (in-session) is enough for the demo; Postgres-backed memory (Supabase)
  is a possible upgrade for persistence across sessions.
