---
read_when: working on release coordination, current release context, S-BL release assignment, AI Coder sprint orchestration, release build gates, regression testing, or release notes
---

# Release Context

## Purpose

This document captures the future model for assigning AI Coder work, S-BL tickets, sprint
loops, branches, PRs, release builds, and regression testing to a shared release context.

A release context is a stable coordination identifier that tells humans and AI Coders which
product release phase they are working toward. It provides the common boundary for sprint
execution, release build, regression testing, release notes, and traceability.

---

## Status

**Elaborated for coordination metadata — Milestone binding decided.**

This document defines how **release context** is represented for coordination: **GitHub
Milestones** with identical names across repositories, aligned G-BL Issues in mis-docs, and
S-BL Issues in component repositories. It still does **not** define CI/CD implementation,
semantic versioning policy, deployment procedures, or detailed merge-train mechanics for the
build/regression phase. It does not replace `governance/README.md`, `docs/ai-coder-workflow.md`,
or `docs/ai-coder-sprint-loop.md` — it complements them.

---

## Scope

This document covers:

- release context identifier format
- current release context concept
- relation to S-BL tickets
- relation to the AI Coder Development Sprint Loop
- relation to branches and PRs
- relation to the release build gate
- relation to regression testing and release notes
- governance and authority over the active release context
- open questions for later elaboration

---

## Non-Goals

This document does not define:

- implementation code
- release automation
- CI/CD pipeline definition
- GitHub Projects configuration
- release manifest file format
- semantic versioning decision
- production deployment process
- changes to project owner approval or merge authority
- changes to the G-BL to S-BL approval model

---

## Core Principle

All concrete S-BL implementation work SHOULD be associated with a release context. The
release context provides the coordination boundary for sprint execution, release build,
regression testing, release notes, and traceability.

Required properties:

- AI Coders should know the current release context before implementing a concrete S-BL task.
- A Dev Sprint Loop should operate against one current release context.
- The release build phase should only start for a release context after the S-BL scope for
  that release is empty or explicitly frozen.
- Release context is coordination metadata, not an authorization mechanism.
- The project owner controls creation and changes to the active release context.

---

## Release Context Identifier

The conceptual identifier pattern is:

```
RELEASEPHASE-YYYY-MM-NNN
```

Where:

- `RELEASEPHASE` — the release phase label, for example `MVP`, `ALPHA`, `BETA`, `RC`, or `PROD`
- `YYYY-MM` — the planned release timeframe: four-digit year and two-digit month
- `NNN` — a zero-padded monotonically increasing sequence number within that release phase
  or release train

Examples:

| Identifier | Meaning |
|---|---|
| `MVP-2026-05-001` | First MVP release context in May 2026 |
| `ALPHA-2026-07-042` | 42nd Alpha release context in July 2026 |
| `RC-2026-09-174` | 174th Release Candidate context in September 2026 |

Suggested release phases:

| Phase | Meaning |
|---|---|
| `MVP` | First minimum viable product release line |
| `ALPHA` | Early user or testing phase |
| `BETA` | Broader testing phase (optional) |
| `RC` | Release candidate |
| `PROD` | Production release line (optional later) |

The prefix indicates the release phase. The `YYYY-MM` segment identifies the planned
release timeframe. The numeric suffix prevents ambiguity between multiple release contexts
in the same month and phase.

The exact governance semantics of the sequence number remain a project-owner concern when
creating new Milestones. **Authoritative storage for the active release target:** GitHub
Milestone on Issues (G-BL and S-BL) plus organization-level Project views filtered by that
Milestone. See [`docs/ai-coder-sprint-loop.md`](../docs/ai-coder-sprint-loop.md) (Release
Context — E7) and [`adr/0012-sprint-loop-orchestration-decisions.md`](../adr/0012-sprint-loop-orchestration-decisions.md).

---

## Current Release Context

The current release context is the active release target for the current Dev Sprint Loop
and concrete S-BL implementation work.

**Authoritative mechanism:** GitHub **Milestones** using the identifier pattern defined in
this document. The same Milestone name MUST exist in each component repository that carries
S-BL work for that release, so views and filters align across repositories.

**Visibility for AI Coders:** before starting an S-BL task, read the **Milestone** on the
assigned Issue (and the parent G-BL Feature Envelope Issue in mis-docs when present). The
organization-level GitHub Project SHOULD expose a Milestone-filtered view for release
readiness.

Optional supplementary cues (PR body text, labels) may exist, but **Milestone assignment is
the normative release association** for Issues in this model.

---

## Relationship to the Sprint Backlog

- S-BL tickets should belong to a release context.
- A release context may contain multiple S-BL tickets.
- A ticket should not silently move between release contexts.
- If a ticket is moved to a different release context, the reason should be recorded.
- Follow-up tickets created by the Coder/QAT process should inherit the release context of
  the original ticket unless the project owner explicitly assigns a different one.
