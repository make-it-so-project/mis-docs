# Agent Interface Security and Trust Model

## Purpose

This document defines the security and trust model for the data path between an Agent Runtime and the mis-backend.

It specifies how Agent Runtimes authenticate, how their identity is established, how sessions govern runtime requests, and how the mis-backend validates incoming Action Requests.

---

## Scope

This document covers:

- trust boundary between Agent Runtimes and the mis-backend
- Agent Runtime identity model
- Agent Runtime authentication (MVP baseline)
- session_id-led runtime request model
- reconnect and session reactivation
- Action Request validation
- replay and duplicate protection
- audit requirements
- MVP and target security model
- threats and mitigations

---

## Non-goals

This document does NOT define:

- Client Registration or User Registration
- Account Recovery
- the mis-client trust model or approval surface
- API framework or implementation code
- specific protocol bindings (REST, MCP, gRPC)
- signed requests, public-key infrastructure, or mTLS as MVP requirements

---

## Core Principle

The Agent Interface separates **preparation of actions** from **authorization of actions**.

An Agent Runtime may prepare, propose, and submit Action Requests. Authorization remains with policy evaluation and, when required, human approval through the mis-client.

The Agent Interface authenticates the technical integration. It does not trust the Agent Runtime to authorize actions or assert user identity.

---

## Trust Boundary

```
+---------------------------+
|   Agent Runtime           |  — authenticated, untrusted proposer
|   (LLM / MCP / bot)       |
+------------+--------------+
             |
             | Action Request (authenticated)
             v
+---------------------------+
|   Agent Interface         |  — trust boundary
|   (mis-backend)           |
+---------------------------+
             |
             | validated request + backend-derived metadata
             v
+---------------------------+
|   Policy Engine           |
+---------------------------+
```

The trust boundary is at the Agent Interface. The backend authenticates the Agent Runtime, validates the active session, and derives authoritative user/client context. Beyond this boundary, self-asserted identity fields are not trusted.

---

## LLM vs Agent Runtime

The LLM / Sprachmodell is **not** a trusted technical identity. The LLM may generate suggestions, plans, text, or parameters — but it is not the component that authenticates to the mis-backend.

The **Agent Runtime** is the technical component that submits structured requests to the mis-backend. The mis-backend authenticates the Agent Runtime, not the LLM.

An Agent Runtime is authenticated so the mis-backend knows which technical integration is submitting requests. Authentication does not make the Agent Runtime trusted to authorize actions. Authorization remains with policy evaluation and, when required, human approval through the mis-client.

---

## Agent Identity Model

Each registered Agent Runtime has a stable technical identity identified by `agent_id`.

### Registered Agent Data Model

```
registered_agents:
  agent_id           — stable technical identifier
  display_name       — human-readable label
  agent_type         — mcp_server | llm_agent | automation_agent | custom
  owner_context      — deployment or operator context
  status             — active | revoked
  auth_method        — api_key | oauth_client_credentials | signed_requests | mtls
  created_at
  last_seen_at
  revoked_at
```

### Identity Rules

- `agent_id` MUST be derived from the authenticated Agent Runtime context.
- `agent_id` MUST NOT be self-asserted by the LLM or accepted only because it appears in the request body.
- Unknown or revoked agents MUST be rejected.
- Agent identity supports audit, rate limiting, abuse prevention, and policy decisions.
- Agent identity does **not** authorize execution.

---

## MVP Agent Authentication

The MVP authentication baseline requires:

- **HTTPS/TLS** for all Agent Runtime to mis-backend communication.
- **Registered agents**: each Agent Runtime has a `registered_agents` entry.
- **Per-agent credential**: each Agent Runtime authenticates with a per-agent credential, e.g. API key / bearer token or OAuth2 client credentials.
- **Rejection of unknown or revoked agents**: requests from agents not in `registered_agents` or with status `revoked` MUST be rejected.

Signed requests and mTLS are deferred to the target architecture.

### MVP Balance

