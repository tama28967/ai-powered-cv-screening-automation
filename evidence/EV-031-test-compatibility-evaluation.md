---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Target runtime compatibility evaluation — Compatible (node type/typeVersion scope)
Classification: Accepted
Artifact: deployment/test-compatibility-evaluation.md
Date: 2026-08-06
---

**Reason:** All 12 required node types accepted by the live instance at the typeVersions the candidate declares, established by live import — evidence-hierarchy **level 2**. Level 3 was attempted first and was insufficient: the provider's bundled metadata reported itself unpopulated for every node queried ("Callers must not infer upgrade safety from this response"), exactly the limitation ADR-0015 recorded. **This resolves the risk class that motivated ADR-0015** for this target. A transient probe carried the eight types not covered by the first import; its persisted structure was read back and verified intact (8 nodes, 7 connections, no coercion, no partial import) before deletion. Scope is deliberately narrow: acceptance and faithful persistence only — not execution correctness, not parameter validity (two defects found), not credential resolution, and **nothing about any Production environment** per the non-promotion rule.

**Confidence:** High, within the stated node-type/typeVersion scope.
**Risk:** none introduced; the probe was deleted and left no residue.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, compatibility, compatible, evidence-level-2, provenance:asdp-automated, env:founder-local-n8n]
