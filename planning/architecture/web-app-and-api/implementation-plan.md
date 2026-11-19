# Web App and API Implementation Plan

Owner: Graham Cohick.
Date: 2026-11-18, revised 2026-11-19. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: decisions WEB-001 to WEB-011 and WEB-013 (Accepted) and interfaces WEB-I01 and WEB-I02 (Proposed). No application or test code exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision, interface, or question still needs agreement before the task can be built. It doesn't mean the task was tried and failed.

**Order.** The API's foundations (T01) → infrastructure (T02 to T04) → API endpoints (T05 to T10) → the dashboard (T11 to T16). The dashboard shell (T11) depends only on this area's own decisions, so it can start alongside T01.

**Tests.** Each task includes unit tests for its acceptance criteria in Phase 2. Tests that cross areas, such as an upload that ends in a parsed bank, belong to the Phase 3 test plans.

**Tasks in other areas.** Tasks cite the other areas' task IDs, which Ryan Fusco supplied on 2026-11-19 (WEB-Q07).

**What's still in the way (2026-11-19).** Graham Cohick has accepted every contract these tasks depend on. What remains is other owners' acceptance of shared items, and one open question:

| Waiting on | Tasks it blocks |
|---|---|
| Luke Campbell accepting the shared `build-and-operation` decisions (OPS-004, OPS-005, OPS-007, OPS-012) | T03, T16 |
| Ryan Fusco and Luke Campbell accepting OPS-I01 to OPS-I04 | T02, T03, T04 |
| Luke Campbell accepting the shared `database` contracts (DB-I01, DB-I03, DB-I04) | T05, T07, T08, T09 |
| Luke Campbell accepting the EB-I03 extension (EB-019) | T06 |
| Luke Campbell and Ryan Fusco: WEB-Q08, and the regeneration amendment to DB-I04 | T10 |

## API foundations

### WEB-T01: Build the API's request handling
Depends on: WEB-001, WEB-005
Acceptance criteria:
1. `GET /health` returns 200 without a token and reads nothing from the table or S3.
2. Every other route in WEB-001 exists. A request that arrives without the authorizer's `sub` claim gets 401, and the handler for that route does not run.
3. A handler receives the user's `sub` only from the authorizer's claims. A `sub` supplied in the path, query, or body is never used as the user.
4. Every error response is `{"error": {"code", "message"}}` with status 400, 401, 404, 409, or 429. An unhandled exception returns 500 with no stack trace or internal detail in the body.
5. A JSON body with a field the endpoint doesn't define is rejected with 400.
6. Logs are JSON and include the route, the status, and `sub`. A test that sends a token, an email address, and resume text through each route finds none of them in the log output.
7. Test claims for local development can be supplied only when a development setting is on. With that setting off, as in every deployed environment, a request without authorizer claims gets 401.
Status: Planned

## Infrastructure

### WEB-T02: Provision sign-in
Depends on: WEB-002, WEB-004, OPS-011, OPS-I01, OPS-T02, OPS-T03
Acceptance criteria:
1. The `app` Terraform module creates a Cognito user pool, a hosted UI domain, and one public app client with no secret, limited to the authorization code flow with the `openid` and `email` scopes.
2. Sign-up requires a verified email address.
3. Self sign-up is controlled by one variable for each environment, and it is off by default. With it off, the hosted UI offers no sign-up and a sign-up request is refused. With it on, a new student can sign up.
4. Callback and sign-out URLs come from a variable for each environment. Localhost is allowed in dev and not in prod.
5. The module outputs the user pool ID, the client ID, and the hosted UI domain, for the API's authorizer and the dashboard's build.
Status: Blocked (OPS-I01 waits for Ryan Fusco and Luke Campbell)

