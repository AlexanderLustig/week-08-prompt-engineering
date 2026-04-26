# Prompt Log — Alexander Lustig

**Project:** AI-Powered Security Alert Triage System
**Team:** Week 8 capstone project
**My Component:** AI Core
**AI Tools Used:** GitHub Copilot Chat, Claude (claude.ai)

---

## How to Use This Log

Add an entry for each significant AI interaction:
- Copilot Chat conversations where you asked it to generate, explain, or debug something
- Moments where Copilot was wrong and you had to fix it (these are the most valuable entries)
- Cases where you refined a prompt to get a better result

Don't log: every autocomplete of a bracket or variable name.

---

## 2026-04-26 — Capstone checkpoint 2 audit

**Context:** Working on Week 8 homework Part 2.3. Had `copilot-instructions.md` open in VS Code with full project context (Airtable schema, component breakdown, current status). Asked Copilot to conduct the 10-question audit interview.

**Prompt:**
> I need you to act as a capstone project advisor for a university AI integration course. I'm going to describe my group project, and I need you to interview me about its current state, then produce a structured gap analysis.
> 
> My project uses: n8n (workflow automation), Flowise (LLM chains), Airtable (database), Groq/HuggingFace (AI APIs), and GitHub.
> 
> The team has 4 components: Ingestion, AI Core, Specialist, and Integration.
> 
> Checkpoint 2 is next week. The requirement is: one complete record flows through all 4 components end-to-end without manual intervention.
> 
> Please interview me with the following questions, ONE AT A TIME. [full 10-question prompt]

**Result:** Copilot produced a well-structured 10-question interview and followed up with a full Checkpoint 2 Readiness Assessment. Correctly identified the Ingestion → AI Core handoff as the critical blocker. Produced specific action items including the exact Airtable polling filter expression to use (`status = pending_classification`). Full output saved to `docs/checkpoint2-audit.md`.

**Evaluation:** Mostly accurate. Copilot correctly understood that the AI Core chains exist but the polling trigger doesn't, making end-to-end flow impossible. It incorrectly suggested building the Specialist component before building the polling trigger — wrong priority order. The fix list needed manual reordering.

**What I changed:** Moved "build AI Core polling trigger" to item #1 and "run end-to-end test" to item #3. Removed a suggestion to add a Redis cache for chain results (out of scope for this project). Added specifics about n8n PATCH vs POST activate API (common gotcha Copilot doesn't know about).

**What I learned:** Copilot's gap analysis is accurate when you give it the real schema with real field names and real status values. A generic description produces generic gaps. The detail in `copilot-instructions.md` was the difference between "test your handoffs" (useless) and "make sure Ingestion writes `status: pending_classification` not `pending classification`" (actionable).

---

## 2026-04-26 — AI Core component README generation

**Context:** Needed to write a full README for the AI Core component for the capstone repo. Had all three chains tested and working, knew the exact Flowise IDs, the n8n webhook URL, and the known limitations (Gemini code fences, Flowise free tier limits).

**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Core component. Include:
> - What it does (2-3 sentences)
> - How it connects to other components (inputs and outputs)
> - Setup instructions (what accounts/keys are needed, what to configure in n8n/Flowise)
> - How to test it (specific steps with curl commands)
> - Known limitations

**Result:** Copilot generated a well-structured README with working curl commands for all three chain endpoints, correct field names from the Airtable schema, and a logical setup flow. The curl command for the Alert Classifier had the correct Flowise prediction URL format.

**Evaluation:** 8/10. The structure and curl commands were correct. It invented a "Redis caching step" in setup that doesn't exist in our project. The "Known Limitations" section was too generic ("may not work for all alert types") and needed replacing with the actual technical limitations I encountered.

**What I changed:** Removed the invented Redis step. Replaced the generic limitations with actual ones: Flowise Cloud 2-flow limit, Gemini code fence stripping, Railway cold start delay, MITRE technique accuracy issues. Added the exact n8n workflow IDs for reference.

**What I learned:** Copilot gets the structure right from the template but fills gaps with plausible-sounding fabrications. The curl commands came directly from the context I provided (chatflow IDs were in the instructions file). Details it invented (Redis, generic limitations) were things not in the context — if it's not in `copilot-instructions.md`, Copilot guesses. Keep the instructions file current and complete.
