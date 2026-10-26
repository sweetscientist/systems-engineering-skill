# Mode: verification

Turn the requirements matrix into a Verification Cross-Reference Matrix (VCRM): each requirement gets the activity, level, article, and pass/fail criterion that will close it. This mode owns `VER-###` IDs.

`/systems-engineering verification [path to requirements.csv]`

Requires `requirements.csv`. If it is missing, stop and suggest requirements mode.

## Plan (read-only)

Read `requirements.csv`, `architecture.md`, and any test facility, model, or acceptance information. Ask only what changes the result:

- **Levels**: which integration levels exist (part, subsystem, system, field)?
- **Facilities and models**: what labs, rigs, simulators, or analysis tools are available? Never assume one exists.
- **Phases**: qualification only, or also acceptance on every unit?

Proposal:

```
## Verification proposal: <system name>

Requirements in scope: <count by method: A / D / I / T>
Levels: <...>
Facilities / models: <...>
Grouping: <how rows will be bundled into activities, e.g. one thermal-vac test event covers PHYS-012..019>
Known gaps going in: <rows with TBD targets, Test rows with no facility>
Output: <system-slug>-se/verification.csv
```

## Build

1. **One primary activity per requirement.** Several requirements may share one activity (a test event), but each requirement maps to exactly one primary `VER-###`.
2. **Method follows the matrix.** The activity's method equals the requirement's Verification Method. If it should change, list the row under Candidates for requirements and leave it unchanged here.
3. **Success criterion is the requirement's target**, copied verbatim with units and condition. A `TBD` target makes the criterion `TBD`.
4. **Level.** Verify at the lowest level that is representative of the requirement: a physical row at the element it's allocated to, and a parent functional row at the level that owns the function.
5. **Article and facility.** Name the article (engineering model, qualification unit, flight/production unit, software build) and the facility, model, or tool. Unknown ones are `TBD`.

## Checks

| Check | Rule |
|---|---|
| Coverage | Every Req ID in the matrix has exactly one primary activity. |
| Method match | Activity method equals the matrix's Verification Method. |
| Criterion | Every activity's success criterion is measurable or explicitly `TBD`. |
| Test resources | Every Test activity names a facility or is listed in Gaps. |
| Resolve | Every referenced Req ID exists in `requirements.csv`. |

## Output

`verification.csv` with the columns `VER ID | Req ID(s) | Method | Level | Activity | Success Criterion | Article | Facility / Model | Phase | Status`. Status starts as `Planned`.

In the reply, show a summary table (counts by method × level), the list of activities, Gaps, and Candidates for requirements.
