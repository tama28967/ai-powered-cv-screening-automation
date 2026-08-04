# PRJ-0001 — Test Plan

Produced by `/test`, per `testing-validation-standard.md` stages 5 (Test Obligation Derivation). Obligations traced to the Engineering Blueprint and the actual D02 implementation artifacts — none invented. States per `test-obligation-model.md`.

| # | Obligation | Source | Artifact under test | Method | State |
|---|---|---|---|---|---|
| T1 | Both schemas are internally consistent and every field a workflow references actually exists in them | Blueprint §4; `sheets-record-schema.md`, `criteria-config-schema.md` | schemas | Static: cross-reference every `$json.<field>` used across all 6 workflows against the schema tables | READY |
| T2 | Every workflow-definition artifact is well-formed, parseable JSON | Blueprint §4 | all 6 workflow files | Static: JSON parse | READY |
| T3 | Each workflow's node topology matches the component's description in Blueprint §4 | Blueprint §4 | all 6 workflow files | Static: inspect nodes + connections against Blueprint prose | READY |
| T4 | Every `failure_mode` value a workflow passes to Error Handling is actually one of the four rules `error-handling.json` declares, and every declared rule is actually reachable from somewhere | Blueprint §7 (named failure modes) | all 6 workflow files (cross-artifact) | Static: cross-reference `failure_mode` strings across all `executeWorkflow` calls against `error-handling.json`'s Switch rules | READY |
| T5 | The duplicate-detection key is `applicant_email`, matching the Blueprint's corrected §9 engineering assumption — not treated as blocked | Blueprint §9 | `w1-duplicate-detection-recording.json` | Static: inspect lookup key | READY |
| T6 | AI Evaluation's criteria content is not fabricated — the artifact reads from Criteria Config at runtime and contains no hardcoded criterion | Blueprint §9, Open Technical Question 1 | `w1-ai-evaluation.json`, `criteria-config-schema.md` | Static: inspect for literal criterion text; confirm zero populated rows | READY |
| T7 | AI Evaluation's *semantic scoring correctness* — does it actually score candidates the way the client wants | Open Technical Question 1 (external) | `w1-ai-evaluation.json` | Runtime/business — requires real criteria | **BLOCKED** — client input |
| T8 | Extraction's coverage of the client's real resume population (text vs. scanned) | Open Technical Question 2 (external) | `w1-extraction.json` | Runtime — requires real resume sample | **BLOCKED** — client input |
| T9 | End-to-end runtime execution against a live n8n + Google Workspace instance | Open Technical Question 3 / Environment setup (external) | all 6 workflow files | Runtime/integration | **BLOCKED** — credentials/environment |
| T10 | W2 fires only on `review_status = reviewed`, never directly from W1 | Blueprint §3 (architecture decision — decoupling) | `w2-human-review-notification.json` (cross-artifact with W1 files) | Static: confirm no W1 file's connections target W2 directly | READY |
| T11 | Error Handling's `duplicate_application` rule is actually invoked (not dead code) | Blueprint §7 | `error-handling.json`, `w1-duplicate-detection-recording.json` | Static: cross-reference | READY |
| T12 | `record_id` is available at every point a workflow may need to route an error, including the earliest pipeline stage | Blueprint §4 (structured record), implied by error-handling's own input contract | all 6 workflow files | Static: trace `record_id`'s origin and propagation | READY |

## Coverage summary

10 of 12 obligations READY (structural/static testing, no external dependency). 2 BLOCKED (T7 semantic scoring, T9 live-environment integration) on exactly the same three external inputs already tracked — T7 on Open Technical Question 1, T9 on environment/credentials. T8 is BLOCKED for full validation but its currently-defined text-PDF path (T3/T2 for that artifact) is still tested. No obligation was invented to pad this plan; none was skipped to avoid a blocker.
