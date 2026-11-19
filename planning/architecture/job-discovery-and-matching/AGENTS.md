# Job Discovery and Matching AGENTS.md

Owner: Luke Campbell
Contributors: Ryan Fusco (the Matcher and bullet selection)

This file is the working agreement for agents in the job-discovery-and-matching area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Find Greenhouse jobs from subscribed boards and select the student’s existing experience for a tailored resume.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-18

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Updated for Luke’s Greenhouse-only planning request. |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Open Questions, Requests From Other Areas | Scope, review, ten decisions, interfaces, and remaining agreements drafted. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Updated 2026-11-18 for Greenhouse-only scope and proposed recovery flow. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Drafted: JDM-T01 to JDM-T10, with acceptance criteria and explicit blockers. |

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

**Contributors:** an agent working for Ryan Fusco may add `Proposed` findings, decisions, interfaces, tasks, and Open Questions about the Matcher and bullet selection only. Luke Campbell accepts them.

**Check with Luke Campbell first before:**
- Changing the Scope.
- Changing an interface that another area consumes. This also needs that area's owner (see "Coordinating Across Areas" in `planning/architecture/AGENTS.md`).
- Proposing anything that adds cost or a new external service.
- Moving any item to `Accepted`, or the area to `In Review` or `Approved`.

**Ground rules:** Greenhouse only for this version. State each decision and its reasoning. Keep work in Phase 1; preserve the submitted baseline; do not attribute agreement to other owners.
