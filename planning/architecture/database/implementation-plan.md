# Database Implementation Plan

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This is a shared area, so every owner must agree to changes to this plan.
Date: 2026-11-19. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: the table conventions (DB-I01), the item contracts (DB-I02 to DB-I05), and access patterns DB-AP01 to DB-AP20. DB-001 and DB-I01/I03/I04 base/I05 are Accepted as of 2026-11-21. DB-I02 awaits the expiry separation in DB-Q10; the DB-I04 regeneration amendment needs separate agreement. Each area writes its own items, so this plan covers only what every area shares: the table definition, one set of key builders, and a test that each access pattern works. No application or test code exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision or interface still needs agreement before the task can be built.

**Order.** The table definition (T01) comes first, and `build-and-operation` provisions it (OPS-T04). Then the key builders (T02), which every area's code imports. Then the access-pattern tests (T03).

| Waiting on | Tasks it blocks |
|---|---|
| DB-I02: agree the DB-Q10 upload-expiry separation | T02, T03 |

DB-I05 is Accepted.

### DB-T01: Define the table
Depends on: DB-I01, DB-I04, DB-Q02, DB-001
Acceptance criteria:
1. One table per environment, with string key attributes `PK` and `SK`, and on-demand capacity.
2. **GSI1** (`GSI1PK`, `GSI1SK`) serves catalog sources, a student's private sources, a source's subscribers, and a board's postings.
3. **GSI2** (`GSI2PK`, `GSI2SK`) serves the digest buckets: `DIGEST#hourly#{UTC-hour}` and `DIGEST#daily#{UTC-date}`, sorted by `{sub}#{createdAt}#{postingKey}`.
4. **GSI3** (`GSI3PK`, `GSI3SK`) serves a student's matches by creation date. There is no local secondary index.
5. Each index's projection holds only the fields its readers need. Detail reads go to the base item.
6. The TTL attribute is `expiresAt` (epoch seconds), used by postings, cost counters, reservations, and replaced bank items.
7. The definition is a single Terraform input that `build-and-operation` consumes in OPS-T04.
Status: Planned

### DB-T02: Shared key builders and the item-type registry
Depends on: DB-T01, DB-I01 to DB-I05, JDM-008, JDM-009, DEL-020
Acceptance criteria:
1. A small package in `libs/` (for example `libs/froop_keys`) builds every `PK`, `SK`, and index key in this design:
   - `USER#`, `SOURCE#`, `POSTING#gh.{boardToken}.{jobPostId}`, `COST#HOUR#`, `COST#DAY#`, and `SYSTEM#JDM`
   - the `DOC#`, `SUB#`, `MATCH#`, `WORK#`, and `COSTRES#` sort keys
   - the GSI keys, including the digest bucket for the next send
2. Source IDs follow JDM-009: `gh.{boardToken}` and `private.{base64url(sub)}.gh.{boardToken}`.
3. Every item carries `itemType`. A test fails if code writes an `itemType` that isn't in the Item Types table.
4. A helper checks an item's serialized size against the 300 KiB ceiling before it's written.
5. Every area's code imports these builders rather than formatting keys itself. Experience items are still written only through the experience-bank package (EB-010), which uses these builders internally.
Status: Blocked (DB-I02 / DB-Q10; regeneration key additions also await the DB-I04 amendment)

### DB-T03: Test every access pattern
Depends on: DB-T01, DB-T02
Acceptance criteria:
1. For each access pattern DB-AP01 to DB-AP20 (except DB-AP12, which is superseded), a test against DynamoDB Local writes sample items and answers the pattern with the stated key or index, using a query, not a scan.
2. The two scans that are allowed, DB-AP15 and DB-AP16, are paginated, and their tests resume from a saved position.
3. Conditional writes block a second posting with the same key and a second match for the same student and posting.
4. A query for one student never returns another student's items.
5. When a new access pattern is added to `architecture.md`, it gets a test here before any code relies on it.
Status: Blocked (DB-T02 / DB-I02 / DB-Q10)
