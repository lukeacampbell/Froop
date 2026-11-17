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
Last reviewed: 2026-11-16

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Architecture (as given), Architecture Review, Decisions, Interfaces, Open Questions | Draft. All sections are drafted. EB-001 to EB-006 and EB-010 to EB-013 are Accepted. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Not created yet. Created in step 6 of "How to Plan an Area". |

Where to look:

- **What the area is responsible for:** `architecture.md` → Scope
- **What other areas rely on from this area:** `architecture.md` → Interfaces → Provides
- **What's still undecided:** `architecture.md` → Open Questions
- **What was actually decided:** `architecture.md` → Decisions. These override the as-given text.
- **What's being built, and in what order:** `implementation-plan.md`

## Working Agreement

**Agents may, without asking:**
- Transcribe the area's part of the RFC into "Architecture (as given)".
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
