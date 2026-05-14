---
read_when: selecting programming languages, defining toolchains, adding build/test/lint commands, creating programming guides, planning AI Coder implementation work, or changing dependency strategy
---

# Language and Toolchain Strategy

## Purpose

This document captures the initial strategy for programming languages, toolchains,
programming guides, and AI Coder guardrails for make-it-so.

The project should prefer language and tooling choices that reduce ambiguity, enable fast
validation, support secure defaults, and are reliable for both human developers and AI
Coders.

---

## Status

**Placeholder / to be elaborated.**

This document captures initial direction and criteria. It is not yet an Architecture
Decision Record. It does not define final implementation repositories, package managers,
frameworks, or deployment targets. It does not add implementation code. It should later
inform one or more ADRs when decisions become binding.

---

## Scope

This document covers:

- candidate programming languages and their rationale
- language selection criteria
- AI Coder friendliness requirements
- standard command surface expectations
- secure coding guardrails
- dependency and supply-chain considerations
- repo and module boundary principles
- cloud and container compatibility
- relationship to release context and sprint loop
- open questions for later elaboration

---

## Non-Goals

This document does not define:

- implementation code
- final framework selection
- final backend hosting model
- final mobile app architecture
- final package manager selection
- CI/CD pipeline implementation
- release process implementation
- a new ADR
- replacements for repository safety or governance rules

---

## Core Principle

Technology choices must be optimized for maintainability, security, testability, and AI
Coder reliability — not only for developer preference.

Required properties of each language and toolchain choice:

- Prefer clear type systems that surface errors early.
- Prefer fast lint, test, and format feedback loops.
- Prefer simple project structures with clear module boundaries.
- Prefer reproducible execution in local and cloud/container environments.
- Prefer boring, well-supported tools over clever or niche frameworks.
- Minimize unnecessary polyglot complexity.
- Every supported language must have standard commands for format, lint, test, typecheck or
  compile equivalent, and build.
- Every supported language must have secure coding guidance before production use.

---

## Candidate Language Direction

The following table captures candidate direction. These are not final binding decisions.
Each choice should be confirmed through an ADR when implementation begins.

| Language | Candidate Use | Rationale | Status |
|---|---|---|---|
| TypeScript | WebApp/PWA, website, frontend, API/client SDKs, Cloudflare-oriented code where appropriate | Strong ecosystem, type checking, web-native, AI Coder friendly, good lint/test tooling | Candidate default for web |
| Go | CLIs, backend tooling, small services, MCP-adjacent tools, simple server-side components | Simple type system, fast builds and tests, easy linting, good fit for AI Coders and operational tooling | Candidate default for tools/services |
| Swift | iOS/macOS native clients and Apple-platform UI | Native platform integration; good for secure device capabilities, biometrics, and iOS/macOS UI | Candidate for Apple native |
| Kotlin | Android native client, if implemented natively | Native Android ecosystem and platform integration | Candidate to evaluate |
| Python | Prototyping, scripts, data migration, tooling only where justified | Useful but can drift without strict typing and linting; should not become a default production backend without an explicit decision | Limited candidate |

Additional languages may be proposed but require an explicit reason, documented build/test/
lint rules, secure coding guidance, and AI Coder guardrails before being adopted.

---

## AI Coder Friendliness Criteria

Each language and toolchain should satisfy the following criteria before being approved for
AI Coder implementation work:

- **Simple build commands** — deterministic, single-command build from a clean checkout
- **Deterministic formatting** — a standard formatter that produces stable, reviewable diffs
- **Standard linting** — lint rules that are checked as part of normal development
- **Fast unit tests** — unit tests that run in seconds without external dependencies
- **Clear compiler and typechecker errors** — error messages that are interpretable without IDE context
- **Small files and clear module boundaries** — no large monolithic files that obscure change scope
- **Explicit dependency management** — dependencies declared in a manifest, not implied
- **No hidden IDE-only build steps** — all required steps runnable from the command line
- **Cloud/container environment compatibility** — build and test runnable in a standard container or CI environment
- **Easy local validation** — AI Coders can verify their changes without GUI-only tooling
- **Clear error output for iterative correction** — errors are machine-readable and actionable

AI Coders should not be forced into toolchains that require opaque local IDE state, manual
GUI steps, or undocumented environment assumptions. If a toolchain requires such steps, it
is not ready for AI Coder use until the requirements are documented and automated.

---

## Standard Command Surface

Each implementation repository or package should document a standard set of commands. The
exact commands vary per language and toolchain; the categories below are the expected
minimum.

| Category | Purpose |
|---|---|
| setup | Install or verify dependencies |
| format | Apply deterministic code formatting |
| lint | Run static analysis and style checks |
| typecheck / compile | Verify types or compile without producing a final artifact |
| test | Run unit tests |
| build | Produce the build artifact |
| run locally | Start the application or tool locally |
| check | Run all required local validation steps in one command where possible |
| security/dependency check | Run available dependency or vulnerability checks where tooling exists |

These categories are conceptual. Final commands are defined per language guide once those
guides are written. Do not create scripts for these commands in this placeholder document.

---

## Secure Coding Guardrails

The following guardrails apply across all languages before production use. Language-specific
expansions will be captured in future per-language guides.

