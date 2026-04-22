# ADR-0001: Terminology: AI Coder vs Agent

## Status

Accepted

## Date

2026-04-09

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

## Decision Drivers

- unambiguous communication across documentation, governance, and AI prompts
- clear separation between development tooling and runtime behavior
- consistency across all contributors and AI Coders

## Considered Options

1. No formal distinction – use "agent" for both meanings
2. Separate terms: "AI Coder" for development, "Agent" for runtime (selected)
3. Alternative labels (e.g., "Builder" / "Operator")

## Option Analysis

### Option 1: No formal distinction

#### Advantages
- no convention to learn

#### Disadvantages
- ambiguous in any cross-domain discussion
- AI prompts cannot reliably distinguish development tools from runtime entities

#### Assessment
Unacceptable for a project where both meanings appear constantly.

---

### Option 2: AI Coder vs Agent (selected)

#### Advantages
- unambiguous and self-explanatory
- "AI Coder" clearly signals development tooling
- "Agent" retains its established runtime meaning

#### Disadvantages
- requires consistent adoption across all contributors and documents

#### Assessment
Best balance of clarity and low adoption overhead.

---

### Option 3: Alternative labels

#### Advantages
- avoids overloading any existing term

#### Disadvantages
- unfamiliar vocabulary; higher learning curve
- no advantage over Option 2

#### Assessment
Unnecessary complexity.

## Decision

The following terminology is adopted for the make-it-so project.

### Agent

Used exclusively for runtime entities operating within the make-it-so platform.

### AI Coder

Used for AI-based development systems responsible for modifying the codebase.

## Consequences

### Positive Consequences

- unambiguous terminology across architecture, governance, and AI prompts
- AI Coders can reliably interpret their own role relative to runtime Agents
- consistent vocabulary in backlog definitions and documentation

### Negative Consequences

- requires active enforcement; legacy uses of "agent" in development contexts
  must be corrected as they appear

### Follow-up Implications

- all future documentation must apply this distinction consistently
- AI Coders reading AGENTS.MD inherit this terminology by default

## Rationale Summary

The ambiguity of "agent" in a project that both governs runtime agents and is
built by AI coding tools is a concrete source of confusion. Separate terms with
clear scope eliminate this at negligible adoption cost.
