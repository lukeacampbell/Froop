# Job Discovery and Matching Implementation Plan

Owner: Luke Campbell. Contributor: Ryan Fusco (Matcher/ranking).
Date: 2026-11-18. Phase: Architecture Planning. State: Draft.

This is the Greenhouse-only build plan. JDM-001/006/007/008/009 are Accepted; JDM-I03/I04 have provider/consumer agreement. Other design choices and shared schemas remain Proposed. Blocked means a named planning dependency needs agreement before implementation, not that the task has been attempted. No application or test code exists in this checkout. Phase 2 begins only after human confirmation under `planning/AGENTS.md`.

Recommended order: adapter and identity → data/recovery foundation → API and polling → ranking/budget → delivery/regeneration → integration. Each Phase 2 task includes unit tests for its acceptance criteria; cross-area/system test plans belong to Phase 3.

**2026-11-21 dependency update:** JDM-I01 and base JDM-I05 have producer/consumer agreement; DB-I01/I03/I04 base and OPS-I05 are Accepted. Remaining regeneration blocker: DB-I04 amendment A (drafted 2026-11-23; Ryan and Graham accept, Luke to review). WEB-Q08/JDM-011 and OPS-I01 to I04 were settled on 2026-11-23. Keep conditional generation ownership, pagination and atomic reservations; review JDM-T02/T04 on November 30 (JDM-012). Provider tasks now include DB-T01/T02, OPS-T02 to T08, EB-T01/T02 and DEL-T04 to T07. Phase 1 remains current.
### JDM-T01: Implement the Greenhouse adapter
Depends on: JDM-001; approved Phase 2 transition
Acceptance criteria:
1. Fetches the fixed public Job Board endpoint with descriptions, without credentials, Chromium, Playwright, or other providers.
2. Validates token/recognized board URL; arbitrary hosts, credentials, ports, malformed tokens and redirects off the fixed API host cannot trigger a fetch.
3. Parses post ID, title, location, URL and description; missing required fields or malformed/partial/oversized responses fail the poll without changing availability. HTML is converted to safe plain text, never executed.
4. A configured 10-second HTTP timeout, 5 MiB response limit and at most three attempts bound work. 429/5xx/timeouts use bounded backoff and honor Retry-After within remaining execution time; persistent errors are visible. These are application defaults, not provider rate-limit claims.
Status: Planned (Phase 1 only; JDM-001 accepted)

### JDM-T02: Implement posting identity, snapshots and work persistence
Depends on: JDM-T01, JDM-002, JDM-004, JDM-008, DB-I04
Acceptance criteria:
1. Uses `gh.{boardToken}.{jobPostId}` (JDM-008). The same board/post ID discovered repeatedly or through catalog/private sources has one posting; distinct IDs with equal titles remain distinct.
2. A complete successful poll updates last-seen/expiry and marks missing jobs inactive; failure does neither. Reads exclude inactive/expired jobs even before TTL removal.
3. Conditional match creation allows one match per user/posting and preserves its creation time and immutable snapshot.
4. Snapshot content over 128 KiB UTF-8 fails visibly before creating the match; every serialized item is checked under the application 300 KiB ceiling.
5. Work creation is durable before enqueue; failed sends can be recovered, claims use leases/fencing, and stale workers cannot commit.
Status: Blocked (JDM-002/004 remain Proposed; DB-I04 base Accepted)

### JDM-T03: Integrate sources, subscriptions and manual refresh
Depends on: JDM-T02, JDM-003, JDM-I01, JDM-009, WEB-T07, JDM-Q01
Acceptance criteria:
1. Seed operation is idempotent for catalog boards; API owns user-added source and subscription writes under the agreed schema.
2. Two users cannot read/modify each other's private source records; public posting reuse reveals no private source membership.
3. Refresh cooldown is atomic across concurrent requests; persistence precedes enqueue; failed enqueue still results in later dispatch.
4. Subscribe to an existing board produces eligible existing jobs on its next successful poll. Unsubscribe/deletion prevents new work but retains historical matches.
Status: Blocked (predecessor JDM-T02; JDM-I01 and DB-I04 base Accepted; WEB-T07 linked)

### JDM-T04: Build dispatcher and board reconciliation
Depends on: JDM-T02, JDM-T03, JDM-003/004, OPS-I05
Acceptance criteria:
1. One 15-minute tick dispatches due polls at a configurable hourly cadence; no schedule resource is created per source.
2. Paginated scans persist continuation checkpoints and finish across bounded runs without losing tail pages. Sources are conditionally leased; expired dispatch claims recover.
3. Every successful poll reconciles current subscribers against active postings, including posts seen before subscription and work missed by failed queue sends.
4. Due budget work and expired worker leases resume; terminal failures stop automatic retries and expose an operator redrive path.
5. One broken board does not stop other boards; metrics distinguish poll failure, queue failure and no new postings.
Status: Blocked (JDM-003/004 remain Proposed; DB-I04 base and OPS-I05 Accepted)

