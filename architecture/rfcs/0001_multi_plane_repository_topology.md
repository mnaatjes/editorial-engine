---
title: "RFC 0001: Multi-Plane Repository Topology and Editorial Pipeline Architecture"
tags: ["rfc", "architecture", "editorial", "mcp", "topology"]
created_at: "2026-09-27"
last_updated_at: "2026-09-27"
rfc_id: "0001"
status: "under_review"
level: "parent"
parent_rfc: null
authors: ["@mnaatjes"]
---

# RFC 0001: Multi-Plane Repository Topology and Editorial Pipeline Architecture

## 1. Overview / Summary

This Request for Comments (RFC) articulates the architectural decomposition and multi-repository topology for an AI-assisted editorial and publishing platform targeting Substack and long-form technical publications. The core design decouples software engineering governance and tool source code from the authoring workspace and operational deployment environments.

By formalizing the separation into three primary planes—**Tooling & Governance (`editorial-engine`)**, **Editorial & Content (`editorial-content`)**, and **Deployment & Infrastructure (`editorial-ops`)**—the system eliminates context window pollution, prevents operational secret leakage into authoring spaces, and establishes a standardized Model Context Protocol (MCP) boundary across all lifecycles.

```mermaid
flowchart TD
    subgraph EnginePlane["1. Platform & Tooling Plane (editorial-engine)"]
        direction TB
        Arch["architecture/ (RFCs, ADRs, SDDs, Schemas)"]
        MCPSrc["src/ (MCP Server Implementation)"]
    end

    subgraph ContentPlane["2. Editorial & Content Plane (editorial-content)"]
        direction TB
        Research["research/ (Clips, PDFs, Notes)"]
        Drafts["drafts/ (Markdown Essays, Frontmatter)"]
        WProfile[".gemini/ (Editorial Persona Rules, Voice)"]
    end

    subgraph OpsPlane["3. Operational & Publishing Plane (editorial-ops)"]
        direction TB
        Collector["Ingests Finished Assets & Drafts"]
        Publisher["Publishes to Target: Substack"]
    end

    EnginePlane -- "Provides Tools & Protocol" --> ContentPlane
    ContentPlane -- "Delivers Finished Draft" --> OpsPlane
    OpsPlane --> Substack["Target Platform: Substack"]
```

---

## 2. Problem Statement & Context

Coupling editorial prose, raw research artifacts, tool source code, and deployment automation within a monolithic repository degrades both the authoring experience and engineering lifecycle:

1. **Context Window Contamination:** Modern agentic LLM assistants (e.g., Antigravity, Gemini CLI) recursively scan workspace trees. Ingesting infrastructure manifests, node modules, Python virtual environments, and test fixtures during a drafting session wastes token budgets and causes hallucinations.
2. **Security & Boundary Violation:** Staging drafts to Substack requires session tokens, browser automation cookies, or platform credentials. Storing or referencing these credentials in an authoring workspace presents accidental leakage risks.
3. **Mismatched Lifecycles:** Editorial drafts evolve non-linearly with rapid commits and revisions. Infrastructure and platform tooling require strict versioning, linting, regression testing, and architectural review (MADRs, SDDs).
4. **Tool Portability Constraints:** Hardcoding custom scripts to specific file paths prevents re-using the editorial tooling across multiple distinct publications or private research vaults.

### Goals & Non-Goals

* **Goals:**
  * Define strict physical and logical repository boundaries for the editorial ecosystem.
  * Define the Model Context Protocol (MCP) interface contract connecting authoring environments to background tool execution.
  * Guarantee zero source code and zero operational credentials reside within the writing workspace.
  * Establish a clear 7-stage professional editorial lifecycle framework (Research $\rightarrow$ Synthesis $\rightarrow$ Outlining $\rightarrow$ Drafting $\rightarrow$ Fact-Checking $\rightarrow$ Line Editing $\rightarrow$ Packaging).
* **Non-Goals:**
  * Re-implementing a native Markdown editor (the system integrates with existing editors via standard MCP clients).
  * Supporting multi-tenant collaborative SaaS drafting in phase 1 (focused on single-author sovereign homelab workflow).

---

## 3. Proposed Architecture & Repository Topology

### 3.1 Repository Taxonomy Breakdown

| Repository Name | Target Persona | Plane / Responsibility | Authoritative Contents |
| :--- | :--- | :--- | :--- |
| **`editorial-engine`** | Software Engineer / System Architect | **Governance & Source Code Plane** | • `architecture/` (RFCs, MADRs, IEEE 1016 SDDs, OpenAPI/JSON-RPC schemas).<br>• Core source code for custom MCP servers (`src/`).<br>• Custom linter definitions, style rule engines, and readability evaluators.<br>• Unit, integration, and contract tests. |
| **`editorial-content`** | Writer / Research Essayist | **Authoring & Editorial Plane** | • `research/` (Clips, PDFs, reading notes, citations).<br>• `drafts/` (Active essays, structural outlines, revision histories).<br>• `assets/` (Visual diagrams, cover images, tables).<br>• `.gemini/` (Editorial persona directives, voice guidelines).<br>• *Zero application code, zero build tools, zero deployment scripts.* |
| **`editorial-ops`** | Systems / DevOps Operator | **Deployment & Staging Plane** | • Container and process management manifests.<br>• Secret management bindings for platform authentication.<br>• Automated collectors and delivery scripts publishing finished content to Substack. |

