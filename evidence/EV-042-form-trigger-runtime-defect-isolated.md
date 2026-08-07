---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Form Trigger 404 root-caused to a provider-compatibility/runtime defect via controlled A/B probe — Deferred
Classification: Deferred
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** EV-041 recorded TC-01 as blocked with the 404 unexplained. This record formalizes the D08 round 5 controlled runtime diagnosis that isolated the cause, preserving provenance for a finding significant enough to change the Next Recommended Action.

**Method:** a controlled A/B probe, not inference. Two fresh, minimal, disposable workflows were created via the n8n API and activated the same way MAIN was activated:

1. `n8n-nodes-base.webhook` (control) — created, activated, tested on both `localhost:5678` and the public host. **Result: HTTP 200 immediately.**
2. `n8n-nodes-base.formTrigger` (isolation target) — created, activated, tested identically at **typeVersion 1** and, after an in-place bump, **typeVersion 2.2**. **Result: HTTP 404 ("Problem loading form") in every combination, on both hosts.**

Both probe workflows were deleted after the test. MAIN (`4nDrnkEQ9I9j8mjw`) was cycled deactivate→reactivate as part of diagnosis (one redeploy/retest cycle); this did not change the outcome and its topology/parameters were otherwise untouched.

**Runtime version, authoritative:** n8n **2.31.7**, obtained via `n8n_audit_instance`'s built-in report and independently confirmed by `n8n --version` executed directly inside the running container. The `n8n_health_check` tool's version field (`2.68.2`) is the `n8n-mcp` connector package version, not the server — a discovery-method distinction now recorded so it is not reused as fact.

**Ruled out by this evidence:** reverse proxy / DNS (localhost fails identically to the public host — same instance, confirmed later in round 6 by direct container inspection), activation lifecycle/staleness (a workflow with zero history fails identically to MAIN), node-shape/typeVersion (1 and 2.2 both fail), and MAIN's own edit history (an untouched fresh probe reproduces the same failure).

**Not established by this evidence:** *why* Form Trigger registration specifically fails while Webhook succeeds on the same instance and the same activation call — that requires either n8n-side source/issue evidence (round 6) or server log access (not available from this MCP boundary).

**Classification:** PROVIDER-COMPATIBILITY / RUNTIME DEFECT. Not a source defect — the MAIN workflow's Form Trigger node validates clean (`errorCount: 0`) and carries a correct `path`/`webhookId`/`active` state.

**Confidence:** High that the defect is Form-Trigger-specific and reproducible, established by direct A/B comparison rather than single-sample observation.
**Risk:** none introduced — both probe workflows were disposable and are deleted; no Production, Google, or Gemini resource touched.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, round-5, tc-01, form-trigger, runtime-defect, provider-compatibility, provenance:asdp-automated, env:founder-local-n8n]
