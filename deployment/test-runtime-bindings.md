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
| ASDP root folder (parent) | `1A1TnE1_dHp7etzPYDLOwNdakyUBHllo6` — created 2026-08-06 by ASDP's own Test Environment designation. **A second, unrelated folder also named `ASDP` exists in this Drive** (`1Ra002pg7owfsz6U7ReXB_ZdEDPepYY-a`, predates this project, created 2026-07-17) — always bind by this ID, never re-discover by searching the name `ASDP` (confirmed by direct Drive API lookup, D08 round 8 Founder Q&A). Canonical reference (separate repository): ASDP's `frameworks/delivery-framework/.claude/knowledge/default-test-environment-profile.md`. |
| Project Drive boundary | `ASDP/PRJ-0001/` — `1G2RiX-oCJqpFPs7lyjCLNgZM5F1EjrgD` (child of the ASDP root folder above) |
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
