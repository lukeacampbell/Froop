# Database AGENTS.md

Owner: Ryan Fusco (pending SYS-Q02)
Contributors: Graham Cohick, Luke Campbell (the item types their areas write)

This file is the working agreement for agents in the database area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

The database area designs the one shared data store that every other area reads and writes, so that each area can get the data it needs without getting in another area's way.

Every area depends on this design, so it's coordinated in one place rather than split across areas.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-12

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Architecture (as given), Architecture Review, Decisions, Access Patterns, Item Types, Interfaces, Open Questions | Draft. Scope, Item Types, Interfaces, and Open Questions are drafted. The as-given text and the access patterns haven't been transcribed yet. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Not created yet. Created in step 6 of "How to Plan an Area". |

Where to look:

- **What the area is responsible for:** `architecture.md` → Scope
- **Every way the data is read or written, and which area needs it:** `architecture.md` → Access Patterns
- **Which item types exist, which area writes each one, and who reads it:** `architecture.md` → Item Types
- **The detailed definition of each item type:** `architecture.md` → Interfaces → Provides
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the as-given text.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the RFC's data model into "Architecture (as given)".
- Add review findings, and draft `Proposed` decisions, access patterns, item types, and Open Questions.
- Update `Last reviewed` after a substantive revision.

**Contributors** (the owners of areas that write data) may add `Proposed` access patterns and item-type definitions for their own area's data. They can't accept them.

**Check with Ryan Fusco first before:**
- Changing the Scope.
- Changing the shared structure that every item type depends on (DB-I01).
- Changing an item type. This also needs the owner of the area that writes it, and every area that reads it must be told (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost, or a second data store.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**Ground rules.** Every proposal in this area must uphold these:
- **Access patterns come first.** Nothing is added to the design unless an access pattern listed in `architecture.md` needs it.
- **One student's data is never returned by a request for another student's data.**
- **Changes are additive.** Data that another area already uses is never renamed or removed without that area's owner agreeing.
- **Duplicates are blocked by the data store, not by luck.** Where a goal requires "no duplicates" (RFC Goal 1, p.2), the design must make duplicate writes impossible.