### WEB-T03: Provision the API
Depends on: WEB-T01, WEB-T02, WEB-005, WEB-010, WEB-011, WEB-013, OPS-004, OPS-007, OPS-I01, OPS-I03, OPS-T05, OPS-T07
Acceptance criteria:
1. The `app` module creates an HTTP API with a JWT authorizer that accepts only access tokens issued by the user pool for the dashboard's app client. A request with a missing, expired, or wrongly issued token gets 401 from API Gateway.
2. `GET /health` is the only route without the authorizer.
3. The API Lambda runs from the `api` image, has no function URL, and has a timeout below 30 seconds.
4. CORS allows only the origins in a variable for each environment, and allows the `Idempotency-Key` header. A browser request from another origin is refused.
5. The API Lambda's role grants only what this area's Requirements Intake row lists, on named resources.
6. Alarms on API 5xx responses, API Lambda errors, and API latency send to the team's alarm topic (OPS-I04).
Status: Blocked (OPS-004 and OPS-007 wait for Luke Campbell; OPS-I01 and OPS-I03 wait for Ryan Fusco and Luke Campbell)

### WEB-T04: Provision CloudFront for the dashboard and resumes
Depends on: WEB-007, WEB-010, DEL-I01, OPS-I02, OPS-I03, OPS-T04, OPS-T07
Acceptance criteria:
1. One distribution serves the dashboard's static files from the static-site bucket, and `resume/*` from the resumes bucket. Neither bucket can be read directly from the internet.
2. A request for `resume/*` without a valid signed URL is refused. A request with a valid, unexpired signed URL returns the PDF.
3. The distribution has no rule that returns `index.html` for a missing or refused file.
4. The public half of the signing key is registered with the distribution. The private half is only in Secrets Manager.
5. The module outputs the distribution's domain and ID, for the callback URLs, the CORS origins, and the publish step.
Status: Blocked (OPS-I02 and OPS-I03 wait for Ryan Fusco and Luke Campbell)

## API endpoints

### WEB-T05: Profile and settings (`GET /me`, `PUT /me`)
Depends on: WEB-T01, WEB-003, WEB-008, DB-I05, DB-I01, DB-T01, DB-T02
Acceptance criteria:
1. A student's first `GET /me` creates a Profile with their verified email address from Cognito, and Settings with the defaults in DB-I05. It returns both.
2. Two first requests at the same time create each item once, and both return the same values.
3. A later `GET /me` returns the stored values and writes nothing.
4. `PUT /me` with valid settings replaces the whole Settings item and returns the new values.
5. `PUT /me` is rejected with 400, and changes nothing, when the delivery mode isn't `instant`, `hourly`, or `daily`; when a list has more than 20 terms; when a term is empty or longer than 60 characters after trimming; or when a list repeats a term ignoring case.
6. One student's request never reads or writes another student's Profile or Settings.
Status: Blocked (DB-I01 waits for Luke Campbell)

### WEB-T06: Upload link and experience bank (`POST /uploads`, `GET /experience`, `DELETE /experience`)
Depends on: WEB-T01, EB-I01, EB-I03 (with the EB-019 extension), EB-T01, EB-T02, OPS-T04
Acceptance criteria:
1. `POST /uploads` calls the bank package's `create_upload` with the caller's `sub` and returns its `docId`, `url`, `fields`, and `expiresAt` unchanged. The API builds no upload policy of its own.
2. A file sent with the returned `url` and `fields` is stored at `uploads/{sub}/{docId}.tex`, where `sub` is the caller's. A different key, a different content type, a file over 1 MB, or a link older than 15 minutes is refused.
3. `GET /experience` returns the latest upload's status from the bank package (`none`, `awaiting_file`, `pending`, `parsed`, `parsed_with_unclassified`, `failed`, or `expired`), the reason when it failed, and the current bank when there is one. A failed or expired upload leaves the earlier bank in the response. Every item has `source` and `text` exactly as the package returned them.
4. `DELETE /experience` deletes the bank through the package and returns 204. Repeating it returns 204.
5. The API reads and deletes the bank only through the bank package. No code in the API queries experience items directly.
Status: Blocked (the EB-019 extension to EB-I03 waits for Luke Campbell)

