# PRJ-0001 — Target Discovery

Produced by `/deploy`, per `deployment-standard.md`'s Target Discovery stage and `target-discovery.md`'s evidence hierarchy.

## Result: UNKNOWN

No target has been identified. This is the strictly weaker state than UNREACHABLE — there is no specific target identity to even attempt a connection against.

**Open Technical Question 3** (Engineering Blueprint §9, unresolved since Sprint D01–D02): does the client already have an n8n instance, or must one be provisioned, and where hosted. This has never been answered by the client. No target identity — no URL, no environment name, nothing — has ever been supplied for PRJ-0001's actual deployment target.

## What was explicitly NOT done

This session's working environment has a **separately configured, unrelated n8n MCP connection** (`https://n8n.nugi.my.id`, discovered during the prior read-only compatibility study). **This is not PRJ-0001's client target.** It was not treated as one, not queried as one, and is not referenced anywhere in this file's actual discovery result. Using an incidentally-available connection as a stand-in for an unidentified client target would be exactly the fabrication this sprint's governance forbids (`target-discovery.md`: "an available connection to *some* instance is never treated as discovery of *the* target unless that instance is actually confirmed as the Project's intended target"). It is not so confirmed.

## Evidence hierarchy applied

| Level | Available for PRJ-0001's actual target? |
|---|---|
| 1. Live introspection of the identified target | No — no target identified |
| 2. Live deployment-provider validation | No — same reason |
| 3. Provider-bundled reference metadata | Not applicable to a specific target — see `platform-capability-simulation.md` for how this level is used for a *platform-capability* proof, explicitly not a PRJ-0001 target claim |
| 4. Official documentation | Not target-specific |
| 5. Implementation artifact assumptions | Recorded in `candidate-requirements.md`; assumptions about the artifact, not the target |
| 6. LLM/training knowledge | Explicitly excluded as authoritative, per `target-discovery.md` and ADR-0015 |

## Consequence

Per `compatibility-contract.md`, with no Observed Target Capabilities, Compatibility Evaluation can only be UNKNOWN — see `compatibility-evaluation.md`.
