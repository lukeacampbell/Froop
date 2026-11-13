# Architecture Planning Inputs AGENTS.md

This folder holds source material for **Phase 1: Architecture Planning** only. That means the predefined architecture the team proposed and anything that describes it, such as diagrams or reviewer feedback. Material that applies to every phase goes in `planning/inputs/` instead.

Inputs are what the architecture plans are built from. Each area's "Architecture (as given)" section is transcribed from the files here. Plans cite inputs, and inputs never cite plans.

## Rules for Agents

- **Read-only.** Don't edit, rename, move, or delete any file here except this `AGENTS.md`. Only humans add or replace inputs.
- **Keep the index current.** This `AGENTS.md` is the only file here that agents may edit. When you find a file that isn't listed in the index below, read it and add a row. If a listed file is gone, remove its row. Don't change anything else in this file without the human you're working for asking you to.
- **Cite by path and location.** When a plan relies on an input, cite the file and the page or section, for example `architecture/inputs/Froop_inital_architecture.pdf p.5`.
- **Don't fill gaps yourself.** If an input is unclear or incomplete, record an Open Question in the affected area. Its owner is that area's owner.

## Reading These Files

- **Skip template guidance.** Gray *italic* text under each heading in the RFC is the course template's instructions, not the team's design. Transcribe only the team's text. Course requirements stated in that guidance are hard constraints. They belong in `planning/inputs/`, so ask a human to add a copy there.
- **Read `.drawio` files as XML.** The `value` attributes hold the component labels, and each component sits in a lane container named after an area. Edges have no labels in the XML, so read the arrows from the exported image.

## Precedence

- **Inputs in `planning/inputs/` win over inputs here.** When a design here breaks a course requirement, record an Open Question. Don't silently follow either one.
- **When a document and its diagram disagree,** the document text wins, because it's the submitted version. Record the disagreement as an Open Question for the affected area's owner.
- **When two versions of the same input exist,** the one marked current in the index wins.

## Adding or Replacing Inputs (for humans)

- **Never overwrite a file.** Add the new version alongside the old one, with a date or version in the name, for example `Froop_architecture-2026-11-20.pdf`, and mark it current in the index.
- **Prefer formats agents can read:** PDF, Markdown, plain text, or `.drawio`. Text in images is harder to cite.
- **Tell the agent** in the next session that something was added, so the index is updated.

## Index

| File | What it is | Current? | Notes |
|---|---|---|---|
| `Froop_inital_architecture.pdf` | RFC 0001: Froop. The course final project proposal, last updated 2026-11-03. It covers the summary, goals, AWS services, data model, API, IaC, CI/CD, security, observability, demo plan, risks, and alternatives. | Yes | The main source. The diagram on p.3 is an export of the `.drawio` file. |
| `architecture(2).drawio` | The source of the architecture diagram, with one lane per area. | Yes | Read the arrows and numbered steps from the image on RFC p.3. |
