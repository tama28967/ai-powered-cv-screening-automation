# PRJ-0001 — TEST Runtime Results (D08 continuation)

First **real runtime execution** in this Project's history. Target: the Founder-Designated Test Environment ([designation](test-environment-designation.md)). All results below are direct observations from actual executions, not inference.

## Authorized TEST boundary (now complete)

| Resource | Identity | Status |
|---|---|---|
| n8n runtime | Founder local n8n (`localhost:5678` / `n8n.nugi.my.id`) | Reachable |
| Google account | Dedicated ASDP Test Account (ADR-0018) | Designated |
| Drive boundary | `ASDP/PRJ-0001/` — folder id `1G2RiX-oCJqpFPs7lyjCLNgZM5F1EjrgD` | **Created** |
| Applicant records Sheet | `PRJ-0001 Test - Applicant Records` — id `1eiVPbSbJrZSiNa3FaKXgwQCJtxuew5zDeW7Qujgdd_8` | **Created**, moved into the boundary |
| Gemini credential | `googlePalmApi` (by reference only) | Available |
| Local fixtures | `local-test-data/synthetic/` — 3 CVs | **Generated**, git-ignored (verified) |

## Runtime executions performed

| # | Path exercised | Method | Result |
|---|---|---|---|
| R1 | **Gemini LLM call** | Real webhook-triggered execution, `googleGemini` v1.2, model `models/gemini-2.5-flash` | **PASS** — returned exactly `PRJ0001-TEST-OK`, `finishReason: STOP`, 3.15s |
| R2 | **Google Drive folder creation** | Real execution, `googleDrive` v3, `folder:create` ×2 | **PASS** — `ASDP/` then `ASDP/PRJ-0001/` created |
| R3 | **Google Sheets spreadsheet creation** | Real execution, `googleSheets` v4, `spreadsheet:create` | **PASS** — spreadsheet created |
| R4 | **Google Drive file move** | Real execution, `googleDrive` v3, `file:move` | **PASS** — spreadsheet relocated into `ASDP/PRJ-0001/` |
| R5 | **Google Sheets header write** | Real execution, `googleSheets` v4, `append` with `dataMode: raw` | **FAIL** — see D08-DEF-003 |
| R6 | **Google Sheets read-back verification** | Real execution, `googleSheets` v4, `read` | **PASS as a check** — and it is what caught R5 |

**Actual Gemini model used: `models/gemini-2.5-flash`** — the preferred default, available and usable on the configured free tier. No paid tier was reached; no billing was enabled.

## D08-DEF-003 — Sheets `append` raw-mode parameter shape ignored

**Observation.** R5 reported execution `status: success` with 3.67s node execution time — a real Google API round trip. But R6's read-back showed the sheet had received **the webhook's own payload as columns** (`headers`, `params`, `query`, `body`, `webhookUrl`, `executionMode`) at `row_number: 2`. The intended 13-column header row was never written; the supplied `rawData` expression did not take effect and the node fell back to mapping the incoming item's fields.

**Why this matters more than the defect itself.** Every layer above the data reported success: the webhook returned HTTP 200, the execution status was `success`, and the node reported `itemsOutput: 1`. Only reading the actual resource state revealed the truth. This is a live, first-party demonstration of `post-deployment-verification.md`'s rule — **invocation success is never proof of deployment success, and deployment success is never proof of runtime correctness.**

**Classification.** Implementation Defect, in the D08 provisioning helper rather than in a PRJ-0001 delivery artifact. **It is the same defect class as D08-DEF-002** (`googleSheets` parameter shape not matching runtime behavior), which was found in a real PRJ-0001 artifact. Two independent instances of the same class, in two different artifacts, from two different authors — that is a genuine pattern rather than a one-off, and it is recorded for `/learn`.

**Current state of the Sheet:** contains one junk row from R5. The 13-column header is **not** correctly written. Recorded honestly rather than cleaned up silently.

---

# Round 2 — Defect-class remediation and verification (2026-08-06)

## Root cause of the googleSheets defect class

The n8n Google Sheets v4 node requires `documentId` and `sheetName` as **resourceLocator objects**, and `matchingColumns` **nested inside `columns`**. All 7 of PRJ-0001's googleSheets nodes shared one defective assumption:

| Field | Artifacts had | Runtime requires |
|---|---|---|
| `sheetName` | plain string | resourceLocator object |
| `documentId` | **absent entirely** | resourceLocator object (required) |
| `matchingColumns` | top-level | nested inside `columns` |

This is why D08-DEF-002 and D08-DEF-003 were one class, not two incidents: DEF-002 was the *validator* rejecting the string-where-object; DEF-003 was the *runtime* silently falling back to auto-mapping when the shape didn't bind. Same cause, two symptoms, one at validation time and one only visible in the resource.

## Remediation — all 7 nodes, not just the observed symptom

