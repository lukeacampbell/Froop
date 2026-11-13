# Build and Operation AGENTS.md

Owner: Ryan Fusco (pending SYS-Q01)
Contributors: —

This file is the working agreement for agents in the build-and-operation area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Build and Operation makes sure every part of Froop can be built, deployed, watched, and paid for safely and the same way every time. It serves the other areas, so its work starts from what they say they need.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-12

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Architecture (as given), Architecture Review, Decisions, Interfaces, Requirements Intake, Open Questions | Draft. Scope, Interfaces, Requirements Intake, and Open Questions are drafted. The as-given text hasn't been transcribed yet. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Not created yet. Created in step 6 of "How to Plan an Area". |

Where to look:

- **What the area is responsible for:** `architecture.md` → Scope
- **What every other area can rely on:** `architecture.md` → Interfaces → Provides
- **What other areas have asked for:** `architecture.md` → Requirements Intake
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the as-given text.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the area's part of the RFC into "Architecture (as given)".
- Add review findings, and draft `Proposed` decisions, interfaces, and Open Questions.
- Add Requirements Intake rows for needs another area has recorded, citing where they're recorded.
- Update `Last reviewed` after a substantive revision.

**Check with Ryan Fusco first before:**
- Changing the Scope.
- Changing an interface. Every area consumes these, so a change also needs the affected owners (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost or a new AWS or third-party service.
- Proposing anything that loosens access or permissions.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**Ground rules.** Every proposal in this area must uphold these:
- **Everything can be rebuilt from the repository.** Anything that has to be done by hand is listed as an exception, with its reason.
- **No secrets in the repository** or in CI settings.
- **Every component gets only the access it needs.**
- **Cost is visible and capped.** Anything that spends money has a metric or an alarm.
- **Needs come from the areas that have them.** Don't add infrastructure that no area has asked for in the Requirements Intake.
