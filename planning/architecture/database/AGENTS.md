# Database AGENTS.md

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This area is shared (see "Shared Ownership" in `planning/architecture/AGENTS.md`; confirmed in SYS-Q02).
Contributors: —

This file is the working agreement for agents in the database area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

The database area designs the one shared data store that every other area reads and writes, so that each area can get the data it needs without getting in another area's way.

Every area depends on this design, so it's coordinated in one place rather than split across areas.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-23

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Access Patterns, Item Types, Interfaces, Open Questions | Draft. DB-001 and DB-I01 to DB-I05 (DB-I04 base) are Accepted. DB-I04 amendment A (regeneration) is Proposed, waiting for Luke Campbell. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-23. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. DB-T01 to DB-T03. All Planned. The regeneration parts of DB-T02 and DB-T03 wait on DB-I04 amendment A. Every owner must agree to changes. |
| Code (Phase 2) | The table definition in `infra/modules/data/` (DB-T01), the key-builder package in `libs/` (DB-T02), and the access-pattern tests (DB-T03). These folders are created by OPS-T01. | Not yet created |

Where to look:

- **What other areas need from the owners:** `architecture.md` → Requests From Other Areas
- **What the area is responsible for:** `architecture.md` → Scope
- **How the area fits together right now:** `architecture.md` → Current Design Diagram
- **Every way the data is read or written, and which area needs it:** `architecture.md` → Access Patterns
- **Which item types exist, which area writes each one, and who reads it:** `architecture.md` → Item Types
- **The detailed definition of each item type:** `architecture.md` → Interfaces → Provides
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the Initial Architecture.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the RFC's data model into "Initial Architecture".
- Add review findings, and draft `Proposed` decisions, access patterns, item types, and Open Questions.
- Update `Last reviewed` after a substantive revision.

**Every owner** may add `Proposed` access patterns and item types, especially for the data their own areas write.

**Check with the owners first before** (ask the person you're working for, and record the other owners' acceptance on the item):
- Changing the Scope.
- Changing the shared structure that every item type depends on (DB-I01).
- Changing an item type. This also needs the owner of the area that writes it, and every area that reads it must be told (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost, or a second data store.
- Moving any item to `Accepted`. That needs every owner (see "Shared Ownership").
- Moving the area to `In Review` (the coordinator) or `Approved` (every owner).

**In Phase 2 (implementation)**, these replace the planning-only items above. The repository's `AGENTS.md` lists the rules for all code, and the Code Map of which folder belongs to which area.

*Agents may, without asking:*
- Implement a task whose status is `Planned`, exactly as its acceptance criteria say, once the group has confirmed Phase 2.
- Write unit tests for each acceptance criterion, in the code folder's own tests.
- Set the task's status to `In Progress` and `Done` in `implementation-plan.md`.
- Record a `Proposed` decision or an Open Question in `architecture.md` when the code shows that a plan is wrong or incomplete.

*Check with the owners (Ryan Fusco, Graham Cohick, Luke Campbell) first before:*
- Changing an acceptance criterion, or building something no task covers. Propose the change in `implementation-plan.md` first.
- Changing the shape of anything listed under Interfaces → Provides. This also needs every consumer's owner.
- Adding a third-party dependency, an AWS service, or anything that costs money.
- Editing a folder that belongs to another area (see the Code Map in the repository's `AGENTS.md`).

*Rules for this area's code:*
- **Every key is built by the key builders** (DB-T02). No other code formats `PK`, `SK`, or index keys itself.
- **A new item type needs a row in Item Types first,** and a test in DB-T03 before any code relies on its access pattern.
- **Queries only,** except the two paginated scans DB-AP15 and DB-AP16.
- **Duplicates are blocked with conditional writes,** never by checking first.
- **Every write checks item size** against the 300 KiB ceiling (DB-T02 criterion 4).

**Ground rules.** Every proposal in this area must uphold these:
- **Access patterns come first.** Nothing is added to the design unless an access pattern listed in `architecture.md` needs it.
- **One student's data is never returned by a request for another student's data.**
- **Changes are additive.** Data that another area already uses is never renamed or removed without that area's owner agreeing.
- **Duplicates are blocked by the data store, not by luck.** Where a goal requires "no duplicates" (RFC Goal 1, p.2), the design must make duplicate writes impossible.
