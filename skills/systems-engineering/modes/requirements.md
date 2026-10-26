# Mode: requirements

Build a Standard Requirements Matrix for a system using the Functional-Physical Requirements Framework (`reference/srm-framework.md`). This mode follows the shared gate: **Plan** is read-only and ends with a proposal, and **Build** writes the matrix only after the user approves it. A matrix built on the wrong system boundary or ID scheme is worthless, so the gate is not optional.

## Sub-modes

- **create** (default): `/systems-engineering requirements <system name or source path>`. Plan, get approval, build the matrix.
- **audit**: `/systems-engineering requirements audit <path>`. Check an existing matrix (CSV, XLSX, or markdown table) against the framework rules and report every violation and gap. No planning phase. Does not modify the file unless asked.

If the user hands over an existing matrix and asks to add to it, treat it as create with the existing file as a source: keep every existing ID, continue the numbering, and never renumber.

## Phase 1 - Plan (read-only)

Write nothing to disk in this phase.

### 1. Gather inputs

Read whatever the user provides or points at, plus the shared workspace. If `architecture.md` exists, its function tree and element tree are the source for the Functional Block and Allocated Element columns. Do not re-derive them. Otherwise look for:

- Mission, stakeholder, or customer needs; ConOps; use cases; top-level system requirements. These become the `SYS-` parents.
- A functional decomposition: Functional Flow Block Diagram (FFBD), SysML activity diagram, function list, or user stories. This becomes the Functional Block column.
- A physical architecture: subsystem list, bill of materials, block diagram, software configuration items, interface control documents. This becomes the Allocated Element column.
- Any existing requirements, standards, or regulatory constraints the system must inherit.
- Verification constraints: what test facilities, models, or acceptance steps exist.

Note what is missing. If there is no functional decomposition, you will have to derive one and the user must approve it. For more than a handful of blocks, recommend running the architecture mode first. If there is no physical architecture, physical requirements cannot be allocated yet and the matrix will carry gaps.

### 2. Ask only the questions that change the matrix

Use `AskUserQuestion`. Do not ask what the inputs already answer. Typical decisive questions:

- **System boundary**: what is inside the system and what is an external interface?
- **Depth**: stop at subsystem allocation, or go down to parts and configuration items?
- **ID scheme**: framework default is `SYS-###` for parents, `FUNC-###` for functional, `PHYS-###` for physical. If the user's documents already use another scheme (for example `REQ-###` with `SYS-M##` parents, as in the framework's own example), adopt theirs.
- **Functional block source**: use their FFBD numbering as-is, or derive `F.X.X` blocks from the mission needs?
- **Output**: markdown only, CSV for Excel, Jama, or DOORS import, or XLSX.

If every question is answerable from the inputs, skip straight to the proposal.

### 3. Propose, then stop

Present this proposal and wait for approval:

```
## Requirements matrix proposal: <system name>

System boundary: <one or two sentences; list external interfaces>
ID scheme: <prefixes and numbering>
Parents (SYS-level): <list of mission/stakeholder requirements that seed the matrix, with IDs>
Functional blocks (F.X.X): <numbered tree, verb + noun each, source noted>
Physical elements: <subsystems / parts / CIs available for allocation; mark "none yet" where unknown>
Verification constraints: <facilities, models, acceptance steps available>
Expected size: <approximate row count, functional vs physical>
Known gaps going in: <what cannot be filled from current inputs>
Output: <files that will be written, with paths>
```

Ask for approval with `AskUserQuestion` (approve / change something). Build only on approval. If the user changes the boundary, ID scheme, or functional blocks, re-issue the proposal before building.

## Phase 2 - Build

### Decomposition procedure

1. **Seed from parents.** Every row traces to a `SYS-` parent or to another row. A row with `Parent ID = None` is allowed only at the top level and must be a stakeholder or mission requirement.
2. **Write functional rows first.** For each functional block, write the functional requirements: what the system shall *do*, with a measurable target. One block may carry several functional rows; every functional row must name exactly one block.
3. **Allocate.** Assign each functional row to the physical element that performs it. If the function is performed by more than one element, do not allocate the functional row to a list. Apply the N-Squared Flow rule: keep the functional row allocated at the level that owns the whole function (the subsystem), then write one child physical requirement per component beneath it.
4. **Write physical rows.** For each allocated element, write the properties it shall *possess*: material, mass, dimensions, interface, environmental tolerance, power, software CI constraints. Parent them to the functional row they support. Reuse the functional block of the parent so the physical constraint stays traceable to the function it serves.
5. **Pick the verification method** per row using the ADIT heuristics in `reference/srm-framework.md`.
6. **Continue numbering** in order of appearance; never leave holes in a prefix series within a single build.

### Writing the Requirement Text

- Imperative sentence, subject named, exactly one `shall`, one requirement per sentence.
- Functional: `The <system or element> shall <verb> <object> <measurable target with units and condition>.`
- Physical: `The <element> shall <be / have / withstand> <property> <value with units and tolerance>.`
- Every mandatory statement carries a measurable target: a number with units, a tolerance or bound, and the condition under which it applies. If the source gives no number, write the row with a `TBD` target, mark it in the gaps report, and do not invent a value.
- Banned words: `and/or`, `etc.`, `as appropriate`, `user-friendly`, `fast`, `robust`, `minimize`, `maximize`, `support`. Replace each with a measurable statement or split the row.
- Never combine a function and a physical constraint in one sentence (Rule 1). If a source sentence does both, split it into a functional parent and a physical child.

## Phase 3 - Validate and deliver

Run every check below before showing the matrix. Fix what can be fixed; report the rest as gaps.

| Check | Rule |
|---|---|
| Unique IDs | Every `Req ID` appears once and matches `[PREFIX]-[NUMBER]`. |
| Parents resolve | Every `Parent ID` is `None` or an ID present in the matrix or in the named source document. |
| Shall test | Every row has exactly one `shall` and a measurable target or an explicit `TBD`. |
| Single type | `Requirement Type` is exactly `Functional` or `Physical`. |
| No mixed sentences | No row states both a new function and a physical constraint (Rule 1). |
| Functional allocation | Every `Functional` row has a `Functional Block` in `F.X.X: Verb Noun` form (Rule 2). |
| Physical allocation | Every row handed to a design team has an `Allocated Element`. Blank cells are listed as architecture gaps (Rule 2). |
| N-Squared | No functional row is allocated to more than one element; multi-element functions have one child physical row per element (Rule 3). |
| ADIT | `Verification Method` is exactly one of `Analysis`, `Demonstration`, `Inspection`, `Test`. |
| Architecture match | If `architecture.md` exists, every Functional Block and Allocated Element appears in it. Unknown ones go under Candidates for architecture. |

### Output

- The matrix as a markdown table in the reply, columns in this exact order and with these exact headers: `Req ID | Parent ID | Requirement Text | Requirement Type | Functional Block (What) | Allocated Element (Where/How) | Verification Method`.
- Below it, a **Gaps** list: every blank Functional Block or Allocated Element, every `TBD` target, every parent that points outside the matrix. State what input would close each gap.
- `requirements.csv` with the same seven headers in the shared workspace (or the path agreed in the proposal). Always write it, because other modes read it. XLSX only on request.
- In audit mode: no matrix, just a findings table (`Req ID | Check failed | Detail | Suggested fix`) followed by the Gaps list.
