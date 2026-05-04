---
read_when: working on account creation, identity proofing, onboarding flows, or user lifecycle management
---

# User Registration

## Purpose

User Registration defines how a new mis-user account is created, how the
user's identity is established, and how the first trusted client is enrolled
— all as a single atomic operation.

This document replaces the earlier placeholder and specifies the complete
registration model for the MVP.

---

## Scope

This document defines:

- the mis_user data model
- the self-service registration flow
- the relationship between User Registration and First Client Bootstrap
- security properties and requirements
- enterprise onboarding (deferred scope)

This document does NOT define:

- Additional Client Registration — see [client-registration.md](client-registration.md)
- Session Connect — see [session-connect.md](session-connect.md)
- Account Recovery — deferred; see below

---

## Core Principle

User Registration and First Client Bootstrap are one combined atomic
operation.

A mis-user without a registered client has no usable approval surface.
Separating these into two independent steps would create a meaningless
intermediate state. On successful registration:

- the mis_user record is created as `active`
- the first mis_client record is created as `active`
- the user can immediately receive and act on approval requests

No partial user state exists. The mis_user record is only created after
the full registration flow completes successfully.

---

## User Data Model

```
mis_user:
  user_id       — UUID, primary identifier, exchanged between components
  email         — verified email address; serves as notification address
  display_name  — optional, human-readable name
  status        — active | suspended
  created_at
```

### Notes

**user_id** is a UUID and serves as both the technical primary key and
the domain identifier exchanged between components (control plane, agents,
mis-clients). A UUID does not reveal database structure or user volume and
requires no separate external identifier.

**email** is verified before the mis_user record exists. It serves as the
primary identifier for login, the out-of-band notification address for
security events, and the contact address for account communications.

**status** has two values for the MVP. `pending_verification` is not
needed: the mis_user record is only created after the registration flow
completes, so the record always represents a fully verified, active account.

---

## Authentication

The sole authentication mechanism is a **passkey** (WebAuthn credential).

No password is issued or accepted. The passkey created during registration
serves as both the login credential and the WebAuthn step-up mechanism
required by ADR-0007 and ADR-0008.

Rationale: passkeys are phishing-resistant by design — they are bound to
the origin and cannot be replayed on a different domain. This is critical
for an approval surface governing AI agent actions. See ADR-0009 for the
full option analysis.

---

## Registration Flow

The flow combines email verification and passkey creation into one
continuous user journey. The magic link sent to the user's email opens
directly onto the passkey creation screen — no intermediate confirmation
page.

```
User enters email
       │
       ▼
mis-backend issues registration token (TTL-limited, single-use)
       │
       ▼
Magic link sent to email address
       │
       ▼
User clicks link → lands directly on passkey creation screen
       │
       ▼
User creates passkey (WebAuthn registration)
       │  (WebCrypto client key pair generated simultaneously)
       ▼
mis-client submits registration completion request
       │
       ▼
mis-backend validates token + WebAuthn credential
       │
       ├─ success → mis_user created (active)
       │            first mis_client created (active)
       │            security notification sent to verified email
       │
       └─ failure → no records created; user informed
```

### Step-by-Step Description

**Step 1 — Email submission**

The user navigates to the registration page and enters their email address.

**Step 2 — Token issuance and magic link delivery**

The mis-backend generates a time-limited, single-use registration token
and sends a magic link containing the token to the provided address.

To prevent user enumeration, the mis-backend returns the same response
regardless of whether the email address is already registered. If the
address is already registered, no link is sent.

**Step 3 — Magic link click**

The user opens the email and clicks the magic link. The mis-client WebApp
validates the token with the mis-backend.

- If the token is expired or invalid: the user is informed and must
  restart from Step 1.
- If the token is valid: the passkey creation screen is presented
  immediately.

**Step 4 — Passkey and client key creation**

The user creates a passkey using the platform authenticator
(WebAuthn registration ceremony). Simultaneously, the mis-client
WebApp generates a WebCrypto key pair for persistent client identity
(per ADR-0007).

Both private keys remain on the device and MUST NOT be transmitted.

