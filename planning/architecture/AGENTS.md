# Architecture Planning AGENTS.md

This folder is the workspace for **Phase 1: Architecture Planning**. It holds the predefined architecture (RFC 0001 in `inputs/`), the review of it, the decisions that come out of that review, and the implementation plan for each area.

The rules in `planning/AGENTS.md` apply here, along with the file formats, ID prefixes, and decision format it defines. This file only adds what's specific to this folder. If the two conflict, `planning/AGENTS.md` wins.

## What's in This Folder

```
architecture/
├── AGENTS.md                      # This file
├── inputs/                        # RFC 0001 and its diagram (read-only); see inputs/AGENTS.md
└── <area>/
    ├── AGENTS.md                  # How to work in this area (guide, not content)
    ├── architecture.md            # What the area is, what was given, what we decided, its boundaries
    └── implementation-plan.md     # How we'll build it
```

### The Three Area Files

Every area has the same three files. Read them in this order, because each one builds on the one before it.

**1. `<area>/AGENTS.md`: how to work in this area.**
- **What it is:** a guide and working agreement between the area's owner and the agents working in it. It holds:
  - the owner and contributors
  - a short purpose statement, in product terms
  - the Status (phase, state, last reviewed)
  - where to find things in the area
  - the Working Agreement: what agents may do without asking, what needs the owner, and the area's ground rules
- **Read it first,** before you touch anything in the area.
- **Update it when** the area's state changes, or when the owner changes how agents should work there.
- **What it isn't:** a place for design or implementation content. No services, schemas, interfaces, or open questions. Those go in `architecture.md`.

**2. `<area>/architecture.md`: what the area is, what was given, and what we decided.**
- **Scope:** what the area owns and what it doesn't, naming the owner of each excluded item.
- **Architecture (as given):** the area's part of RFC 0001, transcribed without changes and cited by page. It changes only to fix transcription errors.
- **Architecture Review:** findings about the given design, each written as finding → impact → recommendation.
- **Decisions:** the append-only log of choices, each `Proposed` or `Accepted`. This is the design the area will actually build. Where a decision differs from the as-given text, the decision wins.
- **Interfaces:** what the area provides to other areas and what it consumes from them. Read neighboring areas' Interfaces before you change one.
- **Open Questions:** unresolved items, each with an owner and what it blocks.
- **Update it** while scoping, reviewing, deciding, and agreeing interfaces. Never delete a decision. Supersede it instead.

**3. `<area>/implementation-plan.md`: how we'll build it.**
- **Holds:** ordered tasks (`<PREFIX>-T01`, …), each with its dependencies, acceptance criteria that can be tested, and a status.
- **Built only from Accepted decisions and agreed interfaces.** A task that depends on something still `Proposed` is marked `Blocked`, with that ID as the reason.
- **Becomes the main working file in Phase 2.** Agents update task status as the code is built.

## Area Owners

Each area has one owner. The owner is the human who accepts decisions, approves the area, and answers its open questions.

| Area | Owner | Contributors |
|---|---|---|
| `web-app-and-api` | Graham Cohick | — |
| `job-discovery-and-matching` | Luke Campbell | Ryan Fusco (Matcher and bullet selection) |
| `experience-bank` | Ryan Fusco | — |
| `delivery` | Ryan Fusco | — |
| `build-and-operation` | Ryan Fusco | — |
| `database` | Ryan Fusco (pending SYS-Q02) | Graham Cohick, Luke Campbell (the item types their areas write) |

### Working on Someone's Behalf

At the start of a session, confirm which group member you're working for. Ask if it isn't clear from the request. What you may edit depends on that person's role in each area:

| Their role | `<area>/AGENTS.md` | `<area>/architecture.md` | `<area>/implementation-plan.md` |
|---|---|---|---|
| **Owner** | Update Status. Change the Working Agreement only when the owner asks | Edit any section, following the update rules in `planning/AGENTS.md` | Add, reorder, and update tasks |
| **Contributor** | Read-only | Add review findings, `Proposed` decisions, `Proposed` interfaces, and Open Questions that fall within their contribution | Add `Proposed` tasks that fall within their contribution |
| **Neither** | Read-only | Read-only | Read-only |

In every case:

- **Only the owner accepts.** A contributor's additions stay `Proposed` until the owner accepts them. Don't restructure or rewrite the owner's existing content.
- **Ask other areas through Open Questions.** When you need something from an area the person doesn't own, record an Open Question in one of their own areas, with the other area's owner as its owner. Or propose an interface (see "Coordinating Across Areas").
- **Never mark anything `Accepted` or `Approved`** on behalf of a group member unless that person tells you to in the current session.

