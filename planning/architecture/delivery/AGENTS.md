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
Last reviewed: 2026-11-24

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions | Draft. All sections are drafted. DEL-001 to DEL-011 and DEL-013 to DEL-023 are Accepted, and DEL-012 is superseded. DEL-I01 is Accepted. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-23. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. DEL-T01 to DEL-T07. All Planned. |
| Code (Phase 2) | `services/renderer/`, `services/notifier/`, and delivery's part of `infra/modules/resumes/` (the Renderer, the Notifier, the render-queue consumer, and the digest schedules). These folders are created by OPS-T01. | Not yet created |

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

**In Phase 2 (implementation)**, these replace the planning-only items above. The repository's `AGENTS.md` lists the rules for all code, and the Code Map of which folder belongs to which area.

*Agents may, without asking:*
- Implement a task whose status is `Planned`, exactly as its acceptance criteria say, once the group has confirmed Phase 2.
- Write unit tests for each acceptance criterion, in the code folder's own tests.
- Set the task's status to `In Progress` and `Done` in `implementation-plan.md`.
- Record a `Proposed` decision or an Open Question in `architecture.md` when the code shows that a plan is wrong or incomplete.

*Check with Ryan Fusco first before:*
- Changing an acceptance criterion, or building something no task covers. Propose the change in `implementation-plan.md` first.
- Changing the shape of anything listed under Interfaces → Provides. This also needs every consumer's owner.
- Adding a third-party dependency, an AWS service, or anything that costs money.
- Editing a folder that belongs to another area (see the Code Map in the repository's `AGENTS.md`).

*Rules for this area's code:*
- **The Renderer compiles only the Froop template,** with the macro allowlist and the hardened settings (DEL-001, DEL-002). The allowlist is `supported_templates()` plus `allowed_commands()` from the package, never a local copy (DEL-023). It never uses a student's preamble.
- **Read the bank only through `libs/experience_bank`** (EB-I03). Never query `DOC#` items.
- **Never change a match's `generation`.** Check it on every write (DEL-017, DB-I04 amendment A).
- **Never overwrite a resume.** Each render writes `resume/{sub}/{postingKey}/g{generation}.pdf` (DEL-009).
- **Email at most once:** `pending` → `sending` → `sent`, with conditional writes (DEL-018, DEL-022). A `regenerate` render never sends email (DEL-008).
- **Never log** email addresses, resume text, or signed URLs.

**Ground rules.** Every proposal in this area must uphold these:
- **A resume contains only the student's words.** Delivery may choose a layout and drop formatting, but may never add, change, or reword text (RFC non-goal "no AI-written resume text", p.2).
- **Nothing a student uploads is ever run as unrestricted code.** That includes when their LaTeX is used to build a resume.
- **A resume fits on one page** (RFC Goal 4, p.2).
- **A student hears about each match at most once,** in the way they chose, even when something is retried.
- **A student's resumes are visible only to that student.**
