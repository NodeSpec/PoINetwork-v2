# Task: Secrets Manager

> **Scope:** implement ONLY this node ("Secrets Manager"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Auth Provider
**Technology:** AWS Secrets Manager
**Description:** Authentication and identity management service

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

- [ ] **T1 — Provision AWS Secrets Manager via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  **Configuration (account-wide baseline per REQ-019/REQ-014):**
  - Rotation: automatic, Lambda-based rotation functions (no manual rotation for production secrets)
  - Database credentials: 30-day rotation window
  - Third-party API keys: 90-day rotation window (or the provider's maximum supported interval, whichever is shorter)
  - Encryption: default AWS-managed KMS key (aws/secretsmanager) unless a customer-managed key is separately required
  - Access: resource policy scoped narrowly to the specific consuming execution roles (API Functions, Heavy Job Worker) — no wildcard principals
  - Access pattern: retrieved by ARN via VPC endpoint from private-subnet compute (ties to REQ-015 — private connectivity), not over the public internet
- [ ] **T2 — Expose the interface API Functions consumes, per Contract "Retrieve DB Credentials" (dependency).**
  Record the endpoint/identifiers API Functions needs in this node's config artifacts — coordinate with API Functions.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
  ↳ serves (unverified match): REQ-014 "Database and third-party credentials are retrieved from a managed secrets store at runtime, never hardcoded or committed to source" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T3 — Expose the interface Heavy Job Worker consumes, per Contract "Retrieve DB Credentials (Worker)" (dependency).**
  Record the endpoint/identifiers Heavy Job Worker needs in this node's config artifacts — coordinate with Heavy Job Worker.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T4 — Configure the service to satisfy: "Secrets support rotation without requiring application redeployment" (REQ-014).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-014 "Secrets support rotation without requiring application redeployment"
- [ ] **T5 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-014: Secrets Management
Category: non-functional | Status: in-progress
The system shall manage all credentials through a centralized, rotatable secrets store, retrieved at runtime rather than hardcoded.

**Acceptance criteria — your task boxes:**
- [ ] Database and third-party credentials are retrieved from a managed secrets store at runtime, never hardcoded or committed to source
  → covered by Task T2
- [ ] Secrets support rotation without requiring application redeployment
  → covered by Task T4

## Interface Contracts

### RECEIVES FROM: API Functions (serverless-function)
- **Contract:** Retrieve DB Credentials
- **Protocol:** dependency
- **Their Technology:** aws-lambda

**Schema:**
```
{
  "type": "reference",
  "reference": "secretArn (environment variable, not hardcoded)",
  "description": "Dependency contract, no payload. API Functions retrieves the DB credential by ARN at cold start and caches it for the container's lifetime."
}
```

### RECEIVES FROM: Heavy Job Worker (serverless-function)
- **Contract:** Retrieve DB Credentials (Worker)
- **Protocol:** dependency
- **Their Technology:** aws-lambda

**Schema:**
```
{
  "type": "reference",
  "reference": "secretArn (environment variable, not hardcoded)",
  "description": "Dependency contract, no payload. Same shape as the API Functions variant — credential retrieved by ARN at cold start."
}
```

## Technology Guidance

**Purpose:** Centralized secrets management service for rotating, managing, and retrieving database credentials, API keys, and other secrets. Supports automatic rotation with Lambda functions, fine-grained IAM access policies, and cross-account secret sharing. Integrates natively with RDS, Redshift, and DocumentDB for automatic credential rotation.

**Best Practices:**
- Enable automatic rotation for database credentials
- Use resource-based policies for cross-account access
- Reference secrets by ARN in application config rather than embedding values
- Use VPC endpoints to access Secrets Manager without traversing the internet
- Tag secrets for cost allocation and access control
- Use staging labels to manage secret versions during rotation

**Anti-Patterns to Avoid:**
- Storing secrets in environment variables, config files, or source code instead
- Not enabling automatic rotation for database credentials
- Granting overly broad IAM permissions to retrieve all secrets
- Not using VPC endpoints when accessing from private subnets

**Suggested File Structure:**
- `terraform/secrets-manager.tf` (config)
- `src/config/secrets.ts` (source)

## Dependency Chain

**Depends on THIS node being available:**
- API Functions (initiates Retrieve DB Credentials against this node (dependency))
- Heavy Job Worker (initiates Retrieve DB Credentials (Worker) against this node (dependency))

**Parent Container:** AWS (aws)
