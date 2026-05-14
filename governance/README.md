# Governance Model

This document defines the development governance for the make-it-so project.

The governance model ensures that development work remains structured, traceable, and controlled.

---

# Development Workspaces

Two workspaces exist for development activities.

---

## Global Backlog (G-BL)

The Global Backlog is used for:

- idea collection
- request refinement
- architectural analysis
- decomposition of larger tasks

Items in the Global Backlog are not executable tasks.

---

## Sprint Backlog (S-BL)

The Sprint Backlog contains approved development tasks.

Only tasks within the Sprint Backlog may trigger implementation work.

---

# Approval Authority

Only the human project owner (CEO role) may move tasks from:

G-BL → S-BL

This transition represents formal approval for implementation.

---

# Execution Rules

AI Coders may only perform development work on tasks contained in the Sprint Backlog.

Tasks outside the Sprint Backlog must not trigger implementation.

---

# Trust Model

Natural language produced by AI Coders is considered untrusted input.

Operational decisions must rely on structured artifacts such as:

- Sprint Backlog tasks
- Architecture Decision Records
- repository policies
- governance rules

---

# AI Coder Operational Rules

The governance model defines what AI Coders may work on and who approves it.

For how AI Coders execute work within the git workflow — branch creation, commits,
pull requests, session handoffs — see:

- `docs/ai-coder-workflow.md` — contribution model and branch lifecycle
- `AGENTS.MD` — operational rules and git safety guidelines

---

## AI Coder Development Sprint Loop

Future sprint execution may be coordinated through an AI Coder Development Sprint Loop.
See [`docs/ai-coder-sprint-loop.md`](../docs/ai-coder-sprint-loop.md) for the placeholder model.

The sprint loop must still respect the governance model:

- only the project owner moves work from G-BL to S-BL
- only S-BL tasks may trigger implementation
- every S-BL ticket requires a Coder AI Agent and an independent QSer AI Agent
- only the project owner approves and merges PRs
