# PRJ-0001 — Founder-Designated Test Environment

Per [ADR-0016](../../../ASDP/project-management/adr/0016-founder-designated-test-environment.md) and the Default Test Environment Profile ([ADR-0017](../../../ASDP/project-management/adr/0017-default-test-environment-profile.md)).

## Designation

| Field | Value |
|---|---|
| **Environment Role** | `TEST` |
| **Target identity** | The Founder's local n8n instance — `http://localhost:5678`, published at `https://n8n.nugi.my.id` |
| **Designated for** | PRJ-0001 only |
| **Designated by** | Founder, explicitly, 2026-08-06 |
| **Revocable** | Yes, at any time; autonomy ends immediately on revocation |
| **Profile inherited** | Default Test Environment Profile (ADR-0017) |
| **Deviations** | Two, recorded below — both material |

## Identity confirmation (how "local" and the public URL were established as one instance)

Direct, live evidence, not inference:

1. `http://localhost:5678/healthz` → `{"status":"ok"}`.
2. `localhost:5678/rest/settings` and `https://n8n.nugi.my.id/rest/settings` returned **byte-identical** responses (476 bytes each).
3. The **local** instance's own settings declare its OIDC base URL as `https://n8n.nugi.my.id` — the target self-reporting its public identity.

This is evidence-hierarchy **level 1** (successful current target-runtime introspection). One instance, two addresses: local n8n published through a public hostname.

**Note for blast-radius purposes:** this instance is internet-reachable. A workflow with a form trigger deployed here is publicly addressable. Workflows are created inactive by default and none has been activated.

## Discovery result: PARTIALLY DISCOVERED

| Fact | Value | Source |
|---|---|---|
| Reachability | Confirmed, both addresses | Level 1 |
| n8n runtime version | **UNKNOWN** — not exposed on any unauthenticated endpoint; the management API reported `version: "unknown"` | Level 1 attempted, unresolved |
| Node type / typeVersion support | Established by live import — see Compatibility | Level 2 |
| Deployment interface | n8n public API via the connected management capability; create/update/delete/execute available | Level 1 |
| Credentials present (by name only) | `Gmail account` (gmailOAuth2) · `Google Drive account` (googleDriveOAuth2Api) · `Google Sheets account` (googleSheetsOAuth2Api) · `Google Sheets Trigger account` (googleSheetsTriggerOAuth2Api) | Level 1 |
| LLM / Gemini credential | **NONE CONFIGURED** | Level 1 |

**The MCP package version (2.68.0) is not the n8n runtime version** and was not treated as one.

## Deviations from the Default Profile — both blocking

### Deviation 1 — the integration account is not evidently a dedicated test account

ADR-0017 requires "a **dedicated** Google Workspace test account, holding no personal or operational Founder data," and states the requirement is load-bearing.

All four configured credentials are owned by **`tama28967@gmail.com`** — the Founder's own account address. No separate dedicated test account is configured on this instance.

Consequence, per ADR-0017 §6 and `test-environment-governance.md`'s external-side-effect boundary: autonomy does **not** extend to this account merely because the Founder owns it. In particular the **Gmail send path is not authorized** — sending real email is irreversible and cannot be rolled back. Drive/Sheets resource creation is reversible but the account's purpose is ambiguous, so mutation is not assumed safe. **STOPPED, surfaced to the Founder.**

### Deviation 2 — the default TEST LLM is not available

ADR-0017's default TEST LLM is Gemini API free tier. **No Gemini or other LLM credential exists on this instance.** The AI evaluation path (`w1-ai-evaluation.json`, an `httpRequest` node to a configurable endpoint) therefore cannot execute. **No billing was enabled, no tier upgraded, nothing purchased.**

## Authorized boundary actually exercised

Only n8n-instance mutation: workflow create, read, validate, delete. No Google resource was created or modified. No email was sent. No LLM call was made. No Production environment was touched, discovered, or referenced.
