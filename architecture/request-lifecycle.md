
# Request Lifecycle

## Overview

This document describes the lifecycle of an action request within make-it-so.

An action request represents an operation that an AI agent wants to perform and that may affect external systems.

The lifecycle ensures that every action passes through governance, approval, and auditing steps.

## Lifecycle Stages

```
Agent Request
     |
     v
REQUEST_CREATED
     |
     v
POLICY_EVALUATION
     |
     +---------> DENIED
     |
     v
APPROVAL_REQUIRED (optional)
     |
     v
APPROVED
     |
     v
EXECUTION
     |
     v
COMPLETED
```

## Stage Descriptions

### Request Created

The AI agent submits a structured action request to make-it-so.

The request contains metadata such as:

- action type
- summary
- agent identifier
- user reference
- optional parameters

The control plane validates the request schema and records the request.

---

### Policy Evaluation

The Policy Engine evaluates the request.

Possible outcomes:

- AUTO_ALLOWED – execution may proceed without approval
- APPROVAL_REQUIRED – human approval is required
- DENIED – the request must not be executed

Policy evaluation ensures consistent governance across agents.

---

### Approval Required

If approval is required, the request is sent to the Approval Service.

The human reviewer receives:

- action summary
- contextual information
- risk indicators

The reviewer can approve or reject the action.

In some cases approval may also produce a short‑lived execution credential (e.g. a one‑time code).

---

### Approved

Once approved, the control plane marks the request as executable.

The decision may include:

- approval token
- optional execution credential
- expiration timestamp

These artifacts allow the agent or execution connector to proceed.

---

### Execution

The action is executed through an Execution Connector.

The connector interacts with the external system responsible for performing the requested operation.

Execution occurs only after the control plane authorizes it.

---

### Completed

After execution:

- the result is recorded
- the lifecycle state becomes COMPLETED
- audit logs capture the entire history

This ensures traceability for governance and debugging.

## Failure Handling

If execution fails:

- the request state may become FAILED
- error information is logged
- retry policies may apply depending on action type

## Design Goals

The lifecycle model ensures:

- explicit governance boundaries
- human accountability for impactful actions
- auditable execution history
- consistent behavior across heterogeneous agents

---

## Related Documents

- [Action Model](action-model.md) — structure and field definitions of an Action Request
- [Component Diagram](component-diagram.md) — components involved in each lifecycle stage
- [Control Plane Architecture](control-plane.md) — architectural principles behind the control flow
