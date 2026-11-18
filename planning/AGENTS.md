# Planning AGENTS.md

This file governs how AI agents read, update, and act on the `/planning` directory.
`/planning` is the source of truth for **what** Froop is building, **why**, and **how we know it works**. Code is the source of truth for **how** it currently works. When the two disagree, stop and fix the disagreement. Do not quietly pick one.

## Project Phases

Agents must know which phase the project is in, because the allowed actions differ. Each block of work has its own planning phase, and that plan is approved before the work starts.

| Phase | Goal | Agents may | Agents may not |
|---|---|---|---|
| **1. Architecture Planning** (current) | Review the predefined architecture and produce an implementation plan for the specified area | Read, critique, propose changes, record decisions and open questions, edit architecture and implementation docs | Write application or test code, scaffold projects, add dependencies, or write test or load plans |
| **2. Implementation** | Build the system according to the approved architecture and implementation plans | Write application code and unit tests that trace back to a planned task; update task status; propose plan changes | Build features the plan doesn't cover; silently change architecture |
| **3. Test Planning** | Produce test plans for each area and for cross-area flows, based on the approved plans and the built system | Read code and planning docs; write and edit test plans; record coverage gaps as open questions | Write or run test code; change application code or architecture |
| **4. Testing** | Test the system according to the approved test plans | Read code and planning docs; write and run test code in test directories; record results; submit GitHub issues; propose changes | Change application code or architecture; implement new features |
| **5. Load Planning** | Define load targets, traffic scenarios, test environments, and a load test plan | Read code, infrastructure config, and planning docs; write and edit the load plan; propose infrastructure decisions | Run load tests; change application code or infrastructure |
| **6. Load Balancing** | Verify and tune the system against the approved load plan | Write and run load tests in the environments the load plan names; record results; propose scaling and infrastructure changes as decisions; submit GitHub issues | Run load against production or shared third-party services without human approval; apply infrastructure changes without an Accepted decision; change application features |

When the current phase changes, mark the new phase **(current)** in the table and the previous phase **(done)**. Only a human may change the project phase.

### Moving Between Phases

- A phase ends when its exit criteria (see "Phase Guidance") are met **and** a human confirms it.
- The project moves through the phases in order. When Phase 4 or 6 finds a defect, the agent files an issue. It does not fix it. A human decides whether to authorize the fix, which then follows the Phase 2 rules for that issue only.
- A plan from an earlier phase can still change later, following the normal update rules: record the change as `Proposed`, and a human accepts it. When the change touches work that is already covered by a plan, update that plan too (see "Keep the plans in sync").

## Directory Layout

Each planning phase owns one folder under `planning/`. Phases 2, 4, and 6 have no folder of their own. They carry out the plan from the phase before them and record status in its files. Each plan file is created in the phase that produces it and does not exist before then. All paths in this document are relative to `planning/`.

```
planning/
├── AGENTS.md                      # This file: rules, phases, directory map, formats
├── inputs/                        # Inputs for every phase
│   └── AGENTS.md
├── architecture/                  # Phase 1: Architecture Planning
│   ├── AGENTS.md
│   ├── inputs/                    # The predefined architecture and related material
│   │   └── AGENTS.md
│   └── <area>/
│       ├── AGENTS.md              # Guide to working in the area: purpose, status, files, working agreement
│       ├── architecture.md        # Scope, current design diagram, initial architecture (frozen), review, decisions, interfaces, open questions
│       ├── architecture.drawio    # Current design diagram (source), with architecture.png exported from it
│       └── implementation-plan.md # Ordered tasks with acceptance criteria
├── testing/                       # Phase 3: Test Planning
│   ├── AGENTS.md
│   ├── inputs/
│   │   └── AGENTS.md
│   ├── <area>-test-plan.md        # Test cases for one area, mapped to its tasks
│   └── system-test-plan.md        # Cross-area tests: end-to-end flows, integration across areas
└── load/                          # Phase 5: Load Planning
    ├── AGENTS.md
    ├── inputs/
    │   └── AGENTS.md
    └── load-plan.md               # Load targets, scenarios, environments, run log
```

