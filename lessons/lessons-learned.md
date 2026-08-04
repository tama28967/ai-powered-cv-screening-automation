# PRJ-0001 — Lessons Learned (INTERIM)

**Status: INTERIM.** This is not engagement closure. PRJ-0001 has not deployed, has not been accepted by a client, and has not operated in production. It is classified INTERIM per `lesson-model.md`; a later final retrospective extends this record and never rewrites the evidence beneath it.

Produced by `/learn`, per `learning-standard.md` stages 5–10. Every Fact below cites the Evidence Record or canonical artifact it rests on. Nothing here is drawn from conversation memory or from recollection of prior sprints.

---

## 1. Evidence base consumed

**Fact.** 24 Evidence Records exist at `evidence/EV-001` through `EV-024`, spanning four lifecycle stages: Build (EV-001–008, D02), Testing & Validation (EV-009–017, D03), Deployment (EV-018–021, D04), Delivery (EV-022–024, D05).

**Fact.** Classification distribution across the 24: **19 Accepted, 5 Deferred, 0 Rejected, 0 Escalated** (verified by direct count over the record headers).

**Fact.** Supporting canonical artifacts read: `engineering-blueprint.md`, `briefcase.md`, `implementation/test-plan.md`, `implementation/test-results.md`, `implementation/validation-report.md`, `implementation/work-ledger.md`, the six workflow definitions and two schemas, the six `deployment/` records, and the seven `delivery/` documents.

### Evidence that does not exist

Stated as Facts about absence, per `observation-model.md` — not softened, not omitted:

- **No runtime execution evidence.** T9 (live runtime/integration) is BLOCKED throughout (`test-results.md`; EV-001).
- **No deployment evidence.** Deployment was never performed; Deployment Readiness is BLOCKED (EV-020).
- **No target-environment evidence.** Target Discovery is UNKNOWN (EV-018); compatibility UNKNOWN (EV-019).
- **No post-deployment verification, client acceptance, production operation, user adoption, or business outcome evidence.** No record of any kind exists for these.

**Consequently, this record states no lesson about production reliability, client acceptance, business outcomes, operational performance, production incidents, or user adoption.** Those lessons are unavailable, not omitted for brevity.

---

## 2. Pattern detection — what the record counts actually mean

**Fact.** Five distinct Implementation Defects were found and remediated in D03: D03-DEF-001 through D03-DEF-005 (`test-results.md`; EV-010, EV-011, EV-012, EV-013, EV-014, EV-016).

**Fact.** D03-DEF-005 (`record_id` unavailable at the earliest pipeline stage) manifested across five artifacts — `sheets-record-schema.md`, `w1-intake-validation-storage.json`, `w1-extraction.json`, `w1-ai-evaluation.json`, `w1-duplicate-detection-recording.json` — and both `error-handling.json` call sites (`test-results.md` T12).

**Fact.** D03-DEF-002 appears in two Evidence Records (EV-014, EV-016) and under two test obligations (T4, T11), which `test-results.md` itself annotates as "same root cause as T4."

Applying the causal-chain rule (`evidence-aggregation-model.md`): the evidence base contains **five independent root causes**, not the fifteen-plus record-level mentions of them, and not three-per-defect for the failure → remediation → retest chain each one went through. Manifestation counts are recorded above because pervasiveness is real information — but pervasiveness of one cause is not corroboration by many.

---

## 3. Lessons

### L-01 — Every defect in this engagement was a cross-artifact contract defect; none was internal to a single artifact

**Supporting facts.** All five root causes are failures of agreement *between* artifacts: DEF-001, fields the schema requires and the producer supplied were silently dropped by the consuming Write nodes (EV-010, EV-014); DEF-002, a rule `error-handling.json` declared was never invoked by any caller (EV-014, EV-016); DEF-003, `evaluation_criteria_version` not propagated to the output record (EV-013); DEF-004, `error-handling.json`'s Sheets `update` could not succeed for the three failure modes that fire before any row exists (EV-016); DEF-005, `record_id` lifecycle spanning six artifacts (EV-010–014). Meanwhile every artifact was individually sound: T2 (JSON well-formedness) PASS on all six workflow files on first execution, and T3 (topology matches Blueprint prose) PASS on first pass (`test-results.md`). `w2-human-review-notification.json` — the one artifact with no inbound field contract from W1, triggering instead off a Google Sheets row change — carried zero defects (EV-015).

**Observation.** For this multi-artifact generated integration, per-artifact correctness did not predict system correctness. The defects lived in the seams, and the one artifact without a seam had none.

**Counter-evidence, recorded.** Two facts genuinely bound this lesson. First, **all detection was static** — no runtime obligation ever executed (T9 BLOCKED), so "zero intra-artifact defects" partly describes what static analysis *can see*, not only what exists; an intra-artifact defect that is syntactically valid and semantically wrong would not have been caught. Second, the sample is eight artifacts, one technology, one Project, produced in a single generation pass by one capability — the pattern may belong to that generation mode rather than to generated integration artifacts in general.

