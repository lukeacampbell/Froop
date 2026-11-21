# Delivery Implementation Plan

Owner: Ryan Fusco.
Date: 2026-11-19. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: decisions DEL-001 to DEL-020 (Accepted, except DEL-012, which is superseded) and interface DEL-I01 (Accepted). It consumes JDM-I03 (the render message) and EB-I03 (the bank package). No application or test code exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision or interface still needs agreement before the task can be built. It doesn't mean the task was tried and failed.

**Order.**
1. The template, safe compile, and page fitting (T01 to T03) are pure logic, so they can start first.
2. The Renderer (T04) and the Notifier (T05, T06) follow once the shared database contracts are agreed.
3. The infrastructure (T07) comes last.

**Tests.** Each task includes unit tests for its acceptance criteria in Phase 2. Sample banks from `experience-bank` and sample render messages are shared by T01 to T04. Tests that cross areas, such as a match that ends in an email, belong to the Phase 3 test plans.

| Waiting on | Tasks it blocks |
|---|---|
| EB-T01 / DB-Q10 and OPS-I01 to I04 provider signoff | See dependency update below |
| EB-T01 / DB-Q10 and OPS-I01 to I04 provider signoff | See dependency update below |
| EB-T01 / DB-Q10 and OPS-I01 to I04 provider signoff | See dependency update below |
| `experience-bank` building its package (EB-T01) | T04 (T01 to T03 use fixtures until then) |

## Building the resume

**Dependency update (2026-11-21):** Luke answered the shared approvals. DB-I01/I03/I04 base, DB-001, OPS decisions and OPS-I05 are Accepted. DB-I02 awaits DB-Q10; OPS-I01 to I04 await Ryan. Existing task sequence/criteria are unchanged; the owner should finalize readiness against these blockers.
### DEL-T01: The Froop resume template
Depends on: DEL-001, DEL-006, EB-015
Acceptance criteria:
1. One template defines the layout and every macro in `supported_templates()`, which currently means Jake's Resume. A test fails if any listed macro is undefined.
2. The header block comes first, then sections in this order: Education, Experience, Projects, Technical Skills, then Awards, Certifications, and Publications.
3. The template uses only packages included in the Renderer image (T07).
4. A sample bank renders to a PDF with no LaTeX errors.
Status: Planned

### DEL-T02: Sanitize source and compile safely
Depends on: DEL-002, EB-009, EB-012
Acceptance criteria:
1. Every item's `source` is checked against the macros the template defines, plus a short safe list (`\textbf`, `\textit`, `\emph`, `\href`, `\&`, `\%`, and the like). Any other macro is reduced to its text arguments. A test checks that the words stay the same.
2. Compilation runs `pdflatex` with `-no-shell-escape`, `openin_any` and `openout_any` set to paranoid, inside a temporary directory, under a time limit (default: 20 seconds).
3. A test file that tries `\write18`, `\input`, or reading `/etc/passwd` produces no side effect, and the render fails cleanly.
4. The page count is read from the PDF that was produced.
Status: Planned

### DEL-T03: Fit the resume to one page
Depends on: DEL-T01, DEL-T02, DEL-005, DEL-013, DEL-015, DEL-016, JDM-I03
Acceptance criteria:
1. **Required items** are always included: the header, all education entries, and the top-ranked role with its top bullet.
2. **Candidate pool:** every other bullet, skill-category line, and possible-section entry (Awards, Certifications, Publications) is a candidate. Possible sections are considered only once Projects and Technical Skills have nothing left that fits. Leadership and unlisted sections are never included.
3. **Effective score:** score × `decay^n`, where `n` is the number of bullets already taken from that entry. `decay` is a setting (default: 0.85), and there's no cap on bullets per entry.
4. **Fill:** the line estimate (header, section headings, role headings, and wrapped bullet lines) adds the highest-scoring candidate that fits until the page is full.
5. **Render and prune:** after rendering, if the PDF runs over one page, the lowest-scoring item that isn't required is removed and it renders again, up to 5 attempts. A role or project left with no bullets is removed too.
6. **Calibration:** the constants are tuned against the sample banks, and a test checks that every sample produces exactly one page.
7. A metric records the render attempts for each resume.
Status: Planned