### WEB-T07: Sources, subscriptions, and refresh
Depends on: WEB-T01, JDM-I01, JDM-009, DB-I04, DB-T01, DB-T02
Acceptance criteria:
1. `GET /sources` returns every catalog source and only the caller's private sources, each marked with whether the caller is subscribed.
2. `POST /sources` accepts a board token or board URL that passes the shared validation in JDM-I01, and returns 201. Anything else is rejected with 400 and nothing is written. Adding the same board twice returns 409.
3. `DELETE /sources/{sourceId}` deletes the caller's own private source and their subscription to it, and returns 204. For a catalog source, or another student's private source, it returns 404 and changes nothing.
4. `POST /subscriptions/{sourceId}` returns 201 for a catalog source or the caller's own private source, 409 when already subscribed, and 404 for another student's private source. `DELETE` returns 204 whether or not the caller was subscribed.
5. `POST /sources/{sourceId}/refresh` returns 202 and records the request before sending anything to a queue. Within the cooldown it returns 429 with `retryAfterSeconds`. Two requests at once start one refresh.
6. Source IDs have the forms in JDM-009. A private source ID that encodes another student's `sub` gives no access to that student's source: ownership is read from the stored record.
Status: Blocked (DB-I04 waits for Luke Campbell)

### WEB-T08: Matches feed and one match (`GET /matches`, `GET /matches/{postingKey}`)
Depends on: WEB-T01, WEB-T04, WEB-006, WEB-007, DB-I03, DB-I04, DEL-I01, DB-T01, DB-T02
Acceptance criteria:
1. `GET /matches` returns the caller's matches newest first, 25 to a page by default and at most 100. A `limit` outside 1 to 100 is rejected with 400.
2. Following `nextCursor` from page to page returns every match exactly once. The last page has no `nextCursor`. A cursor that is malformed, or that was issued to another student, is rejected with 400.
3. `GET /matches/{postingKey}` returns the match's own copy of the posting, its delivery state, and `appliedAt`. For a match that doesn't exist or belongs to another student it returns 404.
4. When `renderStatus` is `rendered`, the response has a signed link to the current resume that works and that expires 15 minutes after it was issued. For any other status, `resume` is null and `renderReason` is returned.
5. `earlierResumes` lists every other PDF under `resume/{sub}/{postingKey}/`, each with its own signed link.
6. Every signed path starts with `resume/{sub}/`, using the caller's `sub`. No request field can change the path that is signed.
7. The signing key is read from Secrets Manager at most once per Lambda instance.
Status: Blocked (DB-I03 and DB-I04 wait for Luke Campbell)

### WEB-T09: Mark a match as applied
Depends on: WEB-T01, DB-I04, DB-T01, DB-T02
Acceptance criteria:
1. `PUT /matches/{postingKey}/applied` sets `appliedAt` on the caller's match and returns it.
2. Repeating the `PUT` returns the first `appliedAt` and doesn't change it.
3. `DELETE /matches/{postingKey}/applied` clears it and returns 204, whether or not it was set.
4. Both return 404 for a match that doesn't exist or belongs to another student, and neither creates a match.
5. Neither changes any other attribute of the match, including its delivery state.
Status: Blocked (DB-I04 waits for Luke Campbell)

### WEB-T10: Regeneration requests
Depends on: WEB-T06, WEB-T08, WEB-013, JDM-I05, DB-I04 (with its regeneration amendment), WEB-Q08
Acceptance criteria:
1. `POST /uploads` with `regenerate: true` writes the upload-regeneration work record for the new `docId` before it answers. With `regenerate` false or absent, no work record is written.
2. If the work record can't be written, `POST /uploads` answers with an error and returns no upload link.
3. `POST /matches/{postingKey}/regenerate` without an `Idempotency-Key` header, or with one that isn't a UUID, returns 400 and writes nothing.
4. With a valid key, for the caller's own match, it writes the work record and claims the match's active request in one write, and returns 202 with the key as `requestId`. For a match that doesn't exist or belongs to another student it returns 404.
5. The same key again for the same match returns the same 202 and `requestId`, and writes nothing new.
6. A different key while a request is still active for that match returns 409 and starts nothing. A key already used for another match returns 409.
7. When sending the work to the queue fails after the record is written, the request still returns 202, and the work record remains for the dispatcher to pick up.
Status: Blocked (DB-I04's regeneration amendment isn't written; WEB-Q08 is open)

## Dashboard

