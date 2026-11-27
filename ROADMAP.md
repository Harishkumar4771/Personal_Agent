# Project Roadmap — Steward (Homingo Hackdays A3)

**Timeline:** **November 27 – December 6, 2026 (10 Days)**  
**Goal:** Ship a live, rock-solid **Personal Chief of Staff** with proactive triage, multi-agent execution, encrypted PII vault, deterministic money/security guardrails, and an executive Command Center UI.

---

## Sprint Overview

| Phase | Dates | Focus | Milestone |
| :--- | :--- | :--- | :--- |
| **Phase 1: Foundation & Trust Core** | Nov 27 – Nov 28 | Repo setup, schemas, Encrypted Vault, Policy Engine, LangGraph Chief of Staff | **M1:** Core orchestrator pauses/resumes cleanly on Tier-3 actions |
| **Phase 2: Specialist Agents** | Nov 29 – Dec 1 | Mail, Calendar, Finance (bill anomaly + Stripe test), Advocate (refunds), Form (Playwright) | **M2:** All 4 hero workflows execute end-to-end in Sandbox + Live adapters |
| **Phase 3: Proactive Loop & UI** | Dec 2 – Dec 3 | Background Morning Brief scheduler, FastAPI/WS backend, Command Center frontend | **M3:** Interactive dashboard with 1-click Approval Queue & Audit Log |
| **Phase 4: Hardening & Demo Polish** | Dec 4 – Dec 6 | Adversarial safety tests, backup video, pitch deck, live rehearsal, submission | **M4:** Feature-frozen, rehearsed 3-min winning demo |

---

## Day-by-Day Execution Plan

### Day 1 — Thu, Nov 27: Architecture Lock, Scaffolding & Trust Primitives
- [ ] Initialize Python 3.11+ virtual environment and install `requirements.txt`.
- [ ] Create directory structure (`app/agents`, `app/core`, `app/adapters`, `app/schemas`, `tests`, `frontend`).
- [ ] Define Pydantic v2 schemas (`ActionIntent`, `ActionTier`, `TriageItem`, `MorningBrief`, `AuditEntry`).
- [ ] Build **Encrypted PII Vault** (`app/core/vault.py`) using `cryptography` (Fernet) + token hydration (`{{CARD_PERSONAL}}`, `{{SSN_LAST4}}`, `{{ADDRESS}}`).
- [ ] Build **Deterministic Policy Engine** (`app/core/policy.py`): Tier `T0`–`T3` rules, spending caps, anomaly multiplier check (`> 1.5x` historical avg), idempotency keys, and prompt-injection blocklist.
- **Definition of Done (DoD):** Unit tests pass proving raw PII never enters logs/prompts and oversized/duplicate payments are blocked or escalated to `T3`.

### Day 2 — Fri, Nov 28: `ChiefOfStaff` Orchestrator & Checkpointing
- [ ] Implement SQLite persistence (`aiosqlite` + `langgraph-checkpoint-sqlite`) for durable agent state and immutable Audit Log.
- [ ] Build `ChiefOfStaff` LangGraph state machine (`app/agents/chief_of_staff.py`) with `interrupt()` support for HITL approvals.
- [ ] Build the `SandboxAdapter` (`app/adapters/sandbox.py`) containing realistic mock inbox messages, calendar events, merchant history, and sample PDF bills.
- **DoD:** Triggering a triage run creates `ActionIntent` items, pauses at `T3` HITL, and resumes cleanly upon API approval.

### Day 3 — Sat, Nov 29: `MailAgent` & `CalendarAgent`
- [ ] Implement `MailAgent`: classifies incoming emails (`bill`, `refund`, `appointment`, `form`, `phishing/injection`), extracts deadlines, drafts replies.
- [ ] Implement `CalendarAgent`: detects overlapping meetings, enforces 15-min travel/prep buffers, suggests optimal appointment slots.
- [ ] Add optional real **Gmail + Google Calendar** OAuth adapter (`app/adapters/google_workspace.py`) alongside the default `DEMO_MODE` sandbox.
- **DoD:** Inbox scan correctly identifies a dentist follow-up email, finds a conflict-free calendar slot, and rejects a prompt-injected phishing email.

### Day 4 — Sun, Nov 30: `FinanceAgent` (Bills, Overcharge Detection & Safe Pay)
- [ ] Implement PDF & email bill parser using `pdfplumber` / `pypdf` + structured LLM extraction.
- [ ] Add merchant baseline comparison (e.g., Electric Bill usually `~$95`, current invoice `$420` → auto-flags anomaly & blocks auto-pay).
- [ ] Integrate **Stripe Test Mode** (or Mock Payment Gateway) behind `PolicyEngine` + idempotency guard.
- **DoD:** Normal recurring bill can be staged for 1-click pay; anomalous `$420` bill triggers prominent warning badge and refuses execution without explicit override.

