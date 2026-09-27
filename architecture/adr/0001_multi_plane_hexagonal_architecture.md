---
status: "accepted"
date: "2026-09-27"
deciders: ["@mnaatjes"]
consulted: []
informed: []
---

# 0001: Multi-Plane Repository Topology and Hexagonal Engine Architecture

## Context and Problem Statement

Integrating AI assistants (Antigravity, Gemini CLI, Claude) into editorial and publication workflows creates architectural friction when code, deployment secrets, and long-form prose coexist in a monolithic repository:
1. Recursive agent indexing contaminates LLM context windows with build tools and test fixtures.
2. Publishing credentials (Substack session cookies/tokens) risk accidental leakage when colocated with active drafts.
3. Tight coupling between core business logic (prose linting, AST compilation) and a single communication protocol (e.g., Model Context Protocol) prevents human authors from executing tools locally when offline or working outside an LLM environment.

## Decision Drivers

* **Context Window Hygiene:** The authoring workspace must contain zero code dependencies or build manifests.
* **Security & Isolation:** Operational publishing credentials must never reside within the content vault.
* **Protocol Decoupling & Author Sovereignty:** Writers must be able to run all linters, search indices, and transformers without requiring an active LLM agent or network connection.
* **Single Language Unified Standard:** Prevent polyglot maintenance tax across platform tools.

## Considered Options

* **Option A: Monolithic Multi-Directory Repository:** All code, drafts, and deployment scripts live in one Git tree.
* **Option B: Protocol-Coupled Daemon:** Core engine written strictly as an MCP server with no independent domain library.
* **Option C: Multi-Plane Repository Topology with Hexagonal Core (Chosen):** Decomposed into three sovereign Git repositories (`editorial-engine`, `editorial-content`, `editorial-ops`), standardizing on Python 3.12+, with pure domain logic isolated in `src/core/` and decoupled inbound adapters (CLI, MCP, REST).

## Decision Outcome

Chosen option: **Option C**, ratified via [RFC 0001](../rfcs/0001_multi_plane_repository_topology.md).

### Key Architectural Directives:
1. **Three Sovereign Repositories:**
   * **`editorial-engine` (Governance & Source Code):** Houses pure domain logic, linters, transformers, and protocol adapters.
   * **`editorial-content` (Editorial Data Vault):** Sovereign, pure Markdown data-store with zero application code and zero publishing secrets.
   * **`editorial-ops` (Operational Plane):** Manages container runtimes, secret injection, and delivery dispatchers to Substack.
2. **Hexagonal Domain Isolation:** Pure editorial logic in `src/core/` has zero dependencies on transport frameworks.
3. **Multi-Adapter Support:** All three adapters (`CLI`, `MCP`, `REST`) reside in `editorial-engine/src/adapters/` and expose the identical core domain capabilities.
4. **Language Standard:** Python 3.12+ is mandatory across the editorial platform.

### Consequences

* **Positive:**
  * Authoring workspace remains 100% clean Markdown; zero token budget wasted on code indexing.
  * Substack credentials and browser profiles are strictly quarantined in `editorial-ops`.
  * Human authors can execute linters and research ingest via CLI without an LLM.
* **Negative:**
  * Managing three Git repositories introduces cross-repository boundary coordination overhead.
  * Inter-plane testing requires integration fixtures simulating mounted content vaults.
