# Autonomous BANT Lead Qualification & CRM Ingestion Engine

An event-driven n8n pipeline that automates web scraping, data extraction, deterministic BANT qualification (OpenAI), and conditional routing into HubSpot CRM alongside real-time Telegram sales alerts.

---

## Architectural Overview

The engine acts as a cost-optimized, automated SDR (Sales Development Representative). It discovers target domains, scrapes landing page contents, sanitizes raw text for compliance, extracts structured corporate identifiers, performs multi-vector BANT analysis via LLM, and synchronizes qualified leads to enterprise CRM.

```text
[Cron / Webhook]
       │
       ▼
 [Serper API] ──(Web Discovery)
       │
       ▼
 [ScraperAPI] ──(HTML Ingestion & Sanitization)
       │
       ▼
 [Regex / Code] ──(Tax ID / NIP Extraction & PII Redaction)
       │
       ▼
[OpenAI Engine] ──(Deterministic BANT Scoring & Synthesis)
       │
       ▼
[Decision Gate] ──(lead_score >= 70?)
      ├── YES ──► [HubSpot CRM Integration] & [Telegram Alert Bot]
      └── NO  ──► [Drop / Archive Lead]

```

---

## Key Features & Design Principles

* **Cost-Optimized Validation Stack:** Built to run on zero-cost infrastructure limits during validation (Serper Free Tier, ScraperAPI base quota, and `gpt-4o-mini`). Includes built-in budget enforcement logic to prevent uncontrolled API credit consumption.
* **Deterministic BANT Evaluation:** Evaluates public web assets across four distinct criteria:
* **Budget:** Analyzes published service tiers, pricing calculators, and enterprise complexity.
* **Authority:** Verifies organizational hierarchy signals and executive contact access.
* **Need:** Identifies operational bottlenecks (e.g., manual accounting, payroll) receptive to automation.
* **Timing:** Detects expansion momentum, active hiring spikes, or regulatory shifts.


* **PII & Data Integrity Safeguards:** Prior to enrichment, public contact routing utilizes deterministic hashed placeholders (`lead-YYYY-MM-DD-[hash]@placeholder.com`), preserving lead uniqueness while isolating validated business tax identifiers (`NIP`) for official registry lookup.
* **Conditional CRM Ingestion:** Enforces an automated quality gate (`lead_score >= 70`). Unqualified leads are purged from execution flow, preserving CRM database cleanliness and preventing outbound sales fatigue.

---

## Proof of Work & Execution Results

### 1. High-Precision CRM Ingestion (HubSpot)

Qualified companies are immediately mapped and pushed into HubSpot with their respective scoring, synthetic identifiers, and domain references:

### 2. Deep Qualitative Qualification

Sales reps gain access to comprehensive qualifying context directly within the CRM custom record view, eliminating manual research overhead:

### 3. Immediate Outbound Alerting (Telegram)

High-priority leads trigger instant actionable briefs to mobile sales channels:

---

## Project Structure

```text
├── assets/
│   ├── hubspot_leads_table.png
│   ├── hubspot_record_bant_view.png
│   ├── n8n_workflow_canvas.png
│   └── telegram_alert.png
├── docs/
│   ├── hubspot_custom_properties.md
│   └── telegram_payload_example.json
├── workflow/
│   └── bant_lead_engine.json
├── .env.example
├── .gitignore
├── docker-compose.yml
└── README.md

```

---

## Setup & Deployment

### 1. Environment Configuration

Clone the repository and copy the environment template:

```bash
cp .env.example .env

```

Populate the missing values in `.env`:

* `OPENAI_API_KEY`
* `SERPER_API_KEY`
* `SCRAPERAPI_KEY`
* `HUBSPOT_ACCESS_TOKEN`
* `TELEGRAM_BOT_TOKEN`
* `TELEGRAM_CHAT_ID`

### 2. HubSpot Custom Properties

Before importing the workflow, ensure custom properties are defined in your HubSpot instance as documented in `docs/hubspot_custom_properties.md`.

### 3. Workflow Import

1. Start the n8n container via Docker:

```bash
docker compose up -d

```

2. Navigate to your n8n interface (`http://localhost:5678`).
3. Select **Import from File** and upload `workflow/bant_lead_engine.json`.
4. Attach your credential configurations to the respective nodes and activate the workflow.

---

## Extensibility & Production Scaling

* **Data Ingestion Agnostic:** The initial discovery step can be swapped from Google/Serper to Apollo.io, LinkedIn Sales Navigator exports, or official national company registers (KRS/CEIDG via REST).
* **Configurable Scoring Thresholds:** Decision gates and prompt weights can be re-calibrated per industry vertical to match more stringent enterprise ICP constraints.
* **Fail-Safe Operation:** Where pages omit structured pricing or team structures, the LLM prompt is engineered to record missing attributes rather than hallucinating validation points.