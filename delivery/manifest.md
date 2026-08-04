# Delivery Manifest — PRJ-0001

Produced by `/deliver`, per `delivery-manifest-contract.md`. Records what is delivered, its ownership, and its status — never duplicates artifact content. Assembled 2026-08-04, against Project state as of ASDP commit `bb615e2` / PRJ-0001 commit `9a195f6`.

## Package-included documents (this delivery)

| Artifact | Category | Purpose | Source/ownership | Status | Inclusion reason |
|---|---|---|---|---|---|
| `delivery/01-project-overview.md` | Documentation — Business | Orient any reader; index the package | Derived from Briefcase §2–6 | IMPLEMENTED | Required — every delivery needs an entry point |
| `delivery/02-architecture-and-implementation.md` | Documentation — Technical | Describe what was built | Derived from Blueprint §3–4, `implementation/` (inspected directly) | IMPLEMENTED | Required — maintainer needs this |
| `delivery/03-testing-and-validation-summary.md` | Documentation — Technical/Business | Report what's tested and validated | Derived from `test-results.md`, `validation-report.md` | STATICALLY TESTED / PARTIALLY VALIDATED | Required — claim-safety demands this be explicit |
| `delivery/04-deployment-status-and-operations.md` | Documentation — Operator/Business | Report deployment status and future operations | Derived from `deployment/target-discovery.md`, `deployment/compatibility-evaluation.md`, `deployment/deployment-readiness.md` | BLOCKED (deployment); UNKNOWN (compatibility) | Required — mandatory per `delivery-package-contract.md`, blockers present |
| `delivery/05-known-limitations-and-client-actions.md` | Documentation — Business | Name every unresolved client input | Derived from Blueprint §9 Open Technical Questions | BLOCKED (3 items) | **Mandatory** — blockers exist |

## Referenced-in-place (not duplicated into `delivery/`, per Single Source of Truth)

| Artifact | Category | Status |
|---|---|---|
| `implementation/schemas/sheets-record-schema.md`, `criteria-config-schema.md` | Implementation — data schema | IMPLEMENTED, STATICALLY TESTED |
| `implementation/workflows/*.json` (6 files) | Implementation — n8n workflow definitions | IMPLEMENTED, STATICALLY TESTED |
| `implementation/test-plan.md`, `test-results.md` | Testing record | STATICALLY TESTED (12 obligations) |
| `implementation/validation-report.md` | Validation record | PARTIALLY VALIDATED |
| `engineering-blueprint.md` | Architecture/design authority | IMPLEMENTED (source for all of the above) |
| `briefcase.md` | Commercial/business authority | ACCEPTED (immutable CF→Project handoff) |

## Explicitly excluded from this package

| What | Why excluded |
|---|---|
| `evidence/` (21 Evidence Records) | Internal ASDP delivery-process evidence — audit trail for the practice, not client-facing content, per `delivery-package-contract.md` |
| `deployment/candidate-requirements.md`, `compatibility-evaluation.md`, `target-discovery.md`, `platform-capability-simulation.md` | Internal reasoning artifacts; their conclusions are summarized in `04-deployment-status-and-operations.md` in client-appropriate language — the raw internal records are not themselves client deliverables |
| Any ASDP framework file (`.claude/` contents, ADRs, DF/CEF internals) | Internal platform machinery, never part of any Delivery Package |
| Any credential or secret value | Never persisted anywhere in this project's records — none exist to exclude |

## Package state

**PRE-DEPLOYMENT** — see `delivery-readiness.md` for the full readiness evaluation.
