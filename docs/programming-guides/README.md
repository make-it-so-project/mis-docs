# Programming Guides

## Purpose

This directory will contain programming language guidance, toolchain rules, secure coding
guardrails, AI Coder development guardrails, and build/test/lint expectations for
make-it-so implementation work.

---

## Status

**Placeholder / to be elaborated.**

The directory captures the topic and initial structure. It does not yet define final
language-specific guides. It does not replace `AGENTS.MD`, `docs/ai-coder-workflow.md`,
`governance/release-context.md`, or repository safety rules. It does not contain
implementation code.

---

## Contents

| File | Purpose |
|---|---|
| [language-and-toolchain-strategy.md](language-and-toolchain-strategy.md) | Placeholder for language selection principles, AI Coder friendliness criteria, toolchain guardrails, and future guide structure |

---

## Planned Future Guides

The following files are not yet created. They are listed here to capture planned scope.

| File | Purpose |
|---|---|
| typescript.md | TypeScript conventions for web, PWA, frontend, and API client work |
| go.md | Go conventions for CLIs, backend tooling, small services, and MCP-adjacent components |
| swift.md | Swift conventions for Apple-platform native clients |
| android.md | Android-native guidance if Kotlin or another native Android approach is selected |
| secure-coding.md | Secure coding requirements across languages |
| ai-coder-guardrails.md | Coding rules and validation expectations optimized for AI Coder work |

---

## Related Documents

- [docs/ai-coder-workflow.md](../ai-coder-workflow.md) — contribution model and branch lifecycle
- [docs/ai-coder-sprint-loop.md](../ai-coder-sprint-loop.md) — Coder/QAT pair workflow and release build gate
- [governance/release-context.md](../../governance/release-context.md) — release coordination model
- [governance/repository-safety-and-canaries.md](../../governance/repository-safety-and-canaries.md) — repository safety principles and protected asset classes
- [AGENTS.MD](../../AGENTS.MD) — operational rules: commits, branches, git safety, slash commands
- [AI_CONTEXT.md](../../AI_CONTEXT.md) — project architecture and domain model
- [adr/0003-domain-and-category-markers.md](../../adr/0003-domain-and-category-markers.md) — domain and category marker system
