
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

A typical action request contains the following fields:

```
{
  "action_type": "string",
  "summary": "human readable description",
  "agent_id": "identifier of requesting agent",
  "user_ref": "logical user identifier",
  "risk_hint": "optional risk level",
  "details": { },
  "idempotency_key": "unique request key"
}
```

### Key Fields

**action_type**  
Defines the type of operation requested.

**summary**  
Human-readable description presented during approval.

**agent_id**  
Identifier of the requesting runtime agent.

**user_ref**  
Logical user context associated with the request.

**risk_hint**  
Optional indicator to assist policy evaluation.

**details**  
Structured parameters for the target system.

**idempotency_key**  
Ensures duplicate actions are not executed twice.

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
