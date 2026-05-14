
# Glossary

This document defines the core terminology used within the make-it-so project.

Maintaining a shared vocabulary ensures consistent communication between human contributors and AI Coders.

---

# Core Concepts

## Agent

An Agent is an operational entity within the runtime environment of the make-it-so platform.

Agents perform actions that can be:

- approved
- executed
- monitored
- audited

Agents belong to the runtime domain of the system.

---

## AI Coder

An AI Coder is an AI-based development unit responsible for implementing or modifying parts of the codebase.

Examples include:

- AI-Coder-Core
- AI-Coder-Frontend
- AI-Coder-Auth
- AI-Coder-Testing

AI Coders belong to the development domain.

---

# Runtime Security Concepts

## Agent Runtime

The technical process or integration that submits structured requests to the
mis-backend on behalf of an LLM, MCP server, automation agent, or bot system.

Characteristics:

- Authenticated to the mis-backend.
- Identified by `agent_id`.
- Authenticated but untrusted proposer.
- Distinct from the LLM / Sprachmodell.

See [architecture/agent-interface-security.md](../architecture/agent-interface-security.md)
and [ADR-0011](../adr/0011-agent-interface-trust-model.md).

---

## LLM / Sprachmodell

Generates suggestions, plans, text, or parameters, but is not the trusted
technical identity authenticated by the mis-backend.

The LLM is distinct from the Agent Runtime. The mis-backend authenticates the
Agent Runtime, not the LLM.

---

## agent_id

Stable technical identifier of a registered Agent Runtime.

- Derived from authenticated Agent Runtime context.
- NOT accepted as self-asserted request body authority.
- Used for audit, rate limiting, policy, and session binding.

See [architecture/agent-interface-security.md](../architecture/agent-interface-security.md).

---

## Authenticated but Untrusted Proposer

A component that is known to the mis-backend but not trusted to authorize
actions. It may submit proposals; authorization remains with policy and
human approval.

All Agent Runtimes are authenticated but untrusted proposers.

---

## Session Reactivation / Reconnect

Mechanism by which an Agent Runtime uses stored `user_id` and `client_id`
reconnect hints to request a new ACTIVE `session_id`.

Two modes:

- **notify_only**: new session created on validation success; mis-client
  receives inform-only notification.
- **approval_required**: new session requires mis-client approval.

See [architecture/session-connect.md](../architecture/session-connect.md)
and [architecture/agent-interface-security.md](../architecture/agent-interface-security.md).

---

## Reconnect Hint

Stored `user_id` and `client_id` held by the Agent Runtime to request session
reactivation. Not an authorization credential.

The Agent Runtime MUST NOT treat reconnect hints as sufficient to submit
authorized Action Requests. An ACTIVE `session_id` is required.

---

## MCP Server

A server implementing the Model Context Protocol (MCP) that enables AI agents or LLM systems to interact with external tools, APIs, or enterprise systems.

Within the make-it-so architecture, MCP servers are considered **untrusted by default** and must request authorization from make-it-so before accessing protected enterprise resources.

---

## Short-Lived Scoped Session

A temporary authorization context granted after human approval.

The session provides narrowly scoped permissions to an MCP server for a limited duration (TTL). The scope defines which resources and operations are allowed.

This mechanism enables just-in-time enterprise access without granting standing privileges.

---

## Scope

A defined set of permissions, resources, and allowed operations granted to an MCP server during an authorized session.

Scopes restrict what the MCP server can access within enterprise systems.

---

## Session TTL (Time To Live)

The maximum duration for which a short-lived scoped session remains valid before it automatically expires.

This ensures that temporary access permissions are automatically revoked after a defined period.

---

## mis-user

A registered human user of the make-it-so platform.

A mis-user has a verified email address and a stable `user_id` (UUID),
which serves as both the internal primary key and the stable domain
identifier exchanged between platform components.

After successful registration, a mis-user normally has at least one
registered ACTIVE mis-client. During lockout or account recovery scenarios,
a mis-user may temporarily have no ACTIVE client. Account Recovery is the
path to re-establish an ACTIVE client.

See [architecture/user-registration.md](../architecture/user-registration.md).

---

## User Registration

