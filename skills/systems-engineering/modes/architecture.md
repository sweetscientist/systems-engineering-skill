# Mode: architecture

Produce the functional and physical architecture the requirements matrix allocates against: a function tree (FFBD), an element tree, a function-to-element allocation, and an N-squared interface matrix. This mode owns `F.X.X` blocks, element names, and `IF-###` IDs.

`/systems-engineering architecture <system name or source path> [depth: subsystem | part]`

## Plan (read-only)

Read the mission needs, ConOps, use cases, block diagrams, BOMs, ICDs, and any existing `architecture.md` or `requirements.csv`. Ask only what changes the result:

- **System boundary** and the list of external actors and systems.
- **Depth**: functions and elements down to subsystem, or to part / CI.
- **Source of truth**: adopt the user's existing FFBD or BOM numbering as-is, or derive it.

Proposal:

```
## Architecture proposal: <system name>

Boundary: <inside vs. external actors/systems>
Top-level functions: <F.1 … F.n, verb + noun, source noted>
Top-level elements: <subsystems / CIs, source noted; "none yet" where unknown>
Depth: <subsystem | part>
Known gaps going in: <...>
Output: <system-slug>-se/architecture.md, interfaces.csv
```

## Build

1. **Function tree.** Decompose top-level functions into `F.X.X: Verb Noun` blocks. Keep 2–7 children per parent. A child must be part of its parent's function, not a separate one. Record each block's inputs and outputs (items, energy, data) and its sequence in the FFBD (series, AND-parallel, OR-alternative). Stop at the agreed depth.
2. **Element tree.** List the physical elements: hardware, software CIs, operators, and facilities. Give each a type and a parent element. Use the user's part or CI numbers when they exist and never invent them.
3. **Allocation.** Assign each leaf function to exactly one owning element. If a function needs several elements, allocate it to the lowest element that contains all of them and note it under **N-squared splits**, so requirements mode writes one physical child per contributing element.
4. **N-squared matrix.** Put the elements on the diagonal. Each off-diagonal cell (row → column) holds the interfaces where the row element sends something to the column element. Number each one `IF-###`, with type (mechanical, electrical, data, thermal, fluid, human) and the item that crosses it. External actors get their own row and column, labeled External.

## Checks

| Check | Rule |
|---|---|
| Naming | Every block is `F.X.X: Verb Noun`; numbering has no holes. |
| Allocation | Every leaf function has exactly one owning element. |
| Orphan elements | Every element performs at least one function, or is flagged as having no function. |
| Flow closure | Every function output is consumed by another function or crosses the boundary through an interface. |
| Interface ends | Every `IF-###` has a valid From and To (element or External). |
| No inventions | Every element and part number comes from a source or is marked `Proposed` for the user to confirm. |

## Output

`architecture.md` with three tables:

- `Block | Name | Parent | Inputs | Outputs | Sequence | Source`
- `Element | Type (HW / SW CI / Operator / Facility / External) | Parent | Part or CI No. | Source`
- `Block | Owning Element | N-squared split? | Contributing Elements`

Then the N-squared matrix as a markdown table.

`interfaces.csv` with the columns `IF ID | From | To | Type | Item Crossing | ICD Ref | Source`.

In the reply, list Gaps and Candidates for requirements, i.e. interfaces and N-squared splits that need `PHYS-` rows. Suggest diagrams mode next.