- Never commit secrets, credentials, tokens, private keys, or recovery codes to the repository.
- Use environment variables or managed secret stores for secrets; never hardcode them.
- Validate all external input at system boundaries before using it.
- Avoid logging credentials, tokens, private keys, recovery codes, or approval payload contents.
- Cryptographic code must use reviewed libraries and be documented against existing architecture decisions; do not invent cryptographic logic.
- Authentication and approval logic must follow existing architecture documents and ADRs.
- Dependency additions must be justified and reviewed; avoid adding dependencies without a clear reason.
- Security-sensitive changes must be called out explicitly in PR bodies.
- Tests must not encode real credentials, operational canary values, or secret-like values.
- Build, dependency, and workflow files belong to protected asset classes; repository safety rules apply.

---

## Dependency and Supply-Chain Guardrails

- Minimize dependencies; prefer the standard library or well-known, actively maintained packages.
- Prefer well-known libraries over obscure packages with few maintainers.
- Pin or lock dependencies where appropriate to ensure reproducible builds.
- Review lockfile changes; they are part of the protected asset class set when they affect reproducibility or security.
- Avoid post-install scripts where possible; they are a common supply-chain attack vector.
- Document why new critical dependencies are needed.
- Security-sensitive dependency changes require QAT attention and project owner review.
- Dependency metadata files belong to protected asset classes when they affect build reproducibility or introduce security risk.

---

## Repo and Module Boundary Principles

- Keep modules small and cohesive; prefer many small files over a few large ones.
- Avoid mixing unrelated language ecosystems in one package.
- Do not introduce a new language into a repository without an explicit documented reason.
- Each new implementation component should document why its language and toolchain were selected.
- Shared contracts between components should be explicit and machine-checkable where possible. Possible future contract technologies include OpenAPI, JSON Schema, Protocol Buffers, or generated types. No final decision is made here.
- Do not decide final contract technology in this placeholder.

---

## Relationship to the AI Coder Sprint Loop

- S-BL implementation tickets should specify the relevant language and toolchain once implementation begins.
- Coder AI Agents should run all documented standard commands before opening a PR.
- QAT AI Agents should independently run or inspect validation commands where possible, and flag toolchain failures as follow-up tickets if they cannot be resolved within the original scope.
- Language or toolchain changes that affect protected asset classes must be called out explicitly in PR bodies.
- Toolchain or dependency changes require the same Coder/QAT review as implementation changes.

---

## Relationship to Release Context

- Language and toolchain decisions should be stable within a release context where possible.
- Toolchain upgrades or language additions during a release context should be explicit, reviewed, and associated with a specific release context.
- Release notes should call out major language or toolchain changes.
- The active release context should eventually identify which implementation targets and technology stacks are in scope for that release.

See [governance/release-context.md](../../governance/release-context.md).

---

## MVP Guidance

For early MVP implementation, prefer the smallest viable set of languages and toolchains.
Introducing all candidate languages at once creates toolchain complexity before the team
has formed stable practices.

**Suggested MVP posture:**
TypeScript and Go are likely early candidates for web, backend-adjacent tooling, and
operational components. Swift and Android-native choices should be introduced only when
native client work becomes part of the active release context and a confirmed platform
decision has been made.

This is guidance, not a binding decision. Final MVP language choices should be confirmed
through ADRs when implementation begins.

---

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Too many languages introduced too early | Require explicit justification for each new language; start with smallest viable set |
| AI Coders struggle with complex or opaque toolchains | Require documented setup/format/lint/test/build commands and cloud/container compatibility |
| IDE-only workflows block AI Coder validation | Prefer command-line-first build and test flows; document all required steps |
| Weak typing or inconsistent linting causes defects | Prefer typed languages and mandatory lint and typecheck commands |
| Insecure dependency introduced | Dependency review; lockfile review; QAT attention; project owner review |
| Secure coding rules diverge per language | Define cross-language guardrails first; add language-specific guides later |
| Mobile platform choices are made too early and then reversed | Keep Swift and Kotlin as candidates until native client work is active in the release context |

---

## Open Questions

The following questions are deferred to later elaboration:

- Which implementation repository receives the first backend prototype?
- Is the backend TypeScript, Go, or split by responsibility?
- Is Cloudflare deployment best served by TypeScript Workers, containerized services, or another approach?
- What is the minimal MVP language set?
- Which package managers are preferred for TypeScript and Go?
- What are the final standard commands for each language?
- Which CI checks are mandatory before PR merge?
- How are shared API contracts represented between components?
- When does Swift get introduced for iOS?
- What is the Android native strategy: Kotlin, cross-platform, or deferred?
- Which secure coding rules need language-specific expansion beyond the cross-language guardrails?
- Which language and toolchain decisions require formal ADRs?
- How do programming guides integrate with future implementation repositories beyond mis-docs?
- How does UI implementation connect to design system guidance, especially for Web/PWA TypeScript work and later native Swift and Android work?

---

## Related Documents

- [README.md](README.md) — index for this programming guides directory
- [docs/ai-coder-workflow.md](../ai-coder-workflow.md) — contribution model and branch lifecycle
- [docs/ai-coder-sprint-loop.md](../ai-coder-sprint-loop.md) — Coder/QAT pair workflow
- [governance/release-context.md](../../governance/release-context.md) — release coordination model
- [governance/repository-safety-and-canaries.md](../../governance/repository-safety-and-canaries.md) — protected asset classes and repository safety
- [governance/README.md](../../governance/README.md) — development governance model
- [AGENTS.MD](../../AGENTS.MD) — operational rules: commits, branches, git safety
- [AI_CONTEXT.md](../../AI_CONTEXT.md) — project architecture and domain model
- [adr/0003-domain-and-category-markers.md](../../adr/0003-domain-and-category-markers.md) — domain and category marker system
