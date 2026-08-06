---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Google Sheets parameter-shape defect class remediated and runtime-verified — Passed
Classification: Accepted
Artifact: implementation/workflows/ (4 files, 7 nodes), deployment/test-runtime-results.md
Date: 2026-08-06
---

**Reason:** **Root cause established from provider schema plus runtime behavior, not guessed.** The n8n Google Sheets v4 node requires `documentId` and `sheetName` as **resourceLocator objects** and `matchingColumns` **nested inside `columns`**. Every one of PRJ-0001's 7 googleSheets nodes supplied `sheetName` as a plain **string**, omitted the required `documentId` **entirely**, and (where present) placed `matchingColumns` at the **top level** — one shared defective assumption across 4 workflows, which is why D08-DEF-002 and D08-DEF-003 were the same class rather than two incidents.

**Remediated across all 7 nodes** in `error-handling.json`, `w1-ai-evaluation.json`, `w1-duplicate-detection-recording.json`, `w2-human-review-notification.json`; verification script confirms **0 defective nodes remain** and all 6 workflow files remain well-formed JSON. `documentId` is bound to the TEST spreadsheet and each affected node carries an explicit TEST-BINDING note plus a workflow-level `meta` entry — an environment binding, **never a client or Production value**, to be rebound before any Production deployment.

**Proven by actual external state, not invocation success.** A probe applying the corrected shape wrote a row and the read-back returned **all 13 schema field names as keys with values in their intended columns** (`row_number: 2`), confirming both the corrected `appendOrUpdate` shape and the correctly-established 13-column header. The remediated `error-handling` candidate was then **redeployed** to the TEST instance and re-validated: previously `errorCount: 1` ("Expected object but got string"), now **`valid: true`, `errorCount: 0`**.

**Confidence:** High — schema-derived root cause, runtime-proven fix, verified by resource read-back and by clean re-validation of the redeployed candidate.
**Risk:** The TEST `documentId` binding must be rebound for Production; recorded in-artifact so it cannot be missed silently.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, defect-class, remediated, redeployed, read-back-verified, provenance:asdp-automated, env:founder-local-n8n]
