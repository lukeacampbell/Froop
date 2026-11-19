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
