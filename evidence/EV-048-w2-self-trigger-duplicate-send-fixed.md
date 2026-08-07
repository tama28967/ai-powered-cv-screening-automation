---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: W2 self-trigger duplicate-notification defect found, stopped in-flight, root-caused, fixed, and verified idempotent — Ratified (TEST scope only)
Classification: Ratified
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** immediately after EV-047 recorded W2's first successful run, a scheduled follow-up check (part of the same continuation, not a separate ask) found a second execution had fired for the *same* record. This record captures the defect, the immediate containment action, the root cause, the fix, and its verification — honestly, including the actual duplicate side effect that occurred before containment.

**Defect: W2 self-triggers on its own write-back.** `notification_sent = true` is written by W2 itself to the same row its Sheets Trigger polls. Because the `Is Reviewed?` condition only checked `review_status == "reviewed"` (not whether `notification_sent` was already `"true"`), the next poll cycle detected W2's own write as a new "row updated" event, re-evaluated the same row, and re-sent the notification. Execution 82 (16:34:18Z) sent a **second** real Gmail message for record `REC-1786090017075-287` (TC-01) — Gmail's own API confirmed `SENT` (`id: 19fdd135117d0c50`) — nine seconds after the first, legitimate send in execution 81.

**Actual impact, stated plainly:** two TEST emails were sent for one record before this was caught, both to the Dedicated ASDP Test Account (`tama28967@gmail.com`) — never a real candidate address, and both bodies explicitly marked TEST ONLY. No real-world harm occurred, but this is recorded as an actual duplicate send, not merely a near-miss.

**Immediate containment:** W2 was deactivated the moment the second execution was noticed, before a third poll cycle could fire again — confirmed by execution list showing no execution beyond id 82 at the time of deactivation.

**Root cause and fix:** `Is Reviewed?`'s condition was missing the idempotency check standard for polling-trigger-driven side effects. Added a second condition (AND-combined): `notification_sent != "true"`. Validated clean, redeployed, reactivated.

**Verification, not assumption:**
1. Waited a full poll cycle after the fix — **no spurious execution** occurred for the already-notified TC-01 record (confirmed via execution list: still only 81, 82).
2. Tested the fix against a genuinely new event: set TC-03's (`REC-1786090459234-741`, Jordan Blake) `review_status` to `"reviewed"` — execution 84 fired correctly, sent exactly one notification (Gmail confirmed `SENT`, `id: 19fdd19725341b52`), and wrote `notification_sent = true`.
3. **Direct proof the fix actually prevents the self-trigger loop**, not just that it didn't happen to recur: execution 85 fired one poll cycle later (W2's own write-back to TC-03's row, the exact same self-trigger condition that caused the original defect) — `Is Reviewed?` correctly routed to its **false** branch this time (`notification_sent` already `"true"`), stopping at 2 nodes with no Gmail send. The self-trigger still occurs (inherent to a Sheets-poll trigger watching a sheet the workflow itself writes to) but is now a correct, cheap no-op instead of a duplicate send.

**Confidence:** High — the fix was verified against the exact failure mode that caused the original defect (a self-triggered re-poll), not merely against a fresh record.
**Risk:** Low going forward — the defect is closed and proven idempotent; the two duplicate TEST emails already sent carry no real-world consequence (TEST account, disclosed here rather than hidden).
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08-continuation, w2, gmail, idempotency, self-trigger, duplicate-send, defect-found-and-fixed, provenance:asdp-automated, env:founder-local-n8n]
