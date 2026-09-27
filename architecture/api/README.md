# Interface & Boundary Contract Specifications

This directory houses the authoritative **machine-readable boundary schemas and interface contracts**, serving the **Analysis & Design Discipline** and **Implementation Discipline** of the Unified Software Development Process (USDP / UP).

---

## 1. Canonical Framework & Standards

* **Governing Framework:** OpenAPI Specification 3.1.0, AsyncAPI Specification, and JSON Schema Draft 2020-12.
* **Format:** Strictly typed YAML/JSON schema definitions (`openapi.yaml`, `asyncapi.yaml`).
* **Canonical Path:** `architecture/api/` (or repository root `api/`).

---

## 2. Directives & Contract-First Enforcements

1. **Contract-First Development:** Interservice REST endpoints, daemon sockets, and event streams must have their schema defined and validated *before* server stubs or client libraries are implemented.
2. **Deterministic Code Generation:** Contracts must serve as the single source of truth for generating typed client libraries, server routing boilerplate, and mock testing fixtures.
3. **Automated Verification:** Interface contracts must be linted via schema linters (e.g., `spectral`) and verified via automated contract tests in CI pipelines.
4. **Zero Ambient Schemas:** In-flight network messages, error envelopes, and query parameters must be explicitly typed with strict validation rules.

---

## 3. OpenAPI 3.1 Schema Blueprint Example (`openapi.yaml`)

```yaml
openapi: 3.1.0
info:
  title: "Workstation Ops Subsystem Daemon API"
  version: "1.0.0"
  description: "Authoritative interface contract for workstation task execution."

paths:
  /api/v1/tasks:
    post:
      summary: "Submit a new workstation task"
      operationId: "submitTask"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/TaskSubmission"
      responses:
        '202':
          description: "Task accepted for execution"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/TaskReceipt"
        '400':
          $ref: "#/components/responses/ValidationError"

components:
  schemas:
    TaskSubmission:
      type: object
      required: ["name", "workload_type"]
      properties:
        name:
          type: string
          minLength: 3
          maxLength: 128
        workload_type:
          type: string
          enum: ["lxc_provision", "vault_sync", "backup"]
        timeout_seconds:
          type: integer
          default: 300
    TaskReceipt:
      type: object
      required: ["task_id", "status", "submitted_at"]
      properties:
        task_id:
          type: string
          format: uuid
        status:
          type: string
          enum: ["PENDING", "RUNNING", "COMPLETED", "FAILED"]
        submitted_at:
          type: string
          format: date-time
  responses:
    ValidationError:
      description: "Invalid request payload"
      content:
        application/json:
          schema:
            type: object
            required: ["error", "details"]
            properties:
              error:
                type: string
              details:
                type: array
                items:
                  type: string
```
