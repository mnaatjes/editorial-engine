# Architecture & Technical Governance

This directory is the **authoritative Single Source of Truth (SSoT)** for internal engineering governance, requirements specifications, architectural trade-offs, and implementation blueprints within this repository.

> **CRITICAL REPOSITORY DIRECTIVE:**
>
> * **Separation of Planes:** This directory (`architecture/`) belongs strictly to the **Maintainer / Engineering Plane**.
> * **Customer / Operator Isolation:** Customer- and operator-facing documentation is strictly isolated under `docs/` adhering to the four-quadrant Diátaxis framework.
> * **Retained Filetypes:** In an implemented workspace, **ONLY** the six canonical architectural document types enumerated below are retained here. Scratch notes, ad-hoc discussions, and general developer guides must not be committed to this tree.

---

## 1. Directory Tree & Layout Blueprint

```text
architecture/
|-- README.md                      # Directory contract & governance index (This file)
|-- risk/                          # Project Management Discipline
|   \-- risk-list.md               # Boehm-UP Master Risk Register (RE = P * I)
|-- use-cases/                     # Requirements Discipline
|   |-- UC-01_provision_lxc.md     # RUP Use-Case Specifications
|   \-- ...
|-- adr/                           # Analysis & Design Discipline
|   |-- README.md                  # Sequential index of all decisions
|   |-- 0001_initial_record.md     # Markdown Architectural Decision Records (MADR)
|   \-- ...
|-- designs/                       # Analysis & Design / Implementation
|   |-- 0001_subsystem_sdd.md      # IEEE 1016 Software Design Documents (SDD)
|   \-- ...
|-- rfcs/                          # Requirements & Architectural Consensus
|   |-- 0001_parent_proposal/      # Collaborative Engineering RFCs
|   |   |-- 0001_overview.md
|   |   \-- 0001a_child_spec.md
|   \-- ...
\-- api/                           # Interface & Contract Specifications
    |-- openapi.yaml               # OpenAPI 3.1 / AsyncAPI boundary contracts
    \-- ...
```

---

## 2. Retained Architectural Document Types & Directives

| Document Type | Standard / Framework | Canonical Path | Retained Status | Purpose & Binding Directives |
| :--- | :--- | :--- | :---: | :--- |
| **Risk List** | Barry Boehm's Software Risk Taxonomy & UP Risk Register | `architecture/risk/risk-list.md` | **Mandatory** | **Quantify and control project hazards.** Ranks risks strictly by calculated Risk Exposure ($RE = P \times I$) to steer architectural mitigation spikes during Inception and Elaboration. |
| **Use-Case Specifications** | Rational Unified Process (RUP) Use-Case Specification | `architecture/use-cases/` | **Mandatory** | **Black-box functional contract.** Specifies actor-system interactions, primary success scenarios, alternative recovery paths, and boundary conditions for Sea-Level goals (`UC-NN_*.md`). |
| **Architectural Decision Records (ADR)** | Markdown Architecture Decision Records (MADR 3.0) | `architecture/adr/` | **Mandatory** | **Immutable historical decision ledger.** Captures context, options considered, decision drivers, and consequences for binding technical trade-offs (`NNNN_*.md`). Never rewritten; superseded via new ADRs. |
| **Software Design Documents (SDD)** | IEEE 1016-2009 & Google Design Doc Standard | `architecture/designs/` | **Mandatory** | **Pre-implementation engineering blueprints.** Translates ADRs into detailed UML diagrams (component, sequence, state), DDL schemas, and phased PR implementation checklists (`NNNN_*.md`). |
| **Engineering RFCs** | Requests for Comments (RFC) Proposal Framework | `architecture/rfcs/` | **Mandatory** | **Collaborative technical proposals.** Used during Inception and Elaboration to reach team consensus on problem statements, goals, non-goals, and multi-option evaluations (`NNNN_*.md`). |
| **Interface Contracts** | OpenAPI 3.1 / AsyncAPI Specification | `architecture/api/` | **Mandatory** | **Machine-readable boundary schemas.** Authoritative interface contracts for REST, gRPC, or event schemas used for client/server stub generation and automated contract verification tests. |

