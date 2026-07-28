# Task: Cognito User Pool

> **Scope:** implement ONLY this node ("Cognito User Pool"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Auth Provider
**Technology:** AWS Cognito
**Description:** Authentication and identity management service

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Provision AWS Cognito via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  **Configuration (account-wide baseline per REQ-019 — adjust if this app has stricter needs):**
  - Password policy: minimum length 14; require uppercase, lowercase, number, and symbol
  - MFA: optional (software TOTP) for standard end users; required for any user granted elevated/admin app roles
  - Account recovery: self-service via verified email
  - Advanced Security Features: enabled (adaptive authentication, compromised-credential detection)
  - App client: public SPA client (no client secret); access/ID token validity 1 hour; refresh token validity 30 days
  - Deployment region: us-east-1 (change to match your primary region decision)
  - This baseline aligns with the account-wide password/MFA conventions documented on the AWS root node (REQ-019); it governs Cognito's own end-user policy and does not inherit or override the separate IAM-console-user policy defined there.
- [ ] **T2 — Expose the interface Frontend App consumes, per Contract "Auth SDK" (dependency).**
  Record the endpoint/identifiers Frontend App needs in this node's config artifacts — coordinate with Frontend App.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T3 — Expose the interface API Gateway consumes, per Contract "Token Authorization" (dependency).**
  Record the endpoint/identifiers API Gateway needs in this node's config artifacts — coordinate with API Gateway.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T4 — Configure the service to satisfy: "Users can sign up, sign in, and sign out via a managed identity provider" (REQ-017).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-017 "Users can sign up, sign in, and sign out via a managed identity provider"
- [ ] **T5 — Configure the service to satisfy: "Passwords and credentials are never stored or handled directly by application code" (REQ-017).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-017 "Passwords and credentials are never stored or handled directly by application code"
- [ ] **T6 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-017: User Identity Management
Category: functional | Status: in-progress
The system shall provide user identity management via a managed identity provider — sign up, sign in, sign out — without the application ever handling raw credentials.

**Acceptance criteria — your task boxes:**
- [ ] Users can sign up, sign in, and sign out via a managed identity provider
  → covered by Task T4
- [ ] Passwords and credentials are never stored or handled directly by application code
  → covered by Task T5

## Interface Contracts

### RECEIVES FROM: Frontend App (frontend-app)
- **Contract:** Auth SDK
- **Protocol:** dependency
- **Their Technology:** react

**Schema:**
```
{
  "type": "sdk_integration",
  "library": "@aws-amplify/auth or amazon-cognito-identity-js",
  "operations": ["signUp", "signIn", "signOut", "getCurrentUser", "fetchAuthSession"],
  "description": "React SPA integrates Cognito via the AWS Amplify Auth SDK or amazon-cognito-identity-js for sign up, sign in, sign out, and JWT token retrieval."
}
```

### RECEIVES FROM: API Gateway (api-gateway)
- **Contract:** Token Authorization
- **Protocol:** dependency
- **Their Technology:** aws-api-gateway

**Schema:**
```
{
  "type": "reference",
  "reference": {"issuer": "Cognito User Pool URL", "audience": "App Client ID"},
  "description": "Dependency contract, no payload. Represents a JWT authorizer configuration referencing the Cognito User Pool's issuer and app client ID."
}
```

## Technology Guidance

**Purpose:** AWS managed identity service providing User Pools for user directory and authentication, Identity Pools for AWS credential federation, Lambda triggers for custom workflows, and deep integration with API Gateway, ALB, and other AWS services

**Best Practices:**
- Use User Pools for user directory management and authentication
- Use Identity Pools only when AWS credential federation is needed
- Configure proper password policies and account recovery
- Use Lambda triggers (pre-signup, post-confirmation, custom-message) for custom workflows
- Implement MFA for sensitive applications
- Use Cognito Groups for role-based access with IAM role mapping
- Deploy infrastructure as code with CDK or Terraform
- Use Advanced Security Features for adaptive authentication
- Configure proper app client settings (token validity, OAuth scopes)
- Implement proper token refresh handling in the client

**Anti-Patterns to Avoid:**
- Using Identity Pools when User Pools alone suffice
- Not implementing MFA for production applications
- Complex Lambda trigger chains that create brittle auth flows
- Not configuring account recovery options
- Hardcoding User Pool IDs in application code
- Not handling token refresh properly
- Using admin-level SDK calls from client-side code

**Suggested File Structure:**
- `infra/cognito/user-pool.tf` (config)
- `infra/cognito/identity-pool.tf` (config)
- `infra/cognito/triggers.tf` (config)
- `src/auth/cognito-config.ts` (config)
- `src/auth/cognito-provider.tsx` (source)
- `lambdas/pre-signup/index.ts` (source)
- `lambdas/post-confirmation/index.ts` (source)
- `lambdas/custom-message/index.ts` (source)

## Manual Steps

**Quick checklist:**
- [ ] Create User Pool *(required)*
- [ ] Create App Client *(required)*
- [ ] Set Environment Variables *(required)*
- [ ] Configure Identity Pool (Optional) *(optional)*

### Required Steps

#### [dashboard_config] Create User Pool
Create a Cognito User Pool in the AWS Console. Configure sign-in options (email, phone, username) and password policies per the baseline in T1.

**Reference:** https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools.html

#### [dashboard_config] Create App Client
Create an App Client in the User Pool settings. Configure OAuth 2.0 grant types and callback URLs.

#### [environment_variable] Set Environment Variables
```bash
export COGNITO_USER_POOL_ID=us-east-1_xxxxx
export COGNITO_CLIENT_ID=your-client-id
export COGNITO_REGION=us-east-1
```

### Optional Steps

#### [dashboard_config] Configure Identity Pool (Optional)
If you need AWS service access from the client, create an Identity Pool linked to the User Pool for temporary AWS credentials.

## Dependency Chain

**Depends on THIS node being available:**
- Frontend App (initiates Auth SDK against this node (dependency))
- API Gateway (initiates Token Authorization against this node (dependency))

**Parent Container:** AWS (aws)