The process by which a private individual creates a mis-user account and
enrolls their first trusted mis-client as one combined atomic operation.

User Registration uses passkey-only authentication. The flow consists of
an email magic link that opens directly onto a passkey creation screen.
The mis-user record is only created after the full flow completes.

User Registration is distinct from:
- **Client Registration** — adding further clients to an existing account
- **Session Connect** — linking an agent session to a registered client

See [architecture/user-registration.md](../architecture/user-registration.md)
and [ADR-0009](../adr/0009-user-registration-model.md).

---

## mis-client

A trusted user device used to approve authorization requests within the make-it-so system.

The mis-client receives approval requests and allows the user to approve or deny them.

---

## Zone 1

A lower-risk enterprise network zone containing internal systems that may be accessed by MCP servers under controlled conditions.

More sensitive systems (for example databases, administrative infrastructure, or production control systems) typically reside in higher security zones.

---

# Backlog Model

The development process distinguishes between two backlog levels.

---

## G-BL (Global Backlog)

The Global Backlog is used for:

- ideas
- request refinement
- architectural exploration
- decomposition of complex requests

Items in the Global Backlog are not executable tasks.

---

## Development Domain

The Development Domain encompasses all activity related to building and maintaining
the make-it-so platform itself.

Participants include human contributors and AI Coders (e.g. Claude Code, Codex, Cursor).

Discussions and tasks in this domain are tagged with the `DEV` domain marker.

---

## Runtime Domain

The Runtime Domain encompasses all activity related to the operational behavior
of the make-it-so platform during execution.

Participants include runtime agents, the control plane, and execution connectors.

Discussions and tasks in this domain are tagged with the `RUN` domain marker.

---

## Domain Marker

A structured prefix applied to issues, PRs, and design discussions to indicate
the domain of the topic.

Format: `[DOMAIN]` or combined with a category: `[DOMAIN/CATEGORY]`

Examples: `[DEV]`, `[RUN]`, `[DEV/OPS]`, `[RUN/SEC]`

See ADR 0003 for the full marker system.

---

## Category Marker

A structured sub-classification applied alongside a domain marker to indicate
the type of concern being addressed.

| Marker | Meaning                             |
|--------|-------------------------------------|
| `ARCH` | Architecture                        |
| `FUNC` | Functional Behavior                 |
| `NFR`  | Non-Functional Requirements         |
| `OPS`  | Operations / Tooling / CI           |
| `SEC`  | Security                            |
| `ADR`  | Architecture Decision Record        |

See ADR 0003 for the full marker system.

---

## Secondary Notification Channel

An optional, out-of-band channel through which make-it-so may deliver event
notifications to a user.

Examples include mobile push notifications, messaging platforms, email, and SMS.

A secondary notification channel:

- MUST only inform or redirect the user
- MUST NOT perform, accept, or transmit approval decisions
- MUST NOT carry credentials of any kind
- MUST NOT act as a trusted authorization surface

Users are directed to the mis-client for any approval action.

See the [notification channel architecture](../architecture/notification-channel.md)
and ADR 0004 for full details.

---

## Client Registration / Client Enrollment

The process by which a mis-client (device or application) is persistently
associated with a mis-user as a trusted approval surface.

A registered client has an explicit lifecycle: `pending → active → revoked`.

Client Registration is distinct from Session Connect (which selects an
already-registered client) and from User Registration (which creates
the mis-user account itself).

See [architecture/client-registration.md](../architecture/client-registration.md).

---

## Registered Client

A mis-client that has completed the Client Registration process and has
status ACTIVE in the mis-backend.

Only registered ACTIVE clients may receive approval requests and participate
in Session Connect flows.

---

## Client Assurance Level

A classification that describes the security properties of a registered
mis-client, based on its `client_type` and key storage capabilities.

| Level | Meaning |
|---|---|
| `basic` | WebApp/PWA with browser-managed key storage |
| `standard` | Native app with OS-managed key storage |
| `high` | Native app with hardware-backed key storage |

The Policy Engine MAY evaluate `assurance_level` when determining
requirements for specific approval types.

See [ADR-0006](../adr/0006-webapp-pwa-mvp-client-assurance.md).

---

## WebApp/PWA Client

