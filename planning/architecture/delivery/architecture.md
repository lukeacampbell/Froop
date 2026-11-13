# Delivery Architecture

Source: RFC 0001 (`architecture/inputs/Froop_inital_architecture.pdf`), pp.1–5, 8, and the "Delivery" diagram lane. Everything below is `Proposed` until the owner accepts it.

## Scope

**Owns:**
- **The Renderer Lambda** (container image with LaTeX):
  - Takes messages from SQS render.
  - Fills the resume template with the selected bullets and skills, escaping LaTeX special characters without changing the text.
  - Makes sure the result fits on one page.
  - Writes `resume/{sub}/{postingKey}.pdf`.
  - For instant delivery, hands off to the Notifier.
- **The resume template.**
- **The `resume/` prefix layout and its 30-day lifecycle rule** (the bucket itself comes from `build-and-operation`).
- **The Notifier Lambda:**
  - Sends instant emails, and hourly or daily digests when the EventBridge digest schedule fires.
  - Finds matches waiting for a digest through GSI2.
  - Signs CloudFront links for emails.
- **Email content and SES templates.**
- **The meaning of delivery state on each match:** rendered, emailed, or waiting for a digest. A retried message must never render twice or send twice (SQS delivers at least once). How that state is stored is defined in `database` (DB-I03).

**Does not own:**
- Creating matches, or selecting and ordering bullets: `job-discovery-and-matching` (the Matcher). Ryan Fusco contributes.
- Bullet text: `experience-bank`.
- How items are stored in the table: `database`.
- The dashboard, `GET /matches/{postingKey}` signed links, the CloudFront distribution, and the user's delivery settings (`PUT /me`): `web-app-and-api`.
- The buckets, the SES identity and production access, the signing-key secret, ECR images, and alarms: `build-and-operation`.

## Architecture (as given)

_None yet._

## Architecture Review

_None yet._

## Decisions

_None yet._

## Interfaces

### Provides

#### DEL-I01: Resume PDF objects
Status: Proposed
Consumers: web-app-and-api (signs dashboard links)
Contract:
- **Key and access:** `resume/{sub}/{postingKey}.pdf`, private, and served only through CloudFront signed URLs. Each object is deleted 30 days after it's created.
- **Missing PDF:** an object may not exist yet (still rendering) or may have expired (DEL-Q05).
Accepted by: —

### Consumes
- **The SQS render message:** from `job-discovery-and-matching`, the producer, which defines it. It needs `sub`, `postingKey`, and the ordered bullet and skill IDs. No interface ID yet (DEL-Q04).
- **Experience items** (bullet and skill text): DB-I02 in `database`, written by `experience-bank`.
- **Match delivery state:** DB-I03 in `database`. Delivery writes it.
- **Posting details for the email** (title, company, location, apply link) and **user settings** (delivery mode, email address): item types in `database`, not defined yet (DB-Q01).
- **Infrastructure from `build-and-operation`:** the CloudFront signing key from Secrets Manager, SES, the LaTeX worker image, buckets, the EventBridge digest schedule, and alarms (OPS-I01–I04).

## Open Questions

- **DEL-Q01: How does the resume reach the student?** This is a risk the RFC raises (p.8). The options are an email attachment, a signed link, or the dashboard. Demo step 4 opens an "attached PDF". Recommendation: attach the PDF in instant emails, and put signed links in digests, so emails stay small and are less likely to be marked as spam.
  - Owner: Ryan Fusco
  - Blocks: the Notifier design, the use of the signing key
- **DEL-Q02: How is "one page" guaranteed?** Recommendation:
  1. The Matcher sends a ranked list that's longer than one page.
  2. The Renderer renders the PDF and checks the page count.
  3. If it's over one page, the Renderer drops the lowest-ranked bullets and renders again, until it fits.
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: DEL-Q04, the Renderer
- **DEL-Q03: Where does the resume header come from?** That's the name, contact details, and education. The RFC doesn't say. The options are profile fields (`web-app-and-api`) or roles from the experience bank.
  - Owner: Ryan Fusco, Graham Cohick
  - Blocks: the template
- **DEL-Q04: What is the shape of the SQS render message?** Ryan Fusco should propose it as a contributor to the Matcher.
  - Owner: Luke Campbell
  - Blocks: the Renderer
- **DEL-Q05: Lifetimes don't line up.** Resume PDFs are deleted after 30 days, postings expire 30 days after last seen, and matches never expire. Should an expired PDF be re-rendered on demand, or should the match show it as expired?
  - Owner: Ryan Fusco, Graham Cohick
  - Blocks: DEL-I01
- **DEL-Q06: Who sets the GSI2 "waiting for a digest" key?** The Matcher could set it when it creates the match, or the Renderer once the PDF exists. Recommendation: the Renderer, so a digest never includes a resume that hasn't been rendered.
  - Owner: Ryan Fusco, Luke Campbell
  - Blocks: DB-I03
