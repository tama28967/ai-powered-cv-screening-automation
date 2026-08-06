# PRJ-0001 — TEST Environment Compatibility Evaluation

Target: the Founder-Designated Test Environment ([designation](test-environment-designation.md)). Per `compatibility-contract.md`. Candidate requirements from [candidate-requirements.md](candidate-requirements.md), unchanged.

## Result: COMPATIBLE (node type / typeVersion scope), evidence level 2

**This resolves the exact risk class that motivated ADR-0015** — a generated n8n workflow being structurally valid while incompatible with the actual runtime's node/typeVersion schema. It is now answered with live evidence rather than assumption.

## Why level 3 could not answer it

The provider's bundled node metadata was queried first and honestly reported itself incomplete for every node checked:

> `"Version metadata not populated for this node. Callers must not infer upgrade safety from this response."` — returned for `googleSheets`, `httpRequest`, `code`, and `if`.

This is precisely the limitation ADR-0015's context recorded. Level 3 was therefore insufficient, and LLM knowledge is never authoritative (`target-discovery.md`). Only a live import could settle it.

## Method — live deployment-provider validation (level 2)

All 12 required node types were submitted to the live instance at their candidate typeVersions, across two imports:

| Node type | Candidate typeVersion | Accepted by live instance |
|---|---|---|
| `stickyNote` | 1 | ✅ |
| `executeWorkflowTrigger` | 1 | ✅ |
| `switch` | 3 | ✅ |
| `googleSheets` | 4 | ✅ |
| `formTrigger` | 1 | ✅ |
| `googleSheetsTrigger` | 1 | ✅ |
| `code` | 2 | ✅ |
| `if` | 2 | ✅ |
| `googleDrive` | 3 | ✅ |
| `httpRequest` | 4 | ✅ |
| `executeWorkflow` | 1 | ✅ |
| `gmail` | 2 | ✅ |

The second import was a transient probe (`PRJ-0001 Test — Compatibility Probe`) carrying the eight types not covered by the first. Its persisted structure was read back and verified — all 8 nodes and all 7 connections intact, **no silent coercion, no dropped node, no partial import** — then the probe was deleted. Deletion is the reversal; the environment carries no residue from it.

## Scope of this claim — deliberately narrow

**COMPATIBLE means:** this instance accepts all 12 node types at the typeVersions the candidate declares, and persists them faithfully.

**COMPATIBLE does not mean:** the workflows execute correctly, that their parameters are well-formed (two are not — see below), that credentials resolve, or anything whatsoever about any Production environment. Per `test-environment-governance.md`'s non-promotion rule, **this evidence belongs to this target only.**

## Runtime defects surfaced by live validation — invisible to static testing

Both were found by the live environment and by nothing D03 could have caught. Every one of these artifacts passed D03's static JSON and topology tests.

### D08-DEF-001 — no node carried an `id` (Implementation Defect) — REMEDIATED

The deployment interface rejected all 6 workflows outright: `"expected string, received undefined"` at `nodes[N].id`. Every node in every artifact lacked the `id` field n8n's import format requires.

**Remediated** across all 6 artifacts — 35 nodes given stable, derived ids (`<workflow>-<node-slug>`). Verified: `nodes=35 missing_id=0`. Redeployment after remediation succeeded.

### D08-DEF-002 — `googleSheets` parameter shape rejected (Implementation Defect) — CONFIRMED, NOT REMEDIATED

Runtime validation of the deployed workflow returned, for node `Record error_detail`: `"Expected object but got string"`. The artifact's `matchingColumns: ["record_id"]` and `columns` mapping do not match the shape `googleSheets` v4 expects.

**Not remediated in this run, deliberately.** The correct shape cannot be verified without a real target spreadsheet, and no spreadsheet exists — creating one is blocked on the integration-account question (see the designation record). Remediating a parameter shape that cannot then be executed would produce an untestable fix. The defect is recorded, localized, and left open rather than half-fixed.

## Deployment performed

| Workflow | n8n ID | State |
|---|---|---|
| `PRJ-0001 Test — Error Handling (shared)` | `yrIl76JCaaQloshO` | Deployed, **inactive**, visible in the dashboard |

The remaining five workflows were not deployed: their runtime obligations depend on the two blocked resources, so deploying them would add deployment count without adding testable evidence.

**No workflow was activated. No Production environment was touched.**