A mis-client implemented as a web application or Progressive Web App,
running in a browser environment.

A WebApp/PWA client MUST be registered with `client_type = web_pwa`
and `assurance_level = basic`. It provides persistent client identity
through browser-managed cryptographic key storage, but is not equivalent
to a native high-assurance client.

---

## Native Client

A mis-client implemented as a native mobile application on iOS or Android.

Native clients offer higher assurance levels than WebApp/PWA clients
through OS-managed or hardware-backed key storage.

`client_type`: `ios_native` | `android_native`

---

## Client Public Key

The public component of the asymmetric key pair generated by a mis-client
during Client Registration.

The mis-backend stores the client public key and uses it to verify
that requests originate from the registered client instance.

---

## Client Private Key

The private component of the asymmetric key pair generated by a mis-client.

The client private key MUST remain on the client at all times.
It MUST NOT be transmitted to the mis-backend or any other party.
It MUST NOT appear in any log.

---

## User Step-up Authentication

An additional authentication step that proves current user presence and
user verification, beyond holding a registered client key.

Used for: Client Registration, high-risk approvals, and any operation
where policy requires proof of human presence.

Implemented via WebAuthn/passkey or equivalent platform authenticator.

Client key possession alone MUST NOT be treated as equivalent to user
step-up authentication.

See [ADR-0007](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md).

---

## First Client Bootstrap

The special case of Client Registration in which a mis-user has no
existing ACTIVE registered client. Because no existing trusted device
is available to confirm the registration, the bootstrap MUST use strong
authentication to establish initial trust.

First Client Bootstrap applies in two contexts:

1. **Initial User Registration** — performed as part of the combined
   User Registration flow; user account and first client are created together.
2. **Recovery** — an existing user re-establishes a trusted client after
   losing all registered clients. Defined in ADR-0010.
   See [account-recovery.md](../architecture/account-recovery.md).

The accepted bootstrap mechanism (per ADR-0009) is an email magic link
that opens directly onto a passkey creation screen — email verification
and WebAuthn/passkey creation in one continuous flow.

See [architecture/client-registration.md](../architecture/client-registration.md)
and [ADR-0009](../adr/0009-user-registration-model.md).

---

## Client Revocation

The permanent deactivation of a registered mis-client.

A revoked client has status REVOKED and MUST NOT receive approval requests.
Revocation is logged and triggers security notifications to all remaining
active clients.

Revocation of the last active client leaves the user without a trusted
approval surface until account recovery is performed.

---

## Signed Client Request

A request from a mis-client to the mis-backend that includes a cryptographic
signature over stable request fields, proving that the caller holds the
registered client private key.

Signed client requests prevent request forgery by attackers who hold a session
token but not the private key. Approval and denial decisions, client revocation,
and other trust-changing operations MUST be signed or challenge-bound.

See [architecture/client-identity-and-secure-communication.md](../architecture/client-identity-and-secure-communication.md).

---

## Approval Payload Binding

The requirement that an approval or denial decision from a mis-client is
cryptographically bound to the specific approval request and action content
the user was shown.

Binding fields include `approval_request_id`, `action_id`, and
`details_hash` / `action_payload_hash`. The mis-backend rejects decisions
where any of these do not match stored backend state.

See [architecture/client-identity-and-secure-communication.md](../architecture/client-identity-and-secure-communication.md).

---

## Trusted Approval Surface

A Trusted Approval Surface is a component that is authorized to collect and
transmit human approval decisions within the make-it-so system.

The **mis-client** is the designated trusted approval surface.

Characteristics of a trusted approval surface:

- operates under the user's direct control
- communicates approval decisions to the control plane through a secured path
- may receive execution credentials (e.g., OTPs) from the control plane

Secondary notification channels are explicitly NOT trusted approval surfaces.

---

## Account Recovery

The process by which an existing mis-user regains access to their account
after losing the ability to authenticate — specifically when both their
passkey and all registered clients are unavailable.

Account Recovery is self-service and requires two factors:

1. A time-limited, single-use magic link sent to the verified email address
2. A valid recovery code

On completion, the recovery flow ends with First Client Bootstrap: all
previous mis_client records are revoked by the recovery flow, a new
mis-client is enrolled as the only ACTIVE registered client, and 2 new
recovery codes are generated.

