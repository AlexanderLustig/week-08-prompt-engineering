# Checkpoint 2 Readiness Assessment
*AI-assisted audit generated via GitHub Copilot Chat on 2026-04-26*
*Using context from `.github/copilot-instructions.md`*

---

## Interview Questions & Answers

**Q1: What is your project name and what problem does it solve?**
The project is an AI-Powered Security Alert Triage System. It solves the problem of alert fatigue — security teams receive hundreds of raw log entries and SIEM alerts daily, most of which are benign. The system automatically classifies severity, identifies attack type, and generates concrete response recommendations so analysts focus only on what matters.

**Q2: What is your role/component?**
AI Core — responsible for the three LLM chain pipeline: Alert Classifier, Threat Analyzer, and Response Recommender. My component reads `status: pending_classification` records from Airtable, runs them through Flowise chains, and writes structured results back.

**Q3: Describe your Airtable schema.**
Two tables: `security_alerts` (primary table, 20 fields including alert_text, severity, attack_type, indicators, mitre_techniques, immediate_actions, status) and `incidents` (linked table for correlated multi-alert incidents). Status field drives handoffs: `pending_classification` → `classified` → `enriched` → `reported`.

**Q4: What is currently WORKING?**
- All three Flowise chains produce valid JSON output (tested via API)
- n8n "Week 8 - LLM Chain Pipeline" calls all 3 chains in sequence end-to-end
- Response Recommender webhook on n8n Railway instance is active and responding
- Flowise Cloud has Alert Classifier and Threat Analyzer deployed

**Q5: What is NOT YET WORKING?**
- Ingestion component: webhook → Airtable write not built yet
- AI Core polling trigger: n8n doesn't yet poll Airtable for pending records
- Specialist: MITRE correlation logic not started
- Integration: Slack notification workflow not started
- End-to-end test with a real Airtable record: not done

**Q6: Tested Ingestion → AI Core handoff?**
No. The Ingestion webhook doesn't write to Airtable yet. AI Core has no trigger that polls Airtable. These two facts together mean the handoff is completely untested and the data flow is broken.

**Q7: Tested AI Core → Specialist handoff?**
No. The AI Core writes results back to Airtable (`status: classified`) but the Specialist workflow doesn't exist yet to pick them up.

**Q8: Field name mismatches?**
Potential issue: AI Core writes `indicators` as a JSON array string (e.g. `'["SSH attempts","192.168.1.45"]'`) but the Specialist may try to read it as a native Airtable multi-select field. Needs explicit agree-on-format before Specialist builds their polling query.

**Q9: How many test records in Airtable?**
Approximately 3 test records created manually — all normal cases. No edge cases (very long alert text, non-English characters, empty fields) and no bad data cases.

**Q10: Biggest worry about Checkpoint 2?**
The Ingestion → AI Core handoff is untested and requires two components built by different people to agree on field names and the trigger mechanism. If Ingestion writes to the wrong field or AI Core polls the wrong status value, nothing flows through.

---

## Checkpoint 2 Readiness Assessment

### Status: AT RISK

### What's Working
- Alert Classifier Flowise chain: valid JSON with `severity`, `confidence`, `reasoning`
- Threat Analyzer Flowise chain: valid JSON with `attack_type`, `indicators`, `mitre_techniques`
- Response Recommender n8n webhook: valid JSON with `immediate_actions`, `containment_strategy`
- n8n pipeline calls all 3 chains sequentially with correct data passing (`$json.text` handoff)
- Airtable base exists with correct schema

### Critical Gaps (must fix before Checkpoint 2)
- **[BLOCKER] AI Core polling trigger missing** — build an n8n Schedule Trigger that polls Airtable for `status = pending_classification`, runs the 3-chain pipeline, writes results back. Owner: Alexander (AI Core)
- **[BLOCKER] Ingestion webhook not writing to Airtable** — Ingestion must write the initial record with `status: pending_classification` for AI Core to pick up. Owner: Ingestion team member
- **[BLOCKER] No end-to-end test** — no record has ever flowed through all 4 components. Must complete at least one before Checkpoint 2. Owner: All team members
- **[HIGH] Specialist workflow not started** — without this, AI Core output has nowhere to go after `status: classified`. Owner: Specialist team member

### Schema Issues Found
- `indicators` field type agreement needed: AI Core writes a JSON-stringified array — confirm Specialist reads it the same way (Long Text, not multi-select)
- `mitre_techniques` same issue — must be stored as JSON string, not native array
- Confirm `status` field values exactly match across all components: use `pending_classification` (with underscore), not `pending classification` (with space) or `Pending Classification` (capitalized)

### Recommended Fix Order
1. **(2 hrs) Alexander: Build AI Core Airtable polling trigger** — n8n Schedule Trigger (every 5 min) → Airtable "List Records" filtered by `status = pending_classification` → existing 3-chain pipeline → Airtable "Update Record" with results + `status: classified`
2. **(1 hr) Ingestion team: Fix webhook → Airtable write** — ensure it creates a record with `alert_text`, `source_ip`, and `status: pending_classification` matching schema exactly
3. **(2 hrs) Run end-to-end test** — submit a test alert via Ingestion webhook, watch it flow through AI Core, confirm Airtable record updates correctly
4. **(2 hrs) Specialist: Build basic MITRE correlation** — even a stub workflow that reads `status: classified` and updates to `status: enriched` is enough for Checkpoint 2
5. **(1 hr) Integration: Build basic Slack notification** — trigger on `status: enriched`, send a Slack message with the alert summary

### Test Data Gaps
- No edge case records — add: alert with empty `source_ip`, alert with 2000+ char `alert_text`, alert in a language other than English
- No bad data records — add: record with `alert_text = null`, record with invalid severity value
- Add 10+ normal case records to test AI Core polling at scale
- Specific record to add: `{"alert_text": "User admin logged in from 10.0.0.1 at 9am", "source_ip": "10.0.0.1", "status": "pending_classification"}` — should classify as INFO confidence ~0.9