## How to Plan an Area

Work through these steps in order. Each step builds on the one before it.

1. **Transcribe the architecture.** Copy the area's part of RFC 0001 into "Architecture (as given)" in `architecture.md`, without changing it. Use "Where Each Area's Material Is" below to find the right lane and pages, and cite each page. Follow the reading rules in `inputs/AGENTS.md`.
2. **Set the scope.** In the Scope section of `architecture.md`, write what the area owns and what it doesn't, naming the owner of each excluded item. Check the scope against the Areas list in `planning/AGENTS.md` and against the scope of every other area. If any of them overlap, settle the overlap before you continue.
3. **Review the architecture.** Record each finding in "Architecture Review" as finding → impact → recommendation. Use the critique checklist from `planning/AGENTS.md`, and also ask of each component:
   - Does it help a demo goal (RFC p.2, p.8) or a course requirement, or is it speculative?
   - What does it cost to run, especially Bedrock, Lambda container images, and anything always on?
   - How is it deployed, configured, and monitored? (Check this with `build-and-operation`.)
   - What user data does it touch, and is that data scoped to the user's Cognito `sub`?
   - What happens when it fails? Consider retries, the DLQ, and whether it can safely run twice.
4. **Record decisions.** Turn each recommendation the owner wants to act on into a decision. Reviewing findings with the owner one at a time works better than proposing everything at once.
5. **Define the interfaces.** List everything the area exposes and consumes, following "Coordinating Across Areas" below.
6. **Write the implementation plan.** Break the accepted design into ordered tasks, with dependencies and acceptance criteria that can be tested. Tasks may depend on tasks in other areas. Cite them by ID, and record an Open Question if that task doesn't exist yet.
7. **Request approval.** Set `Area state: In Review`, then list the root Phase 1 exit criteria with any that aren't met. The owner sets `Approved`.

Steps can be revisited. A review finding can change the scope, and an interface can force a new decision. When that happens, update the earlier step's content rather than working around it.

### Where Each Area's Material Is

The lanes in the RFC 0001 diagram match the area folders. Transcribe each area from its lane and the RFC sections listed below. Shared sections (data model, security, IaC) are split by which area's components they describe. Parts that span areas are copied into every area they affect, citing the same page. The single DynamoDB table sits outside every lane and has its own area, `database`. Other areas refer to its item types rather than transcribing them.

| Area | Diagram lane components | RFC sections (page) |
|---|---|---|
| `web-app-and-api` | Cognito, API Gateway (HTTP API, JWT authorizer), API Lambda (FastAPI + Mangum), CloudFront, S3 static site (React dashboard) | AWS Services rows 2, 3, 6 (p.3–4); API (p.5–6); Security: authentication and authorization (p.7) |
| `experience-bank` | S3 `uploads/` (presigned PUT), Bank parser Lambda (PDF/DOCX/TXT), Bedrock (split text) | Summary, Goal 3 (p.1–2); Data Model: `DOC#` items, `uploads/` prefix (p.5); `POST /uploads`, `DELETE /uploads/{docId}`, `GET /experience` (p.5–6); Demo step 1 (p.8) |
| `job-discovery-and-matching` | EventBridge Scheduler, Catalog dispatcher Lambda, SQS scrape, Scraper Lambda (Playwright image), job sites, SQS match, Matcher Lambda (cost-cap check), Bedrock (pick bullet IDs) | Goals 1–2 (p.2); Data Model: `SOURCE#`, `POSTING#`, `SUB#`, `MATCH#`, `COST#`, postingKey, GSI1, LSI1 (p.5); sources, subscriptions, and matches endpoints (p.6); Demo steps 2–3 (p.8); Alternatives (p.8) |
| `delivery` | SQS render, Renderer Lambda (LaTeX image, one page), S3 `resumes/` (30-day lifecycle), Notifier Lambda (instant, hourly, daily), SES, the student's inbox | Summary, Goal 4 (p.1–2); AWS Services rows 10, 14 (p.4); Data Model: `resume/` prefix, GSI2 (p.5); Demo step 4 (p.8); Risks: how to deliver the resume (p.8) |
| `build-and-operation` | GitHub Actions, ECR, CloudWatch (EMF metrics, alarms, dashboard), SNS | AWS Services rows 1, 4, 5, 8, 11–14 (p.3–4); Infrastructure as Code (p.6–7); CI/CD (p.7); Security: IAM and secrets (p.7); Observability (p.7–8); Demo step 5 (p.8); Alternatives: Fargate (p.8) |
| `database` | DynamoDB (outside the lanes: users, settings, sources, subscriptions, postings, experience bank, matches, hourly cost counters) | AWS Services row 4 (p.4); Data Model: DynamoDB access patterns, table design, postingKey, GSIs (p.4–5) |

