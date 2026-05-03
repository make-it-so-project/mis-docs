
---
read_when: working on approval flows, policy evaluation, action governance, or the mis-client
---

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
- collect approve/deny decision via the mis-client (trusted approval surface)
- optionally generate execution credentials (e.g., one-time code)
- record approval events
- optionally trigger a secondary notification channel to alert the user

The approval decision itself always travels through the mis-client.
Secondary notification channels are inform-only and have no role in the
approval flow. See [notification-channel.md](notification-channel.md).

#### Client Validation Before Approval Routing

Before routing an approval request to a `client_id`, the Approval Service
MUST verify:

- the `client_id` exists in `registered_clients`
- the client belongs to the resolved `user_id`
- the client status is ACTIVE
- the client satisfies any policy requirements for `client_type` and
  `assurance_level` applicable to the requested action

Revoked clients MUST NOT receive approval requests.

If the resolved client is not ACTIVE or does not satisfy policy requirements,
the request MUST be rejected. The agent receives an error directing the
user to re-connect or re-register as appropriate.

### Secondary Notification Channel (Optional)

The control plane may forward event notifications to an external channel
(e.g., mobile push, messaging platform) to improve user awareness.

This is strictly an outbound, inform-only path.

The channel MUST NOT receive or influence approval decisions.
Any approval action redirects the user to the mis-client.

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
4. If required, Approval Service routes request to mis-client for human decision
5. Optionally, a secondary notification channel informs the user out-of-band
6. Decision is returned to the agent
7. Agent or execution connector performs the action
8. Result is logged for audit

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
- [Notification Channel](notification-channel.md) — secondary notification channel definition and constraints
- [Client Registration](client-registration.md) — client lifecycle and status model validated before approval routing
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — client identity verification requirements
