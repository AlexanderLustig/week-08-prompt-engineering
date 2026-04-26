# Capstone Project Context

## Project
- **Name:** AI-Powered Security Alert Triage System
- **Team:** Alexander Lustig — AI Core component
- **What it does:** Ingests raw security alerts via webhook, classifies severity and attack type using LLM chains, correlates with known threat patterns, and routes high-severity incidents to a Slack channel with recommended response actions. Analysts receive structured, actionable reports instead of raw log noise.
- **Project type:** Cybersecurity Threat Intelligence & Incident Response Automation

## Architecture
- **Ingestion:** n8n webhook receives raw alert text (syslog, SIEM export, or manual input), parses and normalizes it, writes a record to Airtable `security_alerts` table with `status: pending_classification`
- **AI Core:** Three Flowise LLM chains (Alert Classifier → Threat Analyzer → Response Recommender) process each alert sequentially; results written back to Airtable. Flowise chains use OpenAI gpt-4o-mini with structured JSON output. n8n polls Airtable for `status: pending_classification` every 5 minutes.
- **Specialist:** n8n workflow reads alerts with `status: classified`; correlates MITRE technique codes against known campaigns; enriches records with analyst assignments and priority scores; updates `status: enriched`
- **Integration:** n8n dashboard workflow reads all `status: enriched` records, posts Slack digest for HIGH/CRITICAL alerts, updates `status: reported`. Airtable views provide per-status monitoring.

## Tech Stack
- n8n Cloud (workflow automation — hosted on Railway)
- Flowise Cloud (LLM chains — Alert Classifier, Threat Analyzer, Response Recommender)
- OpenAI gpt-4o-mini (LLM inference via Flowise)
- Gemini 2.5 Flash (backup AI via n8n HTTP Request nodes)
- Hugging Face Inference API (NER for entity extraction from alert text)
- Airtable (shared database — 2 tables)
- GitHub (repo, documentation)

## Airtable Schema

### security_alerts
| Field | Type | Written By | Status Values |
|-------|------|------------|---------------|
| alert_id | Auto Number | Airtable | — |
| alert_text | Long Text | Ingestion | — |
| source_ip | Single Line Text | Ingestion | — |
| severity | Single Select | AI Core | CRITICAL, HIGH, MEDIUM, LOW, INFO |
| confidence | Number (0–1) | AI Core | — |
| reasoning | Long Text | AI Core | — |
| attack_type | Single Line Text | AI Core | brute force, C2, phishing, insider threat, misconfiguration |
| indicators | Long Text | AI Core | JSON array as string |
| potential_impact | Long Text | AI Core | — |
| mitre_techniques | Long Text | AI Core | JSON array: ["T####", ...] |
| confidence_assessment | Single Select | AI Core | HIGH, MEDIUM, LOW |
| immediate_actions | Long Text | AI Core (Recommender) | JSON array |
| investigation_steps | Long Text | AI Core (Recommender) | JSON array |
| containment_strategy | Long Text | AI Core (Recommender) | — |
| escalation_needed | Checkbox | AI Core (Recommender) | — |
| analyst_assigned | Single Line Text | Specialist | — |
| priority_score | Number | Specialist | 1–10 |
| slack_notified | Checkbox | Integration | — |
| status | Single Select | All | pending_classification, classified, enriched, reported |
| created_at | Date | Ingestion | — |
| updated_at | Date | Last modifier | — |

### incidents
| Field | Type | Written By | Status Values |
|-------|------|------------|---------------|
| incident_id | Auto Number | Airtable | — |
| alert_ids | Link to security_alerts | Specialist | — |
| incident_type | Single Select | Specialist | active_attack, data_breach, policy_violation, false_positive |
| severity | Single Select | Specialist | P1, P2, P3, P4 |
| assigned_team | Single Line Text | Specialist | — |
| resolution_notes | Long Text | Integration | — |
| status | Single Select | All | open, investigating, contained, resolved |
| created_at | Date | Specialist | — |
| updated_at | Date | Last modifier | — |

## Conventions
- Field names: snake_case
- Status values: lowercase (except severity levels which are UPPERCASE)
- Date fields end in `_at`
- Boolean fields use `is_` prefix OR are Airtable Checkbox type
- JSON arrays stored as Long Text fields (stringify before write, parse after read)

## Current State
- **What's working:** Flowise chains (Alert Classifier, Threat Analyzer, Response Recommender) all produce valid structured JSON. n8n LLM Chain Pipeline workflow calls all 3 chains end-to-end. Airtable base exists with security_alerts table.
- **What's in progress:** Ingestion webhook → Airtable write. Specialist correlation logic. Integration Slack notifications.
- **Known issues:** Gemini wraps JSON responses in markdown code fences — stripping logic added to n8n Code node. Flowise Cloud free tier limited to 2 chatflows.
- **Next milestone:** Checkpoint 2 (Week 9) — one complete alert record flowing through all 4 components (Ingestion → AI Core → Specialist → Integration) end-to-end without manual intervention.

## Repository Structure
```
week-08/
├── .github/
│   └── copilot-instructions.md   ← this file
├── docs/
│   └── checkpoint2-audit.md      ← AI-assisted gap analysis
├── screenshots/
│   ├── README.md                  ← screenshot instructions
│   ├── chain1-classifier.png
│   ├── chain2-analyzer.png
│   ├── chain3-recommender.png
│   ├── n8n-chain-pipeline.png
│   ├── copilot-working.png
│   ├── copilot-instructions.png
│   ├── capstone-audit-report.png
│   └── copilot-artifact.png
├── ai-core-README.md             ← Component README (Part 2.4 artifact)
├── prompt-log-alexander.md       ← Prompt log
└── README.md
```