### Day 5 — Mon, Dec 1: `AdvocateAgent` & `FormAgent`
- [ ] Implement `AdvocateAgent`: detects delayed flight / duplicate subscription / overcharge from inbox, drafts evidence-backed refund claim, and schedules follow-up reminders.
- [ ] Implement `FormAgent` using **Playwright**: navigates to a sample refund/insurance web portal (host a clean local demo portal HTML page so it is 100% reliable!), maps fields to Vault tokens, fills form, and captures a **pre-submit screenshot**.
- [ ] Pause at `T3` HITL so user inspects screenshot before clicking "Approve & Submit".
- **DoD:** Flight delay email automatically results in a pre-filled airline compensation form with a verifiable screenshot preview.

### Day 6 — Tue, Dec 2: Proactive Scheduler, FastAPI & WebSockets
- [ ] Wire `APScheduler` for background inbox/due-date scans and refund follow-up timers.
- [ ] Build FastAPI endpoints (`GET /brief`, `POST /triage/run`, `GET /approvals`, `POST /approvals/{id}/decide`, `GET /audit`, `POST /vault/profile`).
- [ ] Stream live agent reasoning & policy checks over WebSockets (`/ws/events`) so judges see real-time multi-agent activity.
- **DoD:** Full backend API documented at `/docs` and live WebSocket updates verified.

### Day 7 — Wed, Dec 3: Command Center Frontend UI
- [ ] Build a clean, executive dark/light **Chief of Staff Command Center** UI (`frontend/` — React/Next.js or fast single-page Tailwind + HTMX/React Vite app):
  1. **Morning Brief Panel:** "4 items triaged · $280 potential refund found · 1 anomaly blocked · Est. time saved: 52 mins".
  2. **Approval Queue:** Risk badges (`LOW / MEDIUM / HIGH`), exact dollar amount, anomaly warnings, Playwright form screenshot preview, `[Approve]` / `[Reject]` / `[Edit]` buttons.
  3. **Live Agent Activity Feed:** Shows `ChiefOfStaff` → Specialist Agents → `PolicyEngine` checks in real time.
  4. **PII Vault & Policy Settings:** Toggle spending caps (`Max Auto-Pay`, `Daily Limit`), view masked tokens (`•••• 4242`), inspect **Immutable Audit Log**.
- **DoD:** Entire user journey works smoothly through the browser without touching a terminal.

### Day 8 — Thu, Dec 4: End-to-End Integration & Adversarial Testing
- [ ] Run the **4 Hero Demo Scenarios** back-to-back from a clean database reset:
  1. Dentist appointment conflict + booking
  2. Utility bill spike (`$420` vs `$95`) flagged + safe bill payment
  3. Prompt-injection phishing email blocked by `PolicyEngine`
  4. Delayed flight refund claim + Playwright form fill with screenshot sign-off
- [ ] Write & run adversarial `pytest` suite (prompt injection, double-submit idempotency, PII redaction verification).
- **DoD:** Zero crashes across 5 consecutive full demo runs; all `pytest` suites green.

### Day 9 — Fri, Dec 5: Polish, Backup Video & Feature Freeze
- [ ] **Feature Freeze by 6:00 PM.** No new features after this point!
- [ ] Add a `"Reset & Seed Demo Scenario"` button in the UI header for instant judge replays.
- [ ] Tune LLM latency (parallelize independent specialist agents, cache static extractions).
- [ ] Record a clean 3-minute **backup demo video** in case venue Wi-Fi or API keys hiccup.
- **DoD:** Stable demo build tagged `v1.0-hackathon`, backup video saved locally.

### Day 10 — Sat, Dec 6: Pitch Deck, Rehearsal & Submission
- [ ] Finalize 6-slide pitch deck (Problem → Solution → Trust Architecture → Live Demo → Market/Impact).
- [ ] Rehearse the 3-minute live demo + 2-minute Q&A at least 5 times with a timer.
- [ ] Verify README screenshots, submission description, and live deployment/local runner.
- [ ] Submit before deadline!

---

## Scope-Cut Rules (If Behind Schedule on Dec 3)

Never sacrifice **reliability** or the **Trust/Money Guardrails** for extra integrations:
1. **Cut First:** Live OAuth bank/healthcare portals (keep Stripe Test + Gmail/Calendar + local mock portal for Playwright).
2. **Cut Second:** Complex multi-user auth (use single-user local workspace profile for demo).
3. **NEVER Cut:**
   - The **Morning Brief + 1-Click Approval Queue**
   - The **Billing Anomaly / Spending Cap + Prompt Injection Block**
   - The **Playwright Pre-Submit Screenshot + PII Vault Masking**
   - The **Immutable Audit Log**
