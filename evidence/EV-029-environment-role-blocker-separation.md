---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: Governance: Environment Role applied; OTQ3 blocker separated into TEST and PRODUCTION — Recorded
Classification: Deferred
Artifact: deployment/environment-roles.md, .gitignore
Date: 2026-08-06
---

**Reason:** Repository ADR-0016's Environment Role model applied to this Project as a governance record only. Open Technical Question 3 — previously one blocker — separates into Test Environment availability (`TEST`, Founder-owned, **NOT YET DESIGNATED**) and client Production Environment availability (`PRODUCTION`, client-owned, **OPEN and unchanged**). **No environment was designated, discovered, connected to, or mutated by this change**, and no client blocker was resolved: evaluation criteria, resume format mix/OCR, the client production environment, and client acceptance all remain exactly as before. The L-07 static-validation debt remains open — a governance change creates no evidence, and only executed runtime testing can reduce it. Repository ignore protection for private/PII test data was added and **verified with `git check-ignore` across eight paths, not assumed**; no already-tracked file is affected. Classified `Deferred` because this records an unresolved state rather than a resolved one.

**Confidence:** High (the separation is a direct consequence of the ratified model; the not-yet-designated state is directly verified).
**Risk:** none introduced — the governance change grants no autonomy over any environment until an explicit, evidenced designation occurs.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D07, environment-role, blocker-separation, not-yet-designated, provenance:asdp-automated]
