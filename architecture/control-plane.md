
# Control Plane Architecture

## Purpose

The make-it-so control plane governs how runtime AI agents request, approve, and execute real-world actions.

It sits between agents and execution systems and ensures that actions follow policy evaluation, human approval (when required), and auditing before execution.

## Core Principle

Agents may *prepare* actions, but they do not autonomously execute actions that change the real world.

The control plane enforces a structured path:

Agent → Action Request → Policy Evaluation → Approval → Execution

## Main Components

### Agent Interface

The Agent Interface exposes APIs (e.g., MCP tools) through which runtime agents submit structured action requests.

Responsibilities:

- receive action requests
- validate request schema
- attach metadata (agent id, user_ref, timestamps)
- forward requests to policy evaluation

### Policy Engine

The Policy Engine evaluates whether an action:

- is allowed automatically
- requires human approval
- must be denied
- must be logged or audited

Policies may consider:

- action type
- risk level
- agent identity
- user context
- environment rules

The Policy Engine must remain deterministic and auditable.

### Approval Service

When policy requires human approval, the request is forwarded to the Approval Service.

Responsibilities:

- present action summary to the human
- collect approve/deny decision
- optionally generate execution credentials (e.g., one‑time code)
- record approval events

### Execution Connectors

Execution Connectors are adapters that interact with external systems such as:

- APIs
- workflow engines
- financial services
- SaaS systems
- infrastructure tools

They perform the actual action *only after approval*.

### Audit Log

All actions are logged with:

- request metadata
- policy decision
- approval result
- execution result

This ensures traceability and accountability.

## Control Flow

1. Agent submits Action Request
2. Control plane validates request
3. Policy Engine evaluates the request
4. If required, Approval Service collects human decision
5. Decision is returned to the agent
6. Agent or execution connector performs the action
7. Result is logged for audit

## Design Characteristics

- stateless interfaces where possible
- explicit approval boundaries
- minimal business logic in adapters
- extensible execution connectors
- strong auditability

The control plane focuses on governance and approval, not business execution.

---

## Related Documents

- [Action Model](action-model.md) — structure of action requests processed by the control plane
- [Request Lifecycle](request-lifecycle.md) — step-by-step flow through all lifecycle stages
- [Component Diagram](component-diagram.md) — visual overview of all components
