# Experience Bank Implementation Plan

Owner: Ryan Fusco.
Date: 2026-11-19, revised 2026-11-23. Phase: Architecture Planning. State: Draft.

This plan builds the design in `architecture.md`: decisions EB-001 to EB-020 (Accepted, except EB-004 and EB-011, which are superseded, and EB-007, which is deferred) and interfaces EB-I01 and EB-I03. No application or test code exists yet. Phase 2 begins only after a human confirms it under `planning/AGENTS.md`.

**What the statuses mean here.** `Planned` means everything the task depends on is Accepted. `Blocked` means a named decision or interface still needs agreement before the task can be built. It doesn't mean the task was tried and failed.

**Order.**
1. The parsing core (T03, T04) depends only on this area's own Accepted decisions, so it can start first.
2. The package and the upload contract (T01, T02) follow once the shared database contract is agreed.
3. Bank replacement, leftovers, and wiring (T05 to T07) come last.

**Tests.** Each task includes unit tests for its acceptance criteria in Phase 2. A set of sample Jake's Resume files (at least one clean, one with leftover sections, one malicious, and one oversized) is shared by T03 to T06. Tests that cross areas, such as an upload that ends in a rendered resume, belong to the Phase 3 test plans.

| Waiting on | Tasks it blocks |
|---|---|
| JDM-T05 (the reservation package, built by `job-discovery-and-matching`) | T06 |

## Parsing core

### EB-T03: Read the upload safely
Depends on: EB-001, EB-009
Acceptance criteria:
1. The reader treats the file as UTF-8 text and never invokes `latex`, `pdflatex`, or any other TeX program.
2. A file containing `\write18`, `\input`, `\include`, `\openin`, `\openout`, `\read`, `\catcode`, or a `\csname` construction is rejected as `failed`, with a reason naming the command.
3. A file over 1 MB, or one that isn't valid UTF-8, is rejected as `failed` with a reason.
4. Parsing stops after a configured time limit (default: 10 seconds) and reports `failed` with the reason `timeout`.
Status: Planned

### EB-T04: Parse header, sections, entries, bullets, and skills without an LLM
Depends on: EB-T03, EB-003, EB-005, EB-006, EB-012, EB-014, EB-015, EB-016
Acceptance criteria:
1. **Header:** the content before the first section is extracted as the header block, with `source` and `text`. A file without one is still parsed, and its status notes the missing header.
2. **Sections:** `\section` and `\section*` are found, and each heading is mapped to a category through the heading dictionary, e.g. "Work Experience", "Professional Experience", and "Employment" all map to experience. Unknown headings are left for T06.
3. **Entries:** Jake's Resume macros produce entries:
   - `\resumeSubheading` under experience produces roles.
   - `\resumeSubheading` under education produces education entries.
   - `\resumeProjectHeading` produces projects.
   - `\resumeItem` produces bullets, in their original order.
4. **Skills:** each category line in the skills section (e.g. `\textbf{Languages}{: Java, Python, C/C++}`) becomes a skill category, and each value becomes a skill. Splitting respects braces and escapes, so `C/C++` and `Node.js` stay whole.
5. **Custom macros:** a simple `\newcommand` in the file's own preamble, whose body is plain text with arguments, is expanded without running anything.
6. **Verbatim:** every item stores `source`, an exact span of the file, and `text`, derived from `source` deterministically. A test checks that every `source` appears character for character in the upload.
7. **Duplicates:** repeated bullets are kept as separate items with their own IDs.
8. **Supported templates:** the macro list comes from one shared definition, the one `supported_templates()` returns (EB-015).
Status: Planned

## Package and upload contract

