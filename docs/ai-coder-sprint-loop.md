---
read_when: defining AI Coder sprint orchestration, S-BL processing, Coder/QAT pairing, release build gating, or AI Coder role separation
---

# AI Coder Development Sprint Loop

## Purpose

This document defines the **AI Coder Development Sprint Loop**: how orchestrated AI Coder
and QAT work is driven from the Sprint Backlog (S-BL), how Coder/QAT pairs operate, how
outcomes and follow-ups are recorded, and how the loop connects to release coordination.

The sprint loop is an **event-driven state machine** implemented as **GitHub Actions**
workflows (repository automation), not as a dedicated “orchestrator” AI Agent. Workflows
react to **GitHub Projects (v2)** item events—specifically `projects_v2_item` when a
**workflow state** custom field changes—and dispatch Coder and QAT sessions to external
tools via API or webhook calls.

---

## Status

**Elaborated — operational specification.**

This version replaces the earlier placeholder. It defines GitHub-native backlog placement,
intake, ticket state layers, QAT evidence, follow-up scope rules, orchestration model,
Coder/QAT routing and independence, milestone-based release context, and how these pieces
fit together. It still does **not** ship workflow YAML in *this* repository (workflows live
in component repositories); it specifies the model those workflows should implement.

---

## Scope

This document covers:

- G-BL and S-BL placement and parent/child relationships (mis-docs vs component repos)
- primary feature request intake into the G-BL
- S-BL processing model and workflow state field
- AI Coder role separation and mandatory Coder/QAT pair
- static Coder/QAT routing policy and QAT independence rules
- QAT outcomes, labels, structured PR comment evidence, and follow-up Sub-Issues
- **Scope Guard** for QAT follow-ups (no unauthorized scope extension)
- GitHub Actions + `projects_v2_item` orchestration model (conceptual contract)
- release context via **GitHub Milestones** and dev-complete signaling
- relationship between workflow automation, QAT labels, and Issue open/closed state

---

## Non-Goals

This document does not define:

- concrete GitHub Actions workflow job definitions (owned by component repositories)
- CI/CD configuration for build, regression, or deployment
- Organization-level GitHub Project field setup (manual configuration task)
- Milestone creation mechanics (manual setup)
- a replacement for `AGENTS.MD`
- a replacement for `docs/ai-coder-workflow.md`
- any change to project owner merge authority
- detailed merge-train or integration mechanics for the build/regression phase (separate
  release engineering concern once the milestone is dev-complete)

---

## Core Principle

Every S-BL ticket MUST be handled by a **Coder/QAT Pair** consisting of one implementing
Coder AI Agent and one independent Quality Assurance Tester (QAT) AI Agent.

- The QAT AI Agent MUST NOT be the same AI Agent as the implementing Coder.
- The QAT AI Agent performs detailed review and may run additional tests and checks.
- Self-review is not sufficient for completing an S-BL ticket.
- The PR remains the review gate for repository changes.
- The project owner remains the only merger.

---

## Backlog Placement (G-BL and S-BL) — E1

### Global Backlog (G-BL)

- G-BL items are **GitHub Issues** in the **mis-docs** repository (single source of truth
  for cross-cutting intent).
- A G-BL Issue acts as a **Feature Envelope**: it documents the full lifecycle from idea
  to release.
- The G-BL Issue is **not** closed when S-BL work is decomposed; it remains open until the
  feature **ships in a release** (see Release Context — E7).

### Sprint Backlog (S-BL)

- S-BL tickets are **GitHub Issues** in the **target component repositories** (for example
  `mis-backend`, `mis-android`, `mis-web`).
- When the project owner promotes a G-BL item to S-BL, work is split into
  **component-specific Sub-Issues** in those repositories, using GitHub **Sub-Issues**
  (including cross-repository sub-issues within the same GitHub organization).
- The **G-BL Issue is the parent**; **S-BL tickets are children** of that parent.

### Unified views

- An **organization-level GitHub Project** provides unified views across G-BL (mis-docs)
  and S-BL (component repos), so humans and automation can track one feature across
  repositories.

---

## Feature Request Intake — E2

The **primary intake channel** for new feature ideas into the G-BL is a **GitHub Issue
Form** in mis-docs:

- File: `.github/ISSUE_TEMPLATE/feature-request.yml`
- Contributors, runtime AI Agents acting in a development context, and guests submit
  feature requests by opening an issue from this template.

