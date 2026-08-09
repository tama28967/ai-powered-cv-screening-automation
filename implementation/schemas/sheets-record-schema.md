# Sheets Record Schema — PRJ-0001

Traces to Engineering Blueprint §4 ("writes one structured record...") and §6 sequencing item 2. One row per applicant.

| Column | Type | Notes |
|---|---|---|
| `record_id` | string | **2026-08-09 (Founder-ratified):** `job_open_id + "_" + applicant_email` (email normalized: trimmed, lowercased). Replaces the earlier `REC-<timestamp>-<rand>` scheme. Doubles as the duplicate-detection key — one `applicant_email` may have at most one record per `job_open_id`, but the same person may apply to a different job opening. Generated at intake (D03-DEF-005) and threaded through the whole pipeline. |
| `job_open_id` | string | **Added 2026-08-09.** References a row in the `Job Opening` sheet (format `<nama_job>_<start_job_opening date>`, e.g. `Front End Developer_2026-08-01`). Selected by the applicant from a dropdown at intake (ADR-0026). Part of the `record_id` composite key. |
| `applicant_name` | string | From form intake. |
| `applicant_email` | string | Required, unique per applicant — the duplicate-detection key (Blueprint §9, engineering assumption). |
| `application_date` | datetime | Set at intake. |
| `resume_drive_ref` | string | Google Drive file reference (Blueprint §4 — structured naming convention: applicant + timestamp). |
| `extraction_status` | enum | `success` \| `failed_unreadable` \| `failed_scanned` — Blueprint §7 named failure modes. |
| `evaluation_criteria_version` | string | Points at the Criteria Config version used — never inlines the criteria itself (keeps the record correct even after criteria change, per Blueprint's "adjust without a rebuild" requirement). |
| `score` | number \| null | Null until AI Evaluation completes; see `w1-ai-evaluation.json`. |
| `recommendation` | string \| null | Structured output from AI Evaluation, not free text (Blueprint §4). |
| `duplicate_flag` | boolean | Set by Duplicate Detection & Recording against `applicant_email`. |
| `review_status` | enum | `pending` \| `reviewed` — the human review gate (Blueprint §4, in-scope by commitment). W2 only fires on `reviewed`. |
| `notification_sent` | boolean | Set by W2. |
| `error_detail` | string \| null | Populated by the shared Error Handling workflow on any named failure mode. |
| `evaluation_reason` | string \| null | **Added 2026-08-09.** Gemini's own short explanation for its `score`/`recommendation`, requested as a third field in the same structured LLM response (`w1-ai-evaluation.json`'s prompt). Threaded through AI Evaluation → Duplicate Detection → this column. Consumed by W2's rejection email. Not evidence of business-semantic correctness — same TEST-ONLY boundary as `score`/`recommendation` (Blueprint OTQ1 still open). |
| `interview_date` | string \| null | **Added 2026-08-09.** Manually filled by the human reviewer (same manual step as setting `review_status`) before marking a shortlisted record `reviewed`, when W2's pass-path notification should state a concrete interview date. No automated scheduling/calendar integration exists — this is deliberately not invented (no Google Calendar / Meet-link generation in the pipeline). If left blank, W2's email falls back to an explicit "to be confirmed separately by HR" placeholder rather than fabricating a date. |

## Job Opening sheet (added 2026-08-09; now referenced by `job_open_id`, still not consumed for scoring)

A second sheet, `Job Opening`, exists in the same spreadsheet (`PRJ-0001 — Applicant Records`). The Founder populated it directly with real columns beyond the original request: `job_open_id`, `job_name` (renamed 2026-08-09, was `nama_job`), `minimal_score`, `skill`, `minimal_pengalaman_kerja`, `start_job_opening`, `end_job_opening`, `interview_date`, `status` (`OPEN`/`CLOSED`), `overide_status`, `interview_quota`, `count_applicant_record`. A third sheet, `Job List`, holds a simpler reference catalog (`job_name` (renamed 2026-08-09, was `nama_job`), `minimal_score`, `skill`, `minimal_pengalaman_kerja`) without dates/status — presumed the source job openings are cloned from when a new dated posting opens, not yet confirmed.

**Now referenced:** the applicant's chosen `job_open_id` (Founder-added dropdown, ADR-0026) is captured at intake and stored on every Applicant Record — see `record_id` and `job_open_id` above.

**Still not wired into scoring.** `minimal_score` / `minimal_pengalaman_kerja` / `skill` gating the AI evaluation's recommendation against the specific job applied for is a business rule not yet requested — the evaluation prompt remains generic, unchanged. Out of scope for this round; flagged for a future decision, not invented.

**Known limitation — Form Trigger's dropdown is static.** n8n's Form Trigger field options are fixed at design time, not fetched live from the `Job Opening` sheet. The dropdown currently lists the two `status: OPEN` postings as of 2026-08-09 (`Front End Developer_2026-08-01`, `Project Management_2026-08-01`). **Whoever changes a job opening's status must manually update the Form Trigger's field options to match** — there is no automatic sync. The `Webhook Intake` path has no such limitation (it accepts whatever `Lowongan`/`job_open_id` value the caller sends) since it's meant for an external platform that can source the live list itself.

## Traceability
- Blueprint source: §4 System Decomposition (Recording), §9 (dedup key assumption).
- Consumed by: the merged `PRJ-0001 — W1 Applicant Pipeline` workflow (2026-08-09 consolidation — was `w1-duplicate-detection-recording.json` as a standalone sub-workflow; `implementation/workflows/*.json` source files predate the consolidation and are not the deployed ground truth, per the same convention already established for the AI Agent migration — see `deployment/test-runtime-bindings.md`), `w2-human-review-notification.json`, `error-handling.json`.
- Open Technical Questions this schema does **not** resolve: none — this schema is structural only and holds regardless of the three external inputs' eventual answers.
- **2026-08-09 defect closed:** `applicant_name` was declared in this schema from the start but never actually threaded past `w1-intake-validation-storage.json`'s `Prepare Extraction Input` node, and never written by `w1-duplicate-detection-recording.json`'s two `Write Record` nodes — the column existed in the header but every row's value was empty. Fixed by threading `applicant_name` through Intake → Extraction → AI Evaluation → Duplicate Detection (all four `<BIND:>`-bound sub-workflow calls), verified end-to-end via live n8n execution (see `deployment/test-runtime-results.md` addendum).
