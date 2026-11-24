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
Last reviewed: 2026-11-24

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions | Draft. All sections are drafted. EB-001 to EB-003, EB-005, EB-006, EB-008 to EB-010, and EB-012 to EB-026 are Accepted. EB-004 and EB-011 are superseded, and EB-007 is deferred. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-23. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. EB-T01 to EB-T08. EB-T06 is Blocked on EB-Q14 (Luke Campbell); the rest are Planned. |
| Code (Phase 2) | `services/parser/`, `libs/experience_bank/`, and the parser's part of `infra/modules/resumes/`. These folders are created by OPS-T01. | Not yet created |

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
- **`libs/experience_bank/` is the only code that reads or writes `DOC#` items or `uploads/` objects** (EB-010). Its public functions are EB-I03; their names, arguments, and return shapes are a contract with three other areas.
- **The supported-template list lives only in the package** (`supported_templates()`, EB-015). Neither the parser nor delivery keeps its own copy.
- **Never invoke TeX** or any other program on an upload (EB-009). Parse it as text.
- **Every stored `source` is an exact substring of the upload.** Keep the verbatim test (EB-T04 criterion 6) passing on every fixture.
- **Test fixtures are synthetic.** Never commit a real student's resume, name, phone number, or email address.
- **Bedrock is called only after a reservation** through JDM-I04 (EB-017), only for leftover sections (EB-003), and only as EB-024 describes: one section, never the header, a forced-tool reply, and every returned span checked against the source.

**Ground rules.** Every proposal in this area must uphold these:
- **The student's words are never changed.** Nothing in this area may rewrite, paraphrase, or "improve" the text a student uploads. RFC Goal 3 and the non-goal "no AI-written resume text" (p.2) require this. A proposal that can't guarantee it gets flagged, not accepted.
- **A student's data is visible only to that student.** Every proposal must state how it keeps one student's material away from every other student.
