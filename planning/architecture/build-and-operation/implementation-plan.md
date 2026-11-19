# Build and Operation Implementation Plan

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This is a shared area, so every owner must agree to changes to this plan.
Date: 2026-11-19. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: decisions OPS-001 to OPS-012 and interfaces OPS-I01 to OPS-I05. OPS-011 is Accepted. The rest have Ryan Fusco's and Graham Cohick's acceptance and wait on Luke Campbell's, except OPS-I05, which waits on Graham Cohick's. No infrastructure exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision or interface still needs agreement before the task can be built. It doesn't mean the task was tried and failed.

**Order.**
1. The repository (T01) and bootstrap (T02) come first, because every area's deploys depend on them.
2. Then modules and environments (T03), data resources (T04), images (T05), and CI (T06).
3. Then IAM and secrets (T07), observability (T08), and the budget (T09).
4. The manual steps (T10) are written down alongside T02 and done before the first demo.

**Tests.** Terraform changes are checked by `terraform validate` and `plan` in CI (T06). Each task's acceptance criteria are checked against a deployed dev environment in Phase 2. Cross-area flows belong to the Phase 3 test plans.

| Waiting on | Tasks it blocks |
|---|---|
| Luke Campbell accepting OPS-001 to OPS-010 and OPS-012 | T01 to T10 |
| Graham Cohick accepting OPS-I05 | T03, T05 (the JDM functions) |
| Graham Cohick and Luke Campbell accepting DB-I01 and DB-001 (shared `database`; Ryan Fusco has) | T04 (the table) |

## Foundations

### OPS-T01: Set up the repository
Depends on: OPS-007, OPS-012
Acceptance criteria:
1. Top-level folders: `services/<name>/` (one per Lambda image), `libs/<name>/` (shared Python packages), `infra/` (Terraform), and `web/` (the dashboard).
2. Terraform's version is pinned to 1.10 or later in `infra/`, and Python's version is pinned for every service.
3. A README explains how to run the unit tests for one service, for one library, and for all of them.
Status: Blocked (OPS-007 and OPS-012 need Luke Campbell's acceptance)

### OPS-T02: Bootstrap each AWS account
Depends on: OPS-T01, OPS-001, OPS-004, OPS-005, OPS-006
Acceptance criteria:
1. A bootstrap configuration, run once by hand, creates the Terraform state bucket (versioned and encrypted, with native lockfile locking), the GitHub OIDC provider, a deploy role, a read-only plan role, and seven ECR repositories (`api`, `scraper`, `renderer`, `parser`, `matcher`, `notifier`, `dispatcher`).
2. Each ECR repository has a lifecycle policy that keeps the last 10 images.
3. Everything is in `us-east-1`.
4. Running bootstrap a second time changes nothing.
5. Two people applying at the same time can't corrupt the state. A test confirms the second apply waits or fails on the lock.
Status: Blocked (OPS-001, OPS-004, OPS-005, and OPS-006 need Luke Campbell's acceptance)

### OPS-T03: Terraform modules and environments
Depends on: OPS-T02, SYS-Q03, OPS-011, OPS-002, OPS-006, OPS-009, OPS-010
Acceptance criteria:
1. **Five modules:**
   - `data`: the table and buckets
   - `ingestion`: dispatcher, scraper, Matcher, and their queues
   - `resumes`: parser, Renderer, Notifier, and the digest schedules
   - `app`: Cognito, API Gateway, the API, and CloudFront
   - `ops`: the dashboard, alarms, SNS, and the budget
2. `envs/dev` and `envs/prod` use the same modules with different `tfvars`.
3. Every setting is a variable in the environment's `tfvars`, with no code change needed:
   - the cost caps (`cost_cap_user_per_hour`, `cost_cap_platform_per_hour`, `cost_cap_platform_per_day`)
   - the budget (`monthly_budget_usd`, `budget_alert_percents`)
   - the Bedrock model for each task
   - the sign-up switch and CORS origins (from `web-app-and-api`)
4. Each module exposes the names its services need (the table, buckets, queues, and secret ARN) as outputs, passed to Lambdas as environment variables (OPS-I01).
Status: Blocked (OPS-002, OPS-006, OPS-009, and OPS-010 need Luke Campbell's acceptance; OPS-I05 needs Graham Cohick's)

### OPS-T04: Data resources
Depends on: OPS-T03, DB-T01, OPS-I02, EB-018, DEL-007
Acceptance criteria:
1. The DynamoDB table is created exactly as DB-T01 defines it, including GSI1 to GSI3, TTL, and on-demand capacity.
2. **Uploads bucket:** private, accepts presigned POST uploads from the dashboard's origin, and sends object-created events for `uploads/` to the parser.
3. **Resumes bucket:** private, with no expiry rule. An optional move to Standard-IA after 30 days is allowed. Served only through CloudFront OAC.
4. **Static-site bucket:** private, served only through CloudFront.
5. Public access is blocked on all three buckets.
Status: Blocked (DB-T01; OPS-I02 needs Luke Campbell's acceptance)

### OPS-T05: Images and the deployment contract
Depends on: OPS-T01, OPS-T02, OPS-003, OPS-004, OPS-011, OPS-I01
Acceptance criteria:
1. A shared base image (Python plus `libs/`) is built first. Each function's image adds only its own code and dependencies.
2. The `scraper` image has no browser. The `renderer` image is the only one with TeX.
3. Each image declares its handler entry point and reads all configuration from environment variables. A missing variable makes the function fail at start-up with a clear message.
4. Images are tagged with the commit SHA.
Status: Blocked (OPS-003, OPS-004, and OPS-I01 need Luke Campbell's acceptance)

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
Status: Blocked (OPS-005 and OPS-012 need Luke Campbell's acceptance)

## Security, observability, and cost

### OPS-T07: IAM roles and the signing-key secret
Depends on: OPS-T03, OPS-I03
Acceptance criteria:
1. A reusable role pattern gives each Lambda its own role, with permissions listed per resource ARN.
2. The CloudFront signing key is stored in Secrets Manager. Only the `api` and `notifier` roles can read it.
3. A check in CI fails if any role uses `"Resource": "*"` for data access.
4. No secret appears in the repository, in CI settings, or in Terraform output.
Status: Blocked (OPS-I03 needs Luke Campbell's acceptance)

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
Status: Blocked (OPS-I04 needs Luke Campbell's acceptance)

### OPS-T09: The monthly budget alert
Depends on: OPS-T03, OPS-009
Acceptance criteria:
1. An AWS Budgets cost budget of `monthly_budget_usd` (default: $50) alerts at the `budget_alert_percents` (default: 50, 80, and 100 percent of actual spend) and at 100 percent of forecast spend.
2. Alerts go to the team's SNS topic.
3. Changing the amount means editing `tfvars` and applying, with no code change.
Status: Blocked (OPS-009 needs Luke Campbell's acceptance)

### OPS-T10: Document and do the manual steps
Depends on: OPS-T02, OPS-006, OPS-008
Acceptance criteria:
1. A runbook in `infra/` lists every step outside Terraform, in order:
   - run bootstrap
   - enable Bedrock model access in `us-east-1`
   - verify the team's and the demo's email addresses in the SES sandbox
   - run the catalog seed script
   - redrive a DLQ
2. A teammate following the runbook from a fresh clone gets a working dev environment.
3. The runbook says how to delete a student's data by hand (WEB-012), pending SYS-Q05.
Status: Blocked (OPS-006 and OPS-008 need Luke Campbell's acceptance)
