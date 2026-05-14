---
read_when: starting a new session, creating a branch, opening a PR, or orienting as a new AI Coder on this project
---

# AI Coder Workflow

## Purpose

This document defines the contribution model for AI Coders working on the
make-it-so project.

It describes how AI Coder sessions map to git branches and pull requests,
how tasks move from the backlog to the repository, and how multiple AI Coders
can work in parallel without interference.

This is the architectural model. Operational rules that implement it are in
`AGENTS.MD`.

---

## The Contribution Cycle

Every change to the make-it-so repository follows this cycle:

```
S-BL Task approved by project owner
  │
  │ AI Coder reads task, opens feature branch, reads relevant docs
  ▼
Feature Branch (isolated work unit)
  │
  │ AI Coder implements, commits incrementally
  ▼
Pull Request opened by AI Coder
  │
  │ Project owner reviews and approves
  ▼
Merged to main
```

Only S-BL tasks may trigger implementation work. AI Coders do not self-select
tasks from the G-BL and do not implement work that has not been approved for the
Sprint Backlog. See `governance/README.md` for the backlog and approval model.

A task in the S-BL maps to exactly one feature branch. Multiple closely related
changes may be committed to the same branch when they belong to a single coherent
S-BL task. Changes from separate S-BL tasks must not share a branch.

---

## Session Model

An AI Coder session is a bounded unit of work corresponding to one contiguous
period of AI Coder activity on a task.

A session:

- begins with reading `AGENTS.MD`, identifying the task and its domain marker,
  and preparing the feature branch
- operates on one feature branch throughout
- produces one or more commits
- ends with a PR opened (task complete) or a `/handoff` note written
  (task incomplete, to be continued in a future session)

A single task may span multiple sessions if it is handed off and resumed.
All sessions for the same task use the same feature branch.

### Session Start

```
1. Read AGENTS.MD and AI_CONTEXT.md
2. Identify the task and its [DOMAIN/CATEGORY] marker
3. Run /pickup if continuing previous work on an existing branch
4. Inspect git status and recent log
5. Read any read_when-tagged documents relevant to the task
6. Confirm scope — if unclear, ask the project owner before proceeding
```

### Session End

For a concrete S-BL task, the normal successful session end is:

```
1. commit all completed work on the feature branch
2. push the feature branch
3. open a PR against main
4. report the PR link, a concise summary, and any test/validation notes
```

The AI Coder does not ask the project owner for separate approval to perform these
steps. The PR is the expected review gate — opening it does not bypass review.
Only the project owner merges PRs.

If the environment cannot push or open a PR, use the Push / PR Failure Fallback
(see below and in `AGENTS.MD`).

```
If task is incomplete:
  → commit all work done so far
  → write a /handoff note so the next session can resume without information loss
  → do not leave uncommitted work in the branch
```

### Post-Merge Cleanup (PMC)

Post-Merge Cleanup (PMC) is a local cleanup step after the project owner has merged a PR.

When instructed with **"PMC"** or **"/pmc"**, the AI Coder:

- refreshes local `main` from `origin/main`
- removes the merged local feature branch if safe
- prunes stale remote-tracking refs if appropriate
- reports the current branch and clean working tree status

If the local branch has unmerged commits, local changes, or unclear state, the AI Coder
must stop and report instead of deleting anything.

PMC does not replace the PR workflow. It does not allow AI Coders to merge PRs.

See `docs/slash-commands/pmc.md` for the full checklist and `AGENTS.MD` for the
operational rule.

---

## Branch Model

### main is Protected

The `main` branch is the source of truth for the repository. It is always
in a coherent, reviewable state.

`main` is protected by GitHub branch protection rules:

- AI Coders MUST NOT push directly to `main`
- All changes to `main` MUST go through a pull request
- This protection is enforced by GitHub and cannot be bypassed

If a push to `main` fails with a branch protection error, the correct response
is to create a feature branch, push the commit there, and open a PR.
Do not attempt to bypass or work around this protection.

### Feature Branches are Ephemeral

Each feature branch is created for a specific task and is deleted after the
PR is merged. Feature branches are not long-lived shared state.

**Branch naming convention:** `<type>/<short-description>`

| Prefix | Use for |
|---|---|
| `docs/` | Documentation changes, architecture docs, ADRs |
| `feat/` | New features or capabilities |
| `fix/` | Bug fixes, corrections, consistency patches |
| `chore/` | Housekeeping, tooling, non-content changes |

