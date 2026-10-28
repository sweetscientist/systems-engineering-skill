# Systems Engineering Skill for Claude Code

A [Claude Code](https://claude.com/claude-code) skill that does structured systems-engineering work. It builds requirements, architecture, verification, traceability, trade studies, risk analysis, and diagrams, all on one shared, traceable set of artifacts.

Every artifact is linked by ID, so a change in one place can be traced through the others. Modes that create artifacts work like plan mode: they read your inputs, ask only the questions that change the result, propose a structure, and write nothing until you approve it.

## Modes

| Mode | What it does | Writes |
|---|---|---|
| `requirements` | Builds or audits a Standard Requirements Matrix in the Functional-Physical framework (seven columns, strict parent/child traceability, ADIT verification) | `requirements.csv` |
| `architecture` | Function tree (FFBD), physical element tree, function-to-element allocation, N² interface matrix | `architecture.md`, `interfaces.csv` |
| `verification` | Verification Cross-Reference Matrix: one activity per requirement, with the success criterion copied from the requirement's target | `verification.csv` |
| `trace` | `impact <ID>` lists everything a change touches; `coverage` finds traceability gaps across all artifacts | Report only (read-only) |
| `trade` | Weighted trade study: must-meet screen, cited scores, ±10-point weight sensitivity | `trades/TRD-###-<slug>.md` |
| `risk` | Design FMEA (S × O × D → RPN) or a 5×5 If/Then risk register, each row linked to requirements, functions, or elements | `risk.csv` |
| `diagrams` | Mermaid context, FFBD, physical, interface, and requirement-trace diagrams, every node keyed to an artifact ID | `diagrams.md` |

## Install

**As a plugin (recommended):**

```
/plugin marketplace add sweetscientist/systems-engineering-skill
/plugin install systems-engineering@systems-engineering-skill
```

Plugin skills are namespaced, so invoke it as `/systems-engineering:systems-engineering <mode> ...`. You can also just describe the task ("build a requirements matrix for…") and Claude will pick the skill up.

**As a plain skill:**

```bash
git clone https://github.com/sweetscientist/systems-engineering-skill.git
cp -R systems-engineering-skill/skills/systems-engineering ~/.claude/skills/
```

Then invoke it as `/systems-engineering <mode> ...`. To scope it to a single project, copy it into `<project>/.claude/skills/` instead.

 ## Usage

  The two features most people use are **building a requirements matrix** and **drawing diagrams from it**. Both work from a short brief, existing documents, or a
  plain-language request. Commands that create files show a proposal first and write nothing until you approve it.

  ### Build a requirements matrix

  **Quick draft.** Point it at a system name or your source documents:

  ```
  /systems-engineering:design:requirements "Regenerative braking system"
  /systems-engineering:design:requirements docs/brake-brief.md docs/brake-bom.xlsx
  ```

  Or just ask: *"Build a requirements matrix for our braking system from docs/brake-brief.md."*

  What happens:

  1. **It reads** your inputs and any existing `<system>-se/` workspace.
  2. **It asks** only the questions that change the result: system boundary, depth (subsystem or part), and ID scheme. If your documents already use `REQ-###`, it
  keeps that scheme.
  3. **It proposes, then stops:**

     ```
     ## Requirements matrix proposal: Regenerative braking system

     System boundary: pedal input to rotor clamping; driver and wheel/rotor are external
     ID scheme: REQ-### (adopted from brake-brief.md)
     Parents (SYS-level): SYS-M01 stop from 60 mph in under 3.0 s (brief §2)
     Functional blocks / elements: F.1 Arrest Kinetic Energy, F.1.1, F.1.2 from brake-brief.md;
                                   elements from brake-bom.xlsx
     Key driving requirements: REQ-001 stopping time
     Verification constraints: dynamometer available; no vehicle track time yet
     Expected size: ~15 rows (8 functional, 7 physical)
     Known gaps going in: pad wear life, operating temperature range
     Output: regenerative-braking-system-se/requirements.csv, rationale.csv, measures.csv
     ```

  4. **After you approve,** it writes the matrix, validates it, and lists the gaps:

  | Req ID | Parent ID | Requirement Text | Type | Functional Block | Allocated Element | Verification |
  |---|---|---|---|---|---|---|
  | REQ-001 | SYS-M01 | The system shall decelerate the vehicle from 60 mph to 0 mph in less than 3.0 seconds. | Functional | F.1: Arrest Kinetic Energy | Braking
  Subsystem | Test |
  | REQ-002 | REQ-001 | The system shall clamp the rotor with a force of at least 8,000 Newtons within 150 milliseconds of pedal actuation. | Functional | F.1.1:
  Apply Clamping Force | Brake Caliper Assembly | Test |
  | REQ-003 | REQ-002 | The caliper housing shall be constructed from aluminum alloy. | Physical | F.1.1: Apply Clamping Force | Brake Caliper Housing (Part 201) |
  Inspection |
  | REQ-004 | REQ-001 | The brake pads shall maintain a coefficient of friction ≥ 0.40 at operational temperatures up to 500 °C. | Physical | F.1.2: Generate
  Friction | Friction Pads (Part 202) | Analysis |

  ```
  Gaps
  - REQ-009 pad wear life: target TBD (brief gives none). Needs: supplier data or a customer goal.
  - REQ-012 operating temperature range: TBD. Needs: the vehicle environment spec.
  ```

  Validation covers the framework rules, the NASA SEH six-step validation, and the Appendix C checklists: one `shall` per row, a measurable target or explicit `TBD`,
  one verification method per row, and no function and physical constraint in the same sentence. It never invents a number. Unknown values stay `TBD` and appear
  under Gaps.

  **Fully traced matrix.** For more than a handful of functions, build the architecture first so every row gets a real block and element:

  ```
  /systems-engineering:design:requirements docs/brake-brief.md    # top-level rows from the brief
  /systems-engineering:design:decomposition                        # F.X.X function tree (FFBD)
  /systems-engineering:design:solution                             # physical element tree
  /systems-engineering:design:requirements allocate                # fill blocks and elements, add child rows
  ```

  **Audit a matrix you already have.** This is read-only and changes nothing:
  
  ```
  /systems-engineering:design:requirements audit specs/brake-srm.xlsx
  ```

  Illustrative findings:
  
  | Req ID | Check failed | Detail | Suggested fix |
  |---|---|---|---|
  | REQ-014 | Shall test | Two `shall`s in one row | Split into two rows; the new row continues the numbering |
  | REQ-021 | Verifiable (App C) | "shall respond quickly" | State a response time with units, or `TBD` with an owner |
  | REQ-030 | N-squared | One functional row allocated to both Caliper Assembly and ABS Module | Keep the functional row at Braking Subsystem; add one physical child
  per element |

  `requirements.csv` is always written. Ask for XLSX or Markdown if you need either.

  ### Draw diagrams

  Diagrams are rendered from the workspace as Mermaid. Every node is an artifact ID, so the diagrams never show structure that isn't in your data.

  ```
  /systems-engineering:model:diagrams all              # every diagram whose source exists
  /systems-engineering:model:diagrams ffbd             # one view
  /systems-engineering:model:diagrams trace REQ-001    # one requirement's full trace
  ```

  | View | Shows | Needs first |
  |---|---|---|
  | `context` | System boundary, external actors, boundary interfaces | design:solution, management:interfaces |
  | `ffbd` | Function flow (FFBD) with sequence and AND/OR junctions | design:decomposition |
  | `physical` | Element tree (PBS) | design:solution |
  | `interfaces` | Element-to-element interfaces, labeled with IF IDs | management:interfaces |
  | `trace <ID>` | Parents, children, verification, and risk links for one requirement | design:requirements |
  | `ngo` | Stakeholders → needs → MOEs → system requirements | design:stakeholders, design:requirements |
  | `stpa` / `fta <FT-###>` / `timeline` | Control structure, fault tree, review timeline | specialty:stpa, specialty:fta, management:planning |

  If a source is missing, `all` skips that view and names the command that produces it.

  Output goes to `<system>-se/diagrams.md`. GitHub renders Mermaid natively, so the file displays as diagrams in the repo. To edit one, paste the block into
  [mermaid.live](https://mermaid.live). To export SVG or PNG, use [mermaid-cli](https://github.com/mermaid-js/mermaid-cli).

  **Example: `ffbd`**

  ```mermaid
  flowchart LR
    subgraph F1["F.1: Arrest Kinetic Energy"]
      direction LR
      F1_1["F.1.1: Apply Clamping Force"] --> F1_2["F.1.2: Generate Friction"]
    end
    IN([Pedal actuation]) --> F1_1
    F1_2 --> OUT([Vehicle stopped])
  ```

  **Example: `physical`**

  ```mermaid
  flowchart TD
    BS["Braking Subsystem"] --> BCA["Brake Caliper Assembly"]
    BCA --> P201["Brake Caliper Housing (Part 201)"]
    BCA --> P202["Friction Pads (Part 202)"]
  ```

  **Example: `trace REQ-001`**

  ```mermaid
  flowchart TD
    SYS_M01["SYS-M01: Stop from 60 mph"] --> REQ_001["REQ-001: 60→0 mph in < 3.0 s"]
    REQ_001 --> REQ_002["REQ-002: Clamp ≥ 8,000 N within 150 ms"]
    REQ_001 --> REQ_004["REQ-004: Pad friction ≥ 0.40 to 500 °C"]
    REQ_002 --> REQ_003["REQ-003: Housing is aluminum alloy"]
    REQ_001 -. verified by .-> VER_001["VER-001: Vehicle stopping test"]
    REQ_002 -. verified by .-> VER_002["VER-002: Clamp force dyno test"]
  ```

  ### Other workflows
  
  | Task | Command |
  |---|---|
  | What breaks if I change a requirement? | `/systems-engineering:management:requirements impact REQ-002` |
  | Find traceability gaps across everything | `/systems-engineering:management:requirements coverage` |
  | Verification matrix (VCRM) from the requirements | `/systems-engineering:realization:verification` |
  | Trade study | `/systems-engineering:management:decision "Select caliper housing material" aluminum titanium steel` |
  | FMEA on an assembly | `/systems-engineering:specialty:fmea "Brake Caliper Assembly"` |
  | Is the project ready for PDR? | `/systems-engineering:lifecycle:reviews PDR` |
  | Stakeholders and ConOps from a mission brief | `/systems-engineering:design:stakeholders brief.md`, then `design:conops` |

## Shared workspace

All modes read and write one folder per system, so each mode builds on the others' output:

```
<system-slug>-se/
├── requirements.csv     # the spine
├── architecture.md      # F.X.X function tree, element tree, allocation
├── interfaces.csv       # IF-### from the N² matrix
├── verification.csv     # VER-### linked to Req IDs
├── risk.csv             # RSK-### / FM-### linked to Req IDs, blocks, elements
├── trades/              # TRD-###-<slug>.md
└── diagrams.md          # Mermaid views of the above
```

| Object | ID format | Owned by |
|---|---|---|
| Stakeholder / mission requirement | `SYS-###` | requirements |
| Functional / physical requirement | `FUNC-###` / `PHYS-###` | requirements |
| Functional block | `F.X.X: Verb Noun` | architecture |
| Interface | `IF-###` | architecture |
| Verification activity | `VER-###` | verification |
| Trade study | `TRD-###` | trade |
| Risk / failure mode | `RSK-###` / `FM-###` | risk |

If your documents already use a different scheme (for example `REQ-###`), the skill adopts yours. A mode only creates its own IDs. When it needs something another mode owns, it lists it as a *candidate* for that mode and doesn't write it.

## The requirements matrix

The matrix has seven fixed columns:

`Req ID | Parent ID | Requirement Text | Requirement Type | Functional Block (What) | Allocated Element (Where/How) | Verification Method`

| Req ID | Parent ID | Requirement Text | Type | Functional Block | Allocated Element | Verification |
|---|---|---|---|---|---|---|
| REQ-001 | SYS-M01 | The system shall decelerate the vehicle from 60 mph to 0 mph in less than 3.0 seconds. | Functional | F.1: Arrest Kinetic Energy | Braking Subsystem | Test |
| REQ-002 | REQ-001 | The system shall clamp the rotor with a force of at least 8,000 Newtons within 150 milliseconds of pedal actuation. | Functional | F.1.1: Apply Clamping Force | Brake Caliper Assembly | Test |
| REQ-003 | REQ-002 | The caliper housing shall be constructed from aluminum alloy with a mass not exceeding 2.5 kg. | Physical | F.1.1: Apply Clamping Force | Brake Caliper Housing (Part 201) | Inspection |

Three rules are enforced:

1. **The "shall" test.** A functional row says what the system *does*, and a physical row says what an element *is* or *has*. A single sentence never does both.
2. **Allocation enforcement.** Every functional row has a functional block, and every row handed to a design team has an allocated element. A blank cell is reported as an architecture gap.
3. **N-squared flow.** A function performed by several components is split into one child physical requirement per component.

The full framework is in [`skills/systems-engineering/reference/srm-framework.md`](skills/systems-engineering/reference/srm-framework.md).

## Design principles

- **Plan before writing.** Requirements, architecture, verification, trade, and risk all propose and wait for approval. A matrix built on the wrong system boundary is worthless.
- **Never invent numbers.** Missing targets, scores, ratings, and part numbers become `TBD` or `Assumed: <basis>` and are listed under Gaps.
- **Never renumber.** Extending an artifact keeps every existing ID and continues the numbering.
- **Diagrams are views, not sources.** Every Mermaid node maps to an artifact ID, and the skill won't draw structure it can't trace.
- **No project data in the skill.** Everything comes from your inputs and the workspace on each run.

## Layout

```
skills/systems-engineering/
├── SKILL.md                 # router: modes, workspace, ID conventions, shared gate and rules
├── modes/                   # one file per mode, loaded only when that mode runs
│   ├── requirements.md
│   ├── architecture.md
│   ├── verification.md
│   ├── trace.md
│   ├── trade.md
│   ├── risk.md
│   └── diagrams.md
└── reference/
    └── srm-framework.md     # Functional-Physical framework, three rules, ADIT guide
```

To add a mode, create `modes/<mode>.md` and add a row to the mode table in `SKILL.md`. If it creates new IDs, add a row to the ID conventions table too.

## Limitations

- This is not a requirements-management tool. It produces CSV and Markdown files you can import into Excel, Jama, DOORS, or similar tools, and it doesn't sync with them.
- Diagrams are Mermaid only, not SysML models.
- The output is a draft for engineering review. The skill enforces structure and traceability, but it can't tell whether a requirement is the right one.

## License

[MIT](LICENSE)
