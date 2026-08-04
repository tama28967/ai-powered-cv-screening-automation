# Engineering Blueprint: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation

Fills [engineering-blueprint-specification.md](../../ASDP/frameworks/claude-engineering-framework/.claude/knowledge/engineering-blueprint-specification.md). CEF's output — decides *how* to build; does not build, schedule, or re-open business decisions.

## 1. Engineering Header
- **Project:** PRJ-0001 (source Opportunity: OP-0002)
- **Source Briefcase:** `Workspace/projects/PRJ-0001/briefcase.md` (consumed immutably, never rewritten)
- **Date:** 2026-08-03 · **Status:** Draft — pending resolution of open external inputs (§9)

## 2. Engineering Intelligence
From the Briefcase's Solution Context: this is a document-intake-and-evaluation pipeline with a hard sequential dependency chain (intake → validate → store → extract → evaluate → dedup → record → human gate → notify), one AI reasoning step whose input (evaluation criteria) is explicitly not yet fixed, and one data-quality risk (PDF text extractability) that is not fully known. The two named "nice to haves" that the accepted Proposal made in-scope — a human review gate and configurable criteria — are not incidental features; they are the load-bearing constraints the whole design has to be built around, not appended afterward. Everything downstream in this Blueprint treats them as first-class, not optional.

The engineering problem decomposes into three concerns of different character: (a) a linear intake/storage/extraction pipeline — mechanical, low-risk; (b) an AI evaluation step whose correctness depends on an input CEF does not own (criteria); (c) a review-and-notify step that must not fire until a human has acted — a control-flow problem, not just a feature.

## 3. Architecture
**Selected approach: single n8n-orchestrated pipeline, decomposed into modular sub-workflows, split into two independently-triggered workflows at the human review gate.**

Rationale: the stated volume (50–300/month), fixed $600 budget, and the client's explicit "easy to maintain and extend" requirement all favor the simplest architecture that satisfies the constraints — n8n-native nodes plus Google Workspace APIs, with no additional hosted infrastructure. n8n is also the client's stated preferred stack (Solution Context, Briefcase §4).

**Alternatives considered:**
- *n8n + external microservice* (e.g., a hosted function for PDF extraction/LLM calls). Rejected: introduces a second hosting surface and operational dependency the client did not ask for and the fixed-price budget does not obviously support; no stated requirement needs the extra scalability it would buy at this volume.
- *Single unbroken n8n workflow, review gate implemented as an in-workflow wait step.* Rejected: would hold an n8n execution open indefinitely pending human action, which is poor operational practice (execution timeouts, resource retention) and worse for the "easy to maintain" goal than two smaller, independently-triggered workflows.

The chosen shape follows the Assessment's own Likely Build Shape (Briefcase §4): Form & Intake → Validation & Drive Storage → Resume Extraction → AI Evaluation & Scoring → Sheets Recording & Duplicate Detection → Notification Layer → Error Handling & Modularization → Documentation & Deployment Handover — realized here as concrete sub-workflows in §4.

