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

## What was NOT exercised, and why

The PRJ-0001 delivery workflows themselves were **not** run end-to-end. Their entry point is a form trigger feeding extraction → evaluation → recording → notification, and three prerequisites remain unmet: the Sheet's header row is not correctly established (D08-DEF-003), the artifacts' own `googleSheets` parameter shapes are still defective (D08-DEF-002, unremediated), and the AI evaluation step needs a labelled TEST evaluation configuration that does not yet exist.

No end-to-end PASS is claimed. **Integration paths are proven; the assembled pipeline is not.**

## Blockers — unchanged, not resolved here

- **Evaluation criteria (OTQ1)** — client business input. AI *connectivity* is now proven (R1); AI *semantic correctness* remains BLOCKED. These are different questions and are reported separately.
- **Resume format mix / OCR (OTQ2)** — client business input. Synthetic **text-based** PDFs exist; **no scanned-PDF or OCR coverage is claimed or inferred** from them.
- **Client Production Environment** — untouched, unchanged.
- **Client acceptance** — not sought, not inferred.
