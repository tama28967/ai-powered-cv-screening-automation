# PRJ-0001 — Candidate Requirements

Produced by `/deploy`, per `deployment-standard.md`'s Dry Run stage and `compatibility-contract.md`. Derived directly from the actual six workflow-definition artifacts (`implementation/workflows/*.json`), verified by grep, not recalled from memory.

## Distinct node types the candidate requires

| Node type | typeVersion used | Used in |
|---|---|---|
| `n8n-nodes-base.stickyNote` | 1 | all 6 workflows (traceability only — not a runtime dependency) |
| `n8n-nodes-base.formTrigger` | 1 | w1-intake-validation-storage |
| `n8n-nodes-base.executeWorkflowTrigger` | 1 | w1-extraction, w1-ai-evaluation, w1-duplicate-detection-recording, error-handling |
| `n8n-nodes-base.executeWorkflow` | 1 | w1-intake-validation-storage, w1-extraction, w1-ai-evaluation, w1-duplicate-detection-recording |
| `n8n-nodes-base.if` | 2 | w1-intake-validation-storage, w1-extraction, w1-ai-evaluation, w1-duplicate-detection-recording, w2-human-review-notification |
| `n8n-nodes-base.code` | 2 | w1-intake-validation-storage, w1-extraction, w1-ai-evaluation |
| `n8n-nodes-base.googleDrive` | 3 | w1-intake-validation-storage |
| `n8n-nodes-base.googleSheets` | 4 | w1-ai-evaluation, w1-duplicate-detection-recording, w2-human-review-notification, error-handling |
| `n8n-nodes-base.googleSheetsTrigger` | 1 | w2-human-review-notification |
| `n8n-nodes-base.httpRequest` | 4 | w1-ai-evaluation |
| `n8n-nodes-base.switch` | 3 | error-handling |
| `n8n-nodes-base.gmail` | 2 | w2-human-review-notification |

## Requirement statement (capability/constraint semantics, per `compatibility-contract.md`)

- **Runtime capability required:** an n8n instance supporting sub-workflow execution (`executeWorkflow`/`executeWorkflowTrigger`), the 12 node types above, and Google Workspace + Gmail node integrations.
- **Version constraint:** each node type must be supported **at or compatible with** the `typeVersion` this candidate specifies (table above) — n8n's own versioning is per-node, not a single platform-wide semver number; a target's "n8n version" alone does not resolve this, each node type must be checked individually.
- **No version constraint is asserted beyond what the artifacts actually declare.** No claim is made here about whether these typeVersions are "the latest" or "the safest" — that is a target-specific compatibility question, not a candidate requirement.

## What this file deliberately does not claim

Whether any actual target satisfies these requirements. That is `target-discovery.md` and `compatibility-evaluation.md`'s question, not this one's.