Future optional channels (web forms, chat integrations) MUST proxy into the same shape:
they create or open GitHub Issues that satisfy the same information expectations, rather
than bypassing the G-BL.

---

## Release Context and Milestones — E7

### Milestone as release context carrier

- **Release Context** maps to **GitHub Milestones**.
- **All component repositories** use **identically named** Milestones for the same release
  (for example `MVP-2026-06-032`).
- The organization-level Project supports **milestone-filtered** release views.

### Dev-complete signal

- When **all S-BL Issues** assigned to a given Milestone are **closed**, the sprint loop
  for that release is **dev-complete** for that milestone scope.
- Dev-complete signals readiness for the **build and regression phase**, which is a
  **separate concern** from this development loop document.

### G-BL alignment

- G-BL parent Issues in mis-docs carry the **same Milestone** as their child S-BL work.
- G-BL Feature Envelope Issues are **closed when the release ships**, not when children
  are merely created.

### Traceability chain

```
Feature Request (G-BL Issue in mis-docs)
  → Milestone (release context)
    → S-BL Sub-Issues (component repos)
      → PRs
        → Release
```

See [`governance/release-context.md`](../governance/release-context.md) for identifier
format, authority, and additional properties of the release context concept.

---

## Roles

### GitHub Actions orchestration (state machine) — E4

- Implements the sprint loop **automation** as workflows: conditionals, API calls,
  dispatches, and guards.
- **Does not** use an AI Agent as the orchestrator; orchestration logic is deterministic
  automation.
- Triggers on **`projects_v2_item`** events when the **workflow state** Project field
  changes (see Ticket State Representation — E8).
- Dispatches **Coder sessions** and **QAT sessions** to external platforms (for example
  `opencode.wolli.at`, `claude.ai/code`) via API or webhook calls with a bounded payload
  (issue reference, branch, PR link, milestone, routing metadata).
- **Build jobs, CI gates, and regression suites** are **out of scope** for this loop;
  they belong to release engineering and component pipelines.

### Coder AI Agent

- implements the assigned S-BL ticket
- works on a feature branch
- commits work
- pushes branch and opens PR when task is complete, according to existing workflow rules
- documents scope, validation, and known limitations

### QAT AI Agent (Quality Assurance Tester)

- independently reviews the Coder output under the **QAT independence rule** (see below)
- reviews branch, PR, diffs, docs, tests, and other relevant artifacts
- posts a **structured QAT Report** as a **PR comment** (mandatory evidence — E6)
- applies or requests application of outcome labels (`qat:completed` or
  `qat:follow-up-required`) per policy
- may request creation of **follow-up Sub-Issues** when outcome is `follow_up_required`,
  subject to the **Scope Guard Rule** (E3)
- does not silently rewrite the original implementation unless explicitly assigned a
  separate correction ticket

### Project Owner

- controls G-BL → S-BL promotion and parent/child backlog structure
- approves and merges PRs
- can override, stop, or reprioritize the sprint loop

---

## Ticket State Representation — E8

Ticket state is intentionally split into **three layers** with distinct responsibilities.

| Layer | Mechanism | Values / meaning | Purpose |
|---|---|---|---|
| **Workflow state** | Organization Project **custom single-select field** | `ready_for_coder`, `in_implementation`, `ready_for_qat`, `in_qat` | Drives `projects_v2_item` automation (E4). |
| **QAT outcome** | **GitHub Labels** on the Issue | `qat:completed`, `qat:follow-up-required` | Persistent, repo-visible record of QAT result, independent of board layout. |
| **Issue status** | Native **Open / Closed** | open → closed after QAT records an outcome | Terminal lifecycle state on the Issue. |

**Interpretation:**

- The **Project field** is the control input for orchestration (what the loop should do
  next).
- **Labels** document **what happened** in QAT for auditability outside the Project view.
- **Open/Closed** reflects completion of the **original ticket’s** QAT cycle (the ticket
  closes for both `completed` and `follow_up_required`; follow-up work is tracked on new
  child Sub-Issues).

---

## S-BL Ticket Lifecycle (Workflow Field)

These states are stored in the **workflow state** Project field (E8). They are not
synonyms for issue open/closed.

| Workflow state | Meaning |
|---|---|
| `ready_for_coder` | Ticket approved for implementation; awaiting Coder dispatch |
| `in_implementation` | Coder session active |
| `ready_for_qat` | Coder completed work; awaiting QAT dispatch |
| `in_qat` | QAT session active |

