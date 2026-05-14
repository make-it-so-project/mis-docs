---
read_when: defining AI Coder sprint orchestration, S-BL processing, Coder/QAT pairing, release build gating, or AI Coder role separation
---

# AI Coder Development Sprint Loop

## Purpose

This document captures the future development sprint loop for orchestrated AI Coder work.

The sprint loop defines how an orchestration instance coordinates multiple AI Coders
to process the Sprint Backlog using mandatory Coder/QAT Pair, follow-up tickets,
and a release build gate.

---

## Status

**Placeholder / to be elaborated.**

This document captures the topic and initial principles. It is not yet a complete
operational specification. It does not yet define automation, GitHub Projects setup,
issue templates, or release build mechanics.

---

## Scope

This document covers:

- S-BL processing model
- AI Coder role separation
- mandatory Coder/QAT Pair for every S-BL ticket
- QAT outcome handling
- follow-up ticket creation
- orchestration loop
- release build gate after S-BL is empty

---

## Non-Goals

This document does not define:

- implementation automation
- CI/CD configuration
- release process implementation
- detailed GitHub Projects configuration
- a replacement for `AGENTS.MD`
- a replacement for `docs/ai-coder-workflow.md`
- any change to project owner merge authority

---

## Core Principle

Every S-BL ticket MUST be handled by a Coder/QAT Pair consisting of one
implementing Coder AI Agent and one independent Quality Assurance Tester (QAT) AI Agent.

- The QAT AI Agent MUST NOT be the same AI Agent as the implementing Coder.
- The QAT AI Agent performs detailed review and may run additional tests and checks.
- Self-review is not sufficient for completing an S-BL ticket.
- The PR remains the review gate for repository changes.
- The project owner remains the only merger.

---

## Roles

### Orchestration Instance

- coordinates the sprint loop
- assigns or schedules S-BL tickets
- tracks ticket state
- ensures every ticket gets a Coder and an independent QAT AI Agent
- creates or requests follow-up tickets when quality assurance review identifies unfinished work
- starts release build only after S-BL is empty

### Coder AI Agent

- implements the assigned S-BL ticket
- works on a feature branch
- commits work
- pushes branch and opens PR when task is complete, according to existing workflow rules
- documents scope, validation, and known limitations

### QAT AI Agent (Quality Assurance Tester)

- independently reviews the Coder output
- reviews commit, branch, PR, docs, tests, or other relevant artifacts
- performs additional tests and checks where useful
- records QAT outcome
- does not silently fix the original implementation unless explicitly assigned a separate correction ticket

### Project Owner

- controls G-BL to S-BL transition
- approves and merges PRs
- can override, stop, or reprioritize the sprint loop

---

## S-BL Ticket Lifecycle

The following are conceptual states. They do not define final GitHub issue labels.

| State | Meaning |
|---|---|
| `ready_for_coder` | ticket approved for implementation; awaiting Coder assignment |
| `in_implementation` | Coder AI Agent is actively working on the ticket |
| `ready_for_qat` | Coder has completed work; awaiting QAT AI Agent assignment |
| `in_qat` | QAT AI Agent is actively reviewing the output |
| `closed_completed` | QAT outcome: no follow-up required |
| `closed_follow_up_required` | QAT outcome: correction needed; new ticket added to S-BL |

The original S-BL ticket is always closed after the QAT AI Agent records an outcome:

- `closed_completed` — implementation satisfies the ticket scope; no follow-up ticket required.
- `closed_follow_up_required` — implementation needs correction or additional work; the original
  ticket is closed and a new correction/follow-up ticket is added to the S-BL.

---

## Pair-Programming Rule

Every S-BL ticket MUST be assigned to a Pair-Programming Team.

The pair consists of exactly two distinct roles:

- **Coder AI Agent** — implements the ticket
- **QAT AI Agent** (Quality Assurance Tester) — independently reviews the output

Rules:

- The QAT AI Agent must be independent from the Coder.
- The QAT AI Agent must review the actual branch, commit, or PR output.
- The QAT AI Agent may add tests and checks beyond those provided by the Coder.
- The QAT outcome determines whether a follow-up ticket is needed.

---

## QAT Outcome Model

### completed

- implementation satisfies the ticket scope
- no follow-up S-BL ticket required
- original ticket is closed with quality assurance result `completed`

### follow_up_required

- implementation is not yet sufficient
- original ticket is still closed with quality assurance result `follow_up_required`
- a new follow-up or correction ticket is added to the S-BL
- the follow-up ticket references the original ticket, branch, PR, and QAT findings

---

## Sprint Loop

The orchestration loop runs as long as the S-BL contains tickets.

**While S-BL contains tickets:**

1. Orchestration selects the next ready ticket.
2. Orchestration assigns a Coder AI Agent.
3. Coder implements the ticket on a feature branch.
4. Coder commits work, pushes the branch, and opens a PR.
5. Orchestration assigns an independent QAT AI Agent.
6. QAT AI Agent reviews the commit, branch, PR, and relevant artifacts.
7. QAT AI Agent records a QAT outcome (`completed` or `follow_up_required`).
8. Original ticket is closed with the recorded QAT outcome.
9. If `follow_up_required`: a new correction ticket is added to the S-BL and the loop continues.

**When S-BL is empty:**

- The release build phase can start.
- Feature branches and PRs are integrated according to the release process.
- Regression testing is performed.
- Release notes and release artifacts may be prepared.

This description is conceptual. It does not include executable pseudocode or scripts.

---

## Security and Quality Rationale

This model belongs to DEV/SEC as well as DEV/ARCH and DEV/OPS:

- avoids self-review by the same AI Agent, which could allow incomplete or incorrect
  work to pass undetected
- creates explicit role separation between implementation and quality assurance
- improves auditability: each ticket has a named Coder and a named QAT AI Agent in the record
- reduces risk of AI Coder hallucination or incomplete implementation passing unchecked
- encourages independent tests and checks beyond the Coder's own validation
- keeps the project owner as the final merge authority, preserving human oversight
- supports safer parallel AI Coder work by isolating responsibility per ticket

---

## Open Questions

The following questions are deferred to later elaboration:

- How is the orchestration instance implemented?
- How are S-BL tickets represented (GitHub Issues, project fields, a manifest)?
- Are ticket states implemented via GitHub labels, milestones, project fields, or a release manifest?
- How are Coder/QAT pairs assigned — manual, automated, or policy-driven?
- How is QAT AI Agent independence enforced?
- How are follow-up tickets generated — by the QAT AI Agent, by the orchestration instance, or by the project owner?
- How is the release build triggered when the S-BL is empty?
- How are multiple feature branches merged and regression-tested during the release build phase?
- How are release context variables connected to the sprint loop?
- What evidence must a QAT AI Agent provide to record an outcome?
- Which checks are mandatory vs optional for the QAT AI Agent?

---

## Related Documents

- [docs/ai-coder-workflow.md](ai-coder-workflow.md) — contribution model: session lifecycle, branch model, PR workflow
- [governance/README.md](../governance/README.md) — backlog model (G-BL → S-BL) and approval authority
- [docs/session-start-prompt.md](session-start-prompt.md) — opening prompt for bootstrapping an AI Coder session
- [AGENTS.MD](../AGENTS.MD) — operational rules: commits, branches, git safety, slash commands
- [adr/0003-domain-and-category-markers.md](../adr/0003-domain-and-category-markers.md) — domain and category marker system
- [docs/github-labels.md](github-labels.md) — GitHub label scheme matching the marker system
