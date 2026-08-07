---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Form Trigger root-caused as a provider-core defect; Webhook Trigger adopted as Blueprint-conformant engineering adaptation — Ratified (engineering-level, TEST scope)
Classification: Ratified
Artifact: implementation/workflows/w1-intake-validation-storage.json (source not yet synced — see note), deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** D08 round 8 was the final authorized Form Trigger diagnostic round. This record captures the root-cause finding, the engineering decision it forced, and why that decision is Blueprint-conformant rather than a business-scope change.

**Root cause (installed n8n 2.33.5 package source, read-only inspection inside the running container):** `n8n-nodes-base`'s `FormTriggerV2` node declares its production route via **two webhook descriptors** (`GET`/`setup` and `POST`/`default`), both explicitly tagged `nodeType: 'form'` — a distinct registration path from the ordinary `Webhook` node's single, untagged descriptor. A live runtime log capture (`docker logs`) during a controlled activation showed n8n's own dispatcher responding to a Form Trigger GET request with `"Received request for unknown webhook: ... is not registered"` — direct evidence that the `nodeType: 'form'`-tagged route was never persisted into the live registry, while an identically-activated plain Webhook node registered and served immediately in the same test. This reproduced identically at n8n 2.31.7 and 2.33.5 (EV-042, EV-044), across typeVersion 1 and 2.2, and on both localhost and the public host — ruling out proxy/DNS, activation lifecycle, node-shape, and version as the cause.

**Root-cause classification: B — CONFIRMED, NOT PRACTICALLY REMEDIABLE.** Fixing this would require patching n8n's own installed `n8n-nodes-base` package (out of the authorized TEST mutation boundary, and any fix would be silently reverted by the next image pull/update, so it is not a durable remediation even if attempted).

**Blueprint conformance check (performed before adopting the fallback, per instruction):** `engineering-blueprint.md` §4 describes the capability generically ("form trigger receives application + PDF...") and §"Solution Context" explicitly frames the mechanism as **"native n8n form vs. external service — a build-time choice per the Briefcase, Solution Context confidence: Medium."** This is stated as an open engineering choice, not a client-mandated technology. Therefore adopting `n8n-nodes-base.webhook` in place of `n8n-nodes-base.formTrigger` is an **ENGINEERING IMPLEMENTATION ADAPTATION**, not a business-scope change — verified against all 10 criteria in the round's Webhook Fallback Decision Rule: the business capability (receive Applicant Name, Applicant Email, Resume PDF and start the pipeline), the full downstream chain, W2, error handling, and security boundaries are all unchanged; only the trigger node type and its POST-to-`/webhook/prj0001-intake` invocation contract changed.

**Implementation (MAIN workflow, `4nDrnkEQ9I9j8mjw`):** `Form Trigger` node replaced with a `Webhook` node (`POST /webhook/prj0001-intake`, `multipart/form-data`, `options.binaryPropertyName: "resume_pdf"`). Downstream code nodes updated to read `$json.body['Applicant Name']`/`['Applicant Email']` (multipart text fields nest under `body`) with the original bare-key path kept as a fallback, and to reference `$('Webhook Intake')` instead of `$('Form Trigger')`.

**Deviation disclosed, not hidden:** the deployed MAIN workflow's entry mechanism is now Webhook, not Form Trigger, for the TEST environment. This is recorded here and in `deployment/test-runtime-bindings.md`; it is not silently presented as if Form Trigger were still in use.

**Confidence:** High on root cause (direct runtime log + source inspection, not inference); High on Blueprint conformance (direct textual citation, not interpretation).
**Risk:** Low — TEST-only workflow mutation, fully reversible (MAIN workflow versioned in n8n; no Production or client-facing artifact touched).
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, round-8, form-trigger, root-cause, webhook-fallback, blueprint-conformance, engineering-adaptation, provenance:asdp-automated, env:founder-local-n8n]
