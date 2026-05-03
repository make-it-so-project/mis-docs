---
read_when: working on client enrollment security, new device confirmation, or bootstrap authentication requirements
---

# ADR-0008: Client Registration Confirmation Model

## Status

Accepted

## Date

2026-05-03

## Context

Registering a mis-client creates a persistent trusted approval surface
for a mis-user. Any device that becomes registered ACTIVE can receive and
act on approval requests on behalf of that user.

This means registering a new client is a high-value operation:
an attacker who can register a client gains persistent access to the
user's approval flow.

The question is: what authentication is required to register a new client?

Two cases must be handled differently:

1. **First client bootstrap**: the user has no existing ACTIVE registered
   client. There is no existing trusted surface to confirm the new registration.
2. **Additional client registration**: the user already has at least one
   ACTIVE registered client. That client can act as an authorizing party.

---

## Decision Drivers

- registering a new client creates a persistent trusted approval surface
  and therefore requires stronger controls than Session Connect
- additional client registration must not be grantable by password/session
  credentials alone
- first client bootstrap cannot rely on an existing trusted client
- confirmation, denial, expiry, and revocation events must be auditable
- security notifications must be sent for all client lifecycle events
- the registration model must remain practical for real users

---

## Considered Options

### Option 1: User login alone can register a new client

Any authenticated user session can add a new registered client.

#### Advantages
- lowest friction for the user

#### Disadvantages
- a stolen session token would allow an attacker to register a new
  approval device silently
- no second factor; no confirmation from a trusted device
- approval surface can be expanded by credential compromise alone

#### Assessment
Unacceptable. Login credentials alone are insufficient for an operation
that creates a persistent approval surface.

---

### Option 2: Login plus step-up can register any new client

Step-up authentication (e.g., WebAuthn/passkey) is required for any
client registration, but no confirmation from an existing device is needed.

#### Advantages
- stronger than login alone
- consistent model for both first and additional registrations

#### Disadvantages
- for additional client registration, the user's existing trusted device
  is bypassed — it never learns about the new device
- an attacker with access to the user's credentials and step-up factor
  can add a new device without any signal reaching the existing devices
- no defense-in-depth: the existing trusted surface provides no protection

#### Assessment
Insufficient for additional client registration. Step-up is necessary
but not sufficient when an existing trusted surface is available.

---

### Option 3: First client uses strong bootstrap; additional clients require step-up plus confirmation through an existing active trusted client (selected)

Treat first and additional registration separately.

**First client bootstrap:** strong authentication (e.g., WebAuthn with
user verification, out-of-band verified link). No existing client to confirm.

**Additional client registration:** step-up on the new client, plus
explicit confirmation through an existing ACTIVE registered client.

#### Advantages
- existing trusted devices act as authorization gatekeepers for new enrollments
- an attacker cannot silently add a device without the user's existing
  trusted device approving the request
- first bootstrap is handled as a special high-security case
- the trust chain for each registered device is explicit and auditable

#### Disadvantages
- additional friction when a user genuinely adds a new device
- user must have access to an existing active client to confirm
  (if unavailable, the registration is blocked; recovery is a separate flow)

#### Assessment
Best balance of security and practicality. The confirmation requirement
is the correct architectural choice for an operation that creates a
persistent trusted surface.

---

## Decision

Adopt a split confirmation model:

### First Client Bootstrap

- Requires strong authentication proving user identity and presence.
- Acceptable mechanisms include WebAuthn with user verification or
  equivalent out-of-band verified link (implementation to be confirmed).
- The mis-backend activates the first client directly upon successful
  bootstrap authentication.
- A security notification MUST be sent to the user via an out-of-band
  channel (e.g., registered email) confirming first enrollment.

### Additional Client Registration

- Requires step-up authentication on the new client.
- Additionally REQUIRES confirmation through at least one existing ACTIVE
  registered client of the same user.
- The mis-backend creates a PENDING entry and routes the confirmation
  request to existing active clients.
- The new client becomes ACTIVE only after confirmation is received.
- If the confirmation is not acted on within a defined time window,
  the PENDING entry expires.
- If confirmation is denied, the registration is rejected.

### Audit and Notification Requirements

All of the following events MUST be logged:

- registration request received (PENDING created)
- confirmation request sent to existing clients
- confirmation approved
- confirmation denied
- registration expired
- client activated (ACTIVE)
- client revoked (REVOKED)

All of the following events MUST trigger security notifications
to active clients:

- new client registration pending (to existing active clients)
- new client activated (to all active clients including new one)
- registration denied (to user)
- registration expired (to user)
- client revoked (to remaining active clients)

Notifications are inform-only and MUST NOT carry credentials or
approval decisions. See [notification-channel.md](../architecture/notification-channel.md).

### Recovery Boundary

If no existing ACTIVE registered client is available to confirm a new
registration, the registration is blocked. Account recovery after loss
of all clients is a separate future high-security flow and is explicitly
out of scope here.

---

## Consequences

### Positive Consequences

- adding a new approval device requires explicit confirmation from
  an existing trusted device
- silent device registration by an attacker requires compromising
  both credentials AND an existing registered device
- all lifecycle events are auditable
- the model scales cleanly as users add more devices

### Negative Consequences

- users must have access to an existing device to confirm new registrations
- if all devices are lost, the user is blocked from adding new devices
  until recovery is defined

### Follow-up Implications

- account recovery after total loss of registered clients must be defined
  in a future high-security ADR
- the confirmation time window (PENDING expiry) must be defined during
  implementation
- the Policy Engine may later define per-action-type requirements for
  which `assurance_level` of confirming client is acceptable

---

## Rationale Summary

Registering a new mis-client creates a persistent trusted approval surface.
This warrants stronger controls than login credentials alone. Using an
existing trusted device as the confirming party for additional registrations
establishes a clear trust chain and ensures that adding a new device
requires the user's active participation through an already-trusted endpoint.
