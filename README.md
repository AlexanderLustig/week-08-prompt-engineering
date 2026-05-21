# Prompt Engineering & AI-Assisted Development

**What this demonstrates:** How to iterate on LLM outputs using prompt engineering, and how to use AI tools (GitHub Copilot) to accelerate technical development.

This repo captures two independent but related projects:

## Part 1: Multi-Chain LLM Pipeline for Alert Triage

### The Problem
Security alerts need intelligent triage: What's the severity? What type of attack? What should I do about it? A single LLM can do this, but breaking it into steps (classify → analyze → recommend) gives more reliable, structured output.

### What I Built

**Three chained LLM models** that work together on a security alert:

| Chain | Purpose | Output Format |
|-------|---------|---|
| **Alert Classifier** | Assess severity (critical/high/medium/low) with confidence | JSON: `{severity, confidence, reasoning}` |
| **Threat Analyzer** | Identify attack type + MITRE techniques + indicators of compromise | JSON: `{attack_type, indicators, mitre_techniques, severity_justification}` |
| **Response Recommender** | Suggest immediate actions and investigation steps | JSON: `{immediate_actions[], investigation_steps[]}` |

### Implementation

- **Chains 1-2:** Built in Flowise Cloud using OpenAI `gpt-4o-mini`
- **Chain 3:** Implemented as n8n webhook (Flowise free tier limit = 2 flows), calling Gemini 2.5 Flash
- **Orchestration:** n8n pipeline runs all 3 chains sequentially on a test alert (SSH brute force + privilege escalation)
- **Key Learning:** Different LLMs have different strengths; Gemini 2.5 Flash was better at structured reasoning than OpenAI for this use case

### How to Run

1. Open the n8n workflow: `JVcfpTbIodYXVulB` on [Railway n8n](https://primary-production-bbbc9.up.railway.app)
2. Trigger manually with a test alert JSON
3. Observe output from all 3 chains in sequence

---

## Part 2: AI-Assisted Capstone Development

### The Problem
Large capstone projects need clear documentation, consistent architecture, and continuous validation. Manually writing all of this is slow and repetitive.

### What I Did

1. **GitHub Copilot Context File** (`.github/copilot-instructions.md`)
   - Documented capstone schema, conventions, and architecture
   - Grounded Copilot in the project's context so it generates consistent code

2. **Checkpoint 2 Readiness Audit** (`docs/checkpoint2-audit.md`)
   - Used Claude to conduct a "10-question AI interview"
   - Identified gaps in architecture, testing, and documentation
   - This is research-backed: structured gap analysis approach based on technical readiness frameworks

3. **Component README** (`ai-core-README.md`)
   - Generated with AI assistance but reviewed and refined by hand
   - Shows how AI can accelerate documentation without sacrificing accuracy

4. **Prompt Log** (`prompt-log-alexander.md`)
   - Artifacts capturing how I actually direct AI tools
   - Context → Prompt → Result → Evaluation → Learning for each interaction

### Key Insight

AI tools are multiplicative when you provide good context. The better you write the initial setup (architecture doc, schema definitions, conventions), the better the AI output. This is fundamentally about communication with machines.

---

## The Prompt Log (Portfolio Artifact)

My `prompt-log-alexander.md` shows:
- How I break complex problems into AI-solvable chunks
- Where AI succeeds (code generation, documentation drafts, gap analysis)
- Where AI needs human judgment (architecture decisions, prioritization)
- How I evaluate AI output (does it match the spec? is it production-ready?)

This log is unusual—most portfolios don't show this. It demonstrates that I know how to work WITH AI, not just use it.

---

## Stack

- **LLM Chains:** Flowise Cloud (OpenAI gpt-4o-mini)
- **Orchestration:** n8n
- **AI-Assisted Development:** GitHub Copilot, Claude
- **Documentation:** Markdown + handwritten refinement

---

## Files in This Repo

```
├── README.md                         ← You are here
├── prompt-log-alexander.md           ← My AI interaction log (portfolio artifact)
├── .github/copilot-instructions.md   ← Project context for Copilot
├── docs/checkpoint2-audit.md         ← Structured readiness audit
├── ai-core-README.md                 ← AI-generated component README
└── screenshots/                      ← LLM chain execution screenshots
```

---

## What I Learned

1. **Prompt engineering is iterative** — First drafts from LLMs are rarely production-ready. You need to evaluate, refine, and iterate.
2. **Structure beats cleverness** — Well-formatted JSON prompts > conversational prompts. When I need structured output, I specify the schema.
3. **Context is everything** — A vague prompt to a generic LLM produces vague output. Grounding with domain knowledge (security frameworks, capstone architecture) produces usable results.
4. **AI is a multiplier, not a replacement** — AI doesn't replace judgment; it speeds up execution once you know what you want.
5. **Three-chain architecture is more robust than one chain** — Separating concerns (classify → analyze → act) prevents a single bad prompt from derailing the entire pipeline.

---

## Portfolio Signal

This repo demonstrates:
- **Prompt engineering chops:** I know how to structure prompts for reliable LLM output
- **System design:** Chaining models for robustness
- **AI-aware development:** I use AI tools effectively, and I document how
- **Technical writing:** Clear documentation of architectural decisions
- **Critical evaluation:** I don't blindly trust AI; I validate output against requirements

This is what "AI Integration Specialist" means in practice.
