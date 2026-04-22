---
read_when: applying or interpreting domain/category markers on issues, PRs, or discussions
---

# ADR-0003: Introduce domain and category markers for agent communication

## Status

Accepted

## Date

2026-04-22

## Context

The make-it-so project involves two distinct domains of activity:

- the **Development Domain** (AI Coders, contributors, tooling)
- the **Runtime Domain** (operational agents, control plane, execution)

As the number of contributors and AI Coders grows, discussions in issues, PRs,
and design threads increasingly mix concerns from both domains. This creates
ambiguity for both human contributors and AI Coders when interpreting the
scope and intent of a discussion.

A lightweight, machine-readable tagging system is needed to make domain
boundaries explicit at the point of communication.

## Decision Drivers

- reduce ambiguity between development and runtime discussions
- enable AI Coders to correctly scope their responses and changes
- support consistent filtering and routing of issues and PRs
- low overhead: markers must be quick to apply and easy to parse

## Considered Options

1. Free-form labels in issue/PR titles
2. Structured `[DOMAIN/CATEGORY]` markers as a project-wide convention
3. Separate repositories per domain

## Option Analysis

### Option 1: Free-form labels

#### Advantages
- no convention to learn

#### Disadvantages
- inconsistent; not machine-readable; hard to enforce

#### Assessment
Insufficient for multi-agent coordination.

---

### Option 2: Structured markers (selected)

#### Advantages
- machine-readable
- compact and unambiguous
- composable: domain + category in one token

#### Disadvantages
- requires convention adoption by all contributors and AI Coders

#### Assessment
Best balance of clarity and low overhead.

---

### Option 3: Separate repositories

#### Advantages
- hard separation of concerns

#### Disadvantages
- high overhead; poor for cross-cutting discussions

#### Assessment
Too heavy for the current project stage.

## Decision

Adopt structured `[DOMAIN/CATEGORY]` markers for all agent interactions,
issues, PRs, and design discussions.

### Domains

| Marker | Meaning            |
|--------|--------------------|
| `DEV`  | Development Domain |
| `RUN`  | Runtime Domain     |

### Categories

| Marker | Meaning                                  |
|--------|------------------------------------------|
| `ARCH` | Architecture                             |
| `FUNC` | Functional Behavior                      |
| `NFR`  | Non-Functional Requirements              |
| `OPS`  | Operations / Tooling / CI                |
| `SEC`  | Security                                 |
| `ADR`  | Architecture Decision Record             |

### Usage

Markers appear at the start of an issue title, PR title, or discussion thread:

```
[DEV/ADR]   – a development-domain architecture decision
[DEV/OPS]   – CI configuration, tooling, build system
[RUN/SEC]   – security concern in the runtime system
[RUN/FUNC]  – functional behavior of a runtime agent
```

### Rules

- Every structured discussion SHOULD include a domain marker.
- If the domain or category of a task is unclear, agents MUST ask for
  clarification before proceeding.
- AI Coders MUST NOT assume domain scope from context alone when a marker
  is absent from an ambiguous task.

## Consequences

### Positive Consequences
- unambiguous scope signal for AI Coders and human contributors
- enables structured filtering of issues and PRs by domain
- reduces cross-domain confusion in multi-agent workflows

### Negative Consequences
- requires consistent application; gaps will occur during adoption

### Follow-up Implications
- GitHub label scheme may mirror these markers in a later step
- Slash command templates (`/handoff`, `/pickup`) should include the marker

## Rationale Summary

The `[DOMAIN/CATEGORY]` convention provides the clearest signal-to-noise ratio
for a multi-agent, multi-contributor project. It is cheap to apply, easy to
parse, and directly maps to the domain model already established in the project.
