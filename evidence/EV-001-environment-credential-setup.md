---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: Implementation: Environment & credential setup — Blocked
Classification: Deferred
Artifact: none
Date: 2026-08-03
---

**Reason:** Requires Google Workspace API scopes, an LLM provider API key, and n8n instance access — none available in this execution session. Additionally depends on the Blueprint's own unresolved Open Technical Question 3 (does an n8n instance already exist, and where is it hosted). Per `credential-environment-governance.md`, this session never stores, transmits, or reasons about literal credential values, and provisioning is always a Founder/client action outside any ASDP artifact.

**Confidence:** High (blocker is unambiguous, not a judgment call).
**Risk:** none named beyond the blocker itself.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D02, blocked, credentials, n8n-instance]
