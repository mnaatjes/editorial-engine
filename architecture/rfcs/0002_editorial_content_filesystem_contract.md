---
title: "RFC 0002: Editorial Content Filesystem Schema and Data-Store Contract"
tags: ["rfc", "architecture", "editorial-content", "filesystem", "schema"]
created_at: "2026-09-27"
last_updated_at: "2026-09-27"
rfc_id: "0002"
status: "under_review"
level: "child"
parent_rfc: "0001"
authors: ["@mnaatjes"]
---

# RFC 0002: Editorial Content Filesystem Schema and Data-Store Contract

## 1. Overview / Summary

This RFC establishes the filesystem architecture, directory layout, document schemas, and governing policies for the **`editorial-content`** repository. 

In our multi-plane architecture (defined in [RFC 0001](file:///home/michael/src/github.com/mnaatjes/editorial-engine/architecture/rfcs/0001_multi_plane_repository_topology.md)), `editorial-content` serves as the authoritative, sovereign **Data / Model Tier**. It houses all raw research materials, synthesized notes, working drafts, and visual media assets. 

This RFC specifies a **minimal, flat, and standard-compliant data-store** that maximizes writer ergonomics in standard Markdown editors (e.g., Obsidian, VS Code, Neovim) while providing deterministic boundary contracts for ingestion, linting, and AST compilation by `editorial-engine`.

```mermaid
flowchart TD
    subgraph VaultRoot["editorial-content/ (Sovereign Data Store)"]
        direction TB
        subgraph Topography["Directory Topography"]
            D["drafts/ (Active & Published Essays)"]
            R["research/ (Notes & Bibliographies)"]
            A["assets/ (Images, SVGs, Figures)"]
            G[".gemini/ (Workspace Persona & Directives)"]
        end
        
        subgraph Schemas["Document Schemas"]
            DSchema["Draft Schema: YAML Frontmatter + CommonMark Body"]
            RSchema["Research Schema: YAML Frontmatter + Clean Text"]
        end
        
        D --- DSchema
        R --- RSchema
    end

    subgraph Consumer["editorial-engine (Filesystem Governor)"]
        Gov["Chroot Sandbox & Path Validator"]
    end

    Gov -- "Mounts POSIX Root" --> VaultRoot
```

---

## 2. Problem Statement & Scope

Without an explicit structural contract, authoring repositories degenerate into chaotic filing systems:
1. **Arbitrary Directory Sprawl:** Deep, nested topic hierarchies make manual filing tedious and break relative asset links.
2. **Schema Inconsistency:** Inconsistent frontmatter metadata prevents automated linters and search indexers from reliably determining an essay's status, tags, or citations.
3. **Tool Lock-in:** Using proprietary application formatting or non-standard syntax breaks readability outside specific software tools.

### Goals & Non-Goals

* **Goals:**
  * Define a minimalist, shallow directory topography optimized for human writing velocity.
  * Establish strict, machine-readable YAML frontmatter contracts for drafts and research notes.
  * Define explicit filesystem policies (naming conventions, encoding, immutability, asset linking).
  * Register anti-patterns to prevent architectural creep into the content plane.
* **Non-Goals:**
  * Define the internal database schema of `editorial-engine` (e.g., SQLite FTS5 index tables).
  * Specify automated publishing or deployment scripts (isolated strictly to `editorial-ops`).

---

## 3. Directory Topography & Layout

The directory structure of `editorial-content` is intentionally kept flat (maximum depth of 2 levels) to eliminate filing friction and simplify path resolution:

```text
editorial-content/
├── .gemini/                       # Workspace directives & style prompts
│   ├── rules/                     # Active agent behavioral rules (e.g., voice, tone)
│   └── antigravity.json           # Declarative MCP connection configuration
├── drafts/                        # All long-form essays and publications
│   ├── 2026-09-27_sample-post.md  # ISO-dated slugged drafts
│   └── ...
├── research/                      # Ingested primary sources & synthesis notes
│   ├── notes/                     # Normalized CommonMark research summaries
│   │   └── attention-paper.md
│   └── bibliography.bib           # Optional master BibTeX citation register
└── assets/                        # Static visual media referenced by drafts
    └── 2026-09-27_sample-post/    # Asset folder scoped per article
        ├── banner.png             # 16:9 publication cover image
        └── figure-1.svg           # Inline architecture diagram
```

---

## 4. Document Schemas & Contracts

All content within `editorial-content` consists of UTF-8 CommonMark Markdown with mandatory YAML frontmatter.

### 4.1 Draft Document Contract (`drafts/*.md`)

Every essay or article must begin with the standard `DraftDocument` frontmatter header:

```yaml
---
title: "Article or Publication Title"
slug: "article-slug-for-url"
status: "draft" # draft | review | ready_for_ops | published
created_at: "YYYY-MM-DD"
last_updated_at: "YYYY-MM-DD"
target: "substack"
tags: ["systems", "architecture"]
summary: "Single-sentence executive summary or email pre-header."
cover_image: "../assets/2026-09-27_sample-post/banner.png"
citations:
  - key: "vaswani2017"
    source: "research/notes/attention-paper.md"
---

# Article or Publication Title

Draft body prose adheres strictly to CommonMark.
Unresolved facts or statistics must declare explicit `[TK]` sentinels:
> Revenue grew by [TK: pull Q3 ARR number] year-over-year.
```

#### Status Lifecycle State Machine
* `draft`: Active authoring; Mode 1 (Discovery Engine). Rapid revisions and structural changes permitted.
* `review`: Mode 2 (Publishing Factory). Structural argument frozen; active line-editing, fact-checking, and linting.
* `ready_for_ops`: Convergence Gate passed. Zero `[TK]` sentinels, linter passes clean, assets compiled for `editorial-ops`.
* `published`: Post has been staged or dispatched to the publication target. Immutable archive.

---

### 4.2 Research Note Contract (`research/notes/*.md`)

Normalized research notes generated by the Ingestion Pipeline or authored manually must adhere to the `ResearchDocument` contract:

```yaml
---
title: "Source Document or Article Title"
source_uri: "https://arxiv.org/abs/1706.03762"
source_type: "pdf" # web | pdf | epub | interview | raw_text
captured_at: "YYYY-MM-DDTHH:MM:SSZ"
content_hash: "sha256:4f3a..."
author: "Primary Author Name"
tags: ["transformer", "deep-learning"]
---

# Source Document or Article Title

## Core Thesis & Key Findings
* Bulleted key takeaways and verified factual assertions extracted from source.

## Significant Quotes & Claims
> "Exact quotation with page or section reference." (p. 4)

## Extracted Tables / Structured Data
| Metric | Baseline | Proposed |
| :--- | :--- | :--- |
| BLEU Score | 28.4 | 41.8 |
```

---

## 5. Governing Filesystem Policies

All operations within `editorial-content` must comply with four mandatory policies:

1. **Policy 1: Universal Portability & Zero Code:**
   * `editorial-content` must contain **zero application source code**, zero build manifests (`package.json`, `pyproject.toml`), and zero deployment scripts.
   * Every file must be natively readable and editable in any standard Markdown editor.
2. **Policy 2: Predictable POSIX Naming:**
   * All directories and files must use lowercase alphanumeric characters, numbers, dashes, and underscores (`[a-z0-9_-]`).
   * Filenames must never contain whitespace, special characters, or uppercase letters.
   * Drafts must follow the naming pattern: `YYYY-MM-DD_slug-name.md`.
3. **Policy 3: Relative & Localized Asset Isolation:**
   * Media assets must reside inside `assets/<draft-slug>/` and be referenced using relative paths (e.g., `![Figure 1](../assets/my-post/figure-1.png)`).
   * External remote image links (e.g., hotlinked Imgur/CDN images) are prohibited to guarantee offline data sovereignty.
4. **Policy 4: Append-Only Research & State Immutability:**
   * Research notes in `research/notes/` are immutable historical records of source material at the time of capture.
   * Drafts in `drafts/` are append-and-revise, but once marked `status: published`, they must not be rewritten. Subsequent corrections or errata must be appended as revisions.

---

## 6. Anti-Patterns to Avoid (Registered Hazards)

To preserve system longevity, the following anti-patterns are explicitly registered and forbidden:

| Anti-Pattern Name | Description & Hazard | Governing Enforcement |
| :--- | :--- | :--- |
| **1. The Deep Taxonomy Anti-Pattern** | Creating deep nested directories by category (e.g., `research/tech/ai/llm/agents/2026/`). Causes filing friction and broken relative paths. | **Enforce flat directories:** Group purely by document type (`drafts/`, `research/notes/`). Use YAML `tags:` for categorization. |
| **2. The Proprietary Markup Anti-Pattern** | Inventing custom bracket syntax, non-standard callouts, or tool-specific tags that break standard CommonMark parsers. | **Enforce CommonMark standard:** Use standard blockquotes `>` or standard YAML frontmatter for custom attributes. |
| **3. The Shared State / Database Leak Anti-Pattern** | Storing database files (`index.db`), lock files, or temporary cache directories inside `editorial-content`. | **Enforce stateless vault:** All caches and SQLite indices belong in `editorial-engine/.cache/` or system temp spaces. |
| **4. The Secret Leakage Anti-Pattern** | Storing Substack session cookies, API tokens, or credentials in draft frontmatter or environment files in the vault. | **Enforce zero credentials:** `editorial-content` has zero secrets. Credentials reside strictly in `editorial-ops`. |
| **5. The Split-Brain Workspace Anti-Pattern** | Mixing application development notes or infrastructure runbooks with publication content. | **Enforce pure editorial focus:** Ops documentation lives in `homelab-ops` or `editorial-ops`. Only public/publication content lives here. |

---

## 7. Next Steps

1. Complete RFC 0002 review and obtain user consensus.
2. Apply the initial schema template and directory layout to `/home/michael/src/github.com/mnaatjes/editorial-content/`.
3. Proceed to RFC 0003 specifying the Filesystem Governor and validation rules in `editorial-engine`.
