# Mode: risk

Build a design FMEA against the architecture or a program risk register, with every row linked to the requirements, blocks, or elements it threatens. This mode owns `FM-###` and `RSK-###` IDs.

`/systems-engineering risk [fmea | register] [scope: element, block, or whole system]`

## Plan (read-only)

Read `architecture.md`, `requirements.csv`, any existing `risk.csv`, and the user's notes (test failures, supplier issues, schedule concerns). Ask only what changes the result:

- **Format**: `fmea` for failure modes of elements and functions (needs `architecture.md`), or `register` for program, technical, schedule, and cost risks.
- **Scope**: the whole system or a named subtree.
- **Rating scales**: the defaults are below. Use the user's scales if their organization has them.

Proposal:

```
## Risk proposal: <system name> — <fmea | register>

Scope: <elements / blocks / program areas>
Scales: <S/O/D 1–10, or L×C 1–5>
Action threshold: <RPN ≥ 100 or S ≥ 9 (FMEA); score ≥ 12 or C = 5 (register)>
Sources: <...>
Output: <system-slug>-se/risk.csv
```

## Build: FMEA

For each in-scope element and each function it performs (from the allocation table):

1. **Failure mode**: how the function fails (no output, partial, intermittent, unintended, or late).
2. **Effect** at system level and **Severity** (S, 1–10).
3. **Cause** and **Occurrence** (O, 1–10).
4. **Current controls** and **Detection** (D, 1–10, where 10 means it can't be detected before it reaches the user).
5. **RPN** = S × O × D. Rows at or above the threshold get an action, an owner, and a target date (`TBD` if unknown).

## Build: register

1. **Statement**: "If <condition>, then <consequence> to <linked ID>."
2. **Likelihood** (L, 1–5) and **Consequence** (C, 1–5), with **Score** = L × C.
3. **Mitigation** (avoid, mitigate, transfer, or accept), the **trigger** that would show the risk is materializing, and the owner.

## Rating rule

Every rating carries a one-line rationale from the evidence. Without evidence, write `Assumed: <basis>` and show the row's ratings to the user for confirmation before finalizing. Never present unbased ratings as fact.

## Checks

| Check | Rule |
|---|---|
| Linkage | Every row references at least one existing Req ID, block, or element. |
| Statement form | Register rows are If/Then and FMEA rows name mode, effect, and cause separately. |
| Threshold actions | Every row above the threshold has an action and an owner (or `TBD` in Gaps). |
| Rationale | Every rating has a rationale or `Assumed:` basis. |

## Output

`risk.csv` columns:

- **FMEA:** `FM ID | Element | Block | Failure Mode | Effect | S | Cause | O | Controls | D | RPN | Action | Owner | Due | Linked Req IDs | Rationale`
- **Register:** `RSK ID | Statement | L | C | Score | Mitigation | Trigger | Owner | Linked IDs | Rationale`

In the reply, show the rows above the threshold sorted by RPN or Score, plus Gaps. Mitigations that need a new design property go under Candidates for requirements, and new detection steps go under Candidates for verification.
