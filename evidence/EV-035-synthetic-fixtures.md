---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Synthetic CV fixtures generated (local-only) — Produced
Classification: Accepted
Artifact: local-test-data/synthetic/ (local-only, never committed)
Date: 2026-08-06
---

**Reason:** Three synthetic, text-extractable CV PDFs generated under `local-test-data/synthetic/`, each tied to an existing test obligation: **TC-01** a valid applicant (intake/extraction path), **TC-02** the same `applicant_email` as TC-01 (duplicate-detection path, T5), **TC-03** a differing-seniority applicant (scoring-variation path). All identities, email addresses (`@example.invalid`) and phone numbers are fictional; no real candidate data and no client-confidential content is present. Per ADR-0018 the Founder was not required to supply a fixture first. **Ignore protection was verified with `git check-ignore` BEFORE generation and re-verified after** — `git status` reports nothing, confirming none of the three is visible to Git. Evidence references test-case identifiers only; **no fixture content is reproduced in any artifact or record**. These are text-based PDFs — **no scanned-PDF or OCR coverage is claimed or inferable** from them (OTQ2 unchanged).

**Confidence:** High — generation, PDF validity, text extractability, and ignore behavior each directly verified.
**Risk:** none — fictional data, local-only, verified unversioned.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, synthetic-fixtures, local-only, pii-safe, provenance:asdp-automated]
