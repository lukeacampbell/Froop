# Architecture Planning AGENTS.md

This folder is the workspace for **Phase 1: Architecture Planning**. It holds the predefined architecture (RFC 0001 in `inputs/`), the review of it, the decisions that come out of that review, and the implementation plan for each area.

The rules in `planning/AGENTS.md` apply here, along with the file formats, ID prefixes, and decision format it defines. This file only adds what's specific to this folder. If the two conflict, `planning/AGENTS.md` wins.

## What's in This Folder

```
architecture/
├── AGENTS.md                      # This file
├── inputs/                        # RFC 0001 and its diagram (read-only); see inputs/AGENTS.md
├── revised-rfc/                   # The current RFC, with every Accepted decision applied; see revised-rfc/AGENTS.md
└── <area>/
    ├── AGENTS.md                  # How to work in this area (guide, not content)
    ├── architecture.md            # What the area is, what was first proposed, what we decided, its boundaries
    ├── architecture.drawio        # The area's current design diagram (source)
    ├── architecture.png           # The diagram exported from the .drawio file, embedded in architecture.md
    └── implementation-plan.md     # How we'll build it
```

### The Area Files

Every area has the same three documents, plus a diagram (see "Keeping Area Diagrams Current"). Read them in this order, because each one builds on the one before it.

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

**2. `<area>/architecture.md`: what the area is, what was first proposed, and what we decided.**
- **Scope:** what the area owns and what it doesn't, naming the owner of each excluded item.
- **Current Design Diagram:** `architecture.png`, exported from `architecture.drawio`, showing the design as it stands now.
- **Initial Architecture:** the area's part of RFC 0001 *as originally submitted*, transcribed without changes and cited by page. It's a frozen baseline. It is **never** updated when decisions change the design, so reviewers can always see what changed and why. The *current* design is the Decisions in this file, and the whole current system is in `revised-rfc/`. It changes only to fix transcription errors.
- **Architecture Review:** findings about the given design, each written as finding → impact → recommendation.
- **Decisions:** the append-only log of choices, each `Proposed` or `Accepted`. This is the design the area will actually build. Where a decision differs from the Initial Architecture, the decision wins.
- **Interfaces:** what the area provides to other areas and what it consumes from them. Read neighboring areas' Interfaces before you change one.
- **Open Questions:** unresolved items, each with an owner and what it blocks.
- **Update it** while scoping, reviewing, deciding, and agreeing interfaces. Never delete a decision. Supersede it instead.

**3. `<area>/implementation-plan.md`: how we'll build it.**
- **Holds:** ordered tasks (`<PREFIX>-T01`, …), each with its dependencies, acceptance criteria that can be tested, and a status.
- **Built only from Accepted decisions and agreed interfaces.** A task that depends on something still `Proposed` is marked `Blocked`, with that ID as the reason.
- **Becomes the main working file in Phase 2.** Agents update task status as the code is built.

## Area Owners

The owner is the human who accepts decisions, approves the area, and answers its open questions. Most areas have one owner. `build-and-operation` and `database` are owned by the whole group (see "Shared Ownership").

| Area | Owner | Contributors |
|---|---|---|
| `web-app-and-api` | Graham Cohick | — |
| `job-discovery-and-matching` | Luke Campbell | Ryan Fusco (Matcher and bullet selection) |
| `experience-bank` | Ryan Fusco | — |
| `delivery` | Ryan Fusco | — |
| `build-and-operation` | Shared: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell | — |
| `database` | Shared: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell | — |

### Shared Ownership

Every area runs on `build-and-operation` and stores its data through `database`, and every owner will need to make decisions in both. So the group owns them together:
- **Any owner may edit it,** with the same rights as an owner of a single-owner area.
- **The coordinator** (Ryan Fusco, for both areas) keeps the work moving: drafting, collecting requirements (the Requirements Intake, access patterns, and item types), and asking for acceptances. The coordinator has no extra decision power.
- **A decision or interface is Accepted only when every owner has accepted it.** Record each one, for example `Accepted by: Ryan Fusco (2026-11-18), Graham Cohick (2026-11-18), Luke Campbell (2026-11-19)`. Until then it stays `Proposed`.
- **Approval:** the area moves to `In Review` when the coordinator asks for it, and to `Approved` only when every owner agrees.

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
- **Leave a pointer in the other area.** Whatever your role, you may add a row to the "Requests From Other Areas" table in that area's `architecture.md`, linking to the question, interface, or decision that needs its owner. See "Requests From Other Areas" below.
- **Never mark anything `Accepted` or `Approved`** on behalf of a group member unless that person tells you to in the current session.

