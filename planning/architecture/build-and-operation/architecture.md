# Build and Operation Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`), pp.3–8, and the "Build and Operate" diagram lane. Everything below is `Proposed` until the owner accepts it.

## Scope

**Owns:**
- **Bootstrap.** It's run once by hand and creates the Terraform state bucket (versioned and encrypted, with locking), the GitHub OIDC provider, the CI roles, and the ECR repositories.
- **Terraform layout.** The modules are `data`, `ingestion`, `resumes`, `app`, and `ops`. `envs/dev` and `envs/prod` use the same modules with different settings, and each environment keeps its own state. This area also sets the module conventions (SYS-Q03).
- **The `data` module:** provisions the DynamoDB table exactly as `database` designs it (DB-I01), and the S3 buckets (uploads with event notifications, resumes with the 30-day lifecycle, and the static site).
- **The `ops` module:**
  - the CloudWatch dashboard and alarms
  - the SNS alarm topic and email subscriptions
  - spend alarms
- **CI/CD (`deploy.yml`):**
  - Runs on every push to `main`.
  - Builds the `api`, `scraper`, and `worker` images and pushes them to ECR, tagged with the commit SHA.
  - Runs `terraform apply` for dev.
  - Deploys prod only when started by hand.
  - Authenticates to AWS through OIDC only.
- **IAM conventions.** Each Lambda gets its own role, with access limited to specific resource ARNs.
- **Secrets Manager.** It holds the CloudFront signing key, which only the `api` and `notifier` roles can read.
- **Steps outside Terraform:** the SES production access request, and running the catalog seed script. The catalog JSON itself is owned by `job-discovery-and-matching`.
- **Scaling and throttling settings:** Lambda reserved concurrency, SQS batch size and visibility timeout, maximum receive count before the DLQ, and API Gateway throttling. These matter in Phase 6.

**Does not own:**
- The table design: keys, indexes, TTL, and item types. That's `database`.
- Lambda code and message payloads: each area owns its own.
- Cognito, API Gateway routes, and the CloudFront distribution's behavior: `web-app-and-api`. This area only provides IaC conventions and secrets for them (pending SYS-Q01).
- Bedrock spend accounting (the `COST#` items): `job-discovery-and-matching`. This area alarms on the metric.

## Architecture (as given)

_None yet._

## Architecture Review

_None yet._

## Decisions

_None yet._

## Interfaces

### Provides

#### OPS-I01: Deployment contract
Status: Proposed
Consumers: web-app-and-api, job-discovery-and-matching, experience-bank, delivery
Contract:
- **Images:** every Lambda is a container image in ECR, built from the `api`, `scraper` (Chromium and Playwright), or `worker` (LaTeX) image (OPS-Q03).
- **Image contract:** each image states its handler entry points.
- **Config:** comes from environment variables that Terraform sets: table name, bucket names, queue URLs, and the secret ARN.
- **Environments:** pushes to `main` deploy to dev automatically. Prod deploys only when started by hand.
- **Area resources:** each area's Lambdas, queues, roles, and alarms live in that area's Terraform module (SYS-Q03).
Accepted by: —

#### OPS-I02: Storage resources
Status: Proposed
Consumers: all areas, database
Contract:
- **Table:** one DynamoDB table per environment, provisioned from DB-I01. Its name is passed to services as config (OPS-I01).
- **Buckets:**
  - uploads: private, and fires an event to the Bank parser
  - resumes: private, with a 30-day lifecycle, served through CloudFront OAC
  - static site: served through CloudFront

  Each area owns the key layout of its own prefix (EB-I01, DEL-I01).
Accepted by: —

#### OPS-I03: IAM and secrets
Status: Proposed
Consumers: all areas
Contract: Each Lambda has its own role, granted only the permissions its area lists in the Requirements Intake. The only secret is the CloudFront signing key, stored in Secrets Manager and readable only by `api` and `notifier`. Job boards need no keys, and nothing is stored in git or in GitHub secrets.
Accepted by: —

