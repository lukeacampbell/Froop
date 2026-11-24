# Build and Operation Implementation Plan

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This is a shared area, so every owner must agree to changes to this plan.
Date: 2026-11-19, revised 2026-11-23. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: decisions OPS-001 to OPS-012 and interfaces OPS-I01 to OPS-I05. OPS-001 to OPS-012 and OPS-I05 are Accepted as of 2026-11-21, and OPS-I01 to I04 as of 2026-11-23. OPS-013 (dev only) is Proposed. No infrastructure exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision or interface still needs agreement before the task can be built. It doesn't mean the task was tried and failed.

**Order.**
1. The repository (T01) and bootstrap (T02) come first, because every area's deploys depend on them.
2. Then modules and environments (T03), data resources (T04), images (T05), and CI (T06).
3. Then IAM and secrets (T07), observability (T08), and the budget (T09).
4. The manual steps (T10) are written down alongside T02 and done before the first demo.

**Tests.** Terraform changes are checked by `terraform validate` and `plan` in CI (T06). Each task's acceptance criteria are checked against a deployed dev environment in Phase 2. Cross-area flows belong to the Phase 3 test plans.

| Waiting on | Tasks it blocks |
|---|---|
| Luke Campbell accepting OPS-013 (dev only) | Only the prod parts of T02, T03, and T06. Each would be dropped. |
| Luke Campbell accepting OPS-014 (signing key made by hand) | The signing-key step in T10 |

## Foundations

### OPS-T01: Set up the repository
Depends on: OPS-007, OPS-012
Acceptance criteria:
1. Top-level folders: `services/<name>/` (one per Lambda image), `libs/<name>/` (shared Python packages), `infra/` (Terraform), and `web/` (the dashboard).
2. Terraform's version is pinned to 1.10 or later in `infra/`, and Python's version is pinned for every service.
3. A README explains how to run the unit tests for one service, for one library, and for all of them.
4. Every folder in the Code Map of the repository's `AGENTS.md` gets a short `AGENTS.md` naming its area and owner, its tasks, the interfaces it provides and consumes, and a pointer to that area's `planning/architecture/<area>/AGENTS.md`. *(Added 2026-11-23 by Ryan Fusco and Graham Cohick; waiting for Luke Campbell.)*
Status: Planned

### OPS-T02: Bootstrap each AWS account
Depends on: OPS-T01, OPS-001, OPS-004, OPS-005, OPS-006
Acceptance criteria:
1. A bootstrap configuration, run once by hand, creates the Terraform state bucket (versioned and encrypted, with native lockfile locking), the GitHub OIDC provider, a deploy role, a read-only plan role, and seven ECR repositories (`api`, `scraper`, `renderer`, `parser`, `matcher`, `notifier`, `dispatcher`).
2. Each ECR repository has a lifecycle policy that keeps the last 10 images.
3. Everything is in `us-east-1`.
4. Running bootstrap a second time changes nothing.
5. Two people applying at the same time can't corrupt the state. A test confirms the second apply waits or fails on the lock.
Status: Planned

### OPS-T03: Terraform modules and environments
Depends on: OPS-T02, SYS-Q03, OPS-011, OPS-002, OPS-006, OPS-009, OPS-010
Acceptance criteria:
1. **Five modules:**
   - `data`: the table and buckets
   - `ingestion`: dispatcher, scraper, Matcher, and their queues
   - `resumes`: parser, Renderer, Notifier, and the digest schedules
   - `app`: Cognito, API Gateway, the API, and CloudFront
   - `ops`: the dashboard, alarms, SNS, and the budget

   This task creates each module's skeleton (inputs, outputs, and conventions). Each area fills its own module in its own tasks: EB-T07 and DEL-T07 (`resumes`), WEB-T02 to WEB-T04 (`app`), JDM-T10 (`ingestion`).
