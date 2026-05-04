---
read_when: working on client enrollment, device trust, first client bootstrap, or client lifecycle management
---

# Client Registration

## Purpose

Client Registration defines how a mis-client becomes a persistently registered
trusted approval surface for a mis-user.

A registered mis-client is the device or application through which a user
receives and acts on approval requests. It is the endpoint that the control
plane trusts to collect and transmit human approval decisions.

---

## Problem

The Session Connect mechanism assumes that a `client_id` already exists in
`registered_clients` for the given user. It does not define how a client
enters that set.

Without a defined registration process:

- the trust basis for the approved client list is undefined
- there is no lifecycle model (active, revoked, expired)
- additional clients could be added without appropriate confirmation controls
- first-time users have no defined path to enroll their first device

Client Registration fills this gap.

---

## Scope

This document defines:

- the lifecycle and states of a registered mis-client
- the first client bootstrap flow
- the additional client registration flow
- deregistration and revocation
- security notification requirements

This document does NOT define:

- User Registration or account creation — see [user-registration.md](user-registration.md)
- Session Connect — see [session-connect.md](session-connect.md)
- Client identity and key material — see [client-identity-and-secure-communication.md](client-identity-and-secure-communication.md)
- Approval flows and policy evaluation — see [control-plane.md](control-plane.md)

---

## Core Concept

A **registered mis-client** is a persistent entry in the mis-backend that
associates a device or application instance with a mis-user.

```
registered_clients:
  client_id        — unique identifier
  user_id          — owning mis-user
  client_type      — web_pwa | ios_native | android_native
  assurance_level  — basic | standard | high
  public_key       — client public key for identity verification
  status           — pending | active | revoked
  display_name     — human-readable label for this client
  created_at
  last_seen_at
  revoked_at
```

Registration is distinct from Session Connect:

| Concept | Purpose | Frequency | Security level |
|---|---|---|---|
| Client Registration | Enroll a new trusted device | Rare, explicit | High |
| Session Connect | Link an agent session to a registered client | Per session | Low |

Session Connect MUST NOT register new clients. It selects from the
`registered_clients` list; it does not add to it.

---

## Client Lifecycle

```
[start]
   │
   ▼
PENDING
   │  confirmation received (or bootstrap complete)
   ▼
ACTIVE  ◄──────────────────────────────────┐
   │                                        │
   │  user or system revokes               │
   ▼                                        │
REVOKED                           (re-registration would be a new entry)
```

### PENDING

The client has been created in the backend but is not yet authorized to
receive or act on approval requests.

Possible causes:
- awaiting confirmation from an existing active client
- awaiting strong bootstrap authentication to complete

A PENDING client MUST NOT receive approval requests.

### ACTIVE

The client is fully registered and authorized to receive and act on
approval requests for the associated user.

### REVOKED

The client has been explicitly deregistered. Revocation is permanent for
that registration entry. A new registration would create a new `client_id`.

Revoked clients MUST NOT receive approval requests.

Optional future states (not in MVP scope):
- `suspended` — temporarily disabled but not permanently revoked
- `lost_reported` — user has reported this client as potentially compromised
- `registration_denied` — registration was rejected during confirmation
- `registration_expired` — registration timed out before confirmation

---

## Client Registration vs Session Connect

Session Connect operates on the assumption that the client is already ACTIVE.

The mis-backend MUST validate client status before accepting a Connect:

- if `client_id` is not in `registered_clients` → reject
- if client status is not ACTIVE → reject, recommend re-registration

Session Connect does not grant, modify, or verify registration status.
It is a session-level binding mechanism, not an enrollment mechanism.

---

## First Client Bootstrap

First Client Bootstrap applies when a mis-user has no existing ACTIVE
registered client. Because no trusted surface is available to confirm
the registration, the bootstrap MUST rely on a stronger authentication
mechanism to establish initial trust.

This situation arises in two distinct contexts:

**Context 1 — Initial User Registration**

The user has no account yet. First Client Bootstrap is performed as
part of the User Registration flow. The mis_user record and the first
mis_client record are created atomically in a single operation.
See [User Registration](user-registration.md).

**Context 2 — Recovery**

The user has an existing account but all registered clients have been
lost or revoked. First Client Bootstrap re-establishes a trusted
approval surface for the existing user, without creating a new account.
Recovery is a high-security flow defined in ADR-0010.
See [account-recovery.md](account-recovery.md).

### Bootstrap Mechanism

The accepted bootstrap mechanism is defined in ADR-0009:

