---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Deployment binding layer established, separating runtime identity from implementation — Produced
Classification: Accepted
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-06
---

**Reason:** Assembly required values that are **not implementation facts** — n8n workflow IDs, credential IDs, Google resource IDs, model bindings. Writing them into provider-neutral source would bind the implementation to one environment's identity. A minimal binding layer was established instead: source carries `<BIND:token>` placeholders **inside the correct runtime shape** (so the shape stays verifiable), and `deployment/test-runtime-bindings.md` maps token → actual TEST value. No new configuration platform or abstraction was invented — one markdown mapping file, consistent with existing deployment artifacts.

One deliberate exception is recorded rather than hidden: `googleSheets.documentId` still carries the TEST spreadsheet ID directly (from D08 round 2) because the node requires a concrete resourceLocator value to validate; each such node carries an explicit `TEST BINDING` note, and migrating it to a token is named as follow-up. Credentials appear **by id/name only** — no value in this file, any artifact, or any Evidence Record.

**Confidence:** High for the separation as implemented.
**Risk:** The binding resolution path is **not yet exercised** — no workflow has been deployed through it, so it is structurally sound but not runtime-proven.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, deployment-binding, source-vs-runtime, no-secrets, provenance:asdp-automated]
