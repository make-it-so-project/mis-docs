---
read_when: working on agent session initialization, user identification, client routing, or the Connect flow
---

# Session Connect

## Purpose

The Session Connect mechanism allows a runtime agent to identify the initiating
user and the user's preferred approval client (mis-client) for the duration of
a session.

It establishes the binding that enables the make-it-so control plane to route
approval requests to the correct device without requiring the agent to manage
client details directly.

---

## Problem

When an agent submits an action request to make-it-so, the control plane must
know:

- **who** initiated the request (user identity)
- **where** to route the resulting approval request (client device)

Without an explicit connect step, the agent has no way to supply this
information, and the control plane cannot route approvals correctly.

---

## Boundary: Session Connect and Client Registration

Session Connect operates on `registered_clients` — it selects a client
from that list. It MUST NOT register new clients.

A `client_id` used in Session Connect MUST already exist in
`registered_clients` with status ACTIVE. The mis-backend MUST reject
Connect attempts that reference a non-existent, PENDING, or REVOKED client.

Client Registration — the process by which a client enters `registered_clients`
— is a separate, more strongly secured flow. See [client-registration.md](client-registration.md).

### Agent Runtime Boundary

Session Connect binds an authenticated Agent Runtime session to a user/client
pair. It does not define Agent Runtime registration or credential provisioning.
Agent Runtime authentication is defined in [agent-interface-security.md](agent-interface-security.md)
and [ADR-0011](../adr/0011-agent-interface-trust-model.md).

Session Connect does not register new clients. It does not grant or modify
Agent Runtime identity. Both client registration and agent registration are
separate, more strongly secured flows.

### Agent Runtime Boundary

Session Connect binds an authenticated Agent Runtime session to a user/client
pair. It does not define Agent Runtime registration or credential provisioning.
Agent Runtime authentication is defined in [agent-interface-security.md](agent-interface-security.md)
and [ADR-0011](../adr/0011-agent-interface-trust-model.md).

Session Connect does not register new clients. It does not grant or modify
Agent Runtime identity. Both client registration and agent registration are
separate, more strongly secured flows.

---

## Core Concept

At the start of a session, the user performs a **Connect** that links the
current authenticated Agent Runtime session to their identity and a chosen
mis-client device.

After a successful Connect:

- the mis-backend stores `session_id → (agent_id, user_id, client_id,
  status, created_at, expires_at)`
- the Agent Runtime holds an ACTIVE `session_id`
- the Agent Runtime MAY persist `user_id` and `client_id` only as
  **reconnect hints**
- later Action Requests use `session_id`
- the mis-backend resolves `user_id` and `client_id` from the server-side
  session binding for every subsequent Action Request

The Agent Runtime does not manage client details. Client routing is resolved
dynamically by the mis-backend on each request.

---

## Data Model

```
mis-backend:

  registered_clients:
    user_id  →  [client_id, ...]          (account-level, managed separately)

  active_sessions:
    session_id  →  (agent_id, user_id, client_id, status, created_at, expires_at)
                                          (session_id is the runtime authority)

agent runtime:
  active:
    session_id
  reconnect_hints:
    user_id
    client_id
```

A user may have multiple registered clients. Each Connect explicitly selects
one client for the session. All approval requests within that session are
routed to the selected client.

The Agent Runtime may store `user_id` and `client_id` as reconnect hints
only. These values are not authoritative request credentials and are not
sufficient to submit authorized Action Requests. The Agent Runtime MUST
obtain an ACTIVE `session_id` from the mis-backend before submitting
Action Requests.

The Agent Runtime may store `user_id` and `client_id` as reconnect hints
only. These values are not authoritative request credentials and are not
sufficient to submit authorized Action Requests. The Agent Runtime MUST
obtain an ACTIVE `session_id` from the mis-backend before submitting
Action Requests.

---

## Connect Flow

The Connect flow establishes a new session binding for an authenticated
Agent Runtime.

```
mis-client (User X, Client Y)
  │  User triggers "Open Channel"
  │  mis-client fetches a short-lived pairing code from mis-backend
  │  Code encodes: (user_id=X, client_id=Y)
  │  Code is valid for ~30 seconds
  │  mis-backend ensures uniqueness within a ±30 second window
  ▼
  Code displayed to user: e.g., "durstiger-affe"

Agent Runtime
  │  Agent asks user for the Connect code ("Clearance Code")
  │  User types code into the chat
  ▼
  Agent Runtime  →  POST /connect { code, session_id }
  ▼
mis-backend
  │  Authenticates Agent Runtime → derives agent_id
  │  Validates code (not expired, not already used)
  │  Resolves (user_id=X, client_id=Y) from code
  │  Stores: session_id → (agent_id, user_id, client_id, status=active, ...)
  │  Rejects unknown session_ids
  ▼
  Returns: session_id and user/client context to Agent Runtime

Agent Runtime
  │  Stores session_id as active runtime context
  │  May store (user_id, client_id) as reconnect hints only
  ▼
  Session is established. All Action Requests include session_id.
  The backend resolves user_id and client_id from the session binding.
```