A time-limited, single-use magic link is sent to the user's verified
email address. Clicking the link opens directly onto a passkey creation
screen (WebAuthn registration ceremony). Email verification and passkey
creation form one continuous flow; both are required.

This satisfies the two-layer requirement:
- the magic link proves ownership of the registered email address
  (out-of-band verified link)
- the passkey creation proves user presence via the platform authenticator
  (WebAuthn with user verification)

The bootstrap flow MUST be treated as a high-security operation.
It MUST be logged and MUST trigger a security notification to the user
via the verified email address confirming that a first client was enrolled.

---

## Additional Client Registration

When a mis-user already has at least one ACTIVE registered client,
registering a new client MUST require step-up authentication on the new
client and MUST require confirmation from at least one existing ACTIVE
trusted client of the same user.

This ensures that:

- an attacker who gains access to user credentials alone cannot silently
  add a new approval device
- the user's existing trusted device acts as the authorizing party

Flow:

```
New client (Client B)
  │  generates key material
  │  user initiates registration on Client B
  ▼
mis-backend
  │  creates PENDING entry for Client B
  │  sends confirmation request to one or more existing ACTIVE clients
  ▼
At least one existing trusted client
  │  user reviews: "New client '[display name]' is requesting registration."
  │  user approves or denies
  ▼
mis-backend
  │  if approved: sets Client B status → ACTIVE
  │  if denied: sets Client B status → registration_denied (future state)
  │  sends security notifications to all active clients
  ▼
Client B is now ACTIVE.
```

If no existing ACTIVE client is available, the flow is not Additional
Client Registration and MUST fall back to Account Recovery.

While status is PENDING, the new client MUST NOT receive approval requests.
Only after successful confirmation may the status transition to ACTIVE.

If the confirmation is not acted on within a defined time window,
the PENDING entry expires. The registration must be retried.

---

## Deregistration and Revocation

A user MAY revoke any of their registered clients at any time.

Revocation:

- immediately sets the client status to REVOKED
- prevents further approval requests from being routed to that client
- does NOT invalidate any in-flight action requests already approved
- MUST be logged with timestamp and initiating context
- MUST trigger security notifications to all remaining active clients

Revocation may be initiated:

- by the user through any active trusted client
- by the mis-backend in response to a security event (future scope)

Deregistration of the last active client leaves the user without a
trusted approval surface. See [account-recovery.md](account-recovery.md).

---

## Recovery

Recovery after loss of all registered clients is out of scope for this
Client Registration document. It is defined separately in
[account-recovery.md](account-recovery.md) and ADR-0010.

When a user has no ACTIVE clients, approvals cannot be routed until Account
Recovery re-establishes an ACTIVE client.

---

## Security Notifications

The following client lifecycle events MUST trigger security notifications
to the affected user:

| Event | Notify |
|---|---|
| First client bootstrap complete | User (via out-of-band channel) |
| New client registration pending | All existing active clients |
| New client activated | All active clients (including new one) |
| Registration denied | User |
| Registration timed out / expired | User |
| Client revoked | All remaining active clients |

Security notifications are inform-only. They MUST NOT contain credentials,
approval decisions, or action data. See [notification-channel.md](notification-channel.md).

---

## Security Properties

| Property | Requirement |
|---|---|
| First bootstrap | MUST use strong authentication; treated as high-security |
| Additional client | MUST require step-up authentication on the new client and confirmation from at least one existing ACTIVE client |
| PENDING clients | MUST NOT receive approval requests |
| REVOKED clients | MUST NOT receive approval requests |
| Revocation | MUST be immediate; MUST be logged; MUST notify remaining clients |
| Audit | All registration, confirmation, denial, and revocation events MUST be logged |

---

## Related Documents

- [Session Connect](session-connect.md) — uses registered clients; does not register them
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key material model for registered clients
- [User Registration](user-registration.md) — combined User Registration and First Client Bootstrap flow for new accounts
- [Control Plane Architecture](control-plane.md) — validates client status before routing approvals
- [Notification Channel](notification-channel.md) — delivers client lifecycle security notifications
- [Use Case: Register Client](../use-cases/use-case-register-client.md) — end-to-end registration flow for additional clients
- [Use Case: Register a mis-user](../use-cases/use-case-user-registration.md) — end-to-end flow for new account creation with first client
- [ADR-0008: Client Registration Confirmation Model](../adr/0008-client-registration-confirmation-model.md) — decision record for the confirmation approach
- [ADR-0009: User Registration Model](../adr/0009-user-registration-model.md) — decision record for passkey-only auth and combined registration flow