See [architecture/account-recovery.md](../architecture/account-recovery.md)
and [ADR-0010](../adr/0010-account-recovery-model.md).

---

## Recovery Code

A pre-generated, single-use secret used as a second authentication factor
during Account Recovery.

Recovery codes are:

- generated at the time of User Registration (2 codes issued)
- stored as cryptographic hashes in the mis-backend (originals displayed once)
- regenerated after a successful recovery or via step-up auth from an active client
- formatted as XXXX-XXXX-XXXX (alphanumeric, unambiguous characters)

A user who has exhausted all recovery codes must contact support for
out-of-band identity verification.

See [architecture/account-recovery.md](../architecture/account-recovery.md).

---

## S-BL (Sprint Backlog)

The Sprint Backlog contains tasks that have been approved for implementation.

Only tasks inside the Sprint Backlog may trigger development activity.

The transfer from G-BL → S-BL represents formal approval for implementation.

Only the human project owner performs this transition.

---

# AI Coder Sprint Loop Concepts

---

## Orchestration Instance

A coordinating entity that manages the AI Coder Development Sprint Loop.

Responsibilities include assigning S-BL tickets to Coder/QAT pairs, tracking ticket state,
creating follow-up tickets when quality assurance review identifies unfinished work, and triggering the release build
phase after the S-BL is empty.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md).

---

## Coder AI Agent

An AI Coder assigned to implement a specific S-BL ticket.

The Coder AI Agent works on a feature branch, commits the implementation, and opens a PR
according to existing workflow rules.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md).

---

## QAT AI Agent (Quality Assurance Tester)

An AI Coder assigned to independently review the output of a Coder AI Agent.

The QAT AI Agent MUST NOT be the same AI Agent as the implementing Coder. The QAT AI Agent reviews the
commit, branch, PR, and relevant artifacts, and records a QAT outcome.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md).

---

## Pair-Programming Team

The mandatory pairing of one Coder AI Agent and one independent QAT AI Agent (Quality Assurance Tester) for a single
S-BL ticket.

Every S-BL ticket MUST be handled by a Pair-Programming Team. Self-review is not sufficient.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md).

---

## QAT Outcome

The quality assurance result recorded by a QAT AI Agent after reviewing a Coder AI Agent's implementation.

Two primary outcomes:

- `completed` — implementation satisfies the ticket scope; no follow-up required.
- `follow_up_required` — implementation needs correction; a new S-BL ticket is created.

The original S-BL ticket is always closed after the QAT outcome is recorded.

See [docs/ai-coder-sprint-loop.md](../docs/ai-coder-sprint-loop.md).

---

# MCP Server JIT Access Concepts

---

## Integration-Enforced JIT

A deployment pattern where the MCP Server itself enforces make-it-so approval before
returning enterprise data, even though the underlying enterprise access may already be
technically available to the MCP Server.

This is the pragmatic MVP pattern. The zero-trust property depends on correct MCP Server
implementation, not on infrastructure enforcement.

See [architecture/mcp-jit-access-building-blocks.md](../architecture/mcp-jit-access-building-blocks.md).

---

## Infrastructure-Enforced JIT

A deployment pattern where enterprise infrastructure enforces the approved scope, TTL, and
access path, so the MCP Server cannot access or return protected enterprise data outside the
approved context.

This is the stronger target pattern for higher assurance deployments.

See [architecture/mcp-jit-access-building-blocks.md](../architecture/mcp-jit-access-building-blocks.md).

---

## Policy Enforcement Point

The component responsible for enforcing the approved scope and access constraints in a
Just-in-Time access model.

In the MVP Integration-Enforced JIT model, the MCP Server is the policy enforcement point.
In the target Infrastructure-Enforced JIT model, enterprise infrastructure (gateway, proxy,
IAM, or dedicated enforcement component) takes this role.

See [architecture/mcp-jit-access-building-blocks.md](../architecture/mcp-jit-access-building-blocks.md).

---

## Scoped Access Context

The temporary authorization context granted after make-it-so approval, defining the approved
scope, TTL, and resource boundaries for a short-lived enterprise access session.

Validated by the MCP Server in the MVP; enforced by enterprise infrastructure in the target model.

