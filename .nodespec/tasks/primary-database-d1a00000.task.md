# Task: Primary Database

> **Scope:** implement ONLY this node ("Primary Database"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Database
**Technology:** aws-rds-postgresql
**Description:** Persistent data storage (relational or document)

## Your Deliverable

**Working code for this component**, honoring the contracts and criteria below, plus its configuration artifacts and tests.

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Scaffold the Database component.**
  Create the source layout, build files, and test harness this node's working code lives in.
- [ ] **T2 — Expose the interface API Functions consumes, per Contract "App Data Queries" (sql).**
  Record the endpoint/identifiers API Functions needs in this node's config artifacts — coordinate with API Functions.
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-006 "Application data is stored in a managed relational database instance" — requirement not mapped to that node; verify or reassign before relying on it
  ↳ serves (unverified match): REQ-006 "The database is not directly reachable from the public internet" — requirement not mapped to that node; verify or reassign before relying on it
  ↳ serves (unverified match): REQ-006 "Automated backups are enabled on the database" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T3 — Expose the interface Heavy Job Worker consumes, per Contract "Worker Data Queries" (sql).**
  Record the endpoint/identifiers Heavy Job Worker needs in this node's config artifacts — coordinate with Heavy Job Worker.
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T4 — Implement: "Multi-AZ failover is an explicit, documented decision (not left implicit); Single-AZ is acceptable pre-production to control cost, with Multi-AZ as the documented upgrade path before production traffic" (REQ-006).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-006 "Multi-AZ failover is an explicit, documented decision (not left implicit); Single-AZ is acceptable pre-production to control cost, with Multi-AZ as the documented upgrade path before production traffic"
- [ ] **T5 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-006: Managed Relational Data Persistence
Category: functional | Status: in-progress
Application data shall persist in a managed relational database, accessible only from the application compute layer.

**Acceptance criteria — your task boxes:**
- [ ] Application data is stored in a managed relational database instance
  → covered by Task T2
- [ ] The database is not directly reachable from the public internet
  → covered by Task T2
- [ ] Automated backups are enabled on the database
  → covered by Task T2
- [ ] Multi-AZ failover is an explicit, documented decision (not left implicit); Single-AZ is acceptable pre-production to control cost, with Multi-AZ as the documented upgrade path before production traffic
  → covered by Task T4

## Interface Contracts

### RECEIVES FROM: API Functions (serverless-function)
- **Contract:** App Data Queries
- **Protocol:** sql
- **Their Technology:** aws-lambda

**Schema:**
```
{
  "type": "connection_contract",
  "description": "Access-pattern contract only — no table/data schema is defined anywhere in the spec; do not fabricate one. Enforces parameterized queries only (no string-concatenated SQL) and no direct DDL access from this role.",
  "authentication": "Credentials retrieved via the Secrets Manager dependency contract, not hardcoded"
}
```

### RECEIVES FROM: Heavy Job Worker (serverless-function)
- **Contract:** Worker Data Queries
- **Protocol:** sql
- **Their Technology:** aws-lambda

**Schema:**
```
{
  "type": "connection_contract",
  "description": "Same access-pattern contract as App Data Queries — no fabricated table schema. Parameterized queries only, Secrets-Manager-issued credentials.",
  "authentication": "Credentials retrieved via the Secrets Manager dependency contract"
}
```

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Depends on THIS node being available:**
- API Functions (initiates App Data Queries against this node (sql))
- Heavy Job Worker (initiates Worker Data Queries against this node (sql))

**Parent Container:** VPC (vpc)
