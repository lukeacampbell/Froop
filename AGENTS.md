# Froop AGENTS.md

This file tells agents how to work in the Froop repository. Read it before you read or write any code.

Froop is a CS 1660 group project: it finds Greenhouse job postings and builds a tailored one-page LaTeX resume for each match, using only the student's own words. It runs on AWS (Lambda, DynamoDB, S3, SQS, Bedrock, SES, Cognito, API Gateway, CloudFront), deployed with Terraform and GitHub Actions.

## The planning docs are the source of truth

`planning/` says **what** Froop builds, **why**, and **how we know it works**. Code says **how** it currently works. When the two disagree, stop and fix the disagreement. Don't quietly pick one.

Before you write code, read, in order:
1. `planning/AGENTS.md`: the phases, the rules for each phase, and the ID formats.
2. `planning/architecture/AGENTS.md`: owners, shared ownership, and how areas coordinate.
3. The area `AGENTS.md` for the code you're touching (see "Code Map" below). It says what you may do there without asking.
4. That area's `implementation-plan.md`: the task you're working on, its dependencies, and its acceptance criteria.
5. That area's `architecture.md`: **Interfaces** (the contracts you must follow exactly) and **Decisions** (why the design is the way it is).
6. The Interfaces of every area your code calls or is called by.

`planning/architecture/revised-rfc/Froop_revised_architecture.md` is the one-document summary of the whole system as it stands. It includes Accepted decisions only.

Order of authority: Accepted decisions > approved plans > Architecture Review > Initial Architecture. An item marked `Proposed` is not agreed: don't build on it.

## Current phase

The current phase is marked **(current)** in the table in `planning/AGENTS.md`. Only a human changes it.

- **In Phase 1 (Architecture Planning),** no application code, test code, project scaffolding, or dependencies are added. The folders in the Code Map don't exist yet. OPS-T01 creates them in Phase 2.
- **In Phase 2 (Implementation),** code is written only for tasks in an `implementation-plan.md`, and only once the task's status is `Planned` and the group has confirmed the move to Phase 2.

## Code Map

Each folder belongs to exactly one area. The area's owner decides what changes there; the area's `AGENTS.md` says what an agent may do without asking. Folder layout comes from OPS-007, OPS-004, and OPS-012.

| Folder | Area (owner) | What it holds | Tasks | Contracts it provides |
|---|---|---|---|---|
| `services/api/` | web-app-and-api (Graham Cohick) | The API Lambda (FastAPI + Mangum) | WEB-T01, WEB-T03, WEB-T05 to WEB-T10 | WEB-I01, WEB-I02 |
| `web/` | web-app-and-api (Graham Cohick) | The React dashboard | WEB-T11 to WEB-T16 | — |
| `services/parser/` | experience-bank (Ryan Fusco) | The Bank parser Lambda | EB-T03 to EB-T07 | EB-I01 (with the API) |
| `libs/experience_bank/` | experience-bank (Ryan Fusco) | The only code that reads or writes the experience bank | EB-T01, EB-T02, EB-T04 | EB-I03 |
| `services/renderer/` | delivery (Ryan Fusco) | The Renderer Lambda (the only image with TeX) | DEL-T01 to DEL-T04 | DEL-I01 |
| `services/notifier/` | delivery (Ryan Fusco) | The Notifier Lambda | DEL-T05, DEL-T06 | — |
| `services/dispatcher/`, `services/scraper/`, `services/matcher/` | job-discovery-and-matching (Luke Campbell; Ryan Fusco contributes to the Matcher) | Discovery, deduplication, and ranking | JDM-T01 to JDM-T09 | JDM-I01 to JDM-I05 |
| the budget reservation package in `libs/` | job-discovery-and-matching (Luke Campbell) | Bedrock cost reservations for the parser and the Matcher | JDM-T05 | JDM-I04 |
| the key-builder package in `libs/` (for example `libs/froop_keys/`) | database (shared) | Every DynamoDB key and index key, and the item-type registry | DB-T02, DB-T03 | DB-I01 to DB-I05 |
| the logging helper in `libs/` | build-and-operation (shared) | JSON logs and EMF metrics | OPS-T08 | OPS-I04 |
| `infra/bootstrap/`, `infra/modules/data/`, `infra/modules/ops/`, `infra/envs/` | build-and-operation (shared) | Bootstrap, the table and buckets, alarms and budget, the environment | OPS-T02 to OPS-T09 | OPS-I01 to OPS-I05 |
| the table definition in `infra/modules/data/` | database (shared) | The table, its indexes, and TTL | DB-T01 | DB-I01 |
| `infra/modules/app/` | web-app-and-api (Graham Cohick) | Cognito, API Gateway, the API Lambda, CloudFront | WEB-T02 to WEB-T04 | — |
| `infra/modules/resumes/` | experience-bank and delivery (Ryan Fusco) | The parser, the Renderer, the Notifier, and the digest schedules | EB-T07, DEL-T07 | — |
| `infra/modules/ingestion/` | job-discovery-and-matching (Luke Campbell) | The dispatcher, scraper, Matcher, and their queues | JDM-T10 | — |
| `.github/workflows/` | build-and-operation (shared) | `ci.yml` and `deploy.yml` | OPS-T06, WEB-T16 | — |

Shared packages whose folder name isn't fixed above get their name from the task that creates them. Once a folder exists, its own `AGENTS.md` (added by OPS-T01) names its area and tasks.

## Rules for writing code

- **Trace every change to a task.** Use the task ID in the branch name, commit messages, and PR description (for example `EB-T04: parse Jake's Resume entries`).
- **Build only what the task says.** If something is needed and no task covers it, propose a task in the area's `implementation-plan.md` and get it approved. Don't build it first.
- **Follow interfaces exactly.** An interface in an area's `architecture.md` is a contract between owners. If code needs it to change, update the planning docs in both areas first (or in the same PR), and get both owners' acceptance.
- **Stay in your area.** Don't edit another area's folders. If you need something from another area, use its interface, or ask through an Open Question in the planning docs.
- **Use the shared packages, never their internals.** Read and delete the experience bank only through `libs/experience_bank` (EB-010). Build DynamoDB keys only with the key builders (DB-T02). Reserve Bedrock spend only through the reservation package (JDM-I04).
- **Write a unit test for each acceptance criterion.** A task is Done only when they pass in CI (OPS-005).
- **Update the task's status** in `implementation-plan.md` when you start (`In Progress`) and finish (`Done`).
- **When the code teaches you something** (an approach fails, an assumption is wrong), record a `Proposed` decision or an Open Question in the area's `architecture.md`. Don't work around it silently.

## Rules every component follows

- **A student is their Cognito `sub`,** taken from the sign-in token by the API and never from a request (WEB-005, WEB-I02). Every item and object a student owns is stored under that `sub`.
- **A student's words are never changed.** No code rewrites, paraphrases, or generates resume text (RFC non-goal "no AI-written resume text").
- **Uploaded LaTeX is never run.** The parser reads it as text (EB-009). The Renderer compiles only its own template with allowed macros (DEL-002).
- **Never log** tokens, email addresses, resume text, job descriptions, or prompts (OPS-I04, WEB-005).
- **No secrets in the repository** or in CI settings. CI reaches AWS only through OIDC (OPS-005). The only secret is the CloudFront signing key (OPS-I03, OPS-014).
- **Configuration comes from environment variables** that Terraform sets (OPS-I01). Cost caps, models, and limits are never hard-coded (OPS-002, OPS-006, OPS-010).
- **Retries are safe.** Queue handlers and event handlers can run twice; use the conditional writes and idempotency keys the interfaces define.
