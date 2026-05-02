# Use Case: Just-in-Time Authorization for MCP Servers

## Purpose

Enable secure integration between external LLM systems and internal enterprise resources by using make-it-so as a human-approved authorization layer.

## Context

Many organizations use MCP servers to connect LLM systems with internal data and services.

This introduces a critical trust boundary:

External AI system → MCP server → Enterprise resources

Without strong controls the MCP server may become an uncontrolled access gateway.

## Goal

Operate MCP servers using a **zero-trust model** where:

- MCP servers have no standing privileges
- access must be explicitly requested
- human approval is required
- authorization is temporary and scoped

## Actors

Human User  
Initiates the AI task and approves access.

LLM / AI Agent  
Generates tasks that may require enterprise access.

MCP Server  
Executes tasks requiring internal resources.

make-it-so Platform  
Evaluates requests and coordinates approval.

Enterprise Systems  
Internal resources in Zone 1.

## Preconditions

- MCP servers are deployed without privileged enterprise access.
- make-it-so is reachable by the MCP server.
- The human user has a registered mis-client device.
- The user has established a mis-connect for this session: a `session_id →
  (user_id, client_id)` binding exists in the mis-backend, created via the
  Session Connect mechanism (see [Session Connect](../architecture/session-connect.md)).

## Main Flow

1. User issues a prompt to an AI agent.
2. The agent determines that enterprise data access is required.
3. The MCP server sends an access request to make-it-so, including the
   active `session_id`.
4. make-it-so resolves `session_id → (user_id, client_id)` to identify the
   initiating user and their chosen approval device. If no active session
   binding exists, the request is rejected and the agent is instructed to
   initiate a fresh Connect.
5. The approval request is sent to the user's mis-client identified by
   `client_id`.
6. The user reviews and approves the request.
7. make-it-so grants a short-lived scoped access session.
8. The MCP server performs the requested task.
9. The session expires automatically.

## Security Model

- MCP servers are **untrusted by default**
- no permanent credentials are stored on MCP servers
- all enterprise access is temporary
- all approvals are auditable

## MVP Constraints

- human approval always required
- initiator acts as approver
- access limited to Zone 1 resources
- read-only operations
- short session TTL

## Architectural Implications

The system requires:

- request/approval workflow
- user-to-device mapping
- temporary access provisioning
- session expiration handling
- audit logging for all approval and execution events

---

## Related Documents

- [ADR-0002: Short-Lived Scoped Sessions for MCP Authorization](../adr/0002-short-lived-scoped-session-mcp-authorization.md) — architectural decision behind the authorization model used in this use case
- [ADR-0005: Session Connect Mechanism](../adr/0005-session-connect-mechanism.md) — how the agent-user binding is established before an access request can be made
- [Session Connect](../architecture/session-connect.md) — Connect flow, session continuation, and data model
- [Action Model](../architecture/action-model.md) — structure of the access request submitted by the MCP server
- [Request Lifecycle](../architecture/request-lifecycle.md) — lifecycle stages the access request passes through
- [Use Case: OpenClaw Integration](use-case-openclaw-integration.md) — approval-gated bot actions for OpenClaw deployments