## Coordinating Across Areas

Most cross-area work is interfaces. In this design, they're mostly SQS messages, DynamoDB item types, and S3 prefixes rather than API calls. Three areas have different owners and depend on each other, so follow these rules:

- **The area that provides an interface owns its definition.**
  - **Message:** for an SQS message, the producer owns the payload shape.
  - **DynamoDB item:** `database` owns every item type's definition. The area that writes an item type proposes its attributes there as a contributor. It's agreed only when both the `database` owner and the writing area's owner accept it.
  - **S3 prefix:** the area that writes the objects owns the key layout.

  The definition lives in the provider's `architecture.md`. A consumer lists the interface under its own Interfaces, cites the provider's definition, and does not redefine it.
- **A new or changed interface starts as `Proposed`** in the provider's area. When the provider is someone else's area, propose it through an Open Question to that owner.
- **An interface is agreed only when both owners have accepted it.** Record each owner's acceptance next to the interface, for example `Accepted by: Ryan Fusco (2026-11-14), Graham Cohick (2026-11-15)`.
- **`build-and-operation` touches every area.** Each area's review should list what it needs from `build-and-operation`: Lambda functions and images, queues, buckets, IAM permissions, secrets, alarms, and Terraform module placement. `build-and-operation` collects those needs in its Requirements Intake table.

### Dependencies From RFC 0001

These dependencies come from the diagram and data model on RFC pp.3–5. Update this list when a decision changes them.

- `web-app-and-api` owns sign-in (Cognito) and the public API. The API Lambda serves reads from every area's DynamoDB items: experience, sources, matches, and settings. Every other area identifies the user by their Cognito `sub`.
- `experience-bank` is fed by an upload through `web-app-and-api` (presigned PUT to `uploads/`). It provides verbatim bullets, roles, and skills (`DOC#` items) to `job-discovery-and-matching`, which selects bullets, and to `delivery`, which renders them.
- `job-discovery-and-matching` provides matches with selected bullet IDs to `delivery`, through SQS render and `MATCH#` items. It reads delivery settings that `web-app-and-api` writes.
- `delivery` provides resume PDFs (`resume/` prefix) and emails. `web-app-and-api` serves signed links to the PDFs (`GET /matches/{postingKey}`).
- `database` designs the shared DynamoDB table: its access patterns, keys, indexes, and every item type. Every area reads or writes through those definitions.
- `build-and-operation` provisions the table as `database` designs it, plus the S3 buckets (the Terraform `data` module), CI/CD, ECR, alarms, and secrets for every area.

## System Open Questions

Questions that span areas or affect the whole architecture. They use the `SYS-Q` prefix.

- **SYS-Q01: Who owns platform work?** The RFC (p.1) lists Graham Cohick as owning "App & platform". The current owner table gives `build-and-operation` to Ryan Fusco. We need to confirm the split, especially for the Terraform `app` module, the CloudFront signing key, and API Gateway throttling.
  - Owner: group
  - Blocks: OPS scope, `web-app-and-api` scope
- **SYS-Q02: Who owns the `database` area?** It holds the shared table design and every item type, so the whole group depends on it. The proposal is Ryan Fusco, with Graham Cohick and Luke Campbell as contributors for the item types their areas write.
  - Owner: group
  - Blocks: accepting any DB interface
- **SYS-Q03: Which Terraform module holds each area's resources?** The RFC (p.6) lists `data`, `ingestion`, `resumes`, `app`, and `ops`, but not which area owns each one. The proposal: `ingestion` goes to `job-discovery-and-matching`, `resumes` covers `experience-bank` and `delivery`, `app` goes to `web-app-and-api`, and `data` and `ops` go to `build-and-operation`. The Matcher could sit in either `ingestion` or `resumes`.
  - Owner: group
  - Blocks: OPS-I01

## When This Phase Is Done

An area is done when its owner sets it to `Approved` and it meets the Phase 1 exit criteria in `planning/AGENTS.md`. The phase is done when every area is approved, every interface is agreed by both of its owners, every System Open Question is answered, and the group confirms the move to Phase 2.
