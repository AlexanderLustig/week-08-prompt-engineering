# Week 8 — Prompt Engineering & AI-Assisted Development

**Course:** AI Integration
**Student:** Alexander Lustig
**Date:** 2026-04-26

---

## What This Repo Contains

### Part 1: Flowise LLM Chains + n8n Pipeline

Three LLM chains built in Flowise using OpenAI gpt-4o-mini, callable via HTTP API:

| Chain | Platform | Flowise/Webhook ID | Purpose |
|-------|----------|-------------------|---------|
| Alert Classifier | Flowise Cloud | `f30f5825-e2b2-4688-a295-b72301af467f` | Classifies alert severity → JSON `{severity, confidence, reasoning}` |
| Threat Analyzer | Flowise Cloud | `2b7b85b7-f7d6-4d2d-99de-97da2b34d9ee` | Analyzes threat → JSON `{attack_type, indicators, mitre_techniques, ...}` |
| Response Recommender | n8n webhook | `/webhook/response-recommender` | Recommends actions → JSON `{immediate_actions, investigation_steps, ...}` |

**n8n Pipeline:** "Week 8 - LLM Chain Pipeline" (ID: `JVcfpTbIodYXVulB`) calls all 3 chains sequentially on a test SSH brute-force + privilege escalation alert.

**Note on Response Recommender:** Flowise Cloud free tier allows 2 chatflows max (both slots used by Classifier + Analyzer). The 3rd chain is implemented as an n8n webhook workflow calling Gemini 2.5 Flash — functionally identical to a Flowise chain from n8n's perspective.

---

### Part 2: AI-Assisted Capstone Development

| File | Description |
|------|-------------|
| `.github/copilot-instructions.md` | Project context file that grounds GitHub Copilot in our capstone's architecture, schema, and conventions |
| `docs/checkpoint2-audit.md` | Full Checkpoint 2 readiness audit — 10-question AI interview + gap analysis |
| `ai-core-README.md` | Component README generated with AI assistance (Part 2.4 artifact) |
| `prompt-log-alexander.md` | Log of AI interactions with context, prompts, results, and evaluations |

---

## Screenshots

See `screenshots/README.md` for instructions on what to capture and where to save each file.

---

## Key Technical Notes

- Flowise LLM Chain node requires a separate `chatPromptTemplate` node (3 nodes total per chain, not 2 as shown in homework)
- Gemini 2.5 Flash wraps JSON in markdown code fences even when instructed not to — strip with regex in Code node
- n8n webhook nodes require `webhookId` (UUID) field to register correctly — missing this causes silent 404
- n8n activate: use `POST /rest/workflows/{id}/activate` with `{versionId}` — PATCH with `active:true` silently fails
