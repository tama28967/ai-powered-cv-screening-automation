# Deployment Status & Operations — PRJ-0001

Source: `deployment/target-discovery.md`, `deployment/compatibility-evaluation.md`, `deployment/deployment-readiness.md`.

## Current status: NOT DEPLOYED

**No deployment has occurred.** No live n8n instance, Google Workspace connection, or any other system has been modified as part of this project. Everything built exists as reviewed, tested definition files, ready to be deployed once the items below are resolved.

## Why deployment hasn't happened

Deployment requires knowing the actual system it will run on — its version, its configuration, what it actually supports. **That system has not yet been identified.** This isn't a missing technical step on our side; it's information only you can provide (see "n8n target environment" in `05-known-limitations-and-client-actions.md`).

Because the target is unknown, compatibility with it is also — honestly — **unknown**. Not "assumed fine," not "probably fine": unknown, because there is nothing yet to check it against. Deployment does not proceed on an assumption here.

## What deployment will require, once a target exists

- Access to your n8n instance (or a decision to provision one)
- Google Workspace API access (Drive, Sheets, Gmail) — the specific scopes needed, not any specific credential value recorded in this package
- An evaluation-model (LLM) API key
- A verification pass once connected, confirming the specific node versions this solution uses are supported by your instance

**No credential values are stored anywhere in this package or this project's records.** Only the *names* of what will eventually be needed are listed above.

## Operational flow, once deployed

1. Applicant submits the form → validated → stored in Drive.
2. Content extracted → evaluated against your configured criteria → scored.
3. Duplicate check → structured record written to Sheets, status "pending review."
4. **Your team reviews the record and marks it reviewed** — this step is manual and yours, by design.
5. Candidate notification sent automatically only after that review.
6. Any failure at any step is logged against that applicant's own record, never silently dropped.

## Recovery / rollback

Once a real target exists, any future deployment action will follow this platform's own governed rollback discipline: a real, verified pre-deployment state capture, and a defined rollback procedure — before any deployment is attempted, not improvised afterward.