| Workflow | Nodes fixed |
|---|---|
| `error-handling.json` | Record error_detail |
| `w1-ai-evaluation.json` | Read Criteria Config |
| `w1-duplicate-detection-recording.json` | Lookup by Email · Write Record (dup=true) · Write Record (dup=false) |
| `w2-human-review-notification.json` | Review Status Changed trigger · Set notification_sent |

Verification: **0 defective nodes remain**; all 6 workflow files still well-formed JSON. `documentId` is bound to the TEST spreadsheet, with an explicit **TEST-BINDING** note on each node and a workflow-level `meta` entry — an environment binding, never a client or Production value.

## Verification by actual resource state

| Check | Result |
|---|---|
| Corrected `appendOrUpdate` writes a row | **PASS** — read-back returned all 13 schema field names as keys, values in intended columns, `row_number: 2` |
| 13-column header correctly established | **PASS** — proven by the above read-back reproducing the schema names |
| Junk row from D08-DEF-003 removed | **PASS** — final read-back returns zero data rows (correct headers-only state) |
| Remediated candidate redeployed | **PASS** — `yrIl76JCaaQloshO` updated |
| Redeployed candidate re-validated | **PASS** — was `errorCount: 1` ("Expected object but got string"), now **`valid: true`, `errorCount: 0`** |

**D08-DEF-002: REMEDIATED. D08-DEF-003: REMEDIATED** (same class, same fix, runtime-proven).

## What was NOT exercised, and why

TC-01, TC-02 and TC-03 were **not run end-to-end**. The two prerequisites that blocked them in round 1 are now cleared — the header is correctly established and the Sheets defect class is remediated — but three genuinely separate pieces of engineering remain, none of which is a Founder decision:

1. **Sub-workflow wiring.** The four `executeWorkflow` nodes reference sibling workflows by identity. Only `error-handling` is deployed; the other five must be deployed and their runtime IDs wired into each caller.
2. **Credential binding across all six workflows.** Only the Sheets node on the deployed candidate is bound. Drive, Gmail and the LLM nodes need binding on the remaining workflows.
3. **A file-upload form trigger invocation.** TC-01 enters through `formTrigger` with a PDF upload; driving that end-to-end requires a multipart submission against the activated form endpoint, not the JSON webhook path used for the probes.

Plus one deliberate gap: the AI evaluation step still needs a clearly-labelled **TEST-ONLY / NOT CLIENT-APPROVED** evaluation configuration, which would prove the technical LLM path without standing in for the client's real criteria.

No end-to-end PASS is claimed. **Integration paths and the Sheets write contract are proven; the assembled pipeline is not.**

## 2026-08-09 addendum — applicant_name defect fixed, evaluation_reason + interview_date added, W2 lolos/tidak-lolos branching built and verified

Resumed via the short-prompt pattern (`/asdp-state` + Founder Objective). Three Founder-directed changes to the live TEST deployment, all verified against real execution, not asserted:

**1. `applicant_name` defect fixed.** `sheets-record-schema.md` declared `applicant_name` from the start, but it was silently dropped after `w1-intake-validation-storage`'s `Prepare Extraction Input` node and never written by `w1-duplicate-detection-recording`'s two `Write Record` nodes — every row's `applicant_name` cell was empty. Threaded through all four W1 sub-workflow calls (Intake → Extraction → AI Evaluation → Duplicate Detection); each hop patched live via `n8n_update_partial_workflow`, each workflow re-validated `errorCount: 0`.

**2. `evaluation_reason` and `interview_date` added to the Applicant Records schema.** `evaluation_reason` is populated by extending the Gemini evaluation prompt (`w1-ai-evaluation`'s `Construct Prompt`) to request a third JSON field (`reason`, alongside the existing `score`/`recommendation`), parsed and threaded the same path as `applicant_name`. `interview_date` is a new **manually-filled** column (same manual pattern as `review_status`) — no calendar/Meet-link automation was built, since none was authorized or specified; left blank, the notification falls back to an explicit placeholder rather than fabricating a date. Both columns added as real Sheet headers (`N1:O1`, confirmed by the Sheets API's own `updatedRange` response), not just referenced in code.

