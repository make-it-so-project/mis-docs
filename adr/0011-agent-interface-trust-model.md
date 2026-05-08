# ADR-0011: Agent Interface Trust Model

## Status

Accepted

## Date

2026-05-08

## Context

Existing architecture documents mention `agent_id` in the Action Model, Control Plane, and Component Diagram, but do not define where `agent_id` comes from or how the Agent Runtime authenticates to the mis-backend. Session Connect (ADR-0005) defines the user/client binding but does not define Agent Runtime identity.

The system needs a clear trust model for the Agent Interface that:

- authenticates Agent Runtimes without overengineering the MVP
- defines `session_id` as the leading runtime authority
- prevents anonymous abuse, session injection, replay, and duplicate execution
- preserves the human approval model (Agent Runtime proposes; user approves or denies)
- keeps a target path open for stronger security (signed requests, mTLS)

## Decision Drivers

- Avoid unnecessary Fort Knox in the MVP — approval-required actions are already protected by human approval through the mis-client.
- Preserve the human approval model: Agent Runtime proposes; user approves or denies.
- Prevent anonymous API abuse and approval spam.
- Prevent session injection.
- Ensure auditability by stable `agent_id`.
- Ensure backend, not the Agent Runtime, derives authoritative `user_id` and `client_id`.
- Prevent replay and duplicate execution.
- Keep target path open for signed requests and mTLS.

## Considered Options

### Option 1: No Agent Runtime authentication

Rely only on HTTPS and human approval for protection.

**Assessment:** Too weak. Allows anonymous abuse, approval spam, audit gaps, session injection attempts, and unsafe auto-allow future behavior.

### Option 2: Require strong signed requests or mTLS from day one

**Assessment:** Strong but too heavy for MVP and external integrations. Operationally complex to provision and manage per-agent keys and certificates.

### Option 3: MVP with authenticated but untrusted Agent Runtimes (selected)

MVP baseline:
- per-agent credentials (API key / OAuth2 client credentials)
- server-side session binding
- replay and duplicate protections
- signed requests / mTLS deferred to target architecture

**Assessment:** Correct balance of security and MVP pragmatism.

## Decision

Adopt Option 3.

- Agent Runtimes are **authenticated but untrusted proposers**.
- `agent_id` is derived from Agent Runtime authentication, not from self-asserted request body fields.
- `session_id` is the leading runtime authority for Action Requests.
- `user_id` and `client_id` may be stored by the Agent Runtime only as **reconnect hints**.
- The mis-backend resolves `user_id` and `client_id` from server-side session binding.
- The mis-backend rejects anonymous, revoked, unknown, expired, replayed, or mismatched requests.
- MVP requires HTTPS/TLS, registered agents, per-agent credentials, `request_id`, `idempotency_key`, session validation, and audit logging.
- Target architecture may add signed requests, public keys, key rotation, and mTLS.

## Consequences

### Positive

- avoids overengineering the MVP
- prevents anonymous abuse
- preserves human approval as final authority
- provides auditability through stable `agent_id`
- prepares target hardening path (signed requests, mTLS)

### Negative

- per-agent credentials must be provisioned and protected
- stolen credentials remain a risk until the target signed request model
- reconnect policy requires careful UX and configuration
- more validation state in mis-backend

## Rationale Summary

The Agent Runtime does not authorize actions. It only submits proposals. However, the backend must know which technical integration is submitting proposals and must ensure requests belong to an ACTIVE server-side session. The selected option provides the minimum viable security without overengineering, while keeping a clear migration path to stronger protection in the target architecture.
