<h1 align="center">AI Ops Assistant</h1>

<p align="center">
  <em>One AI agent on top of everything. Chat with it; it queries your live systems.</em><br/>
  A tool-using, memory-backed agent that answers questions about leads, content, and invoices —
  by calling real data, never guessing.
</p>

<p align="center">
  <img alt="n8n" src="https://img.shields.io/badge/n8n-AI_Agent-EA4B71" />
  <img alt="Groq" src="https://img.shields.io/badge/Groq-llama--3.3--70b-000" />
  <img alt="Supabase" src="https://img.shields.io/badge/Supabase-read--only_tools-3ECF8E" />
  <img alt="agent" src="https://img.shields.io/badge/pattern-tools%20%2B%20memory-8b7cff" />
</p>

<!-- Live chat link + demo GIF slot — add once deployed: assets/demo.gif -->

---

## What it does

A hosted chat (n8n Chat Trigger) backed by an **AI Agent**. Ask it a question and it decides which
**tool** to call, fetches **live data**, and answers — with **conversation memory** so follow-ups
keep context.

- *"How many hot leads do we have?"* → calls the leads tool → answers from the CRM
- *"What content is pending approval?"* → calls the content tool
- *"Any invoices flagged for review?"* → calls the invoices tool
- *"What's the average score of this week's leads?"* → leads tool + calculator

**Why this design:** it's the capstone of a four-project portfolio — the **agentic, tool-using,
memory-backed** pillar — and it ties the other three systems together under one assistant.

## Architecture

```mermaid
flowchart TD
    U["💬 Chat Trigger<br/>(hosted webchat)"] --> AG["🧠 AI Agent<br/>Groq llama-3.3-70b"]
    MEM["🧵 Window memory<br/>(per session)"] -. ai_memory .-> AG
    LM["Groq Chat Model"] -. ai_languageModel .-> AG
    AG -. ai_tool .-> T1["get_leads<br/>Supabase leads_public"]
    AG -. ai_tool .-> T2["get_content<br/>Supabase content_public"]
    AG -. ai_tool .-> T3["get_invoices<br/>Sheets ledger"]
    AG -. ai_tool .-> T4["calculator"]
    AG --> R["grounded answer"]

    classDef ai fill:#1b2750,stroke:#6ea8fe,color:#e8edf7;
    classDef tool fill:#11302a,stroke:#34d399,color:#e8edf7;
    class AG,LM ai;
    class T1,T2,T3,T4 tool;
```

The agent's data tools read **sanitized, read-only views** (anon key, RLS-protected) — no PII, no
write paths, safe to expose on a public chat.

## Stack (100% free, no card)

| Concern   | Tool                                       |
|-----------|--------------------------------------------|
| Chat UI   | n8n Chat Trigger (hosted webchat)          |
| Agent     | n8n AI Agent + Groq `llama-3.3-70b-versatile` |
| Memory    | Window Buffer Memory (per session)         |
| Data tools| Supabase sanitized views · Google Sheets   |

## Engineering decisions & what I learned

<!-- filled in as we build: tool-calling agent vs plain RAG, read-only sanitized tools for a safe
     public chat, tool descriptions as prompt surface, memory for multi-turn, grounding/no-guess
     system prompt, free-tier guardrails. -->

## Status

🚧 In progress — see [`docs/BUILD_GUIDE.md`](docs/BUILD_GUIDE.md) for the build order.

---

> The capstone of a 4-project automation portfolio:
> [Lead Qualification &amp; CRM](https://github.com/karl22puday-eng/ai-lead-qualification-system) ·
> [Content Engine](https://github.com/karl22puday-eng/ai-content-engine) ·
> [Invoice Processor](https://github.com/karl22puday-eng/ai-invoice-processor) ·
> **Ops Assistant** (this repo).
