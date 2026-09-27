---
title: "Project Risk List & Exposure Register"
status: "active"
last_updated_at: "2026-09-27"
iteration: "Elaboration-Iteration-01"
---

# Project Risk List & Exposure Register

## 1. Overview & Iteration Goals

This register is the Single Source of Truth (SSoT) for engineering and operational risks governing the **`editorial-engine`** platform. 

In accordance with Barry W. Boehm's Software Risk Management Framework and the Unified Process (UP), risks are ranked strictly by calculated **Risk Exposure ($RE = P \times I$)**. High-exposure risks ($RE \ge 16$) require active architectural mitigation spikes during the current Elaboration phase before Construction is authorized.

## 2. Quantitative Exposure Scoring Methodology

$$\text{Risk Exposure (RE)} = \text{Probability } (1\text{–}5) \times \text{Impact } (1\text{–}5)$$

* **High Exposure ($RE \ge 16$):** Critical hazard. Blocks full-scale Construction until mitigated.
* **Medium Exposure ($9 \le RE \le 15$):** Moderate hazard. Actively monitored; architectural spikes planned in Elaboration.
* **Low Exposure ($RE \le 8$):** Low hazard. Documented and accepted without active spike tasks.

---

## 3. Master Risk Register Matrix

| Risk ID | Risk Description | Category (Boehm) | Prob (1–5) | Imp (1–5) | RE ($P \times I$) | Mitigation Strategy / Spike Plan | Status |
| :--- | :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| `RISK-ENG-0001` | **Filesystem Boundary Traversal:** Malicious or malformed agent path queries escape the `editorial-content` vault root to read or modify host files. | Straining Technical Capabilities | 3 | 5 | **15 (Med)** | Spike: Implement strict POSIX path canonicalization (`os.path.realpath`) and chroot-like virtual root validation in `src/core/security/`. | Active |
| `RISK-ENG-0002` | **Context Window Exhaustion / Bloat:** Ingesting large PDFs or whitepapers into LLM agent sessions exhausts token limits or causes cognitive degradation. | Real-Time Performance Shortfalls | 4 | 4 | **16 (High)** | Spike: Enforce Ingestion Micro-Pipeline normalizer producing compact CommonMark summaries and chunked embeddings rather than raw text dumping. | Active |
| `RISK-ENG-0003` | **Substack ProseMirror AST Schema Drift:** Substack updates its internal JSON editor schema, breaking the automated Markdown-to-ProseMirror compiler. | Shortfalls in External Components | 3 | 4 | **12 (Med)** | Spike: Decouple AST transformer in `src/core/transformers/` with automated schema regression test fixtures validating against Substack API payloads. | Active |
| `RISK-ENG-0004` | **Split-Brain Working Tree Conflicts:** Engine tools modifying files while the human author is editing causes lost edits or Git conflicts in `editorial-content`. | Wrong Software Functions | 3 | 4 | **12 (Med)** | Spike: Enforce Policy 5.1 (zero automated commits; engine leaves unstaged changes) and implement atomic file replacement (`tempfile` + `os.replace`). | Active |
| `RISK-ENG-0005` | **Citation & Glossary Synonym Drift:** Author uses inconsistent nomenclature or hallucinated citations across multi-issue publication series. | Wrong Software Functions | 4 | 3 | **12 (Med)** | Spike: Implement deterministic AST linter loading `research/glossary.yaml` and `research/bibliography.bib` during Stage 6 line-editing. | Active |

---

## 4. Iteration Spike Execution & Retirement History

* **Iteration 01 (Inception / Elaboration Boundary):**
  * Identified `RISK-ENG-0002` ($RE=16$) as the governing critical risk requiring an architectural mitigation spike during Elaboration before the Ingestion Pipeline is coded.
