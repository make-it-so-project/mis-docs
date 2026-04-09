# Architecture Decision Records

Architecture Decision Records (ADR) document important architectural decisions made during development.

ADRs provide transparency and help maintain long-term architectural consistency.

---

# Existing ADRs

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [ADR-0001](0001-terminology-ai-coder-vs-agent.md) | Terminology: AI Coder vs Agent | Accepted | 2026-04-09 |
| [ADR-0002](0002-short-lived-scoped-session-mcp-authorization.md) | Use Short-Lived Scoped Sessions for MCP Server Authorization | Accepted | 2026-04-09 |

---

# Naming Scheme

ADR files use a numeric prefix.

Example:

0001-terminology
0002-backlog-model
0003-agent-architecture

---

# ADR Structure

Each ADR contains the following sections:

- Status
- Date
- Context
- Decision Drivers
- Considered Options
- Option Analysis
- Decision
- Consequences
- Rationale Summary

For new ADRs use the [adr-template.md](adr-template.md).

---

# Status Values

Possible status values include:

- Proposed
- Accepted
- Deprecated
- Superseded
