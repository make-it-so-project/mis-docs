# ADR-0001 Terminology: AI Coder vs Agent

## Status
Accepted

---

## Context

The term "agent" is widely used in AI systems.

Within the make-it-so project two different meanings could lead to confusion:

1. AI systems used to develop the codebase
2. operational entities acting inside the runtime platform

Without clear terminology this ambiguity would create problems in:

- architecture documentation
- governance rules
- development workflows
- AI prompts

---

## Decision

The following terminology is adopted.

### Agent

Used exclusively for runtime entities operating within the make-it-so platform.

### AI Coder

Used for AI-based development systems responsible for modifying the codebase.

---

## Consequences

This terminology separation ensures clarity across:

- architecture documentation
- governance documentation
- backlog definitions
- AI development workflows
