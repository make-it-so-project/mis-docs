# /pickup

Use this command when starting a session that continues work from another agent or session.

Follow the steps below in order before making any changes.

---

## Pickup Checklist

1. **Read AGENTS.MD** – operational rules, commit format, git safety, docs style.

2. **Read AI_CONTEXT.md** – project architecture, domain model, key concepts.

3. **Check git status**
   ```
   git status
   git log --oneline -5
   ```

4. **Find the handoff note** – look in the most recent PR comment, issue thread,
   or session log for a `## Handoff` block.

5. **Read the handoff** – orient on Done / In Progress / Next / Blockers / Context.

6. **Read referenced files** – if the handoff mentions specific files or ADRs,
   read them before touching anything.

7. **Confirm scope** – identify the domain marker (`[DOMAIN/CATEGORY]`) and
   stay within that scope. If unclear, ask before proceeding.

8. **Resume** – pick up from "In Progress" or "Next", whichever is more urgent.

---

## Rules

- Do not skip the checklist, even for small tasks.
- Do not assume the previous agent's work is complete without checking git.
- If no handoff note exists, treat the current branch state as ground truth
  and ask the user for context before starting.
