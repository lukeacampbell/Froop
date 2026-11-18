# Job Discovery and Matching Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`). See "Where Each Area's Material Is" in `planning/architecture/AGENTS.md` for this area's lane and pages. Items are `Proposed` until the owner accepts them. Accepted decisions override the Initial Architecture.

## Scope

_None yet._

## Current Design Diagram

![Job Discovery and Matching: current design](architecture.png)

Source: `architecture.drawio`. The diagram shows the design **as it stands now**:
- **Solid borders:** the initial architecture or Accepted decisions.
- **Dashed borders:** Proposed or pending items.
- **Grey boxes:** parts that belong to other areas.

Keep it in step with this file (see "Keeping Area Diagrams Current" in `planning/architecture/AGENTS.md`).

## Initial Architecture

> **Frozen baseline.** This is RFC 0001 *as originally submitted* (`architecture/inputs/Froop_inital_architecture.pdf`), transcribed without changes. It is never updated when decisions change the design. The current design is in **Decisions** below, and the whole current system is in `architecture/revised-rfc/`.

- **Summary (p.1):** "Froop finds new internship/job postings and builds a tailored one-page resume for each match from the student's own words." "On a schedule, it checks a built-in catalog of GitHub internship lists and Greenhouse, Lever, and Ashby boards, plus any private pages a student adds. For each match, an AI model selects and orders the student's bullets and skills without editing a word"
- **Motivation (p.2):** "Internship postings are spread across hundreds of job boards" "It finds postings automatically and builds each resume only from text the student wrote."
- **Goal 1 (p.2):** "Be able to run the same sources twice and get no duplicate postings, even when two sources list the same job."
- **Goal 2 (p.2):** "See new postings from the catalog and from a private page we add, including one that needs JavaScript."
- **Non-Goals (p.2):** "No AI-written resume text, No sharing user-added sources between users"
- **AWS Services (p.4):**
  - EventBridge Scheduler: "Starts scrapes and digest emails on a schedule". Why: "No server running all day just to wake up a few times"
  - SQS: "Queues work between the scraper, matcher, and renderer". Why: "Handles bursts and retries failures without losing work"
  - Bedrock: "All LLM tasks (requirement extraction, advanced parsing, resume item selection)". Why: "Managed LLM with IAM; don't need to worry about GPU/server and can call whenever work is done"
  - ECR: "Chromium and LaTeX are too large for Lambda's zip limit"
- **Data Model, access patterns (p.5):** "Add a posting only if it's new"; "Find users subscribed to a source, then filter by their settings"; "Create a match once per user and posting; list a user's matches, newest first"; "Manage a user's private sources and subscriptions"; "List catalog sources; find matches waiting for a digest; cap Bedrock spend per hour"
- **Data Model, table design (p.5):** "postingKey: a hash of normalized company, title, and location, so the same job from two sources gets one key. Conditional puts block duplicate postings and matches." "Postings store title, company, location, full description, apply link, and the sources that listed them. Postings expire 30 days after last seen." "GSI1: a source's subscribers, and all catalog sources"; "LSI1: matches by created date"
- **API (p.6):** the sources, subscriptions, and matches endpoints (the full table is in `web-app-and-api/architecture.md`), and "409 - sources already added or already subscribed"; "429 - refresh requested too often"
- **Infrastructure as Code (p.7):** "The job catalog is data, so a seed script loads it from a JSON file in the repo."
- **Security (p.7):** "Job boards need no keys"
- **Observability (p.7):** "Every Lambda logs JSON to CloudWatch with the source and posting IDs, so we can trace one posting from scrape to email."
- **Demo steps 2, 3, 5 (p.8):** "Add a private careers page that needs JavaScript, then trigger a refresh. New postings from that page and from the catalog show up in the feed." "Trigger the refresh again. No new postings or emails appear, and a job listed on both a Greenhouse board and the GitHub list shows up once. (Goal 1)" "Point a source at a broken URL and show the alarm email while the other sources keep updating."
- **Risks (p.8):** "Using chromium in lambda, which Claude has said requires a large image and a lot of memory. None of us have used Bedrock yet."
- **Alternatives Considered (p.8):** "We considered Fargate but chose Lambda, because the work is idle most of the day, and container-image Lambdas can run Chromium" "We considered an LLM scraper for every source, but Greenhouse, Lever, and Ashby have JSON APIs that are cheaper and more reliable. We only use LLM extraction for pages users add."
- **Diagram, Job Discovery and Matching lane (p.3):**
  - "EventBridge Scheduler (per-source · catalog · digest)" → (1) → "SQS scrape (queue + DLQ)" → (2) → "Scraper (Lambda · Playwright image)"
  - EventBridge Scheduler → "Catalog dispatcher (Lambda · fan-out)" → SQS scrape
  - Scraper → "fetch" (3) → "Job sites (Greenhouse · Lever · Ashby · GitHub · JS)"
  - Scraper → (4) "new postings only (conditional put) · matches" → DynamoDB
  - Scraper → (5) → "SQS match (queue + DLQ)" → "Matcher (Lambda · cost cap check)"
  - Matcher → (6) → "Bedrock (split text · pick bullet IDs)"
  - Matcher → (7) → "SQS render (queue + DLQ)"
  - EventBridge Scheduler → "digest schedule" → Notifier

