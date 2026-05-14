---
read_when: working on MCP Server enterprise access, Just-in-Time privileged access, scoped sessions, enterprise enforcement patterns, or MCP audit correlation
---

# MCP Server JIT Access Building Blocks

## Purpose

This document captures the architecture topic and initial building blocks for realizing
Just-in-Time privileged access for MCP Servers in enterprise environments.

The document distinguishes two enforcement models:

- **Integration-Enforced JIT** — the pragmatic MVP pattern, where the MCP Server itself
  acts as the policy enforcement point.
- **Infrastructure-Enforced JIT** — the stronger target pattern, where enterprise
  infrastructure enforces the approved scope and access path.

---

## Status

**Placeholder / to be elaborated.**

This document captures the topic and initial principles. It is not yet a complete
implementation specification. It does not define a concrete IAM, gateway, proxy,
privileged access management, or network enforcement technology. It does not replace
ADR-0002 or the existing MCP Server JIT authorization use case.

---

## Scope

This document covers:

- MCP Server as Agent Runtime
- make-it-so approval as the authorization gate
- short-lived scoped access sessions
- MVP enforcement assumptions and constraints
- target enforcement patterns
- required audit correlation between mis-backend and MCP Server
- building blocks for both enforcement models
- security risks and mitigations
- open questions for future design

---

## Non-Goals

This document does not define:

- implementation code
- a concrete enterprise IAM product, gateway product, proxy product, or privileged access
  management product
- enterprise network design
- a hard requirement for enterprise network or IAM changes in the MVP
- any weakening of human approval requirements
- changes to Session Connect
- changes to the Agent Interface trust model
- changes to ADR-0002

---

## Core Principle

An MCP Server must not return enterprise data or perform approved enterprise operations for
an Agent Runtime unless a valid make-it-so approval and scoped access context exists.

Additional required properties:

- The MCP Server acts as an Agent Runtime. It must be registered and authenticate to the
  mis-backend as `agent_id`.
- The MCP Server is an **authenticated but untrusted proposer**. Authentication does not
  authorize the MCP Server to return protected data.
- The mis-backend validates `session_id` and resolves `user_id`/`client_id` from the
  server-side session binding.
- Human approval happens through the mis-client. The MCP Server cannot approve its own
  requests.
- Every approval and every enterprise data access or response event must be auditable and
  correlatable.

---

## MVP Model: Integration-Enforced JIT

**Definition:**
A deployment pattern where the MCP Server itself enforces make-it-so approval before returning
enterprise data, even though the underlying enterprise access may already be technically
available to the MCP Server.

In the MVP, the MCP Server may already have broad technical access inside the enterprise
environment. make-it-so does not necessarily control the enterprise IAM or network path.
The MCP Server becomes the local policy enforcement point and must check make-it-so approval
before returning protected enterprise data to the calling agent.

**This model is pragmatic but honest: the MCP Server has standing technical access to Zone 1
resources. The zero-trust property depends on the MCP Server correctly enforcing make-it-so
approval. It is not infrastructure-enforced.**

Required behavior of the MCP Server in this model:

- The MCP Server may have standing technical access to Zone 1 resources.
- make-it-so does not necessarily control the enterprise IAM or network path in the MVP.
- The MCP Server MUST request make-it-so approval before returning protected enterprise data.
- The MCP Server MUST check that the approval matches the requested scope, `session_id`,
  `agent_id`, `user_id`, and TTL.
- The MCP Server MUST NOT return enterprise data after the approval TTL has expired.
- The MCP Server MUST log both the approval reference and the enterprise data access and
  response event.
- This model is easier to adopt but depends on correct MCP Server implementation.

---

## Target Model: Infrastructure-Enforced JIT

**Definition:**
A deployment pattern where enterprise infrastructure enforces the approved scope, TTL, and
access path, so the MCP Server cannot access or return protected enterprise data outside the
approved context.

In the target model, the MCP Server does not have standing enterprise privileges. Access is
technically activated only after make-it-so approval.

Possible realization patterns (none mandated at this stage):

- temporary enterprise identity issued after approval
- scoped access token with short TTL
- temporary role assignment in enterprise identity management
- enterprise policy enforcement point that gates data access
- gateway or proxy that enforces approved scope
- privileged access management integration
- network-level or service-level allow rule activated on approval
- just-in-time service account with constrained permissions

Required properties of the target model:

- Stronger security than Integration-Enforced JIT.
- Lower dependence on MCP Server self-enforcement.
- Requires deeper enterprise integration.
- May be harder to deploy in the MVP because administrators may decline network or IAM changes.
- Remains the target direction for higher assurance deployments.

---

## Building Blocks