After QAT finishes:

- the Issue receives the appropriate **`qat:`** label,
- the Issue is **closed**,
- if `follow_up_required`, a **child Sub-Issue** is created under the same parent scope
  rules (E3).

---

## Scope Guard Rule (QAT Follow-Ups) — E3

When QAT outcome is `follow_up_required`, automation or the QAT Agent may create **follow-up
Sub-Issues**.

**Rule:** every follow-up Sub-Issue MUST be a **child of** the **original S-BL ticket**
that failed QAT.

**Scope inheritance:** follow-up tickets **inherit the scope** of the parent S-BL ticket.
They may only describe **corrections or completions within that inherited scope**.

**Prohibited:** using a follow-up ticket to introduce **net-new requirements**, expand
acceptance criteria beyond the parent, or perform unrelated refactors under the guise of
QAT repair.

**If scope must expand:** the actor MUST file a **new G-BL feature request** through the
normal intake channel (E2). That request may later be promoted to S-BL by the project owner
using the standard G-BL → S-BL rules.

**Rationale:** the hierarchy preserves autonomous loop progression without silent scope
creep.

---

## Pair Assignment, Routing, and QAT Independence — E5

### Static routing policy

Coder and QAT sessions are assigned by a **static routing policy** encoded in GitHub
Actions workflow configuration (for example YAML inputs, matrices, or hard-coded
endpoint maps):

- **Example pattern:** Coder sessions dispatch to **Platform A**; QAT sessions dispatch to
  **Platform B**.

The exact endpoints are deployment-specific; the **architectural requirement** is that
policy is explicit, versioned, and reviewable like code.

### QAT independence (normative)

QAT independence is guaranteed by **both** of the following:

1. **Separate sessions with fresh context** — the QAT session MUST NOT receive Coder chat
   transcripts, tool logs, or other Coder session state. It receives only what is needed to
   review: **branch name**, **PR link**, **issue/ticket reference**, milestone/release
   metadata, and policy pointers.
2. **Separate platform or model instance** — Coder and QAT MUST NOT share the same
  provider session continuity tricks that collapse independence (configure distinct
  endpoints or instances per role).

**Operational rule (also recorded in `AGENTS.MD`):** *A QAT Agent receives the branch, the
PR link, and the ticket — but never the session context of the Coder. The QAT always starts
from a fresh perspective.*

---

## Pair-Programming Rule

Every S-BL ticket MUST be assigned to a Pair-Programming Team.

The pair consists of exactly two distinct roles:

- **Coder AI Agent** — implements the ticket
- **QAT AI Agent** — independently reviews the output

Rules:

- The QAT AI Agent must be independent from the Coder (see above).
- The QAT AI Agent must review the actual branch, commits, and PR.
- The QAT AI Agent may add tests and checks beyond those provided by the Coder.
- The QAT outcome determines whether a follow-up Sub-Issue is needed.

---

## QAT Outcome Model

### `completed`

- implementation satisfies the ticket scope
- no follow-up Sub-Issue is required for scope satisfaction
- Issue receives label **`qat:completed`**
- original Issue is **closed**

### `follow_up_required`

- implementation is not yet sufficient within the **inherited scope**
- Issue receives label **`qat:follow-up-required`**
- original Issue is still **closed** after QAT records the outcome
- a **follow-up Sub-Issue** is created as a **child of the original S-BL ticket**, describing
  concrete gaps (see QAT Evidence — E6) and obeying the **Scope Guard Rule** (E3)

---

## QAT Evidence (QAT Report) — E6

The QAT Agent MUST post a **structured QAT Report** as a **top-level PR comment** (not
only inline review threads). This comment is the **authoritative evidence** for the QAT
outcome.

### Mandatory sections (every report)

1. **Scope Check** — does the implementation satisfy the ticket scope **and only** the
   ticket scope (no unauthorized expansion, no silent scope shrink)?
2. **Branch/PR Review** — which artifacts were reviewed; diff coherence; **Conventional
   Commits** compliance at a high level (types/scopes, no merge commits on feature branch).
3. **Outcome** — `completed` or `follow_up_required` with a short **written rationale**.

### Optional sections (ticket-dependent)

- additional test execution notes (unit, integration, manual)
- security review notes
- performance notes

