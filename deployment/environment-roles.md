# PRJ-0001 — Environment Roles and Blocker Separation

Records how repository [ADR-0016](../../../ASDP/project-management/adr/0016-founder-designated-test-environment.md)'s Environment Role model applies to this Project. **No environment has been designated, discovered, or mutated.** This is a governance record, not a discovery result.

## The blocker that was one question, and is now two

PRJ-0001 has tracked a single blocker since before D02 — Open Technical Question 3, "does the client already have an n8n instance, and where is it hosted." Under the Environment Role model that question resolves into two genuinely independent ones:

| Question | Environment Role | Owner | Status |
|---|---|---|---|
| Is a Founder-controlled n8n Test Environment available and designated? | `TEST` | Founder | **NOT YET DESIGNATED** — the Founder intends to provide a local instance; no designation, discovery, or connection attempt has occurred |
| Does the client have a production n8n environment, and where? | `PRODUCTION` | Client | **OPEN, UNCHANGED** — never answered; this is the original OTQ3 blocker, undiminished |

Resolving the first does **not** resolve the second. They were previously conflated because only one target concept existed.

## What a designated Test Environment would and would not change

**Would enable** — target discovery at evidence-hierarchy level 1 (live introspection, never previously reachable for this Project); real compatibility evaluation against the 12 node types and their typeVersions in [candidate-requirements.md](candidate-requirements.md); deployment to TEST; execution of T9 (live runtime/integration), currently BLOCKED on an Environment Blocker; runtime confirmation of the five remediated D03 defects; and the autonomous build → deploy → test → remediate → redeploy → retest loop.

**Would not change** — Production Target Discovery (still UNKNOWN), Production compatibility, Production Deployment Readiness (still BLOCKED), or any client-owned input. Per `test-environment-governance.md`'s non-promotion rule, evidence obtained against a TEST target satisfies no Production gate, and Environment Role is never mutated from TEST to PRODUCTION.

## Client blockers — preserved, unresolved

1. **Evaluation criteria** (OTQ1) — client business input. Unchanged.
2. **Resume format mix / OCR determination** (OTQ2) — client business input. Unchanged. A Founder could exercise the OCR *path* with sample inputs, but the actual format mix remains a client determination.
3. **Client Production Environment** (OTQ3, production half) — client input. Unchanged.
4. **Client acceptance** — a later lifecycle point, never inferred from ASDP automated verification or from Founder manual verification.

## Static-validation debt — not resolved by this record

The debt recorded as L-07 in [lessons-learned.md](../lessons/lessons-learned.md) — that this Project's entire validated state rests on static analysis — **remains open**. A governance change creates no evidence. It may be reduced only after real runtime testing actually executes against a designated Test Environment and produces evidence.

## Test data

Any runtime testing follows `test-data-governance.md`: synthetic inputs first, then Founder-created non-sensitive samples. Client-provided real candidate CVs are not used without explicit authorization for that use. Private content never enters this repository — `.gitignore` protection is in place and was verified with `git check-ignore`, not assumed.
