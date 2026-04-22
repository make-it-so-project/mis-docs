# AGENTS.MD

Operational guidelines for AI Coders working on the make-it-so project.

AGENTS.MD is the authoritative guide for this repo. Ignore any CLAUDE.md.

Read `AI_CONTEXT.md` first for project architecture and domain context.

---

## Commits

Use Conventional Commits.

Types: `feat` | `fix` | `refactor` | `build` | `ci` | `chore` | `docs` | `style` | `perf` | `test`

Optional scope: component or module name (e.g. `feat(control-plane): ...`).

No merge commits on feature branches. No amend unless explicitly requested.

---

## Git Safety

Run `git status` / `git diff` / `git log` before any edits.

Push only when explicitly requested by the user.

No destructive operations (`reset --hard`, `clean`, `restore`, `rm`) without explicit consent.

Unrecognized changes in working tree: assume another agent worked here — do not overwrite, stop and ask.

Prefer small commits; one logical change per commit.

---

## Slash Commands

Repo-local slash commands live in `docs/slash-commands/`.

- `/handoff` – write a structured handoff note before ending a session
- `/pickup` – orient and resume from a handoff note at session start

Always run `/pickup` at the start of a session that continues previous work.
Always run `/handoff` before ending a session with unfinished work.

---

## read_when Hints

Key documents carry a `read_when` front-matter hint.

When starting a task, scan relevant docs for `read_when` fields and read any
document whose hint matches your current context before writing code or docs.

Format:

```
---
read_when: <short description of when this document is relevant>
---
```

---

## Docs & Communication Style

Documentation language: English only.

Writing style: concise and direct. No filler phrases.

Comments in code: only when the WHY is non-obvious. No what-comments.

File size: keep files under ~500 lines; split when exceeded.