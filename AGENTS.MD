# AGENTS.MD

Operational guidelines for AI Coders working on the make-it-so project.

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

## Docs & Communication Style

Documentation language: English only.

Writing style: concise and direct. No filler phrases.

Comments in code: only when the WHY is non-obvious. No what-comments.

File size: keep files under ~500 lines; split when exceeded.