Each folder's `AGENTS.md` holds the rules for that folder only. If one of them conflicts with this file, this file wins.

### Inputs

- **What inputs are:** `inputs/` folders hold source material that humans provide, such as requirements, the predefined architecture, assignment specs, and traffic estimates. The root `planning/inputs/` applies to every phase. A phase's `inputs/` applies only to that phase.
- **Agents only read inputs.** They never edit, move, or delete them. If an input looks wrong or out of date, record an Open Question.
- **Plans cite the inputs they rely on** by path. "Initial Architecture" is transcribed from `architecture/inputs/`.
- **Don't resolve conflicts between inputs.** When two inputs disagree, record an Open Question. Don't pick one.

### Areas

Each area is a folder under `architecture/`. Test plans in `testing/` use the same area names.

The system runs on AWS and is defined in RFC 0001 (`architecture/inputs/`).

- `/build-and-operation`:
  - Terraform: the bootstrap, the modules, and the `envs/dev` and `envs/prod` environments.
  - GitHub Actions CI/CD with OIDC, and ECR.
  - Provisioning the shared DynamoDB table (as `database` designs it) and the S3 buckets.
  - IAM, Secrets Manager, CloudWatch metrics, alarms, and the dashboard, plus SNS alerts.
  - Cost controls.
  - The scaling and throttling configuration: Lambda concurrency, SQS settings, and API Gateway throttling.
- `/database`: the design of the single shared DynamoDB table: its access patterns, key formats, indexes, TTL, and every item type, with the area that writes each one.
- `/delivery`: rendering one-page LaTeX resume PDFs for each match (the Renderer Lambda), storing them in S3 `resume/`, and emailing students instantly or in a digest (the Notifier Lambda and SES).
- `/experience-bank`: turning a student's single uploaded LaTeX resume (`.tex`) into a verbatim bank of the header, sections, entries, bullets, and skills (S3 `uploads/`, the Bank parser Lambda, and Bedrock only for leftover sections), and storing, replacing, and deleting it. Other areas use the bank through its shared package.
- `/job-discovery-and-matching`:
  - The catalog, private sources, and subscriptions.
  - Scheduled scraping (EventBridge Scheduler, SQS, and the Scraper Lambda).
  - Deduplicating postings.
  - Matching postings to users, and selecting bullets with Bedrock under a cost cap (the Matcher Lambda).
- `/web-app-and-api`: sign-in (Cognito), the REST API (API Gateway and the API Lambda), the React dashboard (S3 and CloudFront), user profiles and settings, and signed resume links.

Every concern belongs to exactly one area, and areas that use it refer to that owner. Tests and load scenarios that span areas belong in `testing/system-test-plan.md` and `load/load-plan.md`, which are owned by their planning phase rather than by an area. If an area description above is wrong, or two areas both claim the same thing, fix it here first.

### Which Files to Read in Each Phase

Bare area file names (`AGENTS.md`, `architecture.md`, `implementation-plan.md`) mean the files in `architecture/<area>/`.

| Phase | Primary files | Also read |
|---|---|---|
| 1. Architecture Planning | `architecture.md`, `implementation-plan.md` | Neighboring areas' `architecture.md` (Interfaces) |
| 2. Implementation | `implementation-plan.md`, `architecture.md` (Interfaces) | `architecture.md` (Decisions) |
| 3. Test Planning | `implementation-plan.md` (acceptance criteria), `architecture.md` (Interfaces) | `architecture.md` (Decisions), the code under test |
| 4. Testing | `testing/<area>-test-plan.md`, `testing/system-test-plan.md` | `implementation-plan.md` (acceptance criteria) |
| 5. Load Planning | `architecture/build-and-operation/architecture.md`, every area's `architecture.md` (Interfaces) | `testing/system-test-plan.md` (end-to-end flows), infrastructure config |
| 6. Load Balancing | `load/load-plan.md` | `architecture/build-and-operation/architecture.md` |

## File Formats

Every file uses the sections below, in this order, so agents can find things the same way in every area. Leave a section empty with `_None yet._` rather than deleting it.

### Area `AGENTS.md`

