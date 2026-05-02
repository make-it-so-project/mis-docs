# AGENTS.MD

Operational guidelines for AI Coders working on the make-it-so project.

AGENTS.MD is the authoritative guide for this repo. Ignore any CLAUDE.md.

Read `AI_CONTEXT.md` first for project architecture and domain context.

---

## Project Repositories

Three repositories exist under the [make-it-so-project](https://github.com/make-it-so-project) organization.

| Repository | Purpose |
|---|---|
| [mis-docs](https://github.com/make-it-so-project/mis-docs) | Architecture, decisions, governance — single source of truth (this repo) |
| [mis-showcase](https://github.com/make-it-so-project/mis-showcase) | Early PoC prototype; not a reference implementation |
| [mis-website](https://github.com/make-it-so-project/mis-website) | Public marketing site; TanStack Start on Cloudflare Workers |

**mis-website note:** This repo is bidirectionally synced with Lovable. Pushes to
`main` are picked up by Lovable automatically. Never edit `src/routeTree.gen.ts`
(auto-generated). Do not run destructive git operations in mis-website without
explicit user confirmation.

---

## Session Start

At the start of every session, before any other action, state one line:

> Current branch: `<branch>` — Task: `[DOMAIN/CATEGORY]`

Example: `Current branch: claude/control-plane-docs — Task: [DEV/ARCH]`

This confirms AGENTS.MD was read and orients any observer on scope.

If no branch or domain marker is known yet, run `/pickup` and ask the user
for the task marker before proceeding.

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