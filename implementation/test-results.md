# PRJ-0001 — Test Results

Produced by `/test`, per `testing-validation-standard.md` stages 6–8 (Test Execution, Failure Classification, Remediation). Every result below is evidence-backed — no PASS is reported for a test that did not actually run.

| # | Obligation | Method | Initial Result | Defect | Classification | Remediation | Retest |
|---|---|---|---|---|---|---|---|
| T1 | Schema/field consistency | Static cross-reference | **FAIL** | `w1-duplicate-detection-recording.json`'s Write nodes mapped only `duplicate_flag`/`review_status`, silently dropping `record_id`, `applicant_email`, `score`, `recommendation`, `evaluation_criteria_version` even though several were received as inputs (D03-DEF-001) | Implementation Defect | Rewrote both Write nodes to map every relevant received field | **PASS** |
| T2 | JSON well-formedness | `node -e "JSON.parse(...)"` (STATIC TEST — actually executed, not reviewed by eye) | PASS (all 6, both before and after remediation) | none | — | — | PASS |
| T3 | Topology matches Blueprint prose | Static inspection | PASS (5 of 6 on first pass); see T5/T10 | — | — | — | PASS |
| T4 | `failure_mode` cross-reference | Static cross-reference | **FAIL** | `w1-duplicate-detection-recording.json` never invoked Error Handling's `duplicate_application` rule at all — dead rule (D03-DEF-002); `error-handling.json` used a plain `update`, which finds no row for any failure occurring before Duplicate Detection & Recording ever writes one (D03-DEF-004) | Implementation Defect (both) | Added a parallel Error Handling call on the duplicate-found branch; changed the Sheets operation to `appendOrUpdate` keyed on `record_id` | **PASS** |
| T5 | Dedup key = engineering assumption | Static inspection | PASS | none | — | — | PASS |
| T6 | No fabricated criteria | Static inspection + `criteria-config-schema.md` row count | PASS — zero populated rows confirmed | none | — | — | PASS |
| T7 | Semantic scoring correctness | — | **BLOCKED** | Missing Client Requirement (evaluation criteria) | Missing Client Requirement | Not delegable — external | BLOCKED (unchanged) |
| T8 | Real resume-population coverage | — | **CONDITIONAL** | Missing Client Requirement (resume format mix) for the *scanned* share only; the defined text-PDF path (T2/T3 for this artifact) tested PASS | Missing Client Requirement (partial) | Not delegable for the blocked part | CONDITIONAL (unchanged) |
| T9 | Live runtime/integration | — | **BLOCKED** | Environment Blocker (no n8n instance/credentials this session) | Environment Blocker | Not delegable — external | BLOCKED (unchanged) |
| T10 | W2 never fires directly from W1 | Static — grep every W1 file's `connections` for a reference to the W2 workflow | PASS | none | — | — | PASS |
| T11 | `duplicate_application` rule reachable | Static cross-reference | **FAIL** (same root cause as T4) | D03-DEF-002 | Implementation Defect | Same fix as T4 | **PASS** |
| T12 | `record_id` available at earliest stage | Static trace | **FAIL** | Schema documented `record_id` as "generated at recording time" (the *last* step) — unusable by any earlier failure route (D03-DEF-005) | Implementation Defect | Generate `record_id` at intake (first step); thread through `w1-extraction.json`, `w1-ai-evaluation.json`, `w1-duplicate-detection-recording.json`, and both `error-handling.json` call sites; corrected the schema's own note | **PASS** |

## Summary

12 obligations. **8 executed and PASS** (5 after remediation: T1, T4, T11, T12 — T4/T11 share one remediation pass). **1 CONDITIONAL** (T8 — defined path tested, scanned-format coverage blocked). **2 BLOCKED**, both external (T7 evaluation criteria, T9 environment/credentials) and one of T8's two parts. **5 real Implementation Defects found and remediated** (D03-DEF-001 through 005) — all Project-local, reversible, within Blueprint scope, remediated without Founder approval per `remediation-governance.md`. **Zero defects required Founder or client judgment** — the only unresolved items are the three external inputs already tracked before this sprint began.
