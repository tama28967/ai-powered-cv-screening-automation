# PRJ-0001 — Improvement Recommendations (from INTERIM Lessons Learned)

Produced by `/learn`, per `learning-standard.md` stage 10. Each recommendation traces to a Lesson in [lessons-learned.md](lessons-learned.md), and through it to named Evidence Records.

**None of these is adopted.** Per `self-modification-boundary.md`, `/learn` surfaces recommendations and stops. No canonical contract, principle, standard, or accepted decision was modified in producing them.

---

## REC-01 — Make cross-artifact contract conformance an explicit test-obligation class

| | |
|---|---|
| **Evidence** | D03-DEF-001 through D03-DEF-005; EV-010, EV-011, EV-012, EV-013, EV-014, EV-016; `test-results.md` T1/T4/T11/T12. Counter-instance: EV-015 (the one artifact with no inbound field contract had zero defects). |
| **Lesson** | L-01 — all five root causes were cross-artifact contract defects; none was internal to a single artifact. |
| **Proposed improvement** | In `test-obligation-model.md`, derive cross-artifact contract conformance as an obligation class in its own right — every field a producer emits and a consumer must carry, every declared handler actually invoked by some caller, every identifier available at the earliest stage that references it — rather than reaching those checks indirectly through per-artifact obligations. |
| **Expected benefit** | The defect class that accounted for 100% of this engagement's defects would be derived by construction rather than found by a cross-reference obligation that happened to be written broadly enough. |
| **Risk** | Low for correctness; real for proportionality. On a single-artifact or loosely-coupled Project this adds derivation work with no yield. If specified too rigidly it could push `/test` toward technology-specific knowledge, which ADR-0014 places outside DF. |
| **Scope** | Delivery Framework contract refinement (`test-obligation-model.md`). Not repository-level. |
| **Confidence** | Moderate. Five independent root causes, cross-stage, with a supporting negative instance — but one engagement, one technology, one generation pass, and detection was static-only, so the "zero intra-artifact defects" half of the evidence is bounded by what static analysis can see. |
| **Founder judgment required** | No. |
| **Requires an ADR** | No — a DF Knowledge refinement within ADR-0014's existing charter, of the same kind D03 made when it added `executable-work-model.md`'s vocabulary. |
| **Gather more evidence first** | **Yes.** Per P-004, one engagement is an anecdote. Reopen at a second, independent Project involving multiple interacting artifacts. |

---

## REC-02 — Examine whether the Evidence classification vocabulary needs a "blocked" distinction

| | |
|---|---|
| **Evidence** | EV-001, EV-017, EV-018, EV-019, EV-020 — all five `Deferred` records mean *unresolved/blocked*; none means *decision postponed*. EV-017 reasons the choice explicitly. |
| **Lesson** | L-05 — `Deferred` carries two distinct meanings, with the distinction living in the free-text Reason field rather than the structured classification. |
| **Proposed improvement** | **None proposed.** The recommendation is to *frame the question*, not to answer it: does the platform Evidence classification need to distinguish "blocked by an external condition" from "decision deliberately postponed," or is Reason-field disambiguation adequate and cheaper? |
| **Expected benefit** | If pursued: blocked-state evidence becomes queryable by classification rather than by prose, which matters more as evidence accumulates across Projects. |
| **Risk** | Meaningful. The schema is owned by the Consulting Framework's Evidence Architecture (CF ADR-0003) and serves both frameworks; ADR-0010 adopts it explicitly *by reference and never forks it*. A change would touch a platform contract, and would need CF's own business-decision usage examined first — which this engagement did not do. |
| **Scope** | Architectural candidate — platform Evidence contract. |
| **Confidence** | Moderate that the strain is real; Weak as a basis for any specific change. Per the ceiling rule the lower governs: this reaches ADR Candidate (a surfaced question) and no further. |
| **Founder judgment required** | **Yes**, if pursued. |
| **Requires an ADR** | Yes, if pursued — it would modify a platform-level contract. |
| **Gather more evidence first** | **Yes.** A second engagement, plus a look at how CF's six business decisions actually use `Deferred`. |

---

## REC-03 — Do not treat `defect-classification.md` as validated by this engagement

| | |
|---|---|
| **Evidence** | All five defects classified Implementation Defect; zero `Escalated` records across all 24; `test-results.md` — "Zero defects required Founder or client judgment." |
| **Lesson** | L-04 — the exercised routing was correct, but Blueprint Issue, Test Defect, and Capability Failure never occurred. |
| **Proposed improvement** | No change to the model. The recommendation is a caution: three of six categories — including the one routing to CEF's architecture governance — are unexercised, and the engagement's clean routing record should not be read as evidence that they work. |
| **Expected benefit** | Prevents a false confidence signal from propagating into a future decision to simplify or rely on the model. |
| **Risk** | None from adopting the caution. |
| **Scope** | Capability. |
| **Confidence** | Moderate for what was exercised; Insufficient for what was not — which is precisely the point. |
| **Founder judgment required** | No. |
| **Requires an ADR** | No. |
| **Gather more evidence first** | Yes, inherently — this recommendation *is* a request for evidence. |

---

## What is deliberately not recommended

No recommendation is made about deployment behavior, target-runtime compatibility in practice, runtime correctness, client acceptance, operational performance, or business outcomes. PRJ-0001 has produced no evidence in any of those areas (see `lessons-learned.md` §1), and a recommendation without evidence is a guess wearing a table.
