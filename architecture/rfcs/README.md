# Request for Comments (RFC) Engineering Proposals

This directory houses collaborative **technical proposals and architectural RFCs**, serving the **Requirements Discipline** and **Analysis & Design Discipline** during project Inception and Elaboration.

---

## 1. Canonical Framework & Standards

* **Governing Framework:** Engineering RFC Proposal Lifecycle (inspired by Rust, Kubernetes KEP, and IETF RFCs).
* **Format:** Multi-section exploratory design proposals with explicit option evaluation matrices (`NNNN_proposal_title/` or `NNNN_title.md`).
* **Canonical Path:** `architecture/rfcs/` (or repository root `rfcs/`).

---

## 2. Directives & Lifecycle Enforcements

1. **Collaborative Problem Solving:** RFCs are drafted when an architectural change involves significant trade-offs, multiple viable technical paths, or cross-cutting platform boundaries.
2. **Explicit Option Comparison:** Every RFC must evaluate at least two competing architectural options in Section 4, presenting an objective trade-off matrix before selecting a recommendation.
3. **Parent/Child Hierarchy:** Complex platform initiatives use a parent overview RFC (`0001_parent_initiative/0001_overview.md`) decomposed into focused child RFCs (`0001a_subsystem_spec.md`).
4. **Resolution to ADR:** When an RFC reaches team consensus and status changes to `approved`, its binding conclusions are distilled into an immutable MADR in `architecture/adr/`.

---

## 3. RFC Document Template Example

```markdown
---
title: "RFC 0001: Workstation Task Orchestration Subsystem"
rfc_id: "0001"
status: "under_review" # draft | under_review | approved | superseded | abandoned
level: "parent" # parent | child
parent_rfc: null
created_at: "YYYY-MM-DD"
updated_at: "YYYY-MM-DD"
tags: ["rfc", "orchestration", "architecture"]
authors: ["@mnaatjes"]
---

# RFC 0001: Workstation Task Orchestration Subsystem

## 1. Overview / Summary
Executive summary of the proposal, core architectural changes, and target outcomes.

## 2. Problem Statement & Context
Detailed explanation of why current shell scripts are failing under concurrent workloads.

### Goals & Non-Goals
* **Goals:**
  * Support concurrent execution of up to 16 tasks with isolated logging.
  * Guarantee zero task loss across unexpected host reboots.
* **Non-Goals:**
  * Will NOT build a multi-node distributed consensus cluster.

## 3. Proposed Deep Dive Solution
Detailed technical description of the proposed worker architecture, queue topologies, and daemon mechanics.

## 4. Alternatives Considered & Trade-Off Matrix

| Dimension | Option A: In-Memory Goroutines | Option B: PostgreSQL Backed Workers (Recommended) | Option C: External Redis Queue |
| :--- | :--- | :--- | :--- |
| **Crash Recovery** | Zero (Tasks lost on reboot) | Complete (ACID state persisted) | Good (AOF / RDB snapshots) |
| **Infrastructure Overhead** | None | Low (Reuses existing DB) | Medium (Additional daemon) |
| **Operational Simplicity** | High | High | Moderate |

## 5. Security, Risk & Operational Impact
* Evaluates failure modes, token handling, and operational rollback paths.

## 6. Unresolved Questions
* What is the optimal worker queue backpressure threshold under memory pressure?
```
