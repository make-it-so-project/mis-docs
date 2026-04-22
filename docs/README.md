# docs

This directory contains operational documentation for contributors and AI Coders
working on the make-it-so project.

It is distinct from the architecture and ADR directories, which document the
system design. This directory documents how to work in this repository.

---

## Contents

### Slash Commands

Templates and checklists for structured agent session management.

| File | Purpose |
|---|---|
| [slash-commands/handoff.md](slash-commands/handoff.md) | Template for writing a handoff note when ending a session |
| [slash-commands/pickup.md](slash-commands/pickup.md) | Checklist for picking up work from a previous agent or session |

Use `/handoff` before ending any session with unfinished work.
Use `/pickup` at the start of any session that continues previous work.

---

### GitHub Labels

| File | Purpose |
|---|---|
| [github-labels.md](github-labels.md) | Label scheme mirroring the `[DOMAIN/CATEGORY]` marker system; includes setup commands |

See [ADR-0003](../adr/0003-domain-and-category-markers.md) for the full marker system definition.