The area `AGENTS.md` is a guide to working in the area, an agreement between its owner and the agents working there. It never contains design or implementation content. That goes in `architecture.md` and `implementation-plan.md`.

```markdown
# <Area> AGENTS.md

Owner: <name>
Contributors: <name (what they contribute)> | —

## Purpose
One or two sentences on what the area is responsible for, in product terms. No services, schemas, or implementation choices.

## Status
Project phase: 1. Architecture Planning
Area state: Draft | In Review | Approved
Last reviewed: YYYY-MM-DD

## Files
Where to find each kind of content in this area, and which file to read first for common tasks.

## Working Agreement
What agents may do here without asking, what they must check with the owner first,
and any area-specific rules they must uphold.
```

`Area state` describes the area's deliverable for the current phase. It goes back to `Draft` when a new planning phase starts.

### `architecture.md` (Phase 1)

```markdown
## Scope
What this area owns. What it explicitly does not own (and who does).

## Current Design Diagram
![<Area>: current design](architecture.png), exported from `architecture.drawio`. Shows the current design, with Proposed items dashed.

## Initial Architecture
The architecture for this area as originally submitted (RFC 0001), recorded unchanged. It's a frozen baseline: never updated when decisions change the design. The current design is in Decisions.

## Architecture Review
Critique of the given architecture. Each item: finding → impact → recommendation.

## Decisions
Numbered, append-only log. See "Recording Decisions".

## Interfaces
### Provides
Interfaces this area defines (see the interface format below).
### Consumes
Interfaces this area uses, cited by the provider's ID.

## Open Questions
Unresolved items. Each one has an ID (EB-Q01), an owner (human or area), and what it blocks.

## Requests From Other Areas
Pointers to items defined in other areas that this area's owner has to act on:
| ID | Defined in | What's needed |
```

Interface format, used under "Provides":

```markdown
#### EB-I01: Experience data model
Status: Proposed | Accepted | Superseded by EB-I04
Consumers: job-discovery-and-matching, delivery
Contract: what is exchanged (schema, operations, errors, versioning).
Accepted by: provider owner (date), each consumer owner (date)
```

### `implementation-plan.md` (Phase 1)

Write acceptance criteria that can be tested. Phase 2 unit tests and the Phase 3 test plan are both built from them.

```markdown
### EB-T04: Add experience entry schema
Depends on: EB-T01, WEB-T02
Acceptance criteria:
1. ...
Status: Planned | In Progress | Done | Blocked (reason)
```

### `testing/<area>-test-plan.md` and `testing/system-test-plan.md` (Phase 3)

Each test case covers at least one acceptance criterion, and every acceptance criterion is covered by at least one test case. Unit tests written in Phase 2 are listed here too, so that coverage can be checked in one place.

```markdown
### EB-TC-01: Rejects an experience entry missing a start date
Covers: EB-T04 (criterion 1)
Type: unit | integration | e2e
Preconditions: ...
Steps: ...
Expected: ...
Status: Not Run | Pass | Fail (#issue)
```

`testing/system-test-plan.md` uses the prefix `SYS-TC-`. It covers the main end-to-end user flows and any integration that crosses areas.

### `load/load-plan.md` (Phase 5)

```markdown
## Targets
### SYS-LT-01: Job matching under peak traffic
Flow or endpoint: ...            (links to SYS-TC-xx where one exists)
Traffic profile: users, request rate, ramp, duration
Targets: p95 latency, error rate, throughput
Environment: ...
Status: Not Run | Met | Not Met (#issue)

## Environments
Where load tests run, how they're provisioned, and what they must never touch.

## Runs
Append-only log: date, configuration, results vs. targets, observed bottleneck.
```

### ID Prefixes

