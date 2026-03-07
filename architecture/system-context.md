# System Context

## 1. Purpose of make-it-so

**make-it-so** is a human-in-the-loop control plane for AI agents.

Its purpose is to provide a standardized control and approval layer between **runtime AI agents** and **real-world effects**. Instead of allowing agents to directly execute impactful actions, make-it-so receives structured action requests, applies governance and policy checks, optionally requests human approval, and returns a decision that determines whether the action may proceed.

The system is intended to support controlled autonomy for AI agents while preserving human accountability for actions that change the state of the world.

---

## 2. Problem Statement

Modern AI agents can already plan and prepare complex operations, for example:

- calling tools
- executing code
- accessing APIs
- interacting with websites and systems
- triggering workflows
- retrieving or processing sensitive data

In many cases, these operations are not purely informational. They may create obligations, spend money, alter systems, disclose confidential information, or otherwise produce external consequences.

Without an explicit control layer, such agents operate with unclear boundaries between:
- observation and action
- recommendation and execution
- machine autonomy and human responsibility

This creates practical and architectural problems:

- unsafe or irreversible actions may be executed automatically
- auditability is weak or missing
- approval flows are inconsistent across agents and tools
- accountability is blurred
- security-sensitive execution patterns are hard to standardize

make-it-so addresses this by inserting a **mandatory approval and governance layer** between agent intent and action execution.

---

## 3. System Concept

make-it-so is not the agent, not the execution engine, and not the business system being acted upon.

It is a **control plane** that sits between AI agents and action execution.

Its core function is to accept structured action requests from agents and determine, according to policy and context, whether the request:

- may proceed automatically
- requires explicit human approval
- must be denied
- must be logged or audited

At a conceptual level, make-it-so follows a simple distinction:

- **information gathering and analysis** are generally non-destructive and may be allowed without approval
- **actions that change the world** require explicit control and, in many cases, explicit human approval

This distinction is similar to the separation between **sensing** and **actuation** in robotics:
- reading is not the same as acting
- preparing is not the same as committing
- suggesting is not the same as executing

In this model, the AI agent may prepare an action, but the authority to allow execution remains outside the agent.

---

## 4. Runtime Domain vs Development Domain

The project distinguishes two clearly separated domains.

### Runtime Domain

The runtime domain contains the AI systems that interact with make-it-so during actual operation.

These are called **Agents**.

Examples include:
- LLM-based agents
- MCP-based tool agents
- autonomous research agents
- code execution agents
- systems similar to OpenClaw

These agents are runtime clients of make-it-so. They request actions and receive control decisions.

### Development Domain

The development domain contains AI systems that assist in building the make-it-so project itself.

These are called **AI Coders**.

Examples include:
- Cursor
- Claude Code
- ChatGPT Codex
- other coding assistants

AI Coders are development tools. They are not part of the runtime platform and must not be confused with runtime Agents.

This distinction is important because the project serves runtime agent governance, while AI Coders are merely used to develop the software.

---

## 5. High-Level Interaction Model

At a high level, interaction follows this pattern:

```text
Agent
  │
  │ structured action request
  ▼
make-it-so
  │
  │ policy evaluation
  │ approval routing
  │ audit / logging
  ▼
execution decision
  │
  ├─ allow
  ├─ require approval
  └─ deny
```

In a more complete flow:

```text
Agent
  │
  │ request action
  ▼
make-it-so
  │
  ├─ evaluate request metadata and policy
  ├─ determine approval requirement
  ├─ optionally notify a human via trusted interface
  ├─ collect approval or denial
  └─ return decision and related control data
  ▼
Agent or connected execution system
  │
  └─ executes action only if permitted
```

A key architectural constraint is that make-it-so is a **control and approval layer**, not the component that derives actions from user prompts and not necessarily the component that executes them. It governs execution; it does not replace the agent.

---

## 6. Role of Human Approval

Human approval is central to the system.

The purpose of approval is not to manually review every informational operation. The purpose is to ensure that actions with real external consequences remain under human control.

Typical examples of approval-relevant actions include:
- financial transactions
- bookings and reservations
- execution of workflows with binding effects
- changes to systems or configurations
- access to confidential or personal data
- actions protected by strong authentication or second-factor mechanisms

In this model:
- the agent may prepare the action
- make-it-so may normalize, route, and present it
- the human remains the final decision-maker

This preserves a clear responsibility model:
- the AI does not hold autonomous authority over impactful actions
- the approval layer does not invent intent
- the human explicitly authorizes execution

In some scenarios, approval may also include the controlled release of an execution-enabling artifact, such as a one-time code or similar second-factor credential. Even in those cases, the control principle remains the same: the human explicitly authorizes the final step.

---

## 7. Design Goals

The system is designed with the following goals:

### Safety
Prevent agents from directly executing unsafe, irreversible, or unauthorized actions.

### Human Accountability
Keep responsibility for impactful actions with the human, not the agent.

### Controlled Autonomy
Allow agents to act autonomously where appropriate, but only within governed boundaries.

### Standardization
Provide a common control and approval model across heterogeneous agent frameworks and tool ecosystems.

### Auditability
Create a reliable record of requests, decisions, and approvals.

### Clear Separation of Concerns
Separate:
- agent reasoning
- action governance
- human approval
- execution

### Simplicity
Prefer a minimal, composable architecture over embedding business logic into every integration point.

### Extensibility
Support multiple agent types, approval mechanisms, and execution targets without changing the core system concept.

---

## 8. Non-Goals

make-it-so is not intended to be:

### Not an Agent Framework
It does not replace the runtime agent or define how agents reason internally.

### Not an Execution Engine
It does not inherently execute tools, workflows, transactions, or business operations itself.

### Not a General Workflow Platform
It is not a generic BPM system or orchestration suite for arbitrary enterprise process modeling.

### Not a Full Identity or Device Platform
Identity, device binding, second-factor generation, and similar concerns may be integrated, but they are not the defining purpose of the system.

### Not a Development Tool for AI Coders
Although AI Coders may help build the project, they are outside the runtime scope.

### Not a Policy-Free Approval UI
The system is not merely a notification mechanism. Its purpose is structured control, not passive messaging.

---

## Summary

make-it-so is a human-in-the-loop control plane that governs how runtime AI agents move from **requested action** to **permitted execution**.

It exists to make agentic systems safer, more auditable, and more accountable by ensuring that real-world effects are subject to explicit control rather than implicit autonomy.