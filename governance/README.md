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
