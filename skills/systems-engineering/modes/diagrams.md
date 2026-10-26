# Mode: diagrams

Render the workspace artifacts as Mermaid diagrams. Diagrams are views of the data: every node and edge comes from an ID in an artifact, and the diagram adds no structure of its own.

`/systems-engineering diagrams [context | ffbd | physical | interfaces | trace <ID> | all]`

## Sources

| Diagram | Mermaid type | Source |
|---|---|---|
| context | `flowchart LR` | System boundary + external actors and the `IF-###` rows that touch them (`architecture.md`, `interfaces.csv`) |
| ffbd | `flowchart LR` | Function tree and the sequence column; AND / OR junctions as small labeled nodes |
| physical | `flowchart TD` | Element tree |
| interfaces | `flowchart LR` | `interfaces.csv`; edge label = `IF-### item` |
| trace `<ID>` | `flowchart TD` | Parent chain and descendants of `<ID>` from `requirements.csv`, plus its `VER-###` and `RSK-###` links |

If the needed artifact is missing, say which mode produces it. If the user only has prose (no workspace), derive the structure with the architecture mode's plan and approval gate first. Never draw guessed structure.

## Rules

- The node ID is the artifact ID made Mermaid-safe (`F.1.2` → `F1_2`). The node label is `ID: Name`.
- Keep each diagram to 40 nodes or fewer. Split larger ones by subtree and say how they were split.
- One diagram per fenced ```` ```mermaid ```` block, each under a heading that names the diagram and its source artifact.
- Mark external actors with a distinct shape (`([External])`), and group the system's own elements in a `subgraph` named after the system.

## Checks

| Check | Rule |
|---|---|
| Grounded | Every node and edge maps to an ID in a source artifact. |
| Complete | Every ID in scope appears, or the split note says where it went. |
| Renders | Node IDs are unique and contain no `.`, spaces, or quotes, and labels with special characters are quoted. |

## Output

Write `diagrams.md` in the workspace, replacing only the sections being regenerated, and show the diagrams in the reply.
