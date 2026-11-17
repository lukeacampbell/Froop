# Revised RFC AGENTS.md

This folder holds the **current** version of the Froop architecture proposal. It begins as the submitted RFC 0001 (in `architecture/inputs/`, which never changes) and takes in each decision once it's Accepted. Anyone who wants to know what Froop's architecture is *now* reads this folder. Anyone who wants to know *why* reads the Decisions in each area.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here.

## Files

| File | Holds |
|---|---|
| `AGENTS.md` | This guide |
| `Froop_revised_architecture.md` | The revised RFC, with the same sections as the original, plus a Change Log |
| `architecture.drawio` | The revised diagram source |
| `architecture.png` | An image exported from `architecture.drawio`, embedded in the revised RFC |

## Working Agreement

**What goes in:**
- **Only Accepted decisions.** `Proposed` decisions and open questions never change this document.
- **Pending text is marked, not changed.** When an open question will change some text, leave the text as it is and mark it *(pending XX-Q00)*.
- **No new design.** This document reports decisions made in the areas. If something here needs deciding, record an Open Question in the right area.

**How to update:**
1. **When to update:** in the same change that moves a decision to `Accepted` (or soon after), update every section the decision affects.
2. **Keep the original wording** wherever a decision doesn't change it. Change only what the decision requires.
3. **Log it:** add a row to the Change Log with the date, the decision ID, and the sections changed.
4. **Superseded decisions:** when an Accepted decision is superseded, apply the new one and log it the same way.
5. **Keep the diagram in step:** if a decision changes a component, a label, or an arrow, update `architecture.drawio` and re-export `architecture.png` in the same change. Never edit the PNG without the `.drawio`.
6. **Bump `Last updated`** in the header table.

**Who may update:** the agent of any area owner, for decisions in that owner's areas. Changes to shared sections (Team, Summary, Goals, Demo Plan) need the group, so record a System Open Question for those.

## Exporting the Diagram

If draw.io is installed, use File → Export as → PNG, or run `draw.io -x -f png -o architecture.png architecture.drawio`.

Without it, use the draw.io viewer in a headless browser:
1. Write a small HTML page that embeds the `.drawio` XML in a `<div class="mxgraph" data-mxgraph="…">` and loads `https://viewer.diagrams.net/js/viewer-static.min.js`.
2. Screenshot it with headless Edge or Chrome: `--headless=new --virtual-time-budget=15000 --window-size=1680,1140 --screenshot=architecture.png`.
3. Check the image before you commit it.