## Architecture Review

_None yet._

## Decisions

_None yet._

## Interfaces

### Provides

_None yet._

### Consumes

_None yet._

## Open Questions

_None yet._

## Requests From Other Areas

Pointers to items defined in other areas that this area's owner has to act on. Answer each one where it's defined, then delete its row (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).

| ID | Defined in | What's needed |
|---|---|---|
| SYS-Q01 | `architecture/AGENTS.md` | Confirm shared ownership of `build-and-operation`, with Ryan Fusco coordinating. |
| SYS-Q02 | `architecture/AGENTS.md` | Confirm shared ownership of `database`, with Ryan Fusco coordinating. |
| SYS-Q03 | `architecture/AGENTS.md` | Group: confirm which Terraform module holds each area's resources, including whether the Matcher sits in `ingestion` or `resumes`. |
| DEL-Q13 | `delivery/architecture.md` | Accept the Matcher's side of three decisions: DEL-010 (the render message, with ranked experiences, projects, and skills by category; scores comparable across entries), DEL-011 (marking matches for the next digest), and DEL-014 (re-ranking matches when a student regenerates). |
| EB-004 | `experience-bank/architecture.md` | Agree that the Bank parser also reads and writes the `COST#` counter this area owns. |
| EB-I02 | `experience-bank/architecture.md` | Accept the term resolver, and use it on terms from postings (EB-007). |
| EB-I03 | `experience-bank/architecture.md` | Accept the experience-bank package as the Matcher's way to read the bank. Unclassified items are excluded (EB-013). |
| DB-Q01 | `database/architecture.md` | Propose the attributes of Subscription, Match, Source, Posting, and Cost counter items. |
| DB-Q02 | `database/architecture.md` | Decide with Ryan Fusco whether to keep LSI1, before the first deploy. |
| DB-Q03 | `database/architecture.md` | Decide with Graham Cohick who writes Subscription items and private Source items. |
| DB-Q05 | `database/architecture.md` | Copy the posting details onto each match when creating it. Matches never expire, but postings do (DEL-007). |
| DB-I01 to DB-I03 | `database/architecture.md` | As a co-owner, accept the table conventions, the experience items, and the match delivery state. The Matcher writes the digest mark. |
| DB-Q07 | `database/architecture.md` | Add per-user hourly and daily platform cost counters next to the hourly platform counter (OPS-002, OPS-010). |
| OPS-002, OPS-010 | `build-and-operation/architecture.md` | Decide what the Matcher does when a cost cap is reached (hourly per-user, hourly platform, or daily platform). For example, defer the ranking until the cap resets. |
| OPS-Q07 | `build-and-operation/architecture.md` | Confirm this area's row in the Requirements Intake. |
| OPS-001 to OPS-007, OPS-I01 to OPS-I04 | `build-and-operation/architecture.md` | As a co-owner, accept or challenge Ryan Fusco's answers: the state lock, the hourly caps, the image split, CI, the Bedrock model and region (OPS-006 sets a separate model for each Matcher task), and the repo layout. Also accept the four contracts. |
| OPS-008 to OPS-010 | `build-and-operation/architecture.md` | As a co-owner, accept staying in the SES sandbox, the $50 monthly budget alert, and the $1.50 daily Bedrock cap. |
