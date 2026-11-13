# Database Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`), Data Model, pp.4–5, and the DynamoDB column of the diagram (p.3). Everything below is `Proposed` until the owner, and for each item type the owner of the area that writes it, accept it.

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

## Architecture (as given)

_None yet._

## Architecture Review

_None yet._

## Decisions

_None yet._

## Access Patterns

_None yet._ Transcribe these from RFC pp.4–5. Then give each one an ID (`DB-AP01`, …), the area that needs it, and the key or index that serves it.

## Item Types

The item types listed in RFC p.5. Every row is `Proposed`.

| Item type | `PK` | `SK` | Written by (area) | Read by | Definition |
|---|---|---|---|---|---|
| Profile | `USER#{sub}` | `PROFILE` | web-app-and-api | delivery (DEL-Q03) | DB-Q01 |
| Settings | `USER#{sub}` | `SETTINGS` | web-app-and-api | job-discovery-and-matching, delivery | DB-Q01 |
| Experience (uploads, roles, bullets, skills) | `USER#{sub}` | `DOC#…` | experience-bank | job-discovery-and-matching, delivery, web-app-and-api | DB-I02 |
| Subscription | `USER#{sub}` | `SUB#{sourceId}` | DB-Q03 | job-discovery-and-matching, web-app-and-api | DB-Q01 |
| Match | `USER#{sub}` | `MATCH#{postingKey}` | job-discovery-and-matching, plus delivery state from delivery | web-app-and-api, delivery | DB-Q01, DB-I03 |
| Source | `SOURCE#{id}` | `META` | DB-Q03 | job-discovery-and-matching, web-app-and-api | DB-Q01 |
| Posting | `POSTING#{postingKey}` | `META` | job-discovery-and-matching | delivery, web-app-and-api | DB-Q01 |
| Cost counter | `COST#{hour}` | `META` | job-discovery-and-matching (and experience-bank, EB-Q03) | job-discovery-and-matching | DB-Q01 |

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
Consumers: job-discovery-and-matching (the Matcher reads bullets and skills), delivery (the Renderer reads bullet text by ID), web-app-and-api (`GET /experience`, `DELETE /uploads/{docId}`)
Contract: Item types under `PK=USER#{sub}`, `SK=DOC#…`:
- **Upload record:** `docId`, file name, type, status (pending, parsed, failed), error reason, and upload time.
- **Role:** title, organization, and dates, exactly as written.
- **Bullet:** a stable `bulletId`, the verbatim text, the `docId` it came from, the role it belongs to, and its position.
- **Skill:** the skill text, and the `docId` it came from.

The exact sort-key formats follow DB-I01. The bullet `text` is never modified after it's written.
Accepted by: —

#### DB-I03: Match delivery state
Status: Proposed
Written by: delivery (on Match items, which job-discovery-and-matching writes)
Consumers: web-app-and-api (shows the state), job-discovery-and-matching
Contract: Attributes on `USER#{sub}` / `MATCH#{postingKey}`:
- `renderStatus` (pending, rendered, failed), `pdfKey`, `renderedAt`
- `notifyStatus` (pending, sent, failed), `sentAt`
- the GSI2 keys that mark a match as waiting for a digest (DEL-Q06)

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