See [architecture/mcp-jit-access-building-blocks.md](../architecture/mcp-jit-access-building-blocks.md).

---

## Audit Correlation

The requirement that every enterprise data response returned by an MCP Server can be traced
to a specific make-it-so approval record, or to a clear rejection or expiration record.

Achieved by linking `approval_request_id`, `session_id`, `agent_id`, `user_id`, and
timestamps across mis-backend and MCP Server logs.

See [architecture/mcp-jit-access-building-blocks.md](../architecture/mcp-jit-access-building-blocks.md).

---

# Repository Safety Concepts

---

## Canary

A deliberately placed signal or marker used to detect unexpected access, modification,
deletion, or bypass behavior. A canary detects; it does not prevent.

Concrete canary values are intentionally not documented publicly.

See [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md).

---

## Watcher

A process, check, or review mechanism that observes changes to protected assets or monitors
for suspicious patterns. Watchers may be automated, semi-automated, or human-driven.

See [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md).

---

## Tripwire

A detection rule or condition that triggers attention when a protected asset or expected
invariant changes. A form of watcher with a specific trigger condition.

Concrete tripwire conditions are intentionally not documented publicly.

See [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md).

---

## Protected Asset Class

A category of repository files that requires special care, explicit callout in PR bodies,
and QAT and project owner review when changed.

Examples: governance rules, AI Coder operating instructions, architecture decisions,
security-sensitive documentation, CI/CD configuration, dependency metadata.

See [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md).

---

# Release Coordination Concepts

---

## Release Context

A stable coordination identifier that assigns AI Coder work, S-BL tickets, sprint loops,
branches, PRs, release builds, regression testing, and release notes to a shared release
target.

Pattern: `RELEASEPHASE-YYYY-MM-NNN` — e.g. `MVP-2026-05-001`, `ALPHA-2026-07-042`.

See [governance/release-context.md](../governance/release-context.md).

---

## Current Release Context

The active release context for the current Dev Sprint Loop and concrete S-BL implementation
work. Controlled by the project owner. AI Coders must not change it unless explicitly assigned.

See [governance/release-context.md](../governance/release-context.md).

---

## Release Build Gate

The set of conditions that must be satisfied before a release build may start for a given
release context. Includes S-BL completion, QAT outcomes, PR merge status, safety checks,
and regression readiness.

See [governance/release-context.md](../governance/release-context.md).

---

## Release Phase

The stage of the product lifecycle a release context belongs to.

| Phase | Meaning |
|---|---|
| `MVP` | First minimum viable product release line |
| `ALPHA` | Early user or testing phase |
| `BETA` | Broader testing phase (optional) |
| `RC` | Release candidate |
| `PROD` | Production release line (optional later) |

See [governance/release-context.md](../governance/release-context.md).

---

# Programming Language and Toolchain Concepts

---

## Toolchain

The set of tools used to build, format, lint, typecheck, test, and package an implementation
component. Each supported language must have a documented toolchain before AI Coder
implementation work begins.

See [docs/programming-guides/language-and-toolchain-strategy.md](../docs/programming-guides/language-and-toolchain-strategy.md).

---

## Standard Command Surface

The minimum set of documented commands every implementation package must provide:
setup, format, lint, typecheck or compile, test, build, and a combined check command.

Required so AI Coders can validate changes without GUI-only or undocumented tooling.

See [docs/programming-guides/language-and-toolchain-strategy.md](../docs/programming-guides/language-and-toolchain-strategy.md).

---

## Programming Guide

A document in `docs/programming-guides/` that defines conventions, toolchain commands,
secure coding rules, and AI Coder guardrails for a specific language or cross-language
concern.

See [docs/programming-guides/README.md](../docs/programming-guides/README.md).

---

## Secure Coding Guardrails

Cross-language rules that apply to all make-it-so implementation work, covering secrets
handling, input validation, logging constraints, cryptographic library use, and dependency
safety.

See [docs/programming-guides/language-and-toolchain-strategy.md](../docs/programming-guides/language-and-toolchain-strategy.md).

---

# Backend Hosting and Operations Concepts

---

## Cloudflare-First MVP Posture

