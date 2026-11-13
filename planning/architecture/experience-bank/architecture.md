# Experience Bank Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`), pp.1–2, 4–6, 8, and the "Experience Bank" diagram lane. Everything below is `Proposed` until the owner accepts it.

## Scope

**Owns:**
- The layout and upload rules for the `uploads/{sub}/{docId}` S3 prefix: allowed types (PDF, including scanned; DOCX; TXT) and size limits.
- The Bank parser Lambda. An S3 upload event triggers it. It extracts text, uses Bedrock to split the text into roles, bullets, and skills, and writes the results.
- The verbatim guarantee: every stored bullet is an exact span of the original document's text (EB-Q02).
- What uploads, roles, bullets, and skills are, and the rules they must follow. How they're stored is defined in `database` (DB-I02).
- Upload status (pending, parsed, failed), so the dashboard can show progress and errors.
- What deleting an upload removes: the S3 object and every item derived from it.

**Does not own:**
- The upload, list, and delete endpoints (`POST /uploads`, `GET /experience`, `DELETE /uploads/{docId}`), presigned URLs, and sign-in: `web-app-and-api`. They follow EB-I01 and DB-I02.
- Choosing and ordering bullets for a job: `job-discovery-and-matching` (the Matcher). Ryan Fusco contributes.
- Rendering bullets into a resume: `delivery`.
- How items are stored in the table: `database`.
- The bucket, table, Lambda image, IAM role, and alarms themselves: `build-and-operation` provisions them from the Requirements Intake.

## Architecture (as given)

_None yet._

## Architecture Review

_None yet._

## Decisions

_None yet._

## Interfaces

### Provides

#### EB-I01: Upload object contract
Status: Proposed
Consumers: web-app-and-api
Contract:
- **Key:** `uploads/{sub}/{docId}`, where `sub` comes from the Cognito token and never from the request (RFC p.7).
- **Upload:** one object per presigned PUT, limited to the allowed content types and a maximum size (to be set).
- The parser takes the user from the key, so the key layout is part of the security model.
Accepted by: —

### Consumes
- **Experience items:** DB-I02 in `database`. This area writes them, and proposes changes to them as a contributor there.
- **The user's Cognito `sub`:** through the upload key (EB-I01), which `web-app-and-api` issues.
- **Bedrock, and the hourly cost counter:** `COST#{hour}`, owned by `job-discovery-and-matching` (see EB-Q03).
- **The uploads bucket with event notifications, the shared table, the parser Lambda image, and its IAM role:** from `build-and-operation` (OPS-I01, OPS-I02, OPS-I03).

## Open Questions

- **EB-Q01: How are scanned PDFs read?** Text extraction finds nothing in a scanned PDF, and demo step 1 uses one. The RFC doesn't name a way to read them. The options are:
  - send the document straight to Bedrock as input to a multimodal model
  - add Textract, which would be a new AWS service
  - run OCR such as Tesseract in the parser image

  How the text is read also decides what counts as "the original text" for the verbatim check.
  - Owner: Ryan Fusco
  - Blocks: the parser design, the image size, Goal 3
- **EB-Q02: How is "verbatim" enforced?** An LLM asked to split text can quietly change words. Recommendation: Bedrock returns character offsets (or exact strings), and the parser slices or checks each bullet against the extracted text. A bullet that doesn't match exactly is rejected.
  - Owner: Ryan Fusco
  - Blocks: DB-I02, the Goal 3 demo
- **EB-Q03: Do parser calls to Bedrock count toward the hourly cost cap?** The Matcher enforces that cap, and the counter is owned by `job-discovery-and-matching`.
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: the parser's Bedrock usage
- **EB-Q04: What happens to existing matches and resumes when an upload is deleted?** They may refer to bullets that no longer exist.
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: the delete behavior
- **EB-Q05: How are duplicates across uploads handled?** If several uploads contain the same bullet, are duplicates merged, kept, or flagged? This affects what the Matcher can choose from.
  - Owner: Ryan Fusco
  - Blocks: DB-I02
- **EB-Q06: Must skills also be verbatim,** or may they be normalized (for example "JS" to "JavaScript")? The non-goal "no AI-written resume text" suggests verbatim.
  - Owner: Ryan Fusco
  - Blocks: DB-I02
- **EB-Q07: How does the API Lambda work with experience-bank items?** It could read and delete `DOC#` items directly, following DB-I02, or call shared code owned by this area.
  - Owner: Ryan Fusco, Graham Cohick
  - Blocks: DB-I02, the code layout
