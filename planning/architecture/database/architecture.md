# Database Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`), Data Model, pp.4–5, and the DynamoDB column of the diagram (p.3). Everything below is `Proposed` until every owner accepts it (shared area). For each item type, the owner of the area that writes it should be the one who proposes it.

## Scope

**Owns:**
- The logical design of the single DynamoDB table: the access patterns it serves, the key formats (`PK`, `SK`), GSI1, GSI2, LSI1, and the TTL attribute.
- The registry of item types: which area writes each one and which areas read it.
- The attribute definitions of every item type, proposed by the area that writes it.
- Conditional-write rules that enforce uniqueness, such as one posting per postingKey and one match per user and posting.

**Does not own:**
- Provisioning the table in each environment, and its Terraform: `build-and-operation` (OPS-I02).
- S3 key layouts: the area that writes each prefix (EB-I01, DEL-I01).
- The code that reads and writes items: each area.
- What the data means to the product, such as what makes a bullet valid or when a match is created. Those rules belong to the area that owns the concept.

## Current Design Diagram

![Database: current design](architecture.png)

Source: `architecture.drawio`. The diagram shows the design **as it stands now**:
- **Solid borders:** the initial architecture or Accepted decisions.
- **Dashed borders:** Proposed or pending items.
- **Grey boxes:** parts that belong to other areas.

Keep it in step with this file (see "Keeping Area Diagrams Current" in `planning/architecture/AGENTS.md`).

## Initial Architecture

> **Frozen baseline.** This is RFC 0001 *as originally submitted* (`architecture/inputs/Froop_inital_architecture.pdf`), transcribed without changes. It is never updated when decisions change the design. The current design is in **Decisions** below, and the whole current system is in `architecture/revised-rfc/`.

- **AWS Services, DynamoDB (p.4):** "Database to store our info". Why: "Access patterns are well defined and key based; serverless; and works with un/semistructured data"
- **Data Model (p.4):** "All app data, from user profiles to job matches, lives in DynamoDB. S3 holds uploaded files and generated resume PDFs."
- **Access patterns (p.4–5):**
  - "Get or update a user's profile and settings"
  - "Save, list, or delete a user's experience bank"
  - "Add a posting only if it's new"
  - "Find users subscribed to a source, then filter by their settings"
  - "Create a match once per user and posting; list a user's matches, newest first"
  - "Manage a user's private sources and subscriptions"
  - "List catalog sources; find matches waiting for a digest; cap Bedrock spend per hour"
- **Table design, single table (p.5):**
  - "Partition key: USER#{cognito sub}, SOURCE#{id}, POSTING#{postingKey}, or COST#{hour}"
  - "Sort key: PROFILE, SETTINGS, DOC#... (uploads, roles, bullets, skills), SUB#{sourceId}, MATCH#{postingKey}, or META"
  - "postingKey: a hash of normalized company, title, and location, so the same job from two sources gets one key. Conditional puts block duplicate postings and matches."
  - "Postings store title, company, location, full description, apply link, and the sources that listed them. Postings expire 30 days after last seen."
  - GSIs: "GSI1: a source's subscribers, and all catalog sources"; "GSI2: matches waiting for a digest"; "LSI1: matches by created date"
- **Security (p.7):** "All user data in DynamoDB and S3 is stored under that ID, so users can only see their own data."
- **Diagram, DynamoDB column (p.3):** "DynamoDB (single table): users · settings, sources · subscriptions, postings (TTL 30 d), experience bank, matches, hourly cost counters"

## Architecture Review

_None yet._

## Decisions

_None yet._

## Access Patterns

Every way the data is read or written. They come from the RFC (pp.4–5) and the decisions since. Every row is `Proposed`. Nothing goes into the table design unless a row here needs it.