The current candidate hosting strategy for the make-it-so backend MVP. Cloudflare is the
primary candidate for early backend hosting where technically feasible, due to low or no
cost at early usage levels, managed edge infrastructure, and reduced operational overhead.

This is a candidate posture, not a final binding decision. Suitability depends on runtime
requirements (long-running processes, durable state, queues, WebSockets) that must be
confirmed before a final hosting ADR is written.

See [architecture/backend-hosting-and-operations.md](../architecture/backend-hosting-and-operations.md).

---

## Standby Level

A classification of how quickly a backup or secondary node can be activated in a failover
scenario. Three levels are defined:

| Level | Definition |
|---|---|
| Cold standby | Backup exists; restore requires manual action and time |
| Warm standby | State is periodically synced; activation requires deliberate manual steps |
| Hot standby | Active failover-ready node; automatic or near-automatic activation |

The MVP does not yet decide which standby level is required or feasible.

See [architecture/backend-hosting-and-operations.md](../architecture/backend-hosting-and-operations.md).

---

## Restore Test

A deliberate exercise of the restore procedure from a backup to verify that the backup is
actually usable. A backup that has never been restored should not be treated as a proven
safety net. Restore tests are a prerequisite for production readiness claims.

See [architecture/backend-hosting-and-operations.md](../architecture/backend-hosting-and-operations.md).

---

## Operational Monitoring

Infrastructure- and service-level observability covering service health, errors, latency,
usage metrics, and cost visibility. Required before external testing with real users.
Distinct from Audit Logging.

See [architecture/backend-hosting-and-operations.md](../architecture/backend-hosting-and-operations.md).

---

## Audit Logging (Operations)

The recording of security- and approval-relevant events for integrity, retention, and
compliance purposes. Distinct from operational monitoring logs. Audit logs must avoid
recording secrets, private keys, recovery codes, approval payload secrets, and unnecessary
personal data.

See [architecture/backend-hosting-and-operations.md](../architecture/backend-hosting-and-operations.md).

---

# UI Design System Concepts

---

## Design System

The set of visual identity guidance, UI style rules, design tokens, component
definitions, interaction state rules, accessibility requirements, and platform mapping
guidance for make-it-so. The design system must balance the LCARS-inspired visual
identity with clear, accessible, and secure user interaction.

See [docs/design-system/README.md](../docs/design-system/README.md).

---

## Approval UX

The user-facing experience through which a user receives, understands, and acts on an
approval request. Approval UX is a security-relevant concern for make-it-so because the
mis-client is the trusted approval surface. Approve and Deny must be visually and
spatially distinct, and high-risk approvals must require deliberate interaction.

See [docs/design-system/ui-style-guide-and-approval-ux.md](../docs/design-system/ui-style-guide-and-approval-ux.md).

---

## Interaction State

A defined visual and behavioral condition of a UI element. Required states include:
default, hover, focused, pressed/active, selected, disabled, pending, approved, denied,
expired, warning, and error. Each state must be explicitly represented so users never
need to guess element behavior or outcome.

See [docs/design-system/ui-style-guide-and-approval-ux.md](../docs/design-system/ui-style-guide-and-approval-ux.md).

---

## Decorative Surface

A non-interactive UI element — panel, border, color block, or layout accent — whose
purpose is visual identity or structure, not action. Decorative surfaces must not be
visually confused with interactive controls. In the LCARS-inspired design direction,
ensuring this distinction is a required design constraint.

See [docs/design-system/ui-style-guide-and-approval-ux.md](../docs/design-system/ui-style-guide-and-approval-ux.md).

---

## Interactive Surface

A UI element that the user can activate to perform an operation: buttons, links,
approval controls, and form inputs. Interactive surfaces must have consistent and
unambiguous affordance across all UI element states.

See [docs/design-system/ui-style-guide-and-approval-ux.md](../docs/design-system/ui-style-guide-and-approval-ux.md).

---

## Design Token

A named, platform-agnostic value for a visual attribute such as color, spacing,
typography, radius, focus style, or motion timing. Design tokens are the future
contract between design decisions and implementation. Final token names and values are
not yet defined.

See [docs/design-system/ui-style-guide-and-approval-ux.md](../docs/design-system/ui-style-guide-and-approval-ux.md).