## 4. System Decomposition
- **W1 — Intake, Validation & Storage:** form trigger receives application + PDF; validates file type/size/readability; on pass, uploads to Google Drive under a structured naming convention (applicant + timestamp); on fail, routes to the Error Handling sub-workflow.
- **W1 — Extraction:** extracts text content from the stored PDF; a non-extractable PDF (e.g., scanned image) routes to Error Handling rather than failing silently.
- **W1 — AI Evaluation:** reads current evaluation criteria from a configurable source (§8); constructs the evaluation prompt; calls the LLM; parses the response into a score and recommendation with a defined output contract (not free text).
- **W1 — Duplicate Detection & Recording:** checks the new applicant against existing Sheets records on a defined dedup key (§9); writes one structured record (applicant data, extracted content reference, score, recommendation, dedup flag, review status = "Pending").
- **W2 — Human Review Gate & Notification:** triggered independently (by a reviewer marking a record's status in Sheets), reads the reviewed record, and sends the appropriate candidate notification email. Never fires from W1 directly — decouples "record written" from "candidate notified."
- **Error Handling (shared):** a central n8n error-handling workflow, wired to every sub-workflow, covering the three named failure modes (unreadable PDF, duplicate application, extraction failure) plus LLM call/parse failure.

## 5. Dependency Analysis
**Internal:** strict sequential dependency W1 stages (intake → validation → storage → extraction → evaluation → dedup → recording); W2 depends on a record existing in "Pending" state from W1, and is otherwise independent of W1's execution timing.

**External:** Google Drive API, Google Sheets API, Gmail/email API, LLM provider API, n8n instance, and the form intake mechanism (native n8n form vs. external service — a build-time choice per the Briefcase, Solution Context confidence: Medium).

**Ordering constraints:** credential provisioning (Google Workspace scopes, LLM API key, n8n access) must precede any sub-workflow build; the Sheets record schema and the Criteria Config schema (§8) must be frozen before the AI Evaluation and Recording sub-workflows are built, since both consume them. The dedup key (§9 — applicant email, an engineering assumption) needs no external confirmation before the Recording sub-workflow's lookup logic is finalized.

## 6. Implementation Sequencing
Decision-altitude build order, not a dated schedule (scheduling is Delivery Framework's, named not produced):
1. Environment & credential setup (n8n access, Google Workspace API scopes, LLM API key).
2. Data schema design — Sheets record schema and Criteria Config schema.
3. W1: Intake, Validation & Storage.
4. W1: Extraction.
5. W1: AI Evaluation (depends on 2 and 4).
6. W1: Duplicate Detection & Recording (depends on 2 and 5; the dedup key is already set as an engineering assumption — §9 — and needs no external confirmation).
7. W2: Human Review Gate & Notification.
8. Central Error Handling wired across W1/W2.
9. Documentation & deployment instructions.
10. End-to-end validation pass.

## 7. Technical Risks & Mitigations
- **Non-text (scanned) PDFs fail extraction.** Mitigation: validation/extraction stage detects non-extractable PDFs and routes to Error Handling rather than passing bad data downstream; OCR is out of current scope (Open Question, §9 — depends on external input on actual PDF mix).
- **LLM response isn't reliably structured.** Mitigation: constrain the evaluation call to a defined output contract (e.g., structured/JSON response) with a parsing step and an explicit error branch on malformed output, rather than trusting free-text parsing.
- **Dedup logic under- or over-matches applicants.** Mitigation: dedup key must be explicit and documented, not implicit (§9); false-negative risk (same person, different email) is named, not silently accepted.
- **Review gate becomes a bottleneck.** Not an engineering defect — decoupling W1/W2 means a slow reviewer delays notification but never blocks intake, evaluation, or recording. Documented for the client as an operational dependency, not solved by engineering.
- **n8n hosting/version unknown.** Mitigation: build with standard, version-broadly-compatible node patterns; confirm and document the actual n8n version once hosting is known (§9).
- **Recurring LLM cost at volume.** Already disclosed to the client in the accepted Proposal (Briefcase §6); not a build risk, named here only so cost-monitoring guidance is not dropped from the deployment documentation.

## 8. Engineering Trade-offs
- **Single n8n orchestrator vs. external microservice split:** simplicity and low operational overhead chosen over headroom for scale the client hasn't asked for, appropriate at 50–300/month.
- **Google Sheets as the datastore vs. a dedicated database:** Sheets chosen — it's already required by the client and adds zero new infrastructure — at the cost of a query/scale ceiling; named as a future constraint if volume grows materially, not a current problem.
- **Two independently-triggered workflows (W1/W2) vs. one linear workflow with an in-line wait:** the split avoids holding an execution open pending human action, at the cost of building and wiring one extra workflow — favored for reliability and operational simplicity over minimizing workflow count.
- **Criteria read from a configurable Sheet vs. hardcoded in the workflow:** a configurable source is chosen to satisfy the client's explicit "adjust without a rebuild" requirement, at the cost of slightly more prompt-construction logic in the AI Evaluation sub-workflow.

## 9. Assumptions & Open Technical Questions
**Explicit engineering assumptions (CEF's to make; not blockers — Capability Profile deferred, so a stack choice is marked assumption, never asserted as fact):**
- **LLM provider** is not asserted here. A provider capable of constrained/structured output will be selected at build time; its recurring per-resume API cost is already disclosed to and accepted by the client (Briefcase §6). This is CEF's assumption to make at build time, not a blocker to Blueprint completion.
- **Duplicate-detection key** — applicant email, on the assumption the form requires and uniquely captures it. This is a technical implementation choice grounded in the stated form-intake requirement; it does not require client business judgment, so it stays an engineering assumption rather than an external blocker. Revisit only if the form's actual field design turns out not to capture email as a required, unique field.

**Open Technical Questions — external input required, not CEF's to supply (per `briefcase-intake.md`, labelled rather than invented):**
1. **Evaluation criteria** — what specifically makes a candidate score well. Business-owned, already flagged in the Briefcase (§10) as the single largest unknown carried from Assessment; the architecture is built to consume criteria from a configurable source regardless of the final values, so this does not block the Blueprint, but it does block finalizing the AI Evaluation prompt.
2. **Resume format mix** — predominantly text-based PDFs, or a meaningful share of scanned images. Determines whether OCR is in scope; current design assumes text-extractable PDFs per the Briefcase's own labelled assumption.
3. **n8n instance** — does one already exist, or must one be stood up, and on what hosting. Affects sequencing step 1 and the exact node/version guidance in deployment documentation.

None of these three re-open the accepted scope or the business decision — they are inputs the accepted scope already anticipated as unresolved (Briefcase §10, Known Unknowns) and that implementation cannot proceed past without.

## 10. Handoff Notes
For a future implementation capability and the Delivery Framework (context only — no schedule, resourcing, or pace set here): the credential checklist in §5; the two schemas (Sheets record, Criteria Config) to be frozen once Open Question 1 is answered; the dedup key (§9 — applicant email, already set as an engineering assumption, no external confirmation needed); the confirmed n8n hosting target from Open Question 3 before sequencing step 1 can complete; and the accepted delivery window (5–7 business days, Briefcase §6) as a Delivery Framework input, not scheduled by this Blueprint.

---

## Architecture Review

- **Status** — 🟡
- **Reason** — Architecture, decomposition, sequencing, and risk analysis are complete and internally consistent with the Briefcase; nothing here re-opens scope or fabricates a stack. Yellow rather than green because three Open Technical Questions (§9) — evaluation criteria, resume format mix, n8n hosting — are external inputs implementation cannot proceed past, none of which CEF can resolve on its own. (The duplicate-detection key, previously listed as a fourth blocking item, is corrected as of Sprint D00 — Canonical Cleanup — to an engineering assumption per §9; it required no client business judgment and should not have been listed as an external blocker.)
- **Debt** — None yet incurred (no code exists); the three Open Technical Questions are the only unresolved items, and they are inputs, not defects.
- **Recommended next steps** — Resolve the three Open Technical Questions in §9 (all external/client inputs; the evaluation-criteria question is the same one already flagged to the client in the accepted Proposal's Close). Once resolved, sequencing step 1 (credentials) can begin. Scheduling that work is the Delivery Framework's, not this Blueprint's.

---
*Packaging only insofar as it consolidates CEF's own reasoning. No business reasoning, scope change, or Briefcase rewrite. Execution of this Blueprint is a future capability.*
