# Task: Application Storage

> **Scope:** implement ONLY this node ("Application Storage"). Work belonging to other nodes appears here solely as interfaces and coordination points — do not implement or re-derive it.
> This document is DERIVED from the NodeSpec model + catalog (fingerprinted, regenerable via generate_task_docs). Node context/export is the model truth; propose model changes through the proposal flow — hand-edits to model facts here do not change the model.

## Component Purpose

**Role:** Object Storage
**Technology:** Amazon S3
**Description:** Blob and object storage for files, images, backups, and unstructured data

## Your Deliverable

This is a provider-managed service — no application code implements it.
- **Provisioning configuration (IaC)** — declare the service (Terraform / CDK / CloudFormation / Helm) as config artifacts: existence, sizing, wiring, permissions.
- **Connection contracts** for every interface below

## Implementation Tasks

Ordered WORK ORDERS synthesized from the model — this node's deliverable kind, contracts, criterion attribution, configuration, and dependency chain. They guarantee coverage, scope, and traceability; they deliberately do NOT contain the implementation detail — that is your job (see the expansion directive below the list).

- [ ] **T1 — Provision Amazon S3 via IaC.**
  Author the provisioning definition as config artifacts bound to this node — existence, wiring, permissions, deployed under AWS.
  [PLACEHOLDER: config — no user configuration recorded for this node; confirm sizing/domains/settings with the user]
- [ ] **T2 — Expose the interface API Functions consumes, per Contract "User File Access" (rest).**
  Record the endpoint/identifiers API Functions needs in this node's config artifacts — coordinate with API Functions.
  Build to the contract schema EXACTLY (see Interface Contracts).
  ↳ serves: REQ-008 "Authenticated users can upload and retrieve application files distinct from the frontend build artifacts" — coordinate with API Functions
  ↳ serves: REQ-008 "The storage bucket is not publicly writable and denies anonymous access by default" — coordinate with API Functions
  ↳ serves: REQ-008 "Access to stored files is mediated by the API layer, not granted directly to clients" — coordinate with API Functions
- [ ] **T3 — Verify every acceptance criterion above and tick its box.**
  Completion evidence flows back to the requirement criteria. This node is complete only when every criterion box is ticked and no `[PLACEHOLDER: …]` tag remains open.

**Your first action — expand these work orders.** Each task above guarantees WHAT must be covered, not HOW. Before writing any code or configuration, expand every task with the concrete implementation steps for THIS technology in THIS project — the specific resources, settings, files, schemas, and tests — using the Configuration, Interface Contracts, Technology Guidance, and node context as your references. Record the expanded list in this section via update_artifact (propose_patches) after this doc is accepted, keeping task IDs, criterion citations, and open `[PLACEHOLDER: …]` tags intact. Resolve placeholders with the user through the proposal flow; this node is never complete while one remains open.

### Platform Capability Equivalence

This node is semantically equivalent to a "AWS S3" (aws-s3) platform_capability node. Treat it as the managed AWS service for spec generation, code scaffolding, and architecture decisions.
- **Equivalent Role:** aws-s3 (AWS S3)
- **Provider:** aws

## Project Context

Verification bench for V2 work: a minimal task API whose requirements intentionally span multiple architecture nodes.

## Requirements — Your Scope

### REQ-008: Application File Storage
Category: functional | Status: in-progress
_Shared with: API Functions — their slices live in their own task docs._
The system shall provide durable object storage for user- and application-generated files, separate from static frontend assets, with access mediated by the API.

**Acceptance criteria — your task boxes:**
- [ ] Authenticated users can upload and retrieve application files distinct from the frontend build artifacts
  → covered by Task T2
- [ ] The storage bucket is not publicly writable and denies anonymous access by default
  → covered by Task T2
- [ ] Access to stored files is mediated by the API layer, not granted directly to clients
  → covered by Task T2

## Interface Contracts

### RECEIVES FROM: API Functions (serverless-function)
- **Contract:** User File Access
- **Protocol:** rest
- **Their Technology:** aws-lambda

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

## Technology Guidance

_Reference for executing the Implementation Tasks above — apply where relevant. The task list stands even where this guidance is thin._

**Purpose:** AWS's object storage service providing virtually unlimited, highly durable (99.999999999%) storage for any type of data. Use for static asset hosting, file uploads, data lake storage, backups, log archival, and as an origin for CloudFront CDN. S3 supports storage classes (Standard, Intelligent-Tiering, Glacier) for cost optimization across access patterns. Its event notification system integrates with Lambda, SQS, and SNS for event-driven processing. S3 is the default choice for storing files, media, and unstructured data in AWS architectures. Don't use for data requiring POSIX filesystem semantics (use EFS instead). Don't use for structured data that needs querying -- use a database. Avoid S3 Standard for archival data accessed less than once per year -- use S3 Glacier Deep Archive for massive cost savings.

