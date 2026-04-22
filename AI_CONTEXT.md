# AI Context for make-it-so

This document provides context for AI coding assistants and automated development tools
(e.g., ChatGPT Codex, Claude Code, Cursor, GitHub Copilot Chat) that read this repository.

Its purpose is to help AI systems correctly interpret the architecture, terminology,
and intent of the *make-it-so* project.

---

# Project Overview

**make-it-so** is a human-in-the-loop control plane for AI agents.

The system acts as a governance layer between runtime AI agents and actions that
affect external systems or the real world.

Instead of allowing agents to execute actions directly, agents submit structured
action requests to make-it-so. The platform evaluates those requests according to
policy rules and may require explicit human approval before the action can proceed.

The goal is to provide:

- safe operation of AI agents
- explicit human accountability
- auditable decision flows
- controlled autonomy for agent systems

---

# Core Architectural Principle

The architecture separates **preparation of actions** from **authorization of actions**.

AI agents may:

- gather information
- plan actions
- prepare action parameters

However, actions that change the external world must pass through the make-it-so
control plane before execution.

Conceptually:

Agent → Action Request → Policy Evaluation → Approval → Execution

This separation ensures that the AI system does not autonomously execute
high-impact operations.

---

# Runtime Domain vs Development Domain

The project distinguishes two categories of AI systems.

## Runtime Agents

Runtime Agents are AI systems that interact with make-it-so during operation.

Examples include:

- LLM-based agents
- MCP-based tool agents
- autonomous research agents
- code execution agents
- web automation agents

Runtime agents request actions through the make-it-so control plane.

They are part of the **runtime system**.

---

## AI Coders

AI Coders are AI tools that assist in the development of the make-it-so project itself.

Examples include:

- ChatGPT Codex
- Claude Code
- Cursor
- GitHub Copilot

These tools operate in the **development domain**, not in the runtime system.

They read this repository to understand architecture and generate
implementation suggestions or code.

---

# Key Concepts

## Control Plane

The make-it-so platform functions as a **control plane**.

It governs when and how runtime agents are allowed to execute real-world actions.

Responsibilities include:

- receiving action requests
- evaluating policies
- requesting human approval when required
- recording audit events

The control plane does not necessarily execute the action itself.

---

## Action Request

An **Action Request** is a structured message sent by a runtime agent describing
an operation it intends to perform.

Typical properties include:

- action type
- summary description
- agent identifier
- user reference (user_ref)
- optional parameters

Action requests are evaluated by the control plane before execution is allowed.

---

## Human Approval

Certain actions require explicit human approval.

Examples include:

- financial transactions
- bookings or reservations
- system modifications
- access to confidential information

Approval decisions are typically made via a trusted user interface
(e.g., a mobile app).

---

## Execution Connectors

Execution connectors are adapters that interact with external systems.

Examples:

- APIs
- financial services
- workflow engines
- SaaS platforms
- infrastructure management tools

Execution connectors perform the action only after the control plane authorizes it.

---

# Optional Execution Credentials

In some cases, approval may also produce a short-lived execution credential,
for example:

- one-time codes
- second-factor tokens
- temporary authorization tokens

These credentials enable the runtime agent to complete the operation in the
target system.

The control plane transports these credentials but may not generate them.

---

# Design Goals

The make-it-so architecture prioritizes:

- safety of agent operations
- explicit human oversight
- strong auditability
- minimal coupling between agents and execution systems
- extensibility across different agent frameworks

---

# Domain Markers

All structured discussions, issues, PRs, and design threads SHOULD be prefixed
with a `[DOMAIN/CATEGORY]` marker.

## Domains

| Marker | Meaning            |
|--------|--------------------|
| `DEV`  | Development Domain |
| `RUN`  | Runtime Domain     |

## Categories

| Marker | Meaning                             |
|--------|-------------------------------------|
| `ARCH` | Architecture                        |
| `FUNC` | Functional Behavior                 |
| `NFR`  | Non-Functional Requirements         |
| `OPS`  | Operations / Tooling / CI           |
| `SEC`  | Security                            |
| `ADR`  | Architecture Decision Record        |

## Examples

```
[DEV/ADR]   – development-domain architecture decision
[DEV/OPS]   – CI configuration, tooling, build system
[RUN/SEC]   – security concern in the runtime system
[RUN/FUNC]  – functional behavior of a runtime agent
```

## Rule

If the domain or category of a task is unclear, AI Coders MUST ask for
clarification before proceeding. Do not assume scope from context alone.

See ADR 0003 for full rationale.

---

# Guidance for AI Coding Assistants

When generating code, documentation, or architectural proposals for this project:

1. Treat the architecture documents as the source of truth.
2. Respect the separation between agent preparation and action authorization.
3. Assume that all impactful actions must pass through the control plane.
4. Maintain clear boundaries between runtime systems and development tools.
5. Preserve auditability and human approval as central system principles.
