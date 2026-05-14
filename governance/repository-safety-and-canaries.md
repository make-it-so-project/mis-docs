---
read_when: working on repository safety, AI Coder safety, canary or tripwire concepts, protected asset classes, security review gates, or release safety checks
---

# Repository Safety and Canary Principles

## Purpose

This document captures public safety principles for protecting the make-it-so repositories
against accidental or malicious changes by humans, AI Coders, automations, or compromised
tooling.

It describes what classes of assets require special care and what categories of safeguards
may exist. It intentionally does not disclose concrete canary values, detection logic,
private alert routing, secret names, incident response internals, or operational response
details that would help an attacker or misbehaving agent bypass safety mechanisms.

---

## Status

**Placeholder / to be elaborated.**

This document captures public principles and initial scope. It is not a complete security
implementation. It does not define concrete canary values, exact scanning logic, private
runbooks, or operational response paths. It does not replace `AGENTS.MD`,
`docs/ai-coder-workflow.md`, or `governance/README.md`.

---

## Scope

This document covers:

- AI Coder repository safety principles
- public canary, watcher, and tripwire concepts
- protected asset classes
- review and quality gates
- public documentation boundaries
- relation to the Coder/QAT pair workflow
- relation to future release safety checks
- open questions for later elaboration

---

## Non-Goals

This document does not define:

- concrete tripwire implementation details
- secret values of any kind
- fake secret or honeytoken values
- exact detection queries or scanning rules
- exact CI validation rules
- alert routing details
- incident response runbooks
- backup or restore credentials or paths
- changes to GitHub branch protection
- changes to merge authority
- changes to sprint loop mechanics

---

## Core Principle

Repository safety is based on layered controls. Canary and watcher mechanisms may detect
suspicious or unexpected changes, but they do not replace least privilege, branch protection,
Coder/QAT separation, required review, CI checks, backups, rollback, and auditability.

Additional required properties:

- A canary is a signal, not a complete protection mechanism.
- Watchers and tripwires detect unexpected changes or dangerous patterns.
- Concrete canary values and detection details must not be disclosed publicly.
- AI Coders must treat protected asset classes with special care.
- Security and governance weakening must never be hidden inside unrelated changes.

---

## Public vs Private Boundary

The public repository may document:

- safety principles and philosophy
- protected asset classes
- review expectations for sensitive changes
- broad categories of checks that may exist
- that canary, watcher, or tripwire mechanisms may exist
- that critical governance, security, workflow, CI, and architecture files require special
  attention from AI Coders and reviewers

The public repository must not document:

- concrete canary values or markers
- hashes of protected content used as tripwires
- fake secret values or honeytoken values
- exact detection queries or match patterns
- exact alert-routing rules or destination details
- private incident response runbooks
- internal backup, restore, or failover paths
- exact thresholds or bypass windows that would help evade detection
- secret names, token scopes, or private GitHub App configuration details

The public documentation describes the safety model and protected asset classes. Concrete
canary values, detection rules, alert routing, and operational response details are
intentionally kept outside the public repository.

---

## Protected Asset Classes

The following asset classes require special care. Changes to these files must be called out
explicitly in the PR body and are subject to QAT review and project owner approval.

| Asset Class | Examples | Risk if Weakened |
|---|---|---|
| Governance rules | `governance/README.md`, `docs/ai-coder-workflow.md`, `docs/ai-coder-sprint-loop.md` | Weakening approval authority, S-BL rules, or Coder/QAT separation |
| GitHub G-BL intake templates | `.github/ISSUE_TEMPLATE/` | Bypassing the G-BL intake channel, weakening intake quality, or enabling unvetted scope into the backlog |
| AI Coder operating instructions | `AGENTS.MD`, `docs/session-start-prompt.md`, `docs/slash-commands/` | Changing AI Coder behavior, bypassing PR flow, weakening git safety |
| Architecture decisions | `adr/`, `architecture/` | Silently changing approved decisions or security boundaries |
| Security-sensitive documentation | Client identity, agent interface security, account recovery, repository safety | Weakening authentication, approval, recovery, or audit requirements |
| CI/CD and workflow configuration | GitHub workflow files, build scripts, deployment manifests (when they exist) | Disabling checks, bypassing review gates, injecting malicious deployment behavior |
| Dependency and supply-chain metadata | Lockfiles, package manifests, dependency configuration (when they exist) | Introducing unsafe dependencies or weakening reproducibility |
| Release and sprint coordination artifacts | GitHub Milestones (release context), organization-level GitHub Projects, future release manifests | Misaligning AI Coder work, releasing incomplete or unreviewed changes |
| Design system and approval UX documentation | `docs/design-system/`, future token and component definitions that affect approval behavior | Weakening interaction clarity, accessibility, or the trusted approval surface model |

