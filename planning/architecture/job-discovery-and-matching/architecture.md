# Job Discovery and Matching Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`). See "Where Each Area's Material Is" in `planning/architecture/AGENTS.md` for this area's lane and pages. Items are `Proposed` until the owner accepts them. Accepted decisions override the Initial Architecture.

## Scope

**Owns:** Greenhouse board catalog and user-added Greenhouse sources; subscriptions as a domain concept; scheduled/manual discovery; posting identity and lifecycle; filtering, matching, ranking, cost accounting, and durable handoff to delivery. Luke Campbell owns this area; Ryan Fusco contributes to ranking.

**Does not own:** sign-in, public endpoints, settings and dashboard (Graham / web-app-and-api); resume parsing and term dictionary (Ryan / experience-bank); PDF rendering and email (Ryan / delivery); table definitions (shared database); provisioning and deployment (shared build-and-operation).

**Current scope:** Greenhouse only, explicitly requested by Luke on 2026-11-18. No GitHub, Lever, Ashby, arbitrary careers pages, Chromium, Playwright, or LLM webpage extraction. A user-added source means a public Greenhouse board saved privately to that user, not an authenticated/private Greenhouse API. Initial Architecture below remains the historical baseline.

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

1. Multiple board formats and JavaScript scraping multiply failure modes and image size. Use one HTTP JSON adapter (JDM-001).
2. Hashing company/title/location merges distinct openings and changes identity when labels change. Use board token plus Greenhouse job-post ID (JDM-002).
3. A database write followed by SQS send can fail between steps. Persist resumable work and reconcile it; queue delivery is at least once (JDM-004).
4. New subscribers must receive existing active postings, not only globally new postings. Reconcile all active postings for a source on each successful poll (JDM-003).
5. AI ranking every job before filtering wastes budget. Apply explicit user filters first, then one structured ranking call per eligible user/posting/bank revision (JDM-005).
6. Checking spend after a call permits concurrent overspend. Reserve a conservative upper bound atomically before calling, and defer durably when unavailable (JDM-006).
7. A posting disappearing from a failed or incomplete response is not evidence it closed. Change availability only after a complete valid board response (JDM-002).
8. The original JavaScript and cross-provider demo no longer represents the requested product. SYS-Q04 records the group-level demo change; it does not block drafting this Greenhouse design.

## Decisions

Decision authority: JDM-001 records Luke's explicit instruction. The remaining choices are concrete recommendations made under Luke's request to use best judgment, kept Proposed where implementation or another owner's contract still needs agreement. The 2026-11-18 urgent-question pass records Luke’s decisions under his explicit request to answer assigned questions using best judgment. Ryan’s recorded acceptances are preserved; no agreement is attributed to Graham.