### `follow_up_required` additional requirement

The report MUST include a **concrete description** of what is missing, failing, or
ambiguous. That text is the **primary input** for the follow-up Sub-Issue body (E3).

---

## Sprint Loop (Conceptual Control Flow)

The loop runs until a Milestone is **dev-complete** (all in-scope S-BL Issues closed — E7).

**While a Milestone has open S-BL Issues in scope:**

1. Project field moves ticket to `ready_for_coder` (or automation picks next eligible).
2. GitHub Actions dispatches a **Coder** session (per routing policy).
3. Coder implements on a feature branch; commits; pushes; opens PR.
4. Project field advances through `in_implementation` → `ready_for_qat`.
5. GitHub Actions dispatches a **QAT** session on an independent platform/instance.
6. QAT reviews artifacts; posts the **QAT Report** PR comment (E6).
7. QAT outcome label is applied; Issue is **closed**.
8. If `follow_up_required`, a **child follow-up Sub-Issue** is created (E3), inheriting
   Milestone and scope constraints; loop continues for that new ticket.

**When all S-BL Issues for the Milestone are closed:**

- the milestone is **dev-complete** for development-loop purposes;
- **build, packaging, regression, and release publication** proceed under separate
  release-engineering processes.

This description is a **control-flow contract** for automation authors; it is not
executable code.

---

## Security and Quality Rationale

This model belongs to DEV/SEC as well as DEV/ARCH and DEV/OPS:

- avoids self-review by the same AI Agent, which could allow incomplete or incorrect work
  to pass undetected
- creates explicit role separation between implementation and quality assurance
- improves auditability: each ticket has explicit QAT evidence on the PR; labels survive
  outside Project views
- reduces risk of incomplete implementation passing unchecked
- keeps the project owner as the final merge authority, preserving human oversight
- enforces **scope discipline** on autonomous follow-ups (E3)
- uses GitHub-native structures (Issues, Sub-Issues, Projects, Milestones, Actions) to
  reduce ad-hoc parallel state machines

---

## Open Questions (Resolved)

All previously listed open questions for this document are now addressed within scope by
decisions **E1–E8** (see [`adr/0012-sprint-loop-orchestration-decisions.md`](../adr/0012-sprint-loop-orchestration-decisions.md)).

| Former question | Resolution |
|---|---|
| How is orchestration implemented? | **E4** — GitHub Actions on `projects_v2_item`, no orchestrator AI. |
| How are S-BL tickets represented? | **E1** — Issues + Sub-Issues; org Project for views. |
| Ticket states: labels, milestones, fields? | **E8** three-layer model; **E7** Milestones for release context. |
| Coder/QAT assignment model? | **E5** — static routing in workflow configuration. |
| QAT independence enforcement? | **E5** — fresh context + separate platform/instance. |
| Who creates follow-ups? | **E3** — QAT or automation as child Sub-Issues under Scope Guard. |
| Release build when S-BL empty? | **E7** — Milestone dev-complete; build/regression separate. |
| Multi-branch merge + regression mechanics? | Out of scope for this doc; triggered after dev-complete per release process. |
| Release context connection? | **E7** — Milestones across repos + Project views. |
| QAT evidence requirements? | **E6** — structured PR comment QAT Report. |
| Mandatory vs optional QAT checks? | **E6** — mandatory sections fixed; deeper tests/security/perf optional by ticket. |

---

## Related Documents

- [`docs/ai-coder-workflow.md`](ai-coder-workflow.md) — contribution model: session lifecycle, branch model, PR workflow
- [`governance/README.md`](../governance/README.md) — backlog model (G-BL → S-BL) and approval authority
- [`governance/release-context.md`](../governance/release-context.md) — release context via Milestones
- [`docs/session-start-prompt.md`](session-start-prompt.md) — opening prompt for bootstrapping an AI Coder session
- [`AGENTS.MD`](../AGENTS.MD) — operational rules: commits, branches, git safety, QAT independence, Scope Guard
- [`adr/0003-domain-and-category-markers.md`](../adr/0003-domain-and-category-markers.md) — domain and category marker system
- [`adr/0012-sprint-loop-orchestration-decisions.md`](../adr/0012-sprint-loop-orchestration-decisions.md) — consolidated ADR for E1–E8
- [`docs/github-labels.md`](github-labels.md) — label scheme including QAT outcome labels
