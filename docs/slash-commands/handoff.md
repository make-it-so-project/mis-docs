# /handoff

Use this command when ending a session and passing work to another agent or session.

Write a handoff note as a comment or message in the active thread, using the
structure below. Keep it short and machine-readable.

---

## Handoff Note Structure

```
## Handoff

**Branch:** <branch-name>
**Domain:** [DOMAIN/CATEGORY]

### Done
- <what was completed in this session>

### In Progress
- <what is partially done; include file and line if relevant>

### Next
- <what should be picked up next>

### Blockers
- <anything the next agent needs to know or resolve first>
  (none if empty)

### Context
- <any decision, convention, or constraint that is not obvious from the code>
  (none if empty)
```

---

## Rules

- Always include the branch name so the next agent can orient immediately.
- Always include the domain marker so scope is clear.
- "Done" means committed and pushed – not just locally complete.
- "Next" should be actionable, not vague.
- Read `AGENTS.MD` and `AI_CONTEXT.md` before writing the handoff to ensure
  the note reflects current project conventions.
