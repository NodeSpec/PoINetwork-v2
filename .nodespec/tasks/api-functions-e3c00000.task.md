# Task: API Functions

> **Scope:** implement ONLY this node ("API Functions"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Serverless Function
**Technology:** AWS Lambda
**Description:** Serverless or edge-deployed function (AWS Lambda, Cloudflare Workers, Deno Deploy, etc.)

## Your Deliverable

**Working code for this component**, honoring the contracts and criteria below, plus its configuration artifacts and tests.

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Scaffold the AWS Lambda component.**
  Create the source layout, build files, and test harness this node's working code lives in.
  Start from the catalog's suggested structure: `src/handlers/index.ts`, `infra/aws/lambda.tf`.
- [ ] **T2 — Implement the integration with Primary Database (aws-rds-postgresql) per Contract "App Data Queries" (sql).**
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-008 "Authenticated users can upload and retrieve application files distinct from the frontend build artifacts" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T3 — Implement the integration with Application Storage (aws-s3) per Contract "User File Access" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves: REQ-008 "The storage bucket is not publicly writable and denies anonymous access by default" — coordinate with Application Storage
  ↳ serves: REQ-008 "Access to stored files is mediated by the API layer, not granted directly to clients" — coordinate with Application Storage
- [ ] **T4 — Implement the integration with Job Queue (rabbitmq) per Contract "Enqueue Heavy Job" (custom).**
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T5 — Implement the integration with Secrets Manager (aws-secrets-manager) per Contract "Retrieve DB Credentials" (dependency).**
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T6 — Expose the interface API Gateway consumes, per Contract "Invoke Lambda" (rest).**
  Record the endpoint/identifiers API Gateway needs in this node's config artifacts — coordinate with API Gateway.
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T7 — Implement: "Standard CRUD/business-logic requests are handled by serverless functions with no idle compute cost" (REQ-009).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-009 "Standard CRUD/business-logic requests are handled by serverless functions with no idle compute cost"
- [ ] **T8 — Implement: "Function execution is stateless and safe to run concurrently" (REQ-009).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-009 "Function execution is stateless and safe to run concurrently"
- [ ] **T9 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-009: Serverless Compute Properties
Category: technical | Status: in-progress
Business logic shall run as serverless functions with no idle compute cost, executing statelessly and safely under concurrent invocation.

**Acceptance criteria — your task boxes:**
- [ ] Standard CRUD/business-logic requests are handled by serverless functions with no idle compute cost
  → covered by Task T7
- [ ] Function execution is stateless and safe to run concurrently
  → covered by Task T8

### REQ-008: Application File Storage
Category: functional | Status: in-progress
_Shared with: Application Storage — their slices live in their own task docs._
The system shall provide durable object storage for user- and application-generated files, separate from static frontend assets, with access mediated by the API.

**Acceptance criteria — your task boxes:**
- [ ] Authenticated users can upload and retrieve application files distinct from the frontend build artifacts
  → covered by Task T2
- [ ] The storage bucket is not publicly writable and denies anonymous access by default
  → covered by Task T3
- [ ] Access to stored files is mediated by the API layer, not granted directly to clients
  → covered by Task T3

## Interface Contracts

### RECEIVES FROM: API Gateway (api-gateway)
- **Contract:** Invoke Lambda
- **Protocol:** rest
- **Their Technology:** aws-api-gateway

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

### SENDS TO: Primary Database (database)
- **Contract:** App Data Queries
- **Protocol:** sql
- **Their Technology:** aws-rds-postgresql

**Schema:**
```
{
  "type": "connection_contract",
  "description": "Access-pattern contract only — no table/data schema is defined anywhere in the spec; do not fabricate one. Enforces parameterized queries only (no string-concatenated SQL) and no direct DDL access from this role.",
  "authentication": "Credentials retrieved via the Secrets Manager dependency contract, not hardcoded"
}
```

### SENDS TO: Application Storage (object-storage)
- **Contract:** User File Access
- **Protocol:** rest
- **Their Technology:** aws-s3

**Schema:**
```
{
  "type": "presigned_url_exchange",
  "request": {
    "userId": "string, from validated JWT claim",
    "operation": "upload|download",
    "objectKeyPrefix": "users/{userId}/"
  },
  "response": {
    "presignedUrl": "string",
    "expiresInSeconds": "number"
  },
  "operations": [
    "generate_upload_url",
    "generate_download_url"
  ],
  "description": "API Functions never proxies file bytes; it issues time-limited presigned S3 URLs scoped to the authenticated user's key prefix."
}
```

### SENDS TO: Job Queue (queue)
- **Contract:** Enqueue Heavy Job
- **Protocol:** custom
- **Their Technology:** rabbitmq

**Schema:**
```
{
  "type": "message_envelope",
  "fields": {
    "jobId": "string (UUID, generated by sender for idempotency)",
    "jobType": "string — extensible, no fixed enum defined yet",
    "payload": "object — job-type-specific, intentionally not fabricated here",
    "enqueuedAt": "ISO 8601 timestamp"
  },
  "format": "JSON",
  "description": "Generic extensible envelope. Specific job types/payloads must be proposed via propose_patches when a concrete job type is introduced."
}
```

### SENDS TO: Secrets Manager (auth-provider)
- **Contract:** Retrieve DB Credentials
- **Protocol:** dependency
- **Their Technology:** aws-secrets-manager

**Schema:**
```
{
  "type": "reference",
  "reference": "secretArn (environment variable, not hardcoded)",
  "description": "Dependency contract, no payload. API Functions retrieves the DB credential by ARN at cold start and caches it for the container's lifetime."
}
```

## Technology Guidance

_Reference for executing the Implementation Tasks above — apply where relevant. The task list stands even where this guidance is thin._

**Purpose:** Serverless compute service that runs code in response to events and automatically manages the underlying compute resources. Supports Node.js, Python, Java, Go, .NET, Ruby, and custom runtimes.

**Best Practices:**
- Keep functions small and single-purpose
- Use environment variables for configuration
- Implement proper error handling and dead letter queues
- Use Lambda Layers for shared dependencies
- Set appropriate memory allocation for cost/performance balance
- Use provisioned concurrency for latency-sensitive workloads
- Use Step Functions for orchestrating multiple Lambda invocations
- Monitor with CloudWatch and X-Ray tracing

**Anti-Patterns to Avoid:**
- Monolithic Lambda functions doing too many things
- Not setting appropriate memory and timeout limits
- Storing state inside the function
- Not handling cold starts
- Synchronous invocation chains across multiple Lambdas
- Ignoring concurrency limits

**Suggested File Structure:**
- `src/handlers/index.ts` (source)
- `infra/aws/lambda.tf` (config)

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Must be available BEFORE this node starts:**
- Primary Database (this node calls/depends on it via App Data Queries (sql))
- Application Storage (this node calls/depends on it via User File Access (rest))
- Job Queue (this node calls/depends on it via Enqueue Heavy Job (custom))
- Secrets Manager (this node calls/depends on it via Retrieve DB Credentials (dependency))

**Depends on THIS node being available:**
- API Gateway (calls this node via Invoke Lambda (rest))

**Parent Container:** VPC (vpc)
