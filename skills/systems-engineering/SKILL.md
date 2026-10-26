---
name: systems-engineering
description: Systems-engineering workbench built on one shared requirements spine. Modes - requirements (build or audit a Standard Requirements Matrix in the Functional-Physical framework), architecture (functional decomposition / FFBD, physical breakdown, allocation, N-squared interfaces), verification (VCRM and test plan from the matrix), trace (change-impact and coverage gaps), trade (weighted trade study with sensitivity), risk (FMEA or 5x5 risk register), diagrams (Mermaid context, FFBD, physical, interface, and trace diagrams). Artifact-creating modes plan first and wait for approval. Use when the user says "systems engineering", "requirements matrix", "SRM", "decompose requirements", "audit this requirements table", "FFBD", "functional decomposition", "N2 diagram", "VCRM", "verification plan", "impact of changing REQ-", "trade study", "decision matrix", "FMEA", "risk register", or hands over a mission, stakeholder, or ConOps document. Usage - /systems-engineering <mode> [system name or path] [mode options]
---

# Systems Engineering

One skill, seven modes, one shared set of artifacts. Read this file, then read **only** the mode file you need from `modes/`, plus `reference/srm-framework.md` whenever the mode touches requirement rows.

## Modes

| Mode | File | Produces | Gate |
|---|---|---|---|
| requirements | `modes/requirements.md` | `requirements.csv` (create) or findings report (audit) | Yes (create) / No (audit) |
| architecture | `modes/architecture.md` | `architecture.md`, `interfaces.csv` | Yes |
| verification | `modes/verification.md` | `verification.csv` | Yes |
| trace | `modes/trace.md` | Impact or coverage report in the reply | No (read-only) |
| trade | `modes/trade.md` | `trades/TRD-###-<slug>.md` | Yes |
| risk | `modes/risk.md` | `risk.csv` | Yes |
| diagrams | `modes/diagrams.md` | `diagrams.md` (Mermaid) | No, unless it has to derive structure from prose |

Pick the mode from the first argument. If there is none, infer it from the request. If two modes fit, ask with `AskUserQuestion`. A request that spans modes ("decompose this and draw it") runs them in the natural order: architecture → requirements → verification → trace/diagrams. Each gated mode still gets its own approval.

## Shared workspace

Every mode reads and writes one folder per system: `<system-slug>-se/` in the working directory, unless the user names another path. Before asking the user for inputs, read whatever already exists there.

```
<system-slug>-se/
├── requirements.csv     # the spine: Req ID, Parent ID, Text, Type, Functional Block, Allocated Element, Verification Method
├── architecture.md      # function tree (F.X.X), element tree, allocation table
├── interfaces.csv       # IF-### rows from the N-squared matrix
├── verification.csv     # VER-### activities linked to Req IDs
├── risk.csv             # RSK-### or FM-### rows linked to Req IDs, blocks, elements
├── trades/              # TRD-###-<slug>.md
└── diagrams.md          # Mermaid, every node keyed to an ID above
```

### ID conventions (shared across modes)

| Object | Format | Owned by |
|---|---|---|
| Stakeholder / mission parent | `SYS-###` (or the user's scheme) | requirements |
| Functional / physical requirement | `FUNC-###`, `PHYS-###` (or the user's scheme) | requirements |
| Functional block | `F.X.X: Verb Noun` | architecture |
| Physical element | `<Name> (<Part or CI number>)` when a number exists | architecture |
| Interface | `IF-###` | architecture |
| Verification activity | `VER-###` | verification |
| Trade study | `TRD-###` | trade |
| Risk / failure mode | `RSK-###` / `FM-###` | risk |

A mode may only *reference* IDs owned by another mode. It never creates them. If a mode needs a block, element, or requirement that doesn't exist yet, it lists it under **Candidates for <owning mode>** in its output and leaves the artifact alone.

## Shared gate (gated modes)

1. **Plan (read-only).** Read the inputs and workspace. Ask only the questions that change the result (`AskUserQuestion`). Write nothing.
2. **Propose, then stop.** Show the mode's proposal block and ask for approval (approve / change something). If the user changes scope, IDs, or structure, re-issue the proposal.
3. **Build** only after approval.
4. **Validate** with the mode's checks before showing results. Fix what can be fixed and report the rest under **Gaps**.

## Shared rules

- Never invent a number, score, rating, or part. Unknown values are `TBD` (or `Assumed: <basis>` where the mode allows assumptions) and appear in Gaps.
- Never renumber or delete existing IDs when extending an artifact. Continue the numbering.
- Fixed columns stay fixed. Add extra columns only if the user asks, and always after the fixed ones.
- Every row in every artifact traces to at least one ID in another artifact or in a named source document.
- Read-only modes never modify files unless the user asks.
- The skill holds no project data. Everything comes from the user's inputs and the workspace on each run.
- End every run with: what was written (paths), Gaps, Candidates for other modes, and the suggested next mode.
