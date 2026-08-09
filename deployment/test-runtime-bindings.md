# PRJ-0001 — TEST Runtime Bindings

The **deployment binding layer**. Implementation artifacts stay environment-neutral; every environment-specific value lives here and is resolved at deploy time.

## Why this file exists

D08 round 3 established that several values the runtime requires are **not** implementation facts: n8n workflow IDs, n8n credential IDs, Google resource IDs, and model bindings. Writing them into `implementation/workflows/*.json` would contaminate provider-neutral source with one environment's identity.

The source therefore carries **`<BIND:token>`** placeholders inside the correct runtime *shape*, and this file maps token → actual TEST value. Deployment resolves them; source never changes between environments.

**One deliberate exception, already recorded:** `googleSheets.documentId` currently carries the TEST spreadsheet ID directly (D08 round 2), because the node requires a resourceLocator value to validate. Each such node carries an explicit `TEST BINDING` note. Migrating it to a `<BIND:>` token is a follow-up, not a new decision.

## Workflow ID bindings

**Superseded 2026-08-09 (consolidation) — see the dated section below for the current 3-workflow topology.** Historical record of the original 6-workflow topology, kept for traceability:

| Token | Target workflow | TEST n8n ID | Status |
|---|---|---|---|
| `<BIND:error-handling>` | PRJ-0001 — Error Handling (shared) | `yrIl76JCaaQloshO` | Still active, unchanged |
| `<BIND:w1-extraction>` | PRJ-0001 — W1 Resume Extraction | `snd72efJIOvHR9cA` | **Deleted — merged into the pipeline workflow** |
| `<BIND:w1-ai-evaluation>` | PRJ-0001 — W1 AI Evaluation and Scoring | `pCr8bayqGDByE7S8` | **Deleted — merged into the pipeline workflow** |
| `<BIND:w1-duplicate-detection-recording>` | PRJ-0001 — W1 Duplicate Detection and Recording | `O0oZ4UOg1rdZc77X` | **Deleted — merged into the pipeline workflow** |
| *(entry, no caller)* | PRJ-0001 — W1 Intake, Validation and Storage (MAIN) | `4nDrnkEQ9I9j8mjw` | **Renamed + expanded in place — now the merged pipeline** (same ID) |
| *(independent trigger)* | PRJ-0001 — W2 Human Review Gate and Notification | `jc9ZA8Np8Dpts9Ey` | Still active, unchanged |

**2026-08-09 (naming/folder only):** all six renamed (`Test` dropped from display name, per ADR-0023) and moved into a single n8n folder `PRJ-0001` (folder id `TCIWOJSBtFrFHPCE`). Workflow IDs, credentials, and Environment Role were unaffected by this step.

## 2026-08-09 — W1 consolidated into one workflow (Founder-directed)

The Founder found the 4-workflow W1 split (Intake, Extraction, AI Evaluation, Duplicate Detection) hard to read and directed consolidation. This is the exact scoped consolidation ADR-0021 left open for "a future Founder decision," now made: `w1-extraction`, `w1-ai-evaluation`, and `w1-duplicate-detection-recording` are merged directly into the former Intake workflow (`4nDrnkEQ9I9j8mjw`), which keeps its ID and is renamed to **`PRJ-0001 — W1 Applicant Pipeline (Intake → Extraction → AI Evaluation → Duplicate Detection)`**. All internal `executeWorkflow` hops between these four stages are gone — the whole thing is one sequential chain of 23 nodes, readable top-to-bottom in a single canvas. The three now-empty sub-workflows were deleted after the merge.

**What did NOT change:**
- **Error Handling stays a separate workflow** — it is a genuine shared sink called from 4 different points inside the merged pipeline (all fan into one `Route to Error Handling` node), the reuse-by-multiple-callers exception ADR-0021 itself names.
- **W2 stays separate** — Blueprint §SS4 requires it triggered independently of W1 (decoupling "record written" from "candidate notified"), which one workflow cannot express (a workflow has exactly one trigger). Unaffected by this consolidation.
- No node's logic changed — every expression, credential, and Sheets/Drive/Gmail binding is copied as-is; only 3 internal cross-references were updated from `$('Sub-workflow Trigger')` (a node that no longer exists) to the equivalent identity-carrying node in the merged chain (`Generate record_id` for early identity fields, `Parse Structured Response` for the post-evaluation record used by Duplicate Detection).

