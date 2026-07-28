# Task: AWS WAF

> **Scope:** implement ONLY this node ("AWS WAF"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Load Balancer
**Technology:** AWS WAF
**Description:** Traffic distribution across service instances

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

- [ ] **T1 — Provision AWS WAF via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  **Configuration (account-wide baseline per REQ-019/REQ-011 — tune after observing real traffic):**
  - Scope: CLOUDFRONT (Web ACL associated with the CloudFront distribution; must be created in us-east-1 regardless of app region — CloudFront WAF requirement)
  - Managed rule groups: AWSManagedRulesCommonRuleSet (OWASP Core) + AWSManagedRulesAmazonIpReputationList — deliberately NOT stacking additional groups, per REQ-011's cost/protection balance
  - Deployment mode: Count for the first 1–2 weeks in each new environment to catch false positives, then switch to Block
  - Rate-based rule: 2000 requests / 5-minute sliding window per source IP as the starting throttle
  - Logging: enabled, delivered to S3 (via Kinesis Data Firehose) for audit and tuning
- [ ] **T2 — Expose the interface CloudFront consumes, per Contract "WAF Protection" (dependency).**
  Record the endpoint/identifiers CloudFront needs in this node's config artifacts — coordinate with CloudFront.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
  ↳ serves (unverified match): REQ-011 "WAF is scoped to essential managed rule groups (e.g., core rule set, IP reputation) rather than stacking unnecessary rule groups, balancing protection against cost" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T3 — Configure the service to satisfy: "Public-facing traffic is filtered by a web application firewall against common exploit patterns before reaching compute" (REQ-011).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-011 "Public-facing traffic is filtered by a web application firewall against common exploit patterns before reaching compute"
- [ ] **T4 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-011: Perimeter Security (WAF)
Category: non-functional | Status: in-progress
The system shall enforce perimeter security via a scoped web application firewall filtering public-facing traffic against common exploit patterns.

**Acceptance criteria — your task boxes:**
- [ ] Public-facing traffic is filtered by a web application firewall against common exploit patterns before reaching compute
  → covered by Task T3
- [ ] WAF is scoped to essential managed rule groups (e.g., core rule set, IP reputation) rather than stacking unnecessary rule groups, balancing protection against cost
  → covered by Task T2 / T1

## Interface Contracts

### RECEIVES FROM: CloudFront (cdn)
- **Contract:** WAF Protection
- **Protocol:** dependency
- **Their Technology:** aws-cloudfront

**Schema:**
```
{
  "type": "reference",
  "reference": "webAclArn",
  "description": "Dependency contract, no payload. Represents a Web ACL association attached to the CloudFront distribution, not a network hop."
}
```

## Technology Guidance

**Purpose:** Web application firewall that protects web applications from common exploits and bots. Attaches to ALB, API Gateway, CloudFront, or AppSync to filter HTTP/HTTPS requests. Provides managed rule groups (OWASP Top 10, bot control, IP reputation) and custom rules using rate limiting, geo-matching, and regex patterns.

**Best Practices:**
- Use AWS Managed Rules as a baseline (AWSManagedRulesCommonRuleSet for OWASP Top 10)
- Enable logging to S3 or CloudWatch for audit and debugging
- Start with Count mode before switching rules to Block to avoid false positives
- Use rate-based rules to protect against DDoS and brute-force attacks
- Apply IP reputation lists to block known malicious sources
- Create custom rules for application-specific protections

**Anti-Patterns to Avoid:**
- Deploying rules directly in Block mode without testing in Count mode first
- Not monitoring WAF metrics and logs for false positives
- Applying WAF only to ALB while leaving CloudFront or API Gateway unprotected
- Creating overly broad rules that block legitimate traffic

**Suggested File Structure:**
- `terraform/waf.tf` (config)
- `terraform/waf-logging.tf` (config)

## Dependency Chain

**Depends on THIS node being available:**
- CloudFront (initiates WAF Protection against this node (dependency))

**Parent Container:** AWS (aws)
