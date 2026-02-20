# AI Sales Lead Generation + Qualification Agent Blueprint

## 1) Business Goal
Build an AI-driven outbound sales agent that:
- Finds and enriches B2B prospects in **India + Middle East**.
- Focuses on software categories: **AI Agents, ERP, HRMS, CRM, Helpdesk, LMS**.
- Prioritizes and qualifies opportunities with stronger focus on **ERP implementation demand**.
- Calls the qualified person (or provided contact) and captures outcomes.
- Pushes activity, qualification score, and next steps into **Frappe CRM/ERPNext**.

---

## 2) High-Level Architecture

```text
[Lead Sources]
  India/Middle East directories, website forms, uploaded CSV, campaign lists
          |
          v
[Ingestion + Enrichment Pipeline]
  - company normalization
  - persona detection (IT head, CFO, HR head, Ops)
  - intent and firmographic enrichment
          |
          v
[Lead Scoring + Qualification Agent]
  - ICP fit score
  - need urgency score
  - ERP implementation readiness score
          |
   (qualified leads only)
          v
[Conversation Agent + Calling Bot]
  - multilingual scripts (English/Hindi/Arabic optional)
  - discovery questions, objection handling
  - meeting booking or callback scheduling
          |
          v
[Frappe CRM Sync Layer]
  - create/update Lead
  - create Call Log/Note/Task/Opportunity
  - update status + next action + owner
          |
          v
[Analytics + Human Review]
  - conversion funnel
  - model quality drift
  - QA review of call summaries
```

---

## 3) Core Components (Practical Build)

### A) Data Ingestion + Enrichment
- Input channels:
  - CSV imports from your sales team.
  - Website/contact forms.
  - Outbound data providers (legal/compliant sources).
- Enrichment fields:
  - company size, industry, employee count, ERP usage status, region/country.
  - role/title seniority and decision-making authority.
  - contact confidence score.

**Suggested open-source stack**
- Workflow: **n8n** or **Apache Airflow**.
- Processing API: **FastAPI**.
- Storage: **PostgreSQL** + **Redis** cache.

### B) Lead Qualification Engine
Create a rules + AI hybrid score:

`Final Score = 35% ICP Fit + 35% ERP Need + 20% Contact Quality + 10% Engagement Intent`

Qualification examples:
- ICP Fit: region in India/Middle East, company size threshold, target industry.
- ERP Need: signs of multi-branch operations, legacy systems, manual workflows.
- Contact Quality: verified phone, business email, decision-maker role.
- Engagement Intent: replied positively, requested demo, downloaded brochure.

Set stage thresholds:
- **Hot (>=80):** immediate call + meeting offer.
- **Warm (60–79):** nurture + callback task.
- **Cold (<60):** sequence-based outreach only.

### C) Calling Agent
- Integrate telephony provider (Exotel, Twilio, Asterisk/FreeSWITCH).
- Use speech pipeline:
  - STT: **Whisper** (open source).
  - NLU + response generation: open model (see model section).
  - TTS: **Coqui TTS / Piper**.
- Mandatory call outputs:
  - call summary
  - pain points
  - budget/timeline signals
  - competitor/current system info
  - next action date/time

### D) Frappe CRM Integration
Use Frappe REST API or server-side app hooks:
- Upsert `Lead` by email/phone.
- Create `Communication` / call note after each interaction.
- Update custom fields:
  - `lead_score`
  - `qualification_status`
  - `erp_readiness`
  - `next_followup_at`
  - `last_call_disposition`
- Auto-create `Task` for sales reps when handoff needed.

Recommended dedupe key: `normalized_phone + company_domain`.

---

## 4) Suggested LLM Strategy

### Primary (Open-source first)
1. **Llama 3.1 70B Instruct**
   - Best quality among open options for nuanced qualification and call summarization.
2. **Qwen2.5 32B Instruct**
   - Strong multilingual behavior; useful for India + Middle East contexts.
3. **Mixtral 8x7B Instruct**
   - Efficient inference with good general performance.

### Cost-optimized routing approach
- Use smaller local model for:
  - lead cleaning, classification, draft notes.
- Use stronger model only for:
  - complex qualification reasoning,
  - objection handling,
  - final call summary + CRM note.

### If token-based managed model is needed
- For high-accuracy production fallback, use a premium API model for only 10–20% of hard cases (router pattern).
- Keep PII-safe policy + redaction before external calls.

---

## 5) Workflow in Production
1. New lead enters pipeline.
2. Enrichment service collects firmographic + role signals.
3. Qualification engine assigns score and stage.
4. Hot/Warm leads assigned to voice agent queue.
5. Voice call happens; transcript + summary generated.
6. Frappe CRM is updated automatically.
7. If meeting interest is positive, create Opportunity + Task for human AE.
8. Daily dashboard tracks lead-to-meeting conversion by region and product line.

---

## 6) MVP Delivery Plan (6–8 Weeks)

### Phase 1 (Week 1–2): Foundation
- Frappe custom fields and API connector.
- Lead ingestion endpoint + deduplication.
- Basic scoring (rule-based).

### Phase 2 (Week 3–4): AI Qualification
- LLM prompt templates for ICP + ERP readiness extraction.
- Confidence scoring and fallback rules.
- CRM writeback with audit logs.

### Phase 3 (Week 5–6): Voice Agent
- Telephony integration.
- STT/TTS + scripted conversation flow.
- Call outcome taxonomy.

### Phase 4 (Week 7–8): Optimization
- Human QA loop.
- Prompt tuning by region/language.
- Conversion analytics and SLA automation.

---

## 7) Governance and Compliance Checklist
- Consent and calling window controls by country.
- DND suppression list before auto-dial.
- PII encryption at rest and in transit.
- Role-based access in Frappe.
- Prompt + response logging with masked sensitive data.

---

## 8) Recommended Tech Stack (Concrete)
- Backend: FastAPI (Python)
- Orchestration: n8n
- Queue: Celery + Redis
- DB: PostgreSQL
- Vector DB (optional): Qdrant
- Models: vLLM serving Llama/Qwen/Mixtral
- Speech: Whisper + Coqui/Piper
- Telephony: Exotel/Twilio/Asterisk
- CRM: Frappe REST + webhooks
- Observability: Prometheus + Grafana + Sentry

---

## 9) Immediate Next Steps
1. Finalize ICP scoring rules for each product (ERP/HRMS/CRM/Helpdesk/LMS).
2. Confirm telephony provider for India + Middle East routes.
3. Define Frappe field mapping and pipeline stages.
4. Build MVP with 500-lead pilot dataset.
5. Measure KPI targets:
   - lead qualification precision
   - call connect rate
   - meeting-booking rate
   - opportunity conversion rate

This blueprint gives you an implementation-ready base to start building a production sales agent with open models and optional premium model fallback.
