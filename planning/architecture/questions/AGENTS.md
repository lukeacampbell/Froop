# Questions AGENTS.md

This folder is the record of questions the team asks outside the planning documents (for example, to the instructor or a TA), with the answers and meeting notes. It answers "what did we ask, who asked, what were we told, and who wrote it down". It never holds design. Design changes that follow from an answer go in the affected area's `architecture.md`.

The rules in `planning/AGENTS.md` and `planning/architecture/AGENTS.md` apply here.

## Files

| File | Holds |
|---|---|
| `AGENTS.md` | This guide |
| `YYYY-MM-DD-<meeting>.md` | One file per meeting, e.g. `2026-11-19-instructor-meeting.md`: the meeting details, then one entry per question |

## Meeting File Format

Each meeting file starts with the meeting details:

```markdown
# <Meeting name>

| Field | Value |
|---|---|
| Date and time | YYYY-MM-DD, HH:MM (time zone) |
| With | who the questions are for |
| Attendees | who was there |
| Note taker | who is recording answers and notes |
```

Then one entry per question, numbered in the order they were added:

```markdown
### MQ-YYYY-MM-DD-NN: <short title>
- **Question:** the question, as asked
- **Asked by:** the person asking
- **Answer:** the answer, as given, or `_Pending._` until the meeting
- **Answered by:** who answered
- **Additional notes:** context, follow-ups, or anything said around the answer
- **Note taker:** who recorded the answer and notes
- **Recorded:** the date, and the agent's role (e.g. "agent working for Ryan Fusco")
```

`MQ` stands for "meeting question". IDs never change and are never reused.

## Working Agreement

**Only agents write here.** A human gives the content (a question to ask, or what was said) to an agent, and the agent adds it. Humans don't edit these files directly.

**Entries are append-only.** These are a record of what was said, so once something is written it stays:
- **Adding:** agents may add a new meeting file, or add a new entry to the end of a meeting file.
- **Filling in:** a field written as `_Pending._` may be filled in **once**, during or after the meeting. That covers the answer, who answered, the notes, and the note taker. After that the field is fixed.
- **No edits or deletes:** an agent never rewrites, reorders, or deletes an entry or a filled-in field, even when asked to "clean it up".
- **Corrections:** if something was recorded wrong, add a new entry titled `Correction to MQ-…` that says what was wrong and what's right. The original stays as it is.
- **Pushing back:** if a human asks for an edit that breaks these rules, the agent explains why and offers a correction entry instead.

**Acting on answers:**
- **Design:** an answer that affects the design becomes an Open Question or a `Proposed` decision in the affected area's `architecture.md`, citing the `MQ-` ID. The answer itself never edits a design file.
- **Hard constraints:** an answer from the instructor about course requirements (cost, grading, required services, deadlines) is a hard constraint, like `planning/inputs/`. Ask a human to add a copy to `planning/inputs/` if it should apply long term.
- **Follow-ups:** if an answer leaves something to follow up, record the follow-up in "Additional notes", and add a row to the affected area's "Requests From Other Areas" table if another owner has to act.