**Verified:** `n8n_validate_workflow` on the merged pipeline: **`errorCount: 0`**, 23 enabled nodes, 25 valid connections. **Not yet re-run end-to-end with a real CV upload in this session** (this session's tool sandbox cannot reach the webhook directly for a multipart file POST) — a TC-01-style live re-run is recommended before the next Founder Manual Verification pass, same open item already carried from the AI Agent migration earlier the same day.

Current topology (3 workflows in the `PRJ-0001` n8n folder):
```
PRJ-0001 — W1 Applicant Pipeline (4nDrnkEQ9I9j8mjw)
  Webhook Intake → Generate record_id → Validate File Type/Size → Upload to Drive
    → Prepare Extraction Input (decode PDF text) → Extraction Succeeded?
    → AI Agent (prompt inlined, no separate Construct Prompt step; + Gemini Chat Model)
    → Parse Structured Response → Response Valid?
    → Lookup Existing Records → Determine Duplicate Status → Duplicate Found?
    → Write Record (true/false)
  each failure branch → its own direct "Route to Error Handling (<failure_mode>)" call
    → PRJ-0001 — Error Handling (shared) (yrIl76JCaaQloshO)

PRJ-0001 — W2 Human Review Gate and Notification (jc9ZA8Np8Dpts9Ey)
  independent Sheets Trigger, no inbound edge from the pipeline above (Blueprint SS4)
```

## 2026-08-09, same day — further slimmed: 5 glue-only Code nodes removed, error-routing de-indirected

Founder questioned why a `Construct Prompt` Code node existed instead of feeding the AI Agent directly, and asked for the leanest pipeline possible without changing functional behavior. Five nodes whose entire job was gluing literals/fields together for the *next* node — not doing any real cross-item or cross-node work — were removed, folding their content into the node that actually needed it:

| Removed | Folded into |
|---|---|
| `TEST-ONLY Criteria (not client-approved)` | Inlined as static text directly in the `AI Agent` node's prompt expression |
| `Construct Prompt` | Same — `AI Agent`'s `text` parameter now reads `=You are scoring a CV. ...{{ $json.resume_text }}...` directly, no intermediate `prompt` field |
| `Extract Text Content` | Folded into `Extraction Succeeded?`'s own IF condition (`{{ $json.resume_text.trim() }}` `notEmpty`) — the separate `extracted_text`/`extraction_status` fields it computed are gone; downstream reads `resume_text` directly |
| `Build Error Input (unreadable_pdf/extraction_failure/llm_parse_failure/duplicate_application)` (4 nodes) | Each failure branch now calls Error Handling **directly** with its `record_id`/`failure_mode` set inline via the `executeWorkflow` node's own `defineBelow` mapping — no separate node just to build that 2-field object first |

**What was deliberately kept, and why (same functional-quality bar):**
- `Generate record_id` — real work (ID generation, webhook body parsing), not glue.
- `Prepare Extraction Input` (renamed `Prepare Extraction Input (decode PDF text)`) — real work (binary→text decode across two node references), not glue.
- `Parse Structured Response` — real work (regex JSON extraction, error containment); `evaluation_criteria_version` changed from a value read off a now-deleted upstream node to an inline literal constant, same value, one fewer hop.
- `Determine Duplicate Status` — genuinely not foldable into the following IF node: the Sheets lookup node's output *replaces* `item.json` with the matched row (or an empty object on no match), so this step both restores the pre-lookup payload and guarantees the IF node never starves on a 0-match read. No single field expression can do both of those.

## 2026-08-09, same day — client-artifact de-TEST-ing pass (ADR-0025)

Founder ratified that no client-deliverable artifact may contain the word "TEST" in any form. Applied across all three PRJ-0001 workflows: sticky notes reworded (W1's `Blueprint Traceability` and `TEST-ONLY Evaluation Config` notes, W2's `Blueprint Traceability` note, Error Handling's `Blueprint Traceability` note), the AI Agent's prompt text ("TEST ONLY:" prefix removed), `Parse Structured Response`'s stored `evaluation_criteria_version` literal (`TEST-ONLY-v0-not-client-approved` → `v0-pending-approval`), and W2's two email subject lines (`[TEST] ` prefix removed) plus the rejection draft's footer note. The spreadsheet itself was renamed via the Drive API (confirmed by Google's own response) from `PRJ-0001 Test - Applicant Records` to `PRJ-0001 — Applicant Records`.

**Unchanged, deliberately:** every actual environment binding (credentials, hardcoded TEST Gmail recipient, spreadsheet/document ID) — only the visible wording moved. Environment identity stays tracked in this file per ADR-0016/0017/0018, exactly as before. All 3 workflows re-validated `errorCount: 0` after this pass.

## 2026-08-09, same day — dual intake, new record_id/dedup scheme, real PDF extraction, OCR attempted and reverted

Founder ratified three changes and asked one open question, all now resolved into the live pipeline (ADR-0026 + this section):

**1. Dual intake (ADR-0026).** Added an `n8n Form Trigger` node (`Applicant Form`: Applicant Name, Applicant Email, `Lowongan` dropdown, Resume file) alongside the existing `Webhook Intake`, both feeding the same `Generate record_id` node with the same field names. The Form Trigger will still 404 on this instance (D08's root-caused, not-practically-remediable defect, unchanged) — it's present for contract-completeness and for whenever that defect is resolved, per ADR-0026. `Webhook Intake` remains the only currently-working entry point on this instance.

**2. New record_id / dedup scheme.** `record_id` is now `job_open_id + "_" + applicant_email` (email trimmed + lowercased), replacing `REC-<timestamp>-<rand>`. This is also the new duplicate-detection key: `Lookup Existing Record by ID` now filters `Applicant Records` by `record_id` directly (was: by `applicant_email` alone). One `applicant_email` may have at most one record per `job_open_id`; the same person may still apply to a different job opening — confirmed with the Founder as "per lowongan," not global. `job_open_id` is a new column on `Applicant Records`, sourced from the applicant's dropdown selection at intake (values match `Job Opening` sheet rows, e.g. `Front End Developer_2026-08-01`).

**3. Real PDF text extraction.** The naive `Buffer.from(...).toString('latin1')` byte-decode (never true text extraction, just happened to sort-of work for simple PDFs) is replaced by n8n's built-in, free, local `Extract From PDF` node (`n8n-nodes-base.extractFromFile`, operation `pdf`) — confirmed via live execution against a real stored CV (`Ahmad Pratama_REC-1786110275137-757.pdf`) to correctly extract the actual document text, not a garbled byte-decode. `Upload to Google Drive`'s `inputDataFieldName` and `Extract From PDF`'s `binaryPropertyName` are both now the expression `={{ Object.keys($binary)[0] }}` instead of a hardcoded key, so either trigger's differently-named file field works without a workflow edit.

**4. OCR fallback for scanned/image PDFs — attempted, found unsafe, reverted.** Investigated a free approach (no new paid service): route to the standalone `Google Gemini` node (`resource: document, operation: analyze`) using the same already-free Gemini credential, only when native extraction returns no text. **Live-tested against the same real PDF twice and found it does not work safely**: instead of transcribing the actual attached file, the node returned a fully fabricated, generic resume — a *different* fabricated resume on each of the two runs (a "John Doe / Project Manager" CV neither run related to the real "Ahmad Pratama / UI-UX Designer" PDF actually sent). Root cause not fully isolated (tried both an expression and a literal `binaryPropertyName`; both hallucinated) — plausible causes include the binary genuinely not reaching the API despite the parameter being set, or a node-version/API mismatch on this n8n instance. **This is a hallucination risk, not an honest failure mode** — scoring a candidate against fabricated content would be worse than the pre-existing "reject with extraction_failure" behavior it would have replaced. Removed entirely rather than shipped half-verified: `Has Native Text?`'s false branch now correctly routes to `Route to Error Handling (extraction_failure)`, same as before this round. **Scanned/image-only PDFs remain unsupported — Blueprint OTQ2 stays open.** A safe OCR path is a candidate for a future round with more investigation budget (isolate whether the binary attachment itself is the defect, consider n8n version, consider a different node/approach) — not attempted again without a resolved root cause.

## 2026-08-09, same day — dynamic job dropdown, OCR (Tesseract.js) attempted and reverted

**1. Dynamic job dropdown, no credential.** The Founder pushed back on a self-mutating-workflow approach (would need an n8n API key stored as a credential — impractical and a confidentiality concern for a client-deliverable artifact) and asked for a "custom script Expression" approach instead. Correct call: n8n's `Form Trigger` fields are genuinely static (no expression support — there's no upstream data at trigger time), but the separate `n8n Form` node (used for page 2+ of a multi-page form) supports **Define Form → Using JSON**, and that JSON field *does* accept a runtime expression. Rebuilt intake as two pages: page 1 (`Applicant Form`, Form Trigger) collects Name/Email/Resume only; a `Fetch Open Jobs` → `Filter OPEN` → `Build Dropdown JSON` chain runs between the pages and feeds page 2 (`Job Selection`, `n8n Form` node) a dropdown built live from the `Job Opening` sheet's `status: OPEN` rows — column `job_name` (renamed from `nama_job`, confirmed via the Sheets API's own `updatedRange` response) is the source per Founder direction. No API key, no self-mutation, no credential embedded in the workflow.

