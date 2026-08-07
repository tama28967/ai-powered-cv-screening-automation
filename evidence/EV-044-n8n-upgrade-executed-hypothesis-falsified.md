---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: n8n upgraded 2.31.7 -> 2.33.5 under Founder authorization; Form Trigger upgrade hypothesis FALSIFIED at runtime — Deferred
Classification: Deferred
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** Founder issued explicit, scoped authorization ("APPROVE TEST n8n upgrade to 2.33.5") following the EV-043 decision study. This record captures the controlled execution and its runtime verdict.

**Pre-upgrade state (independently rediscovered, not reused from round 6's prompt context):**
- Running image: `docker.n8n.io/n8nio/n8n` (untagged), full digest `sha256:e6bc77c3ace603bb66a1cf805756041986f721bfb8e1c1b88721e6f1b9d89c2a` — confirmed identical to round 6's figure.
- Server version: `2.31.7`, confirmed via `docker exec n8n-app n8n --version`.
- Volume: `n8n-docker_n8n_data`, external, mounted at `/home/node/.n8n`, SQLite-backed.
- Compose: `01. LESSON/06. DOCKER BACKUP/docker-compose.yml`, two services (`n8n`, `cloudflare-tunnel`).
- All six PRJ-0001 workflows present and in their expected active/inactive state.

**Backup (hard gate, completed before mutation):** `n8n-app` was stopped (minimum TEST interruption, for SQLite-file consistency — WAL/SHM captured in a stopped-and-quiesced state). A disposable Alpine container tarred the entire `n8n-docker_n8n_data` volume read-only to a host-local, git-ignored path: `01. LESSON/06. DOCKER BACKUP/n8n-backups/n8n_data_pre-2.33.5_upgrade_20260807-143111.tgz` (568KB). **Backup verified, not merely created:** `gzip -t` passed; `tar tzf` listed 13 entries including `database.sqlite`, `database.sqlite-wal`, `database.sqlite-shm`, `config`, `nodes/`, `storage/` — the complete expected persistence structure, nothing truncated. Backup is retained locally and NOT committed to Git.

**Mutation:** the compose file's `n8n` service image line was changed from `docker.n8n.io/n8nio/n8n` to `docker.n8n.io/n8nio/n8n:2.33.5` — the only change made. `docker compose pull n8n` then `docker compose up -d n8n` recreated only the `n8n` service; the volume, `cloudflare-tunnel` service, and network were untouched.

**Startup/migration:** container started cleanly. Log evidence: 14 SQLite migrations ran and each reported "Finished migration ..." with no failure; the instance itself logged `Recorded version change: 2.31.7 -> 2.33.5` and `Version: 2.33.5`; all five previously-active PRJ-0001 workflows were auto-reactivated by n8n on startup (W2, previously inactive, correctly stayed inactive); `/healthz` returned 200. No restart loop, no credential-decryption failure, no fatal error.

**Post-upgrade baseline:** `n8n_list_workflows` shows all six PRJ-0001 workflows present with unchanged IDs; MAIN (`4nDrnkEQ9I9j8mjw`) re-validates at `errorCount: 0`. No workflow disappeared or corrupted. The unrelated `"My workflow"` (`KkhmpKcxUoVnEwGA`) is unchanged (still inactive, same node count) — confirmed untouched.

**Control probe (Webhook, typeVersion 2):** fresh minimal workflow, created and activated via the API on 2.33.5. **Result: HTTP 200 on both `localhost:5678` and `https://n8n.nugi.my.id`, immediately.** Baseline trigger-registration mechanism confirmed healthy on the new version.

**Experiment (Form Trigger, typeVersion 2.2):** fresh minimal workflow, identical create/activate path, tested on both hosts. **Result: HTTP 404 ("Problem loading form") on both `localhost:5678` and `https://n8n.nugi.my.id`, identical to the pre-upgrade round-5 result.**

**Verdict: UPGRADE HYPOTHESIS FALSIFIED FOR 2.33.5.** The upgrade itself succeeded — clean migration, all data and workflows preserved, Webhook registration confirmed healthy — but it did not resolve the Form Trigger route-registration defect. Both probe workflows were deleted after evidence capture; no diagnostic artifact left on the shared instance.

**Rollback:** NOT TRIGGERED. Per the round's explicit rule, "Form Trigger still 404" is not a rollback trigger — the upgrade itself did not fail or regress anything; only the remediation hypothesis was disproven. The instance remains on 2.33.5. The verified backup is retained locally as a rollback artifact regardless, per instruction, until the environment is proven stable over time.

**Consequence:** TC-01 remains BLOCKED — the intended entry point (Form Trigger) still cannot be legitimately invoked. No further version was tried and Form Trigger was not replaced with Webhook Trigger, per explicit instruction; this is a genuine stop condition, not an engineering gap to iterate past this round.

**Confidence:** High — direct, repeated (pre- and post-upgrade), controlled A/B comparison on the same instance.
**Risk:** Low and bounded — one authorized, backed-up, verified TEST-only infrastructure mutation; no Production, client, or unrelated-workflow impact; two disposable probe workflows created and deleted.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, round-7, n8n-upgrade, form-trigger, hypothesis-falsified, infrastructure-mutation, provenance:asdp-automated, env:founder-local-n8n]
