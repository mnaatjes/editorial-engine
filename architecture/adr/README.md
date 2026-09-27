# Architectural Decision Records (ADR)

This directory serves as the **immutable historical decision ledger** for binding technical choices, serving the **Analysis & Design Discipline** of the Unified Software Development Process (USDP / UP).

---

## 1. Canonical Framework & Standards

* **Governing Framework:** Markdown Architectural Decision Records (MADR 3.0) and Martin Fowler's Conversational Architecture pattern.
* **Format:** Monotonically numbered Markdown records (`NNNN_descriptive_title.md`).
* **Canonical Path:** `architecture/adr/`

---

## 2. Directives & Immutability Rules

1. **Immutable Historical Ledgers:** Accepted ADRs are point-in-time snapshots of architectural consensus. **They are never rewritten or edited** when system architecture evolves.
2. **Supersession via New ADRs:** When a technical decision changes (e.g., migrating databases), author a new sequential ADR (e.g., `0012_use_sqlite.md`) that explicitly references and marks the earlier record as `Supersedes [ADR-0002](...)`.
3. **Sequential Naming Convention:** All files must follow the 4-digit zero-padded format: `NNNN_snake_case_title.md` (e.g., `0001_record_architecture_decisions.md`).
4. **Mandatory Decision Drivers:** Every record must articulate the technical forces, constraints, and operational requirements that drove the choice.

---

## 3. MADR 3.0 Template Example

```markdown
---
status: "accepted" # proposed | accepted | rejected | superseded | deprecated
date: "YYYY-MM-DD"
deciders: ["@lead-architect", "@platform-engineer"]
consulted: ["@security-team"]
informed: ["@engineering-all"]
---

# Use PostgreSQL for Transactional State Storage

## Context and Problem Statement

The platform requires ACID-compliant persistence for distributed workstation state records. We must select a database engine that handles relational state alongside unstructured JSON telemetry payloads under Linux host management.

## Decision Drivers

* Strict ACID transactional guarantees.
* First-class native JSON indexing (`JSONB`).
* Standardized Linux package and container deployment workflows.
* Zero external licensing or vendor lock-in.

## Considered Options

* PostgreSQL 16
* MySQL 8
* SQLite 3

## Decision Outcome

Chosen option: "PostgreSQL 16", because it natively satisfies relational consistency while offering fast indexable binary JSON storage without introducing secondary document stores.

### Consequences

* Good, because telemetry and relational state share a unified backup and replication topology.
* Good, because ecosystem tooling (pgBackRest, psql) is mature across Linux hosts.
* Bad, because operational memory footprints are higher than embedded alternatives.
* Neutral, because developers must be trained on PostgreSQL-specific indexing strategies.

## Validation

Validated via automated integration test suite running transactional rollbacks and throughput benchmarks against a PostgreSQL container instance.

## Pros and Cons of the Options

### PostgreSQL 16

* Good, because native `JSONB` support allows GIN indexing on unstructured logs.
* Bad, because connection memory overhead requires connection pooling (PgBouncer) under high concurrency.

### SQLite 3

* Good, because zero-configuration single-file deployment requires no background daemon.
* Bad, because database-level write locks cause contention under concurrent multi-process writes.
```
