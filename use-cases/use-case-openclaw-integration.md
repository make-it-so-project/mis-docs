# Use Case: OpenClaw Integration – Approval-Gated Bot Actions

## Purpose

Enable OpenClaw bot operators to restrict specific bot actions behind make-it-so
approval gates, so that bots can act autonomously within allowed bounds but must
request explicit human approval before executing configured restricted actions.

## Context

OpenClaw is an open-source autonomous AI agent framework that runs as a persistent
local Gateway daemon, connecting LLMs (Claude, GPT, DeepSeek) to messaging platforms
such as Telegram, Discord, WhatsApp, and Signal.

By design, OpenClaw bots act autonomously. The framework provides tool allow/deny
lists and sandbox policies, but currently has no mechanism to pause execution and
request external human approval before performing a specific action.

This creates a governance gap for use cases where full bot autonomy is undesirable:

- bots may execute destructive or irreversible actions without human review
- there is no audit trail tied to explicit human approval decisions
- approval granularity cannot be configured without modifying the bot logic

make-it-so addresses this gap by acting as an external approval layer that OpenClaw
can consult before executing restricted actions.

## Goal

Provide a configurable approval integration for OpenClaw that allows operators to:

- define which tool types require human approval before execution
- route approval requests through the make-it-so control plane
- receive approval or denial on a trusted device (mobile app or web app)
- allow the bot to proceed only after explicit human sign-off

## Actors

**OpenClaw Operator**
Configures which tools require approval via the OpenClaw settings.

**User / Chat Initiator**
Sends prompts to the OpenClaw bot via the messaging platform.
In the MVP, the initiator is also the approver.

**OpenClaw Bot**
Executes tasks autonomously. Requests approval for restricted tool calls.

**make-it-so Platform**
Receives approval requests, routes them to the approver, and returns the decision.

**Approver (mis-client)**
Reviews and approves or denies the request via the mobile app or web app.

## Action Restriction Model

### Configuration Primitive: Tool Name

Restrictions are configured at the **tool name level**, using the same vocabulary
already present in OpenClaw's config (`tools.allow`, `tools.deny`).

Example configuration:

```yaml
mis_approval_required:
  - exec
  - write
  - browser
  - bash
```

This is unambiguous, directly aligned with OpenClaw's existing permission model,
and requires no new abstraction layer.

### Semantic Label for Human Approval: action_type

Tool names alone are not meaningful to a human approver.

make-it-so maps tool names to a semantic `action_type` label before presenting
the request to the approver. This label corresponds to the `action_type` field
in the make-it-so Action Request structure.

Example mapping:

| OpenClaw Tool | action_type              | Approver-facing label          |
|---------------|--------------------------|--------------------------------|
| exec          | bot.code_execution       | Code Execution                 |
| write         | bot.file_write           | File Write                     |
| browser       | bot.web_automation       | Web Automation                 |
| bash          | bot.shell_command        | Shell Command                  |

The bot includes a human-readable `summary` field in the request, derived from
the action context (e.g., "Write report to /home/user/output.txt").

This two-level model provides:
- **operator clarity** — restrict by tool name as in standard OpenClaw config
- **approver clarity** — see semantic action type and plain-language summary

### Configuration Scope

The scope at which restrictions are configured evolves between MVP and Production.

#### MVP: Global Instance-Level Configuration

In the MVP, a single flat list in the OpenClaw settings applies to all bots on
the instance. The operator defines it once; all agents on that instance are subject
to it equally.

```yaml
mis:
  host: https://<mis-host>
  approval_required:
    - exec
    - write
    - bash
    - browser
```

This requires no per-agent logic and is sufficient for single-operator or
small-team deployments.

#### Production: Hierarchical Per-Agent Configuration

For deployments with multiple agents of different trust levels, a two-tier
configuration model applies: instance-level defaults with per-agent overrides.

The governing security principle is:

> **Agents may narrow (further restrict) the global policy but never expand
> (relax) it. An agent cannot remove a tool from the global restriction list.**

```yaml
mis:
  host: https://<mis-host>
  # Instance-level default — applies to all agents unless overridden
  approval_required:
    - exec
    - write

  agents:
    research-bot:
      # Narrower than global — only exec restricted for this agent
      approval_required:
        - exec
    admin-bot:
      # Stricter than global — adds bash and browser on top of global defaults
      approval_required:
        - exec
        - write
        - bash
        - browser
```

Any attempt to configure an agent with fewer restrictions than the global list
must be rejected at config parse time.

This model aligns with OpenClaw's existing `agents[].tools` configuration
pattern and with the least-privilege principle.

| Dimension            | MVP: Global Flat              | Production: Hierarchical            |
|----------------------|-------------------------------|-------------------------------------|
| Config complexity    | Minimal                       | Moderate                            |
| Granularity          | Instance-wide                 | Per agent                           |
| Security principle   | All agents treated equally    | Least-privilege per agent           |
| Consistent with      | OpenClaw global `tools.deny`  | OpenClaw `agents[].tools` pattern   |
| Override direction   | n/a                           | Restrict only — never relax         |

---

## Failure Behavior

### make-it-so Unreachable: Fail-Closed (Default)

If make-it-so cannot be reached when a restricted tool call is attempted,
the default behavior is **DENY**.

The bot must not execute the restricted action and must inform the user
that the approval service is unavailable.

```
Bot attempts to call restricted tool
       │
       ▼
make-it-so MCP call fails (timeout / connection error)
       │
       ▼
Result treated as DENIED
       │
       ▼
Bot informs user: "Approval service is currently unavailable. Action not executed."
       │
       ▼
Event logged with status: APPROVAL_SERVICE_UNREACHABLE
```

