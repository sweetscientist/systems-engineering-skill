# Mode: trace

Read-only analysis across every artifact in the workspace. Two sub-modes. No gate, and no files are modified.

## impact

`/systems-engineering trace impact <ID> [<ID> ...]`

The user is about to change one or more IDs (a requirement, block, element, or interface). Report everything that change touches.

1. **Up**: walk Parent IDs to the `SYS-` root, since the change may break the parent's intent.
2. **Down**: walk every descendant requirement.
3. **Across**: list every artifact row referencing the changed ID or any descendant, including the block and element in `architecture.md`, `IF-###` rows on those elements, `VER-###` activities, `RSK-###`/`FM-###` rows, and `TRD-###` studies that used the requirement as a criterion.

Output:

`Affected ID | Artifact | Relationship (parent / child / allocated / interface / verifies / risk / trade criterion) | What to re-check`

Follow it with a count by artifact. Each "what to re-check" must name the specific field, e.g. "success criterion copies the old 3.0 s target".

## coverage

`/systems-engineering trace coverage`

Find the traceability holes across the whole workspace:

| Gap | Rule |
|---|---|
| Undecomposed parents | `SYS-` rows with no children. |
| Orphan requirements | Rows whose Parent ID does not resolve. |
| Unrequired functions | `F.X.X` blocks in `architecture.md` with no requirement rows. |
| Unrequired elements | Elements with no allocated requirement. |
| Unverified requirements | Req IDs with no `VER-###`. |
| Uncovered interfaces | `IF-###` rows with no `PHYS-` requirement naming the interface. |
| Dangling references | Any ID referenced in one artifact but missing from its owning artifact. |

Output a findings table (`Gap | ID | Artifact | Suggested fix | Owning mode`), then counts per gap type. If a whole artifact is missing, say which checks were skipped because of it. Don't report them as passing.
