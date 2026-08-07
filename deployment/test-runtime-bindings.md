# PRJ-0001 — TEST Runtime Bindings

The **deployment binding layer**. Implementation artifacts stay environment-neutral; every environment-specific value lives here and is resolved at deploy time.

## Why this file exists

D08 round 3 established that several values the runtime requires are **not** implementation facts: n8n workflow IDs, n8n credential IDs, Google resource IDs, and model bindings. Writing them into `implementation/workflows/*.json` would contaminate provider-neutral source with one environment's identity.

The source therefore carries **`<BIND:token>`** placeholders inside the correct runtime *shape*, and this file maps token → actual TEST value. Deployment resolves them; source never changes between environments.

**One deliberate exception, already recorded:** `googleSheets.documentId` currently carries the TEST spreadsheet ID directly (D08 round 2), because the node requires a resourceLocator value to validate. Each such node carries an explicit `TEST BINDING` note. Migrating it to a `<BIND:>` token is a follow-up, not a new decision.

## Workflow ID bindings

| Token | Target workflow | TEST n8n ID |
|---|---|---|
| `<BIND:error-handling>` | PRJ-0001 - Error Handling (shared) | `yrIl76JCaaQloshO` |
| `<BIND:w1-extraction>` | PRJ-0001 Test - W1 Resume Extraction | `snd72efJIOvHR9cA` |
| `<BIND:w1-ai-evaluation>` | PRJ-0001 Test - W1 AI Evaluation and Scoring | `pCr8bayqGDByE7S8` |
| `<BIND:w1-duplicate-detection-recording>` | PRJ-0001 Test - W1 Duplicate Detection and Recording | `O0oZ4UOg1rdZc77X` |
| *(entry, no caller)* | PRJ-0001 Test - W1 Intake, Validation and Storage (MAIN) | `4nDrnkEQ9I9j8mjw` |
| *(independent trigger)* | PRJ-0001 Test - W2 Human Review Gate and Notification | `jc9ZA8Np8Dpts9Ey` |

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
| Project Drive boundary | `ASDP/PRJ-0001/` — `1G2RiX-oCJqpFPs7lyjCLNgZM5F1EjrgD` |
| Applicant records Sheet | `PRJ-0001 Test - Applicant Records` — `1eiVPbSbJrZSiNa3FaKXgwQCJtxuew5zDeW7Qujgdd_8` |

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
