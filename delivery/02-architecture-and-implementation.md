# Architecture & Implementation — PRJ-0001

For your technical team or a future maintainer. Every claim below is drawn directly from the actual built artifacts (`implementation/`) and the Engineering Blueprint, not from memory or intention.

## Architecture (source: `engineering-blueprint.md` §3)

A single n8n-orchestrated pipeline, decomposed into modular sub-workflows, split into two independently-triggered workflows at the human review gate — chosen over a single unbroken workflow specifically so a slow reviewer never blocks intake, evaluation, or recording, and over an external microservice split because nothing at this volume (50–300 applicants/month) needs the extra scalability that would cost.

## Components built (source: the actual files in `implementation/workflows/`)

| Component | File | What it does |
|---|---|---|
| Intake, Validation & Storage | `w1-intake-validation-storage.json` | Form intake, file-type validation, Drive upload with structured naming; generates the record identifier used throughout the pipeline |
| Extraction | `w1-extraction.json` | Extracts text content from the stored PDF; designed for text-based PDFs (see Known Limitations for scanned-document coverage) |
| AI Evaluation & Scoring | `w1-ai-evaluation.json` | Reads evaluation criteria from a configurable source at runtime (never hardcoded), calls the evaluation model, parses a structured score + recommendation |
| Duplicate Detection & Recording | `w1-duplicate-detection-recording.json` | Checks incoming applicants against existing records by email, writes the structured Sheets record |
| Human Review Gate & Notification | `w2-human-review-notification.json` | Fires only once a record is marked reviewed — never automatically from the pipeline above; sends the candidate notification |
| Error Handling (shared) | `error-handling.json` | Covers unreadable PDFs, duplicate applications, extraction failures, and evaluation-parsing failures |

## Data schemas (source: `implementation/schemas/`)

- **Sheets record schema** — one row per applicant: identity fields, extraction status, evaluation criteria version (a pointer, never the criteria content itself), score, recommendation, duplicate flag, review status, notification status, error detail.
- **Criteria Config schema** — the structure your team will use to define what makes a candidate score well. **Currently has zero populated rows** — the structure is built and ready; the content is exactly what's needed from you (see Known Limitations).

## Key design decisions worth knowing about

- **The human review gate is architectural, not optional.** No candidate is notified without your team marking the record reviewed first.
- **Evaluation criteria are configuration, not code.** Changing what you score for never requires touching the workflow itself.
- **Duplicate detection uses applicant email** as the matching key — a deliberate engineering choice, not something requiring your input.
- **Error handling covers every named failure mode** the design anticipates (unreadable file, duplicate application, extraction failure, evaluation failure) — each is logged against the applicant's own record, not silently dropped.

## What this document does not claim

Nothing here claims the pipeline has been run against a live environment — it has not (see `04-deployment-status-and-operations.md`).
