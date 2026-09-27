---
title: "Use-Case Specification: UC-01 Initialize and Validate Editorial Content Vault"
use_case_id: "UC-01"
status: "approved"
version: "1.0.0"
level: "sea_level"
primary_actor: "Writer / System Operator"
last_updated_at: "2026-09-27"
---

# Use-Case Specification: UC-01 Initialize and Validate Editorial Content Vault

## 1. Use-Case Name & Metadata
* **Identifier:** UC-01
* **Name:** Initialize and Validate Editorial Content Vault
* **Scope:** `editorial-content` Workspace / `editorial-engine` Boundary
* **Level:** User-Goal (Sea Level)
* **Primary Actor:** Writer / System Operator
* **Stakeholders & Interests:**
  * *Author:* Demands a sovereign, distraction-free Markdown directory layout that opens seamlessly in local editors (Obsidian, VS Code) without broken paths or platform dependencies.
  * *System Architect:* Requires strict adherence to RFC 0001, RFC 0002, and ADR 0002 (shallow topography $\le 2$ levels, zero AI config directories, human Git sovereignty).

## 2. Pre-conditions & Post-conditions
* **Pre-conditions:**
  * Target filesystem directory exists under user home space (e.g., `~/src/github.com/mnaatjes/editorial-content/`).
  * Directory is initialized as a sovereign Git repository.
* **Post-conditions:**
  * **Success:** The `editorial-content` vault contains the canonical shallow layout (`drafts/`, `research/notes/`, `assets/`), baseline template files (`research/glossary.yaml`, `research/bibliography.bib`), zero `.gemini/` configuration files, and passes all filesystem policy validations.
  * **Failure:** Directory remains untouched or partial scaffolding is reported with actionable errors.

## 3. Flow of Events

### 3.1 Basic Flow (Main Success Scenario)
1. Actor initiates vault layout scaffolding for `editorial-content`.
2. System establishes the canonical shallow topography:
   * Creates `drafts/` directory for standalone drafts and single-level series bundles.
   * Creates `research/notes/` directory for normalized research captures.
   * Creates `assets/` directory for scoped media folders.
3. System provisions the initial publication-wide reference contracts:
   * Instantiates `research/glossary.yaml` conforming to the `GlossaryContract` schema.
   * Instantiates `research/bibliography.bib` for master citation keys.
4. System validates vault governance invariants:
   * Confirms directory depth does not exceed 2 levels.
   * Verifies that no build files (`pyproject.toml`, `package.json`), deployment configs, or secrets exist.
   * Confirms complete absence of AI agent directive directories (`.gemini/`, `.cursor/`).
5. System displays verification confirmation that the content vault is ready for human authoring and engine attachment.

### 3.2 Alternative Flows (Extensions)
* **4a. Prohibited AI Directory or Build Files Detected:**
  * System detects `.gemini/` or build artifacts inside the content vault.
  * System alerts actor of policy violation (Anti-Pattern 6: Agent Config Leakage).
  * System halts validation until unauthorized directories are removed.
* **4b. Directory Nesting Depth Violation:**
  * System detects directory paths exceeding 2 levels deep (Anti-Pattern 1: Deep Taxonomy).
  * System flags paths for flattening.

## 4. Special Requirements & Invariants
* **Universal Portability (Policy 1):** Every scaffolded file must be UTF-8 CommonMark or standard YAML, readable without external software.
* **Zero Machine Commits (Policy 5.1):** Vault initialization files are created in the working directory; the human writer retains exclusive authority to review, stage, and execute `git commit`.
