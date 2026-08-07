---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: First real W2 execution — Human Review Gate + Gmail applicant notification PASS, with external Gmail/Sheets read-back — Ratified (TEST scope only)
Classification: Ratified
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** D08 round 8 deliberately left W2 (an independent Sheets-poll trigger) and Gmail notification unexercised, as a separate risk decision from that round's Form Trigger scope. This session resumed with a short-prompt objective ("complete automated TEST verification for W2 and Gmail") reconstructed autonomously from `asdp-state.md` per the Proportional Assurance Model and Architecture-Review-Gated Auto-Chaining ratified earlier the same day — no long prescriptive brief was supplied.

**Defect found and fixed before W2 could activate:** `googleSheetsTrigger` node "Review Status Changed to Reviewed" had `sheetName.mode: "list"` with a plain sheet **name** as its value ("Applicant Records") instead of a numeric sheet ID — activation failed with `"Sheet with ID Applicant Records not found"`. This is a recurrence of the D08 round 4 DEF-008 class (the Sheets **Trigger** node variant rejects `mode: "name"`, unlike the regular Sheets node, which accepts it) — round 4's remediation evidently did not reach this specific node because W2 was never activated/tested until now. Fixed by looking up the tab's actual `sheetId` (gid `1629274334`) via a direct Sheets API call and setting `mode: "id"` with that value — the correct, runtime-validated shape for this node type.

**Execution proof (record `REC-1786090017075-287`, TC-01's Priya Raman row):** `review_status` set to `"reviewed"` via a bounded TEST-only Sheets write. W2's polling Sheets Trigger picked up the change on its next poll cycle. Full execution graph inspected — all 4 nodes ran successfully: `Review Status Changed to Reviewed` → `Is Reviewed?` (routed true) → `Send Candidate Notification` → `Set notification_sent = true`.

**External state independently confirmed, not merely claimed:**
- **Gmail** — the `Send Candidate Notification` node's own output is Gmail's own API response: message `id: 19fdd126d23066bf`, `labelIds: ["UNREAD","SENT","INBOX"]`. The `SENT` label is Gmail's own confirmation the message was actually transmitted, not merely queued or attempted.
- **Google Sheets** — `Set notification_sent = true` node's output confirms the write: `{record_id: "REC-1786090017075-287", notification_sent: "true"}`.

**TEST boundary preserved:** the Gmail node's `sendTo` is hardcoded to `tama28967@gmail.com` — the same Dedicated ASDP Test Account used for Drive/Sheets/Gemini throughout D08 (confirmed by Drive file ownership metadata in round 8's own read-back) — never a real candidate address, matching ADR-0018 §2's bounded-TEST-recipient rule. The message body is explicit: *"TEST ONLY — synthetic fixture notification... never to a real candidate address."*

**Diagnostic probes used and removed:** one probe to look up the Sheets tab's `sheetId` via the Sheets API (`ZZ-DIAG-sheet-gid-lookup`), one probe to set `review_status` on the target row (`ZZ-DIAG-mark-reviewed`) — both deleted immediately after use, per Tier 1 discipline.

**W2 left active** (previously inactive pending this proof) — it is now a proven, working component of the pipeline like the other five workflows, not a diagnostic artifact.

**Confidence:** High — Gmail's own `SENT` label and Sheets' own write confirmation are both direct API responses captured in n8n's execution data, not inferred from top-level workflow success.
**Risk:** Low — TEST-only recipient (Founder's own designated test account), one synthetic record reused, fully reversible Sheets state change, no real candidate contacted.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08-continuation, w2, gmail, human-review-gate, e2e-pass, defect-recurrence, DEF-008-class, provenance:asdp-automated, env:founder-local-n8n]
