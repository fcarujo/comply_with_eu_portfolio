# 🛡️ ComplyWith.eu

**EU cyber compliance self-assessment, powered by agentic AI and real enforcement data.**

Live at → **[complywith.eu](https://complywith.eu)**

---

## What it does

ComplyWith.eu helps EU organisations understand their cybersecurity regulatory exposure — across GDPR, NIS2, DORA, and the Cyber Resilience Act — in 30 minutes, grounded in real enforcement data.

The tool has three sections:

| Section | What it does |
|---|---|
| **Am I affected?** | 4-question relevance checker. Tells you exactly which EU regulations apply to your organisation and why. |
| **Explore** | Free-form chat against 1,701 real GDPR enforcement decisions and the full text of EU regulations and BSI threat reports. Ask anything. |
| **Assess myself** | 132-item compliance questionnaire across 10 control domains. Gap report shows only what's missing. AI-powered investigation per domain grounded in real enforcement data. |

---

## The problem it solves

SMEs face growing regulatory exposure under GDPR, NIS2, and DORA — but have no affordable way to understand it.

- A compliance consultant charges €150-300/hour. A basic gap assessment costs €6,000-20,000.
- Enterprise compliance platforms (OneTrust, Venvera, Copla) start at €300-3,000/month.
- Free tools are static Excel checklists with no intelligence.

ComplyWith.eu sits in the gap: AI-powered, enforcement-grounded, priced for SMEs.

---

## What makes it different

**Real enforcement data.** Every AI investigation is grounded in 1,701 actual GDPR enforcement decisions scraped from privacyaffairs.com — real companies, real fines, real violations. Not generic advice.

**Agentic investigation.** A ReActAgent autonomously queries two data sources — the enforcement database and regulation/threat documents — and synthesises sector-specific findings. It reasons step by step, not just retrieves.

**Multi-regulation coverage.** GDPR + NIS2 + DORA + CRA in one assessment. Checklist items are filtered by which regulations actually apply to the user's organisation.

**Plain language.** 132 compliance requirements rewritten from legal text into practical yes/no controls that non-lawyers can answer honestly.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  complywith.eu                       │
│              Streamlit · Cloud Run                   │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │   RouterQueryEngine     │
          │   LLMSingleSelector     │
          └──────┬──────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
┌──────────────┐  ┌──────────────────┐
│  SQL Engine  │  │   RAG Engine     │
│NLSQLTable    │  │VectorStoreIndex  │
│QueryEngine   │  │similarity_top_k=5│
└──────┬───────┘  └───────┬──────────┘
       │                  │
       ▼                  ▼
┌──────────────┐  ┌──────────────────┐
│  Cloud SQL   │  │   ChromaDB       │
│  PostgreSQL  │  │   Cloud Storage  │
│  1,701 GDPR  │  │   585 chunks     │
│  decisions   │  │   Voyage AI      │
└──────────────┘  │   voyage-law-2   │
                  └──────────────────┘

         Gap Investigation (on demand)
┌─────────────────────────────────────┐
│         AgentWorkflow               │
│         ReActAgent                  │
│  ┌─────────────┐ ┌───────────────┐  │
│  │  SQL Tool   │ │   RAG Tool    │  │
│  │ enforcement │ │ regulations + │  │
│  │    data     │ │ BSI reports   │  │
│  └─────────────┘ └───────────────┘  │
└─────────────────────────────────────┘
```

---

## Tech stack

| Layer | Technology |
|---|---|
| **LLM** | Claude Haiku 4.5 (Anthropic) |
| **Embeddings** | Voyage AI `voyage-law-2` — specialised for legal text |
| **Agent framework** | LlamaIndex 0.14 — AgentWorkflow + ReActAgent |
| **Vector store** | ChromaDB persistent — 585 chunks from 7 source documents |
| **SQL engine** | LlamaIndex NLSQLTableQueryEngine — natural language to SQL |
| **Router** | LlamaIndex RouterQueryEngine + LLMSingleSelector |
| **UI** | Streamlit |
| **Database** | PostgreSQL 15 on Cloud SQL (1,701 GDPR enforcement decisions) |
| **Storage** | Cloud Storage (vectorstore), Secret Manager (API keys) |
| **Deployment** | Cloud Run — europe-west1 |
| **Auth** | Supabase Auth + Cloudflare Turnstile |
| **PDF** | ReportLab Platypus |
| **Domain** | complywith.eu — Infomaniak, mapped to Cloud Run with auto-SSL |

---

## Source documents indexed (585 chunks)

| Document | Source |
|---|---|
| GDPR full text | EUR-Lex |
| NIS2 Directive | EUR-Lex |
| DORA Regulation | EUR-Lex |
| Cyber Resilience Act | EUR-Lex |
| BSI Threat Report 2022 | bsi.bund.de |
| BSI Threat Report 2023 | bsi.bund.de |
| BSI Threat Report 2024 | bsi.bund.de |

---

## LlamaIndex learning journey

This project was built as a deep-dive into LlamaIndex and agentic AI. Key concepts explored:

**SQL Query Engine** — `NLSQLTableQueryEngine` converts natural language questions into SQL queries against a PostgreSQL database of 1,701 GDPR enforcement decisions. The LLM generates the SQL, executes it, and synthesises the result into a human-readable answer.

**RAG Pipeline** — `VectorStoreIndex` over 585 chunks of EU regulation text and BSI threat reports, embedded with Voyage AI `voyage-law-2` (a domain-specific legal embedding model). `similarity_top_k=5` retrieves the most relevant chunks per query.

**Router** — `RouterQueryEngine` with `LLMSingleSelector` automatically decides whether each question should go to the SQL engine (enforcement data) or the RAG engine (regulation text). No manual routing rules.

**ReAct Agent** — `AgentWorkflow` + `ReActAgent` implements the full ReAct loop (Reason → Act → Observe → repeat). The agent autonomously decides which tools to call, in what order, and when it has enough information to synthesise a finding. This is the step from retrieval-augmented generation to genuine agentic AI.

**Key technical challenge solved** — Mac Python 3.12's async SSL certificate issue: Voyage AI's async embedding client uses `aiohttp` which requires system SSL certificates. Fixed by running Apple's certificate installer. Took significant debugging to isolate.

---

## Compliance domains covered

1. Legal basis & data governance
2. Transparency & privacy notices
3. Data subject rights
4. Security measures & encryption
5. Incident detection & reporting
6. Access control & identity
7. Processor & third-party management
8. Business continuity & recovery
9. Risk management & governance
10. Vulnerability & product security

132 checklist items across 10 domains, all rewritten from legal text into plain language actionable controls. Items are filtered by applicable regulations — a food company never sees DORA-specific requirements.

---

## Maturity thresholds

Aligned with audit and enforcement practice:

| Score | Label | What it means |
|---|---|---|
| 0.0 – 1.0 | 🔴 Critical | No evidence of controls. Clear non-compliance. Immediate audit finding. |
| 1.1 – 2.5 | 🟡 Moderate | Controls exist but not documented or tested. Would fail a formal audit. |
| 2.6 – 3.4 | 🟢 Minor | Mostly compliant. Would pass with recommendations. Low exposure. |
| 3.5 – 4.0 | ✅ Good standing | Documented, tested, consistently applied. Audit-ready. |

---

## Status

| Component | Status |
|---|---|
| Am I affected? | ✅ Live |
| Explore | ✅ Live |
| Assess myself — questionnaire | ✅ Live |
| AI gap investigations | ✅ Live |
| PDF gap report | ✅ Live |
| User accounts (Supabase) | ✅ Live |
| Freemium paywall | ✅ Live |
| Free premium upgrade (beta) | ✅ Live |
| Custom domain + SSL | ✅ Live |
| Stripe payment | 🔲 Planned |
| EU AI Act coverage | 🔲 Planned (Aug 2026) |

---

## About

Built as a portfolio project to master LlamaIndex and agentic AI — while solving a real problem for EU organisations navigating an increasingly complex regulatory landscape.

The code is private. Questions and feedback welcome via [LinkedIn](https://www.linkedin.com/in/franciscotrindade/)]

---

*Not legal advice. ComplyWith.eu is a self-assessment tool for contextual awareness only.*