### WEB-T11: Dashboard shell and sign-in
Depends on: WEB-002, WEB-009, WEB-010
Acceptance criteria:
1. A student who isn't signed in is sent to the hosted UI. After signing in, they return to the dashboard signed in.
2. Sign-in uses the authorization code flow with PKCE. No token appears in a URL.
3. Every API request carries the access token. When the API answers 401, the dashboard sends the student to sign in again.
4. The dashboard calls `GET /me` each time it loads, before showing any screen.
5. The five screens in WEB-009 are reachable by hash-based routes, and reloading the page on any of them shows the same screen.
6. Signing out clears the tokens and returns the student to the hosted UI.
7. The API's address and the Cognito settings come from build settings. None is written in the source.
Status: Planned

### WEB-T12: Resume screen
Depends on: WEB-T11, WEB-T06, WEB-T10
Acceptance criteria:
1. The student can choose one `.tex` file and upload it. A file with another extension is refused before any request is made.
2. The upload form offers "keep my existing resumes" (selected by default) and "regenerate resumes for my existing matches", and sends the choice with `POST /uploads`.
3. The file is sent to the returned `url` with the returned `fields`. The screen then checks `GET /experience` until the status is no longer `awaiting_file` or `pending`, and shows the parsed bank or the failure reason. A file S3 refuses, such as one over 1 MB, is reported to the student at once.
4. The bank is shown grouped by section, using each item's `text` exactly as returned.
5. A bank parsed with unclassified sections shows those sections, marked as not yet classified.
6. Deleting the bank asks for confirmation first, and afterwards the screen shows that there is no bank.
Status: Blocked (WEB-T06 and WEB-T10 are Blocked)

### WEB-T13: Sources screen
Depends on: WEB-T11, WEB-T07
Acceptance criteria:
1. The screen lists catalog sources and the student's private sources separately, with each one's subscription state, last successful check, and last error.
2. The student can subscribe and unsubscribe, and the list shows the new state without a page reload.
3. The student can add a private board by token or URL. A 400 or 409 from the API is shown as a message next to the form.
4. The student can delete a private board, after confirming.
5. Refresh shows that a check was requested. On 429 it shows how long to wait.
Status: Blocked (WEB-T07 is Blocked)

### WEB-T14: Matches and Match screens
Depends on: WEB-T11, WEB-T08, WEB-T09, WEB-T10
Acceptance criteria:
1. The Matches screen lists matches newest first and loads the next page on request until there are none left.
2. The Match screen shows the posting details, a link to apply, and the delivery state.
3. When the resume is rendered, the screen offers a link that opens the PDF. When it isn't, the screen shows the status and reason and no link.
4. Earlier resumes are listed, each with its own link.
5. A link that has expired is replaced by asking the API for a fresh one. The student isn't shown an error page.
6. "Mark as applied" and "unmark" update the screen from the API's response.
7. "Regenerate" sends a new `Idempotency-Key` for each press and reuses it when retrying that press. It shows that a request is running, and on 409 it tells the student one is already running.
Status: Blocked (WEB-T08, WEB-T09, and WEB-T10 are Blocked)

### WEB-T15: Settings screen
Depends on: WEB-T11, WEB-T05
Acceptance criteria:
1. The screen shows the student's delivery mode and the three filter lists from `GET /me`.
2. The student can change the delivery mode, and add and remove terms in each list.
3. The form stops a list at 20 terms and a term at 60 characters, before any request is made.
4. Saving sends the whole settings object with `PUT /me`. A 400 from the API is shown as a message, and the form keeps what the student entered.
5. The screen tells the student that an empty "title includes" list matches every posting.
Status: Blocked (WEB-T05 is Blocked)

### WEB-T16: Build and publish the dashboard in CI
Depends on: WEB-T11, WEB-T04, OPS-012, OPS-005, OPS-T06
Acceptance criteria:
1. A pull request that changes the dashboard runs its lint, unit tests, and build, and can't be merged if any fails.
2. A push to `main` builds the dashboard with the dev environment's settings, taken from Terraform outputs, uploads the build to the static-site bucket, and invalidates CloudFront.
3. After a deploy, the dashboard served from CloudFront is the newly built version.
4. Prod is published only when the deploy is started by hand.
5. No AWS key is stored in GitHub. The workflow uses OIDC.
Status: Blocked (OPS-012 and OPS-005 wait for Luke Campbell; WEB-T04 is Blocked)