2. `envs/dev` and `envs/prod` use the same modules with different `tfvars`.
3. Every setting is a variable in the environment's `tfvars`, with no code change needed:
   - the cost caps (`cost_cap_user_per_hour`, `cost_cap_platform_per_hour`, `cost_cap_platform_per_day`)
   - the budget (`monthly_budget_usd`, `budget_alert_percents`)
   - the Bedrock model for each task
   - the sign-up switch and CORS origins (from `web-app-and-api`)
   - log retention, ECR retention, API throttling, and the SES sender address (OPS-015, once Accepted)
   - the parser's leftover limits: `PARSER_MAX_LEFTOVER_SECTIONS`, `PARSER_MAX_SECTION_CHARS`, `PARSER_MAX_OUTPUT_TOKENS`, `PARSER_BEDROCK_TIMEOUT_SECONDS` (EB-024)
   - the parser Lambda's timeout, memory, and asynchronous retry attempts (EB-025)
4. Each module exposes the names its services need (the table, buckets, queues, and secret ARN) as outputs, passed to Lambdas as environment variables (OPS-I01).
Status: Planned

### OPS-T04: Data resources
Depends on: OPS-T03, DB-T01, OPS-I02, EB-018, DEL-007
Acceptance criteria:
1. The DynamoDB table is created exactly as DB-T01 defines it, including GSI1 to GSI3, TTL, and on-demand capacity.
2. **Uploads bucket:** private, accepts presigned POST uploads from the dashboard's origin, and sends object-created events for `uploads/` to the parser.
3. **Resumes bucket:** private, with no expiry rule. An optional move to Standard-IA after 30 days is allowed. Served only through CloudFront OAC.
4. **Static-site bucket:** private, served only through CloudFront.
5. Public access is blocked on all three buckets.
6. The table has deletion protection on (OPS-015, once Accepted).
Status: Planned

### OPS-T05: Images and the deployment contract
Depends on: OPS-T01, OPS-T02, OPS-003, OPS-004, OPS-011, OPS-I01
Acceptance criteria:
1. A shared base image (Python plus `libs/`) is built first. Each function's image adds only its own code and dependencies.
2. The `scraper` image has no browser. The `renderer` image is the only one with TeX.
3. Each image declares its handler entry point and reads all configuration from environment variables. A missing variable makes the function fail at start-up with a clear message.
4. Images are tagged with the commit SHA.
Status: Planned

### OPS-T06: CI and deployment workflows
Depends on: OPS-T01, OPS-T02, OPS-T05, OPS-005, OPS-012
Acceptance criteria:
1. **`ci.yml` (every pull request):** linting; unit tests for the services, `libs/`, and `web/` that changed; and `terraform fmt -check`, `validate`, and `plan` for dev with the read-only role.
2. **`deploy.yml` (push to `main`):**
   - unit tests
   - build and push the changed images
   - `terraform apply` for dev
   - build the dashboard with dev's settings, upload it, and invalidate CloudFront

   Prod runs the same steps only when started by hand.
3. Both workflows authenticate through OIDC only. No AWS keys are stored in GitHub.
4. Pull requests to `main` require `ci.yml` to pass.
5. Unit tests need no AWS account and no Docker service: AWS calls are faked with `moto` (DB-T03).
Status: Planned

## Security, observability, and cost

### OPS-T07: IAM roles and the signing-key secret
Depends on: OPS-T03, OPS-I03
Acceptance criteria:
1. A reusable role pattern gives each Lambda its own role, with permissions listed per resource ARN.
2. The CloudFront signing key is stored in Secrets Manager. Only the `api` and `notifier` roles can read it.
3. A check in CI fails if any role uses `"Resource": "*"` for data access.
4. No secret appears in the repository, in CI settings, or in Terraform output.
Status: Planned

### OPS-T08: Logging, metrics, dashboard, and alarms
Depends on: OPS-T03, OPS-I04
Acceptance criteria:
1. A small shared logging helper in `libs/` writes JSON logs with the service, the environment, a request ID, and `sourceId`, `postingKey`, `sub`, or `docId` where relevant. A test confirms it drops tokens, email addresses, and resume text.
2. Metrics are emitted in EMF: postings found, matches, PDFs rendered, emails sent, Bedrock spend per hour, deferred work, and invalid ranking output.
3. A CloudWatch dashboard shows these next to queue depth and Lambda errors.
4. **Alarms**, all going to the team's SNS topic by email:
   - DLQ depth above 0
   - scrape errors
   - API 5xx
   - SES bounces
   - 80% of the hourly and daily Bedrock caps
