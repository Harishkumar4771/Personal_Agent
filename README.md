# Steward — Everyone's Personal Chief of Staff

> **Homingo Hackdays — Problem Statement A3: *A personal staff for everyone***  
> *Wealthy people have assistants, accountants, lawyers, and someone to handle errands. Everyone else handles it all alone, badly and late. Steward gives everyone a trusted, proactive Chief of Staff that handles mail, calendars, bills, refunds, appointments, and forms—without ever being reckless with your money or private data.*

---

## Why Steward?

Most AI "assistants" fail on real life admin for two reasons:
1. **They are reactive toys, not Chiefs of Staff** — You still have to prompt them step-by-step instead of waking up to problems already triaged and staged.
2. **They cannot be trusted with money or PII** — Giving an LLM raw credit card numbers, SSNs, or unrestricted write access is dangerous.

**Steward** solves both with a **Multi-Agent Chief of Staff** backed by a **Deterministic Policy & Vault Layer**:
- **Proactive Triage**: Watches incoming mail, bills, and calendar conflicts, then prepares a concise **Action Brief**.
- **Code-Enforced Safety (Not Prompt Hopes)**: Money movement, form submission, and external messages pass through deterministic permission tiers (`T0`–`T3`), hard spending caps, anomaly detection, and Human-in-the-Loop (HITL) approval checkpoints.
- **Zero-Exposure PII Vault**: Sensitive user data (cards, SSN, passport, bank routing) lives in an encrypted local vault. The LLM operates on opaque tokens (`{{CARD_PERSONAL}}`, `{{SSN_TOKEN}}`); real values are injected only at the verified execution boundary after user approval.

---

## Core Capabilities

| Domain | Specialist Agent | What It Does |
| :--- | :--- | :--- |
| **Inbox & Comms** | `MailAgent` | Triages inbox, detects hidden deadlines/bills/refunds, drafts replies, blocks phishing & prompt-injection emails |
| **Scheduling** | `CalendarAgent` | Resolves conflicts, protects focus/travel buffers, negotiates and books time slots |
| **Bills & Money** | `FinanceAgent` | Extracts invoice amounts/due dates from PDFs/emails, flags abnormal charges (e.g., 2x spike), stages sandboxed bill payments under hard caps |
| **Refunds & Disputes** | `AdvocateAgent` | Identifies refund eligibility (flight delays, duplicate charges, unwanted renewals), generates evidence-backed claims, tracks follow-ups |
| **Appointments & Errands** | `ErrandAgent` | Books medical/service appointments within user preferences (hours, location, price ceiling) |
| **Forms & Paperwork** | `FormAgent` | Maps PDF and web forms (via Playwright) using tokenized profile fields and captures a pre-submit screenshot for 1-click sign-off |

---

## Trust, Money & Privacy Architecture

Steward separates **reasoning** (LLMs) from **authorization** (deterministic code + user approval):

```text
  [Email / PDFs / Calendar / Web]
                 │
                 ▼
       ┌───────────────────┐
       │  ChiefOfStaff     │  ◄── Plans, prioritizes, builds Morning Brief
       │  (Orchestrator)   │
       └─────────┬─────────┘
                 │  Dispatches typed tasks
     ┌───────────┼───────────┬───────────┐
     ▼           ▼           ▼           ▼
 MailAgent  CalendarAgent FinanceAgent Advocate/Form
     └───────────┴───────────┴───────────┘
                 │  Proposes ActionIntent
                 ▼
  ┌─────────────────────────────────────────┐
  │     Deterministic Policy Engine         │
  │  • Tier 0-3 Permission Check            │
  │  • Spending Caps & Anomaly Thresholds   │
  │  • Prompt-Injection / Phishing Guard    │
  │  • Idempotency Key (no double-pay)      │
  └──────────────┬──────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
   [T0/T1 Safe]     [T2/T3 High-Risk]
   Auto-Execute     Pause & Interrupt (HITL)
        │                 │
        │                 ▼
        │        [Approval Queue UI] ──► User Approves / Rejects
        │                 │
        ▼                 ▼
  ┌─────────────────────────────────────────┐
  │  Encrypted PII Vault + Execution Layer  │
  │  • Hydrates {{TOKENS}} at last millisecond
  │  • Executes Gmail / Calendar / Stripe / Playwright
  │  • Writes immutable Audit Log entry     │
  └─────────────────────────────────────────┘
```

### Action Permission Tiers

- **Tier 0 — Read & Analyze (Autonomous):** Read inbox, parse bill PDFs, check calendar availability, detect refund eligibility.
- **Tier 1 — Stage & Draft (Autonomous):** Draft emails, pre-fill web forms (no submit), compute optimal calendar slots.
- **Tier 2 — Low-Risk Mutate (Policy-Governed):** Send routine scheduling confirmations, create tentative calendar holds within configured rules.
- **Tier 3 — Money, Legal & Sensitive PII (Mandatory HITL):** Pay any bill, submit legal/tax/insurance forms, send dispute claims, share PII externally, or exceed any cap/anomaly threshold.

---

## Planned Repository Structure

```text
Personal_AI_Agent/
├── AGENTS.md                 # Multi-agent roster, safety contracts & coding rules
├── HACKATHON_STRATEGY.md     # Winning strategy, 3-min demo script & pitch playbook
├── ROADMAP.md                # 10-day sprint plan (Nov 27 – Dec 6, 2026)
├── README.md                 # Project overview & setup
├── requirements.txt          # Python dependencies
├── app/
│   ├── main.py               # FastAPI + WebSocket entrypoint
│   ├── config.py             # Environment & policy configuration
│   ├── agents/               # ChiefOfStaff + specialist agents
│   ├── core/                 # Policy engine, PII vault, audit log, scheduler
│   ├── adapters/             # Gmail, Calendar, Stripe (test), Playwright, Mock sandbox
│   └── schemas/              # Pydantic models for ActionIntent, Approvals, Briefs
├── frontend/                 # Chief of Staff Command Center UI
└── tests/                    # Unit, integration & adversarial safety tests
```

---

## Quickstart (Local Dev)

> **Note:** Project scaffolding is planned across Days 1–2 of [ROADMAP.md](./ROADMAP.md). Once initialized:

```bash
# 1. Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Install Playwright browser binaries (for FormAgent)
playwright install chromium

# 4. Configure environment variables
cp .env.example .env
# Add your LLM API key (e.g., GEMINI_API_KEY) and set DEMO_MODE=true for sandbox adapters

# 5. Run the API server
uvicorn app.main:app --reload --port 8000
```

---

## Related Docs

- [AGENTS.md](./AGENTS.md) — Agent definitions, trust contracts, and developer guidelines
# Personal_Agent