| Building Block | Purpose | MVP | Target |
|---|---|---|---|
| MCP Server | Enterprise integration endpoint and Agent Runtime | May have standing Zone 1 access; enforces make-it-so approval before returning data | No standing privileges; receives temporary scoped access |
| Agent Runtime Identity | Stable `agent_id` and authentication to mis-backend | Per-agent credential (API key / OAuth2 client credentials) | Signed requests or stronger enterprise authentication where appropriate |
| Session Binding | `session_id` → `agent_id` / `user_id` / `client_id` | Server-side mis-backend binding | Same; potentially linked to enterprise access context |
| Approval Request | Human decision through mis-client | Always required for enterprise access | May support policy-based or delegated approvals in a later phase |
| Scoped Access Context | Approved scope, TTL, resource boundaries | Validated by MCP Server before returning data | Enforced by enterprise IAM, gateway, proxy, or policy enforcement point |
| Audit Correlation | Trace approval to enterprise access and response event | MCP Server and mis-backend both log correlation IDs | Enterprise logs also include correlation IDs |
| Policy Enforcement Point | Component enforcing approved scope | MCP Server | Enterprise infrastructure or dedicated gateway or proxy |

---

## Required Audit Correlation

For every enterprise data response returned by the MCP Server to an Agent Runtime, there must
be a traceable make-it-so approval or a clear rejection or expiration record.

The following fields must be correlatable between mis-backend and MCP Server logs:

- `approval_request_id`
- `action_id` or `access_request_id`
- `session_id`
- `agent_id`
- MCP Server identity
- `user_id`
- `client_id`
- approved scope
- approved TTL and expiration timestamp
- enterprise resource category
- operation type (e.g. read-only in MVP)
- timestamp of approval
- timestamp of MCP Server access to the enterprise resource
- timestamp of data returned to caller
- decision result
- denial or expiration reason where applicable

---

## MVP Constraints

The following constraints apply to the MVP model:

- human approval is always required
- initiator acts as approver
- access limited to Zone 1 resources
- read-only operations only
- short TTL
- MCP Server must be registered as Agent Runtime with `agent_id`
- an ACTIVE `session_id` is required before submitting an access request
- make-it-so approval must be obtained and validated before enterprise data is returned
- audit correlation between mis-backend and MCP Server is required
- enterprise IAM and network enforcement may be deferred to a later phase

---

## Security Risks and Mitigations

| Risk | Mitigation |
|---|---|
| MCP Server has broad standing enterprise access in MVP | MCP Server must enforce approval before returning data; audit correlation required; limit to Zone 1 and read-only; short TTL |
| MCP Server implementation bypasses make-it-so approval | Code review, QAT review, integration tests, audit checks, future infrastructure enforcement |
| Agent receives enterprise data without valid approval | Active `session_id` required; `approval_request_id` required; scope and TTL validation before response |
| Approval scope is too broad | Scope must be explicit, human-readable, and logged; MVP limited to read-only Zone 1 |
| Audit gap between approval and data return event | Correlation IDs across mis-backend and MCP Server logs |
| Administrators reject deep IAM or network integration | MVP starts with Integration-Enforced JIT; Infrastructure-Enforced JIT documented as the future target path |
| MCP Server becomes uncontrolled enterprise data gateway | Registered Agent Runtime; session validation; approval gating; rate limits; logging; future policy enforcement point |

---

## Open Questions

The following questions are deferred to later elaboration:

- How exactly is the MCP Server registered and configured as an Agent Runtime?
- How does the MCP Server validate a make-it-so approval before returning data?
- What is the minimal approval validation API that the mis-backend exposes to the MCP Server?
- Which access scopes are supported in the MVP?
- How are Zone 1 resources defined and enumerated?
- How are enterprise resource categories modeled in the access request?
- How are `approval_request_id` and `access_request_id` correlated in the log schema?
- What log schema is required on the MCP Server side?
- Which log events are forwarded to the mis-backend and which are retained locally?
- How is the short-lived scoped access context represented and transported?
- How are expired approvals handled if the MCP Server is mid-operation when TTL expires?
- What happens if the MCP Server can technically access data but the mis-backend is unavailable?
- What tests prove that the MCP Server does not return data without a valid approval?
- Which future enterprise enforcement patterns are most realistic for early adopters?

---

## Related Documents

- [use-cases/use-case-jit-authorization-mcp-servers.md](../use-cases/use-case-jit-authorization-mcp-servers.md) — JIT authorization flow and use case definition
- [adr/0002-short-lived-scoped-session-mcp-authorization.md](../adr/0002-short-lived-scoped-session-mcp-authorization.md) — architectural decision for short-lived scoped sessions
- [architecture/agent-interface-security.md](agent-interface-security.md) — Agent Runtime trust model and session-led request validation
- [architecture/session-connect.md](session-connect.md) — session binding mechanism
- [architecture/action-model.md](action-model.md) — Action Request structure
- [architecture/request-lifecycle.md](request-lifecycle.md) — lifecycle stages for access requests
- [architecture/control-plane.md](control-plane.md) — overall control flow
- [adr/0011-agent-interface-trust-model.md](../adr/0011-agent-interface-trust-model.md) — Agent Interface trust model decision