Status: Planned

### OPS-T09: The monthly budget alert
Depends on: OPS-T03, OPS-009
Acceptance criteria:
1. An AWS Budgets cost budget of `monthly_budget_usd` (default: $50) alerts at the `budget_alert_percents` (default: 50, 80, and 100 percent of actual spend) and at 100 percent of forecast spend.
2. Alerts go to the team's SNS topic.
3. Changing the amount means editing `tfvars` and applying, with no code change.
Status: Planned

### OPS-T10: Document and do the manual steps
Depends on: OPS-T02, OPS-006, OPS-008
Acceptance criteria:
1. A runbook in `infra/` lists every step outside Terraform, in order:
   - run bootstrap
   - enable Bedrock model access in `us-east-1`
   - verify the SES sender address and the team's and the demo's email addresses in the SES sandbox
   - run the catalog seed script
   - before the first deploy: create the CloudFront signing key pair and commit its public half; after the first deploy: store the private half in the secret Terraform created, then delete the local copy (OPS-014, once Accepted)
   - redrive a DLQ
2. A teammate following the runbook from a fresh clone gets a working dev environment.
3. The runbook says how to delete a student's data by hand (WEB-012, SYS-Q05).
Status: Planned

## AWS account (Proposed, 2026-11-23)

These tasks come from brainstorming what the account holder has to do (OPS-Q12: Froop runs in Ryan Fusco's account). They are Proposed until every owner agrees, and the instructor meeting (MQ-2026-11-19-01) may still change which account is used. Choices marked *Rec* are recommendations, not decisions.

### OPS-T11: Set up the AWS account and access
Depends on: OPS-Q12
Acceptance criteria:
1. **Account hygiene:** the root user has MFA and no access keys; alternate contacts are set; billing alerts, Cost Explorer, and cost anomaly detection are on; account-level S3 Block Public Access is on; the default CloudTrail management-events trail is kept. GuardDuty is not turned on (it's paid after a trial).
2. **Credits and free tier:** the account's credits and free-tier terms are checked, including whether they cover Bedrock, and the result is written in the runbook.
3. **Teammate access:** *Rec:* IAM Identity Center (free), with one user each for Ryan Fusco, Graham Cohick, and Luke Campbell, MFA required, and sign-in through `aws sso login`. No long-lived access keys. Two permission sets: Admin (the account holder only) and FroopDeveloper (read everything, CloudWatch logs, Lambda invoke, SQS redrive, DynamoDB reads).
4. **Deploys go through CI only.** Nobody applies Terraform to dev from a laptop, except the one-time bootstrap (OPS-T02).
5. **CI trust:** the GitHub OIDC provider's deploy role trusts only `lukeacampbell/Froop` on `main`, and the read-only plan role trusts only that repository's pull requests. Luke Campbell, as the repository's owner, adds the role ARNs as repository variables and turns on branch protection for `main`.
6. **Deploy role limits:** *Rec:* the deploy role can create IAM roles and policies only with names starting `froop-` and only with the `froop-boundary` permissions boundary, rather than full administrator access.
7. **Quotas checked early:** Lambda concurrent executions (new accounts often start at 10, which breaks reserved concurrency and SQS scaling; request an increase if so) and Bedrock tokens per minute for the configured models.
8. **Manual service setup:** Bedrock model access in `us-east-1` (Anthropic models may need a one-time use-case form) for the parser's small Claude Haiku model (EB-024) and the Matcher's model, with each one's model ID and price per token written in the runbook, and the SES sender and demo recipients verified (OPS-008, OPS-015).
9. **Data responsibility:** the account holder receives data-deletion requests and follows the runbook's manual deletion steps (WEB-012).
Status: Blocked (Proposed; OPS-Q12 teammate access and the instructor meeting)

### OPS-T12: Tear down after the course
Depends on: OPS-T11
Acceptance criteria:
1. A teardown date is set (*Rec:* one week after grades are posted).
2. The runbook lists, in order: `terraform destroy` for dev; delete the ECR images and repositories, the state bucket, the signing-key secret, and the SES identities; remove teammates' Identity Center access and the GitHub OIDC roles.
3. A check one billing cycle later confirms the bill is $0.
Status: Blocked (Proposed)
