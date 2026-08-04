# Criteria Config Schema — PRJ-0001

Traces to Engineering Blueprint §4 ("configurable evaluation criteria... adjust without a rebuild") and §8 (Criteria read from a configurable Sheet vs. hardcoded — chosen for the client's explicit requirement). Structural schema only — **contains no actual criteria values**, since the criteria themselves are Open Technical Question 1 (external, client-owned, unresolved).

| Column | Type | Notes |
|---|---|---|
| `config_version` | string | Incremented on any edit — referenced by `evaluation_criteria_version` in the Sheets record schema. |
| `criterion_name` | string | e.g. "years of experience" — **placeholder rows only until the client answers Open Technical Question 1.** |
| `weight` | number | Relative weighting, client-adjustable. |
| `description` | string | What the criterion means, for the client's own future editing. |
| `active` | boolean | Allows disabling a criterion without deleting its history. |

## Placeholder state (as of this D02 run)

This sheet ships with **zero populated criterion rows** — only the column structure above. The AI Evaluation workflow (`w1-ai-evaluation.json`) is built to read from this structure at runtime; it does not hardcode any criterion. Populating real rows is blocked on Open Technical Question 1 and is explicitly not this artifact's job to guess.

## Traceability
- Blueprint source: §4, §8 (Engineering Trade-offs — "Criteria read from a configurable Sheet vs. hardcoded").
- Consumed by: `w1-ai-evaluation.json`.
- Blocks: the *content* of this sheet, not its structure. Structure is READY; content is BLOCKED (external, client-owned).
