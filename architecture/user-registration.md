---
read_when: working on account creation, identity proofing, onboarding flows, or user lifecycle management
---

# User Registration

## Status

**Placeholder — intentionally underspecified.**

This document exists to mark User Registration as a separate future
concern and to prevent it from being conflated with Client Registration
or Session Connect.

---

## Purpose

User Registration defines how a new mis-user account is created and
how a user's identity is established within the make-it-so platform.

This is distinct from:

- **Client Registration** — enrolling a device or application as a
  trusted approval surface for an existing user
- **Session Connect** — linking an agent session to a registered client
  of an existing user

---

## Current Assumption

All current architecture documents assume that a **mis-user already
exists** before Client Registration or Session Connect is performed.

How that user came to exist is intentionally not defined yet.

---

## Out of Scope for Current Work

The following topics are explicitly deferred:

- identity proofing and verification
- account recovery flows
- credential lifecycle (passwords, passkeys, recovery codes)
- organizational accounts and multi-tenant onboarding
- legal and compliance requirements for user onboarding
- first account creation UX
- email verification, invitation flows, or self-service sign-up

These concerns will be addressed in a future architecture document
and corresponding ADR when the project reaches that stage.

---

## Why This Placeholder Exists

As the project grows, contributors and AI Coders may encounter tasks
that involve "registering" a user or device. This document ensures that:

- User Registration is not silently assumed to be solved by Client
  Registration
- Client Registration is not modified to include account creation logic
- Session Connect is not extended to handle enrollment of unknown users

These are three distinct flows with different security requirements.
They MUST remain separate.

---

## Related Documents

- [Client Registration](client-registration.md) — enrollment of a device for an existing user
- [Session Connect](session-connect.md) — agent-session binding for an existing registered client
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key model for registered clients