**Step 5 — Registration completion request**

The mis-client submits the following to the mis-backend:

- registration token (email ownership proof)
- WebAuthn credential (passkey public key and credential ID)
- client public key (WebCrypto, for client identity)
- `client_type` = `web_pwa`
- `display_name` (auto-detected from browser/OS, or user-provided)

**Step 6 — Atomic record creation**

The mis-backend validates:

- the registration token (valid, not expired, not previously used)
- the WebAuthn registration response (credential integrity)
- the client public key (format and algorithm)

On success, the mis-backend atomically creates:

- a `mis_user` record with `status = active`
- a `mis_client` record with `status = active`, `client_type = web_pwa`,
  `assurance_level = basic`

The registration token is marked as used.

If any validation step fails, no records are created.

**Step 7 — Security notification**

The mis-backend sends a security notification to the verified email:

> "Your make-it-so account has been created and your first client
> has been enrolled."

The notification is inform-only and MUST NOT contain credentials.
See [notification-channel.md](notification-channel.md).

**Step 8 — Session established**

The user is authenticated via the newly created passkey and may
immediately begin using make-it-so.

---

## Relationship to First Client Bootstrap

The First Client Bootstrap defined in [client-registration.md](client-registration.md)
applies in two distinct contexts:

1. **Initial User Registration** — this document. User and first client
   are created together. No existing client is available to confirm.
   The bootstrap is the email magic link plus passkey creation.

2. **Recovery** — an existing user has lost all registered clients.
   The user re-bootstraps a first client without re-registering.
   This is a separate high-security flow, deferred to a future ADR.

The bootstrap mechanism is the same in both contexts: out-of-band
verified email link combined with WebAuthn/passkey creation in one flow.
The surrounding context and the records affected differ.

---

## Onboarding Model

**MVP: Self-Service**

Private individuals register themselves without administrator involvement.
The token-based email flow provides inherent protection against automated
abuse without requiring CAPTCHA.

**Future: Enterprise Self-Hosted**

A self-hosted backend instance MAY restrict self-service registration to
specific email domains via a configuration parameter (e.g.,
`allowed_email_domains`). Registration attempts from non-matching domains
are rejected at Step 2. Trust for domain-restricted registration is
delegated to the security of the company's email domain.

This is a deployment-time configuration concern and is deferred to
post-MVP.

---

## Account Recovery

Account recovery — regaining access when a user's passkey or all
registered clients are lost — is explicitly out of scope for this
document and the current architecture phase.

Recovery is a high-security flow that requires careful design to avoid
becoming an account takeover vector. It will be addressed in a future
dedicated ADR.

Until recovery is defined:

- a user who loses their passkey and all registered clients cannot
  authenticate or receive approval requests
- agents receive rejection responses directing the user to contact support

---

## Security Properties

| Property | Requirement |
|---|---|
| Authentication | Passkey only; no password accepted |
| Phishing resistance | Passkeys are origin-bound and cannot be replayed on a different domain |
| Email verification | MUST be confirmed before mis_user record is created |
| Registration token | MUST be time-limited and single-use |
| mis_user record | MUST NOT be created until full flow completes successfully |
| Client private keys | MUST NOT be transmitted to the mis-backend at any point |
| Atomic creation | mis_user and first mis_client MUST be created together or not at all |
| Security notification | MUST be sent to verified email after successful registration |
| Audit | All registration events MUST be logged |

---

## Related Documents

- [Client Registration](client-registration.md) — lifecycle and security model for registered clients
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key material model
- [Session Connect](session-connect.md) — depends on an ACTIVE registered client
- [Notification Channel](notification-channel.md) — security notification delivery
- [Use Case: Register a mis-user](../use-cases/use-case-user-registration.md) — end-to-end registration flow
- [ADR-0007: WebCrypto Client Key and WebAuthn Step-up](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md)
- [ADR-0008: Client Registration Confirmation Model](../adr/0008-client-registration-confirmation-model.md)
- [ADR-0009: User Registration Model](../adr/0009-user-registration-model.md)
