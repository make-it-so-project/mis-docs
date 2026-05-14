# Architecture Overview

This directory contains the high-level architecture description of the make-it-so platform.

The architecture describes how system components interact and how responsibilities are distributed.

---

# Documents in this Directory

| Document | Description |
|----------|-------------|
| [agent-interface-security.md](agent-interface-security.md) | Agent Runtime identity, Agent Interface trust boundary, session_id-led request validation, reconnect model, and MVP/target security model |
| [system-context.md](system-context.md) | Purpose, problem statement, design goals, and non-goals |
| [component-diagram.md](component-diagram.md) | Main components and their relationships |
| [control-plane.md](control-plane.md) | Control flow and core architectural principle |
| [action-model.md](action-model.md) | Action Request structure and action lifecycle states |
| [request-lifecycle.md](request-lifecycle.md) | Detailed lifecycle from request creation to completion |
| [notification-channel.md](notification-channel.md) | Secondary notification channel definition, constraints, and flow |
| [session-connect.md](session-connect.md) | Session Connect mechanism: agent-user binding, pairing code flow, session continuation |
| [client-registration.md](client-registration.md) | Client lifecycle, enrollment flows, first bootstrap, confirmation, and revocation |
| [client-identity-and-secure-communication.md](client-identity-and-secure-communication.md) | Client key model, WebApp/PWA key handling, proof of possession, backend validation |
| [user-registration.md](user-registration.md) | User Registration model: passkey-first auth, combined first client bootstrap, self-service onboarding |
| [account-recovery.md](account-recovery.md) | Account Recovery: self-service model with email magic link and recovery codes |
| [mcp-jit-access-building-blocks.md](mcp-jit-access-building-blocks.md) | Placeholder for MCP Server Just-in-Time access realization patterns, distinguishing Integration-Enforced JIT for MVP from Infrastructure-Enforced JIT as target architecture |
| [backend-hosting-and-operations.md](backend-hosting-and-operations.md) | Placeholder for backend hosting strategy, Cloudflare-first MVP posture, VPS secondary option, cost model, state and persistence, backup/restore, standby levels, resilience, secrets management, monitoring, audit logging, and operations readiness |

---

# System Areas

The platform is expected to include several functional areas.

## Core Backend

The central backbone responsible for executing system transactions and coordinating services.

---

## Authentication and Authorization

Responsible for:

- identity management
- authentication
- permission control

---

## User Management

Provides functionality for managing users, including:

- onboarding
- account management
- self-service capabilities

---

## Agent Management

Provides functionality for managing operational agents, including:

- registration
- activation
- configuration
- deactivation

---

## Agent Communication API

Defines the interfaces used by agents to communicate with the make-it-so backend.

---

## Testing Infrastructure

Responsible for automated testing and regression testing.

---

## Security and Monitoring

Responsible for system monitoring, security controls, and audit logging.
