
# Component Diagram

## Overview

This document describes the main architectural components of the make-it-so system and their relationships.

make-it-so acts as a control plane between runtime AI agents and systems that perform real-world actions.

## High-Level Architecture

```
+-------------------+
|   Runtime Agents  |
| (LLM / MCP tools) |
+---------+---------+
          |
          | Action Request
          v
+-------------------------+
|      make-it-so         |
|       Control Plane     |
+-------------------------+
| Agent Interface         |
| Policy Engine           |
| Approval Service        |
| Audit Log               |
+-----------+-------------+
            |
            | Approved Action
            v
+-------------------------+
|   Execution Connectors  |
+-----------+-------------+
            |
            v
+-------------------------+
| External Systems / APIs |
+-------------------------+
```

## Core Components

### Agent Interface

The Agent Interface is the entry point for runtime agents.

Responsibilities:
- receive structured action requests
- validate request schema
- attach metadata (agent_id, timestamps)
- forward requests to the Policy Engine

Typical implementations may expose MCP tools or REST APIs.

---

### Policy Engine

The Policy Engine evaluates incoming action requests.

It determines whether the request:

- is automatically allowed
- requires human approval
- must be denied
- must be logged or audited

Policy decisions may consider:

- action type
- agent identity
- user context
- configured risk policies
- environment settings

The Policy Engine must remain deterministic and auditable.

---

### Approval Service

If an action requires human approval, the request is forwarded to the Approval Service.

Responsibilities:

- present action details to a human reviewer
- collect approval or rejection
- optionally generate execution credentials
- record approval events

Human approvals typically occur through a trusted device such as a mobile application.

---

### Execution Connectors

Execution Connectors interact with external systems.

Examples include:

- APIs
- SaaS services
- financial systems
- infrastructure tools
- workflow engines

Execution connectors perform the action only after the control plane authorizes execution.

---

### Audit Log

All important events must be recorded in the audit log:

- action requests
- policy decisions
- approval events
- execution results

The audit log ensures traceability and supports governance requirements.

---

## Design Principles

- minimal coupling between components
- explicit approval boundaries
- strong auditability
- extensible connectors
- stateless interfaces where possible