Since approval-required actions are finalized by the user through the mis-client, Agent Runtimes are treated as **untrusted proposers**. Minimal Agent Interface security is still required to prevent:

- anonymous abuse
- session injection
- replay and duplicate execution
- credential return to wrong parties
- audit gaps

---

## Session-ID-Led Runtime Model

`session_id` is the leading runtime authority for Action Requests.

### Core Model

- `session_id` is the leading runtime authority for Action Requests.
- `agent_id` identifies the authenticated Agent Runtime.
- `user_id` and `client_id` identify the user/client binding selected through Session Connect.
- The Agent Runtime MAY persist `user_id` and `client_id`, but only as **reconnect hints**.
- Stored `user_id` and `client_id` are NOT sufficient to submit authorized Action Requests.
- The Agent Runtime MUST obtain an ACTIVE `session_id` from the mis-backend before submitting Action Requests.
- The mis-backend MUST resolve `user_id` and `client_id` from server-side session binding.
- The mis-backend MUST NOT rely on self-asserted `user_id` or `client_id` in Action Requests.

### Active Session Data Model

```
active_sessions:
  session_id     — unique runtime session identifier
  agent_id       — authenticated Agent Runtime
  user_id        — resolved user identity
  client_id      — resolved approval client
  status         — active | expired | revoked
  created_at
  expires_at
  last_seen_at
```

### Validation Requirements

For each Action Request, the mis-backend MUST:

1. Authenticate the Agent Runtime and derive `agent_id`.
2. Validate `session_id`.
3. Verify the session exists and is ACTIVE.
4. Verify the session belongs to the authenticated `agent_id`.
5. Resolve `user_id` and `client_id` from the server-side session binding.
6. Reject requests with missing, expired, unknown, or mismatched session bindings.

---

## Reconnect / Session Reactivation

Reconnect is the mechanism by which an Agent Runtime uses stored `user_id` and `client_id` as hints to obtain a new ACTIVE `session_id`.

### Reconnect Data Flow

```
Agent Runtime (persistent storage):
  user_id          — reconnect hint
  client_id        — reconnect hint

Agent Runtime requests reactivation:
  authenticated as agent_id
  submits user_id, client_id, and local session reference / new session context

mis-backend:
  authenticates Agent Runtime → agent_id
  validates user_id and client_id
  validates client is ACTIVE and belongs to user_id
  evaluates reconnect policy
  creates new ACTIVE session_id if allowed
  stores session_id → (agent_id, user_id, client_id)
  returns session_id to Agent Runtime
```

### Reconnect Modes

Two reconnect modes are defined:

**notify_only**

- The mis-backend creates a new ACTIVE `session_id` when validation succeeds.
- The mis-client receives a security notification.
- The notification is inform-only.

**approval_required**

- The mis-backend creates a pending reactivation request.
- The mis-client receives an approval request for session reactivation.
- The new `session_id` becomes ACTIVE only after approval.

### Reconnect Policy

- Reconnect mode MAY be user-configurable.
- Reconnect mode MAY be policy-configurable.
- Reconnect SHOULD default to `notify_only` for low-risk MVP use.
- Reconnect MAY be forced to `approval_required` for higher-risk agents, suspicious patterns, changed `agent_id`, long inactivity, or policy-defined sensitive contexts.

Reconnect reactivates a runtime session for an already registered client. It is not a client registration flow.

---

## Action Request Validation

### MVP Action Request Body (conceptual)

```json
{
  "session_id": "string",
  "action_type": "string",
  "summary": "human readable description",
  "risk_hint": "optional risk level",
  "details": {},
  "request_id": "unique request id",
  "idempotency_key": "unique idempotency key"
}
```

### Backend-Derived Metadata

The following fields are derived by the mis-backend and MUST NOT be accepted as trusted request-body input:

- `agent_id` — derived from authenticated Agent Runtime context
- `user_id` — resolved from server-side session binding
- `client_id` — resolved from server-side session binding
- `received_at` — assigned by the mis-backend

