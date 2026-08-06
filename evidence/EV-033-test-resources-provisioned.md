---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: Google Workspace TEST resources provisioned within ASDP/PRJ-0001/ — Produced
Classification: Accepted
Artifact: deployment/test-runtime-results.md
Date: 2026-08-06
---

**Reason:** The Founder-designated Dedicated ASDP Test Account's Project boundary `ASDP/PRJ-0001/` was created by **real n8n workflow execution** (Drive folder id `1G2RiX-oCJqpFPs7lyjCLNgZM5F1EjrgD`), and the Project TEST spreadsheet `PRJ-0001 Test - Applicant Records` (id `1eiVPbSbJrZSiNa3FaKXgwQCJtxuew5zDeW7Qujgdd_8`) was created and relocated into that boundary. Google Drive folder-create, Drive file-move, and Sheets spreadsheet-create are therefore proven at runtime against the real target. Resources are Project-namespaced per ADR-0018; no resource outside `ASDP/PRJ-0001/` was created or mutated; no other Project's boundary was touched. Credentials were referenced by id/name only — no value read, transmitted, or stored. One transient provisioning helper workflow was created for the run and deleted afterward, leaving no residue.

**Confidence:** High — each resource identity is a direct API response, not an inference.
**Risk:** none beyond the recorded junk row from EV-034's defect; all resources are disposable TEST resources inside the Project boundary.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, provisioning, google-workspace, drive, sheets, provenance:asdp-automated, env:founder-local-n8n]
