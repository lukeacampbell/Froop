# Delivery AGENTS.md

Owner: Ryan Fusco
Contributors: —

This file is the working agreement for agents in the delivery area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Delivery gives a student a finished one-page resume for every job match, and tells them about it in the way they've chosen.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-18

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions | Draft. All sections are drafted. DEL-004 to DEL-008, DEL-009 to DEL-011 and DEL-014 to DEL-020 are Accepted. DEL-012 is superseded. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-17. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Not created yet. Created in step 6 of "How to Plan an Area". |

Where to look:

- **What other areas need from this owner:** `architecture.md` → Requests From Other Areas
- **What the area is responsible for:** `architecture.md` → Scope
- **How the area fits together right now:** `architecture.md` → Current Design Diagram
- **What other areas rely on from this area:** `architecture.md` → Interfaces → Provides
- **What this area needs from others:** `architecture.md` → Interfaces → Consumes
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
- Changing what a student receives, or how and how often they receive it.
- Changing what the Renderer allows from student LaTeX.
- Proposing anything that adds cost or a new external service.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**Ground rules.** Every proposal in this area must uphold these:
- **A resume contains only the student's words.** Delivery may choose a layout and drop formatting, but may never add, change, or reword text (RFC non-goal "no AI-written resume text", p.2).
- **Nothing a student uploads is ever run as unrestricted code.** That includes when their LaTeX is used to build a resume.
- **A resume fits on one page** (RFC Goal 4, p.2).
- **A student hears about each match at most once,** in the way they chose, even when something is retried.
- **A student's resumes are visible only to that student.**