**Evidence strength: Moderate.** Five genuinely independent root causes, cross-stage (originating in D02, surfaced in D03), consistent, with a supporting negative instance — but one engagement, one technology, and bounded by the static-only limitation above.

**Scope:** Delivery-process. **Applicability:** multi-artifact integration work where artifacts exchange records by field contract. **Limitations:** unproven for single-artifact work, for other technologies, and for defects only a runtime test could expose.

**Recommended action:** consider making cross-artifact contract conformance an explicit obligation class in `test-obligation-model.md`, rather than something reached indirectly through per-artifact obligations. **Action classification: Process Improvement Recommendation** (see REC-01). **Founder judgment required:** no. **Further evidence required:** yes — a second, independent engagement, per P-004.

### L-02 — Static verification produced real defect yield with no environment access whatsoever

**Supporting facts.** Ten of twelve obligations were READY and executed by static methods — `JSON.parse` execution, static cross-reference, static trace, grep over `connections` (`test-plan.md`, `test-results.md`). Five real Implementation Defects were found and remediated. Zero credentials were used, and no external system was touched (EV-001, EV-018).

**Observation.** Meaningful defect detection occurred entirely without the target environment — the blocked environment delayed proof of runtime behavior, but did not prevent finding real defects.

**Counter-evidence, recorded.** The residual is unmeasured: because no runtime obligation ever ran, there is no evidence about what static verification *missed*. This lesson therefore claims yield, and explicitly does **not** claim sufficiency.

**Evidence strength: Moderate** for the yield claim; **Insufficient** for any sufficiency claim, which is accordingly not made.

**Scope:** Delivery-process. **Recommended action:** none — `/test`'s existing testability classification already derives and executes static obligations without treating a blocked environment as a reason to skip them. **Action classification: Retain.** **Founder judgment required:** no.

### L-03 — "Unknown is never converted to pass or fail" held at every stage, including where a shortcut was available

**Supporting facts.** Three external inputs were named before D02 began and remained exactly three, unresolved and un-expanded, through D05. At each stage the honest-but-inconvenient path was taken: EV-005 produced AI-evaluation *structure* while criteria content stayed BLOCKED, fabricating no criteria (T6 confirmed zero populated rows); EV-012 recorded T8 as CONDITIONAL rather than FAIL for a genuinely untested share; EV-017 classified validation `Deferred` rather than `Rejected`, reasoning explicitly that unresolved is not failed; EV-018 declined to treat an n8n MCP connection that was actually available in the working environment as PRJ-0001's target; EV-020 left every unknown Deployment Plan field named rather than guessed; EV-024 disclosed all three blockers in the client-facing package.

**Observation.** The rule survived contact with a real, available shortcut. EV-018 is the strongest instance: a usable connection existed, and using it would have produced a cleaner-looking result.

**Evidence strength: Strong.** Direct, six independent observations, spanning four lifecycle stages and four different capabilities, with no counter-evidence found.

**Scope:** Capability. **Recommended action:** none — the behavior is the contract working as designed (`validation-model.md`, `target-discovery.md`). **Action classification: Retain.** **Founder judgment required:** no.

### L-04 — Every defect that occurred was delegable; four of six defect categories never occurred at all

**Supporting facts.** All five defects classified Implementation Defect, remediated without Founder approval (`test-results.md`: "Zero defects required Founder or client judgment"). Zero Evidence Records carry the `Escalated` classification. The unresolved items were Missing Client Requirement (T7, T8) and Environment Blocker (T9) — external by definition, correctly never escalated as engineering decisions.

**Observation.** `defect-classification.md`'s authority split routed every actual occurrence correctly. But the defect population was homogeneous: **Blueprint Issue, Test Defect, and Capability Failure never occurred**, so three of the six categories — including the one that routes to CEF — have never been exercised in practice.

**Evidence strength: Moderate** for the routing that was exercised; **Insufficient** for the three categories that were not.

**Scope:** Capability. **Recommended action:** retain the model; do not treat PRJ-0001 as validating the unexercised branches. **Action classification: Retain + Future Investigation** (see REC-03). **Further evidence required:** yes.

### L-05 — The Evidence Record classification vocabulary carries two distinct meanings under one value

**Supporting facts.** All five `Deferred` records mean *unresolved or blocked by an external condition*: EV-001 (credentials unavailable), EV-017 (partially validated), EV-018 (target unknown), EV-019 (compatibility unknown), EV-020 (readiness blocked). **None** means *a decision deliberately postponed*. EV-017 documents the choice explicitly — `Deferred` was selected over `Rejected` because "nothing here demonstrates non-conformance; the open items are unresolved, not failed."

**Observation.** The four-value vocabulary (`Accepted`/`Rejected`/`Deferred`/`Escalated`) has no value meaning "blocked on an external input," so `Deferred` absorbs it. A reader must consult each record's Reason field to learn which of the two meanings applies — which works, but places the distinction outside the structured field that exists to carry it.

**Counter-evidence, recorded.** The schema is owned by the Consulting Framework's Evidence Architecture (CF ADR-0003) and serves both frameworks; CF's own business-decision usage of `Deferred` was not examined here and may not show the same strain. A vocabulary change would affect a platform contract on the basis of one framework's usage in one engagement.

