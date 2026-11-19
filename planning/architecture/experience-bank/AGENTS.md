# Experience Bank AGENTS.md

Owner: Ryan Fusco
Contributors: —

This file is the working agreement for agents in the experience-bank area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

The experience bank holds everything a student has done, in the student's own words, so that every resume Froop produces is built only from text the student wrote.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-19

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions | Draft. All sections are drafted. EB-001 to EB-003, EB-005, EB-006, EB-008 to EB-010, and EB-012 to EB-019 are Accepted. EB-004 and EB-011 are superseded, and EB-007 is deferred. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-17. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. EB-T01 to EB-T07. The parsing core (EB-T03, EB-T04) is Planned. The rest is Blocked on the shared database and build-and-operation contracts. |

Where to look:

- **What other areas need from this owner:** `architecture.md` → Requests From Other Areas
- **What the area is responsible for:** `architecture.md` → Scope
- **How the area fits together right now:** `architecture.md` → Current Design Diagram
- **What other areas rely on from this area:** `architecture.md` → Interfaces → Provides
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the Initial Architecture.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the area's part of the RFC into "Initial Architecture".
- Add review findings, and draft `Proposed` decisions, interfaces, and Open Questions.
- Update `Last reviewed` after a substantive revision.

**Check with Ryan Fusco first before:**
- Changing the Scope.
- Changing an interface that another area consumes. This also needs that area's owner (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost, a new external service, or a new kind of user data.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**Ground rules.** Every proposal in this area must uphold these:
- **The student's words are never changed.** Nothing in this area may rewrite, paraphrase, or "improve" the text a student uploads. RFC Goal 3 and the non-goal "no AI-written resume text" (p.2) require this. A proposal that can't guarantee it gets flagged, not accepted.
- **A student's data is visible only to that student.** Every proposal must state how it keeps one student's material away from every other student.