**3. W2 now branches lolos vs. tidak lolos.** Previously W2 sent one generic message regardless of outcome. Added a `Lolos Evaluasi?` IF node (`recommendation == "shortlist"`) between the existing `Is Reviewed?` gate and notification send, with two Gmail nodes converging back into the existing `Set notification_sent = true` idempotency step (D08's self-trigger duplicate-send fix is untouched). Templates follow the Founder-ratified copy:
- **Lolos:** `Hi, {applicant_name}. Selamat CV kamu lolos seleksi, silahkan ikuti interview secara Online melalui aplikasi Google Meet pada tanggal {interview_date, or a placeholder if blank}...`
- **Tidak lolos:** a drafted "mohon maaf" version including `{evaluation_reason}` — explicitly marked in the message body as **DRAFT, pending Founder correction**, per the Founder's own instruction ("nanti saya koreksi ulang").

**Verification performed (live, real execution, not simulated):**
- All 5 touched workflows re-validated via `n8n_validate_workflow`: `errorCount: 0` on all.
- Two synthetic record pairs (one `shortlist`, one `reject`) written directly to the TEST Applicant Records sheet via a disposable seeding workflow, then marked `reviewed`. W2's Sheets Trigger picked up the change on its next poll (execution `222`): `Lolos Evaluasi?` correctly routed the shortlist record to `Send Candidate Notification` and the reject record to `Send Rejection Notification`; both Gmail sends confirmed `SENT` by Gmail's own API response (message ids `19fe5442fe41b98c`, `19fe5443b6f187ab`); `Set notification_sent = true` ran for both. Recipient was the Dedicated ASDP Test Account throughout — never a real candidate address.
- A field-name bug was caught during this same verification pass: the rejection template initially referenced `$json.reason` instead of the actual column `evaluation_reason` (silently falling back to the generic placeholder text). **Fixed and re-validated (`errorCount: 0`), but NOT independently re-confirmed by a live trigger fire** — three further seeded row-pairs (`TEST-LOLOS/REJECT-1786258022204`, `...-1786258445618`) were written and marked `reviewed` after the fix, including one attempt after explicitly deactivating/reactivating W2, but W2's Sheets Trigger did not pick any of them up within this session's observation window (~12 minutes). Read-back confirms the row data itself is correct at rest (`applicant_name`, `evaluation_reason`, `interview_date` all populated as expected — see the four extra rows now in the TEST sheet, left in place as evidence, not cleaned up). **Classified: TEST-environment trigger-polling reliability issue, not a defect in this round's logic change** — the same Sheets Trigger already shows several unrelated `status: error, startedAt: null` executions predating this session (2026-08-08), i.e. a pre-existing intermittent-registration pattern, not something this round introduced. Treat the rejection-template fix as **code-reviewed and schema-consistent, confirmed correct by static inspection and by the successful first-run execution's parallel logic (`Send Candidate Notification` used the equivalent correct field name and worked), but not yet independently proven by its own live Gmail send.**
- Disposable seeding/verification workflows (`TEMP Seed Job Opening Sheet`, `TEMP Add Sheet Headers`, `TEMP W2 Branch Verification`, `TEMP Read Applicant Records`) were deleted after evidence capture — none left in the TEST n8n instance.

**Boundary preserved:** semantic/business correctness of the AI evaluation criteria remains unclaimed (Blueprint OTQ1 still open) — this addendum only proves the new plumbing and branching execute correctly against whatever `recommendation` the (still TEST-ONLY, not client-approved) criteria produce.

## 2026-08-09, same day — LLM call migrated from raw HTTP Request to the AI Agent node

Founder directed the AI evaluation step to use n8n's `AI Agent` node instead of a raw `httpRequest` call to the Gemini REST endpoint. `w1-ai-evaluation`'s `LLM Evaluation Call` (httpRequest) node replaced with `AI Agent` (`@n8n/n8n-nodes-langchain.agent`, typeVersion 3.1) wired to a `Google Gemini Chat Model` sub-node (`@n8n/n8n-nodes-langchain.lmChatGoogleGemini`, typeVersion 1.1, same `models/gemini-2.5-flash` TEST binding, same `googlePalmApi` credential) via an `ai_languageModel` connection — not `main`; this connection type had to be set through a full-workflow update (`n8n_update_full_workflow`) after the partial-update tool's `addConnection` operation repeatedly defaulted new AI sub-node links to `main` regardless of the `connectionType` parameter, which the validator correctly rejected each time.

`Parse Structured Response` updated to read the Agent's `output` field (`item.json.output`) ahead of the old `candidates[...].text` path (kept as a fallback, harmless either way — the same regex-based JSON extraction covers both).

**Verified by live execution**, not simulated: a disposable smoke-test workflow (Webhook → AI Agent → Google Gemini Chat Model, deleted after) confirmed a real Gemini round-trip returning the expected 3-field structured JSON (`score`, `recommendation`, `reason`) inside the Agent's `output` string. `pCr8bayqGDByE7S8` re-validated `errorCount: 0` after the change. Not yet re-run through the full W1 pipeline end-to-end in this session (multipart CV upload requires a real file POST, which this session's tool sandbox could not reach directly — only the MCP-proxied `n8n_test_workflow` path was reachable) — the isolated Agent+Model smoke test is the verification basis; a full TC-01-style re-run is recommended before the next Founder Manual Verification pass.

## Blockers — unchanged, not resolved here

- **Evaluation criteria (OTQ1)** — client business input. AI *connectivity* is now proven (R1); AI *semantic correctness* remains BLOCKED. These are different questions and are reported separately.
- **Resume format mix / OCR (OTQ2)** — client business input. Synthetic **text-based** PDFs exist; **no scanned-PDF or OCR coverage is claimed or inferred** from them.
- **Client Production Environment** — untouched, unchanged.
- **Client acceptance** — not sought, not inferred.