## How to Plan an Area

Work through these steps in order. Each step builds on the one before it.

1. **Transcribe the architecture.** Copy the area's part of RFC 0001 into "Initial Architecture" in `architecture.md`, without changing it. Use "Where Each Area's Material Is" below to find the right lane and pages, and cite each page. Follow the reading rules in `inputs/AGENTS.md`.
2. **Set the scope.** In the Scope section of `architecture.md`, write what the area owns and what it doesn't, naming the owner of each excluded item. Check the scope against the Areas list in `planning/AGENTS.md` and against the scope of every other area. If any of them overlap, settle the overlap before you continue.
3. **Review the architecture.** Record each finding in "Architecture Review" as finding → impact → recommendation. Use the critique checklist from `planning/AGENTS.md`, and also ask of each component:
   - Does it help a demo goal (RFC p.2, p.8) or a course requirement, or is it speculative?
   - What does it cost to run, especially Bedrock, Lambda container images, and anything always on?
   - How is it deployed, configured, and monitored? (Check this with `build-and-operation`.)
   - What user data does it touch, and is that data scoped to the user's Cognito `sub`?
   - What happens when it fails? Consider retries, the DLQ, and whether it can safely run twice.
4. **Record decisions.** Turn each recommendation the owner wants to act on into a decision. Reviewing findings with the owner one at a time works better than proposing everything at once. When a decision is Accepted, update the revised RFC (see "Keeping the Revised RFC Current").
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
- **Requests from other areas are listed where the owner will see them.** Every area's `architecture.md` ends with a "Requests From Other Areas" table. When you record something another owner has to act on (an Open Question they own, an interface they consume or provide, or a decision they have to accept), add one row to *their* table:
  - the ID
  - the area where it's defined
  - one line on what's needed

  The request is answered at its source, never in the table. Once it's answered, delete the row. These rows are pointers, not content, so any agent may add or delete them, whatever their role in the area.
- **`build-and-operation` touches every area.** Each area's review should list what it needs from `build-and-operation`: Lambda functions and images, queues, buckets, IAM permissions, secrets, alarms, and Terraform module placement. `build-and-operation` collects those needs in its Requirements Intake table.

### Dependencies From RFC 0001

These dependencies come from the diagram and data model on RFC pp.3–5. Update this list when a decision changes them.

- `web-app-and-api` owns sign-in (Cognito) and the public API. The API Lambda serves sources, matches, and settings from the table, and serves the experience bank through the experience-bank package (EB-I03). It also offers the regenerate choice, "mark as applied", and per-match regenerate (DEL-Q12). Every other area identifies the user by their Cognito `sub`.
- `experience-bank` is fed by a single `.tex` upload through `web-app-and-api` (presigned PUT to `uploads/`), and each upload replaces the bank. It provides verbatim roles, bullets, and skills (`source` and `text`), plus the term resolver, through its package (EB-I03). Users are `web-app-and-api` (list and delete), `job-discovery-and-matching` (selects bullets), and `delivery` (renders them). No other area reads its items from the table directly.
- `job-discovery-and-matching` provides matches to `delivery` through `MATCH#` items and the SQS render message: ranked experiences, projects, and skills by category (DEL-010). It marks matches for the next digest (DEL-011), and re-ranks matches when a student regenerates (DEL-014). It reads the bank through the experience-bank package (EB-I03) and settings that `web-app-and-api` writes.
- `delivery` fits and renders one-page resume PDFs (`resume/` prefix, kept for as long as the match exists), and sends instant emails with the PDF attached and digests with signed links. `web-app-and-api` serves signed links to the PDFs (`GET /matches/{postingKey}`).
- `database` designs the shared DynamoDB table: its access patterns, keys, indexes, and every item type. Every area reads or writes through those definitions.
- `build-and-operation` provisions the table as `database` designs it, plus the S3 buckets (the Terraform `data` module), CI/CD, ECR, alarms, and secrets for every area.