**Dependency update (2026-11-23):** DB-I01, DB-I03, DB-I04 (base), DB-I05, DB-001, every OPS decision, and OPS-I01 to I05 are Accepted, as are EB-I01 and EB-I03 with its extension. DB-I02 is Accepted too (DB-Q10, 2026-11-23), so nothing in this area waits on an agreement.
### EB-T01: Build the experience-bank package
Depends on: EB-010, EB-I03, EB-008, DB-I02, OPS-007
Acceptance criteria:
1. The package lives in `libs/experience_bank` (OPS-007) and is the only code that reads or writes `DOC#` items (EB-010).
2. `get_bank(sub)` resolves the current-upload pointer, then returns the header and the items grouped by section category, with skills grouped by skill category. Unclassified items are marked so the Matcher can exclude them.
3. `get_items(sub, ids)` returns the items that exist and lists missing IDs without raising an error.
4. `delete_bank(sub)` deletes every `DOC#` item and the uploaded files, and is safe to repeat.
5. `upload_status(sub, docId=None)` returns one of `none`, `awaiting_file`, `pending`, `parsed`, `parsed_with_unclassified`, `failed`, or `expired`, with a reason when failed. It reports `expired` when `uploadExpiresAt` has passed with no file. Given a `docId`, it also says whether that upload is the current bank.
6. `supported_templates()` returns the macro list used by T04 and by the delivery template (DEL-001).
7. Every function is scoped to the given `sub`, and no function returns another student's items.
Status: Planned

### EB-T02: Issue upload links (`create_upload`)
Depends on: EB-T01, EB-018, EB-019, EB-I01, OPS-I02
Acceptance criteria:
1. `create_upload(sub, regenerate=False)` generates a new `docId` and writes an upload record with status `awaiting_file`, `uploadExpiresAt` (not the TTL attribute `expiresAt`, DB-Q10), and the `regenerate` choice. The record has no TTL.
2. It returns `{docId, url, fields, expiresAt}` for a presigned POST whose policy fixes the key `uploads/{sub}/{docId}.tex`, the content type `application/x-tex`, a size of 1 byte to 1 MB, and a 15-minute expiry.
3. A test confirms S3 rejects a file over 1 MB, a file with another content type, and a file sent to a different key.
4. Creating an upload doesn't change the current bank.
Status: Planned (after EB-T01)

## Parser pipeline

### EB-T05: Replace the bank safely
Depends on: EB-T01, EB-T04, EB-002, EB-008, EB-013
Acceptance criteria:
1. When a file arrives, the record moves from `awaiting_file` to `pending`.
2. A successful parse writes every item under the new `docId`, then switches the current-upload pointer in one conditional write, then deletes the old upload's items. A TTL on old items is the safety net.
3. A reader during the switch sees either the whole old bank or the whole new bank, never a mix. A test interleaves reads with a replacement to check this.
4. A failed parse leaves the current bank and its pointer untouched, and records `failed` with a reason.
5. If two uploads finish out of order, only the newer upload becomes current.
6. A file that arrives after its link expired is ignored and stays `expired`.
Status: Planned (after EB-T01 and EB-T04)

### EB-T06: Classify leftover sections with Bedrock, under the budget
Depends on: EB-T04, EB-T05, EB-003, EB-013, EB-017, EB-020, JDM-I04, JDM-T05, OPS-006
Acceptance criteria:
1. Only sections T04 couldn't place are sent to Bedrock, using the configured parser model (`BEDROCK_MODEL_PARSER_LEFTOVERS`).
2. Before every call, the parser reserves a budget through JDM-I04 with a new attempt ID, using a worst-case estimate (input tokens plus the configured maximum output tokens).
3. Bedrock must return a category and exact spans. A span that isn't an exact substring of the upload is rejected and logged, never stored.
4. If the reservation is deferred, the sections are stored as `unclassified` and the status becomes `parsed_with_unclassified`. Nothing retries them; a new upload tries again (EB-020). If it's `blocked_configuration`, the status says so.
5. Actual tokens and estimated cost are logged as metrics, without resume text.
Status: Planned (after JDM-T05, which builds the reservation package)

### EB-T07: Wire the parser Lambda
Depends on: EB-T05, EB-T06, OPS-004, OPS-I01, OPS-I03, OPS-I04, OPS-T04, OPS-T05
Acceptance criteria:
1. The parser image is small: Python and the package, with no TeX install and no OCR (OPS-004).
2. An S3 object-created event on `uploads/` triggers the parser. The event's key gives the `sub` and `docId`, and a key that doesn't match the layout is rejected.
3. The parser's IAM role can only read `uploads/`, write `DOC#` items, call the configured Bedrock model, and use the cost reservation records.
4. Logs are JSON with `sub` and `docId`, and never contain resume text. Metrics count uploads parsed, failed, and left unclassified.
5. A retried S3 event for an upload already processed changes nothing.
Status: Planned (after EB-T05 and EB-T06)
