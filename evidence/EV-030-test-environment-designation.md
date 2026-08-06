---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Founder-Designated Test Environment designated and discovered — Partially Discovered
Classification: Accepted
Artifact: deployment/test-environment-designation.md
Date: 2026-08-06
---

**Reason:** The Founder's local n8n instance designated as PRJ-0001's Test Environment (Environment Role TEST), scoped to this Project and revocable. Identity confirmed by direct live introspection, not inference: `localhost:5678/healthz` healthy; `/rest/settings` byte-identical (476 bytes) between the local address and the public hostname; and the local instance self-reports that hostname as its own OIDC base URL — one instance, two addresses, evidence-hierarchy level 1. Discovery result **PARTIALLY DISCOVERED**: reachability and deployment interface established; **n8n runtime version remains UNKNOWN** (exposed on no unauthenticated endpoint; the management API returned `version: "unknown"`) — the MCP package version 2.68.0 was explicitly not treated as the runtime version. Four credentials observed **by name only**; no value read, transmitted, or stored. Instance is internet-reachable, which is recorded as a blast-radius fact; no workflow was activated.

**Confidence:** High for identity, reachability, and interface; runtime version honestly UNKNOWN.
**Risk:** Two profile deviations found and surfaced rather than absorbed — see EV-032.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, test-environment, designation, partially-discovered, provenance:asdp-automated, env:founder-local-n8n]