| Area | Prefix | Decision | Task | Interface | Test case | Question |
|---|---|---|---|---|---|---|
| experience-bank | `EB` | `EB-001` | `EB-T01` | `EB-I01` | `EB-TC-01` | `EB-Q01` |
| job-discovery-and-matching | `JDM` | `JDM-001` | `JDM-T01` | `JDM-I01` | `JDM-TC-01` | `JDM-Q01` |
| web-app-and-api | `WEB` | `WEB-001` | `WEB-T01` | `WEB-I01` | `WEB-TC-01` | `WEB-Q01` |
| delivery | `DEL` | `DEL-001` | `DEL-T01` | `DEL-I01` | `DEL-TC-01` | `DEL-Q01` |
| build-and-operation | `OPS` | `OPS-001` | `OPS-T01` | `OPS-I01` | `OPS-TC-01` | `OPS-Q01` |
| database | `DB` | `DB-001` | `DB-T01` | `DB-I01` | `DB-TC-01` | `DB-Q01` |
| system (root) | `SYS` | `SYS-001` | — | — | `SYS-TC-01`, load targets `SYS-LT-01` | `SYS-Q01` |

## How Agents Read Planning

Before proposing a change, writing code, or running tests:

1. **Read this file**, then the current planning folder's `AGENTS.md`, the root `inputs/`, and the current planning folder's `inputs/`. Then read the files listed for the current phase in every area your task touches.
2. **Read the Interfaces sections of neighboring areas.** Most mistakes happen at area boundaries.
3. **Check Decisions and Open Questions** before suggesting something. It may already be decided or under discussion.
4. **Treat the docs in this order of authority:** Accepted decisions > approved plans (implementation, test, load) > Architecture Review > Initial Architecture. A later Accepted decision overrides the original architecture.

If the docs don't answer what you need, don't guess and keep going. Add an Open Question and either ask the human or carry on with a stated assumption, labeled as one.

## How Agents Update Planning

- **Only edit the plans your phase allows.** Earlier plans change only through a `Proposed` change. Later plans don't exist yet.
- **Edit in place, keep the history.** Change plans directly, but never delete entries from Decisions or load-plan Runs. Mark an old decision `Superseded by <PREFIX>-NNN` instead.
- **Keep changes small and scoped.** One logical change per edit. A change that affects another area updates that area's Interfaces too, in the same change.
- **Separate fact from proposal.** Mark anything a human hasn't agreed to yet as `Proposed`. Only a human moves an item to `Accepted` or an area to `Approved`.
- **Don't edit "Initial Architecture"** except to fix transcription errors. Put critique in Architecture Review and changes in Decisions.
- **Keep the plans in sync.** Once a test plan or load plan exists, a change to a task's acceptance criteria, an interface, or the infrastructure updates the test cases or load targets it affects, in the same change.
- **Update `Last reviewed`** whenever you substantively revise an area.
- **Write for the next agent.** Each entry should make sense to someone with no memory of this conversation. Use specific nouns, name the files, and give the reasoning.

### Recording Decisions

```markdown
### EB-003: Store experience entries as structured records, not free text
Status: Proposed | Accepted | Superseded by EB-007
Date: YYYY-MM-DD
Context: Why this came up; what constraint or review finding triggered it.
Decision: What we will do.
Alternatives: What else was considered and why it lost.
Consequences: What this affects, including other areas and any test cases or load targets.
```

## How Agents Use Planning to Make Decisions

- **Ground every recommendation in the docs.** Cite the decision, task, test case, or load target you're relying on (for example, "per WEB-002").
- **Push back on the plan you're producing.** In every planning phase, critique is the job:
  - In Phase 1, look for:
    - missing failure modes and unclear ownership
    - overbuilt parts
    - scaling or cost problems
    - security and privacy gaps
    - parts that can't be tested
  - In Phase 3, look for untested edge cases, error paths, and area boundaries.
  - In Phase 5, look for targets that are unrealistic or can't be measured, and for environments that could affect real users or third parties.

  Give a recommendation, not just a list of concerns.
- **Prefer the simplest design that meets the stated requirements.** If you're adding complexity for a future need, write that need down explicitly.
- **Escalate instead of deciding** when a choice is hard to undo, crosses areas, affects cost or security, or contradicts an Accepted decision. Record it as a `Proposed` decision or an Open Question and flag it to the human.

## Phase Guidance

### Phase 1: Architecture Planning

Exit criteria for each area:

