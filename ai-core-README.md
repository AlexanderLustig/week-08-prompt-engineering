# AI Core Component — Security Alert Triage System

## What It Does

The AI Core component takes raw security alerts classified as `pending_classification` in Airtable and runs them through a three-stage LLM chain pipeline:

1. **Alert Classifier** — classifies severity (CRITICAL/HIGH/MEDIUM/LOW/INFO) with confidence score and one-sentence reasoning
2. **Threat Analyzer** — identifies attack type, key indicators, potential impact, and MITRE ATT&CK technique codes
3. **Response Recommender** — generates ordered lists of immediate actions and investigation steps, plus a containment strategy

Results are written back to the `security_alerts` Airtable table and the record status is updated to `classified` so the Specialist component can pick it up.

## How It Connects to Other Components

**Inputs (from Ingestion):**
- Reads Airtable `security_alerts` records where `status = pending_classification`
- Required fields: `alert_text` (the raw alert string), `source_ip` (optional)

**Outputs (to Specialist):**
- Updates the same Airtable record with: `severity`, `confidence`, `reasoning`, `attack_type`, `indicators` (JSON string), `potential_impact`, `mitre_techniques` (JSON string), `immediate_actions` (JSON string), `investigation_steps` (JSON string), `containment_strategy`, `escalation_needed`
- Sets `status = classified` to trigger the Specialist handoff

## Setup Instructions

### Required Accounts & Keys
- **Flowise Cloud** account at [cloud.flowiseai.com](https://cloud.flowiseai.com)
  - OpenAI API key configured as a Flowise credential
  - Two deployed chatflows: "Alert Classifier" and "Threat Analyzer"
  - Flowise API key (Bearer token for prediction API calls)
- **n8n** instance (Railway or n8n Cloud)
  - Gemini API key credential configured (type: `httpQueryAuth`, key name: `key`)
  - Airtable credential with read/write access to the shared base
- **Airtable** base — get the Base ID and Table name from your team

### n8n Workflows to Activate
1. **Week 8 - Response Recommender Chain** (`BCFavVIUAeRhFmEl`) — must be active, webhook at `/webhook/response-recommender`
2. **Week 8 - LLM Chain Pipeline** (`JVcfpTbIodYXVulB`) — the 5-node test pipeline (Manual Trigger version)
3. *(To build)* **AI Core Polling Trigger** — Schedule Trigger + Airtable read + pipeline + Airtable write

### Flowise Chatflow IDs
- Alert Classifier: `f30f5825-e2b2-4688-a295-b72301af467f`
- Threat Analyzer: `2b7b85b7-f7d6-4d2d-99de-97da2b34d9ee`
- Response Recommender: n8n webhook (not a Flowise chatflow)

## How to Test

### Test the Flowise chains individually
```bash
# Alert Classifier
curl -X POST https://cloud.flowiseai.com/api/v1/prediction/f30f5825-e2b2-4688-a295-b72301af467f \
  -H "Authorization: Bearer <FLOWISE_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"question": "Multiple failed SSH login attempts from 203.0.113.45 over 30 seconds"}'

# Expected: {"text": "{\"severity\": \"HIGH\", \"confidence\": 0.85, \"reasoning\": \"...\"}"}

# Threat Analyzer
curl -X POST https://cloud.flowiseai.com/api/v1/prediction/2b7b85b7-f7d6-4d2d-99de-97da2b34d9ee \
  -H "Authorization: Bearer <FLOWISE_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"question": "CRITICAL: Outbound DNS queries to known C2 domain from workstation-47"}'

# Expected: {"text": "{\"attack_type\": \"C2\", \"indicators\": [...], ...}"}

# Response Recommender (n8n webhook)
curl -X POST https://primary-production-bbbc9.up.railway.app/webhook/response-recommender \
  -H "Content-Type: application/json" \
  -d '{"question": "Recommend response for: C2 activity, high confidence"}'

# Expected: {"text": "{\"immediate_actions\": [...], ...}"}
```

### Test the full pipeline
1. Open n8n at `https://primary-production-bbbc9.up.railway.app`
2. Open workflow "Week 8 - LLM Chain Pipeline"
3. Click "Test Workflow"
4. Inspect each node output — all 5 nodes should show green checkmarks
5. Final node (Chain 3) output should contain `immediate_actions` JSON

### Verify output format
Each chain must return a `text` field containing valid JSON (no markdown code fences). Run:
```javascript
const response = /* chain output */;
JSON.parse(response.text); // Should not throw
```

## Known Limitations

- **Flowise Cloud free tier**: limited to 2 simultaneous chatflows. Alert Classifier and Threat Analyzer occupy both slots. Response Recommender runs as an n8n webhook instead.
- **Gemini code fences**: Gemini 2.5 Flash occasionally wraps JSON in ` ```json ... ``` ` code fences despite explicit instructions. The n8n Code node strips these automatically.
- **Rate limits**: Flowise predictions share the OpenAI API rate limit. If processing many records in parallel, add a 1-second delay between Airtable record iterations.
- **Cold start**: The Response Recommender n8n webhook may take 5–10 seconds on first call after the Railway container restarts.
- **MITRE techniques**: Gemini and GPT-4o-mini may return `"unknown"` for uncommon attack patterns rather than a valid T#### code.
