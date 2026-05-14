# docs

This directory contains operational documentation for contributors and AI Coders
working on the make-it-so project.

It is distinct from the architecture and ADR directories, which document the
system design. This directory documents how to work in this repository.

---

## Contents

### Session Start

| File | Purpose |
|---|---|
| [session-start-prompt.md](session-start-prompt.md) | Opening prompt for the project owner — bootstraps a fresh AI Coder session |

Use this before the AI Coder has read any repository files.

---

### AI Coder Workflow

| File | Purpose |
|---|---|
| [ai-coder-workflow.md](ai-coder-workflow.md) | Contribution model: session lifecycle, branch model, PR workflow, parallel sessions |

This is the architectural model for how AI Coders work in this repository.
Read `AGENTS.MD` at the repo root for the operational rules that implement it.

---

### AI Coder Sprint Loop

| File | Purpose |
|---|---|
| [ai-coder-sprint-loop.md](ai-coder-sprint-loop.md) | Operational specification for orchestrated S-BL processing: G-BL/S-BL placement, GitHub Actions orchestration, Coder/QAT routing and independence, QAT evidence, Scope Guard follow-ups, Milestone release context, and three-layer ticket state |

Status: **elaborated** (see ADR-0012).

---

### Slash Commands

Templates and checklists for structured agent session management.

| File | Purpose |
|---|---|
| [slash-commands/handoff.md](slash-commands/handoff.md) | Template for writing a handoff note when ending a session |
| [slash-commands/pickup.md](slash-commands/pickup.md) | Checklist for picking up work from a previous agent or session |
| [slash-commands/pmc.md](slash-commands/pmc.md) | Post-Merge Cleanup: refreshes main and removes the merged local feature branch if safe, after the project owner merges a PR |

Use `/handoff` before ending any session with unfinished work.
Use `/pickup` at the start of any session that continues previous work.

---

### GitHub Labels

| File | Purpose |
|---|---|
| [github-labels.md](github-labels.md) | Label scheme mirroring the `[DOMAIN/CATEGORY]` marker system; includes setup commands |

See [ADR-0003](../adr/0003-domain-and-category-markers.md) for the full marker system definition.

---

### Programming Guides

| File | Purpose |
|---|---|
| [programming-guides/README.md](programming-guides/README.md) | Index for future programming language guides, toolchain strategy, secure coding guardrails, and AI Coder implementation guidance |
| [programming-guides/language-and-toolchain-strategy.md](programming-guides/language-and-toolchain-strategy.md) | Placeholder for language selection principles, AI Coder friendliness criteria, command surface expectations, and toolchain guardrails |

Status: placeholder / to be elaborated.

---

### Design System

| File | Purpose |
|---|---|
| [design-system/README.md](design-system/README.md) | Index for future UI style guide, design token concepts, component guidance, accessibility, platform mapping, and approval UX safety documentation |
| [design-system/ui-style-guide-and-approval-ux.md](design-system/ui-style-guide-and-approval-ux.md) | Placeholder for LCARS-inspired visual identity, interaction clarity, approval UX safety, UI states, accessibility, and platform mapping |

Status: placeholder / to be elaborated.

---

### Governance and Safety

| File | Purpose |
|---|---|
| [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md) | Public placeholder for repository safety principles, protected asset classes, canary/watcher/tripwire concepts, AI Coder safety expectations, and review gates |
| [governance/release-context.md](../governance/release-context.md) | Release context coordination: Milestone-based model across repos, relationship to S-BL and sprint loop, release build gate concepts |

Status: **release-context** elaborated for Milestones; **repository-safety** remains a high-level placeholder for concrete tripwires and CI details.
