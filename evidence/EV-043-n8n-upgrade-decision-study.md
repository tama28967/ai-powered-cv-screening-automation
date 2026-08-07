---
Initiative: PRJ-0001 — AI-Powered Resume Screening & Recruitment Automation
Decision: TEST: n8n upgrade decision study — no proven fix found; upgrade to 2.33.5 recommended as supported hypothesis; Founder authorization required — Deferred
Classification: Deferred
Artifact: deployment/test-runtime-bindings.md
Date: 2026-08-07
---

**Reason:** D08 round 6 is a study round evaluating whether upgrading the Founder-designated TEST n8n instance (2.31.7) is an evidence-supported remediation for the EV-042 Form Trigger defect. No upgrade was performed. This record separates **external research conclusions** (provenance: public n8n release notes / GitHub, not this instance) from **local runtime facts** (provenance: this instance, directly inspected).

**External research (provenance: n8n official GitHub releases, docs.n8n.io — not runtime evidence about the Founder's instance):**
- `n8n@2.32.7` changelog: one unrelated core fix (security-audit-reporter file-extension import). No form/webhook/trigger content.
- `n8n@2.32.0` (parent minor of 2.32.7): no form/webhook/trigger-registration content in its changelog.
- `n8n@2.33.5` changelog: one unrelated editor fix (markdown toolbar focus). No form/webhook/trigger content in the patch itself.
- `n8n@2.33.0` (parent minor of 2.33.5) changelog **does** touch the relevant subsystem: "Redirect form-resume waiting requests to the form endpoint" (#34725), "Properly handle quotes in Form Trigger Node name" (#34650), plus workflow-lifecycle changes in the same release — new publish/unpublish API endpoints (#34745), deprecation of the activate/deactivate public API endpoints (#34771), a new `workflow.deactivate` external hook (#34746), and "prevent concurrent instance startups from racing database migrations" (#34685).
- No exact GitHub issue or PR was found describing *this* symptom precisely: a Form Trigger that validates clean, reports `active: true`, and still never registers its production route while a plain Webhook on the same instance registers immediately. Adjacent issues exist (#15901 — form served under `/webhook/` instead of `/form/` on n8n 1.93.0, closed with a workaround, not a code fix; #23808 — production webhook 404 on n8n 2.1.5, closed as a support issue, no fix) but neither matches the exact mechanism and neither is in the 2.31–2.33 range.
- **Conclusion: no proven fix.** The 2.33.0 changes are circumstantial support for 2.33.5 being the stronger of the two candidates (real code changed in form-endpoint routing and workflow-activation-lifecycle in that release window) — not confirmation the exact defect is fixed there.

**Local runtime facts (provenance: this instance, directly inspected in round 6, `env:founder-local-n8n`):**
- `n8n --version` executed directly inside the running `n8n-app` container returns **2.31.7**, independently confirming the round-5 audit-report figure by a second, unrelated mechanism.
- Container image: `docker.n8n.io/n8nio/n8n`, **no explicit version tag** — resolves to `:latest` as pulled 2026-07-27T07:26Z, currently pinned in practice only by the untagged image already present locally (digest `sha256:e6bc77c3ace6…`). A future `docker compose pull` would not reliably land on any specific version, including the ones evaluated here — a separate operational observation, not remediated this round.
- Persistence: **SQLite** (`database.sqlite` + WAL files, confirmed present) inside the external named volume `n8n-docker_n8n_data`, mounted at `/home/node/.n8n`, alongside the encrypted credential store, instance `config`, community `nodes/`, and binary `storage/`. `N8N_ENCRYPTION_KEY` and the Cloudflare tunnel token are supplied via `.env` (names only recorded; values never read).
- Compose file: `01. LESSON/06. DOCKER BACKUP/docker-compose.yml` — two services, `n8n` (the target) and `cloudflare-tunnel` (unaffected by an n8n version change), sharing one bridge network. The `n8n_data` volume is `external: true` — it outlives the container and is the correct backup unit.
- **Shared-instance blast radius:** one other workflow exists on this instance, `"My workflow"` (`KkhmpKcxUoVnEwGA`, inactive). Structure-only inspection (no payload/content read) shows 4 LangChain nodes (`@n8n/n8n-nodes-langchain.agent`, two `lmChatGoogleGemini`, one `outputParserStructured`) — no Form Trigger, no Webhook, no Google Drive/Sheets/Gmail node. It is inactive, so an upgrade cannot break a running production path for it; residual risk is limited to whether its LangChain node versions remain compatible if it is ever reactivated, which was not tested (out of scope — it is not ASDP's workflow to mutate or execute).
- Migration risk is **not theoretical** for this instance: 2.33.0's own changelog names a startup/database-migration race as something its authors found worth fixing in this exact version window, corroborating that upgrading SQLite-backed n8n across these versions is a genuine (not merely cautious) migration-risk case.

**Recommendation basis:** given (a) no proven fix, (b) 2.33.5 carries more relevant code motion than 2.32.7 in the exact subsystem, (c) blast radius on the shared instance is low (one inactive, unrelated workflow), and (d) a verifiable backup/rollback unit exists (the named volume) — upgrade to **2.33.5** is a SUPPORTED HYPOTHESIS worth Founder authorization, not a proven fix. This is a Founder infrastructure decision, not an autonomous action.

**Confidence:** Moderate that 2.33.5 is the better of the two candidates; Low that either resolves the exact defect (no direct match found).
**Risk:** none introduced this round — read-only research, read-only local inspection (container exec of `ls`/`n8n --version`, `docker inspect`), no image pulled, no container recreated, no volume touched, no other workflow mutated.
**Industry:** Human Resources
**Client Type:** SMB (11–50 employees)
**Tags:** [D08, round-6, n8n-upgrade, decision-study, form-trigger, founder-decision-pending, provenance:asdp-automated, provenance:external-research, env:founder-local-n8n]