### JDM-001: Discover jobs only through the Greenhouse Job Board API
Status: Accepted (Luke Campbell, explicit scope instruction, 2026-11-18)
Date: 2026-11-18
Context: Luke requested Greenhouse only to simplify the project.
Decision: Use the public Job Board API, not Harvest. Fetch `GET https://boards-api.greenhouse.io/v1/boards/{board_token}/jobs?content=true`. GET access requires no authentication. Accept a board token or a recognized Greenhouse board URL; extract a validated token, then construct the API URL on the fixed host. Do not fetch arbitrary submitted URLs or follow redirects to arbitrary hosts. The provider supplies post IDs and descriptions. No browser or LLM is needed to extract listings.
Reasoning: One adapter removes browser packaging, website-specific selectors, and three other provider integrations while retaining discovery and tailored resumes.
Alternatives: General-purpose scraping and other providers are out of scope for this version.
Consequences: API help text, catalog data, scraper image requirements, demo and diagrams need the Greenhouse scope. Group-owned goal/demo wording is tracked in SYS-Q04. Board choice is explicit; this API is not a global search of every Greenhouse customer.
Evidence: [Greenhouse Job Board API](https://docs.greenhouse.io/job-board), Authentication and List jobs, checked 2026-11-18.

### JDM-002: Use stable provider identity and explicit availability
Status: Proposed
Date: 2026-11-18
Context: Greenhouse-only discovery does not need fuzzy cross-provider deduplication.
Decision: `postingKey = gh:{boardToken}:{jobPostId}`; use the API's job-post `id`, not `internal_job_id`. Preserve the validated board token's spelling. Catalog sources use `gh:{boardToken}`; user-added sources use `private:{sub}:gh:{boardToken}`. Shared posting records contain only public job content; never expose another user's private source or subscription. The same board added through either route maps to the same posting. Distinct post IDs remain distinct, even if titles match.
Reasoning: Stable provider keys avoid accidental merging and make repeated discovery idempotent.
Alternatives: Company/title/location hashes and fuzzy merging are unnecessary and less reliable here.
Consequences: On a complete successful poll, upsert observations, set `lastSeenAt`, `expiresAt = lastSeenAt + 30 days`, and mark absent postings inactive for that board. Failed/oversized/malformed responses leave availability unchanged. Read paths check active state and expiry explicitly; TTL deletion is not the availability clock. Matches retain an immutable bounded posting snapshot. A reopened same-ID job does not create a second match or email. See DB-I04.

### JDM-003: One recurring dispatcher, with hourly board polling
Status: Proposed
Date: 2026-11-18
Context: Per-source schedules are extra lifecycle work for this small project.
Decision: One dispatcher runs every 15 minutes, enqueues due source polls (hourly by default), and resumes due work. Manual refresh uses the same queue with an atomic five-minute source cooldown. Start with a small curated Greenhouse catalog. User-added boards remain supported. API writes source/subscription records through the agreed JDM validation contract (JDM-I01); the seed script writes catalog entries. Scraper rechecks source existence and authorization scope before working. Digest scheduling remains delivery's separate concern.
Reasoning: A single dispatcher avoids creating/deleting schedules with every source and also handles deferred work.
Alternatives: One schedule per source is deferred until scale justifies it.
Consequences: A successful poll reconciles every active posting with current subscribers, rechecking authoritative subscriptions rather than trusting a stale subscriber index alone. This handles new subscribers, missing banks, and prior enqueue failures. Dispatcher uses paginated, filtered table scans initially, checkpointing across bounded invocations; this is a small-demo tradeoff, not a scaling claim. OPS-I05 covers resources and limits; DB-I04 covers access patterns. Failed sources are isolated and visible.

### JDM-004: Persist work before queueing; make retries safe
Status: Proposed
Date: 2026-11-18
Context: DynamoDB writes and SQS sends are not one transaction.
Decision: Store per-user work under `WORK#` before sending a match-queue reference. New-match work ID is deterministic for `(sub, postingKey)`; regeneration uses `(sub, postingKey, requestId)`. Queue messages refer to work IDs. Claim work with a conditional lease and fencing generation. Persist ranking output before delivery, then transactionally create/update the match and mark the work `render_pending`. Send the stored render payload and mark work `complete` only after SQS acknowledges. Dispatcher retries due work and expired leases. Every state update checks lease ownership/generation; duplicate messages cannot overwrite newer work.
Reasoning: A crash can cause a repeated send, but cannot permanently lose a committed match. Saving ranking output also avoids needless repeat AI calls.
Alternatives: DynamoDB Streams/outbox infrastructure is deferred; silent best-effort enqueue is rejected.
Consequences: Renderer must deduplicate by `(sub, postingKey, docId, requestId)` and reject stale generations (JDM-I03). Only one render-changing operation per match is active at a time; regeneration waits until the prior delivery operation is terminal, preventing old workers replacing new PDFs. New-match conditional creation preserves one match per user/posting. Recheck subscription before starting a new match; regeneration authorizes the owned match instead. DLQ messages remain recoverable by operator redrive; exhausted attempts are `failed`, not automatically requeued forever. Delivery uses Ryan’s at-most-once email policy (DEL-018): an uncertain send is not retried automatically and may result in no email; the dashboard retains the result.

### JDM-005: Filter deterministically, then rank existing bank IDs in one call
Status: Proposed
Date: 2026-11-18
Context: Discovery and resume tailoring have different responsibilities.
Decision: All subscribed postings passing configured filters become candidates. Empty filters mean no restriction. Within title or location include lists use OR; across dimensions use AND; exclusions win. Match case-insensitive normalized plain text; do not infer remote eligibility from missing location. API settings schema must agree (JDM-Q01). Read the bank only through EB-I03; EB-I02 is deferred in v1 (JDM-008). Missing bank leaves work `waiting_bank`; exclude unclassified sections. One Bedrock call ranks role/project/bullet/skill IDs using the job description and bank, with no generated resume text or separate requirement-extraction call.
Reasoning: Explainable filtering limits AI spend. One ranking call is simpler than a separate extraction pipeline, and ID validation preserves the student's words.
Alternatives: AI decides whether a user qualifies, or rewrites resume bullets: rejected for opacity and verbatim risks.
Consequences: Ranking returns finite scores in [0,1], with one common relevance rubric across roles, projects, and their bullets. Validate all IDs against the same `docId`, parent relationships, uniqueness, allowed sections, ordering and score ranges. Stable ID breaks ties. Reject malformed/hallucinated output; one bounded retry reserves a fresh cost allowance, then visible failure. Job descriptions are untrusted data, never instructions or executable HTML. Recheck bank version before commit; changed bank returns to `waiting_bank`/rerank instead of rendering stale IDs. Requests over configured model input/output limits fail visibly rather than silently dropping resume sections.

### JDM-006: Share conservative cost reservations; defer without a retry storm
Status: Accepted (Luke Campbell, delegated urgent-question answers, 2026-11-18)
Date: 2026-11-18
Context: Matcher and bank parser share the proposed hourly and daily caps.
Decision: JDM-I04 provides a shared reservation package to both callers. Reserve integer micro-USD against all three counters atomically before every attempt: $1/user/hour, $5/platform/hour, $1.50/platform/UTC day, configurable per OPS-002/010. Use a configured model price and an upper bound from input tokens plus maximum output tokens. Each attempt gets a unique reservation ID; uncertain outcomes remain charged conservatively and cannot be reused for another invocation. Record actual usage separately; no refund in v1. Missing/unknown pricing fails closed.
Reasoning: Conservative accounting is simpler than safe refund/reconciliation and prevents concurrent calls from passing independent checks.
Alternatives: Check-after-call and SQS retrying until midnight are rejected.
Consequences: When denied, persist `deferred_budget` and `nextAttemptAt` at the latest reset among the exhausted windows, then acknowledge SQS. Dispatcher resumes later. A single call larger than a whole cap becomes `blocked_configuration`, not an infinite daily retry. Bank-ready work and backlog use oldest-due-first ordering within bounded pages. $1.50/day can consume $46.50 in a 31-day month; it is not a guarantee that total AWS charges fit $50. Parser and Matcher use the same shared package (EB-004, DB-I04); model choice remains an OPS configuration decision.

### JDM-007: Preserve delivery's ranked payload, digest and regeneration behavior
Status: Accepted (Luke Campbell, delegated urgent-question answers, 2026-11-18)
Date: 2026-11-18
Context: DEL-010/011/014 already describe Ryan's expected Matcher behavior.
Decision: Supply the ranked shape in JDM-I03. New hourly/daily matches get the DB-I03 digest mark atomically with creation. Regenerations get no new digest mark and no email. Upload-triggered regeneration selects owned, unapplied matches whose postings are active and unexpired; per-match regeneration uses the retained posting snapshot even when the posting has closed. Read current bank and emit `reason: regenerate`.
Reasoning: This preserves Ryan's product behavior while making retry and stale-version handling explicit.
Alternatives: Regenerate every historical match automatically: rejected because users may already have submitted those resumes.
Consequences: Luke accepts DEL-010/011/014/017/020 and JDM-I03. At creation, hourly matches target the next strictly future UTC hour; daily matches target the next strictly future 13:00 UTC. At exactly a send boundary, target the following window to avoid racing its reader. Set `GSI2PK=DIGEST#hourly#{YYYY-MM-DDTHH}` or `DIGEST#daily#{YYYY-MM-DD}` and `GSI2SK={sub}#{createdAt}#{postingKey}` atomically with the match, following DB-I03. Instant matches and regeneration requests get no new digest keys. UTC timing avoids per-user timezone scheduling; it deliberately shifts local clock time with daylight saving. Shared database acceptance remains pending.

### JDM-008: Close the urgent delivery, bank and identity handoffs
Status: Accepted (Luke Campbell, delegated urgent-question answers, 2026-11-18)
Date: 2026-11-18
Context: Ryan accepted JDM-I03/I04 and supplied DEL-020, but Luke’s responses were blocking delivery and parser planning.
Decision:
- **DEL-Q15:** use **postingKey = gh.{boardToken}.{jobPostId}**. This overrides only the colon-delimited posting-key spelling proposed in JDM-002; source IDs are unchanged. Board tokens cannot contain dots under JDM-I01 and job-post IDs are decimal strings, so the parts are unambiguous. There is no deployed data migration in this planning-only repository.
- **DEL-Q13:** accept ranked role/project/bullet/skill IDs with comparable scores, request ID/generation checks, digest marks and the regeneration rules in JDM-007/I03. At-most-once email (DEL-018) can miss an email after an uncertain send; do not silently retry it. Users can still retrieve the resume from the dashboard.
- **EB-I03:** accept the bank package as the Matcher’s sole bank-access interface, excluding unclassified items. Defer EB-I02 and canonical-term dictionaries in v1, as Ryan requested; no required dictionaryVersion on new matches until that feature returns.
- **DB-Q05/08:** preserve an immutable bounded posting snapshot on each match; retain PDF history in the S3 match folder per DEL-009, with only the current pdfKey on the match. No growing history array or extra history item type.
Reasoning: These choices align with the consumers’ existing work, eliminate a dictionary dependency and duplicate history storage, and settle a single key spelling before code exists. Dots simplify URL handling; this is a naming convention, not a claim that colons are forbidden in S3. URL construction/signing must remain consistent for every field.
Alternatives: Fuzzy identity, dictionary normalization now, and a second PDF history store add work without helping the current Greenhouse demo.
Consequences: JDM-I03/I04 are agreed at the producer/consumer boundary. Shared DB/OPS proposals retain their required group approval. JDM-002/005 remain Proposed for their other behavior. The implementation plan and current RFC carry these resolved details.

### JDM-009: Settle source IDs and filter matching for the API
Status: Accepted (Luke Campbell, delegated urgent-question answers, 2026-11-18)
Date: 2026-11-18
Context: Graham added WEB-Q04 and WEB-Q06 after the original urgent answers; DB-I05 now defines the settings schema.
Decision: Catalog source IDs are gh.{boardToken}; private source IDs are private.{ownerKey}.gh.{boardToken}, where ownerKey is unpadded base64url of the UTF-8 Cognito sub. Preserve board-token spelling. This overrides only the source-ID spellings in JDM-002; posting keys follow JDM-008. Authorize using the source record’s ownerSub, never by trusting a decoded path segment.
Decision: Filters use literal substring matching after Unicode NFKC normalization, case folding, trimming and collapsing whitespace on both term and field. No regex, stemming, synonym dictionary or whole-word requirement. Thus intern matches Intern, Internship and Internal; the latter is a known false positive of this deliberately simple rule. Title exclusions take precedence; OR within each include list and AND between title/location dimensions. Empty includes impose no restriction. A missing location fails a nonempty location filter, and is never inferred to mean remote.
Reasoning: One separator convention simplifies route construction; base64url keeps any valid sub unambiguous without assuming its character set. Substring matching makes Graham’s intern default useful with no dictionary dependency, at the cost of some visible false positives.
Alternatives: Colon-separated IDs, UUID-only assumptions, whole-word matching and semantic classification add either inconsistent handling or unnecessary complexity for this version.
Consequences: Luke accepts DB-I05 as a reader/co-owner: daily delivery, titleIncludes=[intern], empty remaining filters, 20 terms/list and 1–60 trimmed characters/term. Missing Settings means no matching work, not implicit defaults. Existing matches keep their original delivery mode. DB-I05 still needs Ryan’s acceptance. JDM-I01 remains Proposed pending Graham’s agreement to the final source contract.

### JDM-010: Route regeneration through persisted JDM work
Status: Proposed (Luke’s concrete answer to WEB-Q03). Ryan Fusco agrees (2026-11-19). Awaiting Graham.
Date: 2026-11-18
Context: WEB-T10 needs a durable way to retain the upload choice and reach the Matcher. The parser already publishes the current bank through its package.
Decision: Use JDM-I05 below. The API persists an upload-regeneration work record only when regenerate=true, keyed by the authenticated sub and docId, before returning the upload response. JDM’s existing dispatcher waits for that exact upload to become the successful current bank, then expands it into per-match work. The parser does not send a second regeneration event. Per-match API requests persist work before best-effort match-queue enqueue; dispatcher recovery covers a failed enqueue.
Reasoning: Reusing the existing work store and dispatcher avoids a new queue, event service or parser-to-Matcher callback. The choice cannot disappear with a failed message send, and a failed upload cannot regenerate against the old bank.
Alternatives: API fire-and-forget queue messages lose intent on failures; parser-triggered fanout adds a second owner and duplicate-trigger path.
Consequences: Graham must agree API writes/idempotency semantics; Ryan must agree per-upload status lookup through EB-I03. DB-I04 and OPS-I05 must include these work fields/API permissions before implementation. This answer is reviewable but is not recorded as their approval.

## Interfaces

### Provides

#### JDM-I01: Greenhouse sources, subscriptions and refresh
Status: Proposed
Consumers: web-app-and-api
Contract: Shared source validation accepts a token matching `[A-Za-z0-9_-]{1,128}` or HTTPS board URL on `boards.greenhouse.io` / `job-boards.greenhouse.io` with a single token path. Reject arbitrary hosts, credentials, ports, extra paths and unsupported providers; token input handles boards not covered by the URL parser. Catalog/private IDs follow Accepted JDM-009; this overrides the old JDM-002 spellings. API derives `sub` from JWT, writes sources/subscriptions (DB-I04), and authorizes catalog access or private-source ownership. Refresh persists `nextPollAt=now` subject to cooldown before best-effort queue send, so dispatcher recovers failed sends. Deletion disables future work; unsubscribe does not delete historical matches. 400 invalid source, 404 missing/not owned, 409 duplicate, 429 cooldown. Returns source state and `lastSuccessAt`/`lastErrorCode`; errors contain no resume data. Subscription creation takes effect by the next successful poll.
Accepted by: Luke Campbell (2026-11-18, provider). Waiting for: Graham Cohick (consumer), including JDM-009 ID format.

#### JDM-I02: Internal work queues
Status: Proposed
Consumers: JDM dispatcher, scraper, Matcher; build-and-operation
Contract: UTF-8 JSON with `schemaVersion: 1`. Scrape payload `{sourceId, requestId}`; source record supplies the validated board token and scope. Match payload `{sub, workId}`; canonical work record supplies posting, reason and current generation. No arbitrary URLs or resume text in queues. Unknown versions/malformed payloads are rejected with structured errors and bounded retry/DLQ handling. Source/work IDs are opaque strings to infrastructure. All handlers are idempotent; partial SQS batch failures report only failed messages.
Accepted by: —

#### JDM-I03: Render request
Status: Accepted
Consumers: delivery
Contract: `schemaVersion:1`, `sub`, `postingKey`, `docId`, `requestId`, `generation`, `reason` (`new_match` or `regenerate`), `rankedExperiences:[{roleId,score,rankedBullets:[{bulletId,score}]}]`, `rankedProjects:[{projectId,score,rankedBullets:[{bulletId,score}]}]`, `skills:[{category,score,rankedSkills:[skillId]}]`. Scores use JDM-005. Education and allowed unranked sections come from EB-I03. No rewritten text. Serialize at most 200 KiB (an application limit); oversized output fails visibly. Same work retry sends byte-equivalent stored payload. Delivery uses generation and request ID for conditional ownership and deduplication, preserves old PDFs, and suppresses regeneration email. Producer and delivery consumer agree this payload; shared DB schema approval remains separate.
Accepted by: Ryan Fusco (2026-11-18, consumer: delivery, recorded as DEL-017). Accepted by: Luke Campbell (2026-11-18, provider).

#### JDM-I04: Shared Bedrock budget reservation
Status: Accepted (provider and parser consumer; shared storage/provisioning approval remains separate)
Consumers: experience-bank, Matcher; database and build-and-operation
Contract: `reserve(sub, attemptId, upperBoundMicroUsd, nowUtc)` returns `reserved`, `already_reserved` (not permission to invoke again), `deferred(nextAttemptAt)`, or `blocked_configuration`. Transaction writes one unique reservation and increments three counters only if all caps permit it. Counter window identifiers are UTC and unambiguous (`HOUR#...`, `DAY#...`). Retry of the transaction with the same attempt cannot double charge; a new actual invocation needs a new attempt ID. Counters/reservations remain at least seven days beyond the window. Crash/timeout after reserving may waste allowance but never release uncertain spend. Caller logs actual tokens and estimated cost separately without logging bank/job content. See JDM-006 and DB-I04.
Accepted by: Ryan Fusco (2026-11-18, consumer: experience-bank, recorded as EB-017). Accepted by: Luke Campbell (2026-11-18, provider). Shared database and build-and-operation parts follow DB-I04 and OPS-I05.

#### JDM-I05: Regeneration command intake
Status: Proposed
Consumers: web-app-and-api (command producer), JDM dispatcher/Matcher; experience-bank (status provider), database
Contract:
- Reuse USER#{sub}/WORK#{workId}; workType distinguishes upload_regenerate, per_match_regenerate and existing new_match work. API gets sub from the authenticated token. Store requestId, requestedAt, reason=regenerate, state, and targetDocId or postingKey as applicable. JDM owns state transitions after creation.
- Upload workId is regen-upload:{docId}; targetDocId is fixed. Write conditionally only for regenerate=true. The dispatcher reads the package’s per-upload status and current bank. Only successful parsed/parsed-with-unclassified targetDocId equal to the current bank may fan out. Failed or superseded uploads finish without regenerating; an upload never completed expires as failed upload_not_completed after 24 hours. No inference from a previous bank’s success. Ryan must expose per-upload status keyed by sub/docId through EB-I03; no direct DOC# reads.
- Fanout selects active/unexpired/unapplied matches under JDM-007. Child work IDs derive deterministically from parent requestId and postingKey; store a continuation cursor so retries cannot lose later pages. Recheck targetDocId before each child starts; if replaced again, skip that old command. Regeneration never creates a new digest mark or email.
- Per-match requests require an Idempotency-Key (UUID), used as requestId under the authenticated sub and postingKey. Atomically create work and claim the match’s activeRegenerationRequestId. Repeating the same key for the same match returns the same 202/requestId; another key while active returns 409; a key reused with a different target returns 409. Missing/invalid key returns 400. Ownership failure returns 404. This HTTP addition needs Graham’s agreement.
- After commit, best-effort enqueue JDM-I02 {sub,workId}; persisted pending work is the source of truth if enqueue fails. Generation is allocated atomically when the operation is admitted; retries reuse it. Clear an active request only with a conditional requestId/generation check on terminal completion/failure. Pending render handoff must not release it prematurely.
Accepted by: Luke Campbell (2026-11-18, JDM side), Ryan Fusco (2026-11-19, upload-status provider: `upload_status(sub, docId)` in EB-019). Waiting for: Graham Cohick; shared DB/OPS contracts remain Proposed.

### Consumes

- EB-I03 bank package: use `get_bank`, never read bank items directly. Luke accepts this consumer contract (JDM-008). EB-I02/term dictionaries are deferred in v1; provider/other-consumer signoff on EB-I03 remains separate.
- DB-I01 table conventions, DB-I03 delivery fields and DB-I04 discovery/work/cost schema. JDM proposes replacing LSI1 with a matches-by-date GSI; existing shared conventions remain pending agreement.
- OPS-I01 to OPS-I04 deployment, storage, permissions and metrics; proposed amended intake OPS-I05.
- API-owned settings, authenticated commands and upload-completion/regeneration signal: JDM-Q01 asks Graham for the producer contract. Use existing WEB-T07 and WEB-T10; JDM-I05 is the proposed regeneration intake.

## Open Questions

- **JDM-Q01:** Graham: agree JDM-I01, settings (`titleIncludes`, `titleExcludes`, `locationIncludes`, `deliveryMode`), and durable regeneration command producer contract. Upload regeneration must wait for successful bank replacement; per-match commands must authorize `sub` and use an idempotent request ID. Blocks JDM-T03/T07/T09; API task IDs now exist: WEB-T07 and WEB-T10.
  - **Luke’s answers (2026-11-18 restoration):** WEB-Q04/06 are answered by JDM-009; DB-I05 has Luke’s acceptance; WEB-Q03 has a concrete proposed contract in JDM-010/I05, awaiting Graham/Ryan.
  - **Graham Cohick's answers so far (2026-11-18):** the four settings, their limits, and their defaults are proposed in DB-I05. The endpoints are decided in WEB-001. Still open on his side: JDM-I01 (waiting on source ID format, WEB-Q04) and the regeneration producer contract (WEB-Q03).
- **JDM-Q02:** Ryan: agree JDM-I03 generation/deduplication contract, EB-I02/03 usage, and JDM-I04 parser cost reservations. Confirm how SES send/record failures are reconciled: an SQS/digest conditional write alone cannot guarantee exactly-once external email. Blocks JDM-T06/T08/T09 and full duplicate-email demo signoff. Provider implementation task IDs do not exist yet.
  - **Ryan Fusco's answers (2026-11-18):**
    - JDM-I03 accepted, including `requestId` and `generation` (DEL-017).
    - Resume keys use the generation (DEL-009).
    - JDM-I04 accepted for the parser (EB-017).
    - Emails are sent **at most once** (DEL-018).
    - The term resolver (EB-I02, EB-007) is deferred, so the Matcher doesn't depend on it in v1.
    - Separately, Ryan recommends a URL-safe `postingKey` separator (DEL-Q15).
    - Digest index keys are now supplied by DEL-020; Luke agrees under JDM-007. Shared DB approval remains pending.
  - **Luke’s answer (2026-11-18):** JDM-I03/I04 and DEL-Q13/15 resolved by JDM-006/007/008. EB-I03 accepted as consumer; EB-I02 deferred. JDM-Q02 no longer waits for Luke’s design answer. Provider implementation task IDs and overall EB/DB contract signoff remain outstanding.
- **JDM-Q03:** Group: accept DB-I04 and OPS-I05, including GSI instead of LSI1 and dispatcher scan tradeoff; attach DB/OPS task IDs when written. Blocks integration/deploy, not completion of this planning draft.
- **JDM-Q04 (risk, raised by Ryan Fusco as contributor, 2026-11-18): Is there a simpler fallback if time runs short?** No action is needed now. This is a watch item.
  - **The concern:** the project is due in about a month. This design includes:
    - an 8-state work machine
    - leases with fencing generations
    - checkpointed table scans
    - a reservation ledger

    Most JDM tasks are blocked on cross-area agreements.
  - **Possible fallbacks** (Luke's call), if a checkpoint the group picks (for example, the end of November) arrives with JDM-T02 and JDM-T04 unfinished:
    - Drop the scan checkpoints. A small Greenhouse catalog fits in one DynamoDB scan page.
    - Use SQS visibility timeouts plus conditional status changes instead of leases and fencing. Keep only the match `generation` (DEL-017).
    - Cut the work states down to about four: pending, deferred, done, and failed.
  - Owner: Luke Campbell
  - Blocks: nothing yet
- **SYS-Q04 (defined in architecture/AGENTS.md):** all three owners now agree; shared RFC wording updated in this restoration.

### Planning exit check (2026-11-18)
- Scope and review drafted; frozen baseline preserved.
- JDM-001/006/007/008/009 are Accepted; JDM-I03/I04 have producer/consumer agreement. JDM-010/I05 regeneration intake and remaining shared approvals are explicitly Proposed.
- Interfaces drafted on provider side with requests to consumers; shared acceptances outstanding.
- Ordered implementation plan exists; blocked tasks name unresolved contracts.
- Current diagram includes proposals with dashed borders.
- Area stays Draft until cross-area contracts are agreed; no phase change or application code is part of this work.

## Requests From Other Areas

Update 2026-11-18: recommended answers now appear in JDM-003 to JDM-007, DB-I04 and OPS-I05. Answered Luke-owned pointers were removed on 2026-11-18; outstanding requests below still need action. Luke’s SYS-Q01/02/03 confirmations are recorded at source; other owners’ approval remains separate.

Pointers to items defined in other areas that this area's owner has to act on. Answer each one where it's defined, then delete its row (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).

| ID | Defined in | What's needed |
|---|---|---|
| DB-001 | `database/architecture.md` | As a co-owner, accept on-demand capacity for the table (answers DB-Q04). |
| EB-I03 (EB-019 extension) | `experience-bank/architecture.md` | Accept `upload_status(sub, docId)`, which reports a specific upload and whether it is the current bank, for JDM-I05. |
| EB-Q10 | `experience-bank/architecture.md` | Confirm the experience-bank wording in the RFC's Summary, Goal 3, and Demo step 1 (one LaTeX resume, verbatim bullets). |
| WEB-Q03 | `web-app-and-api/architecture.md` | Luke answered with JDM-010/I05. Await Graham/Ryan’s agreement to API idempotency and per-upload bank status; no further Luke design answer pending. |
| DB-I01 to DB-I03 | `database/architecture.md` | As a co-owner, accept the table conventions, the experience items, and the match delivery state. The Matcher writes the digest mark. |
| OPS-001 to OPS-007, OPS-I01 to OPS-I04 | `build-and-operation/architecture.md` | As a co-owner, accept or challenge Ryan Fusco's answers: the state lock, the hourly caps, the image split, CI, the Bedrock model and region (OPS-006 sets a separate model for each Matcher task), and the repo layout. Also accept the four contracts. |
| OPS-008 to OPS-010 | `build-and-operation/architecture.md` | As a co-owner, accept staying in the SES sandbox, the $50 monthly budget alert, and the $1.50 daily Bedrock cap. |