### Validation Rules

- `agent_id` MUST NOT be accepted as trusted request-body input.
- `user_id` MUST NOT be accepted as trusted request-body input.
- `client_id` MUST NOT be accepted as trusted request-body input.
- If included for debugging or UX, these values MUST be validated against server-side state and ignored as authority.
- `session_id` is required for runtime Action Requests.
- `request_id` and `idempotency_key` are required for replay and duplicate protection.

---

## Replay and Duplicate Protection

- Requests MUST include `request_id`.
- Requests MUST include `idempotency_key`.
- Requests SHOULD include a timestamp, or the backend MUST assign and audit `received_at`.
- The mis-backend MUST reject duplicate `request_id` values within an appropriate replay window.
- The mis-backend MUST use `idempotency_key` to prevent duplicate execution.
- Replay and duplicate checks SHOULD be scoped to `agent_id + session_id`.

---

## Audit Requirements

The mis-backend MUST log:

- all incoming Action Requests with `request_id`, `session_id`, and backend-derived `agent_id`
- authentication failures (unknown or revoked agents)
- session validation failures (missing, expired, mismatched sessions)
- reconnect requests and outcomes
- replay and duplicate rejection events
- all policy decisions and approval outcomes

Audit logs support traceability, abuse detection, and governance.

---

## MVP Baseline

| Requirement | MVP |
|---|---|
| Transport security | HTTPS/TLS |
| Agent registration | Required |
| Per-agent credential | Required (API key / OAuth2 client credentials) |
| session_id validation | Required |
| Request body validation | Required |
| request_id + idempotency_key | Required |
| Replay window | Required |
| Duplicate detection | Required |
| Audit logging | Required |
| Signed requests | Deferred |
| mTLS | Deferred |
| Public key infrastructure | Deferred |

---

## Target Security Model

The target architecture MAY add:

- **Signed Agent Runtime requests**: per-agent signing of request bodies using asymmetric keys.
- **Per-agent public keys**: each registered Agent Runtime has a registered public key.
- **Key rotation**: agents may rotate their signing keys; old keys are retired.
- **Optional mTLS**: for enterprise or self-hosted deployments requiring mutual TLS.
- **Stronger replay protection**: signed timestamps and body hashes.

These are target capabilities, not MVP requirements.

---

## Threats and Mitigations

| Threat | Mitigation |
|---|---|
| Network MITM modifies request | HTTPS/TLS; future signed requests |
| Anonymous actor submits fake requests | Registered agents and per-agent credentials |
| Agent submits request for another user's session | session_id bound to agent_id; backend resolves user_id/client_id |
| Agent self-asserts user_id/client_id | Backend ignores self-asserted user/client authority |
| Replayed request | request_id, timestamp/received_at, replay window |
| Duplicate execution | idempotency_key |
| Stolen agent credential | Rate limits, audit, revocation; future signed requests / key rotation |
| Approval spam | Registered agents, rate limits, policy evaluation |
| LLM prompt injection creates malicious proposal | Agent is untrusted proposer; policy + human approval remain authority |
| Session reactivation abuse | Reconnect policy: notify_only or approval_required |

---

## Related Documents

- [Session Connect](session-connect.md) — session binding mechanism
- [Action Model](action-model.md) — Action Request structure
- [Control Plane Architecture](control-plane.md) — overall control flow
- [Component Diagram](component-diagram.md) — component responsibilities
- [Request Lifecycle](request-lifecycle.md) — lifecycle stages
- [Notification Channel](notification-channel.md) — secondary notification channel
- [Use Case: JIT Authorization for MCP Servers](../use-cases/use-case-jit-authorization-mcp-servers.md) — primary use case
- [ADR-0005: Session Connect Mechanism](../adr/0005-session-connect-mechanism.md) — session connect decision
- [ADR-0011: Agent Interface Trust Model](../adr/0011-agent-interface-trust-model.md) — this decision record