---

## Protected Asset Class Evaluation

Protected asset classes are not a static, fixed list. New document areas, configuration
files, workflow files, implementation areas, release artifacts, or design artifacts may
create new protected asset classes as the project evolves.

AI Coders must evaluate protected asset impact when introducing any such area and must
make the evaluation visible in the PR body.

### When to Evaluate

Evaluate protected asset impact whenever a task introduces a new:

- documentation area (new top-level directory or major sub-directory)
- configuration area (new configuration files, environment definitions)
- workflow area (new automation, sprint coordination, or release coordination artifacts)
- implementation area (new implementation repository, package, or module)
- release artifact type (new build artifact, manifest, or release coordination file)
- design or system artifact (UI design system files, token definitions, component specs)
- operational area (hosting config, deployment manifests, backup or restore processes)

### Evaluation Criteria

A new artifact area should be considered for protected asset status if it can affect:

- governance authority or approval model
- AI Coder behavior, branch workflow, or PR merge rules
- security posture, authentication, approval, client trust, recovery, or audit behavior
- deployment, hosting, operations, backup, restore, or failover
- dependencies, build reproducibility, or supply-chain risk
- release coordination, release gates, or regression evidence
- user-facing safety, approval clarity, accessibility, or the trusted approval surface
- architecture decisions or architectural trust boundaries

### Required AI Coder Action

For every task that introduces a new artifact area, the AI Coder must do exactly one of:

1. Update the Protected Asset Classes table above with a concise new or expanded entry,
   and state this in the PR body as: `Protected Asset Class: added/updated — <summary>`
2. State that the new area is covered by an existing protected asset class and name it:
   `Protected Asset Class: covered by existing — <which class and why>`
3. State that no protected asset class update is needed and explain why:
   `Protected Asset Class: none needed — <reason>`

This evaluation must appear in the PR body. An absent evaluation is a gap that QAT must
flag.

### What This Section Does Not Define

This section does not define concrete canary values, tripwire rules, detection queries,
alert routing, scanning thresholds, or private operational response details. Those are
kept outside the public repository.

---

## Safety Mechanism Categories

The following categories of safeguards may apply to this repository. Exact rules,
thresholds, and implementations are intentionally not disclosed here.

- **Branch protection** — prevents direct pushes to `main`; requires pull requests
- **Feature branch workflow** — isolates work in progress from the main branch
- **PR review and project owner merge authority** — all merges require project owner approval
- **Coder/QAT pair workflow** — mandatory independent review for every S-BL ticket
- **QAT review scope** — branch, commit, PR, tests, and relevant artifacts
- **Secret scanning** — detects accidentally committed credentials
- **Dependency review** — flags dependency changes for additional scrutiny
- **CI validation** — automated checks on pull requests
- **Link and documentation consistency checks** — detects broken internal references
- **Protected asset change detection** — may flag or call attention to changes in sensitive files
- **Release gate checks** — safety conditions evaluated before a release proceeds
- **Backup and restore capability** — git history and external backups
- **Audit logs and PR history** — permanent record of changes and approvals
- **Canary, watcher, and tripwire mechanisms** — may exist; details intentionally not disclosed

---

## AI Coder Safety Expectations

AI Coders must observe the following when working in this repository:

