---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: Implementation: Central Error Handling — Produced (definition-level)
Classification: Accepted
Artifact: implementation/workflows/error-handling.json
Date: 2026-08-03
---

**Reason:** Covers exactly the failure modes the Blueprint names in §7 (unreadable PDF, duplicate application, extraction failure) plus LLM call/parse failure — no additional failure modes invented, none of the named ones omitted. Wired as a shared sub-workflow, invoked from W1's own stages (see EV-003, EV-004, EV-005).

**Confidence:** High.
**Risk:** none beyond live-deployment credentials, tracked under EV-001.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D02, ready, n8n, workflow-definition, local-artifact, error-handling]
