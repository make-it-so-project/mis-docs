# ADR-0012: Sprint loop orchestration decisions (E1–E8)

## Status

Accepted

## Date

2026-05-14

## Context

The **AI Coder Development Sprint Loop** (`docs/ai-coder-sprint-loop.md`) and related
governance documents described Coder/QAT pairing, QAT outcomes, and a future orchestration
layer, but left critical mechanics as placeholders: where G-BL and S-BL live, how intake
works, how ticket state is represented, how orchestration runs, how QAT proves its verdict,
how follow-ups stay in scope, how Coder/QAT assignment preserves independence, how release
context ties to the loop, and how all of this maps to GitHub.

This ADR records eight elaboration decisions (**E1–E8**) agreed for the **development
domain** so AI Coders, automation authors, and reviewers share one operational model. The
**primary specification** updated alongside this ADR is `docs/ai-coder-sprint-loop.md`.

**Domain marker:** `[DEV/ARCH]`

## Decision Drivers

- Use **GitHub-native** structures (Issues, Sub-Issues, Projects, Milestones, Actions) for
  traceability and tooling compatibility.
- Keep **mis-docs** as the **single source of truth** for cross-cutting intent (G-BL).
- Preserve **project owner authority** over G-BL → S-BL promotion and merges.
- Prevent **self-review** and preserve **QAT independence** with reviewable rules.
- Allow **autonomous follow-ups** without **silent scope creep**.
- Avoid burning **model tokens** on deterministic orchestration logic.
- Make **QAT outcomes auditable** outside of a single Project board view.
- Align **release context** with a simple, filterable coordination mechanism across repos.

## Considered options (orchestration and state)

| Option | Summary |
|---|---|
| A | Dedicated “orchestrator” AI Agent interpreting board state |
| B | GitHub Actions workflows driven by Project/item events (`projects_v2_item`) |
| C | Polling scripts or label-only proxies for board state |

Assessment: **B** is selected (see E4). **A** couples reliability and cost to an LLM for
pure control flow. **C** is higher latency and easier to drift from board truth.

## Decision

Adopt decisions **E1 through E8** as specified in this ADR and in
`docs/ai-coder-sprint-loop.md`. Component repositories SHOULD implement GitHub Actions
workflows that honor this contract; **mis-docs** documents the contract and hosts G-BL
intake.

---

## E1 — G-BL and S-BL location

**Decision**

- **G-BL:** GitHub Issues in **mis-docs**. A G-BL Issue is a **Feature Envelope** for the full
  lifecycle; it closes when the feature **ships in a release**, not when S-BL children are
  created.
- **S-BL:** GitHub Issues (including **Sub-Issues**) in **component repositories**, with the
  G-BL Issue as **parent** and S-BL tickets as **children** (cross-repo within the org where
  supported).
- **Unified views:** an **organization-level GitHub Project** spans both backlogs.

**Rationale**

- mis-docs remains the architectural and governance anchor.
- Component repos own executable work items.
- Native parent/child preserves traceability without a parallel issue system.

**Rejected / alternatives**

- **Single-repo S-BL** — loses component ownership and permission boundaries.
- **Free-form markdown backlog** — weak structure, poor automation hooks.
- **Closing G-BL when splitting S-BL** — loses Feature Envelope traceability through release.

---

## E2 — Feature request intake channel

**Decision**

- Primary intake is a **GitHub Issue Form** in mis-docs:
  `.github/ISSUE_TEMPLATE/feature-request.yml`.
- Future channels (web forms, chat) MUST proxy into the same intake shape.

**Rationale**

- One visible queue for triage; forms enforce minimum structured context.
- Guests and AI agents can contribute without bespoke accounts per channel.

**Rejected / alternatives**

- **Discussions-only intake** — harder to gate, weaker linkage to ADR/governance workflow.
- **Direct-to-S-BL filing by contributors** — violates G-BL → S-BL approval authority.

---

## E3 — QAT follow-up scope guard

**Decision**

- On `follow_up_required`, follow-ups are **Sub-Issues** of the **original S-BL ticket**.
- Follow-ups **inherit parent scope**; they may only complete or correct work **inside** that
  scope.
- Broader scope changes require a **new G-BL feature request** via normal intake (E2).

**Rationale**

- Enables autonomous corrections without owner micro-management for every defect.
- Parent/child hierarchy blocks scope smuggling in follow-ups.

**Rejected / alternatives**

- **Owner-only follow-up creation** — slows the loop; still valid as override, not default.
- **Flat new issues without parent linkage** — traceability loss and scope ambiguity.

