# Instructor Meeting

| Field | Value |
|---|---|
| Date and time | 2026-11-19, 10:00 AM (America/New_York) |
| With | Course instructor |
| Attendees | Ryan Fusco, Graham Cohick, Luke Campbell, and the instructor |
| Note taker | _Pending._ |

### MQ-2026-11-19-01: Pooling AWS credits across separate accounts
- **Question:** Each of us has our own AWS account, each with its own credits. Froop is one system: a single shared database, queues, Lambdas, and Bedrock calls, deployed by CI from one repository. Can we pool our credits for the project, for example through a shared course account, consolidated billing, or moving credits to one account? If not, what do you recommend so the project doesn't get expensive for whoever's account hosts it?
- **Asked by:** Ryan Fusco
- **Answer:** _Pending._
- **Answered by:** _Pending._
- **Additional notes:** _Pending._ Context for the discussion:
  - Our plan caps spending at a $50 monthly AWS budget alert, a $1.50 daily Bedrock cap, and $1 per student per hour (OPS-002, OPS-009, OPS-010).
  - Those caps assume one account per environment (dev and prod).
  - The answer may change OPS-T02 (bootstrap) and which account hosts dev and prod.
- **Note taker:** _Pending._
- **Recorded:** 2026-11-19 (agent working for Ryan Fusco)

### MQ-2026-11-19-02: The best way to add more job postings
- **Question:** For adding more job postings, what would the best way be? Claude is suggesting the Playwright-on-Lambda route, and I'm skeptical at best.
- **Asked by:** Luke Campbell
- **Answer:** _Pending._
- **Answered by:** _Pending._
- **Additional notes:** _Pending._ Context for the discussion:
  - The initial RFC used a Playwright scraper image (RFC p.3). JDM-001 (Accepted) replaced it with Greenhouse's public Job Board JSON API only, with no browser scraping.
  - Today, more postings means more Greenhouse boards: catalog boards plus boards students add themselves (JDM-001, JDM-I01).
  - The answer may lead to a `Proposed` decision in `job-discovery-and-matching/architecture.md`, citing this entry.
- **Note taker:** _Pending._
- **Recorded:** 2026-11-24 (agent working for Ryan Fusco, question from Luke Campbell)

### MQ-2026-11-19-03: Where the bottleneck would be at real scale
- **Question:** If this were an actual service that a reasonable number of people used, where would the bottleneck be, first for speed and then for cost?
- **Asked by:** Luke Campbell
- **Answer:** _Pending._
- **Answered by:** _Pending._
- **Additional notes:** _Pending._ Context for the discussion:
  - The parts that grow with use are the Matcher's Bedrock ranking call (one per posting and student, JDM-005), the Renderer's TeX compile in a large container image (DEL-T01), and the parser's occasional Bedrock call (EB-024).
  - Limits that apply now: the Bedrock cost caps ($1 per student per hour, $5 per hour, $1.50 per day; OPS-002, OPS-010), Bedrock tokens per minute, and the account's Lambda concurrency (OPS-T11 criterion 7).
  - The answer feeds Phase 5 (load planning).
- **Note taker:** _Pending._
- **Recorded:** 2026-11-24 (agent working for Ryan Fusco, question from Luke Campbell)

### MQ-2026-11-19-04: CloudWatch
- **Question:** CloudWatch.
- **Asked by:** Luke Campbell
- **Answer:** _Pending._
- **Answered by:** _Pending._
- **Additional notes:** _Pending._ Context for the discussion:
  - Recorded as Luke Campbell gave it. He'll state the full question at the meeting.
  - What the plans use CloudWatch for: JSON logs kept 14 days, EMF metrics, alarms (including DLQ depth above 0), a dashboard, and SNS alerts (OPS-I04, OPS-015).
- **Note taker:** _Pending._
- **Recorded:** 2026-11-24 (agent working for Ryan Fusco, question from Luke Campbell)
