---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Two runtime defects found; runtime testing blocked on two profile deviations — Deferred
Classification: Deferred
Artifact: deployment/test-compatibility-evaluation.md, implementation/workflows/
Date: 2026-08-06
---

**Reason:** Live deployment surfaced two Implementation Defects that **static testing could not have caught** — every affected artifact passed D03's JSON and topology tests. **D08-DEF-001:** no node in any of the 6 artifacts carried the `id` field the deployment interface requires; all 6 were rejected outright. Remediated across all artifacts (35 nodes, verified `missing_id=0`) and redeployment then succeeded. **D08-DEF-002:** runtime validation rejected the `googleSheets` parameter shape ("Expected object but got string"); recorded and deliberately left unremediated, because verifying the correct shape requires a target spreadsheet that cannot be created while the integration-account question is open — a half-fix that cannot be executed is not a fix. Runtime execution testing is **BLOCKED** on two deviations from the ratified Default Test Environment Profile: (1) the configured Google credentials belong to the Founder's own account rather than a dedicated test account, so autonomy does not reach them and the irreversible Gmail send path is not authorized; (2) no Gemini or other LLM credential exists, so the AI evaluation path cannot execute. No billing was enabled and nothing was purchased. One workflow is deployed and inactive (`yrIl76JCaaQloshO`).

**Confidence:** High — both defects are direct observations from the live environment; both blockers are directly verified absences.
**Risk:** No Google resource created, no email sent, no LLM call made, no workflow activated, no Production environment touched.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, runtime-defect, remediated, blocked, provenance:asdp-automated, env:founder-local-n8n]
