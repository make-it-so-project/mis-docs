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

## Branch Workflow

The `main` branch is protected. All changes to `main` must go through a
pull request. Never push directly to `main`.

If a push to `main` fails with a branch protection error, create a feature
branch and open a PR. Do not attempt to bypass the protection.

### Feature Branches

Always work on a feature branch. Create one at the start of every session
that involves commits.

```bash
git checkout main && git pull origin main
git checkout -b <type>/<short-description>
```

Branch naming: `<type>/<short-description>`

| Prefix | Use for |
|---|---|
| `docs/` | Documentation changes, ADRs, architecture docs |
| `feat/` | New features or capabilities |
| `fix/` | Bug fixes, corrections, consistency patches |
| `chore/` | Housekeeping, tooling, non-content changes |

Examples: `docs/account-recovery-qs`, `feat/session-connect-api`, `fix/user-id-consistency`

### Pull Requests

Open a PR when work on a task is complete.

PR title format: `[DOMAIN/CATEGORY] <type>: <short description>`

Example: `[DEV/ARCH] docs: add AI Coder workflow model`

PR body must include: what changed and why, scope boundary (what was NOT
changed), links to relevant ADRs or architecture documents, and a test plan.

Only the project owner merges PRs. AI Coders open PRs; they do not merge them.

See `docs/ai-coder-workflow.md` for the full conceptual model.

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

---

## Communication Shorthands

The following shorthands may appear in task instructions from the project owner:

| Shorthand | Meaning |
|---|---|
| C&P | Copy & Paste — deliver the result as a self-contained block ready for direct copying |