**Rationale:** Safety over availability. An unreachable control plane must not
silently degrade into unrestricted execution. The operator can address availability
separately; the default must protect the governance boundary.

This applies to both MVP (MCP tool timeout) and Production (Gateway hook timeout).

---

## Integration Architecture

Two integration stages are defined: MVP and Production.

---

### Stage 1 — MVP: MCP Server Integration (Cooperative)

make-it-so exposes an MCP server that OpenClaw connects to via its existing MCP
extension mechanism. The bot's skill or system prompt instructs it to call the
`request_approval` MCP tool before executing any restricted tool.

The MCP tool call blocks until make-it-so returns the approval decision.

#### Configuration

The OpenClaw settings file enables the make-it-so MCP server:

```yaml
mcp_servers:
  - name: make-it-so
    url: https://<mis-host>/mcp
    tools:
      - request_approval
```

Restricted tools are listed under `mis_approval_required` (see above).

The bot's skill or system prompt contains an instruction such as:

> Before calling any tool listed in `mis_approval_required`, call `request_approval`
> with the action type and a plain-language summary. Wait for the result.
> Proceed only if the result is `APPROVED`. If `DENIED`, inform the user and stop.

#### Main Flow

```
User sends prompt
       │
       ▼
OpenClaw bot determines a restricted tool is needed
       │
       ▼
Bot calls make-it-so MCP tool: request_approval(action_type, summary, user_ref)
       │
       ▼
make-it-so evaluates request and notifies approver via mis-client
       │
       ▼
Approver reviews and approves or denies
       │
       ├─ APPROVED → bot calls restricted tool → execution proceeds → result logged
       └─ DENIED   → bot informs user in chat → stops
```

#### Security Model

- The restriction relies on **LLM compliance**: the bot must follow the instruction
  to call `request_approval` before acting.
- A misbehaving, jailbroken, or misconfigured LLM could bypass the approval step.
- This model is appropriate for **trusted LLM contexts** (controlled deployment,
  known model providers) and for **prototyping and validation**.
- It does not provide enforcement at the system execution boundary.

---

### Stage 2 — Production: Gateway Hook Integration (Enforced)

make-it-so is integrated as a **Gateway-level middleware** in OpenClaw.
The Gateway intercepts outbound tool calls matching the configured restricted
tool list and suspends execution until make-it-so returns a decision.

The LLM is not involved in the approval step. The enforcement happens below
the model layer, at the execution boundary.

This stage requires either:
- an official extension/hook API in OpenClaw (aligned with community discussion,
  see GitHub Issue #5641: "Tiered agent permissions with sandboxed tool policies")
- or a targeted contribution to OpenClaw upstream

#### Main Flow

```
User sends prompt
       │
       ▼
OpenClaw bot decides to call a restricted tool
       │
       ▼
OpenClaw Gateway intercepts the tool call (before execution)
       │
       ▼
Gateway sends approval request to make-it-so
       │
       ▼
make-it-so notifies approver via mis-client
       │
       ▼
Approver reviews and approves or denies
       │
       ├─ APPROVED → Gateway allows tool execution → result logged
       └─ DENIED   → Gateway returns denial to bot → bot informs user
```

#### Security Model

- Enforcement is at the **system execution boundary** — the LLM cannot bypass it.
- Applies unconditionally, regardless of model behavior or prompt injection.
- This is the appropriate model for **production deployments** with genuine
  security or governance requirements.

---

## Comparison: MVP vs. Production

| Dimension             | MVP: MCP Server           | Production: Gateway Hook        |
|-----------------------|---------------------------|---------------------------------|
| Enforcement level     | LLM-cooperative           | System-enforced                 |
| OpenClaw changes      | None required             | Extension point or upstream PR  |
| LLM bypass risk       | Present                   | Eliminated                      |
| Implementation effort | Low                       | Medium to high                  |
| Suitable for          | Prototypes, trusted LLMs  | Production, security-sensitive  |
| Aligns with           | ADR-0002, MCP extension   | OpenClaw Issue #5641            |

---

## MVP Constraints

- Human approval always required (no auto-allow policy in MVP)
- Initiator of the chat session is also the approver
- Approval interface: mobile app (primary) or web app (prototype fallback)
- Access scope limited to the specific tool call in context
- Short approval window (TTL applies — request expires if not acted on)

## Preconditions

- OpenClaw is deployed with make-it-so MCP server enabled in settings
- The restricted tool list is configured in OpenClaw settings
- The user has a registered mis-client device
- make-it-so is reachable by the OpenClaw Gateway (if unreachable, fail-closed applies — see Failure Behavior)

## Architectural Implications

The integration requires:

- make-it-so to expose a synchronous or long-polling MCP tool: `request_approval`
- user identity resolution: OpenClaw `user_ref` must map to a registered mis-client
- action type mapping: tool name → `action_type` label
- audit logging for all approval requests, decisions, and execution outcomes
- session TTL for pending approval requests
- bot-side messaging to inform the user that approval is pending (MVP: via skill instruction)

---

## Related Documents

- [ADR-0002: Short-Lived Scoped Sessions for MCP Authorization](../adr/0002-short-lived-scoped-session-mcp-authorization.md)
- [Action Model](../architecture/action-model.md)
- [Request Lifecycle](../architecture/request-lifecycle.md)
- [Component Diagram](../architecture/component-diagram.md)
- [Use Case: JIT Authorization for MCP Servers](use-case-jit-authorization-mcp-servers.md)
