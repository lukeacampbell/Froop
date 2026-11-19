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
Last reviewed: 2026-11-19

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current. Purpose and ground rules approved by the owner on 2026-11-18. |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions, Requests From Other Areas | Draft. Scope (confirmed), Initial Architecture, Architecture Review (17 findings), Decisions WEB-001 to WEB-013, Interfaces (WEB-I01 and WEB-I02, both Proposed), and Open Questions have content (WEB-Q01 to WEB-Q07 answered; WEB-Q05 and WEB-Q08 open). Requests From Other Areas is empty. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Out of date since 2026-11-18. Drawn on 2026-11-17 from the initial architecture plus other areas' decisions. It doesn't yet carry the IDs of WEB-001 to WEB-011, and its "Waiting on this owner" note still lists EB-Q10 and DB-Q06, which are answered. No component changed. Update and re-export it before the area goes to `In Review`. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. 16 tasks (WEB-T01 to WEB-T16). WEB-T01 and WEB-T11 are Planned, and the rest are Blocked on named decisions, interfaces, or questions. |

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

**Ground rules.** Every proposal in this area must uphold these:
- **The user always comes from the sign-in token,** never from the request.
- **A student can never see or change another student's data.**
- **The API carries other areas' features but doesn't redefine their rules.** Upload rules, source validation, matching, and rendering stay with the areas that own them.
- **Prefer the simplest thing the demo needs.** Anything that adds cost or a new service needs Graham Cohick's say.
