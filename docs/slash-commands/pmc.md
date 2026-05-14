# /pmc — Post-Merge Cleanup

## Purpose

Clean up local repository state after the project owner has merged a PR.

---

## When to Use

The project owner says **"PMC"** or **"/pmc"** after merging a PR.

This is the shorthand for:

> "Merge is done — refresh local main and remove the merged local feature branch if safe."

PMC is post-merge only. It does not authorize merging, force deletion, force pushing,
rewriting history, or bypassing branch protection.

---

## Steps

1. Run `git status` — verify the working tree is clean and no unexpected changes exist.
2. Note the current feature branch name.
3. Run `git fetch origin` — update remote-tracking refs.
4. Run `git checkout main` — switch to main.
5. Run `git pull origin main` — fast-forward or pull main from origin.
6. Delete the local feature branch only if it is safe (see Safety Stop Conditions below).
7. Prune stale remote-tracking refs if appropriate (`git remote prune origin` or
   `git fetch --prune`).
8. Report: current branch, removed branch name, whether main is up to date, and
   working tree status.

---

## Safety Stop Conditions

Stop and report instead of deleting anything if any of the following apply:

- uncommitted local changes in the working tree
- unpushed local commits on the feature branch
- branch not confirmed as merged and not clearly safe to delete
- detached HEAD state
- uncertain remote or main state
- failed fetch or pull
- any git error or unexpected git output

When in doubt, report the current state and ask the project owner before proceeding.

---

## Expected Output

Report the following after a successful PMC:

- **Current branch:** `main`
- **Main:** up to date with `origin/main`
- **Removed local branch:** `<branch-name>` (or "not removed — see note" with reason)
- **Working tree:** clean

---

## What PMC Does Not Do

- PMC does not merge PRs — only the project owner merges PRs.
- PMC does not force-delete branches.
- PMC does not push to any branch.
- PMC does not rewrite history.
- PMC does not bypass branch protection.

---

## Related Documents

- [AGENTS.MD](../../AGENTS.MD) — Post-Merge Cleanup (PMC) operational rule
- [docs/ai-coder-workflow.md](../ai-coder-workflow.md) — PMC workflow reference
- [docs/slash-commands/handoff.md](handoff.md) — session handoff checklist
- [docs/slash-commands/pickup.md](pickup.md) — session pickup checklist
