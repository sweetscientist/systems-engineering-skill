# The Functional-Physical Requirements Framework

This framework uses a Standard Requirements Matrix (SRM) layout. When implementing it in tools like Excel, Jama, or DOORS, use these column definitions to maintain strict bi-directional traceability.

| Column Group | Attribute Name | Format Rule | Framework Definition / Purpose |
|---|---|---|---|
| Traceability | Req ID | `[PREFIX]-[NUMBER]` | Unique identifier for tracking and tracing (e.g., SYS-001, FUNC-042, PHYS-108). |
| Traceability | Parent ID | `[ID]` or `None` | Links directly to the source stakeholder, mission, or higher-level system requirement. |
| Core Statement | Requirement Text | Imperative sentence | Must use the word "shall" for mandatory constraints, followed by a measurable performance target. |
| Architecture | Requirement Type | Single selection | Categorizes the requirement as Functional (action/behavior) or Physical (constraint/material/interface). |
| Functional Map | Functional Block (What) | `[F.X.X] [Verb + Noun]` | The specific function from the Functional Flow Block Diagram (FFBD) or SysML activity diagram that this requirement describes. |
| Physical Map | Allocated Element (Where/How) | `[Subsystem / Part Name]` | The physical component, hardware module, or software configuration item (CI) assigned to perform the function. |
| Verification | Verification Method | Single selection | How the requirement will be proven: Analysis, Demonstration, Inspection, or Test (ADIT). |

## Blueprint matrix example

| Req ID | Parent ID | Requirement Text | Requirement Type | Functional Block (What) | Allocated Element (Where/How) | Verification Method |
|---|---|---|---|---|---|---|
| REQ-001 | SYS-M01 | The system shall decelerate the vehicle from 60 mph to 0 mph in less than 3.0 seconds. | Functional | F.1: Arrest Kinetic Energy | Braking Subsystem | Test |
| REQ-002 | REQ-001 | The system shall clamp the rotor with a force of at least 8,000 Newtons within 150 milliseconds of pedal actuation. | Functional | F.1.1: Apply Clamping Force | Brake Caliper Assembly | Test |
| REQ-003 | REQ-002 | The caliper housing shall be constructed from aluminum alloy with a mass not exceeding 2.5 kg. | Physical | F.1.1: Apply Clamping Force | Brake Caliper Housing (Part 201) | Inspection |
| REQ-004 | REQ-001 | The brake pads shall maintain a coefficient of friction ≥ 0.40 at operational temperatures up to 500 °C. | Physical | F.1.2: Generate Friction | Friction Pads (Part 202) | Analysis |

## Three core rules for maintaining this framework

1. **The "Shall" Test.** Functional requirements specify what the system shall do (action). Physical requirements specify what properties a physical asset shall possess (constraint). Never combine a brand-new function and a physical material constraint into a single sentence.
2. **Allocation Enforcement.** Every requirement marked Functional must have a mapped Functional Block. Every requirement passed down to a design team must have an Allocated Physical Element. If a cell is blank, the architecture has a gap.
3. **N-Squared Flow.** If a functional requirement maps to multiple physical components, split it into child physical requirements, one per component, to prevent blame-shifting during integration and test.

## Choosing the Verification Method (ADIT)

| Method | Use when | Typical rows |
|---|---|---|
| **Test** | The target is a measured performance value under controlled, instrumented conditions. | Timing, force, throughput, accuracy, power draw, range. |
| **Analysis** | Proving by test is impractical or unsafe, so the claim is shown by calculation, model, simulation, or data from a representative article. | Fatigue life, thermal margin, reliability, behavior at environmental extremes, friction at temperature. |
| **Demonstration** | The behavior can be observed to occur without quantitative measurement. | A mode exists, an operator can complete a procedure, a fault is annunciated. |
| **Inspection** | The attribute is confirmed by examining the item, its markings, or its records. | Material, mass on a scale, dimensions, part number, labeling, software version, documentation present. |

Pick exactly one method per row. If a row seems to need two, it is two requirements.
