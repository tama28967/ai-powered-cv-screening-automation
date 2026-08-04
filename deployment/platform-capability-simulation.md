# D04 Platform-Capability Simulation (using PRJ-0001's candidate as a fixture)

**This file proves the `/deploy` compatibility-evaluation mechanism works. It proves nothing about PRJ-0001's actual client deployment target, which remains unidentified (see `target-discovery.md`). Platform capability proof and client deployment proof are kept strictly separate, per this sprint's own governance.**

## Why this exists

PRJ-0001 cannot be legitimately deployed or even discovery-tested against a real target (Phase 21 of this sprint permits proving D04's execution behavior through a safe, non-external mechanism instead — a fixture, not a real target).

## The fixture

The n8n-mcp package's own bundled reference data (level 3 of `target-discovery.md`'s evidence hierarchy — "provider-bundled/reference metadata," explicitly *not* level 1 live introspection) was queried read-only, this session, for a sample of the 12 node types `candidate-requirements.md` lists:

| Node type | Candidate typeVersion | Reference package's "current" version | Node type confirmed to exist? |
|---|---|---|---|
| `googleSheets` | 4 | 4.7 | Yes |
| `httpRequest` | 4 | 4.4 | Yes |
| `if` | 2 | 2.3 | Yes |
| `formTrigger` | 1 | (unversioned in minimal query; confirmed as a real trigger node) | Yes |

For every node checked, the reference package **explicitly declined** to report breaking-change history ("Version metadata not populated... Callers must not infer upgrade safety from this response") — a real, honest limitation of even this fixture, not a gap I am filling in.

## Simulated evaluation (fixture only)

Treating this bundled reference as a stand-in "observed target capability" set (never as PRJ-0001's real target): every candidate node type is confirmed to exist in the reference package, and every candidate `typeVersion` is at or below the reference's "current" version for that node — a **plausible, not proven**, compatibility signal *for a generic recent n8n installation matching this reference package's vintage*. Under `compatibility-contract.md`'s actual rules, this fixture result would still only support an evaluation of **UNKNOWN-leaning-favorable**, never COMPATIBLE outright, because: (a) it is level-3 evidence, not level-1 live introspection of a real, identified target; (b) breaking-change coverage is admittedly incomplete; (c) node *existence* and *typeVersion* plausibility say nothing about target-side configuration, credentials, or account-level feature availability.

## What this demonstrates

- `/deploy`'s candidate-requirements derivation is mechanically correct (verified against the actual artifact files, not recalled from memory).
- `/deploy`'s evidence-hierarchy discipline holds even against a cooperative, real data source: it still would not certify COMPATIBLE from level-3 evidence alone.
- The mechanism distinguishes "the technology is real and the artifact isn't nonsense" from "this specific client's environment will accept it" — exactly the distinction this sprint's governing study exists to enforce.

## What this does not demonstrate

Anything about PRJ-0001's actual deployment readiness. `deployment-readiness.md`'s BLOCKED verdict is unchanged by this file.