- G-BL **Feature Envelope** Issues in mis-docs SHOULD carry the **same Milestone** as their
  child S-BL Issues once those children are scheduled for a concrete release target.

---

## Relationship to the AI Coder Development Sprint Loop

- **GitHub Actions** orchestration SHOULD coordinate work within the current release context
  (Milestone) encoded on each Issue.
- Every Coder/QAT Pair should know the release context of the S-BL ticket being processed.
- QAT outcomes should be recorded against the same release context as the original ticket.
- The release build phase should start for a release context only after all in-scope S-BL
  tickets are completed or explicitly deferred by the project owner.
- This document extends, but does not replace, `docs/ai-coder-sprint-loop.md`.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md) for the sprint loop
model, Coder/QAT pair rule, and QAT outcome model.

---

## Relationship to Branches and PRs

- Branch names do not need to include the release context identifier by default. The
  existing branch naming convention (`docs/`, `feat/`, `fix/`, `chore/`) remains unchanged.
- PR bodies SHOULD name the **Milestone** / release context the PR serves.
- PR titles may remain focused on domain/category markers and the task description.
- Release context is primarily carried by **Issue Milestones**; PR linkage inherits through
  the linked S-BL Issue.

---

## Release Build Gate

A release build for a release context should not start until the following conditions are
met:

- all in-scope S-BL tickets are completed or explicitly deferred by the project owner
- required QAT outcomes exist for all completed tickets in scope
- relevant PRs are merged or explicitly excluded from this release context
- release safety checks are clear or explicitly accepted
- regression testing can run against a known and coherent set of changes

The final release process and CI/CD pipeline definition are deferred to later elaboration.

---

## Regression Testing and Release Notes

- Regression testing should be scoped to the release context.
- Release notes should be generated or curated per release context.
- Release notes should summarize completed S-BL items, follow-up items, known limitations,
  and safety-relevant changes for the release context.
- QAT outcomes and the PR history form part of the release audit trail for each release
  context.

---

## Governance and Authority

- The project owner controls creation and changes to the active release context.
- AI Coders may read and report the release context when working on S-BL tasks.
- AI Coders must not silently change the active release context unless explicitly assigned
  a task to do so.
- Release context changes are protected coordination changes and must be called out
  explicitly in PR bodies.
- Repository safety rules apply to future release context files and manifests. See
  [governance/repository-safety-and-canaries.md](repository-safety-and-canaries.md).

---

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| AI Coders work against different release targets | Current release context made explicit and reported in task and PR metadata |
| S-BL tickets are merged without release association | Release context required for concrete implementation work |
| Release build starts before all intended work is complete | Release build gate checks S-BL state and QAT outcomes before proceeding |
| Follow-up tickets lose traceability | Follow-up tickets inherit release context and reference original ticket, branch, PR, and QAT findings |
| Release context is changed accidentally | Project owner authority; protected asset class review; PR callout required |
| Too much process introduced too early | Milestones are lightweight metadata; heavy automation remains optional |

---

## Open Questions

Questions about **where release context lives** and **how S-BL and PRs reference it** are
resolved by **ADR-0012 / E7** (GitHub Milestones + organization Project views). The
following items remain **outside** this coordination document or are **project-owner
process** choices:

- semantic versioning and marketing version strings vs Milestone identifiers
- CI/CD pipeline layout and mandatory regression evidence artifacts
- who may create, rename, freeze, or close Milestones (governance: project owner authority)
- minimum viable release note format per release
- repository safety check integration details beyond high-level release gates

---

## Resolved coordination questions (E7)

| Topic | Decision |
|---|---|
| Authoritative release association for Issues | GitHub **Milestone** (same name across repos) |
| S-BL ticket reference | Issue belongs to the release **Milestone** |
| PR reference | PR SHOULD state Milestone; inherits via linked Issue |
| Follow-up ticket assignment | Follow-ups **inherit** parent Milestone unless project owner reassigns |
| Sprint loop integration | When all Milestone S-BL Issues are **closed**, development is **dev-complete** for that Milestone; build/regression follows separately |

---

## Related Documents

- [governance/README.md](README.md) — development governance model, backlog rules, approval authority
- [governance/repository-safety-and-canaries.md](repository-safety-and-canaries.md) — repository safety principles and protected asset classes
- [docs/ai-coder-workflow.md](../docs/ai-coder-workflow.md) — contribution model and branch lifecycle
- [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md) — Coder/QAT pair workflow and release build gate
- [docs/session-start-prompt.md](../docs/session-start-prompt.md) — opening prompt for AI Coder sessions
- [AGENTS.MD](../AGENTS.MD) — operational rules: commits, branches, git safety, slash commands
- [adr/0012-sprint-loop-orchestration-decisions.md](../adr/0012-sprint-loop-orchestration-decisions.md) — sprint loop orchestration decisions (E1–E8)
- [docs/github-labels.md](../docs/github-labels.md) — GitHub label scheme matching the marker system
