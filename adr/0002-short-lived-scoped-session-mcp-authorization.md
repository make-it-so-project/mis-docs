
# ADR-0002: Use Short-Lived Scoped Sessions for MCP Server Authorization

## Status
Accepted

## Date
2026-04-09

## Context

The make-it-so platform acts as a human-in-the-loop control plane for AI agents and MCP servers.

In enterprise environments MCP servers are increasingly used to connect external or public
LLM systems with internal enterprise systems and data sources. This creates a critical trust boundary:

External AI → MCP Server → Enterprise Systems

To reduce risk, MCP servers must follow a **zero-trust access model**:

- MCP servers have **no standing privileges**
- Enterprise access must be **explicitly requested**
- Access requires **human approval**
- Authorization must be **temporary and scoped**

A key architectural decision is how access should be granted after approval.

## Decision Drivers

The decision should balance:

- Zero-trust architecture
- Least privilege
- Practical usability for real MCP workflows
- Implementation complexity
- Human approval clarity
- Compatibility with enterprise identity systems (IAM, AD, LDAP, SSO)
- Auditability and traceability

## Considered Options

### Option 1: Single Task Grant

Grant authorization only for one specific action or operation.

### Option 2: Short-Lived Scoped Session

Grant a temporary access session with narrowly scoped permissions and a short
time-to-live (TTL).

## Option Analysis

### Option 1 – Single Task Grant

Advantages:

- Strict interpretation of least privilege
- Clear approval semantics for humans
- Very precise auditability
- Minimal misuse window

Disadvantages:

- Poor support for multi-step workflows
- High approval frequency
- High friction for real MCP interactions
- Difficult to map to enterprise IAM patterns
- Forces agents to fragment tasks artificially

Assessment:

This model is secure but impractical for real-world MCP usage where a single
logical task may require multiple system interactions.

---

### Option 2 – Short-Lived Scoped Session

Advantages:

- Supports multi-step MCP workflows
- Lower approval friction
- Aligns with enterprise Just-in-Time access models
- Compatible with temporary identities, scoped tokens, or role assignments
- Better balance between usability and security

Disadvantages:

- Slightly broader authorization window
- Requires session expiration handling
- Requires clear scope definition
- Requires audit correlation between approval and session activity

Assessment:

This model enables realistic MCP-based enterprise workflows while maintaining
strong security guarantees through short TTL, narrow scope, and human approval.

## Decision

make-it-so will use **short-lived scoped sessions** as the authorization model
for MCP server access.

After approval, make-it-so grants the MCP server a temporary enterprise access
context with the following constraints for the MVP:

- Human approval is always required
- The request initiator is also the approver
- Access is limited to **Zone 1 enterprise resources**
- Access is **read-only**
- The session has a **short time-to-live**
- Access is bound to the **approved work context**

The exact technical implementation may vary. Possible realizations include:

- temporary enterprise identity
- scoped access token
- short-lived access session
- temporary role assignment in enterprise IAM systems

This ADR defines the **authorization pattern**, not the specific technology.

## Consequences

### Positive Consequences

- Enables realistic MCP workflows
- Aligns with zero-trust architecture
- Supports enterprise IAM integration
- Reduces approval friction for users
- Maintains human accountability

### Negative Consequences

- Requires session lifecycle management
- Requires careful scope definition
- Requires strong audit logging
- Slightly larger access window than single-task grants

### Follow-up Implications

Future architecture work may include:

- policy-based approvals
- delegated approvals
- automated risk classification
- finer-grained scope definitions
- support for write operations

## Rationale Summary

A single-task authorization model is secure but too restrictive for realistic
MCP workflows.

Short-lived scoped sessions provide the best balance between security,
usability, and architectural extensibility. They enable multi-step workflows
while maintaining strong control through human approval, limited scope,
and automatic expiration.
