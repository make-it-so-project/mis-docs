
---
read_when: defining or modifying action request structures, lifecycle states, or execution credentials
---

# Action Model

## Purpose

The Action Model defines the structure and lifecycle of action requests processed by make-it-so.

It standardizes how AI agents describe actions and how the control plane evaluates and governs them.

## Action Definition

An action represents a request by an AI agent to perform an operation that may affect external systems or real-world state.

Examples include:

- executing a workflow
- triggering a payment
- booking a reservation
- modifying infrastructure
- accessing protected data

## Action Request Structure

Action Requests are session-led. The request body carries fields from the
Agent Runtime; authoritative identity and context fields are derived by the
mis-backend from the authenticated session binding.

### Request Body (from Agent Runtime)

```
{
  "session_id": "string",
  "action_type": "string",
  "summary": "human readable description",
  "risk_hint": "optional risk level",
  "details": { },
  "request_id": "unique request id",
  "idempotency_key": "unique request key"
}
```

### Backend-Derived Metadata

The following fields are derived by the mis-backend and MUST NOT be
accepted as trusted request-body input:

- **agent_id** — derived from authenticated Agent Runtime context
- **user_id** — resolved from server-side session binding
- **client_id** — resolved from server-side session binding
- **received_at** — assigned by the mis-backend

### Key Fields

**session_id**
Required field. The leading runtime authority for Action Requests.
The mis-backend validates that the session exists, is ACTIVE, and is
bound to the authenticated agent_id. user_id and client_id are resolved
from this session binding.

**action_type**
Defines the type of operation requested.

**summary**
Human-readable description presented during approval.

**risk_hint**
Optional indicator to assist policy evaluation.

**details**
Structured parameters for the target system.

**request_id**
Unique request identifier. Supports replay detection and audit.
The mis-backend MUST reject duplicate request_id values within an
appropriate replay window.

**idempotency_key**
Ensures duplicate actions are not executed twice.
The mis-backend MUST use idempotency_key to prevent duplicate execution.

### Self-Asserted Identity Fields

agent_id, user_id, and client_id are NOT accepted as trusted request-body
inputs. If included in the request body for debugging or UX purposes, these
values MUST be validated against server-side state and ignored as authority.

## Action Lifecycle

Actions move through a defined lifecycle:

```
CREATED
  ↓
POLICY_EVALUATION
  ↓
APPROVED | DENIED | AUTO_ALLOWED
  ↓
EXECUTED
  ↓
COMPLETED
```

### Created

The agent submits a structured action request.

### Policy Evaluation

The control plane evaluates policies to determine whether approval is required.

### Approved / Denied

If approval is required, a human reviews the request via a trusted interface.

### Executed

Once permitted, the action is executed through an execution connector.

### Completed

Execution results are recorded in the audit log.

## Optional Execution Credentials

Some actions require additional credentials, such as one‑time codes.

If human approval generates such a credential, it is returned alongside the approval decision:

```
{
  "status": "APPROVED",
  "approval_token": "...",
  "action_credential": {
    "type": "one_time_code",
    "value": "123456",
    "expires_at": "timestamp"
  }
}
```

The control plane transports these credentials but does not generate or persist them.

## Design Principles

- actions must be explicitly described
- approval decisions must be auditable
- execution credentials are optional and short-lived
- action schemas must remain extensible

The Action Model ensures that AI agents interact with the control plane using predictable, structured requests.

---

## Related Documents

- [Request Lifecycle](request-lifecycle.md) — detailed description of all lifecycle stages an action passes through
- [Control Plane Architecture](control-plane.md) — how the control plane enforces the action path
- [Component Diagram](component-diagram.md) — components that process action requests
- [Agent Interface Security](agent-interface-security.md) — session validation, authentication, and trust model
- [Session Connect](session-connect.md) — how session_id is established and bound to agent/user/client