Names are lowercase with hyphens. No spaces, no special characters.

Examples:

```
docs/account-recovery-qs
docs/ai-coder-workflow-and-branch-guidelines
feat/session-connect-api
fix/user-id-consistency
chore/cleanup-adr-links
```

### Creating a Branch

```bash
git checkout main
git pull origin main
git checkout -b <type>/<short-description>
```

Always create the branch from the latest `main`. Do not branch from another
feature branch unless the dependency is explicitly required by the task.

---

## Pull Requests

Every merge to `main` goes through a pull request. The PR is both the
review gate and the permanent audit record for the change.

### PR Title

The PR title carries the domain marker and a short description:

```
[DOMAIN/CATEGORY] <type>: <short description>
```

Multiple markers are allowed when a change spans domains or categories.

Examples:

```
[DEV/ARCH] docs: add AI Coder workflow model
[RUN/SEC][RUN/ARCH] feat: define client revocation model
[DEV/OPS] chore: update github label scheme
[RUN/ARCH][RUN/SEC][RUN/ADR] docs: account recovery model
```

### PR Body

The PR body must include:

- **What changed and why** — the why matters more than the what
- **Scope boundary** — what was explicitly NOT changed and why
- **References** — links to relevant ADRs or architecture documents
- **Test plan** — a checklist of acceptance criteria to verify the change
- **Protected Asset Class evaluation** — required when the task introduces a new
  artifact area (see below)

#### Protected Asset Class Evaluation in PR Bodies

When a task introduces a new documentation area, configuration area, workflow area,
implementation area, release artifact type, or other artifact class, the PR body must
state one of:

- `Protected Asset Class: added/updated — <summary>` — if a new class was added or an
  existing class was expanded in `governance/repository-safety-and-canaries.md`
- `Protected Asset Class: covered by existing — <which class and why>` — if an existing
  class already covers the new area
- `Protected Asset Class: none needed — <reason>` — if the area does not create
  protected asset concerns

An absent evaluation is a gap. QAT must flag missing evaluations for new artifact areas.

See `governance/repository-safety-and-canaries.md` for evaluation criteria and the full
list of protected asset classes.

### Who Merges

Only the project owner approves and merges pull requests.

AI Coders open PRs. They do not merge them, close them, or request review
from other AI Coders.

---

## Parallel AI Coder Sessions

Multiple AI Coder sessions may be active concurrently on the same repository,
each working on a separate feature branch. This is safe because:

- branches are isolated — each session writes only to its own branch
- there is no shared mutable state between feature branches
- merge conflicts, if they arise, are resolved by the project owner at PR merge time

An AI Coder session is unaware of other active sessions. If a session
encounters files, branches, or working tree state that it did not create,
it MUST stop and report to the project owner rather than assuming those
changes are safe to overwrite.

---

## Audit Model

The permanent record of AI Coder work consists of:

- committed code and documentation (in git history)
- pull request descriptions (in the GitHub PR thread)
- ADR decision records (in `adr/`)

AI Coder natural language — comments, summaries, explanations in chat — is
informational only. It is not an authoritative record and does not substitute
for a commit or a PR.

The `/handoff` note is a communication artifact for session continuity.
It does not replace a PR or a commit.

See `governance/README.md` for the project trust model.

---

## Push / PR Failure Fallback

If a push or PR cannot be completed — due to a tool error, network failure,
permission error, or unexpected git state — the AI Coder MUST:

1. Report the failure clearly. Do not claim the push or PR succeeded.
2. Preserve all committed work. Do not reset or discard local commits.
3. Report current state: `git status`, `git log --oneline -5`, and
   `git diff origin/main...HEAD`.
4. Generate a patch with `git format-patch origin/main --stdout` and
   present the output so the project owner can transfer the work manually.

This rule applies regardless of which tool or environment is being used.
The permanent audit record requires an actual PR on GitHub — a chat summary
is not a substitute.

See `AGENTS.MD` for the operational details of this fallback.

---

## Related Documents

- `AGENTS.MD` — operational rules: commits, branch commands, git safety, slash commands
- `governance/README.md` — backlog model (G-BL → S-BL) and approval authority
- `AI_CONTEXT.md` — project architecture, domain model, key concepts
- `docs/slash-commands/handoff.md` — handoff note structure and rules
- `docs/slash-commands/pickup.md` — session pickup checklist
- `adr/0003-domain-and-category-markers.md` — domain and category marker system
- `docs/github-labels.md` — GitHub label scheme matching the marker system