## System Open Questions

Questions that span areas or affect the whole architecture. They use the `SYS-Q` prefix.

- **SYS-Q01: Who owns platform work?** The RFC (p.1) lists Graham Cohick as owning "App & platform". **Proposed answer (Ryan Fusco, 2026-11-17):** the whole group owns `build-and-operation`, with Ryan Fusco as coordinator (see "Shared Ownership"). Platform choices such as the Terraform `app` module, the CloudFront signing key, and API Gateway throttling are then made there by all owners. **Graham Cohick confirms (2026-11-18).** Resolved: Luke confirms on 2026-11-18; Ryan’s proposed shared ownership, Graham’s confirmation and Luke’s confirmation now agree.
  - **Luke’s answer (2026-11-18):** Confirm shared build-and-operation ownership, Ryan coordinating. Every area relies on it, so the coordinator should organize rather than decide alone.
  - Owner: group
  - Blocks: OPS scope, `web-app-and-api` scope
- **SYS-Q02: Who owns the `database` area?** It holds the shared table design and every item type, so the whole group depends on it. **Proposed answer (Ryan Fusco, 2026-11-17):** the whole group owns it, with Ryan Fusco as coordinator (see "Shared Ownership"). **Graham Cohick confirms (2026-11-18).** Resolved: Luke confirms on 2026-11-18; Ryan’s proposed shared ownership, Graham’s confirmation and Luke’s confirmation now agree.
  - **Luke’s answer (2026-11-18):** Confirm shared database ownership, Ryan coordinating. Shared keys and indexes require agreement from their writers/readers.
  - Owner: group
  - Blocks: accepting any DB interface
- **SYS-Q03: Which Terraform module holds each area's resources?** The RFC (p.6) lists `data`, `ingestion`, `resumes`, `app`, and `ops`, but not which area owns each one. The proposal: `ingestion` goes to `job-discovery-and-matching`, `resumes` covers `experience-bank` and `delivery`, `app` goes to `web-app-and-api`, and `data` and `ops` go to `build-and-operation`. The Matcher could sit in either `ingestion` or `resumes`. **Proposed answer in OPS-011 (Luke Campbell, 2026-11-18):** `ingestion` holds the dispatcher, scraper, and Matcher. `resumes` holds the parser and delivery (Renderer, Notifier, digest schedules, DEL-019). `data`, `app`, and `ops` stay as above. **Graham Cohick agrees (2026-11-18),** including `app` for `web-app-and-api`. Waiting for: Ryan Fusco.
  - **Luke’s answer (2026-11-18):** Confirm ingestion contains dispatcher, scraper and Matcher; resumes contains parser, Renderer, Notifier and digest schedules. Keep each workflow with its owner. Remaining group acceptance is separate.
  - Owner: group
  - Blocks: OPS-I01

- **SYS-Q04: Confirm the shared goals and demo for Greenhouse-only scope.** Luke explicitly narrowed discovery to Greenhouse on 2026-11-18 (JDM-001). Proposed group wording: Goal 1 demonstrates repeated discovery and catalog/private references to the same Greenhouse board without duplicate postings or matches; Goal 2 demonstrates catalog and user-added Greenhouse boards. Demo step 2 adds a Greenhouse board, step 3 repeats discovery of the same provider ID, and step 5 uses an invalid board token to demonstrate isolated failure. Remove JavaScript-page and cross-provider coverage promises. No rubric is present in planning/inputs; the group should confirm any course-specific implications. JDM scope is accepted; shared Summary/Goals/Demo approval remains pending. **Ryan Fusco agrees (2026-11-18):** Greenhouse is the only job board, and postings come from its Job Board API. **Graham Cohick agrees (2026-11-18),** so Luke Campbell, Ryan Fusco, and Graham Cohick all agree to the proposed wording. Done in the 2026-11-18 restoration: shared Summary, Goals and Demo now use the agreed Greenhouse wording.
  - Owner: group (Ryan, Graham, Luke)
  - Blocks: final shared RFC goal/demo signoff, not the JDM draft.