---

## E4 — Orchestration platform

**Decision**

- Orchestration is **GitHub Actions** implementing a **state machine**.
- Workflows trigger from **`projects_v2_item`** events when the **workflow state** Project
  field changes.
- Workflows **dispatch** Coder/QAT sessions to external tools via API/webhook; no
  orchestrator AI.

**Rationale**

- Deterministic automation is cheaper, testable, and reviewable than LLM orchestration.
- Event-driven updates avoid label proxies and polling.

**Rejected / alternatives**

- **Orchestrator AI Agent** — unnecessary intelligence for routing; higher failure and cost
  surface.
- **Human-only state transitions at scale** — does not scale with parallel Coders.

---

## E5 — Coder/QAT pair assignment and independence

**Decision**

- Routing is a **static policy** in workflow configuration (example: Coder → platform A,
  QAT → platform B).
- Independence requires **(1)** fresh QAT context and **(2)** separate platform or model
  instance from the Coder.

**Rationale**

- Static routing is auditable in YAML and diffs like code.
- Dual separation prevents “soft” self-review via shared session memory.

**Rejected / alternatives**

- **Dynamic LLM-chosen routing** — opaque, harder to audit, brittle.
- **Same platform with only a system prompt wall** — weaker independence evidence.

---

## E6 — QAT evidence requirements

**Decision**

- QAT posts a **structured QAT Report** as a **PR comment** with mandatory sections: **Scope
  Check**, **Branch/PR Review**, **Outcome** (`completed` or `follow_up_required` plus
  rationale).
- Optional sections: extra tests, security, performance.
- `follow_up_required` MUST describe concrete gaps feeding the follow-up Sub-Issue (E3).

**Rationale**

- PR-linked evidence is durable and reviewable by humans and QAT peers.
- Mandatory sections reduce “LGTM” noise outcomes.

**Rejected / alternatives**

- **Issue-only QAT notes** — weaker linkage to the diff under review.
- **Unstructured free-text only** — hard to automate and inconsistent for follow-ups.

---

## E7 — Release context via milestones

**Decision**

- Release context maps to **GitHub Milestones** with **identical names** across component
  repos (for example `MVP-2026-06-032`).
- Org Project provides Milestone-filtered release views.
- When **all S-BL Issues** in a Milestone are **closed**, the loop is **dev-complete** for
  that release slice; build/regression is **separate**.
- G-BL Feature Envelopes carry the same Milestone and close when the **release ships**.

**Rationale**

- Milestones are native, filterable, and work uniformly across repos with naming discipline.
- Dev-complete is an objective graph condition for planners.

**Rejected / alternatives**

- **Manifest file as sole authority** — extra drift risk vs Issues; can supplement later.
- **Project field as sole release carrier** — less portable across views/APIs than Milestones
  for repo-local queries.

---

## E8 — Ticket state representation

**Decision**

Three layers:

1. **Workflow state** — Project **single-select**: `ready_for_coder`, `in_implementation`,
   `ready_for_qat`, `in_qat` (drives automation).
2. **QAT outcome** — Issue **labels**: `qat:completed`, `qat:follow-up-required`.
3. **Issue status** — GitHub **Open/Closed** after QAT records an outcome.

**Rationale**

- Separates control inputs, durable audit labels, and lifecycle completion.
- Avoids overloading any single mechanism.

**Rejected / alternatives**

- **Labels-only state machine** — noisy for driving `projects_v2_item` style automation.
- **Open/Closed as QAT outcome** — conflates ticket completion with board workflow steps.

---

## Consequences

### Positive

- End-to-end traceability from G-BL Feature Envelope → Milestone → S-BL → PR → QAT evidence.
- Clear automation contract for component repos.
- QAT independence and scope guard are enforceable in review and tooling.

### Negative

- Requires **manual discipline** and occasional scripting to keep Milestone names aligned
  across repositories.
- Organization Project configuration is **non-trivial** setup work outside this ADR.

### Follow-up implications

- Component repositories need **workflow implementations**, label catalogs, and Project
  fields matching E8.
- Build/regression/release publication remains to be defined under release engineering.

## Rationale Summary

E1–E8 anchor the sprint loop in GitHub’s native collaboration model, separate deterministic
orchestration from LLM work, harden QAT with evidence and independence rules, bound follow-up
scope, and tie releases to Milestones without pretending CI details are part of the core dev
loop. The detailed operational text lives in `docs/ai-coder-sprint-loop.md`, updated as the
authoritative companion to this ADR.
