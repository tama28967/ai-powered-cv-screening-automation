---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Full 6-workflow pipeline deployed; system-level topology verified — Passed
Classification: Accepted
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-06
---

**Reason:** All 6 workflows deployed to the Founder-Designated TEST environment in **callee-first** dependency order derived from the actual source call graph (dedup → AI evaluation → extraction → intake; W2 independent) — an order that **inverts** the sequence assumed in the task brief, because callers require their callees' runtime IDs. Every `<BIND:>` token resolved to a real n8n ID; **zero unresolved tokens**. All 6 validate at `errorCount: 0`.

**System-level topology verified against the deployed AND published definitions** — the check whose absence allowed D08-DEF-004 to survive four prior rounds. Confirmed: Intake → Extraction → AI Evaluation → Duplicate Detection; all four W1 stages → Error Handling; W2 independent with no inbound W1 edge (correct per Blueprint §4). Every expected edge present, every target resolving to the intended PRJ-0001 TEST workflow, no stale ID, no cross-Project target, no unintended extra edge.

**Three further provider-contract defects found by preflight and remediated:**

- **D08-DEF-006** — filter conditions omitted the required `operator.type`, and the switch node omitted `operator` entirely; 9 conditions across all 6 workflows repaired.
- **D08-DEF-007** — `googleSheets` resource `sheet` has no `lookup` operation; corrected to `read` + `filtersUI`.
- **D08-DEF-008** — `googleSheetsTrigger` rejects `sheetName.mode: "name"` although the non-trigger Sheets node accepts it — a per-node variance no cross-node assumption would predict.
- **D08-DEF-009** (deployment constraint, not a source defect) — n8n refuses to publish a caller whose sub-workflows are unpublished, forcing callee-first publication.

All fixes applied to source and to the deployed candidate; all 6 files remain well-formed JSON.

**Confidence:** High — topology read from deployed/published definitions, not inferred from source.
**Risk:** Topology and validation are proven; **no end-to-end execution has occurred**, so runtime data-contract behavior between stages remains unproven.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, pipeline-assembled, topology-verified, provider-contract-defects, provenance:asdp-automated, env:founder-local-n8n]