- AI Coders must not hide safety-relevant changes inside unrelated tasks.
- AI Coders must not remove or weaken branch protection, review gates, security requirements,
  or governance rules unless explicitly assigned a task to do so.
- AI Coders must call out changes to protected asset classes in the PR body and scope
  boundary section.
- AI Coders must include validation evidence for changes affecting safety-sensitive files.
- A Coder AI Agent must not self-certify sensitive safety changes without independent QAT
  review.
- The QAT AI Agent must pay special attention to protected asset class changes and must
  explicitly confirm whether such changes weaken any safety control.

---

## Canary / Watcher / Tripwire Principles

### Canary

A deliberately placed signal or marker used to detect unexpected access, modification,
deletion, or bypass behavior. A canary detects — it does not prevent. Its value is in
raising a signal that something unexpected occurred, enabling investigation and response.

### Watcher

A process, check, or review mechanism that observes changes to protected assets or monitors
for suspicious patterns. Watchers may be automated, semi-automated, or human-driven.

### Tripwire

A detection rule or condition that triggers attention when a protected asset or expected
invariant changes. A tripwire is a form of watcher with a specific trigger condition.

**Concrete canary values, watcher logic, and tripwire conditions are not defined in this
public document.** They must be stored and managed outside the public repository.

---

## Release Safety Relationship

The future release build phase should include safety checks before a release is considered
ready. The following are high-level categories; exact implementation is not defined here.

Possible release safety checks:

- critical governance, security, and architecture files were not unexpectedly removed or
  weakened
- ADR index and architecture index are coherent with actual file state
- protected asset class changes were called out in PRs and reviewed
- QAT outcomes exist for all S-BL tickets in scope for the release
- dependency and workflow changes received appropriate special review
- no known canary or tripwire signals remain unresolved at release time

This list is illustrative. The release safety process is defined separately.

---

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| AI Coder deletes or rewrites critical governance files | Protected asset class detection; QAT review; PR review; project owner merge authority; backups |
| AI Coder weakens security rules while implementing an unrelated task | PR scope boundary requirement; protected asset callout; QAT review; project owner approval |
| Malicious dependency or supply-chain change | Dependency review; lockfile review; CI checks; release gate |
| Accidental or fake secret committed | Secret scanning; code review; immediate revocation procedure (defined outside public repo) |
| Canary or tripwire details are exposed publicly | Public/private boundary; concrete values and rules kept outside public repo |
| AI Coder self-review misses defects | Mandatory Coder/QAT Pair for every S-BL ticket |
| Repository state is damaged | Branch protection; git history; backups; patch handoff procedure; rollback capability |

---

## Open Questions

The following questions are deferred to later elaboration:

- Which protected asset classes should trigger mandatory QAT review regardless of task scope?
- Which safety checks belong in CI and which belong in private operational tooling?
- Where are private canary values, detection rules, and alert routing configuration stored?
- How are alerts routed to the project owner without exposing routing details publicly?
- How are repository backups created, tested, and stored?
- How should safety checks integrate with the future release context model?
- How should safety checks integrate with the AI Coder Development Sprint Loop?
- What evidence must a QAT AI Agent provide when reviewing protected asset changes?
- Which changes require explicit project owner confirmation before implementation begins,
  beyond the standard S-BL approval?

---

## Related Documents

- [governance/README.md](README.md) — development governance model, backlog rules, approval authority
- [docs/ai-coder-workflow.md](../docs/ai-coder-workflow.md) — contribution model and branch lifecycle
- [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md) — Coder/QAT pair workflow and release build gate
- [docs/session-start-prompt.md](../docs/session-start-prompt.md) — opening prompt for AI Coder sessions
- [AGENTS.MD](../AGENTS.MD) — operational rules: commits, branches, git safety, slash commands
- [adr/0003-domain-and-category-markers.md](../adr/0003-domain-and-category-markers.md) — domain and category marker system
- [docs/github-labels.md](../docs/github-labels.md) — GitHub label scheme matching the marker system
