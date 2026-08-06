---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: W1 inter-stage orchestration was never implemented (D08-DEF-004) — Remediated in source
Classification: Accepted
Artifact: implementation/workflows/, deployment/test-runtime-bindings.md
Date: 2026-08-06
---

**Reason:** Pipeline-topology inventory found that **the only `executeWorkflow` calls in the entire implementation targeted `error-handling`** — nothing called `w1-extraction`, `w1-ai-evaluation`, or `w1-duplicate-detection-recording`. The intake workflow terminated at "Upload to Google Drive" with no onward call. **The W1 chain the Blueprint requires was never wired.**

The Blueprint is unambiguous that it is required: §4 describes W1 as a pipeline (extraction operates on "the stored PDF"), and §6 sequencing states AI Evaluation "depends on 2 and 4" and Duplicate Detection "depends on 2 and 5". W2 is separately and explicitly independent ("Never fires from W1 directly"), so its lack of an inbound call is correct and was left alone.

**This is a Blueprint-conformance defect that survived D02 `/build`, D03 static testing, and three D08 rounds.** D03's T3 obligation ("topology matches Blueprint prose") passed because it checked each workflow's *internal* topology; no obligation checked topology *between* workflows. It could only surface once assembly was attempted.

**Remediated in source** per the Blueprint: intake → extraction → AI evaluation → duplicate detection, each guarded by the existing success branch. **D08-DEF-005** was found alongside it: every `executeWorkflow.workflowId` was a bare **name string** where the runtime requires a resourceLocator — the same shape-class as D08-DEF-002/003, now a fourth instance. Both remediated; all 6 files remain well-formed JSON.

**Environment-neutrality preserved:** targets are `<BIND:token>` placeholders in the correct runtime shape, resolved at deploy time via `deployment/test-runtime-bindings.md`. **No n8n workflow ID was written into implementation source.**

**Confidence:** High — the absence was verified by exhaustive grep across all artifacts, and the requirement by direct Blueprint citation.
**Risk:** The remediated chain is **not yet deployed or executed**; wiring correctness is proven structurally, not at runtime.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, blueprint-conformance, cross-artifact-defect, remediated-in-source, not-yet-runtime-verified, provenance:asdp-automated]