Selecting by `job_name` alone loses the link to `job_open_id` (needed for `record_id`), so a `Resolve job_open_id` step (real logic, not glue — matches the selection against a fresh `status: OPEN` read, accepting either a `job_name` or an already-exact `job_open_id`) runs for **both** intake paths after `Generate record_id`, followed by a `Job Found?` gate routing an unresolvable selection to Error Handling as a new failure mode, `invalid_job_selection` (added as a 5th rule on the shared Error Handling workflow's switch).

**A real bug was caught and fixed during verification, not assumed correct:** `n8n_googleSheets`'s `read` operation with a `filtersUI` filter (`status = OPEN`) silently returns only the **first** match, not all matches — confirmed live (fetching by filter returned 1 row; Project Management, also OPEN, was missing). Both open-jobs reads (dropdown-building and resolution) now read the sheet **unfiltered** and pass through a proper `Filter` node instead. Re-verified live: dropdown JSON correctly lists both currently-OPEN jobs, and resolution correctly matches a `job_name` (`"Project Management"` → `"Project Management_2026-08-01"`), an already-exact `job_open_id`, and correctly returns no match for a `CLOSED` job (`Backend Developer`).

**2. OCR (Tesseract.js) — installed, found incompatible, cleanly removed.** Founder-authorized infrastructure change, following the exact D08 round-7 backup discipline: `n8n-app` stopped, `n8n-docker_n8n_data` volume tarred to a fresh timestamped backup (`n8n_data_pre-tesseractjs-install_20260809-105458.tgz`), integrity verified (`gzip -t` + `tar tzf`, contents confirmed sane), container restarted, `n8n-nodes-tesseractjs` installed into `~/.n8n/nodes` (n8n's standard community-package location) via `npm install`, container restarted again. **Result: does not work on this image.** n8n's own log shows the real cause: `Warning: Cannot load "@napi-rs/canvas" package: "Error: Failed to load native binding"` — a native-binding dependency (image-processing library) incompatible with n8n's Alpine/musl-based official Docker image, not a configuration mistake. n8n never registers the node type as a result (`Unrecognized node type: n8n-nodes-tesseractjs.tesseractjs` on every subsequent load attempt, confirmed by trying to activate a workflow using it). **Cleanly reverted**: `npm uninstall` in the same directory, `installed-nodes/package.json` back to its pre-install empty-dependencies state, container restarted, logs confirmed no more canvas/tesseract errors, all 3 PRJ-0001 workflows re-activated automatically exactly as before, main pipeline re-validated `errorCount: 0`. **Scanned/image-only PDFs remain unsupported; Blueprint OTQ2 stays open.** ChatGPT's other suggestions (Python+Tesseract via a separate microservice, cloud OCR) were not attempted this round — Level 1 (Tesseract.js in-process) was the one actually authorized, and it hit a genuine platform-compatibility wall rather than a fixable configuration issue; a microservice is a materially bigger infrastructure decision (new container, new deployment surface) that would need its own explicit go-ahead, consistent with ADR-0021's monolithic-by-default bias against adding components before they're proven necessary.

## 2026-08-09, same day — OCR root cause isolated, working fix verified (raw Gemini REST call)

Docker-based OCR (Tesseract.js, and a Founder-researched `pdftoppm`+`tesseract-ocr`/`ocrmypdf` Dockerfile approach) ruled out entirely: **Founder needs n8n Cloud compatibility**, which excludes any container-level customization regardless of the native-binding fix's technical merits. Redirected to API-only options per Founder direction ("studi ulang dengan api gratis seperti Gemini").

Re-investigated the earlier Gemini hallucination (see the OCR section above) with a raw `HTTP Request` node calling Gemini's REST API directly, bypassing the `@n8n/n8n-nodes-langchain.googleGemini` wrapper node entirely. **First attempt reproduced the failure mode exactly** — but this time with a diagnosable error instead of silent fabrication: `400 Bad request — Base64 decoding failed for "filesystem-v2"`. Root cause: n8n's binary field `.data` property is a **storage reference string** once n8n's binary-data mode promotes it out of memory (confirmed: `"filesystem-v2"`), not the actual base64 content — reading it directly and forwarding it as a payload sends garbage. This is very likely the same defect the wrapper node hit silently (a tolerant receiver like a multimodal model doesn't validate the "document" it received is real, and answers from the text prompt alone instead of erroring — see the new insight, INS-036).

**Fix:** resolve the real binary buffer via `this.helpers.getBinaryDataBuffer(itemIndex, propertyName)` inside a Code node, then `.toString('base64')`, before constructing the Gemini `inline_data` request body. **Verified working, twice, against the same real stored CV** (`Ahmad Pratama_REC-1786110275137-757.pdf`): both runs returned matching, correct transcription (not fabricated — content matches the same CV's native-extracted text from earlier in this document), and both responses' `usageMetadata.promptTokensDetails` show `{modality: "DOCUMENT", tokenCount: 774}`, Gemini's own confirmation that it processed the actual attached document.

**Integrated into the live pipeline (Founder-confirmed, same day).** `Has Native Text?`'s false branch now runs: `Build OCR Request Body` (resolves the real binary buffer via `getBinaryDataBuffer`, builds the Gemini `inline_data` request) → `Raw Gemini OCR Call` (`HTTP Request` node, same `googlePalmApi` credential) → `Normalize OCR Result` (extracts `candidates[0].content.parts[0].text`) → `AI Agent`, whose prompt now reads `native_text || ocr_text`. The now-unreachable `Route to Error Handling (extraction_failure)` node was removed (Node-Level Leanness Standard, ADR-0024) — if OCR *also* returns empty text, the pipeline degrades to an empty-CV evaluation caught by the existing `Response Valid?` gate as `llm_parse_failure`, not a dedicated `extraction_failure` record; this relabeling is a deliberate proportional simplification, not an oversight. Pipeline re-validated `errorCount: 0`, 33 executable nodes. See [DEF-022](../../../../ASDP/project-management/engineering-knowledge/Deferred-Decisions.md) and [INS-036](../../../../ASDP/project-management/engineering-knowledge/Engineering-Insights.md).

## 2026-08-09, same day — full pipeline live-verified end-to-end (both paths)

Sandbox network limitation worked around: an n8n-internal "self-POST" harness (a disposable workflow using `HTTP Request` with `multipart-form-data` to call the real production `prj0001-intake` webhook, executed via the MCP-proxied `n8n_test_workflow` call rather than this session's own network-isolated shell) let the actual live pipeline be exercised end-to-end for the first time since today's consolidation, dual-intake, and OCR-integration changes — not just each piece in isolation.

**Native-text path (real CV, real submission):** `Webhook Intake` → `Generate record_id` → job resolution (`"Front End Developer"` correctly resolved to `job_open_id: "Front End Developer_2026-08-01"`, ignoring two CLOSED postings also present in the sheet) → `record_id` composite key built correctly → Drive upload (real file, confirmed via Drive's own API response) → `Extract From PDF` (real native text, matches the file's actual content) → `Has Native Text?` correctly true → `AI Agent` (real Gemini call, score 97/shortlist) → dedup check (correctly `duplicate_found: false` on first submission) → row written to `Applicant Records`. **Re-submitting the identical applicant_email + job_open_id a second time correctly flagged `duplicate_flag: true`** — confirms the new dedup key works against a real repeat, not just in isolated logic.

**OCR path (synthetic empty PDF, forced `has_native_text: false`):** a minimal valid but completely blank one-page PDF (no text objects) was generated and submitted the same way. `Extract From PDF` correctly returned no text; `Has Native Text?` correctly routed false; `Raw Gemini OCR Call` executed for real (`usageMetadata.promptTokensDetails` confirms `{modality: "DOCUMENT", tokenCount: 258}` — genuine document processing, not skipped); `Normalize OCR Result` correctly returned an empty `ocr_text` (nothing to transcribe, and — critically — **no fabricated content**, unlike the original wrapper-node defect); `AI Agent` honestly reported `score: 0, recommendation: "hold", reason: "No CV text or role description was provided, making it impossible to assess relevance"` rather than inventing a plausible-sounding evaluation. This is the strongest possible confirmation of INS-034/INS-036's fix: presented with genuinely nothing to read, the corrected pipeline says so instead of guessing.

**Test artifacts cleaned up after verification:** all 11 rows in `Applicant Records` (8 pre-existing synthetic rows from earlier sessions plus 3 from this round) deleted via Sheets API (`batchUpdate` `deleteDimension`, verified `0` rows remaining, header row untouched); both test Drive files deleted. No test data left in the deliverable-facing spreadsheet or Drive folder.

**Outstanding, deliberately not covered by this round:** the Form Trigger (page 1 + dynamic page 2) path was not exercised — it still 404s on this instance per D08's root cause, unchanged by anything in this session — only the Webhook path (real production entry point today) was live-verified.

**Verified:** pipeline re-validated `errorCount: 0` after all four changes (22 executable nodes). `Extract From PDF` verified correct via live execution against a real file. `Generate record_id`'s new composite key is simple string concatenation (low risk) but **not yet exercised end-to-end via a real intake submission in this session** — same sandbox network limitation as every other unverified item this session; still recommended before the next Founder Manual Verification pass, now including: dual-trigger field mapping, the new dedup behavior (submit the same email+job twice, confirm the second is flagged duplicate; submit the same email to a different job, confirm it's accepted), and native-PDF extraction on a fresh live submission (not just a pre-existing stored file).

**Verified:** re-validated `errorCount: 0`, node count **19 executable nodes** (was 23 right after consolidation, 26 before that — down from the original 6-workflow count of ~38 nodes total). A disposable smoke test (prompt-only AI Agent call, deleted after) confirmed the inlined prompt expression produces an identical structured response to the pre-change version. Full pipeline has still not been re-run with a real CV upload in this session (same sandbox network limitation as before) — still recommended before the next Founder Manual Verification pass.

## Credential bindings (by id/name only — never values)

| Class | TEST credential | ID |
|---|---|---|
| Google Drive | Google Drive account | `JwP20ZRGAFZtMnzJ` |
| Google Sheets | Google Sheets account | `dK1PrRScgmvDvDsa` |
| Google Sheets Trigger | Google Sheets Trigger account | `8DLNGCR3YKCQl4Yu` |
| Gmail | Gmail account | `fqN3O2IpirUQDjEL` |
| Gemini / Google AI | Google Gemini(PaLM) Api account | `oi89bxl8q2vWN0Dk` |

**No credential value appears in this file, in any artifact, or in any Evidence Record.**

## Google TEST resource bindings

| Resource | Identity |
|---|---|
| ASDP root folder (parent) | `1A1TnE1_dHp7etzPYDLOwNdakyUBHllo6` — created 2026-08-06 by ASDP's own Test Environment designation. **A second, unrelated folder also named `ASDP` exists in this Drive** (`1Ra002pg7owfsz6U7ReXB_ZdEDPepYY-a`, predates this project, created 2026-07-17) — always bind by this ID, never re-discover by searching the name `ASDP` (confirmed by direct Drive API lookup, D08 round 8 Founder Q&A). Canonical reference (separate repository): ASDP's `frameworks/delivery-framework/.claude/knowledge/default-test-environment-profile.md`. |
| Project Drive boundary | `ASDP/PRJ-0001/` — `1G2RiX-oCJqpFPs7lyjCLNgZM5F1EjrgD` (child of the ASDP root folder above) |
| Applicant records Sheet | `PRJ-0001 — Applicant Records` (renamed 2026-08-09 to drop "Test", em dash to match ADR-0023's convention — per ADR-0025) — `1eiVPbSbJrZSiNa3FaKXgwQCJtxuew5zDeW7Qujgdd_8` |

## Model binding

| Purpose | TEST binding |
|---|---|
| AI evaluation LLM | Gemini free tier, `models/gemini-2.5-flash` (proven, D08 round 1) |

**TEST binding only.** Not a Production provider decision, not a client requirement — per ADR-0017/ADR-0018.

## Current deployment state - all 6 deployed, published, validating clean

All six workflows are deployed and published; every one validates at `errorCount: 0`. The Duplicate Detection workflow additionally reports 11 validator warnings that were evaluated and identified as false positives on n8n cross-node reference syntax, not defects.

**All `<BIND:>` tokens resolved. Zero unresolved tokens.**

## Publish-order constraint (D08-DEF-009)

n8n refuses to publish a workflow whose referenced sub-workflows are unpublished: *"Please publish all referenced sub-workflows first."* Publication therefore follows the same callee-first order as deployment. This is a deployment-ordering constraint, not a source defect.

## Verified deployed topology

Read from the **deployed and published** definitions, not from source:

```
Intake (4nDrnkEQ9I9j8mjw)
  --> Extraction (snd72efJIOvHR9cA)
        --> AI Evaluation (pCr8bayqGDByE7S8)
              --> Duplicate Detection (O0oZ4UOg1rdZc77X)

all four W1 stages --> Error Handling (yrIl76JCaaQloshO)

W2 (jc9ZA8Np8Dpts9Ey) - independent Sheets trigger, no inbound W1 edge (Blueprint SS4)
```

Every expected edge present, every target resolving to the intended PRJ-0001 TEST workflow, no stale ID, no unresolved token, no cross-Project target, no unintended extra edge.

## TC-01 Form Trigger 404 — root cause isolated (D08 round 5)

Round 4 recorded the 404 as an unexplained runtime-capability limit. Round 5 diagnosed it to a specific, reproducible cause using a controlled A/B probe, not inference:

- **Runtime version (authoritative, via `n8n_audit_instance`'s built-in report):** n8n **2.31.7**, outdated (2.32.7 and 2.33.5 available). The earlier "2.68.2" seen in `n8n_health_check` is the `n8n-mcp` npm package version, not the server — do not reuse that figure.
- **Control test:** a fresh, minimal `n8n-nodes-base.webhook` node, created and activated via the API, served **200** immediately on both `localhost:5678` and the public `https://n8n.nugi.my.id` route.
- **Isolation test:** a fresh, minimal `n8n-nodes-base.formTrigger` node, same create/activate path, same two hosts, tested at **typeVersion 1 and 2.2**, returned n8n's own "Problem loading form" 404 in every combination.
- Ruled out: reverse proxy / DNS (localhost fails identically to public), activation lifecycle/staleness (fresh workflow fails immediately), node-shape/typeVersion (both tested versions fail identically), MAIN workflow's own history (isolated probe workflow, no prior edits, fails the same way).
- MAIN workflow (`4nDrnkEQ9I9j8mjw`) Form Trigger itself: `path=prj0001-intake`, `webhookId` present, `typeVersion: 1`, `active: true`, static validation `errorCount: 0` — node definition is not the defect.
- **Classification: PROVIDER-COMPATIBILITY / RUNTIME DEFECT.** Form Trigger route registration does not function on this n8n 2.31.7 instance while ordinary Webhook registration does, independent of workflow history or node version. Not remediable by editing workflow JSON. An n8n version upgrade is the plausible fix but is an infrastructure change outside the Project mutation boundary — requires Founder decision, not an autonomous action.
- Diagnostic probe workflows (`ZZ-DIAG-webhook-registration-probe`, `ZZ-DIAG-form-trigger-probe`) were created read-only-adjacent for this test and deleted afterward; MAIN workflow topology and parameters untouched (only cycled deactivate/activate, which did not change the outcome).
- **TC-01 remains BLOCKED.** TC-02/TC-03/error-path NOT RUN.
- Formalized as Evidence Records: **EV-042** (round 5 runtime facts).

## n8n upgrade decision study (D08 round 6) — no upgrade performed

**Question:** is upgrading the TEST n8n instance an evidence-supported fix for the round-5 Form Trigger defect? **Answer: not proven, but 2.33.5 is a supported hypothesis.**

- **External research** (n8n official GitHub releases/changelog — not runtime evidence about this instance): `2.32.7` and `2.33.5` patches themselves contain no form/webhook/trigger changes. Their parent minor releases differ: `2.32.0` has none either; `2.33.0` (parent of 2.33.5) touches the relevant subsystem — form-endpoint request redirection (#34725), a Form Trigger node fix (#34650), and workflow activation/publish-lifecycle changes (new publish/unpublish API #34745, activate/deactivate API deprecation #34771, a migration-race fix #34685). No exact GitHub issue/PR matches this instance's precise symptom (clean validation + `active:true` + Webhook works + Form Trigger never registers). **No proven fix exists.**
- **Local infrastructure, directly inspected (round 6, no mutation):** container `n8n-app` runs image `docker.n8n.io/n8nio/n8n` with **no explicit version tag** (resolves to whatever `:latest` was on 2026-07-27 pull) — `n8n --version` inside the container independently confirms **2.31.7**. Persistence is **SQLite** in the external named volume `n8n_data`/`n8n-docker_n8n_data` mounted at `/home/node/.n8n` (contains `database.sqlite`, the encrypted credential store, instance `config`, `nodes/`, `storage/`) — this volume is the correct backup/rollback unit. Compose file: `01. LESSON/06. DOCKER BACKUP/docker-compose.yml` (two services: `n8n`, `cloudflare-tunnel`; secrets supplied via `.env`, never read). **Shared-instance blast radius:** one other workflow exists, `"My workflow"` (inactive, LangChain/Gemini nodes only, no Form/Webhook/Google node) — low risk if reactivated later, not ASDP's to mutate.
- **Recommendation: upgrade to 2.33.5 (SUPPORTED HYPOTHESIS, not proven), pending Founder authorization.** 2.32.7 is NOT RECOMMENDED (no relevant code motion in its lineage). Do not upgrade to `:latest`/newest-available generically — pin the explicit tag.
- Formalized as Evidence Record: **EV-043**.
- **No upgrade, no Docker mutation, no Cloudflare change, and no Webhook-Trigger substitution performed this round**, per instruction.

## n8n upgrade EXECUTED (D08 round 7) — hypothesis falsified

**Founder authorized:** "APPROVE TEST n8n upgrade to 2.33.5" (scoped: TEST instance only, backup-gated, no Production/Cloudflare/DNS/other-workflow mutation, no Webhook substitution).

- **Pre-upgrade backup, verified:** `n8n-app` stopped for consistency; full `n8n-docker_n8n_data` volume tarred to a local, git-ignored path (`01. LESSON/06. DOCKER BACKUP/n8n-backups/n8n_data_pre-2.33.5_upgrade_20260807-143111.tgz`, 568KB); `gzip -t` and `tar tzf` confirmed integrity and the complete expected structure (`database.sqlite`, WAL/SHM, `config`, `nodes/`, `storage/`). Retained locally, not committed, not deleted.
- **Upgrade executed:** compose `n8n` image pinned from untagged `docker.n8n.io/n8nio/n8n` to explicit `docker.n8n.io/n8nio/n8n:2.33.5` — the only change made. Recreated only the `n8n` service.
- **Migration/startup: clean.** 14 SQLite migrations completed, no failure; n8n's own log recorded `Recorded version change: 2.31.7 -> 2.33.5`; all previously-active workflows auto-reactivated; `/healthz` 200.
- **Post-upgrade baseline: intact.** All six PRJ-0001 workflow IDs preserved; MAIN re-validates `errorCount: 0`; unrelated `"My workflow"` untouched.
- **Control probe (Webhook):** fresh minimal workflow → **200** on both localhost and public, immediately — trigger registration confirmed healthy on 2.33.5 generally.
- **Experiment (Form Trigger, typeVersion 2.2):** fresh minimal workflow, identical method → **404** on both hosts, identical to the pre-upgrade result.
- **Verdict: UPGRADE HYPOTHESIS FALSIFIED FOR 2.33.5.** The upgrade succeeded on every other dimension; it did not fix Form Trigger registration. Rollback was NOT triggered (per rule: a still-404 Form Trigger is a falsified hypothesis, not an upgrade failure) — the instance remains on 2.33.5, which is a net-neutral-to-positive change (newer, verified-stable, no regression found) even though it did not resolve the blocker.
- Both diagnostic probes deleted after evidence capture.
- **TC-01 remains BLOCKED — genuine stop condition reached** (Form Trigger unavailable on a healthy, current, just-verified instance; per instruction, do not try another version, do not substitute Webhook Trigger, do not ask Founder to click in the UI this round).
- Formalized as Evidence Record: **EV-044**.

## D08 ROUND 8 (FINAL) — root cause confirmed, Webhook fallback adopted, first real E2E PASS

**Root cause confirmed (Verdict B — CONFIRMED, NOT PRACTICALLY REMEDIABLE):** installed `n8n-nodes-base` source shows `FormTriggerV2` registers its production route via two webhook descriptors both tagged `nodeType: 'form'` — a distinct code path from ordinary Webhook nodes. A live `docker logs` capture during a controlled activation caught n8n's own dispatcher saying `"Received request for unknown webhook: ... is not registered"` for the Form Trigger's GET route — direct evidence the `nodeType: 'form'` route is never persisted into the live registry on this deployment, while a plain Webhook registers immediately every time. Fixing this means patching n8n's own installed package — out of the TEST mutation boundary and not durable across image updates.

**Webhook Trigger adopted as the TEST intake mechanism**, verified Blueprint-conformant first: `engineering-blueprint.md` explicitly frames the intake mechanism as *"native n8n form vs. external service — a build-time choice"*, not a client-mandated technology — so this is an **engineering implementation adaptation**, not a business-scope change. MAIN's `Form Trigger` node replaced with `n8n-nodes-base.webhook` (`POST /webhook/prj0001-intake`, multipart, `binaryPropertyName: resume_pdf`). Deviation disclosed here, not hidden.

**First real end-to-end execution.** Four ordinary defects were found and fixed by actual execution (invisible to static validation because nothing had ever reached these code paths): (1) Drive-upload binary field name (`resume_pdf0` vs stale `data`); (2) `executeWorkflow` `workflowId` resourceLocator object unreadable at `typeVersion 1` — bumped to `1.1` on all 6 call sites; (3) duplicate-detection zero-item starvation when no existing row matches — fixed with `alwaysOutputData` + an explicit `duplicate_found` boolean; (4) all 4 "→ Error Handling" call sites silently passed the full upstream item instead of the mapped `{record_id, failure_mode}` object, and Error Handling's own `Switch` node was missing `operator` on all 4 rules (same shape class as round 4 DEF-006, unexercised until now) — fixed with an explicit `Build Error Input` node per call site + `autoMapInputData` + the missing operators + a declared trigger input schema.

**TC-01 PASS, TC-02 PASS, TC-03 PASS, controlled error path PASS** — full execution graphs inspected (not top-level status only); external state independently read back: a real Google Drive file (Google's own API response, not our claim), a real Gemini 2.5 Flash structured response, real Google Sheets rows (TC-02's Lookup independently re-confirmed TC-01's write), and a real Sheets `error_detail` write via the shared Error Handling path.

**Boundary preserved:** runtime correctness proven; semantic/business correctness NOT claimed — TEST-only evaluation criteria remain OPEN, and the low scores are an expected artifact of the already-documented Latin-1-decode "extraction" limitation (OCR/real-text-extraction OTQ2, unchanged).

**W2/Gmail:** NOT RUN — independent, deliberately-inactive Sheets-poll trigger; activating it opens a new autonomous-Gmail-send surface, a separate decision from this round's scope.

Formalized as Evidence Records: **EV-045** (root cause + Webhook fallback decision), **EV-046** (TC-01/02/03 + error path + defect remediation).

## D08 CONTINUATION — W2 / Gmail TEST verification PASS

Resumed via the short-prompt pattern (`/asdp-state` + one-line Objective), reconstructed autonomously with no prescriptive brief.

**Defect found and fixed:** `googleSheetsTrigger`'s `sheetName.mode: "list"` held a plain sheet name instead of a numeric gid — activation failed (`"Sheet with ID Applicant Records not found"`). A recurrence of D08 round 4's DEF-008 class (the Sheets **Trigger** node variant rejects `mode: "name"`, unlike the regular Sheets node) — round 4's fix never reached this node because W2 was never activated until now. Fixed with the tab's actual `sheetId` (gid `1629274334`, looked up via a disposable HTTP probe against the Sheets API) and `mode: "id"`.

**Execution proof (record `REC-1786090017075-287`):** `review_status` set to `"reviewed"` → W2's Sheets Trigger picked it up on its next poll → full 4-node graph PASS → **Gmail's own API response confirms `SENT`** (`id: 19fdd126d23066bf`, `labelIds: ["UNREAD","SENT","INBOX"]`) → Sheets `notification_sent` confirmed written `true`. TEST recipient (`tama28967@gmail.com`) is the same Dedicated ASDP Test Account used throughout D08, never a real candidate address.

**W2 is now active** (previously inactive pending this proof) — a proven working component of the pipeline, matching the other five workflows.

Formalized as Evidence Record: **EV-047**.

**Follow-up, same continuation — self-trigger duplicate-send defect found and fixed.** A second execution fired for the *same* TC-01 record nine seconds after the first: W2 writes `notification_sent` to the row its own Sheets Trigger polls, and `Is Reviewed?` never checked whether a notification had already been sent — a genuine duplicate Gmail send occurred (both to the TEST account, never a real candidate). Contained by deactivating W2 immediately (before a third poll could fire), fixed with an added `notification_sent != "true"` condition, and verified against the exact failure mode: a fresh record (TC-03) sent exactly once, and the inevitable self-triggered re-poll on TC-03's own write-back correctly no-opped (routed false, no send) rather than duplicating.

Formalized as Evidence Record: **EV-048**.