- [ ] Scope is clear and doesn't overlap any other area's
- [ ] The given architecture is recorded and reviewed
- [ ] The major choices are recorded as Accepted decisions
- [ ] Interfaces with every neighboring area are defined and match on both sides
- [ ] The Implementation Plan has ordered tasks with dependencies and acceptance criteria that can be tested
- [ ] No Open Question blocks the first tasks

### Phase 2: Implementation

- **Each piece of code traces to a task ID.** Reference it in branch names, commit messages, and PR descriptions (for example, `EB-T04: add experience entry schema`).
- **Write unit tests for each acceptance criterion** as part of the task. A task is not Done until they pass.
- **Follow the Interfaces exactly.** If a contract needs to change, update the planning docs in both affected areas first, or in the same PR.
- **Update task status** in the Implementation Plan when you start and finish work.
- **When the code teaches you something** (a planned approach fails, an assumption is wrong), record it as a Proposed decision or an Open Question. Don't work around it silently.
- **Don't build unplanned work.** If something is needed and isn't in the plan, add it as a proposed task and get it approved.

Exit criteria:

- [ ] Every task is Done
- [ ] All unit tests pass in CI
- [ ] The implemented interfaces match the planning docs

### Phase 3: Test Planning

- **Start from the acceptance criteria.** Every criterion in `implementation-plan.md` must be covered by at least one test case.
- **List the existing unit tests** from Phase 2 as test cases, then add integration and end-to-end cases for what they don't cover.
- **Test the boundaries.** Each interface in an area's `architecture.md` needs at least one test case. Each test case that spans areas goes in `testing/system-test-plan.md`.
- **Plan against the system that was actually built.** When the code and the plans disagree, record an Open Question. Don't write a test plan for behavior that only exists on paper.

Exit criteria:

- [ ] Every acceptance criterion and every interface is covered by a test case
- [ ] `testing/system-test-plan.md` covers the main end-to-end user flows
- [ ] A human has approved every test plan

### Phase 4: Testing

- **Run tests from the approved plan.** Test code goes only in test directories. If you find a gap in coverage, add a `Proposed` test case and get it approved before writing it.
- **Record every result** in the test case's `Status`.
- **File one GitHub issue per defect.** Search for duplicates first. Each issue includes:
  - Title: `[<TEST-ID>] <short description>`
  - Body: steps to reproduce, expected vs. actual, the environment, and the task or decision you suspect
  - The issue linked back from the test case's `Status`
- **Don't fix defects.** Even an obvious one-line fix waits for human authorization (see "Moving Between Phases").

Exit criteria:

- [ ] Every test case has been run
- [ ] No open issues are labeled blocking
- [ ] A human has signed off on the results

### Phase 5: Load Planning

- **Base scenarios on real user flows.** Build each load target from an end-to-end flow in `testing/system-test-plan.md`, or from an endpoint you expect to be under heavy use, and link to it.
- **Make targets measurable.** Every target states its traffic profile and numeric limits for latency, error rate, and throughput. If expected traffic is unknown, record an Open Question for the human. Don't invent numbers.
- **Isolate the environments.** For each environment, define where it runs and how it is provisioned. List the external services that must be stubbed or rate-limited, and confirm nothing points at production.
- **Record needed infrastructure** (load balancers, autoscaling, caching, test tooling) as `Proposed` decisions in `architecture/build-and-operation/architecture.md`.

Exit criteria:

- [ ] Every target has a traffic profile, numeric limits, and an environment
- [ ] The environments are defined and isolated from production and third-party services
- [ ] A human has approved the load plan

### Phase 6: Load Balancing

- **Test against the approved targets** in `load/load-plan.md`, in the environment each target names. Never use production unless a human explicitly approves it.
- **Record every run** in the load plan's Runs log, and update the target's `Status`.
- **Propose tuning as decisions.** Scaling, load-balancer, caching, or infrastructure changes go in as `Proposed` decisions in `architecture/build-and-operation/architecture.md`. Apply them only after they are Accepted, then re-run the affected targets.
- **File GitHub issues** for application-level bottlenecks, using the format from Phase 4 with the load target ID in the title.

Exit criteria:

- [ ] Every load target is Met, or a human has accepted the gap
- [ ] Every Accepted tuning decision is reflected in the infrastructure config
