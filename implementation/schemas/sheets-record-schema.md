# Sheets Record Schema — PRJ-0001

Traces to Engineering Blueprint §4 ("writes one structured record...") and §6 sequencing item 2. One row per applicant.

| Column | Type | Notes |
|---|---|---|
| `record_id` | string | Generated at recording time. |
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

## Traceability
- Blueprint source: §4 System Decomposition (Recording), §9 (dedup key assumption).
- Consumed by: `w1-duplicate-detection-recording.json`, `w2-human-review-notification.json`, `error-handling.json`.
- Open Technical Questions this schema does **not** resolve: none — this schema is structural only and holds regardless of the three external inputs' eventual answers.