| ID | Access pattern | Needed by | Served by | Source |
|---|---|---|---|---|
| DB-AP01 | Get or update a user's profile and settings | web-app-and-api | `USER#{sub}` / `PROFILE`, `SETTINGS` | RFC p.4 |
| DB-AP02 | Get a user's current bank: resolve the current-upload pointer, then read that upload's items | experience-bank (through its package, EB-010) | `USER#{sub}` / `DOC#…` | RFC p.5, EB-008 |
| DB-AP03 | Replace a user's bank: write the new upload's items, switch the pointer, delete the old items | experience-bank | `USER#{sub}` / `DOC#…` | EB-002, EB-008 |
| DB-AP04 | Add a posting only if it's new | job-discovery-and-matching | `POSTING#{postingKey}` / `META`, conditional put | RFC p.5 |
| DB-AP05 | Find the users subscribed to a source, then filter by their settings | job-discovery-and-matching | GSI1 | RFC p.5 |
| DB-AP06 | List catalog sources | job-discovery-and-matching | GSI1 | RFC p.5 |
| DB-AP07 | Create a match once per user and posting, with a copy of the posting details | job-discovery-and-matching | `USER#{sub}` / `MATCH#{postingKey}`, conditional put | RFC p.5, DB-Q05 |
| DB-AP08 | List a user's matches, newest first | web-app-and-api | LSI1 (DB-Q02) | RFC p.5 |
| DB-AP09 | Manage a user's private sources and subscriptions | web-app-and-api, job-discovery-and-matching (DB-Q03) | `SOURCE#{id}` / `META`; `USER#{sub}` / `SUB#{sourceId}` | RFC p.5 |
| DB-AP10 | Find matches waiting for a digest | delivery (the Notifier) | GSI2. The Matcher sets the mark (DEL-011). | RFC p.5, DEL-011 |
| DB-AP11 | Update a match's delivery state, conditionally | delivery | `USER#{sub}` / `MATCH#{postingKey}` | DB-I03 |
| DB-AP12 | Check, and add to, the cost caps: per-user hourly, platform hourly, and platform daily | experience-bank, job-discovery-and-matching | `COST#{hour}`, `COST#{day}`, `USER#{sub}` / `COST#{hour}` (DB-Q07) | RFC p.5, OPS-002, OPS-010 |
| DB-AP13 | Find a user's matches in the regeneration window: posting still active, not marked applied | job-discovery-and-matching | `USER#{sub}` / `MATCH#…`, filtered on `appliedAt`, then a check that the posting exists | DEL-014 |
| DB-AP14 | Mark a match as applied, or unmark it | web-app-and-api | `USER#{sub}` / `MATCH#{postingKey}` (DB-Q06) | DEL-014 |

## Item Types

The item types listed in RFC p.5. Every row is `Proposed`.

| Item type | `PK` | `SK` | Written by (area) | Read by | Definition |
|---|---|---|---|---|---|
| Profile | `USER#{sub}` | `PROFILE` | web-app-and-api | delivery (email address) | DB-Q01 |
| Settings | `USER#{sub}` | `SETTINGS` | web-app-and-api | job-discovery-and-matching, delivery | DB-Q01 |
| Experience (uploads, roles, bullets, skills) | `USER#{sub}` | `DOC#…` | experience-bank | experience-bank only. Other areas go through its package, EB-I03 (EB-010). | DB-I02 |
| Subscription | `USER#{sub}` | `SUB#{sourceId}` | DB-Q03 | job-discovery-and-matching, web-app-and-api | DB-Q01 |
| Match | `USER#{sub}` | `MATCH#{postingKey}` | job-discovery-and-matching (including the digest mark, DEL-011), plus delivery state from delivery and the applied flag from web-app-and-api (DB-Q06) | web-app-and-api, delivery | DB-Q01, DB-I03 |
| Source | `SOURCE#{id}` | `META` | DB-Q03 | job-discovery-and-matching, web-app-and-api | DB-Q01 |
| Posting | `POSTING#{postingKey}` | `META` | job-discovery-and-matching | delivery, web-app-and-api | DB-Q01 |
| Cost counter (hourly platform total; daily and per-user counters pending DB-Q07) | `COST#{hour}` | `META` | job-discovery-and-matching, experience-bank (EB-004) | job-discovery-and-matching, experience-bank | DB-Q01 |

## Interfaces

### Provides

#### DB-I01: Table conventions
Status: Proposed
Consumers: all areas, build-and-operation (provisions it)
Contract:
- **Table:** a single table with generic key attributes `PK` and `SK`, using the formats in Item Types.
- **GSI1:** a source's subscribers, and all catalog sources.
- **GSI2:** matches waiting for a digest.
- **LSI1:** a user's matches by creation date.
- **TTL:** postings expire 30 days after they were last seen.

Every item carries an item-type attribute. A new item type needs a new row in Item Types before anything writes it.
Accepted by: —

#### DB-I02: Experience items
Status: Proposed
Written by: experience-bank
Consumers: experience-bank only. The Matcher, the Renderer, and the API read and delete through the experience bank package (EB-I03, EB-010).
Contract: Item types under `PK=USER#{sub}`, `SK=DOC#…`:
- **Upload record:** `docId`, file name, status (pending, parsed, parsed with unclassified sections, failed), error reason, and upload time. Only `.tex` files are accepted (EB-001).
- **Section:** its heading (`source` and `text`), its category (`education`, `experience`, `projects`, `technical_skills`, `awards`, `certifications`, `publications`, `leadership`, or `other`), and its position (EB-016, Proposed).
- **Entry (role, education entry, or project):** a stable ID, the entry type, its section, its position, and its heading fields (for example title, organization, and dates), each with `source` and `text` (EB-012).
- **Bullet:** a stable `bulletId`, `source`, `text`, the entry it belongs to, and its position.
- **Skill category:** its name (for example `Languages`) with `source` and `text`, and its position (EB-016, Proposed).
- **Skill:** `source`, `text`, its skill category, its position, and a derived `canonical` term with the dictionary version that produced it (EB-007, Proposed).
- **Unclassified section:** `source` and `text` for a leftover section that hasn't been classified yet, because the cost cap was reached (EB-013).
- **Header block:** `source` and `text` (EB-014).
- **Current-upload pointer:** a per-user item naming the current `docId` (EB-008, Proposed).

