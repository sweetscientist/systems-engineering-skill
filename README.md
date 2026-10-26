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

```
/systems-engineering architecture docs/conops.md
/systems-engineering requirements "Regenerative braking system"
/systems-engineering requirements audit specs/brake-srm.xlsx
/systems-engineering verification
/systems-engineering trace impact FUNC-002
/systems-engineering trace coverage
/systems-engineering trade "Select caliper housing material" aluminum titanium steel
/systems-engineering risk fmea "Brake Caliper Assembly"
/systems-engineering diagrams all
```

A typical flow runs **architecture → requirements → verification → trace coverage → diagrams**, with **trade** and **risk** wherever a decision or a failure mode needs one. Each run ends by listing what it wrote, open gaps, candidate items for other modes, and the suggested next mode.

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
