# Build and Operation AGENTS.md

Owners: Ryan Fusco (coordinator), Graham Cohick, Luke Campbell. This area is shared (see "Shared Ownership" in `planning/architecture/AGENTS.md`; confirmed in SYS-Q01).
Contributors: —

This file is the working agreement for agents in the build-and-operation area. It explains how to find your way around the area and what you may do here. It has no design or implementation content. That lives in `architecture.md` and `implementation-plan.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here. This file only adds what's specific to this area.

## Purpose

Build and Operation makes sure every part of Froop can be built, deployed, watched, and paid for safely and the same way every time. It serves the other areas, so its work starts from what they say they need.

## Status
Project phase: 1. Architecture Planning
Area state: Draft
Last reviewed: 2026-11-24

## Files

| File | Holds | Status |
|---|---|---|
| `AGENTS.md` | This guide | Current |
| `architecture.md` | Scope, Initial Architecture, Architecture Review, Decisions, Interfaces, Requirements Intake, Open Questions | Draft. OPS-001 to OPS-012 and OPS-I01 to OPS-I05 are Accepted. OPS-013 (dev only) is Proposed. |
| `architecture.drawio`, `architecture.png` | The current design diagram, embedded in `architecture.md` | Current as of 2026-11-23. Update it with every change to the design. |
| `implementation-plan.md` | Ordered tasks with acceptance criteria | Draft. OPS-T01 to OPS-T10, all Planned. Every owner must agree to changes. |
| Code (Phase 2) | `infra/bootstrap/`, `infra/modules/data/` and `infra/modules/ops/`, `infra/envs/`, `.github/workflows/`, the shared base image, and the logging helper in `libs/`. These folders are created by OPS-T01. | Not yet created |

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
- **Every setting is a variable** in the environment's `tfvars` (OPS-T03): cost caps, budget, models, sign-up switch, CORS origins.
- **No secret in the repository, in CI settings, or in Terraform state or outputs** (OPS-T07, OPS-014).
- **No IAM statement uses `"Resource": "*"` for data access** (OPS-T07).
- **Each area's Lambdas, queues, and roles go in that area's module** (OPS-011, SYS-Q03). This area provides the modules' shared conventions, not their contents.
- **Changes to CI, the shared base image, or a shared module affect every area.** Tell the other owners in the PR description.
- **Terraform is applied by `deploy.yml`,** except the one-time bootstrap (OPS-T02).

**Ground rules.** Every proposal in this area must uphold these:
- **Everything can be rebuilt from the repository.** Anything that has to be done by hand is listed as an exception, with its reason.
- **No secrets in the repository** or in CI settings.
- **Every component gets only the access it needs.**
- **Cost is visible and capped.** Anything that spends money has a metric or an alarm.
- **Needs come from the areas that have them.** Don't add infrastructure that no area has asked for in the Requirements Intake.