### Pairing Code Properties

- human-readable, language-adapted wordpair (e.g., "durstiger-affe")
- short TTL (~30 seconds)
- unique within a ±30 second window across all active users
- generated by the mis-backend, fetched and displayed by the mis-client
- single-use

---

## Session Reactivation / Reconnect Flow

When a user opens a new chat session and the Agent Runtime has a stored
connection from a previous session, the Agent Runtime may offer to
reactivate it using stored reconnect hints without requiring a new
pairing code.

```
Agent Runtime (new chat session)
  │  Detects stored reconnect hints (user_id, client_id) from previous session
  │  Asks user: "Soll ich mit deinem bisherigen Channel weitermachen?"
  ▼
  User confirms

Agent Runtime  →  POST /session/reactivate { user_id, client_id, new_session_context }
  │  Request is authenticated as agent_id
  ▼
mis-backend
  │  Authenticates Agent Runtime → agent_id
  │  Validates that (user_id, client_id) is still a registered active pair
  │  Validates client is ACTIVE and belongs to user_id
  │  Evaluates reconnect policy
  │  If invalid (client deregistered): rejects → agent recommends fresh Connect
  │  If valid, creates new ACTIVE session_id → (agent_id, user_id, client_id)
  │  Notifies mis-client (mode-dependent):
  │    notify_only: security notification sent (inform-only)
  │    approval_required: approval request sent; session becomes ACTIVE only after approval
  ▼
  Session reactivation confirmed.
```

### Reconnect Modes

Two modes are defined:

- **notify_only**: mis-backend creates ACTIVE `session_id` on validation
  success; mis-client receives inform-only security notification.
- **approval_required**: mis-backend creates pending reactivation request;
  mis-client must approve; session becomes ACTIVE only after approval.

Reconnect mode MAY be user-configurable or policy-configurable.
It SHOULD default to `notify_only` for low-risk MVP use.
It MAY be forced to `approval_required` for higher-risk agents.

The mis-client notification on reactivation is a security signal. It informs
the user that their identity is being reused in a new agent session and allows
them to detect unexpected reuse.

---

## Override Semantics

A new Connect for the same agent session replaces the previous session binding
in the mis-backend. The previous `(session_id → client_id)` entry is
overwritten.

This is intentional: the user explicitly chooses which client to use for a
session. A new Connect is a deliberate re-selection, not an accumulation.

Override events are logged in the mis-backend (pseudonymized) for audit
purposes.

---

## Unknown Session Rejection

If the mis-backend receives an action request with a `session_id` for which
no active binding exists, it returns an error to the agent:

> "No channel established. Please re-connect from your Tricorder."

The agent surfaces this message to the user and recommends initiating a
fresh Connect.

---

## Security Properties

| Property | Guarantee |
|---|---|
| Agent authentication | Agent Runtime is authenticated; agent_id is derived from auth context |
| Pairing code TTL | code expires within ~30 seconds |
| Code uniqueness | no collision within ±30 second window |
| Session validation | unknown session_ids are rejected |
| session_id as runtime authority | session_id is required for Action Requests; backend resolves user/client from session binding |
| user_id/client_id as reconnect hints only | stored values are NOT authoritative request credentials |
| Session-agent binding | session_id is bound to authenticated agent_id; mismatched sessions are rejected |
| Client selection | each session binds to exactly one client |
| Agent isolation | agent holds session_id; user_id/client_id are backend-resolved |
| Reconnect policy | reconnect modes: notify_only or approval_required |
| Reactivation safety | mis-client is notified on session reactivation; approval may be required |
| Override logging | all binding changes are logged |

The pairing code itself is not a security credential. The trust anchor is the
authenticated mis-client that generates it. The approval flow remains the
primary security boundary.

---

## Related Documents

- [ADR-0005: Session Connect Mechanism](../adr/0005-session-connect-mechanism.md) — decision record for this mechanism
- [ADR-0011: Agent Interface Trust Model](../adr/0011-agent-interface-trust-model.md) — Agent Runtime identity and session-led request model
- [Agent Interface Security](agent-interface-security.md) — Agent Runtime trust model, authentication, and session validation
- [Client Registration](client-registration.md) — how a client_id enters registered_clients before Connect can use it
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key model for registered clients
- [Design Language](../design/design-language.md) — user-facing copy and TNG naming conventions for Connect interactions
- [Control Plane Architecture](control-plane.md) — how the control plane uses session_id to route approvals
- [Use Case: JIT Authorization for MCP Servers](../use-cases/use-case-jit-authorization-mcp-servers.md) — primary use case where Connect is a precondition
- [Notification Channel](notification-channel.md) — secondary notification channel used for reactivation alerts