**Evidence strength: Moderate** for the observation (five consistent uses, explicitly reasoned in one); **Weak** as a basis for any specific schema change. Per the ceiling rule, the lower governs the proposed action.

**Scope:** Architectural candidate. **Recommended action:** surface the question; propose no change. **Action classification: ADR Candidate** (see REC-02) — a question framed for the Founder, authorizing nothing. **Founder judgment required:** yes, if pursued. **Further evidence required:** yes — a second engagement, plus examination of CF's own usage.

### L-06 — Proving a platform mechanism required a fixture, and keeping that proof in its own record stopped it reading as client proof

**Supporting facts.** EV-021 records a read-only simulation of `/deploy`'s candidate-requirements derivation against the n8n-mcp package's bundled reference data (evidence-hierarchy level 3), and states in its own text that it "explicitly does not change, and is not cited as changing, PRJ-0001's actual Deployment Readiness." EV-020 (BLOCKED) is unaffected and remains the authority on that question.

**Observation.** With no real target available, the only way to demonstrate the mechanism worked was against a fixture — and a fixture result and a client result are the kind of thing that blur together later, in a summary, when the distinction has stopped being obvious. Separate records kept them separable.

**Evidence strength: Weak** — a single occurrence, and simulation evidence is capped below Strong by construction (`evidence-strength-model.md`).

**Scope:** Delivery-process. **Recommended action:** record as an observed pattern only; propose no standard. Per P-001, a pattern seen once is a pattern, not a standard. **Action classification: Retain.**

### L-07 — PRJ-0001's entire validated state rests on static analysis (technical debt)

**Supporting facts.** T9 BLOCKED (`test-results.md`); every executed obligation used a static method; no deployment occurred (EV-020); no post-deployment verification exists.

**Observation.** The Project is PARTIALLY VALIDATED (EV-017) on a foundation with no runtime component at all. This is correctly reported everywhere it appears — the documentation does not overstate it (EV-022, EV-024) — but it is real carried debt, not merely a reporting nuance.

**Evidence strength: Strong** — a directly verified fact about absence.

**Scope:** Project-specific. **Recommended action:** when a target environment becomes available, execute T9 and re-run the CONDITIONAL and BLOCKED obligations before treating any runtime behavior as proven. **Action classification: Technical Debt / Project-Specific Follow-Up.**

### L-08 — Cross-project evidence aggregation has no source yet

**Supporting facts.** `project-management/evidence/records/` does not exist in the ASDP repository (directly verified). `evidence-repository-specification.md` defines it as the canonical platform store, "created at the first Evidence Record, not pre-seeded." All 24 records live only in this Project's repository.

**Observation.** This is not a defect — the specification deliberately says the store is not pre-seeded, and no platform-level record has ever been written. But it means `/learn` can currently aggregate within a Project and not across Projects, and that the platform half of the lesson-reuse mechanism rests on the `engineering-knowledge/` registries rather than on an evidence store.

**Evidence strength: Strong** — a directly verified fact.

**Scope:** Capability. **Recommended action:** at the second Project, decide whether per-Project records propagate to the canonical store. Relates to ADR-0010's own recorded residual. **Action classification: Future Investigation.** **Further evidence required:** yes — a second Project.

---

## 4. Open client blockers — preserved, unchanged

These are recorded here as delivery evidence, not resolved, and not interpreted as client behavior or fault. Missing input remains missing input.

| Blocker | State | First recorded | Still open at |
|---|---|---|---|
| Evaluation criteria (Open Technical Question 1) | Unresolved | Blueprint, pre-D02 | EV-024 (D05) |
| Resume format mix / OCR determination (Open Technical Question 2) | Unresolved | Blueprint, pre-D02 | EV-024 (D05) |
| n8n instance existence & hosting (Open Technical Question 3) | Unresolved | Blueprint, pre-D02 | EV-024 (D05) |

**Fact.** All three are disclosed to the client in `delivery/05-known-limitations-and-client-actions.md` (EV-022, EV-024). Their persistence across four sprints is analyzed above only as evidence about *how ASDP behaved while blocked* (L-03) — never as evidence about the client.

---

## 5. Current state at the time of this record

Every value below is taken from the Project's own artifacts, not restated as a new source of truth:

| Dimension | State | Source |
|---|---|---|
| Implementation | 8 artifacts, local, undeployed | EV-002–008 |
| Testing | 8 PASS (5 after remediation), 1 CONDITIONAL, 2 BLOCKED | `test-results.md` |
| Validation | PARTIALLY VALIDATED | EV-017 |
| Target compatibility | UNKNOWN | EV-019 |
| Deployment readiness | BLOCKED | EV-020 |
| Deployment | NOT PERFORMED | EV-020 |
| Delivery Package | PRE-DEPLOYMENT | EV-023 |
| Delivery Package Readiness | READY FOR PRE-DEPLOYMENT HANDOFF | EV-024 |

Nothing in this record changes any of them.
