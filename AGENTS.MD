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

Example: `Current branch: docs/control-plane-docs — Task: [DEV/ARCH]`

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
changed), links to relevant ADRs or architecture documents, a test plan, and
for tasks introducing new artifact areas, a Protected Asset Class evaluation
note (see Protected Asset Classes section).

Only the project owner merges PRs. AI Coders open PRs; they do not merge them.

See `docs/ai-coder-workflow.md` for the full conceptual model.

---

## Default Completion Behavior

For a concrete S-BL task, completion includes committing the completed work, pushing the feature branch, and opening a pull request against `main`.

Do not ask the project owner whether to commit or open a PR when the task is complete and no instruction says otherwise. The PR is the expected review gate.

Ask or stop only if:
- the scope is unclear
- the working tree contains unrecognized changes
- checks fail in a way that needs a decision
- the user explicitly asked not to commit, push, or open a PR
- the environment cannot push or open a PR

Only the project owner merges PRs.

---

## Protected Asset Classes

When a task changes governance, security, workflow, CI/CD, dependency, architecture
decision, or release coordination files, call out the change explicitly in the PR body
scope boundary section.

When a task introduces a new documentation area, configuration area, workflow area,
implementation area, release artifact type, or other artifact class, evaluate whether
it creates or changes a Protected Asset Class.

A Protected Asset Class update is required when the new area affects governance,
security, workflow, CI/CD, dependencies, architecture decisions, release coordination,
deployment, operations, trusted approval behavior, user-facing safety, or other
safety-relevant behavior.

If the new area should be protected: update `governance/repository-safety-and-canaries.md`
with a concise entry.

If the new area is already covered by existing protected asset classes or does not need
protected status: state that explicitly in the PR body.

**PR body requirement for new artifact areas:** Include one of:
- `Protected Asset Class: added/updated — <summary>`
- `Protected Asset Class: covered by existing — <which class and why>`
- `Protected Asset Class: none needed — <reason>`

See `governance/repository-safety-and-canaries.md` for the full list of protected asset
classes and AI Coder safety expectations.

---

## Post-Merge Cleanup (PMC)

When the project owner says **"PMC"** or **"/pmc"**, treat it as:

> "Merge is done — refresh local main and remove the local feature branch if safe."

Steps:

1. Verify working tree state.
2. Note the current feature branch name.
3. Fetch origin.
4. Switch to `main` and pull from `origin/main`.
5. Delete the local feature branch only if it is merged or otherwise safe.
6. Prune stale remote-tracking refs if appropriate.
7. Report current branch, removed branch, and clean working tree status.

**Safety rule:** If uncommitted local changes, unpushed commits, unmerged commits, unknown branch state, or any other unsafe condition exists — stop and report instead of deleting anything.

**Scope:** PMC is a post-merge local cleanup only. It does not authorize merging, force deletion, force pushing, rewriting history, or bypassing branch protection. Only the project owner merges PRs.

See `docs/slash-commands/pmc.md` for the full checklist.

---

## Git Safety

Run `git status` / `git diff` / `git log` before any edits.

Do not push directly to `main`. Push the feature branch as part of default completion for concrete S-BL tasks, unless the user explicitly requested no push, the scope is unclear, the branch state is unsafe, or the environment cannot push safely.

No destructive operations (`reset --hard`, `clean`, `restore`, `rm`) without explicit consent.

Unrecognized changes in working tree: assume another agent worked here — do not overwrite, stop and ask.

Prefer small commits; one logical change per commit.

---

## Push / PR Failure Fallback

If pushing a branch or opening a PR fails for any reason (tool error,
network issue, permission error, or unexpected git state):

1. **Do not claim the push or PR succeeded.** Report the failure clearly.
2. **Preserve all work locally.** Do not reset or discard uncommitted or
   unpushed commits.
3. **Run and report** `git status`, `git log --oneline -5`, and
   `git diff origin/main...HEAD` so the project owner can see exactly
   what is committed and what is not.
4. **Generate a patch** with `git format-patch origin/main --stdout` and
   present the output so the project owner can apply or transfer the work
   manually if needed.

Never silently skip the push or PR step. If it cannot be completed, stop
and report rather than continuing as if the task is done.

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