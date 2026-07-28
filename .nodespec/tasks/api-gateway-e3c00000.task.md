# Task: API Gateway

> **Scope:** implement ONLY this node ("API Gateway"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** API Gateway
**Technology:** AWS API Gateway
**Description:** Request routing, rate limiting, and API management

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Provision AWS API Gateway via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  [PLACEHOLDER: config — no user configuration recorded for this node; confirm sizing/domains/settings with the user]
- [ ] **T2 — Declare the wiring to API Functions (aws-lambda) per Contract "Invoke Lambda" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T3 — Declare the wiring to Cognito User Pool (aws-cognito) per Contract "Token Authorization" (dependency).**
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T4 — Expose the interface Frontend App consumes, per Contract "API Calls" (rest).**
  Record the endpoint/identifiers Frontend App needs in this node's config artifacts — coordinate with Frontend App.
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-007 "API requests are authorized using validated JWTs issued by the identity provider" — requirement not mapped to that node; verify or reassign before relying on it
  ↳ serves (unverified match): REQ-013 "The API layer scales automatically per-request without manual capacity planning" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T5 — Expose the interface CloudFront consumes, per Contract "Dynamic API Requests" (rest).**
  Record the endpoint/identifiers CloudFront needs in this node's config artifacts — coordinate with CloudFront.
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T6 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-007: JWT Request Authorization
Category: functional | Status: in-progress
The API layer shall authorize requests using validated JWTs issued by the managed identity provider.

**Acceptance criteria — your task boxes:**
- [ ] API requests are authorized using validated JWTs issued by the identity provider
  → covered by Task T4

### REQ-013: Auto-Scaling Request Routing
Category: technical | Status: in-progress
The API entry layer shall scale automatically per-request without manual capacity planning.

**Acceptance criteria — your task boxes:**
- [ ] The API layer scales automatically per-request without manual capacity planning
  → covered by Task T4

## Interface Contracts

### RECEIVES FROM: Frontend App (frontend-app)
- **Contract:** API Calls
- **Protocol:** rest
- **Their Technology:** react

**Schema:**
```
{
  "auth": "JWT Bearer token in Authorization header, issued by Cognito, validated by API Gateway JWT authorizer",
  "path": "/api/{proxy+}",
  "type": "http_rest",
  "method": "ANY",
  "description": "React SPA makes authenticated REST calls to the API Gateway for all data operations."
}
```

### RECEIVES FROM: CloudFront (cdn)
- **Contract:** Dynamic API Requests
- **Protocol:** rest
- **Their Technology:** aws-cloudfront

**Schema:**
```
{
  "path": "/api/{proxy+}",
  "type": "http_proxy",
  "method": "ANY",
  "description": "CloudFront forwards all /api/* requests unmodified to API Gateway; no caching applied to this path pattern."
}
```

### SENDS TO: API Functions (serverless-function)
- **Contract:** Invoke Lambda
- **Protocol:** rest
- **Their Technology:** aws-lambda

**Schema:**
```
{
  "type": "lambda_proxy_integration",
  "request": {
    "format": "API Gateway Lambda proxy event: httpMethod, path, headers, queryStringParameters, body, requestContext.authorizer.claims"
  },
  "response": {
    "format": "{statusCode, headers, body}"
  },
  "integrationType": "AWS_PROXY"
}
```

### SENDS TO: Cognito User Pool (auth-provider)
- **Contract:** Token Authorization
- **Protocol:** dependency
- **Their Technology:** aws-cognito

**Schema:**
```
{
  "type": "reference",
  "reference": {
    "issuer": "Cognito User Pool URL",
    "audience": "App Client ID"
  },
  "description": "Dependency contract, no payload. Represents a JWT authorizer configuration referencing the Cognito User Pool's issuer and app client ID."
}
```

## Technology Guidance

_Reference for executing the Implementation Tasks above — apply where relevant. The task list stands even where this guidance is thin._

**Purpose:** Fully managed AWS service for creating, publishing, and managing APIs at any scale with built-in authorization and throttling

**Best Practices:**
- Use HTTP API for simple proxying (cheaper, faster)
- Use REST API when you need request/response transformation
- Implement proper throttling and usage plans
- Use Lambda authorizers for custom auth
- Enable CloudWatch logging
- Use stages for deployment management

**Anti-Patterns to Avoid:**
- Not implementing rate limiting
- Using REST API when HTTP API suffices
- Not validating request models
- Ignoring CORS configuration
- Not monitoring 4xx/5xx error rates

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Must be available BEFORE this node starts:**
- API Functions (this node calls/depends on it via Invoke Lambda (rest))
- Cognito User Pool (this node calls/depends on it via Token Authorization (dependency))

**Depends on THIS node being available:**
- Frontend App (calls this node via API Calls (rest))
- CloudFront (calls this node via Dynamic API Requests (rest))

**Parent Container:** AWS (aws)