### JDM-T05: Build shared cost reservation package
Depends on: JDM-006, JDM-I04 (agreed), DB-I04, OPS-002/010
Acceptance criteria:
1. Concurrent reservations cannot exceed any configured per-user hourly, platform hourly or daily cap; all counter updates and unique reservation creation are atomic.
2. Repeating an attempt ID cannot charge twice or authorize a second model invocation. Every new invocation reserves separately, including model retries.
3. UTC hour/day boundaries and counter expiry are explicit; uncertain calls remain charged and actual usage is logged separately.
4. Denied work returns the correct latest exhausted-window reset; a call larger than an entire cap or unknown pricing returns blocked configuration.
5. Parser and Matcher use the same package contract; parser adoption is EB-T06 in `experience-bank/implementation-plan.md` (JDM-Q02).
Status: Planned (JDM-I04, DB-I04 base, OPS-I01 to I05 agreed)

### JDM-T06: Implement bank-aware structured ranking
Depends on: JDM-T02, JDM-T05, JDM-005, EB-I03, JDM-Q02
Acceptance criteria:
1. Reads bank exclusively via EB-I03; term resolver and dictionary version are not required in v1 (JDM-008); missing bank waits durably and unclassified sections are excluded.
2. Uses one structured ranking call per attempt, after filters, with configured model/token bounds and a successful unique cost reservation.
3. Rejects invented/duplicate IDs, wrong parent relationships, nonfinite/out-of-range scores and generated resume text. Ties are stable.
4. Job content cannot alter instructions, call tools or execute markup; stored bank text stays unchanged.
5. Persists validated ranking once; retries after persistence reuse it. Bank replacement before commit prevents stale output and triggers reranking against the current bank.
6. A small versioned sample set includes relevant, unrelated and misleading job descriptions for judging ranking quality before model selection; no unstated precision guarantee is assumed.
Status: Blocked (JDM-005 remains Proposed; predecessor JDM-T05; EB-I03 consumer acceptance recorded)

### JDM-T07: Apply settings and create matches
Depends on: JDM-T04, JDM-T06, WEB-T07, JDM-Q01, JDM-005/007/009, DB-I03/04/05
Acceptance criteria:
1. Per JDM-009, normalized literal substrings apply: intern matches Internship and Internal; missing Settings skips the user. Empty filters include all subscribed jobs; OR within include dimensions, AND across dimensions, exclusions win, and missing location never implies remote eligibility.
2. Rechecks authoritative ownership/subscription before new-match creation; filtered jobs incur no ranking calls.
3. Transaction creates the match, immutable snapshot, DB-I03 digest keys for the next strictly future hourly boundary or daily 13:00 UTC (including exact-boundary cases), and render-pending work consistently.
4. Duplicate poll/queue execution creates no second match and does not reset delivery state or resend completed work.
Status: Blocked (predecessor JDM-T06 and JDM-004/005; settings and delivery contracts agreed)

### JDM-T08: Deliver stored render payloads safely
Depends on: JDM-T07, JDM-I03, JDM-Q02
Acceptance criteria:
1. Payload matches JDM-I03, has only validated IDs, and is at most 200 KiB; oversize is a visible terminal error.
2. Crash before send is recovered; crash after send may resend the same stored payload, which delivery deduplicates by request/generation.
3. Old generations cannot overwrite a newer render; only one render-changing operation per match is active until delivery is terminal.
4. Ranking is not repeated on queue-send failure; complete work is not automatically replayed by polling.
5. Follows DEL-018 at-most-once email: uncertain sends are not automatically retried; dashboard availability remains and no exactly-once email claim is made.
Status: Blocked (JDM-T07; delivery tasks DEL-T04/T07 now exist; JDM-I03 agreed)

### JDM-T09: Add regeneration commands
Depends on: JDM-T08, WEB-T10, JDM-I05, JDM-Q01/02, JDM-007/010
Acceptance criteria:
1. Upload opt-in queues regeneration only after successful bank replacement and selects active, unexpired, unapplied matches.
2. Per-match regeneration authorizes ownership and uses retained posting details even when the original posting is gone.
3. Command retries with the same request ID do not create a second operation; new requests receive monotonically increasing generations.
4. Regeneration produces no new digest mark or email and keeps previous PDFs according to delivery's contract.
Status: Blocked (DB-I04 regeneration amendment; JDM-011 Accepted 2026-11-23; predecessor tasks pending; WEB-T10 linked)

### JDM-T10: Wire infrastructure, metrics and operating instructions
Depends on: JDM-T04/T05/T08/T09, OPS-I05, JDM-Q03, SYS-Q04
Acceptance criteria:
1. Dispatcher/scraper/Matcher use the agreed ingestion module, small independent images and least-privilege roles. Scraper has no Bedrock permission or browser dependency.
2. Queue visibility/worker timeouts, partial-batch failures, bounded receives, DLQs and alarms match OPS-I05; missing config fails clearly.
3. JSON logs and metrics cover poll success/failure, matches, work age, deferred budget, invalid ranking and spend without resume/job bodies or sensitive fields.
4. Operating instructions explain seed, refresh, investigate failure, fix cause and redrive without duplicate matches; documented limitations include scan scaling, conservative budget accounting and email uncertainty.
5. Group agrees revised Greenhouse demo: catalog plus user-added board, repeated poll without duplicate matches, tailored PDF, isolated board failure. Integration validation waits for the approved Phase 3/4 plan.
Status: Blocked (JDM-003/004 remain Proposed, through JDM-T04; OPS-I01 to I05 Accepted; shared Greenhouse demo wording is agreed)