- **SYS-Q05: Add "no account deletion" to the RFC's Non-Goals?** A student can delete their experience bank, but nothing deletes their account, matches, or resume PDFs. **Proposed answer (Graham Cohick, 2026-11-18, WEB-012):** list account deletion as a non-goal for this version, and delete a student's data by hand on request. Waiting for: Ryan Fusco, Luke Campbell.
  - Owner: group
  - Blocks: WEB-012, and the Non-Goals section of the revised RFC

## Keeping Area Diagrams Current

Each area's `architecture.drawio` and `architecture.png` show that area's design **as it stands now**, in more detail than the system diagram in `revised-rfc/`. They differ from the revised RFC in two ways:
- **They include Proposed items** (drawn with dashed borders). The revised RFC shows Accepted decisions only.
- **They zoom into one area.** Parts owned by other areas appear as grey boxes, so the connections are visible.

Rules:
- **Update the diagram in the same change as the decision.** Whenever a decision is Proposed, Accepted, or superseded, or an interface changes, in a way that adds, removes, renames, or reconnects a component, update `architecture.drawio`. Then re-export `architecture.png`. When a Proposed item is Accepted, change its border from dashed to solid. When an item is dropped, remove it.
- **Edit the `.drawio`, never the PNG.** Use draw.io, or edit the XML directly. Then export (see "Exporting a Diagram" below), and look at the image before you commit it.
- **Label with IDs.** Put the decision, interface, or question ID next to each element it comes from, so readers can trace it to `architecture.md`.
- **Keep the house style:**
  - Solid border: initial architecture or Accepted.
  - Dashed border: Proposed or pending.
  - Grey: another area's part.
  - Keep the legend at the bottom.
- **Other areas' diagrams:** when your decision changes how another area connects to yours, add a row to that area's "Requests From Other Areas" table asking for its diagram to be updated. Don't edit a diagram in an area where the person you're working for has no role. For a shared area, any owner may update it.

An area can't be set to `In Review` while its diagram doesn't match its `architecture.md`.

### Exporting a Diagram

If draw.io is installed, use File → Export as → PNG, or run `draw.io -x -f png -o architecture.png architecture.drawio`.

Without draw.io, use the draw.io viewer in a headless browser:
1. Write a small HTML page that embeds the `.drawio` XML in a `<div class="mxgraph" data-mxgraph="…">` and loads `https://viewer.diagrams.net/js/viewer-static.min.js`.
2. Screenshot it with headless Edge or Chrome: `--headless=new --virtual-time-budget=15000 --window-size=<width>,<height> --screenshot=architecture.png`. Set the size a little larger than the diagram's page size.
3. Look at the image before you commit it.

## Keeping the Revised RFC Current

`inputs/` holds the RFC as it was submitted, and it never changes. `revised-rfc/` holds the RFC as it stands **now**. It must stay current, because teammates, reviewers, and Phase 2 agents read it to understand the system without going through every area's decision log.

- **Update it whenever an open question is resolved by an Accepted decision.** That also covers an Accepted decision that wasn't prompted by an open question, and one that supersedes another. Do it in the same change, or right after.
- **What to update:** every affected section of `revised-rfc/Froop_revised_architecture.md`, plus a Change Log row. If the diagram changes, also update `architecture.drawio` and re-export `architecture.png`.
- **Only Accepted decisions go in.** Text waiting on an open question stays as it is and is marked *(pending XX-Q00)*.
- **Follow `revised-rfc/AGENTS.md`** for the full rules and the steps to export the diagram.

An area can't be set to `In Review` while it has Accepted decisions missing from the revised RFC.

## When This Phase Is Done

An area is done when its owner sets it to `Approved` and it meets the Phase 1 exit criteria in `planning/AGENTS.md`. The phase is done when:
- every area is approved
- every interface is agreed by both of its owners
- every System Open Question is answered
- the revised RFC includes every Accepted decision and has no *(pending)* markers left
- every area's diagram matches its `architecture.md`
- the group confirms the move to Phase 2
