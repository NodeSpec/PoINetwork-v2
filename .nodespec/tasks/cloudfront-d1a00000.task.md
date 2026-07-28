# Task: CloudFront

> **Scope:** implement ONLY this node ("CloudFront"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** CDN
**Technology:** AWS CloudFront
**Description:** Content delivery network edge node

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Provision AWS CloudFront via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  [PLACEHOLDER: config — no user configuration recorded for this node; confirm sizing/domains/settings with the user]
- [ ] **T2 — Declare the wiring to Frontend Static Assets (aws-s3) per Contract "CDN Static Origin" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves: REQ-004 "Static assets are cached at edge locations via CDN" — coordinate with Frontend Static Assets
  ↳ serves: REQ-004 "Static assets are served from object storage, not the application compute layer" — coordinate with Frontend Static Assets
- [ ] **T3 — Declare the wiring to Frontend App (react) per Contract "CDN Delivery" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
- [ ] **T4 — Declare the wiring to API Gateway (aws-api-gateway) per Contract "Dynamic API Requests" (rest).**
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves (unverified match): REQ-012 "HTTP requests are redirected to HTTPS" — requirement not mapped to that node; verify or reassign before relying on it
- [ ] **T5 — Declare the wiring to AWS WAF (aws-waf) per Contract "WAF Protection" (dependency).**
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T6 — Expose the interface Route 53 consumes, per Contract "DNS Resolution" (dependency).**
  Record the endpoint/identifiers Route 53 needs in this node's config artifacts — coordinate with Route 53.
  Dependency contract — capture the reference/identifier wiring in this node's config artifacts; no payload schema expected.
- [ ] **T7 — Configure the service to satisfy: "All traffic is served over HTTPS with a valid TLS certificate" (REQ-012).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-012 "All traffic is served over HTTPS with a valid TLS certificate"
- [ ] **T8 — Configure the service to satisfy: "Cache hit ratio is measurable and reported" (REQ-016).**
  No interface contract maps to this criterion — it is this node's internal responsibility.
  ↳ serves: REQ-016 "Cache hit ratio is measurable and reported"
- [ ] **T9 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-004: Static Asset Edge Delivery
Category: non-functional | Status: in-progress
_Shared with: Frontend Static Assets — their slices live in their own task docs._
Static assets shall be cached at CDN edge locations and served from object storage, independent of application compute load.

**Acceptance criteria — your task boxes:**
- [ ] Static assets are cached at edge locations via CDN
  → covered by Task T2
- [ ] Static assets are served from object storage, not the application compute layer
  → covered by Task T2

### REQ-012: HTTPS Enforcement
Category: non-functional | Status: in-progress
The system shall serve all traffic over HTTPS with a valid TLS certificate and redirect HTTP requests to HTTPS.

**Acceptance criteria — your task boxes:**
- [ ] All traffic is served over HTTPS with a valid TLS certificate
  → covered by Task T7
- [ ] HTTP requests are redirected to HTTPS
  → covered by Task T4

### REQ-016: CDN Cache Hit Ratio Observability
Category: non-functional | Status: in-progress
The CDN distribution shall expose cache hit ratio as a measurable, reportable metric.

**Acceptance criteria — your task boxes:**
- [ ] Cache hit ratio is measurable and reported
  → covered by Task T8

## Interface Contracts

### RECEIVES FROM: Route 53 (dns)
- **Contract:** DNS Resolution
- **Protocol:** dependency
- **Their Technology:** aws-route53

**Schema:**
```
{
  "type": "reference",
  "reference": "cloudfrontDistributionDomainName",
  "description": "Dependency contract, no payload. Represents an alias record referencing the CloudFront distribution's domain name, per AWS's standard alias mechanism."
}
```

### SENDS TO: Frontend Static Assets (object-storage)
- **Contract:** CDN Static Origin
- **Protocol:** rest
- **Their Technology:** aws-s3

**Schema:**
```
{
  "path": "/{proxy+}",
  "type": "http_proxy",
  "method": "GET",
  "response": {
    "description": "Raw object bytes with Content-Type inferred from S3 object metadata"
  },
  "description": "CloudFront forwards static asset GET requests to the S3 origin via Origin Access Control. Standard HTTP object retrieval, no custom payload."
}
```

### SENDS TO: Frontend App (frontend-app)
- **Contract:** CDN Delivery
- **Protocol:** rest
- **Their Technology:** react

**Schema:**
```
{
  "path": "/*",
  "type": "http_static_delivery",
  "method": "GET",
  "response": {
    "contentType": "text/html | application/javascript | text/css | image/*",
    "cacheControl": "per-asset (immutable for hashed bundles, no-cache for index.html)"
  },
  "description": "CloudFront serves the React SPA build artifacts from the S3 origin to the end user's browser. The SPA then runs client-side."
}
```

### SENDS TO: API Gateway (api-gateway)
- **Contract:** Dynamic API Requests
- **Protocol:** rest
- **Their Technology:** aws-api-gateway

**Schema:**
```
{
  "path": "/api/{proxy+}",
  "type": "http_proxy",
  "method": "ANY",
  "description": "CloudFront forwards all /api/* requests unmodified to API Gateway; no caching applied to this path pattern."
}
```

### SENDS TO: AWS WAF (load-balancer)
- **Contract:** WAF Protection
- **Protocol:** dependency
- **Their Technology:** aws-waf

**Schema:**
```
{
  "type": "reference",
  "reference": "webAclArn",
  "description": "Dependency contract, no payload. Represents a Web ACL association attached to the CloudFront distribution, not a network hop."
}
```

## Technology Guidance

_Reference for executing the Implementation Tasks above — apply where relevant. The task list stands even where this guidance is thin._

**Purpose:** AWS's global content delivery network (CDN) that caches and delivers content from edge locations worldwide. Use when you need low-latency delivery of static assets (images, CSS, JS), dynamic API acceleration, or DDoS protection at the edge. CloudFront integrates natively with S3, ALB, API Gateway, and Lambda@Edge/CloudFront Functions for edge compute. Ideal for web applications needing global performance, video streaming, and any workload where proximity to end users matters. CloudFront also provides SSL/TLS termination at the edge with AWS Certificate Manager. Don't use when your users are all in a single region and latency is already acceptable -- a simple ALB or S3 with Transfer Acceleration may suffice. Don't use for real-time bidirectional communication (WebSockets) unless you specifically configure it. Avoid when cost sensitivity is extreme and CloudFlare or a simpler CDN would meet your needs at lower cost.

**SDK Initialization:**
```
# AWS CLI create distribution
aws cloudfront create-distribution --distribution-config file://dist-config.json
# Terraform is preferred for CloudFront configuration (see configurationTemplate)
```

**Common API Patterns:**

#### Create Invalidation
Invalidate all cached content after a deployment
```
aws cloudfront create-invalidation --distribution-id E1234567890 --paths "/*"
```

#### CloudFront Function
CloudFront Function for SPA URL rewriting
```
function handler(event) {
  var request = event.request;
  var uri = request.uri;
  // SPA routing: rewrite paths without extensions to /index.html
  if (!uri.includes('.')) {
    request.uri = '/index.html';
  }
  return request;
}
```

#### Signed URL
Generate signed URL for private content access
```
import { getSignedUrl } from "@aws-sdk/cloudfront-signer";
const signedUrl = getSignedUrl({
  url: `https://d123.cloudfront.net/private/file.pdf`,
  keyPairId: process.env.CF_KEY_PAIR_ID!,
  privateKey: process.env.CF_PRIVATE_KEY!,
  dateLessThan: new Date(Date.now() + 3600000).toISOString()
});
```

**Configuration Template:**
```
resource "aws_cloudfront_distribution" "main" {
  enabled             = true
  default_root_object = "index.html"
  aliases             = ["app.example.com"]
  origin {
    domain_name              = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id                = "s3-assets"
    origin_access_control_id = aws_cloudfront_origin_access_control.main.id
  }
  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "s3-assets"
    cache_policy_id        = "658327ea-f89d-4fab-a63d-7e88639e58f6" # CachingOptimized
    viewer_protocol_policy = "redirect-to-https"
    compress               = true
  }
  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.main.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }
  restrictions {
    geo_restriction { restriction_type = "none" }
  }
}
```

**Best Practices:**
- Use Origin Access Control (OAC) to restrict S3 access exclusively through CloudFront
- Set appropriate Cache-Control headers on origins to control edge caching behavior
- Use cache policies and origin request policies instead of legacy cache behavior settings
- Enable compression (gzip, brotli) for text-based content
- Use Lambda@Edge or CloudFront Functions for URL rewriting, auth, and A/B testing
- Configure custom error pages for 4xx/5xx responses
- Use CloudFront access logs in S3 for traffic analysis and debugging

**Anti-Patterns to Avoid:**
- Using CloudFront without Origin Access Control allowing direct S3 access
- Setting excessively long TTLs for dynamic content causing stale data
- Not invalidating cache after deployments leaving users on old versions
- Forwarding unnecessary headers/cookies to origin defeating cache effectiveness
- Using a single behavior for all content instead of separating static and dynamic paths
- Not enabling compression wasting bandwidth on text assets

**Security:** Use Origin Access Control (OAC) to prevent direct access to S3 origins. Attach AWS WAF to CloudFront for bot protection, rate limiting, and IP filtering. Enforce HTTPS with redirect-to-https viewer protocol policy. Use TLS 1.2+ minimum protocol version. Use signed URLs or signed cookies for private content. Set appropriate security headers via response headers policy (CSP, HSTS, X-Frame-Options).

**Integration Patterns:**
- S3 + CloudFront for static website hosting with global edge delivery
- ALB + CloudFront for dynamic API acceleration with edge caching
- AWS WAF for web application firewall protection at the CDN edge

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Must be available BEFORE this node starts:**
- Frontend Static Assets (this node calls/depends on it via CDN Static Origin (rest))
- Frontend App (this node calls/depends on it via CDN Delivery (rest))
- API Gateway (this node calls/depends on it via Dynamic API Requests (rest))
- AWS WAF (this node calls/depends on it via WAF Protection (dependency))

**Depends on THIS node being available:**
- Route 53 (initiates DNS Resolution against this node (dependency))

**Parent Container:** AWS (aws)
