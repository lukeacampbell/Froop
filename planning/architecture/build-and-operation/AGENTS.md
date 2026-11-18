# Build and Operation AGENTS.md

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This area is shared (see "Shared Ownership" in `planning/architecture/AGENTS.md`; pending confirmation in SYS-Q01).
Contributors: —

This file is the working agreement for agents in the build-and-operation area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Build and Operation makes sure every part of Froop can be built, deployed, watched, and paid for safely and the same way every time. It serves the other areas, so its work starts from what they say they need.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-17

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Requirements Intake, Open Questions | Draft. All sections are drafted. OPS-001 to OPS-010 are Proposed, waiting for every owner to accept. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-17. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Not created yet. Created in step 6 of "How to Plan an Area". |

Where to look:

- **What other areas need from the owners:** `architecture.md` → Requests From Other Areas
- **What the area is responsible for:** `architecture.md` → Scope
- **How the area fits together right now:** `architecture.md` → Current Design Diagram
- **What every other area can rely on:** `architecture.md` → Interfaces → Provides
- **What other areas have asked for:** `architecture.md` → Requirements Intake
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the Initial Architecture.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the area's part of the RFC into "Initial Architecture".
- Add review findings, and draft `Proposed` decisions, interfaces, and Open Questions.
- Add Requirements Intake rows for needs another area has recorded, citing where they're recorded.
- Update `Last reviewed` after a substantive revision.

**Check with the owners first before** (ask the person you're working for, and record the other owners' acceptance on the item):
- Changing the Scope.
- Changing an interface. Every area consumes these, so a change also needs the affected owners (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost or a new AWS or third-party service.
- Proposing anything that loosens access or permissions.
- Moving any item to `Accepted`. That needs every owner (see "Shared Ownership").
- Moving the area to `In Review` (the coordinator) or `Approved` (every owner).

**Ground rules.** Every proposal in this area must uphold these:
- **Everything can be rebuilt from the repository.** Anything that has to be done by hand is listed as an exception, with its reason.
- **No secrets in the repository** or in CI settings.
- **Every component gets only the access it needs.**
- **Cost is visible and capped.** Anything that spends money has a metric or an alarm.
- **Needs come from the areas that have them.** Don't add infrastructure that no area has asked for in the Requirements Intake.
