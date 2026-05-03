
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
existing ACTIVE registered client.

Because no existing trusted device is available to confirm the registration,
first client bootstrap MUST use strong authentication to establish initial
trust (e.g., WebAuthn with user verification, or out-of-band verified link).

See [architecture/client-registration.md](../architecture/client-registration.md).

---

## Client Revocation

The permanent deactivation of a registered mis-client.

A revoked client has status REVOKED and MUST NOT receive approval requests.
Revocation is logged and triggers security notifications to all remaining
active clients.

Revocation of the last active client leaves the user without a trusted
approval surface until account recovery is performed.

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

## S-BL (Sprint Backlog)

The Sprint Backlog contains tasks that have been approved for implementation.

Only tasks inside the Sprint Backlog may trigger development activity.

The transfer from G-BL → S-BL represents formal approval for implementation.

Only the human project owner performs this transition.