The exact sort-key formats follow DB-I01. `source` and `text` are never modified after they're written. A new upload replaces the whole set (EB-002).
Accepted by: —

#### DB-I03: Match delivery state
Status: Proposed
Written by: delivery (on Match items, which job-discovery-and-matching writes). The digest mark is the exception: the Matcher sets it (DEL-011).
Consumers: web-app-and-api (shows the state), job-discovery-and-matching
Contract: Attributes on `USER#{sub}` / `MATCH#{postingKey}`:
- `renderStatus` (pending, rendered, skipped, failed), `renderReason` (for example `bank_replaced`, DEL-003), `renderedAt`
- `pdfKey`: the current resume, `resume/{sub}/{postingKey}/{docId}.pdf` (DEL-009, Proposed), and `pdfHistory`, the earlier resume keys, which are never removed (DEL-007)
- `notifyStatus` (pending, sent, failed), `sentAt`
- the GSI2 keys that mark a match as waiting for a digest. The Matcher writes them when it creates the match (for hourly and daily students). The Notifier clears them after sending, or when the match was skipped (DEL-011).

Writes are conditional updates, so a duplicate SQS message changes nothing.
Accepted by: —

### Consumes
- **Provisioning of the table in each environment:** OPS-I02 from `build-and-operation`.

## Open Questions

- **DB-Q01: What attributes do the remaining item types have?** That's Profile, Settings, Subscription, Match (apart from the delivery state), Source, Posting, and Cost counter. The owner of the area that writes each type needs to propose them.
  - Owner: Graham Cohick (Profile, Settings), Luke Campbell (Subscription, Match, Source, Posting, Cost counter)
  - Blocks: DB-I01 acceptance, delivery's use of postings and settings
- **DB-Q02: LSI1 has to be decided before the first deploy.** A local secondary index can only be created along with the table. It also limits each user's partition to 10 GB, and every user's data lives in one partition. The alternatives are a GSI, or sorting matches by an `SK` that includes the date.
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: DB-I01, the first deploy of the `data` module
- **DB-Q03: Who writes Subscription items, and Source items for private pages?** They're created through the API (`POST /subscriptions`, `POST /sources`), but `job-discovery-and-matching` owns the concept.
  - Owner: Graham Cohick, Luke Campbell
  - Blocks: Item Types
- **DB-Q04: Which capacity mode should the table use?** Recommendation: on-demand. Traffic comes in bursts and is idle most of the day (RFC p.3), and on-demand needs no capacity planning before Phase 5.
  - Owner: Ryan Fusco
  - Blocks: DB-I01
- **DB-Q05: Matches need their own copy of the posting details.** Postings expire 30 days after last seen, but matches and resumes never expire (DEL-007). A match has to stay viewable, and able to be re-ranked (DEL-014), after its posting is gone. Recommendation: when the Matcher creates a match, it copies the posting's title, company, location, apply link, and description onto the match item. That copy is never updated.
  - Owner: Luke Campbell (writes Match items), Ryan Fusco
  - Blocks: DEL-004 emails, the dashboard's match view, DEL-008
- **DB-Q06: Where does the "applied" flag on a match live?** Regeneration skips matches the student has applied to (DEL-014). Recommendation: an `appliedAt` attribute on the match, set and cleared by `web-app-and-api` from the dashboard.
  - Owner: Graham Cohick, Ryan Fusco
  - Blocks: DEL-014

- **DB-Q07: Per-user and daily cost counters.** OPS-002 and OPS-010 cap Bedrock spend at $1 per hour per user, $5 per hour for the platform, and $1.50 per day for the platform. The RFC only has one `COST#{hour}` counter. Recommendation:
  - Keep `PK=COST#{hour}, SK=META` as the hourly platform total.
  - Add `PK=COST#{day}, SK=META` as the daily platform total.
  - Add `PK=USER#{sub}, SK=COST#{hour}` for each user.
  - Give each a TTL a few days out.
  - Callers update all three in one transaction, conditioned on none exceeding its cap.
  - Owner: Luke Campbell (writes the cost counters), Ryan Fusco
  - Blocks: OPS-002, OPS-010, EB-004

## Requests From Other Areas

Pointers to items defined in other areas that this area's owner has to act on. Answer each one where it's defined, then delete its row (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).

| ID | Defined in | What's needed |
|---|---|---|
| _None yet._ | | |
