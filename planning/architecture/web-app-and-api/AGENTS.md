# Web App and API AGENTS.md

Owner: Graham Cohick
Contributors: —

This file is the working agreement for agents in the web-app-and-api area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Web App and API is how a student gets into Froop and uses it: signing in, uploading a resume, choosing which job boards to follow, setting how they want to hear about matches, and looking back at each match and the resume made for it.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-24

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current. Purpose and ground rules approved by the owner on 2026-11-18. |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions, Requests From Other Areas | Draft. Scope (confirmed), Initial Architecture, Architecture Review (17 findings), Decisions WEB-001 to WEB-014, Interfaces (WEB-I01 Accepted; WEB-I02 Proposed, waiting for Luke Campbell), and Open Questions have content (WEB-Q01 to WEB-Q08 answered). Requests From Other Areas is empty. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-23. It carries the IDs of the Accepted decisions, the presigned POST (EB-018), and the API's link to job discovery for refresh and regeneration (JDM-I01, JDM-I05, WEB-013). Its dashed "Still open" box lists what waits on Luke Campbell. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. 16 tasks (WEB-T01 to WEB-T16). WEB-T10, WEB-T12, and WEB-T14 are Blocked on DB-I04 amendment A (waiting for Luke Campbell); the rest are Planned. |
| Code (Phase 2) | `services/api/`, `web/`, and `infra/modules/app/`. These folders are created by OPS-T01. | Not yet created |

Where to look:

- **What other areas need from this owner:** `architecture.md` → Requests From Other Areas. **Start here.**
- **What the area is responsible for:** `architecture.md` → Scope
- **How the area fits together right now:** `architecture.md` → Current Design Diagram
- **What other areas rely on from this area:** `architecture.md` → Interfaces → Provides
- **What this area needs from others:** `architecture.md` → Interfaces → Consumes
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the Initial Architecture.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the area's part of the RFC into "Initial Architecture". See "Where Each Area's Material Is" in `planning/architecture/AGENTS.md`.
- Add review findings, and draft `Proposed` decisions, interfaces, and Open Questions.
- Update `Last reviewed` after a substantive revision.

**Check with Graham Cohick first before:**
- Changing the Scope.
- Changing an interface that another area consumes. This also needs that area's owner (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost or a new external service.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**In Phase 2 (implementation)**, these replace the planning-only items above. The repository's `AGENTS.md` lists the rules for all code, and the Code Map of which folder belongs to which area.

*Agents may, without asking:*
- Implement a task whose status is `Planned`, exactly as its acceptance criteria say, once the group has confirmed Phase 2.
- Write unit tests for each acceptance criterion, in the code folder's own tests.
- Set the task's status to `In Progress` and `Done` in `implementation-plan.md`.
- Record a `Proposed` decision or an Open Question in `architecture.md` when the code shows that a plan is wrong or incomplete.

*Check with Graham Cohick first before:*
- Changing an acceptance criterion, or building something no task covers. Propose the change in `implementation-plan.md` first.
- Changing the shape of anything listed under Interfaces → Provides. This also needs every consumer's owner.
- Adding a third-party dependency, an AWS service, or anything that costs money.
- Editing a folder that belongs to another area (see the Code Map in the repository's `AGENTS.md`).

*Rules for this area's code:*
- **Endpoints, request and response shapes, and error codes are exactly WEB-I01.** A new field or status code is an interface change.
- **Take the `sub` only from the validated token** (WEB-005). Report another student's item as 404.
- **Use the other areas' packages:** the bank and uploads through `libs/experience_bank` (EB-I03), every table key through the key builders (DB-T02). Don't reimplement their rules (source validation is JDM-I01, upload rules are EB-I01).
- **Reject unknown request fields** with 400, and keep error messages free of resume text.
- **Never log** tokens, email addresses, or resume text.
- **The dashboard reads its settings** (API URL, Cognito client and domain) from build-time values that CI takes from Terraform outputs (OPS-012). Nothing environment-specific is hard-coded.

**Ground rules.** Every proposal in this area must uphold these:
- **The user always comes from the sign-in token,** never from the request.
- **A student can never see or change another student's data.**
- **The API carries other areas' features but doesn't redefine their rules.** Upload rules, source validation, matching, and rendering stay with the areas that own them.
- **Prefer the simplest thing the demo needs.** Anything that adds cost or a new service needs Graham Cohick's say.