**SDK Initialization:**
```
// Node.js with @aws-sdk/client-s3
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
const s3 = new S3Client({ region: process.env.AWS_REGION });
// [Tailor to project language: Python=boto3.client("s3"), Go=s3.NewFromConfig, Java=S3Client.builder()]
```

**Common API Patterns:**

#### Upload Object
Upload file with server-side encryption
```
await s3.send(new PutObjectCommand({
  Bucket: "my-bucket",
  Key: `uploads/${userId}/${filename}`,
  Body: fileBuffer,
  ContentType: contentType,
  ServerSideEncryption: "AES256"
}));
```

#### Presigned URL
Generate time-limited presigned URL for secure download
```
const url = await getSignedUrl(s3, new GetObjectCommand({
  Bucket: "my-bucket",
  Key: `uploads/${userId}/${filename}`
}), { expiresIn: 3600 });
// Return url to client for temporary download access
```

#### List Objects
List objects by prefix with pagination
```
import { ListObjectsV2Command } from "@aws-sdk/client-s3";
const { Contents = [] } = await s3.send(new ListObjectsV2Command({
  Bucket: "my-bucket",
  Prefix: `uploads/${userId}/`,
  MaxKeys: 100
}));
```

#### Delete Object
Delete a single object
```
import { DeleteObjectCommand } from "@aws-sdk/client-s3";
await s3.send(new DeleteObjectCommand({ Bucket: "my-bucket", Key: objectKey }));
```

**Configuration Template:**
```
resource "aws_s3_bucket" "assets" {
  bucket = "${var.project}-assets"
}
resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration { status = "Enabled" }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "assets" {
  bucket = aws_s3_bucket.assets.id
  rule { apply_server_side_encryption_by_default { sse_algorithm = "AES256" } }
}
resource "aws_s3_bucket_public_access_block" "assets" {
  bucket                  = aws_s3_bucket.assets.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

**Best Practices:**
- Block all public access by default and use presigned URLs for temporary access
- Enable versioning for buckets storing important data to protect against accidental deletion
- Use S3 Lifecycle rules to transition objects to cheaper storage classes automatically
- Use server-side encryption (SSE-S3 or SSE-KMS) for all buckets
- Use multipart upload for files larger than 100MB
- Enable S3 access logging or CloudTrail for audit trails
- Use intelligent-tiering for unpredictable access patterns to optimize costs automatically

**Anti-Patterns to Avoid:**
- Leaving buckets publicly accessible without a clear, justified reason
- Not enabling versioning for buckets storing user-uploaded content
- Storing all data in S3 Standard when lifecycle policies could save costs
- Using S3 for real-time database operations requiring low-latency queries
- Not encrypting buckets containing any form of user or business data
- Uploading large files without multipart upload causing timeout failures

**Security:** Enable S3 Block Public Access at the account level as a guardrail. Use bucket policies with least-privilege IAM principals. Enable server-side encryption on all buckets. Use VPC endpoints for S3 access from private subnets. Use presigned URLs for temporary access instead of making objects public. Enable MFA Delete for versioned buckets containing critical data.

**Integration Patterns:**
- CloudFront as CDN origin for global static asset delivery
- S3 Event Notifications + Lambda for serverless file processing pipelines
- S3 Lifecycle rules for automated archival to Glacier for long-term storage

## Manual Steps

> The following steps require manual action by a human. AI cannot complete these steps automatically.

**Quick checklist:**
- [ ] Create S3 Bucket *(required)*
- [ ] Configure Bucket Access *(required)*
- [ ] Create IAM User or Role *(required)*
- [ ] Set Environment Variables *(required)*
- [ ] Configure CORS for Browser Uploads *(optional)*

### Required Steps

#### [dashboard_config] Create S3 Bucket

In AWS Console > S3 > Create Bucket. Choose a globally unique name and region. Enable or disable versioning based on your needs.

**Reference:** https://console.aws.amazon.com/s3/

#### [permissions] Configure Bucket Access

Set the bucket to private by default. Configure a bucket policy for least-privilege access. For public assets, use CloudFront distribution instead of making the bucket public.

#### [permissions] Create IAM User or Role

Create an IAM user (for server-side) or IAM role (for AWS services) with minimum S3 permissions: s3:GetObject, s3:PutObject, s3:DeleteObject on the specific bucket ARN.

#### [environment_variable] Set Environment Variables

Add AWS credentials and bucket info to your application environment.

```bash
export AWS_ACCESS_KEY_ID=<from-iam-user>
export AWS_SECRET_ACCESS_KEY=<from-iam-user>
export AWS_S3_BUCKET_NAME=my-app-uploads
export AWS_REGION=us-east-1
```

### Optional Steps

#### [dashboard_config] Configure CORS for Browser Uploads

Under Permissions > CORS, add a CORS configuration allowing your application origin to perform PUT/POST for direct uploads from the browser.

## Dependency Chain

Startup/initialization order based on edge directions and interaction patterns.

**Depends on THIS node being available:**
- API Functions (calls this node via User File Access (rest))

**Parent Container:** AWS (aws)