---

### 3.2 System Interconnect & The MCP Protocol Boundary

The bridge between `editorial-content` and `editorial-engine` is governed strictly by the **Model Context Protocol (JSON-RPC 2.0)**, avoiding custom bespoke REST client code inside the writing workspace.

#### Communication Pattern: Detached Runtime via Network MCP (SSE) or Local Stdio
* **Network Mode (Remote/Containerized Daemon):**
  * `editorial-ops` runs the `editorial-engine` daemon in a background container or service on port `8765`.
  * The writer's environment (`editorial-content`) contains only a declarative client configuration file:
    ```json
    {
      "mcpServers": {
        "editorial": {
          "url": "http://127.0.0.1:8765/sse"
        }
      }
    }
    ```
* **Local Subprocess Mode (Self-Contained Binary):**
  * `editorial-engine` compiles a standalone release binary (`editorial-cli`).
  * The authoring environment launches it dynamically:
    ```json
    {
      "mcpServers": {
        "editorial": {
          "command": "editorial-cli",
          "args": ["mcp", "--vault", "."]
        }
      }
    }
    ```

---

### 3.3 The 7-Stage Editorial and Authoring Lifecycle

From a professional non-fiction writer and technical essayist perspective, content production is structured into seven discrete, sequential stages:

```mermaid
flowchart LR
    S1["1. Ingestion & Research"] --> S2["2. Sensemaking & Synthesis"]
    S2 --> S3["3. Thesis & Outlining"]
    S3 --> S4["4. The Zero Draft"]
    S4 --> S5["5. Fact-Checking"]
    S5 --> S6["6. Multi-Pass Line Editing"]
    S6 --> S7["7. Packaging & Staging"]
```

1. **Stage 1: Ingestion & Research:** Systematic capture of primary source materials, academic publications, whitepapers, interview transcripts, and bookmarks into local archives.
2. **Stage 2: Sensemaking & Synthesis:** Extracting core patterns, claims, tensions, and verified data points from research materials into a structured research brief without drafting prose.
3. **Stage 3: Thesis Formulation & Outlining:** Establishing the central thesis statement (governing thought) and building a detailed hierarchical outline (e.g., Minto Pyramid structure) mapping arguments to citations.
4. **Stage 4: Drafting (The Zero Draft):** Rapid, unconstrained translation of the outline into full prose, deliberately bypassing internal self-critique and using placeholders (`[TK]`) for missing data to preserve velocity.
5. **Stage 5: Fact-Checking & Verification:** Rigorous verification pass resolving all `[TK]` placeholders, validating claims against primary sources, and verifying numerical accuracy.
6. **Stage 6: Multi-Pass Line Editing:** Progressive editorial passes refining voice, cadence, clarity, and grammatical hygiene (structural flow $\rightarrow$ line editing $\rightarrow$ copyediting/proofreading).
7. **Stage 7: Packaging & Staging:** Assembly of publication-ready assets including headlines, email pre-headers, pull quotes, 16:9 banner art, and social hooks prior to handing off to operations for publishing.

---

## 4. Alternatives Considered: REST API vs. MCP

* **Custom REST API:** Exposing traditional HTTP/REST endpoints from `editorial-engine` requires writing and maintaining custom client-side glue code, tool adapters, or curl commands inside the writer's environment. Furthermore, OpenAPI schemas must be manually updated whenever server endpoints change.
* **Model Context Protocol (Recommended):** MCP standardizes tool capability negotiation and resource streaming over JSON-RPC 2.0. Native MCP support in modern AI clients (Antigravity, Gemini CLI, Claude, Cursor) enables dynamic runtime discovery without writing client-side boilerplate, while preserving local-first security.

---

## 5. Security, Risk & Operational Impact

* **Secret Isolation:** Substack session tokens, session cookies, and API secrets are never checked into `editorial-content` or exposed to the client LLM context. They are managed exclusively within the operational plane (`editorial-ops`).
* **Filesystem Containment:** `editorial-engine` operates only on paths explicitly designated for authoring. Path traversal outside the configured publication root is rejected.
* **Failure Modes:** If `editorial-engine` is offline, the writer's ability to author, review, or edit local Markdown files remains completely functional; only AI assistance is temporarily unavailable.

---

## 6. Next Steps

1. Review and ratify this high-level repository topology and lifecycle proposal.
2. Define the functional requirements and editorial interaction models for Stages 1 through 7 prior to designing specific MCP primitives.