#### OPS-I04: Observability conventions
Status: Proposed
Consumers: all areas
Contract:
- **Logs:** every Lambda writes JSON logs to CloudWatch, including `sourceId` and `postingKey` (and `sub`/`docId` where relevant), so one posting can be traced from scrape to email.
- **Custom metrics (EMF):**
  - postings found
  - matches
  - PDFs rendered
  - emails sent
  - Bedrock spend per hour
- **Dashboard:** shows those metrics alongside queue depth and Lambda errors.
- **Alarms:**
  - DLQ depth above 0
  - scrape errors
  - API 5xx
  - Bedrock $/hour
  - SES bounces

  All alarms go to SNS, which emails the team.
Accepted by: —

### Consumes
- **The table design:** DB-I01 from `database`.
- **Requirements from every area,** collected below.

### Requirements Intake

One row per need, citing where it came from. Rows taken from the RFC are `Proposed` until the area owner confirms them (OPS-Q07).

| Area | Need | Source | Status |
|---|---|---|---|
| web-app-and-api | Cognito user pool and hosted UI; API Gateway HTTP API with a JWT authorizer; `api` image; CloudFront and the S3 static site; read access to the signing key | RFC p.3–4, 7 | Proposed |
| experience-bank | Uploads bucket with event notifications to the Bank parser; parser Lambda (image and memory per EB-Q01); Bedrock invoke permission; table access for experience items | RFC p.3, 5; EB-I01 | Proposed |
| job-discovery-and-matching | EventBridge Scheduler (per source, catalog, digest); Catalog dispatcher Lambda; SQS scrape, match, and render queues with DLQs; `scraper` image (Chromium); Matcher Lambda; Bedrock invoke permission; catalog seed script | RFC p.3–4, 7 | Proposed |
| delivery | Renderer Lambda (`worker` image with LaTeX); Notifier Lambda; resumes bucket with a 30-day lifecycle; SES identity and production access; read access to the signing key; digest schedule target | RFC p.3–5; DEL-I01 | Proposed |
| database | One table with the keys, GSI1, GSI2, LSI1, and TTL from DB-I01 | RFC p.5; DB-I01 | Proposed |
| all | Spend limits and alarms that email the team | RFC p.8 | Proposed |

## Open Questions

- **OPS-Q01: How should the Terraform state lock work?** The RFC says "a lock file". Recommendation: use the S3 backend's native lockfile (`use_lockfile`, Terraform 1.10 or later), which avoids a separate DynamoDB lock table.
  - Owner: Ryan Fusco
  - Blocks: bootstrap
- **OPS-Q02: What is the budget?** That is the AWS credits or monthly limit, and the thresholds for AWS Budgets and the Bedrock $/hour alarm.
  - Owner: group
  - Blocks: the alarm thresholds, the Phase 5 environments
- **OPS-Q03: Which Lambdas share which image?** Chromium and LaTeX make the images large. Recommendation:
  - `scraper` holds only the Scraper.
  - `worker` holds LaTeX and the Renderer.
  - The parser, Matcher, Notifier, and dispatcher go in a smaller image, or in `worker` if keeping to three images matters more than cold-start time.

  The parser image also depends on EB-Q01.
  - Owner: Ryan Fusco
  - Blocks: OPS-I01
- **OPS-Q04: Where do tests run in CI?** `deploy.yml` builds and applies on every push to `main`, but the diagram says "build · test · deploy", and no tests or pull-request checks are described. Recommendation: add a pull-request workflow that runs unit tests and `terraform plan`, and have `deploy.yml` run tests before it applies.
  - Owner: Ryan Fusco
  - Blocks: OPS-I01
- **OPS-Q05: When do we request SES production access?** Until it's granted, SES only sends to verified addresses. The request goes to AWS support and can take days, so request it early.
  - Owner: Ryan Fusco
  - Blocks: email delivery outside the team
- **OPS-Q06: Which Bedrock model and region,** and is model access enabled in each environment's account?
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: the parser and the Matcher
- **OPS-Q07: Are the RFC-derived Requirements Intake rows right?** Luke Campbell and Graham Cohick need to confirm the rows for their areas.
  - Owner: Luke Campbell, Graham Cohick
  - Blocks: OPS-I02, OPS-I03
