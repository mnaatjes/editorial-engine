# Project Risk Management & Risk Register Governance

This directory houses the **Master Risk Register & Exposure List** (`risk-list.md`) for the repository, serving the **Project Management Discipline** of the Unified Software Development Process (USDP / UP).

---

## 1. Canonical Framework & Standards

* **Governing Framework:** Barry W. Boehm's Software Risk Management Framework (1989/1991) and the Unified Process / RUP Risk List Specification.
* **Primary Artifact:** `risk-list.md` (Single Source of Truth for repository risks).
* **Format:** Quantitative, prioritized risk matrix ranked strictly by calculated **Risk Exposure ($RE$)**.

---

## 2. Directives & Enforcement Rules

1. **Risk-Driven Iterations:** The primary purpose of this directory is to steer architectural spikes, prototypes, and MADRs during the **Inception** and **Elaboration** phases before full-scale Construction is authorized.
2. **Strict Exposure Ordering:** Every cataloged risk must be assigned quantitative Probability ($P$) and Impact ($I$) values (1–5 scale), yielding an objective exposure score:
   $$\text{Risk Exposure (RE)} = P(\text{Unsatisfactory Outcome}) \times \text{Size of Loss (Impact)}$$
   * **High Exposure ($RE \ge 16$):** Critical. MUST be actively mitigated in the immediate iteration (Elaboration phase gate).
   * **Medium Exposure ($9 \le RE \le 15$):** Moderate. Monitored; architectural mitigation spikes planned during Elaboration.
   * **Low Exposure ($RE \le 8$):** Low. Documented and accepted without active spike tasks.
3. **Boehm Top-10 Taxonomy Alignment:** All risks must be categorized using Boehm's foundational taxonomy:
   * *Personnel Shortfalls*, *Unrealistic Schedules/Budgets*, *Wrong Software Functions*, *Wrong User Interface*, *Gold-Plating*, *Continuous Requirements Changes*, *Shortfalls in External Components*, *Shortfalls in External Tasks*, *Real-Time Performance Shortfalls*, *Straining Technical Capabilities*.
4. **Prohibition:** Do **NOT** record routine software bugs (issue tracker), daily sprint tasks, or living end-user documentation here.

---

## 3. Template & Schema Blueprint (`risk-list.md`)

```markdown
---
title: "Project Risk List & Exposure Register"
status: "active" # active | retired | transferred
last_updated_at: "YYYY-MM-DD"
iteration: "Elaboration-Iteration-02"
---

# Project Risk List & Exposure Register

## 1. Overview & Iteration Goals
<Summary of current lifecycle phase, iteration objectives, and risk retirement gates.>

## 2. Quantitative Exposure Scoring Methodology
$$\text{Risk Exposure (RE)} = \text{Probability (1–5)} \times \text{Impact (1–5)}$$

## 3. Master Risk Register Matrix

| Risk ID | Risk Description | Category (Boehm) | Prob (1–5) | Imp (1–5) | RE ($P \times I$) | Mitigation Strategy / Spike Plan | Status |
| :--- | :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| `RISK-ARCH-0001` | PostgreSQL connection pool saturation during peak burst workloads | Real-Time Performance Shortfalls | 4 | 4 | 16 (High) | Spike: Implement PgBouncer connection multiplexer in front of daemon | Active |
| `RISK-OPS-0002` | Proxmox API token expiration breaking automated LXC provisioning | Shortfalls in External Components | 3 | 4 | 12 (Med) | Spike: Author Vault token auto-renewal task in Ansible | Active |
| `RISK-REQ-0003` | Operator configuration drift between inventory.ini and hypervisor | Wrong Software Functions | 2 | 3 | 6 (Low) | Enforce read-only state validation task prior to execution | Monitored |

## 4. Iteration Spike Execution & Retirement History
* **Spike: PgBouncer Evaluation (Completed):** Retired `RISK-ARCH-0001` from $RE=16$ down to $RE=4$. Documented in [ADR-0005](architecture/adr/0005_use_pgbouncer.md).
```
