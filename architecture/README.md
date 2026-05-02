# Architecture Overview

This directory contains the high-level architecture description of the make-it-so platform.

The architecture describes how system components interact and how responsibilities are distributed.

---

# Documents in this Directory

| Document | Description |
|----------|-------------|
| [system-context.md](system-context.md) | Purpose, problem statement, design goals, and non-goals |
| [component-diagram.md](component-diagram.md) | Main components and their relationships |
| [control-plane.md](control-plane.md) | Control flow and core architectural principle |
| [action-model.md](action-model.md) | Action Request structure and action lifecycle states |
| [request-lifecycle.md](request-lifecycle.md) | Detailed lifecycle from request creation to completion |
| [notification-channel.md](notification-channel.md) | Secondary notification channel definition, constraints, and flow |
| [session-connect.md](session-connect.md) | Session Connect mechanism: agent-user binding, pairing code flow, session continuation |

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
