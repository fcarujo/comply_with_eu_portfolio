# 🛡️ ComplyWith.eu

**EU cyber compliance self-assessment, powered by agentic AI and real enforcement data.**

Live at → **[complywith.eu](https://complywith.eu)**

---

## What it does

ComplyWith.eu helps EU organisations understand their cybersecurity regulatory exposure across GDPR, NIS2, DORA, and the Cyber Resilience Act, grounded in real enforcement data.

The tool has two sections:

| Section | What it does |
|---|---|
| **Compliance Assessment** | 10-question organisation profile determines which regulations apply. User selects which regulation to assess first. Regulation-specific questionnaire (GDPR: 86 items across 8 domains · NIS2: 77 items across 10 domains · DORA: 41 items across 5 domains · CRA: 23 items across 4 domains). Gap report shows only what's missing. AI-powered investigation per domain. Assessments saved and restored across sessions. |
| **Ask the regulation** | Free-form chat against 1,701 real GDPR enforcement decisions and the full text of EU regulations and BSI threat reports. Ask anything. |

---

## The problem it solves

SMEs face growing regulatory exposure under GDPR, NIS2, and DORA — but have no affordable way to understand it.

- A compliance consultant charges €150-300/hour. A basic gap assessment costs €6,000-20,000.
- Enterprise compliance platforms (OneTrust, Venvera, Copla) start at €300-3,000/month.
- Free tools are static Excel checklists with no intelligence.

ComplyWith.eu sits in the gap: AI-powered, enforcement-grounded, currently free.

---

## What makes it different

**Real enforcement data.** Every AI investigation is grounded in 1,701 actual GDPR enforcement decisions — real companies, real fines, real violations. Not generic advice.

**Agentic investigation.** A ReActAgent autonomously queries two data sources — the enforcement database and regulation/threat documents — and synthesises sector-specific findings. It reasons step by step, not just retrieves.

**Regulation-specific domains.** Each regulation has its own domain structure aligned with how audits are actually done:
- GDPR: 8 domains matching DPA audit practice
- NIS2: 10 domains matching Art. 21(2) exactly — one domain per legal requirement
- DORA: 5 pillars matching the regulation structure
- CRA: 4 domains based on Annex I essential requirements — *coverage will deepen once ETSI/CEN-CENELEC harmonised technical standards are published (expected 2026)*
- *EU AI Act: to be enforced from December 2027, it is planned for August 2026. It will cover AI system governance and risk classification, which is distinct from CRA (product cybersecurity). Both may apply to the same organisation.*

**Pre-filled answers across regulations.** When assessing NIS2 after GDPR, controls that overlap are pre-filled from your GDPR answers — highlighted for review. 45 overlap mappings across 227 total items.

**Plain language.** 227 compliance requirements written as practical yes/no controls that non-lawyers can answer honestly.

**Saved assessments.** Completed assessments are saved per user and restored on next login — no need to redo work.

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
│  1,701 GDPR  │  │   227 items      │
│  decisions   │  │   Voyage AI      │
│  + user data │  │   voyage-law-2   │
└──────────────┘  └──────────────────┘

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
| **Database** | PostgreSQL 15 on Cloud SQL — GDPR fines, users, usage, assessments |
| **Storage** | Cloud Storage (vectorstore), Secret Manager (API keys) |
| **Deployment** | Cloud Run — europe-west1 |
| **Auth** | Supabase Auth + Cloudflare Turnstile |
| **PDF** | ReportLab Platypus |
| **Domain** | complywith.eu — Infomaniak, mapped to Cloud Run with auto-SSL |

---

## Compliance framework

### GDPR — 8 domains, 86 items
Aligned with DPA audit practice: Lawful basis & purpose limitation · Transparency & privacy notices · Data subject rights · Security of processing · Breach detection & notification · Processors & third-party transfers · Governance & accountability · Special categories & high-risk processing

### NIS2 — 10 domains, 77 items
Matching Art. 21(2) exactly: Risk management policies & governance · Incident handling · Business continuity & crisis management · Supply chain security · Network & system security · Vulnerability handling & disclosure · Cybersecurity training & awareness · Cryptography & encryption · Access control & authentication · Threat monitoring & detection

### DORA — 5 domains, 41 items
Matching the 5 regulatory pillars: ICT risk management framework · ICT incident classification & reporting · Digital operational resilience testing · ICT third-party risk management · Information & intelligence sharing

### CRA — 4 domains, 23 items
Based on Annex I essential requirements: Product security by design · Vulnerability management · Conformity & documentation · Incident & vulnerability reporting

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

**Key technical challenge solved** — Mac Python 3.12's async SSL certificate issue: Voyage AI's async embedding client uses `aiohttp` which requires system SSL certificates. Fixed by running Apple's certificate installer.

---

## Status

| Component | Status |
|---|---|
| Compliance Assessment — regulation-specific questionnaire | ✅ Live |
| GDPR assessment (8 domains, 86 items) | ✅ Live |
| NIS2 assessment (10 domains, 77 items) | ✅ Live |
| DORA assessment (5 domains, 41 items) | ✅ Live |
| CRA assessment (4 domains, 23 items) | ✅ Live |
| Cross-regulation pre-fill (45 overlap mappings) | ✅ Live |
| AI gap investigations (ReActAgent) | ✅ Live |
| PDF gap report | ✅ Live |
| Save & restore assessments across sessions | ✅ Live |
| Ask the regulation (chat) | ✅ Live |
| User accounts (Supabase) | ✅ Live |
| Freemium paywall | ✅ Live |
| Free premium upgrade (beta) | ✅ Live |
| Custom domain + SSL | ✅ Live |
| Stripe payment | 🔲 Planned |
| EU AI Act coverage | 🔲 Planned (Aug 2026) |

---

## About

Built as a portfolio project to master LlamaIndex and agentic AI — while solving a real problem for EU organisations navigating an increasingly complex regulatory landscape.

The code is private (commercial product). Questions and feedback welcome via [LinkedIn](https://www.linkedin.com/in/franciscotrindade/).

---

*Not legal advice. ComplyWith.eu is a self-assessment tool for contextual awareness only.*
