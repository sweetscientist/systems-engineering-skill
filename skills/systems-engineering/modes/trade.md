# Mode: trade

Run a weighted trade study between design alternatives. Criteria trace to requirements, and the result is tested for sensitivity before it is recommended. This mode owns `TRD-###` IDs.

`/systems-engineering trade "<decision>" [alternatives] [path to evidence]`

## Plan (read-only)

Read the decision, the alternatives, any evidence the user provides (datasheets, quotes, test data), and `requirements.csv`. Ask only what changes the result:

- **Decision statement**: one sentence, "Select <what> for <element / function>".
- **Alternatives**: at least two, plus the current baseline if there is one.
- **Criteria and weights**: the user approves the weights; never set them silently.

Proposal:

```
## Trade proposal: TRD-### <decision>

Alternatives: <A, B, C (baseline)>
Must-meet screens: <shall requirements that are pass/fail, with Req IDs>
Scored criteria: <criterion — weight — linked Req ID or MOE — how measured>
Weights total: 100
Scale: 1 / 3 / 9 (default; the user may pick 1–5)
Evidence available: <per alternative>
Output: <system-slug>-se/trades/TRD-###-<slug>.md
```

## Build

1. **Screen.** Check each alternative against every must-meet requirement. Any alternative that fails is eliminated, and the failing Req ID is recorded.
2. **Score.** Score each surviving alternative on each criterion. Every score cites its evidence or is marked `Assumed: <basis>`. A criterion with no evidence for an alternative is scored `TBD` and excluded from that alternative's total, and the exclusion is stated.
3. **Total.** Weighted sum per alternative, plus the normalized score (total ÷ maximum possible).
4. **Sensitivity.** Shift each weight by ±10 points, redistributing proportionally across the others, and record whether the winner changes. If the top two are within 10% or any single shift flips the winner, call the result **too close to call** and name the criterion that decides it.

## Checks

| Check | Rule |
|---|---|
| Traceability | Every criterion links to a Req ID, MOE, or named stakeholder concern. |
| Weights | Sum to 100 and were approved by the user. |
| Evidence | Every score has a citation or `Assumed:` basis. |
| Independence | No two criteria measure the same thing (e.g., mass and "weight"). If they do, merge them. |

## Output

`trades/TRD-###-<slug>.md` contains:

- the decision statement
- the screen table
- the scoring matrix (`Criterion | Weight | Linked ID | A | B | C`, each cell `score (evidence)`)
- totals
- the sensitivity table
- a recommendation that names the 1–2 criteria driving it
- Assumptions and Gaps

List the selected alternative's new constraints under Candidates for requirements.
