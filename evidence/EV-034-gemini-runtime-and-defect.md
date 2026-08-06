---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Gemini runtime path proven; Sheets append defect found by read-back — Partially Passed
Classification: Deferred
Artifact: deployment/test-runtime-results.md
Date: 2026-08-06
---

**Reason:** **First real LLM execution in this Project's history.** A real webhook-triggered n8n execution called Gemini via the `googleGemini` node at model **`models/gemini-2.5-flash`** — the ADR-0018 preferred default — and received exactly the expected reply with `finishReason: STOP` in 3.15s. Free tier sufficed; no paid tier was reached and no billing was enabled. This proves LLM *connectivity and integration* only; **AI semantic correctness remains BLOCKED on the client's evaluation criteria (OTQ1)** — a different question, reported separately and not inferred from connectivity. Separately, **D08-DEF-003**: a Sheets `append` in raw mode reported HTTP 200, execution `status: success`, and `itemsOutput: 1`, yet read-back showed the sheet had received the webhook's own payload as columns instead of the intended 13-column header — the supplied `rawData` expression did not take effect. Only reading the actual resource state revealed this, a first-party demonstration that invocation success is never proof of runtime correctness. Same defect class as D08-DEF-002 (`googleSheets` parameter shape), now observed in two independent artifacts — a pattern, recorded for `/learn`. Classified `Deferred` because the Sheet's header state is left unresolved rather than failed.

**Confidence:** High for the Gemini result and for the defect, both directly observed.
**Risk:** The Sheet contains one junk row, recorded honestly rather than silently cleaned; the header row is not correctly established.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, runtime, gemini-2.5-flash, runtime-defect, provenance:asdp-automated, env:founder-local-n8n]
