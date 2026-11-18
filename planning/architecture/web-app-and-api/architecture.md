# Web App and API Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`). See "Where Each Area's Material Is" in `planning/architecture/AGENTS.md` for this area's lane and pages. Items are `Proposed` until the owner accepts them. Accepted decisions override the Initial Architecture.

## Scope

_None yet._

## Current Design Diagram

![Web App and API: current design](architecture.png)

Source: `architecture.drawio`. The diagram shows the design **as it stands now**:
- **Solid borders:** the initial architecture or Accepted decisions.
- **Dashed borders:** Proposed or pending items.
- **Grey boxes:** parts that belong to other areas.

Keep it in step with this file (see "Keeping Area Diagrams Current" in `planning/architecture/AGENTS.md`).

## Initial Architecture

> **Frozen baseline.** This is RFC 0001 *as originally submitted* (`architecture/inputs/Froop_inital_architecture.pdf`), transcribed without changes. It is never updated when decisions change the design. The current design is in **Decisions** below, and the whole current system is in `architecture/revised-rfc/`.

- **Summary (p.1):** "Students get the PDF by email, instantly or in a digest, and browse all matches in a dashboard"
- **Non-Goals (p.2):** "No mobile app"
- **AWS Services (p.3–4):**
  - Lambda: "Compute". Why: "Tasks come in bursts; the server would be idle most of the time"
  - API Gateway: "Rest API". Why: "Simplicity and development ease"
  - Cognito: "Authz + Authn". Why: "Also easy and integrates w/ ^ We will not need to store passwords ourselves"
  - CloudFront: "CDN for website + resumes". Why: "Time based credentials"
  - Secrets Manager: "Stores the CloudFront signing key". Why: "Keeps the key out of git; only the api and notifier Lambdas can read it"
- **Data Model, access patterns (p.4–5):** "Get or update a user's profile and settings"; "Manage a user's private sources and subscriptions"
- **Data Model, table design (p.5):** "Sort key: PROFILE, SETTINGS, ..."
- **API (p.5–6):**

  | Method | Path | Auth Required? | Description |
  |---|---|---|---|
  | GET | /health | no | Health check |
  | GET | /me | yes | Get profile and settings |
  | PUT | /me | yes | Update settings (delivery mode, filters) |
  | POST | /uploads | yes | Get a presigned URL to upload one file |
  | DELETE | /uploads/{docId} | yes | Delete an upload and its bullets |
  | GET | /experience | yes | List the experience bank |
  | GET | /sources | yes | List catalog sources and the user's private sources |
  | POST | /sources | yes | Add a private page |
  | Delete | /sources/{sourceId} | yes | Delete a private page |
  | POST | /sources/{sourceId}/refresh | yes | Scrape a source now |
  | POST | /subscriptions/{sourceId} | yes | Subscribe to a source |
  | DELETE | /subscriptions/{sourceId} | yes | Unsubscribe from a source |
  | GET | /matches | yes | List matches, newest first |
  | GET | /matches/{postingKey} | yes | Get one match with a signed resume link |

  Errors: "401 - missing or invalid token, rejected by API gateway"; "400 - invalid input, such as a bad URL or file type"; "404 - not found, or belongs to another user"; "409 - sources already added or already subscribed"; "429 - refresh requested too often"
- **Security (p.7):** "Users sign in with Cognito. API Gateway checks each request's token before the API runs." "The API gets the user ID from the token, not from the request. All user data in DynamoDB and S3 is stored under that ID, so users can only see their own data. Upload links only allow one file in the user's own folder, and resume links expire. Each Lambda has its own role with only the access it needs." "The only secret is the CloudFront signing key. It is kept in Secrets Manager, and only the api and notifier can read it."
- **Demo steps 1–2 (p.8):** "Sign up and upload a scanned PDF resume." "Add a private careers page that needs JavaScript, then trigger a refresh. New postings from that page and from the catalog show up in the feed."
- **Diagram, Web App and API lane (p.3):**
  - "Student (browser)" → "sign in" → "Cognito (user pool · hosted UI)"
  - Student → "API Gateway (HTTP API · JWT authorizer)" → "API (Lambda · FastAPI + Mangum)" → "sources, feed, settings" → DynamoDB
  - Student → "CloudFront (dashboard + signed resume links)" → "S3 static site (React dashboard)"; "also serves S3 resumes/ via OAC"
  - API → "presigned upload URL" → "S3 · uploads/"
  - A dashed arrow from "Student (inbox)" back to "Student (browser)" (step 10)

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
| SYS-Q01 | `architecture/AGENTS.md` | Confirm shared ownership of `build-and-operation`, with Ryan Fusco coordinating. Platform choices are made there by all owners. |
| SYS-Q02 | `architecture/AGENTS.md` | Confirm shared ownership of `database`, with Ryan Fusco coordinating. |
| SYS-Q03 | `architecture/AGENTS.md` | Group: confirm which Terraform module holds each area's resources. |
| EB-Q10 | `experience-bank/architecture.md` | Change the API for `.tex`-only uploads (EB-001), and replace `DELETE /uploads/{docId}` with a delete-the-bank endpoint (EB-002). |
| EB-I01 | `experience-bank/architecture.md` | Accept the upload contract as its consumer: the key layout, `.tex` only, and the size limit. |
| EB-I03 | `experience-bank/architecture.md` | Accept the experience-bank package as the API's way to list and delete the bank (EB-010). |
| DEL-I01 | `delivery/architecture.md` | Accept the resume PDF contract as its consumer: the API signs dashboard links, and resumes never expire (DEL-007). |
| DEL-Q12 | `delivery/architecture.md` | Provide the regenerate choice on upload, "mark as applied", and per-match regenerate (DEL-008, DEL-014). |
| DB-Q01 | `database/architecture.md` | Propose the attributes of Profile and Settings items. |
| DB-Q03 | `database/architecture.md` | Decide with Luke Campbell who writes Subscription items and private Source items. |
| DB-Q06 | `database/architecture.md` | Decide where the "applied" flag lives (recommended: `appliedAt` on the match). |
| DB-I01 to DB-I03 | `database/architecture.md` | As a co-owner, accept the table conventions, the experience items, and the match delivery state (the dashboard shows it). |
| OPS-Q07 | `build-and-operation/architecture.md` | Confirm this area's row in the Requirements Intake. |
| OPS-001 to OPS-007, OPS-I01 to OPS-I04 | `build-and-operation/architecture.md` | As a co-owner, accept or challenge Ryan Fusco's answers: the state lock, the $1/user and $5/platform hourly caps, the image split, CI, the Bedrock model and region, and the repo layout. Also accept the four contracts. |
| OPS-008 to OPS-010 | `build-and-operation/architecture.md` | As a co-owner, accept staying in the SES sandbox, the $50 monthly budget alert, and the $1.50 daily Bedrock cap. |
| JDM-Q01, JDM-I01 | `job-discovery-and-matching/architecture.md` | Agree Greenhouse-only source input, API-owned source/subscription writes, settings filters and regeneration producer contract; update API diagram/help text. |