---

## 3. Workflow & Boundary Enforcements

1. **No Mixed Audience:** Never store end-user tutorials, CLI usage guides, or operational deployment runbooks in `architecture/`. Those belong strictly in `docs/` under the Diátaxis taxonomy (`docs/tutorials/`, `docs/how-to/`, `docs/reference/`, `docs/explanation/`).
2. **Immutable vs. Living Assets:**
   * ADRs in `architecture/adr/` are **immutable historical snapshots**. Once accepted, they are never rewritten. If an architectural decision changes, author a new ADR that supersedes the prior one.
   * Use-Cases, SDDs, and API specs are **living engineering baselines** maintained across active development phases.
3. **Traceability Rule:** Every SDD in `architecture/designs/` must declare explicit frontmatter traceability links to its parent ADRs in `architecture/adr/` and upstream requirements in `architecture/use-cases/` or `architecture/rfcs/`.

---

## 4. Engineering Lifecycle Pipeline (Inception to Release)

```text
[Phase: INCEPTION]
  |-- Business Need / Issue Identified
  |-- Define Actor Scope & Event Flows    --> architecture/use-cases/ (UC-NN_*.md)
  \-- Calculate Risk Exposure (RE = P*I)  --> architecture/risk/risk-list.md
        |
        v
[Phase: ELABORATION]
  |-- Propose & Evaluate Options          --> architecture/rfcs/ (NNNN_*.md)
  |-- Record Binding Decisions (MADR)     --> architecture/adr/ (NNNN_*.md)
  |-- Freeze Boundary Contracts           --> architecture/api/ (openapi.yaml)
  \-- Author Tactical Blueprint (SDD)     --> architecture/designs/ (NNNN_*.md)
        |
        v
[Phase: CONSTRUCTION]
  |-- Phased PR 1: Data Models & DDL      --> migrations/ / repository layer
  |-- Phased PR 2: Core Subsystem Engine  --> business logic / state guards
  |-- Phased PR 3: Interface & CLI Wiring --> API routes / CLI subcommands
  \-- Phased PR 4: Telemetry & Hardening  --> tracing / metrics / integration tests
        |
        v
[Phase: TRANSITION]
  |-- Production Deployment & Tag
  |-- Retire Iteration Risks              --> architecture/risk/risk-list.md
  \-- Publish Living Operator Manuals     --> docs/ (Diataxis Tutorials, How-To, Ref)
```

### Shortened Step-by-Step Workflow

1. **Define the Functional Contract (`use-cases/`):** Capture the actor goal, primary success scenario, and alternative recovery branches in a Sea-Level RUP Use-Case Specification.
2. **Score Project Hazards (`risk/`):** Quantify technical and operational risks ($RE = P \times I$) in `risk-list.md`. Identify high-exposure items ($RE \ge 16$) requiring architectural mitigation spikes.
3. **Explore & Ratify Architecture (`rfcs/` -> `adr/` -> `api/`):**
   * Draft an RFC evaluating competing options via an objective trade-off matrix.
   * Upon consensus, record the binding choice as an immutable MADR in `adr/`.
   * Freeze machine-readable schema contracts in `api/`.
4. **Draft the Implementation Blueprint (`designs/`):** Author an IEEE 1016 SDD detailing UML sequence flows, state machines, database DDL, and a phased PR checklist (passing the *Vacation Test*).
5. **Execute Phased Buildout (Code & Tests):** Implement code across small, testable pull requests strictly matching the SDD sequence milestones.
6. **Retire Risks & Publish User Manuals (`risk/` -> `docs/`):**
   * Downgrade or retire mitigated risks in `risk-list.md`.
   * Lock the SDD status to `completed`.
   * Author or update the customer/operator runbooks and reference schemas in `docs/` under the Diátaxis taxonomy.
