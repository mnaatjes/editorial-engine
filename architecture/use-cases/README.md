# Functional Requirements & Use-Case Specifications

This directory houses the authoritative **RUP Use-Case Specifications** for the platform, serving the **Requirements Discipline** of the Unified Software Development Process (USDP / UP).

---

## 1. Canonical Framework & Standards

* **Governing Framework:** Rational Unified Process (RUP) Use-Case Specification Standard and Alistair Cockburn's Goal-Level Hierarchy.
* **Format:** Black-box functional contracts detailing actor interactions, flows of events, and boundary invariants (`UC-NN_descriptive_name.md`).
* **Canonical Path:** `architecture/use-cases/`

---

## 2. Directives & Scoping Enforcements

1. **Black-Box System Boundary:** Specify *what* the system does from the outside in (inputs, observable state changes, and responses). Strictly omit internal code mechanisms, database queries, or private classes.
2. **Sea-Level (User-Goal) Scoping:** Every use case in this directory must represent an elementary business process completed in a single session by a single primary actor yielding measurable value.
   * *Passes (Sea-Level):* `UC-01_provision_lxc_host.md` (passes the Coffee Break Test).
   * *Fails (Underwater/Sub-function):* `inject_ssh_key.md` (this is a step within a use case, not a standalone use case).
3. **Mandatory Event Flow Decomposition:** Every specification must document:
   * **Basic Flow (Main Success Scenario):** The standard, error-free execution path.
   * **Alternative Flows (Extensions):** Numbered recovery branches, exception handlers, and failure exits (e.g., `3a`, `4b`).
   * **Pre-conditions & Post-conditions:** Guarantees that must hold before entry and after completion.

---

## 3. RUP Use-Case Specification Template Example

```markdown
---
title: "Use-Case Specification: UC-01 Provision New LXC Host"
use_case_id: "UC-01"
status: "approved" # draft | under_review | approved | superseded
version: "1.0.0"
level: "sea_level" # sea_level | summary | sub_function
primary_actor: "Infrastructure Operator / CI Pipeline"
last_updated_at: "YYYY-MM-DD"
---

# Use-Case Specification: UC-01 Provision New LXC Host

## 1. Use-Case Name & Metadata
* **Identifier:** UC-01
* **Name:** Provision New LXC Host
* **Scope:** Homelab Automation Platform
* **Level:** User-Goal (Sea Level)
* **Primary Actor:** Infrastructure Operator / CI Pipeline
* **Stakeholders & Interests:** Operators (reliable execution), Security (isolated networking).

## 2. Flow of Events

### 2.1 Basic Flow (Main Success Scenario)
1. Operator submits target container configuration parameters (hostname, cores, memory, VLAN).
2. System validates parameter schema against inventory boundary constraints.
3. System verifies hypervisor cluster health, storage pool headroom, and allocates next available VMID.
4. System clones base OS template and configures network bridges and SSH authorized keys.
5. System boots LXC container and performs network reachability health check.
6. System dynamically registers container IP into active inventory for role execution.

### 2.2 Alternative Flows
* **3a. Insufficient Hypervisor Headroom:**
  1. System detects storage pool or memory threshold breach (<10% free).
  2. System aborts operation without allocating resources.
  3. System logs resource exhaustion failure report to operator.
* **5a. Network Reachability Timeout (60s):**
  1. Container fails network handshake verification within 60 seconds.
  2. System marks container state as DEGRADED and initiates automated rollback deprovisioning.
  3. System alerts operator of bootstrap failure.

## 3. Special Requirements (Non-Functional)
* Container provisioning and network verification must complete in under 90 seconds.
* All API tokens and credentials must be retrieved from vault storage.

## 4. Pre-conditions
* Hypervisor cluster is online and reachable.
* Valid operator credentials exist in vault storage.
* Target base OS template is cached on storage pool.

## 5. Post-conditions
### 5.1 Success Post-condition
* A running, network-accessible LXC container exists with SSH access configured.
### 5.2 Failure Post-condition
* No orphaned or partially configured containers remain; error state is logged.
```