## Renderer and Notifier

### DEL-T04: The Renderer Lambda
Depends on: DEL-T03, DEL-003, DEL-009, DEL-010, DEL-017, DEL-I01, JDM-I03, EB-I03, EB-T01, DB-I03
Acceptance criteria:
1. It accepts JDM-I03 messages with `schemaVersion` 1. An unknown version fails visibly and goes to the DLQ.
2. A repeated `(sub, postingKey, docId, requestId)` is acknowledged and dropped. A `generation` lower than the match's current one is dropped.
3. If `docId` isn't the current upload, or `get_items` reports missing IDs, it sets `renderStatus=skipped` with reason `bank_replaced`, sends no email, and doesn't retry.
4. The PDF is written to `resume/{sub}/{postingKey}/g{generation}.pdf` and never overwrites another file.
5. It updates `renderStatus`, `pdfKey`, `generation`, and `requestId` with writes conditional on `generation`.
6. For an instant-mode `new_match`, it hands off to the Notifier. A `regenerate` request never triggers an email.
7. Logs carry `sub`, `postingKey`, and `generation`, and never contain resume text.
Status: Blocked (DB-I03 Accepted (2026-11-21); EB-T01)

### DEL-T05: Instant emails, sent at most once
Depends on: DEL-T04, DEL-004, DEL-018, DB-I03, DB-I05, OPS-008
Acceptance criteria:
1. One email per instant match, with the PDF attached (raw MIME) and the posting details in the body. The address comes from the Profile item (DB-I05).
2. `notifyStatus` moves `pending` → `sending` with a conditional write before sending. If that write fails, nothing is sent.
3. After SES accepts the message, `notifyStatus` moves to `sent`. A match stuck in `sending` for more than 15 minutes is marked `failed` and never resent.
4. A recipient that isn't verified in the SES sandbox ends as `failed`, with the SES reason recorded.
Status: Planned (DB-I03, OPS-008 and DB-I05 Accepted; execution still waits for Phase 2 and predecessor tasks)

### DEL-T06: Digest schedules and digest emails
Depends on: DEL-T05, DEL-011, DEL-019, DEL-020, DB-I01, DB-I03
Acceptance criteria:
1. Two EventBridge schedules trigger the Notifier: hourly at minute 0 (UTC), and daily at 13:00 UTC.
2. Each run reads the GSI2 bucket for this send, `DIGEST#hourly#{UTC-hour}` or `DIGEST#daily#{UTC-date}`, plus the earlier buckets it may have missed (the last 24 hourly or the last 7 daily).
3. **Rendered matches** go into one email per student, with CloudFront signed links (7-day expiry). Each one is moved through `sending` → `sent`, as in T05, and its GSI2 fields are removed.
4. **Pending matches** are moved to the next send's bucket.
5. **Skipped and failed matches** lose the mark, and no email is sent.
6. A run repeated for the same send sends nothing twice.
Status: Planned (DB-I01/I03 Accepted; execution still waits for Phase 2 and predecessor tasks)

## Infrastructure

### DEL-T07: Provision the Renderer, the Notifier, and the schedules
Depends on: DEL-T04, DEL-T06, OPS-004, OPS-I01, OPS-I03, OPS-I04, OPS-T04, OPS-T05, OPS-T07
Acceptance criteria:
1. The `renderer` image holds a minimal TeX distribution with only the packages the template uses (DEL-T01). The Notifier has its own small image.
2. Both functions, the render queue's consumer settings, and the two schedules are in the `resumes` Terraform module (SYS-Q03).
3. **Renderer role:** read the bank through the package, write `resume/`, and update match delivery fields.
4. **Notifier role:** query GSI2, update delivery fields, read the signing key, and send through SES.
5. Alarms cover render-queue DLQ depth, Renderer errors, and SES bounces.
Status: Blocked (OPS-I01 to I04 still await Ryan; Luke has accepted)
